---
title: "停滞不前"
linkTitle: ""
weight: 9
---

{% hint style="info" %}
只有在 Queue 中配置了至少一个[`QueueScheduler`](../ QueueScheduler .md)实例时，停止的作业检查才会起作用。
{% endhint %}

当一个作业处于活动状态时，也就是说，它正在被一个 worker 处理，它需要不断地更新队列来通知这个 worker 仍在工作。
这个机制可以防止一个 worker 在崩溃或进入一个无限循环时永远保持一个工作处于活动状态。

当一个 worker 无法通知队列它仍在处理一个给定的作业时，该作业将被移回等待列表或失败集。
然后，我们说作业已经停止，队列将发出`stopped`事件。

{% hint style="info" %}
没有'stalled'状态，只有当作业被自动从活动状态转移到等待状态时触发的'stalled'事件。
{% endhint %}

为了避免停滞的任务，确保你的 worker 不会让 Node.js 事件循环太忙，默认的最大停滞检查持续时间是 30 秒，所以只要你不执行超过这个值的 CPU 操作，你就不会得到停滞的任务。

另一种减少停滞作业机会的方法是使用所谓的“沙盒”处理器。在这种情况下，worker 将生成新的独立的 Node.js 进程，独立于主进程运行。

{% code title="main.ts" %}

```typescript
import { Worker } from "bullmq";

const worker = new Worker("Paint", painter);
```

{% endcode %}

{% code title="painter.ts" %}

```typescript
export default = (job) => {
    // Paint something
}
```

{% endcode %}
