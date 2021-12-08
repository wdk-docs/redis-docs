---
title: "Pub/Sub"
linkTitle: ""
weight: 17
---

Redis provides several commands for developers to implement the [Publish–subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). There are two roles in this pattern: publisher and subscriber. Publishers are not programmed to send their messages to specific subscribers. Rather, published messages are characterized into channels, without knowledge of what (if any) subscribers there may be.

By leveraging Node.js's built-in events module, ioredis makes pub/sub very straightforward to use. Below is a simple example that consists of two files, one is publisher.js that publishes messages to a channel, the other is subscriber.js that listens for messages on specific channels.

```javascript
// publisher.js

const Redis = require("ioredis");
const redis = new Redis();

setInterval(() => {
  const message = { foo: Math.random() };
  // Publish to my-channel-1 or my-channel-2 randomly.
  const channel = `my-channel-${1 + Math.round(Math.random())}`;

  // Message can be either a string or a buffer
  redis.publish(channel, JSON.stringify(message));
  console.log("Published %s to %s", message, channel);
}, 1000);
```

```javascript
// subscriber.js

const Redis = require("ioredis");
const redis = new Redis();

redis.subscribe("my-channel-1", "my-channel-2", (err, count) => {
  if (err) {
    // Just like other commands, subscribe() can fail for some reasons,
    // ex network issues.
    console.error("Failed to subscribe: %s", err.message);
  } else {
    // `count` represents the number of channels this client are currently subscribed to.
    console.log(
      `Subscribed successfully! This client is currently subscribed to ${count} channels.`
    );
  }
});

redis.on("message", (channel, message) => {
  console.log(`Received ${message} from ${channel}`);
});

// There's also an event called 'messageBuffer', which is the same as 'message' except
// it returns buffers instead of strings.
// It's useful when the messages are binary data.
redis.on("messageBuffer", (channel, message) => {
  // Both `channel` and `message` are buffers.
  console.log(channel, message);
});
```

It worth noticing that a connection (aka `Redis` instance) can't play both roles together. More specifically, when a client issues `subscribe()` or `psubscribe()`, it enters the "subscriber" mode. From that point, only commands that modify the subscription set are valid. Namely, they are: `subscribe`, `psubscribe`, `unsubscribe`, `punsubscribe`, `ping`, and `quit`. When the subscription set is empty (via `unsubscribe`/`punsubscribe`), the connection is put back into the regular mode.

If you want to do pub/sub in the same file/process, you should create a separate connection:

```javascript
const Redis = require("ioredis");
const sub = new Redis();
const pub = new Redis();

sub.subscribe(/* ... */); // From now, `sub` enters the subscriber mode.
sub.on("message" /* ... */);

setInterval(() => {
  // `pub` can be used to publish messages, or send other regular commands (e.g. `hgetall`)
  // because it's not in the subscriber mode.
  pub.publish(/* ... */);
}, 1000);
```

`PSUBSCRIBE` is also supported in a similar way when you want to subscribe all channels whose name matches a pattern:

```javascript
redis.psubscribe("pat?ern", (err, count) => {});

// Event names are "pmessage"/"pmessageBuffer" instead of "message/messageBuffer".
redis.on("pmessage", (pattern, channel, message) => {});
redis.on("pmessageBuffer", (pattern, channel, message) => {});
```
