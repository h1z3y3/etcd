# Compact 方法

Compact 方法压缩 etcd 键值对存储中的事件历史。

键值对存储应该定期压缩，否则事件历史会无限制的持续增长.

```java
rpc Compact(CompactionRequest) returns (CompactionResponse) {}
```

## 消息体

请求的消息体是 `CompactionRequest`， CompactionRequest 压缩键值对存储到给定修订版本。所有修订版本比压缩修订版本小的键都将被删除：

```java
message CompactionRequest {
  // 键值存储的修订版本，用于比较操作
  int64 revision = 1;

  // physical设置为 true 时 RPC 将会等待直到压缩物理性的应用到本地数据库，到这程度被压缩的项将完全从后端数据库中移除。
  bool physical = 2;
}
```

应答的消息体是 `CompactionResponse`：

```java
message CompactionResponse {
  ResponseHeader header = 1;
}
```
