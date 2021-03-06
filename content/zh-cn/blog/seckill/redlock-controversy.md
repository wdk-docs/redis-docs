---
title: "Redis Redlock 的争论"
linkTitle: "Redlock 争论"
date: " 2021-06-22"
author: "我妻礼弥"
---

> https://juejin.cn/post/6976538149904678925

Redis 作者提出的 Redlock 方案后，马上受到英国剑桥大学、业界著名的分布式系统专家 [Martin]() 的质疑！
认为这个 Redlock 的算法模型是有问题的，并写了篇文件对分布式锁的设计，提出了自己的看法。

之后，Redis 作者 [Antirez]() 面对质疑，不甘示弱，也写了一篇文章，反驳了对方的观点，并详细剖析了 Redlock 算法模型的更多设计细节。

关于这个问题的争论，在当时互联网上也引起了非常激烈的讨论。双方都是分布式系统领域的专家，却对同一个问题提出很多相反的论断，而且二人思路清晰，论据充分，到底谁对谁错，让我们先来看下分布式专家 **Martin** 对于 Relock 的质疑

## 分布式专家 **Martin** 对于 Relock 的质疑

在他的文章中，主要阐述了 4 个论点：

### 分布式锁的偏好

Martin 表示使用分布式锁有两种偏好

1. **效率**：使用分布式锁的互斥能力，避免多次做重复的工作（例如一些“昂贵”的计算任务）。这种情况要求即使锁失效，也不会带来「恶性」的后果。例如多发了 1 次邮件等无伤大雅的场景。
2. **正确性**：使用锁用来防止并发进程互相干扰。如果锁失效，会造成多个进程同时操作同一条数据，产生的后果是数据严重错误、永久性不一致、数据丢失等恶性问题。

Martin 认为，如果你是为了效率，那么使用单机版 Redis 就可以了，即使偶尔发生锁失效（宕机、主从切换），都不会产生严重的后果。而使用 Redlock 太重了，没必要。

而如果你是为了正确性，Martin 认为 Redlock 根本达不到安全性的要求，也依旧存在锁失效的问题！

### 锁在分布式系统中会遇到的问题

Martin 表示，一个分布式系统，存在着各种异常情况，这些异常场景主要包括三大块，这也是分布式系统会遇到的三座大山：**NPC**。

- N：Network Delay，网络延迟
- P：Process Pause，进程暂停
- C：Clock Drift，时钟漂移

Martin 用一个进程暂停的例子，指出了 Redlock 安全性问题：

1. 客户端 1 请求锁定节点 A、B、C、D、E
1. 客户端 1 的拿到锁后，进入进程暂停（时间比较久）
1. 所有 Redis 节点上的锁都过期了
1. 客户端 2 获取到了 A、B、C、D、E 上的锁
1. 客户端 1 GC 结束，认为成功获取锁
1. 客户端 2 也认为获取到了锁，发生「冲突」

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbe61ac0e3be4f60aee223eb87be35da~tplv-k3u1fbpfcp-watermark.awebp)

Martin 认为，进程暂停可能发生在程序的任意时刻，而且执行时间是不可控的。

> 注：当然，即使没有进程暂停，在发生网络延迟、时钟漂移时，也都有可能导致 Redlock 出现此类问题，这里 **Martin** 只是拿进程暂停举例

### 假设时钟正确是不合理的

Relock 有一个隐含条件是所有的主机时间都是正确的，如果时间不正确就会出问题，例如

1. 客户端 1 获取到节点 A、B、C 上的锁
1. 节点 C 上的时钟「向前跳跃」，导致锁到期
1. 客户端 2 获取节点 C、D、E 上的锁
1. 客户端 1 和 2 现在都相信它们持有了锁（冲突）

Martin 认为 Redlock 必须「强依赖」多个节点的时钟是保持同步的，一旦有节点时钟发生错误，那这个算法模型就失效了。而机器的时钟发生错误，是很有可能发生的，比如：

- 系统管理员「手动修改」了机器时钟
- 机器时钟在同步 NTP 时间时，发生了大的「跳跃」

总之，Martin 认为，Redlock 的算法是建立在「同步模型」基础上的，有大量资料研究表明，同步模型的假设，在分布式系统中是有问题的。在混乱的分布式系统的中，你不能假设系统时钟就是对的，所以，你必须非常小心你的假设。

