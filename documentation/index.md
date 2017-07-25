# 官方文档

> 注：内容翻译自 https://github.com/coreos/etcd/blob/master/Documentation

etcd 是一个分布式键值对存储，设计用来可靠而快速的保存关键数据并提供访问。通过分布式锁，leader选举和写屏障(write barriers)来实现可靠的分布式协作。etcd集群是为高可用，持久性数据存储和检索而准备。

## 开始

现在 etcd 的用户和开发者可以从 [下载并构建](https://github.com/coreos/etcd/blob/master/Documentation/dl_build.md) etcd开始。在获取etcd之后，参照 [quick demo](https://github.com/coreos/etcd/blob/master/Documentation/demo.md) 来感受构建和操作etcd集群的基本方式。

## 使用 etcd 开发

开始使用 etcd 作为分布式键值存储的最简单的方式是 [搭建本地集群](dev-guide/local_cluster.md)

- [搭建本地集群](dev-guide/local_cluster.md)
- [和 etcd 交互](dev-guide/interacting_v3.md)
- gRPC [etcd core](dev-guide/api_reference_v3.md) 和 [etcd concurrency](dev-guide/api_concurrency_reference_v3.md) API 参考文档
- [经由 gRPC 网关的HTTP JSON API](dev-guide/api_grpc_gateway.md)
- [gRPC 命名和发现](dev-guide/grpc_naming.md)
- [客户端](clientv3/namespace) 和 [代理](op-guide/grpc_proxy.md#namespacing) 命名空间
- [内嵌的etcd](https://godoc.org/github.com/coreos/etcd/embed)
- [试验性的特性和 API](dev-guide/experimental_apis.md)
- [系统限制](dev-guide/dev-guide/limit.md)

## 操作 etcd 集群

管理员，需要为支持的开发人员创建可靠而可扩展的键值存储，应该从 [多机集群](op-guide/clustering.md) 开始.

### 搭建 etcd

- [配置](op-guide/configuration.md)
- [多成员集群](op-guide/clustering.md)
- [在容器内运行etcd集群](op-guide/container.md)
- [gRPC代理(TBD)](op-guide/grpc_proxy.md)
- [L4 网关](op-guide/gateway.md)

### 系统配置

- [支持平台](op-guide/supported-platform.md)
- [硬件推荐(TBD)](op-guide/hardware.md)
- [性能评测](op-guide/performance.md)
- [调优(TBD)](../tuning.md)

### 平台指南

> 注：暂时未翻译这部分内容

- [Amazon Web Services](https://github.com/coreos/etcd/blob/master/Documentation/platforms/aws.md)
- [Container Linux, systemd](https://github.com/coreos/etcd/blob/master/Documentation/platforms/container-linux-systemd.md)
- [FreeBSD](https://github.com/coreos/etcd/blob/master/Documentation/platforms/freebsd.md)
- [Docker container](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/container.md#docker)
- [rkt container](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/container.md#rkt)

### 安全

－ [TLS](op-guide/security.md)
－ [基于角色的访问控制(TBD)](op-guide/authentication.md)

### 维护和排错

- [常见问题(TBD)](../faq.md)
- [监控(TBD)](op-guide/monitoring.md)
- [维护](op-guide/maintenance.md)
- [故障模式](op-guide/failures.md)
- [灾难恢复](op-guide/recovery.md)

### 升级和兼容

> 注：暂时未翻译这部分内容

- [版本](op-guide/versioning.md)
- [Migrate applications from using API v2 to API v3](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/v2-migration.md)
- [Upgrading a v2.3 cluster to v3.0](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_0.md)
- [Upgrading a v3.0 cluster to v3.1](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md)
- [Upgrading a v3.1 cluster to v3.2](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_2.md)

## 学习

要学习更多 etcd 背后的概念和内部细节，请阅读下面的内容:

- [为什么是etcd?](learning/why.md)
- [理解数据模型](learning/data_model.md)
- [理解API](learning/api.md)
- [术语](learning/glossary.md)
- [API保证](learning/api_guarantees.md)
- Internals
	- [认证子系统(TBD)](learning//auth_design.md)

