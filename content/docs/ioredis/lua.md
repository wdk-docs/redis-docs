---
title: "Lua 脚本"
linkTitle: ""
weight: 17
---

ioredis supports all of the scripting commands such as `EVAL`, `EVALSHA` and `SCRIPT`.
However, it's tedious to use in real world scenarios since developers have to take
care of script caching and to detect when to use `EVAL` and when to use `EVALSHA`.
ioredis exposes a `defineCommand` method to make scripting much easier to use:

```javascript
const redis = new Redis();

// This will define a command echo:
redis.defineCommand("echo", {
  numberOfKeys: 2,
  lua: "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}",
});

// Now `echo` can be used just like any other ordinary command,
// and ioredis will try to use `EVALSHA` internally when possible for better performance.
redis.echo("k1", "k2", "a1", "a2", (err, result) => {
  // result === ['k1', 'k2', 'a1', 'a2']
});

// `echoBuffer` is also defined automatically to return buffers instead of strings:
redis.echoBuffer("k1", "k2", "a1", "a2", (err, result) => {
  // result[0] equals to Buffer.from('k1');
});

// And of course it works with pipeline:
redis.pipeline().set("foo", "bar").echo("k1", "k2", "a1", "a2").exec();
```

If the number of keys can't be determined when defining a command, you can
omit the `numberOfKeys` property and pass the number of keys as the first argument
when you call the command:

```javascript
redis.defineCommand("echoDynamicKeyNumber", {
  lua: "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}",
});

// Now you have to pass the number of keys as the first argument every time
// you invoke the `echoDynamicKeyNumber` command:
redis.echoDynamicKeyNumber(2, "k1", "k2", "a1", "a2", (err, result) => {
  // result === ['k1', 'k2', 'a1', 'a2']
});
```
