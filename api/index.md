# API 参考文档

## 前言

内容来自 ectd3 的 [API reference](https://github.com/coreos/etcd/blob/master/Documentation/dev-guide/api_reference_v3.md)。

原文内容是从 .proto 文件生成的，service和方法定义/message定义一起构成一个庞大的单页HTML文件，不利于阅读。因此在翻译时，遵循gitbook的习惯，按照服务/方法分开形成章节结构。

主要内容在 message 的字段定义上，翻译时没有复制原文中从 .proto 文件生成的表格，而是直接在 .proto 文件的 message 定义上翻译，感觉更直观一些。

> 注： 需要对 proto3 的基本语法有初步的了解，可以查阅 [proto3 中文文档](https://proto3.doczh.cn)。

## 内容

已经整理并翻译的内容：

* [KV service](kv/kv_service.md)
* [Watch service](watch/watch_service.md)
* [Lease service](lease/lease_service.md)

