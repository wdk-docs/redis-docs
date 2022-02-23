---
title: "介绍"
linkTitle: ""
weight: 1
---

Bull 是 BullMQ 的遗留版本。由于它在今天仍然被大量使用，所以它也被用于 bug 的维护，而不是用于新的主要功能。
如果你想使用一个经过战斗测试的队列库，你不需要更好的 typescript 集成，也不需要最新的特性，你可以在未来几年使用这个库。

## 所使用的

长期以来，Bull 一直是 NodeJS 生态系统的一部分，许多组织在商业和开源项目中都使用了 Bull。
特别提到几点:

![](</bull/Screenshot 2022-02-15 at 11.32.39 (1).png>)
![](</bull/mozilla-logo-bw-rgb (2).png>)
![](/bull/autodesk-logo-white.png)
![](</bull/Atlassian-horizontal-blue-rgb (1).webp>)
![](/bull/midwayjs-logo.png)![](/bull/salesforce-logo.png)
![](</bull/entethalliance-logo (1).png>)
![](/bull/kisspng-logo-retail-target-corporation-advertising-5ae5ef43944c89.3404142515250184356074.png)

<div align="center">
  <br/>
  <img src="./support/logo@2x.png" width="300" />
  <br/>
  <br/>
  <p>
    The fastest, most reliable, Redis-based queue for Node. <br/>
    Carefully written for rock solid stability and atomicity.
  </p>
  <br/>
  <p>
    <a href="#-sponsors-"><strong>Sponsors</strong></a> ·
    <a href="#bull-features"><strong>Features</strong></a> ·
    <a href="#uis"><strong>UIs</strong></a> ·
    <a href="#install"><strong>Install</strong></a> ·
    <a href="#quick-guide"><strong>Quick Guide</strong></a> ·
    <a href="#documentation"><strong>Documentation</strong></a>
  </p>
  <p>Check the new <a href="https://optimalbits.github.io/bull/"><strong>Guide!</strong></p>
  <br/>
  <p>
    <a href="https://gitter.im/OptimalBits/bull">
      <img src="https://badges.gitter.im/Join%20Chat.svg"/>
    </a>
    <a href="https://gitter.im/OptimalBits/bull">
      <img src="https://img.shields.io/npm/dm/bull.svg?maxAge=2592000"/>
    </a>
    <a href="http://badge.fury.io/js/bull">
      <img src="https://badge.fury.io/js/bull.svg"/>
    </a>
    <a href="https://coveralls.io/github/OptimalBits/bull?branch=master">
      <img src="https://coveralls.io/repos/github/OptimalBits/bull/badge.svg?branch=master"/>
    </a>
    <a href="http://isitmaintained.com/project/OptimalBits/bull">
      <img src="http://isitmaintained.com/badge/open/optimalbits/bull.svg"/>
    </a>
    <a href="http://isitmaintained.com/project/OptimalBits/bull">
      <img src="http://isitmaintained.com/badge/resolution/optimalbits/bull.svg"/>
    </a>
        <a href="https://twitter.com/manast">
      <img src="https://img.shields.io/twitter/follow/manast?label=Stay%20updated&style=social"/>
    </a>
  </p>
</div>

### 📻 新闻和更新

