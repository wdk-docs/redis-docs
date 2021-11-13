---
title: "Node Redlock"
linkTitle: "Node Redlock"
weight: 2
---

[![Continuous Integration](https://github.com/mike-marcacci/node-redlock/workflows/Continuous%20Integration/badge.svg)](https://github.com/mike-marcacci/node-redlock/actions/workflows/ci.yml)
[![Current Version](https://badgen.net/npm/v/redlock)](https://npm.im/redlock)
[![Supported Node.js Versions](https://badgen.net/npm/node/redlock)](https://npm.im/redlock)

这是分布式 redis 锁的[redlock](http://redis.io/topics/distlock)算法的 node.js 实现。
它在单 redis 和多 redis 环境中都提供了强大的保证，并通过使用多个独立的 redis 实例或集群提供容错性。

## 高可用性的建议

- 至少使用 3 个独立的服务器或集群
- 大多数安装使用奇数个独立 redis **_server_**
- 使用奇数个独立 redis **_clusters_** 进行大规模安装
- 如果可能的话，将 redis 节点分布到不同的物理机器上

## 使用集群/哨兵

**_请确保使用内置集群支持的客户端，如[ioredis](https://github.com/luin/ioredis)._**

通过传递一个预先配置的客户端到 redlock，使用*single* redis 集群或 sentinal 配置是完全可能的。
虽然在这种模式下确实获得了高可用性并极大地提高了吞吐量，但故障模式略有不同，从理论上讲，一个锁可能被获取两次:

假设您正在使用最终一致的 redis 复制，并且您获得了资源的锁。
在获得你的锁后，redis 的碎片主机立即崩溃。
Redis 做了它的事情，并把它的故障转移到还没有同步你的锁的 slave 上。
如果另一个进程试图获取相同资源的锁，它将成功!

这就是为什么 redlock 允许你指定多个独立的节点/集群:
通过要求它们之间达成一致，我们可以安全地取出或故障转移少数节点，而不会使活动锁失效。

要了解更多的算法，请查看[redis dislock 页面](http://redis.io/topics/distlock).

## 我怎么检查东西是否上锁了?

红锁的目的是在一段时间内提供资源的独占性保证，而不是用来报告资源的所有权状态。
例如，如果您在一个网络分区的较小一端，您将无法获得锁，但您不知道另一端是否存在锁;你只知道你不能保证你的独家经营权。
重试行为使情况更加复杂，在获取多个资源上的锁时更是如此。

也就是说，对于许多任务来说，尝试使用 `retryCount=0` 锁定就足够了， 并将失败视为资源被"locked"或(更准确地说)"unavailable"。

Note that with there will be unlimited retries until the lock is aquired.
注意，使用' retryCount=-1 '将有无限的重试，直到获得锁。

## 安装

```bash
npm install --save redlock
```

## 配置

Redlock 被设计成使用[ioredis](https://github.com/luin/ioredis) 来保持它的客户端连接和处理集群协议。

一个 redlock 对象实例化一个至少一个 redis 客户端和一个可选的' options '对象的数组。
Redlock 对象的属性不应该在第一次使用后更改，因为这样做可能会对活锁产生意想不到的后果。

```ts
import Client from "ioredis";
import Redlock from "./redlock";

const redisA = new Client({ host: "a.redis.example.com" });
const redisB = new Client({ host: "b.redis.example.com" });
const redisC = new Client({ host: "c.redis.example.com" });

const redlock = new Redlock(
  // You should have one client for each independent redis node
  // or cluster.
  [redisA, redisB, redisC],
  {
    // The expected clock drift; for more details see:
    // http://redis.io/topics/distlock
    driftFactor: 0.01, // multiplied by lock ttl to determine drift time

    // The max number of times Redlock will attempt to lock a resource
    // before erroring.
    retryCount: 10,

    // the time in ms between attempts
    retryDelay: 200, // time in ms

    // the max time in ms randomly added to retries
    // to improve performance under high contention
    // see https://www.awsarchitectureblog.com/2015/03/backoff.html
    retryJitter: 200, // time in ms

    // The minimum remaining time on a lock before an extension is automatically
    // attempted with the `using` API.
    automaticExtensionThreshold: 500, // time in ms
  }
);
```

## 错误处理

因为 redlock 是为高可用性而设计的，所以它并不关心是否有少数 redis 实例/集群在操作中失败。

但是，监视和记录这种情况是很有帮助的。
当 Redlock 遇到错误时，它会触发一个"error"事件，即使该错误在正常操作中被忽略。

```ts
redlock.on("error", (error) => {
  // Ignore cases where a resource is explicitly marked as locked on a client.
  if (error instanceof ResourceLockedError) {
    return;
  }

  // Log all other errors.
  console.error(error);
});
```

此外，`Lock`和`ExecutionError`类的`attempt`属性上提供了`per-attempt`和`per-client`统计信息(包括错误)。

## 使用

`using`方法在自动扩展锁的上下文中包装并执行例程，返回例程值的承诺。
在自动扩展失败的情况下，将更新一个 AbortSignal，以表明该例程的终止是正确的，并将遇到的错误传递下去。

```ts
await redlock.using([senderId, recipientId], 5000, async (signal) => {
  // Do something...
  await something();

  // Make sure any necessary lock extension has not failed.
  if (signal.aborted) {
    throw signal.error;
  }

  // Do something else...
  await somethingElse();
});
```

或者，可以直接获取和释放锁:

```ts
// Acquire a lock.
let lock = await redlock.acquire(["a"], 5000);

// Do something...
await something();

// Extend the lock.
lock = await lock.extend(5000);

// Do something else...
await somethingElse();

// Release the lock.
await lock.release();
```

## API

请查看(非常简洁的)源代码或 TypeScript 定义，了解 API 的详细分解。
