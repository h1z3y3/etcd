# LeaseGrant 方法

LeaseGrant 方法创建一个租约.

```java
rpc LeaseGrant(LeaseGrantRequest) returns (LeaseGrantResponse) {}
```

## 消息体

请求的消息体是 `LeaseGrantRequest`：

```java
message LeaseGrantRequest {
  // TTL 是建议的以秒为单位的 time-to-live
  int64 TTL = 1;

  // ID 是租约的请求ID。如果ID设置为0, 则出租人(也就是etcd server)选择一个ID。
  int64 ID = 2;
}
```

应答的消息体是 `LeaseGrantResponse`：

```java
message LeaseGrantResponse {
  ResponseHeader header = 1;
  // ID 是承认的租约的ID
  int64 ID = 2;
  // TTL 是服务器选择的以秒为单位的租约time-to-live
  int64 TTL = 3;
  string error = 4;
}
```


