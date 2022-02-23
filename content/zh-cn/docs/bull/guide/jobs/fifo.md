---
title: "FIFO"
linkTitle: ""
weight: 1
description: "First-In, First-Out"
---

我们要描述的第一种类型的工作是 FIFO(先进先出)类型。
这是将作业添加到队列时的标准类型。
作业按照它们插入队列的顺序进行处理。
此订单是单独保存在处理器的数量,不过如果你有超过一个工人或并发性大于 1,尽管工人们将开始工作,他们可能在一个稍微不同的完成订单,因为有些工作可能比其他人需要更多的时间来完成。

```typescript
import { Queue } from "bullmq";

const myQueue = new Queue("Paint");

// Add a job that will be processed after all others
await myQueue.add("wall", { color: "pink" });
```

在将作业添加到队列时，可以使用几个选项。
例如，当作业完成或失败时，你可以指定需要保留多少作业:

```typescript
await myQueue.add("wall", { color: "pink" }, { removeOnComplete: true, removeOnFail: 1000 });
```

在上面的例子中，所有完成的作业将被自动删除，最后 1000 个失败的作业将保留在队列中。

## 默认的工作选择

通常，您会希望为添加到 Queue 中的所有作业提供相同的作业选项。
在这种情况下，你可以在实例化 Queue 类时使用`defaultJobOptions`选项:

```typescript
const queue = new Queue('Paint', { defaultJobOptions: {
  removeOnComplete: true, removeOnFail: 1000
});
```
