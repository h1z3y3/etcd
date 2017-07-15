# Range方法

Range方法从键值存储中获取范围内的key.

```java
rpc Range(RangeRequest) returns (RangeResponse) {}
```

> 注意: 没有操作单个key的方法，即使是存取单个key，也是需要使用 `Range` 方法的。

## 消息体

请求的消息体是 RangeRequest：

```java
message RangeRequest {
  enum SortOrder {
	NONE = 0; // 默认, 不排序
	ASCEND = 1; // 正序，低的值在前
	DESCEND = 2; // 倒序，高的值在前
  }
  enum SortTarget {
	KEY = 0;
	VERSION = 1;
	CREATE = 2;
	MOD = 3;
	VALUE = 4;
  }

  // key是 range 的第一个 key。如果 range_end 没有指定，请求仅查找这个key
  bytes key = 1;

  // range_end 是请求范围的上限 [key, range_end)
  // 如果 range_end 是 '\0'，范围是大于等于 key 的所有 key。
  // 如果 range_end 是 key 加一(例如, "aa"+1 == "ab", "a\xff"+1 == "b")， 那么 range 请求获取以 key 为前缀的所有 key
  // 如果 key 和 range_end 都是'\0'，则 range 查询返回所有 key
  bytes range_end = 2;

  // 请求返回的 key 的数量限制。如果 limit 设置为0，则视为没有限制
  int64 limit = 3;

  // 修订版本是用于 range 的键值对存储的时间点。
  // 如果修订版本小于或等于零，range 是用在最新的键值对存储上。
  // 如果指定修订版本已经被压缩，返回 ErrCompacted 作为应答
  int64 revision = 4;

  // 指定返回结果的排序顺序
  SortOrder sort_order = 5;

  // 用于排序的键值字段
  SortTarget sort_target = 6;

  // 设置 range 请求使用串行化成员本地读(serializable member-local read)。
  // range 请求默认是线性化的;线性化请求相比串行化请求有更高的延迟和低吞吐量，但是反映集群当前的一致性。
  // 为了更好的性能，以可能脏读为交换，串行化范围请求在本地处理，无需和集群中的其他节点达到一致。
  bool serializable = 7;

  // keys_only 被设置时仅返回 key 而不需要 value
  bool keys_only = 8;

  // count_only 被设置时仅仅返回范围内 key 的数量
  bool count_only = 9;

  // min_mod_revision 是返回 key 的 mod revision 的下限；更低 mod revision 的所有 key 都将被过滤掉
  int64 min_mod_revision = 10;

  // max_mod_revision 是返回 key 的 mod revision 的上限；更高 mod revision 的所有 key 都将被过滤掉
  int64 max_mod_revision = 11;

    // min_create_revision 是返回 key 的 create revision 的下限；更低 create revision 的所有 key 都将被过滤掉
  int64 min_create_revision = 12;

  // max_create_revision 是返回 key 的 create revision 的上限；更高 create revision 的所有 key 都将被过滤掉
  int64 max_create_revision = 13;
}
```

应答的消息体是 RangeResponse：

```java
message RangeResponse {
  ResponseHeader header = 1;

  // kvs 是匹配 range 请求的键值对列表
  // 当 count 时是空的
  repeated mvccpb.KeyValue kvs = 2;

  // more 代表在被请求的范围内是否还有更多的 key
  bool more = 3;

  // count 被设置为在范围内的 key 的数量
  int64 count = 4;
}
```

`mvccpb.KeyValue` 来自 `kv.proto`，消息体定义为：

```java
message KeyValue {
  // key 是 bytes 格式的 key。不容许 key 为空。
  bytes key = 1;

  // create_revision 是这个 key 最后一次创建的修订版本
  int64 create_revision = 2;

  // mod_revision 是这个 key 最后一次修改的修订版本
  int64 mod_revision = 3;

  // version 是 key 的版本。删除会重置版本为0,而任何 key 的修改会增加它的版本。
  int64 version = 4;

  // value 是 key 持有的值，bytes 格式。
  bytes value = 5;

  // lease 是附加给 key 的租约 id。
  // 当附加的租约过期时，key 将被删除。
  // 如果 lease 为0,则没有租约附加到 key。
  int64 lease = 6;
}
```

