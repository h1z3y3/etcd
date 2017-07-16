# Lock service

Lock service 提供分布式共享锁的支持。

在 `v3lock.proto` 中 Lock service 定义如下：

```java
// Lock service 以 gRPC 接口的方式暴露客户端锁机制。
service Lock {
  // 在给定命令锁上获得分布式共享锁。
  // 成功时，将返回一个唯一 key，在调用者持有锁期间会一直存在。
  // 这个 key 可以和事务一起工作，以安全的确保对 etcd 的更新仅仅发生在持有锁时。
  // 锁被持有直到在 key 上调用解锁或者和所有者关联的租约过期。
  rpc Lock(LockRequest) returns (LockResponse) {}

  // Unloke 使用 Lock 返回的 key 并释放对锁的持有。
  // 下一个在等待这个锁的 Lock 的调用者将被唤醒并给予锁的所有权。
  rpc Unlock(UnlockRequest) returns (UnlockResponse) {}
}
```




