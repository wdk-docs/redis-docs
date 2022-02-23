---
title: "Events"
linkTitle: ""
weight: 1
---

A queue emits also some useful events:

```js
.on('error', function (error) {
  // An error occured.
})

.on('waiting', function (jobId) {
  // A Job is waiting to be processed as soon as a worker is idling.
});

.on('active', function (job, jobPromise) {
  // A job has started. You can use `jobPromise.cancel()`` to abort it.
})

.on('stalled', function (job) {
  // A job has been marked as stalled. This is useful for debugging job
  // workers that crash or pause the event loop.
})

.on('lock-extension-failed', function (job, err) {
  // A job failed to extend lock. This will be useful to debug redis
  // connection issues and jobs getting restarted because workers
  // are not able to extend locks.
});

.on('progress', function (job, progress) {
  // A job's progress was updated!
})

.on('completed', function (job, result) {
  // A job successfully completed with a `result`.
})

.on('failed', function (job, err) {
  // A job failed with reason `err`!
})

.on('paused', function () {
  // The queue has been paused.
})

.on('resumed', function (job) {
  // The queue has been resumed.
})

.on('cleaned', function (jobs, type) {
  // Old jobs have been cleaned from the queue. `jobs` is an array of cleaned
  // jobs, and `type` is the type of jobs cleaned.
});

.on('drained', function () {
  // Emitted every time the queue has processed all the waiting jobs (even if there can be some delayed jobs not yet processed)
});

.on('removed', function (job) {
  // A job successfully removed.
});

```

### Global events

Events are local by default — in other words, they only fire on the listeners that are registered on the given worker. If you need to listen to events globally, for example from other servers across redis, just prefix the event with `'global:'`:

```js
// Will listen locally, just to this queue...
queue.on('completed', listener):

// Will listen globally, to instances of this queue...
queue.on('global:completed', listener);
```

When working with global events whose local counterparts pass a `Job` instance to the event listener callback, notice that global events pass the **job's ID** instead.

If you need to access the `Job` instance in a global listener, use [Queue#getJob](#queuegetjob) to retrieve it. However, remember that if `removeOnComplete` is enabled when adding the job, the job will no longer be available after completion. Should you need to both access the job and remove it after completion, you can use [Job#remove](#jobremove) to remove it in the listener.

```js
// Local events pass the job instance...
queue.on("progress", function (job, progress) {
  console.log(`Job ${job.id} is ${progress * 100}% ready!`);
});

queue.on("completed", function (job, result) {
  console.log(`Job ${job.id} completed! Result: ${result}`);
  job.remove();
});

// ...whereas global events only pass the job ID:
queue.on("global:progress", function (jobId, progress) {
  console.log(`Job ${jobId} is ${progress * 100}% ready!`);
});

queue.on("global:completed", function (jobId, result) {
  console.log(`Job ${jobId} completed! Result: ${result}`);
  queue.getJob(jobId).then(function (job) {
    job.remove();
  });
});
```
