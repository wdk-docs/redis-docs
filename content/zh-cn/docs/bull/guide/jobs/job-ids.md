---
title: "Job Ids"
linkTitle: ""
weight: 3
---

BullMQ 中的所有作业都需要有唯一的作业 id。
这些 id 用于存储构造一个键，数据存储在 Redis 中，并作为一个指针，因为它正在移动，它可以在其生命周期的不同状态。

默认情况下，作业 id 会自动生成为一个递增的计数器，但是也可以指定一个自定义 id。

能够指定自定义 id 的主要原因是在希望避免**重复作业**的情况下。
由于 id 必须是**唯一**的，如果你添加一个有 id 的作业，那么这个作业将被忽略，根本不会添加到队列中。

{% hint style="danger" %}
从队列中删除的作业，无论是手动删除还是使用 removeOnComplete/Failed 等设置时，都不会被视为重复的作业，这意味着只要之前的作业已经从队列中删除，就可以多次添加相同的作业 id。
{% endhint %}

要指定一个自定义作业 id，只需在将作业添加到队列时使用 jobId 选项:

```typescript
await myQueue.add(
  "wall",
  { color: "pink" },
  {
    jobId: customJobId,
  }
);
```
