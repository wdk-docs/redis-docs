---
title: "快速入门"
linkTitle: ""
weight: 1
---

## 安装

```shell
$ npm install ioredis
```

## 基本用法

```javascript
const Redis = require("ioredis");
const redis = new Redis(); // uses defaults unless given configuration object

// ioredis supports all Redis commands:
redis.set("foo", "bar"); // returns promise which resolves to string, "OK"

// the format is: redis[SOME_REDIS_COMMAND_IN_LOWERCASE](ARGUMENTS_ARE_JOINED_INTO_COMMAND_STRING)
// the js: ` redis.set("mykey", "Hello") ` is equivalent to the cli: ` redis> SET mykey "Hello" `

// ioredis supports the node.js callback style
redis.get("foo", function (err, result) {
  if (err) {
    console.error(err);
  } else {
    console.log(result); // Promise resolves to "bar"
  }
});

// Or ioredis returns a promise if the last argument isn't a function
redis.get("foo").then(function (result) {
  console.log(result); // Prints "bar"
});

// Most responses are strings, or arrays of strings
redis.zadd("sortedSet", 1, "one", 2, "dos", 4, "quatro", 3, "three");
redis.zrange("sortedSet", 0, 2, "WITHSCORES").then((res) => console.log(res)); // Promise resolves to ["one", "1", "dos", "2", "three", "3"] as if the command was ` redis> ZRANGE sortedSet 0 2 WITHSCORES `

// All arguments are passed directly to the redis server:
redis.set("key", 100, "EX", 10);
```

See the `examples/` folder for more examples.

## Transparent Key Prefixing

This feature allows you to specify a string that will automatically be prepended
to all the keys in a command, which makes it easier to manage your key
namespaces.

**Warning** This feature won't apply to commands like [KEYS](http://redis.io/commands/KEYS) and [SCAN](http://redis.io/commands/scan) that take patterns rather than actual keys([#239](https://github.com/luin/ioredis/issues/239)),
and this feature also won't apply to the replies of commands even if they are key names ([#325](https://github.com/luin/ioredis/issues/325)).

```javascript
const fooRedis = new Redis({ keyPrefix: "foo:" });
fooRedis.set("bar", "baz"); // Actually sends SET foo:bar baz

fooRedis.defineCommand("echo", {
  numberOfKeys: 2,
  lua: "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}",
});

// Works well with pipelining/transaction
fooRedis
  .pipeline()
  // Sends SORT foo:list BY foo:weight_*->fieldname
  .sort("list", "BY", "weight_*->fieldname")
  // Supports custom commands
  // Sends EVALSHA xxx foo:k1 foo:k2 a1 a2
  .echo("k1", "k2", "a1", "a2")
  .exec();
```

## Transforming Arguments & Replies

Most Redis commands take one or more Strings as arguments,
and replies are sent back as a single String or an Array of Strings. However, sometimes
you may want something different. For instance, it would be more convenient if the `HGETALL`
command returns a hash (e.g. `{ key: val1, key2: v2 }`) rather than an array of key values (e.g. `[key1, val1, key2, val2]`).

ioredis has a flexible system for transforming arguments and replies. There are two types
of transformers, argument transformer and reply transformer:

```javascript
const Redis = require("ioredis");

// Here's the built-in argument transformer converting
// hmset('key', { k1: 'v1', k2: 'v2' })
// or
// hmset('key', new Map([['k1', 'v1'], ['k2', 'v2']]))
// into
// hmset('key', 'k1', 'v1', 'k2', 'v2')
Redis.Command.setArgumentTransformer("hmset", (args) => {
  if (args.length === 2) {
    if (typeof Map !== "undefined" && args[1] instanceof Map) {
      // utils is a internal module of ioredis
      return [args[0]].concat(utils.convertMapToArray(args[1]));
    }
    if (typeof args[1] === "object" && args[1] !== null) {
      return [args[0]].concat(utils.convertObjectToArray(args[1]));
    }
  }
  return args;
});

// Here's the built-in reply transformer converting the HGETALL reply
// ['k1', 'v1', 'k2', 'v2']
// into
// { k1: 'v1', 'k2': 'v2' }
Redis.Command.setReplyTransformer("hgetall", (result) => {
  if (Array.isArray(result)) {
    const obj = {};
    for (let i = 0; i < result.length; i += 2) {
      obj[result[i]] = result[i + 1];
    }
    return obj;
  }
  return result;
});
```

There are three built-in transformers, two argument transformers for `hmset` & `mset` and
a reply transformer for `hgetall`. Transformers for `hmset` and `hgetall` were mentioned
above, and the transformer for `mset` is similar to the one for `hmset`:

