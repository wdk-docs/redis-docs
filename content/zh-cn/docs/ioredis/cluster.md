---
title: "Cluster"
linkTitle: ""
weight: 18
---

Redis Cluster provides a way to run a Redis installation where data is automatically sharded across multiple Redis nodes.
You can connect to a Redis Cluster like this:

```javascript
const Redis = require("ioredis");

const cluster = new Redis.Cluster([
  {
    port: 6380,
    host: "127.0.0.1",
  },
  {
    port: 6381,
    host: "127.0.0.1",
  },
]);

cluster.set("foo", "bar");
cluster.get("foo", (err, res) => {
  // res === 'bar'
});
```

`Cluster` constructor accepts two arguments, where:

0.  The first argument is a list of nodes of the cluster you want to connect to.
    Just like Sentinel, the list does not need to enumerate all your cluster nodes,
    but a few so that if one is unreachable the client will try the next one, and the client will discover other nodes automatically when at least one node is connected.
1.  The second argument is the options, where:

    - `clusterRetryStrategy`: When none of the startup nodes are reachable, `clusterRetryStrategy` will be invoked. When a number is returned,
      ioredis will try to reconnect to the startup nodes from scratch after the specified delay (in ms). Otherwise, an error of "None of startup nodes is available" will be returned.
      The default value of this option is:

      ```javascript
      function (times) {
        const delay = Math.min(100 + times * 2, 2000);
        return delay;
      }
      ```

      It's possible to modify the `startupNodes` property in order to switch to another set of nodes here:

      ```javascript
      function (times) {
        this.startupNodes = [{ port: 6790, host: '127.0.0.1' }];
        return Math.min(100 + times * 2, 2000);
      }
      ```

    - `dnsLookup`: Alternative DNS lookup function (`dns.lookup()` is used by default). It may be useful to override this in special cases, such as when AWS ElastiCache used with TLS enabled.
    - `enableOfflineQueue`: Similar to the `enableOfflineQueue` option of `Redis` class.
    - `enableReadyCheck`: When enabled, "ready" event will only be emitted when `CLUSTER INFO` command
      reporting the cluster is ready for handling commands. Otherwise, it will be emitted immediately after "connect" is emitted.
    - `scaleReads`: Config where to send the read queries. See below for more details.
    - `maxRedirections`: When a cluster related error (e.g. `MOVED`, `ASK` and `CLUSTERDOWN` etc.) is received, the client will redirect the
      command to another node. This option limits the max redirections allowed when sending a command. The default value is `16`.
    - `retryDelayOnFailover`: If the target node is disconnected when sending a command,
      ioredis will retry after the specified delay. The default value is `100`. You should make sure `retryDelayOnFailover * maxRedirections > cluster-node-timeout`
      to insure that no command will fail during a failover.
    - `retryDelayOnClusterDown`: When a cluster is down, all commands will be rejected with the error of `CLUSTERDOWN`. If this option is a number (by default, it is `100`), the client
      will resend the commands after the specified time (in ms).
    - `retryDelayOnTryAgain`: If this option is a number (by default, it is `100`), the client
      will resend the commands rejected with `TRYAGAIN` error after the specified time (in ms).
    - `retryDelayOnMoved`: By default, this value is `0` (in ms), which means when a `MOVED` error is received, the client will resend
      the command instantly to the node returned together with the `MOVED` error. However, sometimes it takes time for a cluster to become
      state stabilized after a failover, so adding a delay before resending can prevent a ping pong effect.
    - `redisOptions`: Default options passed to the constructor of `Redis` when connecting to a node.
    - `slotsRefreshTimeout`: Milliseconds before a timeout occurs while refreshing slots from the cluster (default `1000`).
    - `slotsRefreshInterval`: Milliseconds between every automatic slots refresh (default `5000`).

### Read-write splitting

A typical redis cluster contains three or more masters and several slaves for each master. It's possible to scale out redis cluster by sending read queries to slaves and write queries to masters by setting the `scaleReads` option.

`scaleReads` is "master" by default, which means ioredis will never send any queries to slaves. There are other three available options:

1. "all": Send write queries to masters and read queries to masters or slaves randomly.
2. "slave": Send write queries to masters and read queries to slaves.
3. a custom `function(nodes, command): node`: Will choose the custom function to select to which node to send read queries (write queries keep being sent to master). The first node in `nodes` is always the master serving the relevant slots. If the function returns an array of nodes, a random node of that list will be selected.

For example:

```javascript
const cluster = new Redis.Cluster(
  [
    /* nodes */
  ],
  {
    scaleReads: "slave",
  }
);
cluster.set("foo", "bar"); // This query will be sent to one of the masters.
cluster.get("foo", (err, res) => {
  // This query will be sent to one of the slaves.
});
```

**NB** In the code snippet above, the `res` may not be equal to "bar" because of the lag of replication between the master and slaves.

### Running commands to multiple nodes

Every command will be sent to exactly one node. For commands containing keys, (e.g. `GET`, `SET` and `HGETALL`), ioredis sends them to the node that serving the keys, and for other commands not containing keys, (e.g. `INFO`, `KEYS` and `FLUSHDB`), ioredis sends them to a random node.

