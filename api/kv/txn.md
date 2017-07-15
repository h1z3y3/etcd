# Txn 方法

Txn 方法在单个事务中处理多个请求。

txn 请求增加键值存储的修订版本并为每个完成的请求生成带有相同修订版本的事件。

不容许在一个 txn 中多次修改同一个 key。

```java
rpc Txn(TxnRequest) returns (TxnResponse) {}
```

## 背景

以下内容翻译来自 proto文件中 TxnRequest 的注释，解释了Txn请求的工作方式.

> 来自 google paxosdb 论文:
>
> 我们的实现围绕强大的我们称为 `MultiOp` 的原生(primitive)。除了游历外的所有其他数据库操作被实现为对 MultiOp 的单一调用。MultiOp 被原子执行并由三个部分组成：
>
> 1. 被称为 `guard` 的测试列表。在 guard 中每个测试检查数据库中的单个项。它可能检查某个值的存在或者缺失，或者和给定的值比较。在 guard 中两个不同的测试可能应用于数据库中相同或者不同的项。guard 中的所有测试被应用然后 MultiOp 返回结果。如果所有测试是 true，MultiOp 执行 t 操作 (见下面的第二项), 否则它执行 f 操作 (见下面的第三项).
> 2. 被称为 `t` 操作的数据库操作列表. 列表中的每个操作是插入，删除，或者查找操作，并应用到单个数据库项。列表中的两个不同操作可能应用到数据库中相同或者不同的项。如果 guard 评价为true 这些操作将被执行
> 3. 被成为 f 操作的数据库操作列表. 类似 t 操作, 但是是在 guard 评价为 false 时执行。

## 消息体

请求的消息体是 `TxnRequest`：

```java
message TxnRequest {
  // compare 是断言列表，体现为条件的联合。
  // 如果比较成功，那么成功请求将被按顺序处理，而应答将按顺序包含他们对应的应答。
  // 如果比较失败，那么失败请求将被按顺序处理，而应答将按顺序包含他们对应的应答。
  repeated Compare compare = 1;

  // 成功请求列表，当比较评估为 true 时将被应用。
  repeated RequestOp success = 2;

  // 失败请求列表，当比较评估为 false 时将被应用。
  repeated RequestOp failure = 3;
}
```

应答的消息体是 `TxnResponse`：

```java
message TxnResponse {
  ResponseHeader header = 1;

  // 如果比较评估为true则succeeded被设置为true，否则是false
  bool succeeded = 2;

  // 应答列表，如果 succeeded 是 true 则对应成功请求，如果 succeeded 是 false 则对应失败请求
  repeated ResponseOp responses = 3;
}
```

`Compare` 消息体：

```java
message Compare {
  enum CompareResult {
    EQUAL = 0;
    GREATER = 1;
    LESS = 2;
    NOT_EQUAL = 3;
  }
  enum CompareTarget {
    VERSION = 0;
    CREATE = 1;
    MOD = 2;
    VALUE= 3;
  }

  // result 是这个比较的逻辑比较操作
  CompareResult result = 1;

  // target 是比较要检查的键值字段
  CompareTarget target = 2;

  // key 是用于比较操作的主题key
  bytes key = 3;

  oneof target_union {
    // version 是给定 key 的版本
    int64 version = 4;

    // create_revision 是给定 key 的创建修订版本
    int64 create_revision = 5;

    // mod_revision 是给定 key 的最后修改修订版本
    int64 mod_revision = 6;

    // value 是给定 key 的值，以 bytes 的形式
    bytes value = 7;
  }
}
```

`RequestOp` 消息体：

```java
message RequestOp {
  // request 是可以被事务接受的请求类型的联合
  oneof request {
    RangeRequest request_range = 1;
    PutRequest request_put = 2;
    DeleteRangeRequest request_delete_range = 3;
  }
}

```

`ResponseOp` 消息体：

```java
message ResponseOp {
  // response 是事务返回的应答类型的联合
  oneof response {
    RangeResponse response_range = 1;
    PutResponse response_put = 2;
    DeleteRangeResponse response_delete_range = 3;
  }
}
```