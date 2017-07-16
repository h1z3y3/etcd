# Campaign 方法

Campaign 方法用于参加选举以期获得领导地位：

```java
// Campaign 等待获得选举的领导地位，如果成功返回　LeaderKey　代表领导地位。
// 然后　LeaderKey 可以用来在选举时发起新的值，在依然持有领导地位时事务性的守护 API 请求，
// 还有从选举中辞职。
rpc Campaign(CampaignRequest) returns (CampaignResponse) {}
```

## 消息定义

请求的消息体是 `CampaignRequest`：

```java
message CampaignRequest {
  // name 是选举的标识符，用来参加竞选
  bytes name = 1;
  // lease is the ID of the lease attached to leadership of the election. If the
  // lease expires or is revoked before resigning leadership, then the
  // leadership is transferred to the next campaigner, if any.
  // lease 是附加到选举领导地位的租约的ID。如果租约过期或者在放弃领导地位之前取消,
  // 则领导地位转移到下一个竞选者，如果有。
  int64 lease = 2;
  // value 是竞选者赢得选举时设置的初始化公告值。
  bytes value = 3;
}
```

应答的消息体是 `CampaignResponse`：

```java
message CampaignResponse {
  etcdserverpb.ResponseHeader header = 1;
  // leader 描述用于持有选举的领导地位的资源
  LeaderKey leader = 2;
}
```

`LeaderKey`　消息体的内容：

```java
message LeaderKey {
  // name 是选举标识符，和领导地位　key　对应
  bytes name = 1;
  // key 是不透明的 key ，代表选举的领导地位。
  // 如果　key 被删除，则领导地位丢失
  bytes key = 2;
  // rev 是　key 的创建修订版本。它可以用来在事务期间测验选举的领导地位，通过测验 key 的创建修订版本匹配　rev
  int64 rev = 3;
  // lease 是选举领导者的租约 ID
  int64 lease = 4;
}
```
