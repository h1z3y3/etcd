# Lock 方法

Lock service 的 `Lock` 方法。

```java
// 在给定命令锁上获得分布式共享锁。
// 成功时，将返回一个唯一 key，在调用者持有锁期间会一直存在。
// 这个 key 可以和事务一起工作，以安全的确保对 etcd 的更新仅仅发生在持有锁时。
// 锁被持有直到在 key 上调用解锁或者和所有者关联的租约过期。
rpc Lock(LockRequest) returns (LockResponse) {}
```

## 消息定义

请求的消息体是 `LockRequest`：

```java
message LockRequest {
  // name 是要获取的分布式共享锁的标识
  bytes name = 1;
  // lease 是将要附加到锁所有权的租约的 ID。如果租约过期或者撤销时正持有锁，则锁将自动释放。
  // 使用相同的租约调用锁将视为单次获取；使用同样租约的第二次锁定将是空操作。
  int64 lease = 2;
}
```

应答的信息体是 `LockResponse`：

```java
message LockResponse {
  etcdserverpb.ResponseHeader header = 1;
  // key 是在 Lock 调用者拥有锁期间存在于 etcd 上的 key。
  // 用户不可以修改这个 key，否者锁将不能正常工作
  bytes key = 2;
}
```



