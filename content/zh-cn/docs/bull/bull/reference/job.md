---
title: "Job"
linkTitle: ""
weight: 2
---

A job includes all data needed to perform its execution, as well as the progress method needed to update its progress.

The most important property for the user is `Job#data` that includes the object that was passed to [`Queue#add`](#queueadd), and that is normally used to perform the job.

### Job#progress

```ts
progress(progress?: number | object): Promise
```

Updates a job progress if called with an argument.
Return a promise resolving to the current job's progress if called without argument.

#### Arguments

```js
  progress: number; Job progress number or any serializable object representing progress or similar.
```

---

### Job#log

```ts
log(row: string): Promise
```

Adds a log row to this job specific job. Logs can be retrieved using [Queue#getJobLogs](#queuegetjoblogs).

---

### Job#getState

```ts
getState(): Promise
```

Returns a promise resolving to the current job's status (completed, failed, delayed etc.). Possible returns are: completed, failed, delayed, active, waiting, paused, stuck or null.

Please take note that the implementation of this method is not very efficient, nor is it atomic. If your queue does have a very large quantity of jobs, you may want to avoid using this method.

---

### Job#update

```ts
update(data: object): Promise
```

Updated a job data field with the give data object.

---

### Job#remove

```ts
remove(): Promise
```

Removes a job from the queue and from any lists it may be included in.

---

### Job#retry

```ts
retry(): Promise
```

Re-run a job that has failed. Returns a promise that resolves when the job is scheduled for retry.

---

### Job#discard

```ts
discard(): Promise
```

Ensure this job is never ran again even if `attemptsMade` is less than `job.attempts`.

---

### Job#promote

```ts
promote(): Promise
```

Promotes a job that is currently "delayed" to the "waiting" state and executed as soon as possible.

---

### Job#finished

```ts
finished(): Promise
```

Returns a promise that resolves or rejects when the job completes or fails.

---

### Job#moveToCompleted

```ts
moveToCompleted(returnValue: any, ignoreLock: boolean, notFetch?: boolean): Promise<string[Jobdata, JobId] | null>
```

Moves a job to the `completed` queue. Pulls a job from 'waiting' to 'active' and returns a tuple containing the next jobs data and id. If no job is in the `waiting` queue, returns null. Set `notFetch` to true to avoid prefetching the next job in the queue.

---

### Job#moveToFailed

```ts
moveToFailed(errorInfo:{ message: string; }, ignoreLock?:boolean): Promise<string[Jobdata, JobId] | null>
```

Moves a job to the `failed` queue. Pulls a job from 'waiting' to 'active' and returns a tuple containing the next jobs data and id. If no job is in the `waiting` queue, returns null.

---
