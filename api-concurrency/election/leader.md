# Leader 方法

Leader 方法用于信息查询：

```java
// Leader 返回当前的选举公告，如果有。
rpc Leader(LeaderRequest) returns (LeaderResponse) {}
```

## 消息定义

请求的消息体是 `LeaderRequest`：

```java
message LeaderRequest {
  // name 是选举标识符,用于查询领导地位信息的
  bytes name = 1;
}
```

应答的消息体是 `LeaderResponse`：

```java
message LeaderResponse {
  etcdserverpb.ResponseHeader header = 1;
  // kv 是键值对，体现最后的领导者更新
  mvccpb.KeyValue kv = 2;
}
```

`mvccpb.KeyValue` 来自 `kv.proto`，消息体定义为：

```java
message KeyValue {
  // key 是 bytes 格式的 key。不容许 key 为空。
  bytes key = 1;

  // create_revision 是这个 key 最后一次创建的修订版本
  int64 create_revision = 2;

  // mod_revision 是这个 key 最后一次修改的修订版本
  int64 mod_revision = 3;

  // version 是 key 的版本。删除会重置版本为0,而任何 key 的修改会增加它的版本。
  int64 version = 4;

  // value 是 key 持有的值，bytes 格式。
  bytes value = 5;

  // lease 是附加给 key 的租约 id。
  // 当附加的租约过期时，key 将被删除。
  // 如果 lease 为0,则没有租约附加到 key。
  int64 lease = 6;
}
```

