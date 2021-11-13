---
title: "ioredis"
linkTitle: ""
weight: 4
---

> https://github.com/luin/ioredis

[![ioredis](https://cdn.jsdelivr.net/gh/luin/ioredis@b5e8c74/logo.svg)](https://github.com/luin/ioredis)

[![Build Status](https://travis-ci.org/luin/ioredis.svg?branch=master)](https://travis-ci.org/luin/ioredis)
[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat-square)](https://github.com/prettier/prettier)
[![Join the chat at https://gitter.im/luin/ioredis](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/luin/ioredis?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)
[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg)](https://github.com/semantic-release/semantic-release)
[![npm latest version](https://img.shields.io/npm/v/ioredis/latest.svg)](https://www.npmjs.com/package/ioredis)
[![npm next version](https://img.shields.io/npm/v/ioredis/next.svg)](https://www.npmjs.com/package/ioredis)
<img alt="" src="">

A robust, performance-focused and full-featured [Redis](http://redis.io) client for [Node.js](https://nodejs.org).

Supports Redis >= 2.6.12 and (Node.js >= 6). Completely compatible with Redis 6.x.

## 特性

ioredis is a robust, full-featured Redis client that is
used in the world's biggest online commerce company [Alibaba](http://www.alibaba.com/) and many other awesome companies.

0. Full-featured. It supports [Cluster](http://redis.io/topics/cluster-tutorial), [Sentinel](http://redis.io/topics/sentinel), [Streams](https://redis.io/topics/streams-intro), [Pipelining](http://redis.io/topics/pipelining) and of course [Lua scripting](http://redis.io/commands/eval) & [Pub/Sub](http://redis.io/topics/pubsub) (with the support of binary messages).
1. High performance.
2. Delightful API. It works with Node callbacks and Native promises.
3. Transformation of command arguments and replies.
4. Transparent key prefixing.
5. Abstraction for Lua scripting, allowing you to define custom commands.
6. Support for binary data.
7. Support for TLS 🔒.
8. Support for offline queue and ready checking.
9. Support for ES6 types, such as `Map` and `Set`.
10. Support for GEO commands 📍.
11. Support for Redis ACL.
12. Sophisticated error handling strategy.
13. Support for NAT mapping.
14. Support for autopipelining

## 链接

- [API Documentation](API.md)
- [Changelog](Changelog.md)
- [Migrating from node_redis](https://github.com/luin/ioredis/wiki/Migrating-from-node_redis)
- [Error Handling](#error-handling)

<hr>

## 赞助商

<a href="https://github.com/sponsors/luin">Become a sponsor!</a>

#### Upstash: Serverless Database for Redis

<a href="https://upstash.com/?utm_source=ioredis"><img align="right" width="320" src="resources/upstash.png" alt="Upstash"></a>

Upstash is a Serverless Database with Redis/REST API and durable storage. It is the perfect database for your applications thanks to its per request pricing and low latency data.

[Start for free in 30 seconds! ](https://upstash.com/?utm_source=ioredis)

<br clear="both"/>

#### Medis: Redis GUI for macOS

<a href="https://getmedis.com/"><img align="right" width="404" src="resources/medis.png" alt="Download on the App Store"></a>

Looking for a Redis GUI for macOS, Windows and Linux? Here's [Medis](https://getmedis.com/)!

Medis is an open-sourced, beautiful, easy-to-use Redis GUI management application.

Medis starts with all the basic features you need:

- Keys viewing/editing
- SSH Tunnel for connecting with remote servers
- Terminal for executing custom commands
- And other awesome features...

[Medis 1 is open sourced on GitHub](https://github.com/luin/medis)

#### Kuber: Kubernetes Dashboard for iOS

<a href="http://bit.ly/kuber-ios"><img src="resources/kuber.png" alt="Download on the App Store"></a>

<hr>

## 错误处理

All the errors returned by the Redis server are instances of `ReplyError`, which can be accessed via `Redis`:

```javascript
const Redis = require("ioredis");
const redis = new Redis();
// This command causes a reply error since the SET command requires two arguments.
redis.set("foo", (err) => {
  err instanceof Redis.ReplyError;
});
```

This is the error stack of the `ReplyError`:

```
ReplyError: ERR wrong number of arguments for 'set' command
    at ReplyParser._parseResult (/app/node_modules/ioredis/lib/parsers/javascript.js:60:14)
    at ReplyParser.execute (/app/node_modules/ioredis/lib/parsers/javascript.js:178:20)
    at Socket.<anonymous> (/app/node_modules/ioredis/lib/redis/event_handler.js:99:22)
    at Socket.emit (events.js:97:17)
    at readableAddChunk (_stream_readable.js:143:16)
    at Socket.Readable.push (_stream_readable.js:106:10)
    at TCP.onread (net.js:509:20)
```

By default, the error stack doesn't make any sense because the whole stack happens in the ioredis
module itself, not in your code. So it's not easy to find out where the error happens in your code.
ioredis provides an option `showFriendlyErrorStack` to solve the problem. When you enable
`showFriendlyErrorStack`, ioredis will optimize the error stack for you:

```javascript
const Redis = require("ioredis");
const redis = new Redis({ showFriendlyErrorStack: true });
redis.set("foo");
```

And the output will be:

```
ReplyError: ERR wrong number of arguments for 'set' command
    at Object.<anonymous> (/app/index.js:3:7)
    at Module._compile (module.js:446:26)
    at Object.Module._extensions..js (module.js:464:10)
    at Module.load (module.js:341:32)
    at Function.Module._load (module.js:296:12)
    at Function.Module.runMain (module.js:487:10)
    at startup (node.js:111:16)
    at node.js:799:3
```

This time the stack tells you that the error happens on the third line in your code. Pretty sweet!
However, it would decrease the performance significantly to optimize the error stack. So by
default, this option is disabled and can only be used for debugging purposes. You **shouldn't** use this feature in a production environment.

## 插入你自己的承诺库

If you're an advanced user, you may want to plug in your own promise library like [bluebird](https://www.npmjs.com/package/bluebird). Just set Redis.Promise to your favorite ES6-style promise constructor and ioredis will use it.

```javascript
const Redis = require("ioredis");
Redis.Promise = require("bluebird");

const redis = new Redis();

// Use bluebird
assert.equal(redis.get().constructor, require("bluebird"));

// You can change the Promise implementation at any time:
Redis.Promise = global.Promise;
assert.equal(redis.get().constructor, global.Promise);
```

## 运行测试

Start a Redis server on 127.0.0.1:6379, and then:

```shell
$ npm test
```

`FLUSH ALL` will be invoked after each test, so make sure there's no valuable data in it before running tests.

If your testing environment does not let you spin up a Redis server [ioredis-mock](https://github.com/stipsan/ioredis-mock) is a drop-in replacement you can use in your tests. It aims to behave identically to ioredis connected to a Redis server so that your integration tests is easier to write and of better quality.

## 调试

You can set the `DEBUG` env to `ioredis:*` to print debug info:

```shell
$ DEBUG=ioredis:* node app.js
```

## 加入!

I'm happy to receive bug reports, fixes, documentation enhancements, and any other improvements.

And since I'm not a native English speaker, if you find any grammar mistakes in the documentation, please also let me know. :)

## 成为一个赞助商

Open source is hard and time-consuming. If you want to invest in ioredis's future you can become a sponsor and make us spend more time on this library's improvements and new features.

<a href="https://opencollective.com/ioredis"><img src="https://opencollective.com/ioredis/tiers/sponsor.svg?avatarHeight=36"></a>

Thank you for using ioredis :-)

## 贡献者

这个项目的存在要感谢所有做出贡献的人:

<a href="https://github.com/luin/ioredis/graphs/contributors"><img src="https://opencollective.com/ioredis/contributors.svg?width=890&showBtn=false" /></a>

## 许可证

MIT

[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fluin%2Fioredis.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fluin%2Fioredis?ref=badge_large)
