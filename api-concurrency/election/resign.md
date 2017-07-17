# Resign 方法

Leader 方法用于放弃选举领导地位：

```java
// Resign　放弃选举领导地位，以便其他参选人可以在选举中获得领导地位。
rpc Resign(ResignRequest) returns (ResignResponse) {}
```

## 消息定义

请求的消息体是 `ResignRequest`：

```java
message ResignRequest {
  // leader 是要放弃的领导地位
  LeaderKey leader = 1;
}
```

应答的消息体是 `ResignResponse`：

```java
message ResignResponse {
  etcdserverpb.ResponseHeader header = 1;
}
```

