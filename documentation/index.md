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

- [搭建etcd集群](op-guide/clustering.md)
- [搭建etcd网关](op-guide/gateway.md)
- [在容器内运行etcd集群](op-guide/container.md)
- [配置](op-guide/configuration.md)
- [加密(TODO)](op-guide/security.md)
- Monitoring
- [维护](op-guide/maintenance.md)
- [理解失败](op-guide/failures.md)
- [灾难恢复](op-guide/recovery.md)
- [性能](op-guide/performance.md)
- [版本](op-guide/versioning.md)
- [支持平台](op-guide/supported-platform.md)

## 学习

要学习更多 etcd 背后的概念和内部细节，请阅读下面的内容:

- Why etcd (TODO)
- [理解数据模型](leaning/data_model.md)
- [理解API](leaning/api.md)
- [术语](leaning/glossary.md)
- [API保证](leaning/api_guarantees.md)
- Internals (TODO)

## FAQ

回答关于 etcd 的[常见问题](faq.md)
