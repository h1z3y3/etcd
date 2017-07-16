# Unloke 方法

Lock service 的 `Unloke` 方法。

```java
// Unloke 使用 Lock 返回的 key 并释放对锁的持有。
// 下一个在等待这个锁的 Lock 的调用者将被唤醒并给予锁的所有权。
rpc Unlock(UnlockRequest) returns (UnlockResponse) {}
```

## 消息定义

请求的消息体是 `UnlockRequest`：

```java
message UnlockRequest {
  // key 是通过 Lock 方法得到的锁所有权 key
  bytes key = 1;
}
```

应答的消息体是  `UnlockResponse`：

```java
message UnlockResponse {
  etcdserverpb.ResponseHeader header = 1;
}
```

