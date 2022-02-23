---
title: "可重复的"
linkTitle: ""
weight: 5
---

有一种特殊类型的 _meta_ 作业叫做 **repeatable**。
这些作业的特殊之处在于，即使您只向队列中添加一个作业，它们也会根据预定义的调度不断重复。

添加一个带有`repeat`选项集的作业实际上会立即做两件事:
创建可重复作业配置，
并为该作业的第一次运行安排一个定期延迟的作业。
第一次运行将被安排在“在一个小时”，也就是说，如果您创建一个在 4:07 每 15 分钟重复一次的作业，该作业将首先在 4:15 运行，然后是 4:30，依此类推。

可重复作业配置不是一个作业，所以它不会显示在`getJobs()`这样的方法中。
要管理可重复作业的配置，请使用[`getRepeatableJobs()`](https://github.com/taskforcesh/bullmq/blob/master/docs/gitbook/api/bullmq.queue.getrepeatablejobs.md)或类似的方法。
这也意味着重复作业**不**参与评估`jobId`的唯一性——也就是说，一个不可重复作业可以具有与可重复作业相同的`jobId`配置，而两个可重复作业配置可以具有相同的`jobId`，只要它们有不同的重复选项。

每当取下一个可重复作业进行处理时，下一个可重复作业将以适当的延迟添加到队列中。
因此，可重复作业只不过是根据某些设置添加到队列中的延迟作业。

{% hint style="info" %}
可重复作业只是延迟的作业，因此您还需要一个 `QueueScheduler` 实例来相应地调度这些作业。
{% endhint %}

有两种方法可以指定可重复对象的作业重复模式，一种是使用 cron 表达式(使用[cron-parser](https://www.npmjs.com/package/cron-parser)的"unix cron w/ optional seconds"格式)，另一种是指定重复之间的固定毫秒数。

```typescript
import { Queue, QueueScheduler } from "bullmq";

const myQueueScheduler = new QueueScheduler("Paint");
const myQueue = new Queue("Paint");

// 每天凌晨3:15重复工作一次
await myQueue.add(
  "submarine",
  { color: "yellow" },
  {
    repeat: {
      cron: "* 15 3 * * *",
    },
  }
);

// 每10秒重复一次，但不要超过100次
await myQueue.add(
  "bird",
  { color: "bird" },
  {
    repeat: {
      every: 10000,
      limit: 100,
    },
  }
);
```

关于可重复的工作，有一些重要的考虑因素:

- Bull 足够聪明，如果重复的选项是相同的，它不会添加相同的可重复的工作。
- 如果没有工人在运行，那么可重复的工作将不会在下一次工人在线时累积。
- 可以删除重复的工作使用[removeRepeatable] (https://github.com/taskforcesh/bullmq/blob/master/docs/gitbook/api/bullmq.queue.removerepeatable.md)方法或(removeRepeatableByKey) (https://github.com/taskforcesh/bullmq/blob/master/docs/gitbook/api/bullmq.queue.removerepeatablebykey.md)。

所有可重复作业都有一个可重复作业键，该键保存可重复作业本身的一些元数据。
可以在调用[getRepeatableJobs](https://github.com/taskforcesh/bullmq/blob/master/docs/gitbook/api/bullmq.queue.getrepeatablejobs.md)的队列中检索当前所有可重复的作业。:

```typescript
import { Queue } from "bullmq";

const myQueue = new Queue("Paint");

const repeatableJobs = await myQueue.getRepeatableJobs();
```

由于可重复的作业是延迟的作业，重复是通过在当前作业开始处理之前生成一个新的延迟作业来实现的。
作业需要唯一的 id，以避免重复，这意味着标准的 jobId 选项与常规作业的工作方式不同。
对于可重复的任务，jobId 用来生成唯一的 id，例如，如果你有两个具有相同名称和选项的可重复任务，你可以使用 jobId 来生成两个不同的可重复的任务:

```typescript
import { Queue, QueueScheduler } from "bullmq";

const myQueueScheduler = new QueueScheduler("Paint");
const myQueue = new Queue("Paint");

// Repeat job every 10 seconds but no more than 100 times
await myQueue.add(
  "bird",
  { color: "bird" },
  {
    repeat: {
      every: 10000,
      limit: 100,
    },
    jobId: "colibri",
  }
);

await myQueue.add(
  "bird",
  { color: "bird" },
  {
    repeat: {
      every: 10000,
      limit: 100,
    },
    jobId: "pigeon",
  }
);
```

## 缓慢的重复工作

有一种情况值得一提，即可重复频率大于处理作业所需的时间。

例如，假设你有一项工作每秒钟都在重复，但是这个工作的过程本身需要 5 秒钟。
如上所述，可重复作业只是延迟的作业，因此这意味着一旦开始处理下一个作业，就会添加下一个可重复作业。

在这个特定的例子中，worker 将拾取下一个作业，并添加下一个可重复作业，延迟 1 秒，因为这是可重复的间隔。
worker 将需要 5 秒来处理该作业，如果只有 1 个 worker 可用，那么下一个作业将需要等待整整 5 秒才能被处理。

另一方面，如果有 5 个可用的工人，那么他们很可能能够以期望的每秒一个工作的频率处理所有可重复的工作。
