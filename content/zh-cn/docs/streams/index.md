# 流（Streams）

Redis 流介绍

Redis 流是一种类似于仅追加日志的数据结构。
您可以使用流实时记录并同时联合事件。
Redis 流用例的例子包括:

- 事件来源(例如，跟踪用户操作、点击等)
- 传感器监测(例如，现场设备的读数)
- 通知(例如，在单独的流中存储每个用户的通知记录)

Redis 为每个流条目生成一个唯一的 ID。
您可以使用这些 id 稍后检索它们相关联的条目，或者读取和处理流中的所有后续条目。

Redis 流支持多种修剪策略(防止流无界增长)和多种消费策略(参见 [`XREAD`](https://redis.io/commands/xread)、[`XREADGROUP`](https://redis.io/commands/xreadgroup) 和 [`XRANGE`](https://redis.io/commands/xrange))。

## 例子

向一个流中添加几个温度读数

```sh
> XADD temperatures:us-ny:10007 * temp_f 87.2 pressure 29.69 humidity 46
"1658354918398-0"
> XADD temperatures:us-ny:10007 * temp_f 83.1 pressure 29.21 humidity 46.5
"1658354934941-0"
> XADD temperatures:us-ny:10007 * temp_f 81.9 pressure 28.37 humidity 43.7
"1658354957524-0"
```

读取从 ID `1658354934941-0`开始的前两个流项:

```sh
> XRANGE temperatures:us-ny:10007 1658354934941-0 + COUNT 2
1) 1) "1658354934941-0"
   2) 1) "temp_f"
      2) "83.1"
      3) "pressure"
      4) "29.21"
      5) "humidity"
      6) "46.5"
2) 1) "1658354957524-0"
   2) 1) "temp_f"
      2) "81.9"
      3) "pressure"
      4) "28.37"
      5) "humidity"
      6) "43.7"
```

读取最多 100 个新的流条目，从流的末尾开始，如果没有条目被写入，阻塞最多 300 毫秒:

```sh
> XREAD COUNT 100 BLOCK 300 STREAMS temperatures:us-ny:10007 $
(nil)
```

## 基本命令

- [XADD](https://redis.io/commands/xadd) 向流中添加新条目。
- [XREAD](https://redis.io/commands/xread) 读取一个或多个条目，从给定位置开始，按时间向前移动。
- [XRANGE](https://redis.io/commands/xrange) 返回两个提供的条目 id 之间的条目范围。
- [XLEN](https://redis.io/commands/xlen) 返回流的长度。

请参阅流[命令的完整列表](https://redis.io/commands/?group=stream)。

## 性能

向流中添加一个条目是 O(1)。
访问任何单个条目都是 O(n)，其中 n 是 ID 的长度。
由于流 id 通常很短且长度固定，这有效地减少了查找时间。
有关原因的详细信息，请注意流是作为[基树](https://en.wikipedia.org/wiki/Radix_tree)实现的。

简单地说，Redis 流提供了高效的插入和读取。详细信息请参见每个命令的时间复杂度。

## 了解更多

- [Redis 流教程](https://redis.io/docs/data-types/streams-tutorial)用许多例子解释了 Redis 流。
- [Redis 流解释](https://www.youtube.com/watch?v=Z8qcpXyMAiA)是一个有趣的介绍在 Redis 流。
- [Redis 大学的 RU202](https://university.redis.com/courses/ru202/?_ga=2.259010314.1682093958.1669266159-1547156487.1656582125&_gl=1*11ladb4*_ga*MTU0NzE1NjQ4Ny4xNjU2NTgyMTI1*_ga_8BKGRQKRPV*MTY2OTI2NjE1OS4zLjEuMTY2OTI2NjI3OC4wLjAuMA..) 是一门免费的在线课程，专门针对 Redis 流。

## 反馈

如果您在此页面上发现了问题，或者有改进建议，请提交合并或在存储库中打开问题的请求。
