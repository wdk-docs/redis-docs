---
title: "egg redis"
linkTitle: ""
weight: 5
---

为`eggjs`框架基于 `ioredis` Redis 客户端(支持 redis 协议)

## 安装

```bash
$ npm i egg-redis --save
```

redis Plugin for egg, support egg application access to redis.

This plugin based on [ioredis](https://github.com/luin/ioredis), if you want to know specific usage, you should refer to the document of [ioredis](https://github.com/luin/ioredis).

## 配置

Change `${app_root}/config/plugin.js` to enable redis plugin:

```js
exports.redis = {
  enable: true,
  package: "egg-redis",
};
```

Configure redis information in `${app_root}/config/config.default.js`:

**Single Client**

```javascript
config.redis = {
  client: {
    port: 6379, // Redis port
    host: "127.0.0.1", // Redis host
    password: "auth",
    db: 0,
  },
};
```

**Multi Clients**

```javascript
config.redis = {
  clients: {
    foo: {
      // instanceName. See below
      port: 6379, // Redis port
      host: "127.0.0.1", // Redis host
      password: "auth",
      db: 0,
    },
    bar: {
      port: 6379,
      host: "127.0.0.1",
      password: "auth",
      db: 1,
    },
  },
};
```

**Sentinel**

```javascript
config.redis = {
  client: {
    sentinels: [
      {
        // Sentinel instances
        port: 26379, // Sentinel port
        host: "127.0.0.1", // Sentinel host
      },
    ],
    name: "mymaster", // Master name
    password: "auth",
    db: 0,
  },
};
```

**No password**

Redis support no authentication access, but we are highly recommand you to use redis `requirepass` in `redis.conf`.

```bash

$vim /etc/redis/redis.conf

requirepass xxxxxxxxxx  // xxxxxxxxxx is your password

```

Because it may be cause security risk, refer:

- https://ruby-china.org/topics/28094
- https://zhuoroger.github.io/2016/07/29/redis-sec/

If you want to access redis with no password, use `password: null`.

See [ioredis API Documentation](https://github.com/luin/ioredis/blob/master/API.md#new_Redis) for all available options.

### 自定义 `ioredis` 版本

`egg-redis` using ioredis@4 now, if you want to use other version of ioredis, you can pass the instance by `config.redis.Redis`:

```js
// config/config.default.js
config.redis = {
  Redis: require("ioredis"), // customize ioredis version, only set when you needed
  client: {
    port: 6379, // Redis port
    host: "127.0.0.1", // Redis host
    password: "auth",
    db: 0,
  },
};
```

**weakDependent**

```javascript
config.redis = {
  client: {
    port: 6379, // Redis port
    host: "127.0.0.1", // Redis host
    password: "auth",
    db: 0,
    weakDependent: true, // this redis instance won't block app start
  },
};
```

## 使用

In controller, you can use `app.redis` to get the redis instance, check [ioredis](https://github.com/luin/ioredis#basic-usage) to see how to use.

```js
// app/controller/home.js

module.exports = (app) => {
  return class HomeController extends app.Controller {
    async index() {
      const { ctx, app } = this;
      // set
      await app.redis.set("foo", "bar");
      // get
      ctx.body = await app.redis.get("foo");
    }
  };
};
```

### 多客户端

If your Configure with multi clients, you can use `app.redis.get(instanceName)` to get the specific redis instance and use it like above.

```js
// app/controller/home.js

module.exports = (app) => {
  return class HomeController extends app.Controller {
    async index() {
      const { ctx, app } = this;
      // set
      await app.redis.get("instance1").set("foo", "bar");
      // get
      ctx.body = await app.redis.get("instance1").get("foo");
    }
  };
};
```

### 客户端依赖于 Redis 集群

Before you start to use Redis Cluster, please checkout the [document](https://redis.io/topics/cluster-tutorial) first, especially confirm `cluster-enabled yes` in Redis Cluster configuration file.

In controller, you also can use `app.redis` to get the redis instance based on Redis Cluster.

```js
// app/config/config.default.js

exports.redis = {
  client: {
    cluster: true,
    nodes: [
      {
        host: "127.0.0.1",
        port: "6379",
        family: "user",
        password: "password",
        db: "db",
      },
      {
        host: "127.0.0.1",
        port: "6380",
        family: "user",
        password: "password",
        db: "db",
      },
    ],
  },
};

// app/controller/home.js

module.exports = (app) => {
  return class HomeController extends app.Controller {
    async index() {
      const { ctx, app } = this;
      // set
      await app.redis.set("foo", "bar");
      // get
      ctx.body = await app.redis.get("foo");
    }
  };
};
```

## 问题与建议

Please open an issue [here](https://github.com/eggjs/egg/issues).
