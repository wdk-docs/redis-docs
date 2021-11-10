---
title: "Redis+Lua 解决高并发场景抢购秒杀问题"
linkTitle: "Redis+Lua秒杀"
date: " 2021-07-16 17:44"
---

> https://www.cnblogs.com/itbsl/p/15021263.html

之前写了一篇 PHP+Redis 链表解决高并发下商品超卖问题，今天介绍一些如何使用 PHP+Redis+Lua 解决高并发下商品超卖问题。

## 为何要使用 Lua 脚本解决商品超卖的问题呢？

Redis 在 2.6 版本后原生支持 Lua 脚本功能，允许开发者使用 Lua 语言编写脚本传到 Redis 中执行。

将复杂的或者多步的 redis 操作，写为一个脚本，一次提交给 redis 执行，减少反复连接 redis 的次数，提升性能。

原子操作。Redis 会将整个脚本作为一个整体执行，中间不会被其他请求插入。因此在脚本运行过程中无需担心会出现竞态条件，无需使用事务。

复用客户端发送的脚本会永久存在 redis 中，这样其他客户端可以复用这一脚本，而不需要使用代码完成相同的逻辑。

## 编写脚本

### 首先，编写 lua 脚本，脚本名为 secKill.lua：

```lua
-- 接收参数
local user_id = KEYS[1]
local goods_id = KEYS[2]

-- 拼接字符串
local stock_key = "secKill:"..goods_id..":stock" -- 秒杀商品库存 key
local users_key = "secKill:"..goods_id..":users" -- 成功秒杀商品的用户集合 key

-- 判断用户是否已经成功秒杀过该商品，如果已经存在在集合中，说明已经成功秒杀该商品，直接返回标志 2，防止重复抢购
local user_exists = redis.call('sismember', users_key, user_id)
if tonumber(user_exists, 10) == 1 then
    return 2
end

-- 获取当前商品库存，如果库存小于等于 0，表名商品已经被抢购完了，否则库存-1，并将抢购成功的用户放入集合中
local left_goods_count = redis.call('get', stock_key)
if tonumber(left_goods_count, 10) <= 0 then
    return 0
else
    redis.call('decr', stock_key)
    redis.call('sadd', users_key, user_id)
end
return 1
```

上述代码中返回的数字 0，1，2 只是一种约定，自己可以根据自己的有业务约定不同状态返回的值。示例代码 0：库存为 0，1：秒杀成功，2：已秒杀成功的用户重复抢购。

### lua 脚本编写完成后，使用 redis-cli 命令生成该脚本的 sha 秘钥

```sh
redis-cli script load "$(cat /usr/local/redis/lua/secKill.lua)"
"63454a53284d9f6b30bdb6e5e12796a74f61f718"
```

### 最后，拿到 lua 脚本的 sha 秘钥，我们就可以在我们的代码中使用了。

```sh
$redis = new Redis();
$redis->connect("192.168.111.128", 6379);
$goodsId = 11211;
$userId = mt_rand(10000, 99999);
$res = $redis->evalSha('63454a53284d9f6b30bdb6e5e12796a74f61f718', [$userId, $goodsId], 2);
```

可以看到，我们将抢购逻辑写到 lua 脚本后，PHP 代码就变得很少了，仅仅只有 5 行代码。

## 测试

编写好代码，接着我们开始对上述代码进行测试。

首先，我们需要设置商品的库存量，正常逻辑是在后台商品管理页填写具体商品的库存量，
此处假设我们的商品 ID 是 11211，商品数量为 10 个。

```sh
$redis-cli
> set secKill:11211:stock 10
```

我们使用 ab 压测工具模拟 2000 个用户并发量 200 来模拟抢购商品 ID 为 11211 的商品。

```sh
$ ab -n 2000 -c 200 http://www.master.com/index.php
```

如果没有 ab 工具需要使用 `yum -y install httpd-tools` 安装

压测完成后，我们通过 `RedisDesktopManager(RDM)` 软件来查看抢购结果，
可以看到即使是 200 的并发量，最终也只有 10 个用户抢购到商品，并且抢购成功的用户被写入到了 secKill:11211:users 的集合中，
我们可以另外开一个守护进程专门用于从集合中获取用户 ID 处理后续事宜(将数据落盘写入数据库、给用户发短信等)

使用 Redis+Lua 来解决抢购秒杀类问题是当前比较流行的一种做法，希望对正在开发秒杀抢购功能的你能产生帮助。
