---
title: "Sentinel"
linkTitle: ""
weight: 17
---

ioredis supports Sentinel out of the box. It works transparently as all features that work when
you connect to a single node also work when you connect to a sentinel group. Make sure to run Redis >= 2.8.12 if you want to use this feature. Sentinels have a default port of 26379.

To connect using Sentinel, use:

```javascript
const redis = new Redis({
  sentinels: [
    { host: "localhost", port: 26379 },
    { host: "localhost", port: 26380 },
  ],
  name: "mymaster",
});

redis.set("foo", "bar");
```

The arguments passed to the constructor are different from the ones you use to connect to a single node, where:

- `name` identifies a group of Redis instances composed of a master and one or more slaves (`mymaster` in the example);
- `sentinelPassword` (optional) password for Sentinel instances.
- `sentinels` are a list of sentinels to connect to. The list does not need to enumerate all your sentinel instances, but a few so that if one is down the client will try the next one.
- `role` (optional) with a value of `slave` will return a random slave from the Sentinel group.
- `preferredSlaves` (optional) can be used to prefer a particular slave or set of slaves based on priority. It accepts a function or array.
- `enableTLSForSentinelMode` (optional) set to true if connecting to sentinel instances that are encrypted

ioredis **guarantees** that the node you connected to is always a master even after a failover. When a failover happens, instead of trying to reconnect to the failed node (which will be demoted to slave when it's available again), ioredis will ask sentinels for the new master node and connect to it. All commands sent during the failover are queued and will be executed when the new connection is established so that none of the commands will be lost.

It's possible to connect to a slave instead of a master by specifying the option `role` with the value of `slave` and ioredis will try to connect to a random slave of the specified master, with the guarantee that the connected node is always a slave. If the current node is promoted to master due to a failover, ioredis will disconnect from it and ask the sentinels for another slave node to connect to.

If you specify the option `preferredSlaves` along with `role: 'slave'` ioredis will attempt to use this value when selecting the slave from the pool of available slaves. The value of `preferredSlaves` should either be a function that accepts an array of available slaves and returns a single result, or an array of slave values priorities by the lowest `prio` value first with a default value of `1`.

```javascript
// available slaves format
const availableSlaves = [{ ip: "127.0.0.1", port: "31231", flags: "slave" }];

// preferredSlaves array format
let preferredSlaves = [
  { ip: "127.0.0.1", port: "31231", prio: 1 },
  { ip: "127.0.0.1", port: "31232", prio: 2 },
];

// preferredSlaves function format
preferredSlaves = function (availableSlaves) {
  for (let i = 0; i < availableSlaves.length; i++) {
    const slave = availableSlaves[i];
    if (slave.ip === "127.0.0.1") {
      if (slave.port === "31234") {
        return slave;
      }
    }
  }
  // if no preferred slaves are available a random one is used
  return false;
};

const redis = new Redis({
  sentinels: [
    { host: "127.0.0.1", port: 26379 },
    { host: "127.0.0.1", port: 26380 },
  ],
  name: "mymaster",
  role: "slave",
  preferredSlaves: preferredSlaves,
});
```

Besides the `retryStrategy` option, there's also a `sentinelRetryStrategy` in Sentinel mode which will be invoked when all the sentinel nodes are unreachable during connecting. If `sentinelRetryStrategy` returns a valid delay time, ioredis will try to reconnect from scratch. The default value of `sentinelRetryStrategy` is:

```javascript
function (times) {
  const delay = Math.min(times * 10, 1000);
  return delay;
}
```
