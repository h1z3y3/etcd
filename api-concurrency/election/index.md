# Election service

Election service 提供观察键值对变化的支持。

在 `v3election.proto` 中 Election service 定义如下：

```java
// Election service 以 gRPC 接口的方式暴露客户端选举机制。
service Election {
  // Campaign 等待获得选举的领导地位，如果成功返回　LeaderKey　代表领导地位。
  // 然后　LeaderKey 可以用来在选举时发起新的值，在依然持有领导地位时事务性的守护 API 请求，
  // 还有从选举中辞职。
  rpc Campaign(CampaignRequest) returns (CampaignResponse) {}

  // Proclaim 用新值更新领导者的旧值
  rpc Proclaim(ProclaimRequest) returns (ProclaimResponse) {}

  // Leader 返回当前的选举公告，如果有。
  rpc Leader(LeaderRequest) returns (LeaderResponse) {}

  // Observe　以流的方式返回选举公告，和被选举的领导者发布的顺序一致。
  rpc Observe(LeaderRequest) returns (stream LeaderResponse) {}

  // Resign　释放选举领导地位，以便其他参选人可以在选举中获得领导地位。
  rpc Resign(ResignRequest) returns (ResignResponse) {}
}
```




