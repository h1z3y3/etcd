# etcd v3 认证设计

## 为什么不重用v2的认证系统?

v3 协议使用 gRPC 传输而不是像 v2 这样的 RESTful 接口。这个新协议提供了迭代和改进v2设计的机会。例如，v3 auth具有基于连接的身份验证，而不是v2的每请求的速度较慢的认证。 此外，在实践中，v2 auth的语义在关于一致性的推理方面有些笨重，将在下一节中描述。对于v3，认证机制有明确定义的描述和实现，可以修复v2认证系统的缺陷。

### 功能需求

* 每连接认证，而不是每请求
   * 基于用户ID + 密码的认证，实现为 gRPC API
   * 在认证政策修改之后，认证必须刷新
* 功能应该和v2一样简单
   * v3 提供扁平键空间，和v2的目录结构不同。提供权限检查,如间隔匹配(as interval matching)
* 应该提供比v2认证更强的一致性保证

### 主要需要更改

* 客户端必须在发送被验证的请求之前创建仅用于认证的专用连接
* 添加权限信息(用户 ID 和 合法 revision) 到 Raft 命令 (`etcdserverpb.InternalRaftRequest`)
* 在状态机层做每个请求的权限检查，而不是在 API 层

### 权限元数据一致性

认证的元数据也应该在存储中存储和管理，该存储被etcd的Raft协议控制，和其他在etcd中的数据一样。要求不牺牲整个etcd集群的可用性和一致性。如果读取或写入元数据（例如权限信息）需要每个节点（超过法定人数）的同意，则单节点故障会让整个集群停止。要求所有节点立即同意意味着，如果任意集群成员宕机，即使群集具有可用的法定人数，检查普通的读/写请求也无法完成。 这种全场一致方案最终会降低集群的可用性; 从raft而来的基于法定人数的共识就足够了，因为合约遵循一致的顺序。

The authentication mechanism in the etcd v2 protocol has a tricky part because the metadata consistency should work as in the above, but does not: each permission check is processed by the etcd member that receives the client request (etcdserver/api/v2http/client.go), including follower members. Therefore, it's possible the check may be based on stale metadata.


This staleness means that auth configuration cannot be reflected as soon as operators execute etcdctl. Therefore there is no way to know how long the stale metadata is active. Practically, the configuration change is reflected immediately after the command execution. However, in some cases of heavy load, the inconsistent state can be prolonged and it might result in counter-intuitive situations for users and developers. It requires a workaround like this: https://github.com/coreos/etcd/pull/4317#issuecomment-179037582

### Inconsistent permissions are unsafe for linearized requests

Inconsistent authentication state is most serious for writes. Even if an operator disables write on a user, if the write is only ordered with respect to the key value store but not the authentication system, it's possible the write will complete successfully. Without ordering on both the auth store and the key-value store, the system will be susceptible to stale permission attacks.

Therefore, the permission checking logic should be added to the state machine of etcd. Each state machine should check the requests based on its permission information in the apply phase (so the auth information must not be stale).

## Design and implementation

### Authentication

At first, a client must create a gRPC connection only to authenticate its user ID and password. An etcd server will respond with an authentication reply. The reponse will be an authentication token on success or an error on failure. The client can use its authentication token to present its credentials to etcd when making API requests.

The client connection used to request the authentication token is typically thrown away; it cannot carry the new token's credentials. This is because gRPC doesn't provide a way for adding per RPC credential after creation of the connection (calling `grpc.Dial()`). Therefore, a client cannot assign a token to its connection that is obtained through the connection. The client needs a new connection for using the token.

#### Notes on the implementation of `Authenticate()` RPC

`Authenticate()` RPC generates an authentication token based on a given user name and password. etcd saves and checks a configured password and a given password using Go's `bcrypt` package. By design, `bcrypt`'s password checking mechanism is computationally expensive, taking nearly 100ms on an ordinary x64 server. Therefore, performing this check in the state machine apply phase would cause performance trouble: the entire etcd cluster can only serve almost 10 `Authenticate()` requests per second.

For good performance, the v3 auth mechanism checks passwords in etcd's API layer, where it can be parallelized outside of raft. However, this can lead to potential time-of-check/time-of-use (TOCTOU) permission lapses:
1. client A sends a request `Authenticate()`
1. the API layer processes the password checking part of `Authenticate()`
1. another client B sends a request of `ChangePassword()` and the server completes it
1. the state machine layer processes the part of getting a revision number for the `Authenticate()` from A
1. the server returns a success to A
1. now A is authenticated on an obsolete password

For avoiding such a situation, the API layer performs *version number validation* based on the revision number of the auth store. During password checking, the API layer saves the revision number of auth store. After successful password checking, the API layer compares the saved revision number and the latest revision number. If the numbers differ, it means someone else updated the auth metadata. So it retries the checking. With this mechanism, the successful password checking based on the obsolete password can be avoided.

### Resolving a token in the API layer

After authenticating with `Authenticate()`, a client can create a gRPC connection as it would without auth. In addition to the existing initialization process, the client must associate the token with the newly created connection. `grpc.WithPerRPCCredentials()` provides the functionality for this purpose.

Every authenticated request from the client has a token. The token can be obtained with `grpc.metadata.FromIncomingContext()` in the server side. The server can obtain who is issuing the request and when the user was authorized. The information will be filled by the API layer in the header (`etcdserverpb.RequestHeader.Username` and `etcdserverpb.RequestHeader.AuthRevision`) of a raft log entry (`etcdserverpb.InternalRaftRequest`).

### Checking permission in the state machine

The auth info in `etcdserverpb.RequestHeader` is checked in the apply phase of the state machine. This step checks the user is granted permission to requested keys on the latest revision of auth store.

### Two types of tokens: simple and JWT

There are two kinds of token types: simple and JWT. The simple token isn't designed for production use cases. Its tokens aren't cryptographically signed and servers must statefully track token-user correspondence; it is meant for development testing.  JWT tokens should be used for production deployments since it is cryptographically signed and verified. From the implementation perspective, JWT is stateless. Its token can include metadata including username and revision, so servers don't need to remember correspondence between tokens and the metadata.

## Notes on the difference between KVS models and file system models

etcd v3 is a KVS, not a file system. So the permissions can be granted to the users in form of an exact key name or a key range like `["start key", "end key")`. It means that granting a permission of a nonexistent key is possible. Users should care about unintended permission granting. In a case of file system like system (e.g. Chubby or ZooKeeper), an inode like data structure can include the permission information. So granting permission to a nonexist key won't be possible (except the case of sticky bits).

The etcd v3 model requires multiple lookup of the metadata unlike the file system like systems. The worst case lookup cost will be sum the user's total granted keys and intervals. The cost cannot be avoided because v3's flat key space is completely different from Unix's file system model (every inode includes permission metadata). Practically the cost won’t be a serious problem because the metadata is small enough to benefit from caching.