Sometimes you may want to send a command to multiple nodes (masters or slaves) of the cluster, you can get the nodes via `Cluster#nodes()` method.

`Cluster#nodes()` accepts a parameter role, which can be "master", "slave" and "all" (default), and returns an array of `Redis` instance. For example:

```javascript
// Send `FLUSHDB` command to all slaves:
const slaves = cluster.nodes("slave");
Promise.all(slaves.map((node) => node.flushdb()));

// Get keys of all the masters:
const masters = cluster.nodes("master");
Promise.all(
  masters
    .map((node) => node.keys())
    .then((keys) => {
      // keys: [['key1', 'key2'], ['key3', 'key4']]
    })
);
```

### NAT Mapping

Sometimes the cluster is hosted within a internal network that can only be accessed via a NAT (Network Address Translation) instance. See [Accessing ElastiCache from outside AWS](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/accessing-elasticache.html) as an example.

You can specify nat mapping rules via `natMap` option:

```javascript
const cluster = new Redis.Cluster(
  [
    {
      host: "203.0.113.73",
      port: 30001,
    },
  ],
  {
    natMap: {
      "10.0.1.230:30001": { host: "203.0.113.73", port: 30001 },
      "10.0.1.231:30001": { host: "203.0.113.73", port: 30002 },
      "10.0.1.232:30001": { host: "203.0.113.73", port: 30003 },
    },
  }
);
```

This option is also useful when the cluster is running inside a Docker container.

### Transaction and pipeline in Cluster mode

Almost all features that are supported by `Redis` are also supported by `Redis.Cluster`, e.g. custom commands, transaction and pipeline.
However there are some differences when using transaction and pipeline in Cluster mode:

0. All keys in a pipeline should belong to slots served by the same node, since ioredis sends all commands in a pipeline to the same node.
1. You can't use `multi` without pipeline (aka `cluster.multi({ pipeline: false })`). This is because when you call `cluster.multi({ pipeline: false })`, ioredis doesn't know which node the `multi` command should be sent to.

When any commands in a pipeline receives a `MOVED` or `ASK` error, ioredis will resend the whole pipeline to the specified node automatically if all of the following conditions are satisfied:

0. All errors received in the pipeline are the same. For example, we won't resend the pipeline if we got two `MOVED` errors pointing to different nodes.
1. All commands executed successfully are readonly commands. This makes sure that resending the pipeline won't have side effects.

### Pub/Sub

Pub/Sub in cluster mode works exactly as the same as in standalone mode. Internally, when a node of the cluster receives a message, it will broadcast the message to the other nodes. ioredis makes sure that each message will only be received once by strictly subscribing one node at the same time.

```javascript
const nodes = [
  /* nodes */
];
const pub = new Redis.Cluster(nodes);
const sub = new Redis.Cluster(nodes);
sub.on("message", (channel, message) => {
  console.log(channel, message);
});

sub.subscribe("news", () => {
  pub.publish("news", "highlights");
});
```

### Events

| Event        | Description                                                                                                                                                                                                |
| :----------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| connect      | emits when a connection is established to the Redis server.                                                                                                                                                |
| ready        | emits when `CLUSTER INFO` reporting the cluster is able to receive commands (if `enableReadyCheck` is `true`) or immediately after `connect` event (if `enableReadyCheck` is false).                       |
| error        | emits when an error occurs while connecting with a property of `lastNodeError` representing the last node error received. This event is emitted silently (only emitting if there's at least one listener). |
| close        | emits when an established Redis server connection has closed.                                                                                                                                              |
| reconnecting | emits after `close` when a reconnection will be made. The argument of the event is the time (in ms) before reconnecting.                                                                                   |
| end          | emits after `close` when no more reconnections will be made.                                                                                                                                               |
| +node        | emits when a new node is connected.                                                                                                                                                                        |
| -node        | emits when a node is disconnected.                                                                                                                                                                         |
| node error   | emits when an error occurs when connecting to a node. The second argument indicates the address of the node.                                                                                               |

### Password

Setting the `password` option to access password-protected clusters:

```javascript
const Redis = require("ioredis");
const cluster = new Redis.Cluster(nodes, {
  redisOptions: {
    password: "your-cluster-password",
  },
});
```

If some of nodes in the cluster using a different password, you should specify them in the first parameter:

```javascript
const Redis = require("ioredis");
const cluster = new Redis.Cluster(
  [
    // Use password "password-for-30001" for 30001
    { port: 30001, password: "password-for-30001" },
    // Don't use password when accessing 30002
    { port: 30002, password: null },
    // Other nodes will use "fallback-password"
  ],
  {
    redisOptions: {
      password: "fallback-password",
    },
  }
);
```

### Special note: AWS ElastiCache Clusters with TLS

AWS ElastiCache for Redis (Clustered Mode) supports TLS encryption. If you use
this, you may encounter errors with invalid certificates. To resolve this
issue, construct the `Cluster` with the `dnsLookup` option as follows:

```javascript
const cluster = new Redis.Cluster(
  [
    {
      host: "clustercfg.myCluster.abcdefg.xyz.cache.amazonaws.com",
      port: 6379,
    },
  ],
  {
    dnsLookup: (address, callback) => callback(null, address),
    redisOptions: {
      tls: {},
    },
  }
);
```

<hr>
