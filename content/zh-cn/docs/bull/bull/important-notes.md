---
title: "重要的笔记"
linkTitle: ""
weight: 4
---

队列的目标是“至少一次”工作策略。
这意味着在某些情况下，一个作业可以被处理多次。
这种情况通常发生在 worker 在整个处理过程中未能为给定的作业保持锁时。

当一个工人正在处理一个作业时，它将保持这个作业“锁定”，这样其他工人就不能处理它了。

理解锁是如何工作的很重要，这可以防止您的作业-失去它们的锁-，并因此而被重新启动。
锁定是通过在内部为`lockRenewTime`间隔(通常是`lockDuration`的一半)创建一个`lockDuration`的锁来实现的。
如果在更新锁之前`lockDuration`已经过期，作业将被认为是停滞的，并自动重新启动;它将被**双重处理**。
这可能发生在以下情况:

1. 运行作业处理器的 Node 进程意外终止。
2. 您的作业处理器过于 cpu 密集型，导致 Node 事件循环陷入停顿，因此，Bull 无法更新作业锁 (请参阅[#488](https://github.com/OptimalBits/bull/issues/488)了解我们如何更好地检测到这一点).
   可以通过将作业处理器分解成更小的部分来解决这个问题，这样就不会有单个部分阻塞 Node 事件循环。
   或者，您可以为`lockDuration`设置传递一个较大的值(这样做的代价是需要更长的时间来识别真正停止的工作)。

因此，您应该始终侦听`stalled`事件，并将其记录到错误监视系统，因为这意味着您的作业可能会得到双重处理。

为了保证有问题的作业不会无限期重启(例如，如果作业处理器总是使其 Node 进程崩溃)，作业将从停止状态中恢复，最大恢复次数为`maxStalledCount`(默认值为`1`)。