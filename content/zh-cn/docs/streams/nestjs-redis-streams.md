# @tamimaj/nestjs-redis-streams

> https://github.com/tamimaj/nestjs-redis-streams

![Nest Logo](https://camo.githubusercontent.com/c704e8013883cc3a04c7657e656fe30be5b188145d759a6aaff441658c5ffae0/68747470733a2f2f6e6573746a732e636f6d2f696d672f6c6f676f5f746578742e737667){ width="300" }

使用 ioredis 库的 NestJS 的 Redis 流传输策略。

!!! NOTICE

    本库可以作为订阅者在NestJS微服务中使用。
    然而，该策略的客户端还没有实现。
    您仍然可以使用[liaoliaots/nestjs-redis](https://github.com/liaoliaots/nestjs-redis)等任何客户机模块作为XADD流的发布器。

## 特性

- 用 TypeScript 编码。
- 简单的方法来听流。
  把你的处理器插入你的控制器，你的流消息就会在那里着陆。
  底层使用来自 Redis 的 XREADGROUP 命令。
- 自动消费组为您的流创建，在启动之前开始收听。
- 简单的方法响应一个流(或多个流)。
- 自动 XACK 和入站消息 id 跟踪。
  库允许您回复然后确认，或直接确认。
- 内置序列化和反序列化。
- 自定义可插拔的序列化和反序列化。

## 安装

with npm

```sh
npm install --save @tamimaj/nestjs-redis-streams
```

with yarn

```sh
yarn add @tamimaj/nestjs-redis-streams
```

## 如何使用?

### 在你的`main.ts`中。像这样初始化自定义策略:

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { RedisStreamStrategy } from "@tamimaj/nestjs-redis-streams";

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    strategy: new RedisStreamStrategy({
      // optional. All ioredis options + url.
      connection: {
        url: "0.0.0.0:6379",
        // host: 'localhost',
        // port: 6379,
        // password: '123456',
        // etc...
      },
      // mandatory.
      streams: {
        block: 5000,
        consumer: "users-1",
        consumerGroup: "users",
      },
      // optional. See our example main.ts file for more details...
      // serialization: {},
    }),
  });

  await app.listen();
}
bootstrap();
```

### 在其中一个控制器中，您要处理来自流的消息。

使用我们的装饰器`@RedisStreamHandler("users-1")`告诉库注册这个处理程序并监听`users-1`流，每当它接收到消息时，这个处理程序将与数据和创建的消息上下文一起被调用。

```ts
import { Ctx, MessagePattern, Payload } from "@nestjs/microservices";
import { RedisStreamHandler, StreamResponse, RedisStreamContext } from "@tamimaj/nestjs-redis-streams";

export class UsersEventHandlers {
  @RedisStreamHandler("users:create") // stream name.
  async handleUserCreate(@Payload() data: any, @Ctx() ctx: RedisStreamContext) {
    console.log("Handler users:create called with payload: ", data);
    console.log("Headers: ", ctx.getMessageHeaders());

    return [
      {
        payload: {
          // optional headers to override or add new headers keys.
          // everything except data is considered headers for our serialization.
          correlationId: "THE BEST CORRELATION ID EVER",
          extraKey: "Whatever1234",

          // data is the only mandatory key. for our serializer/deserializer.
          data: { name: "Tamim", lastName: "Abbas" },
        },

        stream: "user:created",
      },
    ] as StreamResponse;

    // return [] as StreamResponse;

    // return null;
  }
}
```

### 你从你的处理器返回的东西告诉库做什么:

- 如果您不返回任何东西或返回 `null`:库将不会发布任何流，也不会确认接收到的流消息。
- 如果返回空数组:库将只确认接收到的流消息。
- 如果您返回一个或多个有效负载的数组:库将以流的形式发布这些有效负载，然后将接收到的流消息进行确认。

### 默认的序列化/反序列化是如何工作的?

我们已经设计了序列化/反序列化逻辑，使其对企业微服务体系结构有用。
我们记住了头和元数据的用例，用于身份验证令牌，或者唯一地跟踪来自日志服务(如 datdog)的消息。
因此，我们将信息设计成两部分。
头文件部分和数据部分。

消息的头部分只是没有任何序列化存储的键/值字符串。
这是为了更好地搜索日志服务中的 id。

数据部分是一个单键`data`，它有一个对象作为值，你可以在其中存储任何你喜欢的数据。类似于 post 请求的主体。
该数据值被 JSON 字符串化并存储在流消息中。
并且，当我们接收到消息时，我们的反序列化器 JSON 解析它并将它转发给处理程序。

### 库流和序列化/反序列化的步骤。

1. 在监听时收到消息。
2. 创建一个上下文，其中存储入站消息的 id、使用者组、使用者和流名称。我们称之为 `inboundContext`。
3. 原始消息和入站上下文被转发到我们的反序列化器或您的自定义反序列化器。
4. 我们的反序列化器接受这些键/值，并将除“data”键外的所有内容都视为头文件。
5. 反序列化器通过调用 `inboundContext.setMessageHeaders(headers)`将所有的报头存储在入站上下文中;
6. 然后反序列化器解析`data`值的字符串化 JSON 并返回它。我们称之为有效载荷。
7. 现在，有效负载到达从反序列化器返回的相应流处理程序。
8. 流处理程序可以访问负载+入站上下文(如果您需要读取存储的报头、消费者组、入站消息 id 等)。
9. 处理程序应该执行一些业务逻辑，然后返回:
   1. 如果处理程序返回 `null` 或不返回任何东西，流将在这里结束。没有确认将发生，也没有任何流将发布作为回应。
   2. 如果处理程序返回空数组，库将只确认入站消息，而不会发布任何流作为返回响应。
   3. 如果处理程序返回一个或多个有效负载的数组，库将发布这些流，然后确认入站消息。 继续下面的流程…
10. 处理程序返回一个或多个有效负载的数组。
11. 现在，每个有效负载对象都被传递给我们的序列化器或您的带有入站上下文的序列化器。
12. 我们的序列化器接受有效负载对象并提取数据键，并将任何其他键视为标头。这些标头覆盖保存在入站上下文中的标头或扩展它们。
13. 序列化程序将入站上下文的报头与处理程序返回的任何可选报头合并。
14. 序列化器将对数据键的对象进行字符串化，并使其随时准备好。
15. 然后，序列化器将字符串化所有的头键/值，并使所有的内容在 Redis Stream 接受的格式，即`[headersKey1, headersValue1, key2, value2，…]， data, stringifiedJSON]`。
16. 将准备好的数组返回到库。
17. 库将通过来自 Redis 的 `XADD` 命令将每个有效载荷发布到相应的流中。
18. 然后，将从 Redis 通过 `XACK` 命令确认入站消息。
19. 流的结束。回去听……

检查我们的示例，了解如何读取处理程序中的数据和上下文以及返回的有效负载的语法。

### 使用自定义序列化器/反序列化器?

我们在上面提到的流中定义了孔来使用您的自定义序列化器/反序列化器。
你可以在`main.ts`文件中初始化策略时提供它们。
使用传递给构造函数的选项的键:`serialization: {serializer, deserializer}`

反序列化器接收两个参数，从 Redis 接收的行消息和入站上下文，这样您就可以将标题存储在那里。

序列化器接收两个参数，从流处理程序返回的有效负载和入站上下文，以便从中提取消息头并在发布响应消息之前将它们附加回响应消息。

检查我们的例子 `main.ts` 文件，我们已经评论了一些锅炉板使用自定义序列化。
