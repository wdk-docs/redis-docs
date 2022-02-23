---
title: "Getters"
linkTitle: ""
weight: 10
---

当作业被添加到队列中时，它们在生命周期中将处于不同的状态。
BullMQ 提供了从不同状态检索信息和作业的方法。

![](../../.gitbook/assets/image.png)

## Job Counts

It is often necessary to know how many jobs are in a given status:

```typescript
import { Queue } from "bullmq";

const myQueue = new Queue("Paint");

const counts = await myQueue.getJobCounts("wait", "completed", "failed");

// Returns an object like this { wait: number, completed: number, failed: number }
```

The available status are: _completed, failed, delayed, active, wait, paused_ and _repeat._

## Get Jobs

It is also possible to retrieve the jobs with pagination style semantics. For example:

```typescript
const completed = await myQueue.getJobs(["completed"], 0, 100, true);

// returns the oldest 100 jobs
```
