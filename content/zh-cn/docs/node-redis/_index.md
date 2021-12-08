---
title: "Node Redis"
linkTitle: ""
weight: 6
---

node-redis 是 Node.js 的一个现代的，高性能[Redis](https://redis.io)客户端,内置支持 Redis 6.2 命令和模块，包括[reresearch](https://redisearch.io)和[RedisJSON](https://redisjson.io)。

## 安装

```bash
npm install redis
```

> :warning: 新的接口很干净很酷，但是如果你有一个现有的代码库，你会想要阅读[迁移指南](./docs/v3-to-v4.md)。

## 使用

### 基本示例

```typescript
import { createClient } from "redis";

(async () => {
  const client = createClient();

  client.on("error", (err) => console.log("Redis Client Error", err));

  await client.connect();

  await client.set("key", "value");
  const value = await client.get("key");
})();
```

上面的代码连接到端口 6379 上的本地主机。
连接到一个不同的主机或端口，使用格式为`redis[s]://[[username][:password]@][host][:port][/db-number]`的连接字符串:

```typescript
createClient({ url: "redis://alice:foobared@awesome.redis.server:6380" });
```

您还可以使用离散参数、UNIX 套接字甚至 TLS 进行连接。
详细信息请参见[客户端配置指南](./docs/client-configuration.md).

### Redis 命令

内置支持所有的[开箱即用的 Redis 命令](https://redis.io/commands).
他们使用原始的 Redis 命令名(`HSET`, `HGETALL`, 等)和一个更友好的驼式大小写版本(`hSet`, `hGetAll`, 等):

```typescript
// raw Redis commands
await client.HSET("key", "field", "value");
await client.HGETALL("key");

// friendly JavaScript commands
await client.hSet("key", "field", "value");
await client.hGetAll("key");
```

命令的修饰符是使用 JavaScript 对象指定的:

```typescript
await client.set("key", "value", { EX: 10, NX: true });
```

回复将被转换成有用的数据结构:

```typescript
await client.hGetAll("key"); // { field1: 'value1', field2: 'value2' }
await client.hVals("key"); // ['value1', 'value2']
```

### 不支持的 Redis 命令

如果你想运行命令和/或使用节点 Redis 不知道的参数(还!)使用`sendCommand`:

```typescript
await client.sendCommand(["SET", "key", "value", "NX"]); // 'OK'

await client.sendCommand(["HGETALL", "key"]); // ['key1', 'field1', 'key2', 'field2']
```

### 事务 (Multi/Exec)

通过调用`.multi()`启动[事务](https://redis.io/topics/transactions), 然后链接你的命令.
当您完成时，调用`.exec()`，您将得到一个带有结果的数组:

```typescript
await client.set("another-key", "another-value");

const [setKeyReply, otherKeyValue] = await client
  .multi()
  .set("key", "value")
  .get("another-key")
  .exec(); // ['OK', 'another-value']
```

你也可以通过调用`.watch()`来[查看](https://redis.io/topics/transactions#optimistic-locking-using-check-and-set)键。
如果任何被监视的键发生变化，您的事务将中止。

要深入了解事务，请查看[隔离执行指南](./docs/isolated-execution.md).

### 阻塞的命令

通过指定`isolated`选项，可以在新连接上运行任何命令。
当命令的`Promise`完成时，新创建的连接将关闭。

这种模式尤其适用于阻塞命令，例如`BLPOP`和`BLMOVE`:

```typescript
import { commandOptions } from "redis";

const blPopPromise = client.blPop(commandOptions({ isolated: true }), "key", 0);

await client.lPush("key", ["1", "2"]);

await blPopPromise; // '2'
```

要了解更多关于隔离执行的信息，请查看[指南](./docs/isolated-execution.md).

### Pub/Sub

订阅通道需要专用的独立连接。
你可以很容易地通过`.duplicate()`得到一个现有的 Redis 连接。

```typescript
const subscriber = client.duplicate();

await subscriber.connect();
```

一旦你有了一个，只需按需要订阅和取消订阅:

```typescript
await subscriber.subscribe("channel", (message) => {
  console.log(message); // 'message'
});

await subscriber.pSubscribe("channe*", (message, channel) => {
  console.log(message, channel); // 'message', 'channel'
});

await subscriber.unsubscribe("channel");

await subscriber.pUnsubscribe("channe*");
```

在通道上发布消息:

```typescript
await publisher.publish("channel", "message");
```

它还支持缓冲区:

```typescript
await subscriber.subscribe(
  "channel",
  (message) => {
    console.log(message); // <Buffer 6d 65 73 73 61 67 65>
  },
  true
);

await subscriber.pSubscribe(
  "channe*",
  (message, channel) => {
    console.log(message, channel); // <Buffer 6d 65 73 73 61 67 65>, <Buffer 63 68 61 6e 6e 65 6c>
  },
  true
);
```

### 扫描迭代器

[`SCAN`](https://redis.io/commands/scan)结果可以使用[异步迭代器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/asyncIterator)循环:

```typescript
for await (const key of client.scanIterator()) {
  // use the key!
  await client.get(key);
}
```

这也适用于`HSCAN`， `SSCAN`和`ZSCAN`:

```typescript
for await (const { field, value } of client.hScanIterator("hash")) {
}
for await (const member of client.sScanIterator("set")) {
}
for await (const { score, value } of client.zScanIterator("sorted-set")) {
}
```

您可以通过提供一个配置对象来覆盖默认选项:

```typescript
client.scanIterator({
  TYPE: "string", // `SCAN` only
  MATCH: "patter*",
  COUNT: 100,
});
```

### Lua 脚本

使用[Lua 脚本](https://redis.io/commands/eval)定义新的功能在 Redis 服务器执行

```typescript
import { createClient, defineScript } from "redis";

(async () => {
  const client = createClient({
    scripts: {
      add: defineScript({
        NUMBER_OF_KEYS: 1,
        SCRIPT:
          'local val = redis.pcall("GET", KEYS[1]);' + "return val + ARGV[1];",
        transformArguments(key: string, toAdd: number): Array<string> {
          return [key, toAdd.toString()];
        },
        transformReply(reply: number): number {
          return reply;
        },
      }),
    },
  });

  await client.connect();

  await client.set("key", "1");
  await client.add("key", 2); // 3
})();
```

### 断开

There are two functions that disconnect a client from the Redis server.
In most scenarios you should use `.quit()` to ensure that pending commands are sent to Redis before closing a connection.

#### `.QUIT()`/`.quit()`

Gracefully close a client's connection to Redis, by sending the [`QUIT`](https://redis.io/commands/quit) command to the server. Before quitting, the client executes any remaining commands in its queue, and will receive replies from Redis for each of them.

```typescript
const [ping, get, quit] = await Promise.all([
  client.ping(),
  client.get("key"),
  client.quit(),
]); // ['PONG', null, 'OK']

try {
  await client.get("key");
} catch (err) {
  // ClosedClient Error
}
```

#### `.disconnect()`

Forcibly close a client's connection to Redis immediately. Calling `disconnect` will not send further pending commands to the Redis server, or wait for or parse outstanding responses.

```typescript
await client.disconnect();
```

### 自动流水线

Node Redis will automatically pipeline requests that are made during the same "tick".

```typescript
client.set("Tm9kZSBSZWRpcw==", "users:1");
client.sAdd("users:1:tokens", "Tm9kZSBSZWRpcw==");
```

Of course, if you don't do something with your Promises you're certain to get [unhandled Promise exceptions](https://nodejs.org/api/process.html#process_event_unhandledrejection). To take advantage of auto-pipelining and handle your Promises, use `Promise.all()`.

```typescript
await Promise.all([
  client.set("Tm9kZSBSZWRpcw==", "users:1"),
  client.sAdd("users:1:tokens", "Tm9kZSBSZWRpcw=="),
]);
```

### 聚类

Check out the [Clustering Guide](./docs/clustering.md) when using Node Redis to connect to a Redis Cluster.

### 事件

The Node Redis client class is an Nodejs EventEmitter and it emits an event each time the network status changes:

| Event name   | Scenes                                                                                                            | Parameters                                                                                                                         |
| ------------ | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| connect      | The client is initiating a connection to the server.                                                              | _undefined_                                                                                                                        |
| ready        | The client successfully initiated the connection to the server.                                                   | _undefined_                                                                                                                        |
| end          | The client disconnected the connection to the server via `.quit()` or `.disconnect()`.                            | _undefined_                                                                                                                        |
| error        | When a network error has occurred, such as unable to connect to the server or the connection closed unexpectedly. | The error object, such as `SocketClosedUnexpectedlyError: Socket closed unexpectedly` or `Error: connect ECONNREFUSED [IP]:[PORT]` |
| reconnecting | The client is trying to reconnect to the server.                                                                  | _undefined_                                                                                                                        |

The client will not emit any other events beyond those listed above.

## 支持 Redis 版本

Node Redis is supported with the following versions of Redis:

| Version | Supported          |
| ------- | ------------------ |
| 6.2.z   | :heavy_check_mark: |
| 6.0.z   | :heavy_check_mark: |
| 5.y.z   | :heavy_check_mark: |
| < 5.0   | :x:                |

> Node Redis should work with older versions of Redis, but it is not fully tested and we cannot offer support.

## 包

| Name                                    | Description                                                                                                                                                                                                                                                                                           |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [redis](./)                             | [![Downloads](https://img.shields.io/npm/dm/redis.svg)](https://www.npmjs.com/package/redis) [![Version](https://img.shields.io/npm/v/redis.svg)](https://www.npmjs.com/package/redis)                                                                                                                |
| [@node-redis/client](./packages/client) | [![Downloads](https://img.shields.io/npm/dm/@node-redis/client.svg)](https://www.npmjs.com/package/@node-redis/client) [![Version](https://img.shields.io/npm/v/@node-redis/client.svg)](https://www.npmjs.com/package/@node-redis/client)                                                            |
| [@node-redis/json](./packages/json)     | [![Downloads](https://img.shields.io/npm/dm/@node-redis/json.svg)](https://www.npmjs.com/package/@node-redis/json) [![Version](https://img.shields.io/npm/v/@node-redis/json.svg)](https://www.npmjs.com/package/@node-redis/json) [Redis JSON](https://oss.redis.com/redisjson/) commands            |
| [@node-redis/search](./packages/search) | [![Downloads](https://img.shields.io/npm/dm/@node-redis/search.svg)](https://www.npmjs.com/package/@node-redis/search) [![Version](https://img.shields.io/npm/v/@node-redis/search.svg)](https://www.npmjs.com/package/@node-redis/search) [Redis Search](https://oss.redis.com/redisearch/) commands |

## 贡献

If you'd like to contribute, check out the [contributing guide](CONTRIBUTING.md).

Thank you to all the people who already contributed to Node Redis!

[![Contributors](https://contrib.rocks/image?repo=redis/node-redis)](https://github.com/redis/node-redis/graphs/contributors)

## 证书

This repository is licensed under the "MIT" license. See [LICENSE](LICENSE).
