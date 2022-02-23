---
title: "LIFO"
linkTitle: ""
weight: 2
description: "Last-in, First Out"
---

在某些情况下，以后进先出(后进先出)的方式处理作业是有用的。
这意味着添加到队列中的最新作业将在旧作业之前处理。

```typescript
import { Queue } from "bullmq";

const myQueue = new Queue("Paint");

// Add a job that will be processed before all others
await myQueue.add("wall", { color: "pink" }, { lifo: true });
```