### 提出 fecing token 的方案，保证正确性

相对应的，Martin 提出一种被叫作 fecing token 的方案，保证分布式锁的正确性。(这里感叹一句，大神就是大神，不光能发现、提出问题，还能给出解决方案)

这个模型流程如下：

1. 客户端在获取锁时，锁服务可以提供一个「递增」的 token
1. 客户端拿着这个 token 去操作共享资源
1. 共享资源可以根据 token 拒绝「后来者」的请求

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95423a7b261a43009d7451f0f21b0913~tplv-k3u1fbpfcp-watermark.awebp)

这样一来，无论 NPC 哪种异常情况发生，都可以保证分布式锁的安全性，因为它是建立在「异步模型」上的。

而 Redlock 无法提供类似 fecing token 的方案，所以它无法保证安全性。

他还表示

> 一个好的分布式锁，无论 NPC 怎么发生，可以不在规定时间内给出结果，但并不会给出一个错误的结果。也就是只会影响到锁的「性能」（或称之为活性），而不会影响它的「正确性」。

### **Martin** 的结论

1. Redlock 不伦不类：对于偏好效率来讲，Redlock 比较重，没必要这么做，而对于偏好正确性来说，Redlock 是不够安全的。
1. 时钟假设不合理：该算法对系统时钟做出了危险的假设（假设多个节点机器时钟都是一致的），如果不满足这些假设，锁就会失效。
1. 无法保证正确性：Redlock 不能提供类似 fencing token 的方案，所以解决不了正确性的问题。为了正确性，请使用有「共识系统」的软件，例如 Zookeeper。

以上就是 **Martin** 反对使用 Redlock 的观点，有理有据。接下来我们来看 Redis 作者 Antirez 是如何反驳的。

## Redis 作者 Antirez 的反驳

在 Redis 作者的反驳文章中，有 3 个重点

### 时钟问题

首先，Redis 作者一眼就看穿了对方提出的最为核心的问题：时钟问题。

> 为什么 Redis 作者优先解释时钟问题？因为在后面的反驳过程中，需要依赖这个基础做进一步解释。

Redis 作者表示，Redlock 并不需要完全一致的时钟，只需要大体一致就可以了，允许有「误差」，只要误差不要超过锁的租期即可，这种对于时钟的精度要求并不是很高，而且这也符合现实环境。

对于对方提到的「时钟修改」问题，Redis 作者反驳到：

- 手动修改时钟：不要这么做就好了，否则你直接修改 Raft 日志，那 Raft 也会无法工作...
- 时钟跳跃：通过「恰当的运维」，保证机器时钟不会大幅度跳跃（每次通过微小的调整来完成），实际上这是可以做到的

### 解释网络延迟、进程暂停问题

Redis 作者对于对方提出的，网络延迟、进程暂停可能导致 Redlock 失效的问题，也做了反驳

我们重新回顾一下，Martin 提出的问题假设：

1. 客户端 1 请求锁定节点 A、B、C、D、E
1. 客户端 1 的拿到锁后，进入进程暂停（时间比较久）
1. 所有 Redis 节点上的锁都过期了
1. 客户端 2 获取到了 A、B、C、D、E 上的锁
1. 客户端 1 GC 结束，认为成功获取锁
1. 客户端 2 也认为获取到了锁，发生「冲突」

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbe61ac0e3be4f60aee223eb87be35da~tplv-k3u1fbpfcp-watermark.awebp)

Redis 作者反驳到，这个假设其实是有问题的，Redlock 是可以保证锁安全的。还记得前面介绍 Redlock 流程的那 5 步吗？让我们来复习一下。

1. 客户端先获取「当前时间戳 T1」
1. 客户端依次向这 5 个 Redis 实例发起加锁请求（用前面讲到的 SET 命令），并设置超时时间（毫秒级），如果某一个实例加锁失败（包括网络超时、锁被其它人持有等各种异常情况），就立即向下一个 Redis 实例申请加锁
1. 如果客户端从 >=3 个（大多数）以上 Redis 实例加锁成功，则再次获取「当前时间戳 T2」，如果锁的租期 > T2 - T1 ，此时，认为客户端加锁成功，否则认为加锁失败
1. 加锁成功，去操作共享资源
1. 加锁失败或操作结束，向「全部节点」发起释放锁请求（前面讲到的 Lua 脚本释放锁）

