---
title: "Pipelining"
linkTitle: ""
weight: 17
---

## Pipelining

If you want to send a batch of commands (e.g. > 5), you can use pipelining to queue
the commands in memory and then send them to Redis all at once. This way the performance improves by 50%~300% (See [benchmark section](#benchmarks)).

`redis.pipeline()` creates a `Pipeline` instance. You can call any Redis
commands on it just like the `Redis` instance. The commands are queued in memory
and flushed to Redis by calling the `exec` method:

```javascript
const pipeline = redis.pipeline();
pipeline.set("foo", "bar");
pipeline.del("cc");
pipeline.exec((err, results) => {
  // `err` is always null, and `results` is an array of responses
  // corresponding to the sequence of queued commands.
  // Each response follows the format `[err, result]`.
});

// You can even chain the commands:
redis
  .pipeline()
  .set("foo", "bar")
  .del("cc")
  .exec((err, results) => {});

// `exec` also returns a Promise:
const promise = redis.pipeline().set("foo", "bar").get("foo").exec();
promise.then((result) => {
  // result === [[null, 'OK'], [null, 'bar']]
});
```

Each chained command can also have a callback, which will be invoked when the command
gets a reply:

```javascript
redis
  .pipeline()
  .set("foo", "bar")
  .get("foo", (err, result) => {
    // result === 'bar'
  })
  .exec((err, result) => {
    // result[1][1] === 'bar'
  });
```

In addition to adding commands to the `pipeline` queue individually, you can also pass an array of commands and arguments to the constructor:

```javascript
redis
  .pipeline([
    ["set", "foo", "bar"],
    ["get", "foo"],
  ])
  .exec(() => {
    /* ... */
  });
```

`#length` property shows how many commands in the pipeline:

```javascript
const length = redis.pipeline().set("foo", "bar").get("foo").length;
// length === 2
```

## Transaction

Most of the time, the transaction commands `multi` & `exec` are used together with pipeline.
Therefore, when `multi` is called, a `Pipeline` instance is created automatically by default,
so you can use `multi` just like `pipeline`:

```javascript
redis
  .multi()
  .set("foo", "bar")
  .get("foo")
  .exec((err, results) => {
    // results === [[null, 'OK'], [null, 'bar']]
  });
```

If there's a syntax error in the transaction's command chain (e.g. wrong number of arguments, wrong command name, etc),
then none of the commands would be executed, and an error is returned:

```javascript
redis
  .multi()
  .set("foo")
  .set("foo", "new value")
  .exec((err, results) => {
    // err:
    //  { [ReplyError: EXECABORT Transaction discarded because of previous errors.]
    //    name: 'ReplyError',
    //    message: 'EXECABORT Transaction discarded because of previous errors.',
    //    command: { name: 'exec', args: [] },
    //    previousErrors:
    //     [ { [ReplyError: ERR wrong number of arguments for 'set' command]
    //         name: 'ReplyError',
    //         message: 'ERR wrong number of arguments for \'set\' command',
    //         command: [Object] } ] }
  });
```

In terms of the interface, `multi` differs from `pipeline` in that when specifying a callback
to each chained command, the queueing state is passed to the callback instead of the result of the command:

```javascript
redis
  .multi()
  .set("foo", "bar", (err, result) => {
    // result === 'QUEUED'
  })
  .exec(/* ... */);
```

If you want to use transaction without pipeline, pass `{ pipeline: false }` to `multi`,
and every command will be sent to Redis immediately without waiting for an `exec` invocation:

```javascript
redis.multi({ pipeline: false });
redis.set("foo", "bar");
redis.get("foo");
redis.exec((err, result) => {
  // result === [[null, 'OK'], [null, 'bar']]
});
```

The constructor of `multi` also accepts a batch of commands:

```javascript
redis
  .multi([
    ["set", "foo", "bar"],
    ["get", "foo"],
  ])
  .exec(() => {
    /* ... */
  });
```

Inline transactions are supported by pipeline, which means you can group a subset of commands
in the pipeline into a transaction:

```javascript
redis
  .pipeline()
  .get("foo")
  .multi()
  .set("foo", "bar")
  .get("foo")
  .exec()
  .get("foo")
  .exec();
```

---

## Autopipelining

In standard mode, when you issue multiple commands, ioredis sends them to the server one by one. As described in Redis pipeline documentation, this is a suboptimal use of the network link, especially when such link is not very performant.

The TCP and network overhead negatively affects performance. Commands are stuck in the send queue until the previous ones are correctly delivered to the server. This is a problem known as Head-Of-Line blocking (HOL).

ioredis supports a feature called “auto pipelining”. It can be enabled by setting the option `enableAutoPipelining` to `true`. No other code change is necessary.

In auto pipelining mode, all commands issued during an event loop are enqueued in a pipeline automatically managed by ioredis. At the end of the iteration, the pipeline is executed and thus all commands are sent to the server at the same time.

This feature can dramatically improve throughput and avoids HOL blocking. In our benchmarks, the improvement was between 35% and 50%.

While an automatic pipeline is executing, all new commands will be enqueued in a new pipeline which will be executed as soon as the previous finishes.

When using Redis Cluster, one pipeline per node is created. Commands are assigned to pipelines according to which node serves the slot.

A pipeline will thus contain commands using different slots but that ultimately are assigned to the same node.

Note that the same slot limitation within a single command still holds, as it is a Redis limitation.

### Example of automatic pipeline enqueuing

This sample code uses ioredis with automatic pipeline enabled.

```javascript
const Redis = require("./built");
const http = require("http");

const db = new Redis({ enableAutoPipelining: true });

const server = http.createServer((request, response) => {
  const key = new URL(request.url, "https://localhost:3000/").searchParams.get(
    "key"
  );

  db.get(key, (err, value) => {
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end(value);
  });
});

server.listen(3000);
```

When Node receives requests, it schedules them to be processed in one or more iterations of the events loop.

All commands issued by requests processing during one iteration of the loop will be wrapped in a pipeline automatically created by ioredis.

In the example above, the pipeline will have the following contents:

```
GET key1
GET key2
GET key3
...
GET keyN
```

When all events in the current loop have been processed, the pipeline is executed and thus all commands are sent to the server at the same time.

While waiting for pipeline response from Redis, Node will still be able to process requests. All commands issued by request handler will be enqueued in a new automatically created pipeline. This pipeline will not be sent to the server yet.

As soon as a previous automatic pipeline has received all responses from the server, the new pipeline is immediately sent without waiting for the events loop iteration to finish.

This approach increases the utilization of the network link, reduces the TCP overhead and idle times and therefore improves throughput.

### Benchmarks

Here's some of the results of our tests for a single node.

Each iteration of the test runs 1000 random commands on the server.

|                           | Samples | Result        | Tolerance |
| ------------------------- | ------- | ------------- | --------- |
| default                   | 1000    | 174.62 op/sec | ± 0.45 %  |
| enableAutoPipelining=true | 1500    | 233.33 op/sec | ± 0.88 %  |

And here's the same test for a cluster of 3 masters and 3 replicas:

|                           | Samples | Result        | Tolerance |
| ------------------------- | ------- | ------------- | --------- |
| default                   | 1000    | 164.05 op/sec | ± 0.42 %  |
| enableAutoPipelining=true | 3000    | 235.31 op/sec | ± 0.94 %  |
