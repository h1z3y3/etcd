# gRPC 命名与发现

etcd 提供了一个 gRPC 解析器来支持替代名称系统，从 etcd 获取端点以发现 gRPC 服务。底层的机制是基于观察带有服务名称前缀的键的更新。

## 用 go-grpc 使用etcd发现

etcd 客户端提供了一个 gRPC 解析器，用于使用 etcd 后端解析 gRPC 端点。解析器使用 etcd 客户端初始化并给出解析解析的目标：

```go
import (
	"github.com/coreos/etcd/clientv3"
	etcdnaming "github.com/coreos/etcd/clientv3/naming"

	"google.golang.org/grpc"
)

...

cli, cerr := clientv3.NewFromURL("http://localhost:2379")
r := &etcdnaming.GRPCResolver{Client: cli}
b := grpc.RoundRobin(r)
conn, gerr := grpc.Dial("my-service", grpc.WithBalancer(b))
```

## 管理服务端点

etcd 解析器将解析目标加"/"(如"my-service/")的前缀下，并带有 json编码 go-grpc `naming.Update` 值的所有键作为潜在的服务端点对待。通过创建新的键将端点添加到服务中，并通过删除键从服务中删除他们。

### 添加端点

新的端点可以通过 `etcdctl` 添加到服务中：

```bash
ETCDCTL_API=3 etcdctl put my-service/1.2.3.4 '{"Addr":"1.2.3.4","Metadata":"..."}'
```

etcd 客户端的 `GRPCResolver.Update` 方法可以同样用匹配 'Addr' 的键注册新的端点：

```go
r.Update(context.TODO(), "my-service", naming.Update{Op: naming.Add, Addr: "1.2.3.4", Metadata: "..."})
```

### 删除端点

通过 `etcdctl` 可以从服务中删除主机：

```sh
ETCDCTL_API=3 etcdctl del my-service/1.2.3.4
```

etcd 客户端的 `GRPCResolver.Update` 方法可以同样支持删除端点：

```go
r.Update(context.TODO(), "my-service", naming.Update{Op: naming.Delete, Addr: "1.2.3.4"})
```

### 带租约注册端点

带租约注册端点确保如果主机不能维持 keepalive 心跳（例如，它的机器宕机了），它将从服务中删除：

```bash
lease=`ETCDCTL_API=3 etcdctl lease grant 5 | cut -f2 -d' '`
ETCDCTL_API=3 etcdctl put --lease=$lease my-service/1.2.3.4 '{"Addr":"1.2.3.4","Metadata":"..."}'
ETCDCTL_API=3 etcdctl lease keep-alive $lease
```
