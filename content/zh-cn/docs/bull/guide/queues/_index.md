---
title: "队列"
linkTitle: ""
weight: 3
---

Queue 只不过是一个等待处理的作业列表。
作业可以是小的，比如消息，这样队列可以用作消息代理，也可以是大的长时间运行的作业。

队列由 Queue 类控制。与 BullMQ 中的所有类一样，这是一个轻量级类，有少量方法，可以让你控制队列:

```typescript
const queue = new Queue("Cars");
```

{% hint style="info" %}
See [Connections](../connections.md) for details on how to pass Redis details to use by the queue.
{% endhint %}

When you instance a Queue, BullMQ will just _upsert_ a small "meta-key", so if the queue existed before it will just pick it up and you can continue adding jobs to it.

The most important method is probably the _**add**_ method. This method allows you to add jobs to the queue in different fashions:

```typescript
await queue.add("paint", { colour: "red" });
```

The code above will add a job named _paint_ to the queue, with payload `{ color: 'red' }`. This job will now be stored in Redis in a list waiting for some worker to pick it up and process it. Workers may not be running when you add the job, however as soon as one worker is connected to the queue it will pick the job and process it.

When adding a job you can also specify an options object. This options object can dramatically change the behaviour of the added jobs. For example you can add a job that is delayed:

```typescript
await queue.add("paint", { colour: "blue" }, { delay: 5000 });
```

The job will now wait **at** **least** 5 seconds before it is processed.&#x20;

{% hint style="danger" %}
In order for delay jobs to work you need to have at least one QueueScheduler somewhere in your infrastructure. Read more [here](../queuescheduler.md).
{% endhint %}

There are many other options available such as priorities, backoff settings, lifo behaviour, remove-on-complete policies, etc. Please check the remaining of this guide for more information regarding these options.
