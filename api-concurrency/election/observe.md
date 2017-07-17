# Observe 方法

Observe 方法是　Leader 方法的　stream 形式：
```java
// Observe　以流的方式返回选举公告，和被选举的领导者发布的顺序一致。
rpc Observe(LeaderRequest) returns (stream LeaderResponse) {}
```

## 消息定义

消息体和Leader 方法相同，都是 `LeaderRequest`　和　`LeaderResponse`。


