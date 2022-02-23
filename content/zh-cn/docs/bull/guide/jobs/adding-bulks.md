---
title: "批量添加"
linkTitle: ""
weight: 7
---

有时需要以原子的方式添加大量的作业。
例如，可能有这样的要求:所有作业都必须放置在队列中，或者不放置任何作业。
此外，增加大量的工作可以更快，因为它减少了到 Redis 的往返:

```typescript
import { Queue } from "bullmq";

const queue = new Queue("paint");

const jobs = await queue.addBulk([
  { name, data: { paint: "car" } },
  { name, data: { paint: "house" } },
  { name, data: { paint: "boat" } },
]);
```

此调用只能成功或失败，并且将添加所有或不添加作业。
