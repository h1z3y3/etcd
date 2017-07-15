# Put 方法

Put 方法设置指定 key 到键值存储.

Put 方法增加键值存储的修订版本并在事件历史中生成一个事件.

```java
rpc Put(PutRequest) returns (PutResponse) {}
```

## 消息体

请求的消息体是 `PutRequest`：

```java
message PutRequest {
  // byte 数组形式的 key，用来保存到键值对存储
  bytes key = 1;

  // byte 数组形式的 value，在键值对存储中和 key 关联
  bytes value = 2;

  // 在键值存储中和 key 关联的租约id。0代表没有租约。
  int64 lease = 3;

  // 如果 prev_kv 被设置，etcd 获取改变之前的上一个键值对。
  // 上一个键值对将在 put 应答中被返回
  bool prev_kv = 4;

  // 如果 ignore_value 被设置, etcd 使用它当前的 value 更新 key.
  // 如果 key 不存在，返回错误.
  bool ignore_value = 5;

  // 如果 ignore_lease 被设置, etcd 使用它当前的租约更新 key.
  // 如果 key 不存在，返回错误.
  bool ignore_lease = 6;
}
```

应答的消息体是` PutResponse`：

```java
message PutResponse {
  ResponseHeader header = 1;

  // 如果请求中的 prev_kv 被设置，将会返回上一个键值对
  mvccpb.KeyValue prev_kv = 2;
}
```