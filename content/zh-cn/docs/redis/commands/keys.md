# KEYS

## Syntax

```sh
KEYS pattern
```

|             |                                                                                     |
| ----------- | ----------------------------------------------------------------------------------- |
| 自起可用:   | 1.0.0                                                                               |
| 时间复杂性: | O(N)，其中 N 是数据库中的密钥数, 假设数据库中的密钥名称和给定的模式具有有限的长度。 |
| ACL 类别:   | @keyspace, @read, @slow, @dangerous                                                 |

返回所有与模式匹配的键。

虽然此操作的时间复杂度为 O（N），但常数时间相当低。例如，在入门级笔记本电脑上运行的 Redis 可以在 40 毫秒内扫描 100 万个密钥数据库。

警告：将 KEYS 视为只应在生产环境中极其小心地使用的命令。当对大型数据库执行时，它可能会破坏性能。此命令用于调试和特殊操作，例如更改键空间布局。不要在常规应用程序代码中使用 KEYS。如果您正在寻找在密钥空间的子集中查找密钥的方法，请考虑使用 SCAN 或集合。

支持的 glob 样式模式：

- `h?llo` matches `hello`, `hallo` and `hxllo`
- `h*llo` matches `hllo` and `heeeello`
- `h[ae]llo` matches `hello` and `hallo`, but not _hillo_
- `h[^e]llo` matches `hallo`, `hbllo`, ... but not _hello_
- `h[a-b]llo` matches `hallo` and `hbllo`

如果要逐字匹配特殊字符，请使用`\`对其进行转义。

当使用 Redis Cluster 时，搜索会针对暗示单个插槽的模式进行优化。
如果一个模式只能匹配一个槽的键，那么 Redis 在搜索与该模式匹配的键时，只会迭代该槽中的键，而不是整个数据库。
例如，使用模式`{a}h*llo`，Redis 只会尝试将其与插槽 15495 中的密钥进行匹配，这是散列标签`{a}`所暗示的。
要将模式与哈希标记一起使用，请参阅集群规范中的哈希标记以获取更多信息。

## 示例

```sh
redis> MSET firstname Jack lastname Stuntman age 35
Failed to fetch
redis> KEYS *name*
Failed to fetch
redis> KEYS a??
Failed to fetch
redis> KEYS *
Failed to fetch
redis>
```

## RESP2/RESP3 回复

数组回复：匹配模式的键列表。