> 注意，重点是 1-3，在步骤 3，加锁成功后为什么要重新获取「当前时间戳 T2」？还用 T2 - T1 的时间，与锁的过期时间做比较？

Redis 作者强调：如果在 1-3 发生了网络延迟、进程暂停等耗时长的异常情况，那在第 3 步 T2 - T1，是可以检测出来的，如果超出了锁设置的过期时间，那这时就认为加锁会失败，之后释放所有节点的锁就好了！

Redis 作者继续论述，如果对方认为，发生网络延迟、进程暂停是在步骤 3 之后，也就是客户端确认拿到了锁，去操作共享资源的途中发生了问题，导致锁失效，那这不止是 Redlock 的问题，任何其它锁服务例如 Zookeeper，都有类似的问题，这不在讨论范畴内。

所以 Redis 作者的结论是：

- 客户端在拿到锁之前，无论经历什么耗时长问题，Redlock 都能够在第 3 步检测出来
- 客户端在拿到锁之后，发生 NPC，那 Redlock、Zookeeper 都无能为力

所以，Redis 作者认为 Redlock 在保证时钟正确的基础上，是可以保证正确性的。

### 质疑 fencing token 机制

Redis 作者对于对方提出的 **fecing token** 机制，也提出了质疑，主要分为 2 个问题

#### 第一共享资源服务器能拒绝旧 token

这个方案必须要求要操作的「共享资源服务器」有拒绝「旧 token」的能力。

假设共享资源服务器是 MySQL，我们要操作 MySQL，从锁服务拿到一个递增数字的 token，
然后客户端要带着这个 token 去改 MySQL 的某一行，这就需要利用 MySQL 的「事物隔离性」来做。

```sql
// 两个客户端必须利用事物和隔离性达到目的
// 注意 token 的判断条件
UPDATE table T SET val = $new_val WHERE id = $id AND current_token < $token
```

但如果操作的不是 MySQL 而是向磁盘上写一个文件，或发起一个 HTTP 请求，那这个方案就无能为力了，这对要操作的资源服务器，提出了更高的要求。

再者，既然资源服务器都有了「互斥」能力，那还要分布式锁干什么？

所以，Redis 作者认为这个方案是站不住脚的。

#### 第二 Redlock 已经提供了随机值

Redlock “实现” fecing token

退一步讲，即使 Redlock 没有提供 fecing token 的能力，但 Redlock 已经提供了随机值（就是之前讲的 UUID），利用这个随机值，也可以达到与 fecing token 同样的效果。

1. 客户端使用 Redlock 拿到锁
1. 客户端在操作共享资源之前，先把这个锁的 VALUE，在要操作的共享资源上做标记
1. 客户端处理业务逻辑，最后，在修改共享资源时，判断这个标记是否与之前一样，一样才修改（类似 CAS 的思路）

还是以 MySQL 为例,这个实现如下

1. 客户端使用 Redlock 拿到锁
1. 客户端要修改 MySQL 表中的某一行数据之前，先把锁的 VALUE 更新到这一行的某个字段中（这里假设为 current_token 字段)
1. 客户端处理业务逻辑
1. 客户端修改 MySQL 的这一行数据，把 VALUE 当做 WHERE 条件，再修改

```sql
UPDATE table T SET val = $new_val WHERE id = $id AND current_token = $redlock_value
```

可见，这种方案通过依赖 MySQL 的事物机制，也达到对方提到的 fecing token 一样的效果。

### 操作顺序

对于上述通过 Redlock “实现” fecing token 的设计，网友提出了一个问题：两个客户端通过这种方案，先「标记」再「检查 + 修改」共享资源，那这两个客户端的操作顺序无法保证啊？
而用 **Martin** 提到的 fecing token，因为这个 token 是单调递增的数字，资源服务器可以拒绝小的 token 请求，保证了操作的「顺序性」！

Redis 作者对这问题做了不同的解释,我觉得非常有意思，他认为：分布式锁的本质，是为了「互斥」，只要能保证两个客户端在并发时，一个成功，一个失败就好了，不需要关心「顺序性」。

> 前面 **Martin** 的质疑中，一直很关心这个顺序性问题，但 Redis 的作者的看法却不同。

综上，Redis 作者的结论：

