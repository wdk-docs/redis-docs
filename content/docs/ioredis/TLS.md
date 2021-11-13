---
title: "TLS"
linkTitle: ""
weight: 10
---

## TLS 选项

Redis doesn't support TLS natively, however if the redis server you want to connect to is hosted behind a TLS proxy (e.g. [stunnel](https://www.stunnel.org/)) or is offered by a PaaS service that supports TLS connection (e.g. [Redis.com](https://redis.com/)), you can set the `tls` option:

```javascript
const redis = new Redis({
  host: "localhost",
  tls: {
    // Refer to `tls.connect()` section in
    // https://nodejs.org/api/tls.html
    // for all supported options
    ca: fs.readFileSync("cert.pem"),
  },
});
```

Alternatively, specify the connection through a [`rediss://` URL](https://www.iana.org/assignments/uri-schemes/prov/rediss).

```javascript
const redis = new Redis("rediss://redis.my-service.com");
```

### TLS 配置文件

To make it easier to configure we provide a few pre-configured TLS profiles that can be specified by setting the `tls` option to the profile's name or specifying a `tls.profile` option in case you need to customize some values of the profile.

Profiles:

- `RedisCloudFixed`: Contains the CA for [Redis.com](https://redis.com/) Cloud fixed subscriptions
- `RedisCloudFlexible`: Contains the CA for [Redis.com](https://redis.com/) Cloud flexible subscriptions

```javascript
const redis = new Redis({
  host: "localhost",
  tls: "RedisCloudFixed",
});

const redisWithClientCertificate = new Redis({
  host: "localhost",
  tls: {
    profile: "RedisCloudFixed",
    key: "123",
  },
});
```

<hr>
