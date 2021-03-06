---
title: "体系结构"
linkTitle: ""
weight: 13
description: 在这个页面中，我们给出了一个架构概述，关于如何在Redis上实现BullMQ。
---

为了充分利用 Bull 队列的潜力，理解作业的生命周期是很重要的。
From the moment a producer calls the `add` method on a queue instance, a job enters a lifecycle where it will be in different states, until its completion or failure \(although technically a failed job could be retried and get a new lifecycle\).

![Lifecycle of a job](../.gitbook/assets/image%20%281%29%20%281%29%20%281%29%20%281%29.png)

When a job is added to a queue it can be in one of two states, it can either be in the “wait” status, which is, in fact, a waiting list, where all jobs must enter before they can be processed, or it can be in a “delayed” status: a delayed status implies that the job is waiting for some timeout or to be promoted for being processed, however, a delayed job will not be processed directly, instead it will be placed at the beginning of the waiting list and processed as soon as a worker is idle.

The next state for a job Is the “active” state. The active state is represented by a set, and are jobs that are currently being processed, i.e. they are running in the `process` function explained in the previous chapter. A job can be in the active state for an unlimited amount of time until the process is completed or an exception is thrown so that the job will end in either the “completed” or the “failed” status.