```javascript
redis.mset({ k1: "v1", k2: "v2" });
redis.get("k1", (err, result) => {
  // result === 'v1';
});

redis.mset(
  new Map([
    ["k3", "v3"],
    ["k4", "v4"],
  ])
);
redis.get("k3", (err, result) => {
  // result === 'v3';
});
```

Another useful example of a reply transformer is one that changes `hgetall` to return array of arrays instead of objects which avoids an unwanted conversation of hash keys to strings when dealing with binary hash keys:

```javascript
Redis.Command.setReplyTransformer("hgetall", (result) => {
  const arr = [];
  for (let i = 0; i < result.length; i += 2) {
    arr.push([result[i], result[i + 1]]);
  }
  return arr;
});
redis.hset("h1", Buffer.from([0x01]), Buffer.from([0x02]));
redis.hset("h1", Buffer.from([0x03]), Buffer.from([0x04]));
redis.hgetallBuffer("h1", (err, result) => {
  // result === [ [ <Buffer 01>, <Buffer 02> ], [ <Buffer 03>, <Buffer 04> ] ];
});
```

## 监控

Redis supports the MONITOR command,
which lets you see all commands received by the Redis server across all client connections,
including from other client libraries and other computers.

The `monitor` method returns a monitor instance.
After you send the MONITOR command, no other commands are valid on that connection. ioredis will emit a monitor event for every new monitor message that comes across.
The callback for the monitor event takes a timestamp from the Redis server and an array of command arguments.

Here is a simple example:

```javascript
redis.monitor((err, monitor) => {
  monitor.on("monitor", (time, args, source, database) => {});
});
```

Here is another example illustrating an `async` function and `monitor.disconnect()`:

```javascript
async () => {
  const monitor = await redis.monitor();
  monitor.on("monitor", console.log);
  // Any other tasks
  monitor.disconnect();
};
```

## Streamify Scanning

Redis 2.8 added the `SCAN` command to incrementally iterate through the keys in the database.
It's different from `KEYS` in that `SCAN` only returns a small number of elements each call,
so it can be used in production without the downside of blocking the server for a long time.
However, it requires recording the cursor on the client side each time
the `SCAN` command is called in order to iterate through all the keys correctly. Since it's a relatively common use case, ioredis
provides a streaming interface for the `SCAN` command to make things much easier. A readable stream can be created by calling `scanStream`:

```javascript
const redis = new Redis();
// Create a readable stream (object mode)
const stream = redis.scanStream();
stream.on("data", (resultKeys) => {
  // `resultKeys` is an array of strings representing key names.
  // Note that resultKeys may contain 0 keys, and that it will sometimes
  // contain duplicates due to SCAN's implementation in Redis.
  for (let i = 0; i < resultKeys.length; i++) {
    console.log(resultKeys[i]);
  }
});
stream.on("end", () => {
  console.log("all keys have been visited");
});
```

`scanStream` accepts an option, with which you can specify the `MATCH` pattern, the `TYPE` filter, and the `COUNT` argument:

```javascript
const stream = redis.scanStream({
  // only returns keys following the pattern of `user:*`
  match: "user:*",
  // only return objects that match a given type,
  // (requires Redis >= 6.0)
  type: "zset",
  // returns approximately 100 elements per call
  count: 100,
});
```

Just like other commands, `scanStream` has a binary version `scanBufferStream`, which returns an array of buffers. It's useful when
the key names are not utf8 strings.

There are also `hscanStream`, `zscanStream` and `sscanStream` to iterate through elements in a hash, zset and set. The interface of each is
similar to `scanStream` except the first argument is the key name:

```javascript
const stream = redis.hscanStream("myhash", {
  match: "age:??",
});
```

You can learn more from the [Redis documentation](http://redis.io/commands/scan).

**Useful Tips**
It's pretty common that doing an async task in the `data` handler. We'd like the scanning process to be paused until the async task to be finished. `Stream#pause()` and `Stream#resume()` do the trick. For example if we want to migrate data in Redis to MySQL:

```javascript
const stream = redis.scanStream();
stream.on("data", (resultKeys) => {
  // Pause the stream from scanning more keys until we've migrated the current keys.
  stream.pause();

  Promise.all(resultKeys.map(migrateKeyToMySQL)).then(() => {
    // Resume the stream here.
    stream.resume();
  });
});

stream.on("end", () => {
  console.log("done migration");
});
```

## 离线队列

When a command can't be processed by Redis (being sent before the `ready` event), by default, it's added to the offline queue and will be
executed when it can be processed. You can disable this feature by setting the `enableOfflineQueue`
option to `false`:

```javascript
const redis = new Redis({ enableOfflineQueue: false });
```