Follow me on [Twitter](http://twitter.com/manast) for important news and updates.

### 🛠 手册

You can find tutorials and news in this blog: https://blog.taskforce.sh/

---

### 被使用

Bull is popular among large and small organizations, like the following ones:

<table cellspacing="0" cellpadding="0">
  <tr>
    <td valign="center">
      <a href="https://github.com/atlassian/github-for-jira">
        <img
          src="https://876297641-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LUuDmt_xXMfG66Rn1GA%2Fuploads%2FevsJCF6F1tx1ScZwDQOd%2FAtlassian-horizontal-blue-rgb.webp?alt=media&token=2fcd0528-e8bb-4bdd-af35-9d20e313d1a8"
          width="150"
          alt="Atlassian"
      /></a>
    </td>
    <td valign="center">
      <a href="https://github.com/Autodesk">
        <img
          src="https://876297641-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LUuDmt_xXMfG66Rn1GA%2Fuploads%2FvpTe02RdOhUJBA8TdHEE%2Fautodesk-logo-white.png?alt=media&token=326961b4-ea4f-4ded-89a4-e05692eec8ee"
          width="150"
          alt="Autodesk"
      /></a>
    </td>
    <td valign="center">
      <a href="https://github.com/common-voice/common-voice">
        <img
          src="https://876297641-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LUuDmt_xXMfG66Rn1GA%2Fuploads%2F4zPSrubNJKViAzUIftIy%2Fmozilla-logo-bw-rgb.png?alt=media&token=9f93aae2-833f-4cc4-8df9-b7fea0ad5cb5"
          width="150"
          alt="Mozilla"
      /></a>
    </td>
    <td valign="center">
      <a href="https://github.com/nestjs/bull">
        <img
          src="https://876297641-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LUuDmt_xXMfG66Rn1GA%2Fuploads%2FfAcGye182utFUtPKdLqJ%2FScreenshot%202022-02-15%20at%2011.32.39.png?alt=media&token=29feb550-f0bc-467d-a290-f700701d7d15"
          width="150"
          alt="Nest"
      /></a>
    </td>
    <td valign="center">
      <a href="https://github.com/salesforce/refocus">
        <img
          src="https://876297641-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-LUuDmt_xXMfG66Rn1GA%2Fuploads%2FZNnYNuL5qJ6ZoBh7JJEW%2Fsalesforce-logo.png?alt=media&token=ddcae63b-08c0-4dd4-8496-3b29a9bf977d"
          width="100"
          alt="Salesforce"
      /></a>
    </td>

  </tr>
</table>

---

### BullMQ

如果你想开始使用完全用 Typescript 编写的下一个主要版本的 Bull，欢迎使用新的 repo[这里](https://github.com/taskforcesh/bullmq)。
否则，我们非常欢迎你仍然使用 Bull，这是一个安全的、经过战斗测试的代码库。

---

### 🚀 贡献 🚀

[<img src="https://www.redisgreen.com/images/rglogo/redisgreen_transparent_240x48.png" width="150" alt="RedisGreen" style="padding: 100px"/>](https://dashboard.redisgreen.net/new?utm_campaign=BULLMQ)

If you need high quality production Redis instances for your Bull projects, please consider subscribing
to [RedisGreen](https://dashboard.redisgreen.net/new?utm_campaign=BULLMQ),
leaders in Redis hosting that works perfectly with Bull. Use the promo code "BULLMQ" when signing up to help us
sponsor the development of Bull!

---

### 官方的前端

[<img src="http://taskforce.sh/assets/logo_square.png" width="100" alt="Taskforce.sh, Inc" style="padding: 100px"/>](https://taskforce.sh)

Supercharge your queues with a professional front end:

- Get a complete overview of all your queues.
- Inspect jobs, search, retry, or promote delayed jobs.
- Metrics and statistics.
- and many more features.

Sign up at [Taskforce.sh](https://taskforce.sh)

---

### Bull 特性

- [x] Minimal CPU usage due to a polling-free design.
- [x] Robust design based on Redis.
- [x] Delayed jobs.
- [x] Schedule and repeat jobs according to a cron specification.
- [x] Rate limiter for jobs.
- [x] Retries.
- [x] Priority.
- [x] Concurrency.
- [x] Pause/resume—globally or locally.
- [x] Multiple job types per queue.
- [x] Threaded (sandboxed) processing functions.
- [x] Automatic recovery from process crashes.

And coming up on the roadmap...

- [ ] Job completion acknowledgement (you can use the message queue [pattern](https://github.com/OptimalBits/bull/blob/develop/PATTERNS.md#returning-job-completions) in the meantime).
- [ ] Parent-child jobs relationships.

---

### UIs

There are a few third-party UIs that you can use for monitoring:

**BullMQ**

- [Taskforce](https://taskforce.sh)

**Bull v3**

- [Taskforce](https://taskforce.sh)
- [bull-board](https://github.com/vcapretz/bull-board)
- [bull-repl](https://github.com/darky/bull-repl)
- [bull-monitor](https://github.com/s-r-x/bull-monitor)
- [Monitoro](https://github.com/AbhilashJN/monitoro)

**Bull <= v2**

- [Matador](https://github.com/ShaneK/Matador)
- [react-bull](https://github.com/kfatehi/react-bull)
- [Toureiro](https://github.com/Epharmix/Toureiro)

---

### 监测和报警

- With Prometheus [Bull Queue Exporter](https://github.com/UpHabit/bull_exporter)

---

### 特征比较

Since there are a few job queue solutions, here is a table comparing them:

| Feature                   |   Bullmq-Pro    |     Bullmq      |      Bull       |  Kue  | Bee      | Agenda |
| :------------------------ | :-------------: | :-------------: | :-------------: | :---: | -------- | ------ |
| Backend                   |      redis      |      redis      |      redis      | redis | redis    | mongo  |
| Observables               |        ✓        |                 |                 |       |          |        |
| Group Rate Limit          |        ✓        |                 |                 |       |          |        |
| Group Support             |        ✓        |                 |                 |       |          |        |
| Parent/Child Dependencies |        ✓        |        ✓        |                 |       |          |        |
| Priorities                |        ✓        |        ✓        |        ✓        |   ✓   |          | ✓      |
| Concurrency               |        ✓        |        ✓        |        ✓        |   ✓   | ✓        | ✓      |
| Delayed jobs              |        ✓        |        ✓        |        ✓        |   ✓   |          | ✓      |
| Global events             |        ✓        |        ✓        |        ✓        |   ✓   |          |        |
| Rate Limiter              |        ✓        |        ✓        |        ✓        |       |          |        |
| Pause/Resume              |        ✓        |        ✓        |        ✓        |   ✓   |          |        |
| Sandboxed worker          |        ✓        |        ✓        |        ✓        |       |          |        |
| Repeatable jobs           |        ✓        |        ✓        |        ✓        |       |          | ✓      |
| Atomic ops                |        ✓        |        ✓        |        ✓        |       | ✓        |        |
| Persistence               |        ✓        |        ✓        |        ✓        |   ✓   | ✓        | ✓      |
| UI                        |        ✓        |        ✓        |        ✓        |   ✓   |          | ✓      |
| Optimized for             | Jobs / Messages | Jobs / Messages | Jobs / Messages | Jobs  | Messages | Jobs   |

### 安装

```bash
npm install bull --save
```

or

```bash
yarn add bull
```

_**Requirements:** Bull requires a Redis version greater than or equal to `2.8.18`._

### Typescript 定义

```bash
npm install @types/bull --save-dev
```

```bash
yarn add --dev @types/bull
```

Definitions are currently maintained in the [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/bull) repo.

---

### 贡献

We welcome all types of contributions, either code fixes, new features or doc improvements.
Code formatting is enforced by [prettier](https://prettier.io/).
For commits please follow conventional [commits convention](https://www.conventionalcommits.org/en/v1.0.0-beta.2/).
All code must pass lint rules and test suites before it can be merged into develop.

---
