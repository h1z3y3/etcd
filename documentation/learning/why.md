# 为什么是etcd?

"etcd"这个名字源于两个想法，即　unix "/etc" 文件夹和分布式系统"d"istibuted。 "/etc" 文件夹为单个系统存储配置数据的地方，而 etcd 存储大规模分布式系统的配置信息。因此，"d"istibuted　的 "/etc" ，是为 "etcd"。

etcd 以一致和容错的方式存储元数据。分布式系统使用 etcd 作为一致性键值存储，用于配置管理，服务发现和协调分布式工作。使用 etcd 的通用分布式模式包括领导选举，[分布式锁][etcd-concurrency]和监控机器活动。

## 使用案例

- CoreOS 的容器 Linux: 在[Container Linux][container-linux]上运行的应用程序获得自动的不宕机 Linux 内核更新。 容器 Linux 使用[locksmith]来协调更新。locksmith 在 etcd 上实现分布式信号量，确保在任何给定时间只有集群的一个子集重新启动。
- [Kubernetes][kubernetes] 将配置数据存储到etcd中，用于服务发现和集群管理; etcd 的一致性对于正确安排和运行服务至关重要。Kubernetes API 服务器将群集状态持久化在 etcd 中。它使用etcd的 watch API监视集群，并发布关键的配置更改。

## 功能和系统比较

TODO

[etcd-concurrency]: https://godoc.org/github.com/coreos/etcd/clientv3/concurrency
[container-linux]: https://coreos.com/why
[locksmith]: https://github.com/coreos/locksmith
[kubernetes]: http://kubernetes.io/docs/whatisk8s

