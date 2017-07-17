# Proclaim 方法

Proclaim 方法用于更新值：

```java
// Proclaim 用新值更新领导者的旧值
rpc Proclaim(ProclaimRequest) returns (ProclaimResponse) {}
```

## 消息定义

请求的消息体是 `ProclaimRequest`：

```java
message ProclaimRequest {
  // leader 是在选举上持有的领导地位
  LeaderKey leader = 1;
  // value 是打算用于覆盖领导者当前值的更新。
  bytes value = 2;
}
```

应答的消息体是 `ProclaimResponse`：

```java
message ProclaimResponse {
  etcdserverpb.ResponseHeader header = 1;
}
```