1. 作者同意对方关于「时钟跳跃」对 Redlock 的影响，但认为时钟跳跃是可以避免的，取决于基础设施和运维。
2. Redlock 在设计时，充分考虑了 NPC 问题，在 Redlock 步骤 3 之前出现 NPC，可以保证锁的正确性，但在步骤 3 之后发生 NPC，不止是 Redlock 有问题，其它分布式锁服务同样也有问题，所以不在讨论范畴内。

## 基于 Zookeeper 的分布式锁

Martin 在他的文章中，推荐使用 Zookeeper 实现分布式锁，认为它更安全

Zookeeper 又是如何实现的分布式锁的呢？

1. 客户端 1 和 2 都尝试创建「临时节点」，例如 /lock
1. 假设客户端 1 先到达，则加锁成功，客户端 2 加锁失败
1. 客户端 1 操作共享资源
1. 客户端 1 删除 /lock 节点，释放锁

Zookeeper 不像 Redis 那样，需要考虑锁的过期时间问题，它是采用了「临时节点」，保证客户端 1 拿到锁后，只要连接不断，就可以一直持有锁。
如果客户端 1 异常崩溃了，那么这个临时节点会自动删除，保证了锁一定会被释放。

关于 Zookeeper 实现分布式锁的具体详情，参考 [Zookeeper 实现分布式锁](https://juejin.cn/post/7005487404500910116/)

### 保持持有锁

客户端 1 创建临时节点后，Zookeeper 是如何保证让这个客户端一直持有锁呢？

原因就在于，客户端 1 此时会与 Zookeeper 服务器维护一个 Session，这个 Session 会依赖客户端「定时心跳」来维持连接。

如果 Zookeeper 长时间收不到客户端的心跳，就认为这个 Session 过期了，也会把这个临时节点删除。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/078a3fc5d50e437ea339493ddca39bac~tplv-k3u1fbpfcp-watermark.awebp)

### Zookeeper 分布式锁下的 NPC 问题

同样的 Zookeeper 的分布式锁也存在 NPC 问题，以进程暂停为例

1. 客户端 1 创建临时节点 /lock 成功，拿到了锁
1. 客户端 1 发生长时间进程暂停
1. 客户端 1 无法给 Zookeeper 发送心跳，Zookeeper 把临时节点「删除」
1. 客户端 2 创建临时节点 /lock 成功，拿到了锁
1. 客户端 1 进程暂停结束，它仍然认为自己持有锁（冲突）

可见，即使是使用 Zookeeper，也无法保证进程暂停、网络延迟异常场景下的安全性。

这就是前面 Redis 作者在反驳的文章中提到的：如果 进程暂停、网络延迟是发生在客户端拿到锁之后，那不止 Redlock 有问题，其它锁服务都有类似的问题。

这里我们可以得出一个结论：

> 一个分布式锁，在极端情况下，不一定是安全的。

### Zookeeper 分布式锁的优缺点

Zookeeper 的优点：

- 不需要考虑锁的过期时间
- watch 机制，加锁失败，可以 watch 等待锁释放，实现乐观锁

但它的劣势是：

- 性能不如 Redis
- 部署和运维成本高
- 客户端与 Zookeeper 的长时间失联，锁被释放问题

### 到底要不要用 Redlock？

前面也分析了，Redlock 只有建立在「时钟正确」的前提下，才能正常工作，如果你可以保证这个前提，那么可以拿来使用。

但保证时钟正确，并不是简单

- 第一，从硬件角度来说，时钟发生偏移是时有发生，无法避免。例如，CPU 温度、机器负载、芯片材料都是有可能导致时钟发生偏移的。
- 第二，人为错误也是很难完全避免，运维暴力修改时钟，进而影响了系统的正确性

所以，我对 Redlock 的个人看法是，尽量不用它，而且它的性能不如单机版 Redis，部署成本也高，建议优先考虑使用主从 + 哨兵的模式 实现分布式锁

那怎么保证正确性了？

### 正确使用分布式锁

从上面我们也了解到，任何分布式锁都无法完全保证正确性，因此分布式锁时建议

1. 在上层使用分布式锁完成「互斥」目的，虽然极端情况下锁会失效，但它可以最大程度把并发请求阻挡在最上层，减轻操作资源层的压力。
2. 但对于要求数据绝对正确的业务，在资源层一定要做好「兜底」，发生极端情况时，也不会对系统造成影响
