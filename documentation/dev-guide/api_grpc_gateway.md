# gRPC网关

## 为什么用 grpc-gateway

etcd v3 使用 [gRPC](http://www.grpc.io/) 作为它的消息协议。etcd 项目包括基于 gRPC 的 [Go client](https://github.com/coreos/etcd/tree/master/clientv3) 和 命令行工具 [etcdctl](https://github.com/coreos/etcd/tree/master/etcdctl)，通过 gRPC 和 etcd 集群通讯。对于不支持 gRPC 支持的语言，etcd 提供 JSON 的 [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)。这个网关提供 RESTful 代理，翻译 HTTP/JSON 请求为 gRPC 消息。

## 使用 grpc-gateway

网关接受 etcd 的 [protocol buffer](api_reference_v3.md) 消息定义的 [JSON mapping](https://developers.google.com/protocol-buffers/docs/proto3#json) 。注意  `key` 和 `value` 字段被定义为 byte 数组，因此必须在 JSON 中以 base64 编码。下面例子使用 curl，但是任何 HTTP/JSON 客户端都可以如此工作。

### 设置和获取键

使用 `v3alpha/kv/range` 和 `v3alpha/kv/put` 服务来读取和写入键:

```bash
# https://www.base64encode.org/
# foo is 'Zm9v' in Base64
# bar is 'YmFy'

curl -L http://localhost:2379/v3alpha/kv/put \
	-X POST -d '{"key": "Zm9v", "value": "YmFy"}'
# {"header":{"cluster_id":"12585971608760269493","member_id":"13847567121247652255","revision":"2","raft_term":"3"}}

curl -L http://localhost:2379/v3alpha/kv/range \
	-X POST -d '{"key": "Zm9v"}'
# {"header":{"cluster_id":"12585971608760269493","member_id":"13847567121247652255","revision":"2","raft_term":"3"},"kvs":[{"key":"Zm9v","create_revision":"2","mod_revision":"2","version":"1","value":"YmFy"}],"count":"1"}
```

### 观察键

使用 `v3alpha/watch` 服务来观察键:

```bash
curl http://localhost:2379/v3alpha/watch \
        -X POST -d '{"create_request": {"key":"Zm9v"} }' &
# {"result":{"header":{"cluster_id":"12585971608760269493","member_id":"13847567121247652255","revision":"1","raft_term":"2"},"created":true}}

curl -L http://localhost:2379/v3alpha/kv/put \
	-X POST -d '{"key": "Zm9v", "value": "YmFy"}' >/dev/null 2>&1
# {"result":{"header":{"cluster_id":"12585971608760269493","member_id":"13847567121247652255","revision":"2","raft_term":"2"},"events":[{"kv":{"key":"Zm9v","create_revision":"2","mod_revision":"2","version":"1","value":"YmFy"}}]}}
```

### 事务

使用 `v3alpha/kv/txn` 发起事务:

```bash
curl -L http://localhost:2379/v3alpha/kv/txn \
	-X POST \
	-d '{"compare":[{"target":"CREATE","key":"Zm9v","createRevision":"2"}],"success":[{"requestPut":{"key":"Zm9v","value":"YmFy"}}]}'
# {"header":{"cluster_id":"12585971608760269493","member_id":"13847567121247652255","revision":"3","raft_term":"2"},"succeeded":true,"responses":[{"response_put":{"header":{"revision":"3"}}}]}
```

### 认证

使用 `v3alpha/auth` 服务搭建认证:

```bash
# 创建 root 用户
curl -L http://localhost:2379/v3alpha/auth/user/add \
	-X POST -d '{"name": "root", "password": "pass"}'
# {"header":{"cluster_id":"14841639068965178418","member_id":"10276657743932975437","revision":"1","raft_term":"2"}}

# 创建 root 角色
curl -L http://localhost:2379/v3alpha/auth/role/add \
	-X POST -d '{"name": "root"}'
# {"header":{"cluster_id":"14841639068965178418","member_id":"10276657743932975437","revision":"1","raft_term":"2"}}

# 授予 root 角色
curl -L http://localhost:2379/v3alpha/auth/user/grant \
	-X POST -d '{"user": "root", "role": "root"}'
# {"header":{"cluster_id":"14841639068965178418","member_id":"10276657743932975437","revision":"1","raft_term":"2"}}

# 开启认证
curl -L http://localhost:2379/v3alpha/auth/enable -X POST -d '{}'
# {"header":{"cluster_id":"14841639068965178418","member_id":"10276657743932975437","revision":"1","raft_term":"2"}}
```

使用 `v3alpha/auth/authenticate` 为认证 token 做认证:

```bash
# get the auth token for the root user
curl -L http://localhost:2379/v3alpha/auth/authenticate \
	-X POST -d '{"name": "root", "password": "pass"}'
# {"header":{"cluster_id":"14841639068965178418","member_id":"10276657743932975437","revision":"1","raft_term":"2"},"token":"sssvIpwfnLAcWAQH.9"}
```

Set the `Authorization` header to the authentication token to fetch a key using authentication credentials:

```bash
curl -L http://localhost:2379/v3alpha/kv/put \
	-H 'Authorization : sssvIpwfnLAcWAQH.9' \
	-X POST -d '{"key": "Zm9v", "value": "YmFy"}'
# {"header":{"cluster_id":"14841639068965178418","member_id":"10276657743932975437","revision":"2","raft_term":"2"}}
```


## Swagger

生成的 [Swagger](http://swagger.io/) API 定义可以在 [rpc.swagger.json](https://github.com/coreos/etcd/blob/master/Documentation/dev-guide/apispec/swagger/rpc.swagger.json) 找到.

