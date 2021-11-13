---
title: "连接"
linkTitle: ""
weight: 2
---

## 连接到 Redis

When a new `Redis` instance is created, a connection to Redis will be created at the same time.
You can specify which Redis to connect to by:

```javascript
new Redis(); // Connect to 127.0.0.1:6379
new Redis(6380); // 127.0.0.1:6380
new Redis(6379, "192.168.1.1"); // 192.168.1.1:6379
new Redis("/tmp/redis.sock");
new Redis({
  port: 6379, // Redis port
  host: "127.0.0.1", // Redis host
  family: 4, // 4 (IPv4) or 6 (IPv6)
  password: "auth",
  db: 0,
});
```

You can also specify connection options as a [`redis://` URL](http://www.iana.org/assignments/uri-schemes/prov/redis) or [`rediss://` URL](https://www.iana.org/assignments/uri-schemes/prov/rediss) when using [TLS encryption](#tls-options):

```javascript
// Connect to 127.0.0.1:6380, db 4, using password "authpassword":
new Redis("redis://:authpassword@127.0.0.1:6380/4");

// Username can also be passed via URI.
// It's worth to noticing that for compatibility reasons `allowUsernameInURI`
// need to be provided, otherwise the username part will be ignored.
new Redis(
  "redis://username:authpassword@127.0.0.1:6380/4?allowUsernameInURI=true"
);
```

See [API Documentation](API.md#new_Redis) for all available options.

## 自动连接

By default, ioredis will try to reconnect when the connection to Redis is lost except when the connection is closed manually by `redis.disconnect()` or `redis.quit()`.

It's very flexible to control how long to wait to reconnect after disconnection using the `retryStrategy` option:

```javascript
const redis = new Redis({
  // This is the default value of `retryStrategy`
  retryStrategy(times) {
    const delay = Math.min(times * 50, 2000);
    return delay;
  },
});
```

`retryStrategy` is a function that will be called when the connection is lost.
The argument `times` means this is the nth reconnection being made and
the return value represents how long (in ms) to wait to reconnect. When the
return value isn't a number, ioredis will stop trying to reconnect, and the connection
will be lost forever if the user doesn't call `redis.connect()` manually.

When reconnected, the client will auto subscribe to channels that the previous connection subscribed to.
This behavior can be disabled by setting the `autoResubscribe` option to `false`.

And if the previous connection has some unfulfilled commands (most likely blocking commands such as `brpop` and `blpop`),
the client will resend them when reconnected. This behavior can be disabled by setting the `autoResendUnfulfilledCommands` option to `false`.

By default, all pending commands will be flushed with an error every 20 retry attempts. That makes sure commands won't wait forever when the connection is down. You can change this behavior by setting `maxRetriesPerRequest`:

```javascript
const redis = new Redis({
  maxRetriesPerRequest: 1,
});
```

Set maxRetriesPerRequest to `null` to disable this behavior, and every command will wait forever until the connection is alive again (which is the default behavior before ioredis v4).

### Reconnect on error

Besides auto-reconnect when the connection is closed, ioredis supports reconnecting on certain Redis errors using the `reconnectOnError` option. Here's an example that will reconnect when receiving `READONLY` error:

```javascript
const redis = new Redis({
  reconnectOnError(err) {
    const targetError = "READONLY";
    if (err.message.includes(targetError)) {
      // Only reconnect when the error contains "READONLY"
      return true; // or `return 1;`
    }
  },
});
```

This feature is useful when using Amazon ElastiCache instances with Auto-failover disabled. On these instances, test your `reconnectOnError` handler by manually promoting the replica node to the primary role using the AWS console. The following writes fail with the error `READONLY`. Using `reconnectOnError`, we can force the connection to reconnect on this error in order to connect to the new master. Furthermore, if the `reconnectOnError` returns `2`, ioredis will resend the failed command after reconnecting.

On ElastiCache instances with Auto-failover enabled, `reconnectOnError` does not execute. Instead of returning a Redis error, AWS closes all connections to the master endpoint until the new primary node is ready. ioredis reconnects via `retryStrategy` instead of `reconnectOnError` after about a minute. On ElastiCache instances with Auto-failover enabled, test failover events with the `Failover primary` option in the AWS console.

## Connection Events

The Redis instance will emit some events about the state of the connection to the Redis server.

| Event        | Description                                                                                                                                                                                                                                     |
| :----------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| connect      | emits when a connection is established to the Redis server.                                                                                                                                                                                     |
| ready        | If `enableReadyCheck` is `true`, client will emit `ready` when the server reports that it is ready to receive commands (e.g. finish loading data from disk).<br>Otherwise, `ready` will be emitted immediately right after the `connect` event. |
| error        | emits when an error occurs while connecting.<br>However, ioredis emits all `error` events silently (only emits when there's at least one listener) so that your application won't crash if you're not listening to the `error` event.           |
| close        | emits when an established Redis server connection has closed.                                                                                                                                                                                   |
| reconnecting | emits after `close` when a reconnection will be made. The argument of the event is the time (in ms) before reconnecting.                                                                                                                        |
| end          | emits after `close` when no more reconnections will be made, or the connection is failed to establish.                                                                                                                                          |
| wait         | emits when `lazyConnect` is set and will wait for the first command to be called before connecting.                                                                                                                                             |

You can also check out the `Redis#status` property to get the current connection status.

Besides the above connection events, there are several other custom events:

| Event  | Description                                                         |
| :----- | :------------------------------------------------------------------ |
| select | emits when the database changed. The argument is the new db number. |
