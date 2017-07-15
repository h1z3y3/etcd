# LeaseTimeToLive 方法

LeaseTimeToLive 方法获取租约的信息.

```java
rpc LeaseTimeToLive(LeaseTimeToLiveRequest) returns (LeaseTimeToLiveResponse) {}
```

## 消息体

请求的消息体是 `LeaseTimeToLiveRequest`：

```java
message LeaseTimeToLiveRequest {
  // ID 是租约的 ID.
  int64 ID = 1;
  // keys 设置为 true 可以查询附加到这个租约上的所有 key
  bool keys = 2;
}
```

应答的消息体是 `LeaseTimeToLiveResponse`：

```java
message LeaseTimeToLiveResponse {
  ResponseHeader header = 1;
  // ID 是来自请求的 ID.
  int64 ID = 2;
  // TTL 是租约剩余的 TTL，单位为秒；租约将在接下来的 TTL + 1 秒之后过期
  int64 TTL = 3;
  // GrantedTTL 是租约创建/续约时初始授予的时间，单位为秒
  int64 grantedTTL = 4;
  // keys 是附加到这个租约的 key 的列表
  repeated bytes keys = 5;
}
```


