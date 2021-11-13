---
title: "流和二进制"
linkTitle: ""
weight: 10
---

## 流

Redis v5 introduces a new data type called streams. It doubles as a communication channel for building streaming architectures and as a log-like data structure for persisting data. With ioredis, the usage can be pretty straightforward. Say we have a producer publishes messages to a stream with `redis.xadd("mystream", "*", "randomValue", Math.random())` (You may find the [official documentation of Streams](https://redis.io/topics/streams-intro) as a starter to understand the parameters used), to consume the messages, we'll have a consumer with the following code:

```javascript
const Redis = require("ioredis");
const redis = new Redis();

const processMessage = (message) => {
  console.log("Id: %s. Data: %O", message[0], message[1]);
};

async function listenForMessage(lastId = "$") {
  // `results` is an array, each element of which corresponds to a key.
  // Because we only listen to one key (mystream) here, `results` only contains
  // a single element. See more: https://redis.io/commands/xread#return-value
  const results = await redis.xread("block", 0, "STREAMS", "mystream", lastId);
  const [key, messages] = results[0]; // `key` equals to "mystream"

  messages.forEach(processMessage);

  // Pass the last id of the results to the next round.
  await listenForMessage(messages[messages.length - 1][0]);
}

listenForMessage();
```

## 操作二进制文件

Arguments can be buffers:

```javascript
redis.set("foo", Buffer.from("bar"));
```

And every command has a method that returns a Buffer (by adding a suffix of "Buffer" to the command name).
To get a buffer instead of a utf8 string:

```javascript
redis.getBuffer("foo", (err, result) => {
  // result is a buffer.
});
```
