---
title: "Implementation of redis distributed lock"
linkTitle: ""
date: "2021-01-08"
author: "developpaper"
---

> https://developpaper.com/implementation-of-redis-distributed-lock/

Many novices will Distributed lock and Distributed transactionConfusion, personal understanding:lock
It is used to solve the concurrent contention for a shared resource by multiple programs;affair
It is used to ensure the consistency of a series of operations.
I’ve explained distributed transactions in several previous articles.
I’m going to talk about the implementation of 2pc, TCC and asynchronous assurance schemes.

## 1. 定义

In the traditional monomer architecture, the most common lock is JDK lock.
Because the thread is the smallest unit that the operating system can run and schedule,
in Java multithreading development, it is inevitable that different threads compete for resources in the same process.
JDK library provides us with synchronized, lock and update java.util.concurrent . \* etc.
However, they all have unified restrictions. Threads competing for resources are all running in the same JVM process.
In the distributed architecture, different JVM processes cannot use the lock.

In order to prevent multiple processes from interfering with each other in a distributed system,
we need a distributed coordination technology to schedule these processes.
And the core of this distributed coordination technology is to achieve thisDistributed lock。

Take a classic example of “oversold”.
In an e-commerce project, the logic of the interface can be simply divided into:

1. Query whether the inventory is greater than zero;
2. When the inventory is greater than zero, purchase the goods.

When there is only one piece in stock, both user a and user B execute the first step at the same time,
query that the inventory is one piece, and then both execute the purchase operation.
When they finished the purchase,they found that the inventory was – 1 piece.
We can lock the operations of “inventory query” and “inventory reduction” in Java code to ensure that the requests of users a and B cannot be executed concurrently.
But if our interface service is a cluster service, and the requests of user a and user B are respectively forwarded to different JVM processes by load balancing, it will not solve the problem.

## 2. 分布式锁的比较

From the previous examples, we can see that the resource to coordinate and solve distributed locks must not be a resource at the level of the JVM process, but a shared external resource.

Three ways of implementation

There are three common ways to implement distributed lock: 1. Database lock; 2. Zookeeper based distributed lock; 3. Redis based distributed lock.

1. **Database lock**:
   This way is easy to think of, put the competing resources into the database,
   and use the database lock to realize the resource competition.
   Please refer to the previous articleDatabase transactions and locks。
   For example:
   1. Pessimistic lock implementation: the “for update” can be added to the SQL query of inventory goods to achieve exclusive lock,
      and the “inventory query” and “inventory reduction” can be packaged as a transaction commit.
      Before the completion of user a’s query and purchase, user B’s request will be blocked.
   2. Optimistic lock implementation: add the version number field in the inventory table to control.
      Or more simply, when the inventory is less than zero after each purchase, the transaction can be rolled back.
2. **Distributed lock of zookeeper**:
   zookeeper is professional in implementing distributed locks.
   It is similar to a file system. It plays the role of distributed lock by competing for file resources on the file system.
   Specific implementation, please refer to the previous articleDevelopment and application of zookeeper。
3. **Distributed lock of redis**:
   the previous article talked about the development,
   application and transaction of redis, but never about the distributed lock of redis,
   which is also the core content of this article. In short,
   throughsetnxThe value of the competing key.
   “Database lock” competes for table level resources or row level resources,
   “zookeeper lock” competes for file resources, “redis lock” competes for key value resources.
   They all implement distributed locks by competing for shared resources outside the program.

### 对比

However, in the field of distributed locks, zookeeper is more professional.
In essence, redis is also a database.
All the other two schemes are “part-time” to implement distributed locking, and the effect is not as good as zookeeper.

- Low performance consumption: when concurrent lock competition really occurs,
  the implementation of database or redis basically obtains locks by blocking or constantly retrying,
  which has a certain performance consumption. The zookeeper lock registers the listener.
  When a program releases the lock, the next program gets the lock after listening to the message.
- Perfect lock release mechanism: if the client from which redis obtains the lock is bugged or hung up,
  it can only release the lock after the timeout; while for ZK,
  because the temporary znode is created, as long as the client hangs up, the znode will be gone,
  and the lock will be released automatically.
- Strong consistency of cluster: as we all know,
  zookeeper is a typical case of implementing CP transaction,
  and the transaction request is always handled by the leader node in the cluster.
  Redis actually implements AP transactions. If the master node fails and the master-slave switch occurs,
  the lock may be lost.

### 锁的必要条件

In addition, in order to ensure the availability of distributed locks,
we should at least ensure that the implementation of locks meets the following conditions at the same time:

- Mutual exclusion. At any time, only one client can hold the lock.
- There is no deadlock. Even if one client crashes while holding the lock and does not unlock it,
  it can ensure that other clients can lock.
- It is necessary to tie the bell. Locking and unlocking must be the same client.
  The client can’t unlock the lock added by others.

## 3. Redis 实现分布式锁

### 3.1. 锁

Correct locking

```java
public class RedisTool {

    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";

    /**
     *Attempt to acquire distributed lock
     *@ param jedis redis client
     *@ param lockkey lock
     *@ param requestid request ID
     *@ param expireTime
     *Is @ return successful
     */
    public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {

        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);

        if (LOCK_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }
}
```

To see, we lock a line of code:jedis.set(String key, String value, String nxxx, String expx, int time)This set () method has five formal parameters:

- key: we use the key as the lock, because the key is unique.
- value: we are passing a request ID. many children’s shoes may not understand it. It’s enough to have a key as a lock. Why use value? The reason is that when we talked about reliability above, the distributed lock needs to satisfy the fourth condition, and it needs to be released by the ringer. By assigning value to requestid, we can know which request the lock was added, and we can have a basis when unlocking. The requestid can be used UUID.randomUUID (). Tostring() method.
- Nxxx: for this parameter, we fill in NX, which means set if not exist. That is, when the key does not exist, we perform set operation; if the key already exists, we do not perform any operation;
- EXPX: this parameter is Px, which means that we need to add an expired setting to the key. The specific time is determined by the fifth parameter.
- time: corresponds to the fourth parameter, representing the expiration time of the key.

In general, executing the above set () method will only result in two results:

If there is no lock at present (the key does not exist), lock operation will be performed, and a validity period will be set for the lock. At the same time, value indicates the locked client.
There is a lock, no operation.
Not recommended locking method (not recommended!)

I’ve seen many blogs that use the following method to lock, that is, the combination of setnx and GetSet to manually maintain the key expiration time.

```java
public static boolean wrongGetLock2(Jedis jedis, String lockKey, int expireTime) {

    long expires = System.currentTimeMillis() + expireTime;
    String expiresStr = String.valueOf(expires);

    //If the current lock does not exist, return to lock success
    if (jedis.setnx(lockKey, expiresStr) == 1) {
        return true;
    }

    //If the lock exists, get the expiration time of the lock
    String currentValueStr = jedis.get(lockKey);
    if (currentValueStr != null && Long.parseLong(currentValueStr) < System.currentTimeMillis()) {
        //The lock has expired. Get the expiration time of the previous lock and set the expiration time of the current lock
        String oldValueStr = jedis.getSet(lockKey, expiresStr);
        if (oldValueStr != null && oldValueStr.equals(currentValueStr)) {
            //Considering the concurrency of multiple threads, only one thread has the right to lock if its setting value is the same as the current value
            return true;
        }
    }
    //In other cases, the locking failure will be returned
    return false;
}
```

On the surface, this code also implements distributed locking, and the code logic is similar to the above, but there are several problems:

Since the expiration time is generated by the client itself, it is mandatory that the time of each client must be synchronized in the distributed environment.
When the lock expires, if multiple clients execute it at the same time jedis.getSet () method, although only one client can lock, the expiration time of this client’s lock may be covered by other clients.
The lock does not have the owner ID, that is, any client can be unlocked.
This kind of code on the Internet may be based on the earlier versions of jedis, which had great limitations at that time. Redis 2.6.12 and above adds optional parameters to the set instruction, as mentioned abovejedis.set(String key, String value, String nxxx, String expx, int time)API, you can put theSETNXandEXPIREPackage together to execute, and release the expired key to the redis server to manage. Therefore, the actual development process, we do not use this more primitive way of locking.

### 3.2. 解锁

Correct locking

```java
public class RedisTool {

    private static final Long RELEASE_SUCCESS = 1L;

    /**
     *Release distributed lock
     *@ param jedis redis client
     *@ param lockkey lock
     *@ param requestid request ID
     *@ return is released successfully
     */
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {

        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }
}
```

First, get the value value corresponding to the lock, check whether it is equal to the requestid, if it is equal, delete the lock (unlock). So why use Lua? Because to ensure that the operation is atomic. BeforeThread model and transaction of redisIn this paper, we ensure the atomicity of a series of operation instructions through transaction, and Lua script can also achieve similar effect.

Why atomic? If a request obtains the value corresponding to the lock and verifies that the requestid is equal, it will issue a delete instruction. But because of the network and other reasons, the deletion instruction is blocked. At this time, the lock is automatically unlocked because of timeout, and B requests to acquire the lock and re lock it. At this time, a requests the deletion instruction to be executed. As a result, the lock obtained by B requests is deleted.

### 3.3. lua

The computing power of redis command is not very powerful, and Lua language can make up for the deficiency of redis to a great extent. In redis, the execution of Lua language is atomic, that is to say, when redis executes Lua, it will not be interrupted and has atomicity. This feature helps redis to support the consistency of concurrent data.

Redis supports two ways to run scripts, one is to input some Lua language program code directly, the other is to write Lua language into a file. In practical application, some simple scripts can take the first way, and generally use the second way for those with certain logic. For simple scripts, redis supports caching scripts, but it will use SHA-1 algorithm to sign the script, and then return the SHA-1 ID, as long as it runs through this ID.

#### Executing Lua in redis

Here is a brief introduction. The way to directly input some Lua program code can be performed in redis cli as follows:

```lua
eval lua-script key-num [key1 key2 key3 ....] [value1 value2 value3 ....]

--Example 1
eval "return 'Hello World'" 0
--Example 2
eval "redis.call('set',KEYS[1],ARGV[1])" 1 lua-key lua-value
```

- evalRepresents the execution of Lua language commands.
- lua-scriptRepresents Lua language script.
- key-numIt indicates how many keys there are in the parameter. It should be noted that the key in redis starts from 1. If there is no key parameter, write 0.
- [key1 key2 key3…]Key is passed to Lua as a parameter, which can be left blank, but it needs to be corresponding to the number of key num.
- [value1 value2 value3 …]These parameters are passed to Lua language. They can be filled in or not.

#### Calling redis in Lua

Using in Lua redis.call Do the following:

```lua
redis.call(command,key[param1, param2…])

--Example 1
eval "return redis.call('set','foo','bar')" 0
--Example 2
eval "return redis.call('set',KEYS[1],'bar')" 1 foo
```

- commandIs the command, including set, get, del and so on.
- keyIs the key to be operated.
- param1,param2…Represents the parameter given to the key.

For example, implement a Lua script for GetSet

getset.lua

```lua
local key = KEYS[1]
local newValue = ARGV[1]
local oldValue = redis.call('get', key)
redis.call('set', key, newValue)
return oldValue
```

### 3.4. 限制和改善

As we said earlier, there are some limitations in the implementation of distributed lock in redis cluster. It is difficult to ensure consistency when the master and slave are replaced.

#### phenomenon

In the redis sentinel cluster, we have multiple redis, and there is a master-slave relationship between them, such as one master and two slaves. The data corresponding to our set command is written to the master library, and then synchronized to the slave library. When we apply for a lock, the corresponding command is setnx MyKey myvalue. In the redis sentinel cluster, this command first falls into the main library. Suppose that the master database is down and the data has not yet been synchronized to the slave database, sentinel will elect the master database from one of the slave databases. At this time, there is no MyKey data in our new main library. If another client executes setnx MyKey hisvalue, it will also succeed, that is, it will also get the lock. This means that two clients have acquired the lock. This is not what we want to see. Although the record of this situation is very small, it will only happen when the master-slave fails over. In most cases, most systems can tolerate this flaw, but not all systems can tolerate it.

#### solve

In order to solve the defect in the case of fail over, anterez inventedRedlock algorithm。 Using redlock algorithm, multiple redis instances are needed. When locking, it will send setex MyKey myvalue command to most nodes, as long asIf more than half of the nodes are successful, then the locking is successful。 This is very similar to the zookeeper implementation,When the leader of zookeeper cluster broadcasts the command, more than half of the followers must feed back ack to the leader before it takes effect。

In practical work, we can choose the existing open source implementation, which is redlock py in Python and redlock py in JavaRedisson redlock。

Redlock does solve the “unreliable situation” mentioned above. However, while it solves the problem, it also brings the cost. You need multiple redis instances, you need to introduce new libraries, you need to adjust the code, and the performance will be damaged. Therefore, it is true that there is no “perfect solution”. What we need more is to be able to solve the problem according to the actual situation and conditions.

## 4. “Oversold” 示例代码

We simulate a scene of goods rush buying: a commodity has 100 pieces in stock,
and 200 people rush to buy at the same time.
Compare the rush buying situation with lock and without lock.

### 4.1. 代码

The following is the code of this demo. ORM uses JPA, so the code of Dao layer and POJO is not written in this article. There is only one interface in the controller layer, which selects whether to use the lock by passing parameters.

### Table structure

Commodity listproduct

```sql
CREATE TABLE `product` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `amount` int DEFAULT NULL,
  PRIMARY KEY (`id`)
)
```

Purchase recordpurchase_history

```sql
CREATE TABLE `purchase_history` (
  `id` int NOT NULL AUTO_INCREMENT,
  `product_name` varchar(255) DEFAULT NULL,
  `purchaser` varchar(255) DEFAULT NULL,
  `purchase_time` datetime DEFAULT NULL,
  `amount` int DEFAULT NULL,
  PRIMARY KEY (`id`)
)
```

pom.xml

```xml
<dependencies>
    <!--web-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--redis-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <!--lombok-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <!--jpa-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!-- test-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```

application.yml

```yml
server:
  port: 8001
spring:
  redis:
    host: localhost
    port: 6379
 datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/demo
    username: root
    password: password
```

ProductController.java

```java
@RestController
@RequestMapping("/product")
public class ProductController {
    public final static String PRODUCT_APPLE = "apple";
    private final BuyService buyService;

    public ProductController(BuyService buyService) {
        this.buyService = buyService;
    }

    /**
     * Purchase goods Does @ param lock have a lock Y / N
     */
    @GetMapping("/buy")
    public void buy(@RequestParam(value = "lock", required = false) String lock) throws Exception {
        if ("Y".equals(lock)) {
            buyService.buyProductWithLock(PRODUCT_APPLE);
        } else {
            buyService.buyProduct(PRODUCT_APPLE);
        }
    }
}
```

BuyService.java

```java
@Service
public class BuyService {
    private final ProductDao productDao;
    private final PurchaseHistoryDao purchaseHistoryDao;
    private final LockService lockService;

    public BuyService(ProductDao productDao, PurchaseHistoryDao purchaseHistoryDao, LockService lockService) {
        this.productDao = productDao;
        this.purchaseHistoryDao = purchaseHistoryDao;
        this.lockService = lockService;
    }

    /**
     * Purchase: no lock
     *
     * @param productName
     */
    public void buyProduct(String productName) {
        Product product = productDao.findOneByName(productName);
        if (product.getAmount() > 0) {
            // Inventory minus 1
            product.setAmount(product.getAmount() - 1);
            productDao.save(product);
            // Log
            PurchaseHistory purchaseHistory = new PurchaseHistory();
            purchaseHistory.setProductName(productName);
            purchaseHistory.setAmount(1);
            purchaseHistoryDao.save(purchaseHistory);
        }
    }

    /**
     * Purchase: lock
     *
     * @param productName
     */
    public void buyProductWithLock(String productName) throws Exception {
        String uuid = UUID.randomUUID().toString();
        // Lock
        while (true) {
            if (lockService.lock(productName, uuid)) {
                break;
            }
            Thread.sleep(100);
        }
        Product product = productDao.findOneByName(productName);
        if (product.getAmount() > 0) {
            // Inventory minus 1
            product.setAmount(product.getAmount() - 1);
            productDao.save(product);
            // Log
            PurchaseHistory purchaseHistory = new PurchaseHistory();
            purchaseHistory.setProductName(productName);
            purchaseHistory.setAmount(1);
            purchaseHistoryDao.save(purchaseHistory);
        }
        lockService.unlock(productName, uuid);
    }
}
```

LockService.java

```java
@Service
public class LockService {
    private final StringRedisTemplate stringRedisTemplate;

    public LockService(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    /**
     * Lock
     *
     * @param lockKey
     * @param requestId
     * @return
     */
    public boolean lock(String lockKey, String requestId) {
        return stringRedisTemplate.opsForValue().setIfAbsent(lockKey, requestId, Duration.ofSeconds(3));
    }

    /**
     * Unlocking
     *
     * @param lockKey
     * @param requestId
     */
    public void unlock(String lockKey, String requestId) {
        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setResultType(Long.class);
        script.setScriptText(
                "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end");
        stringRedisTemplate.execute(script, Collections.singletonList(lockKey), requestId);
    }
}
```

### 4.2. 测试和分析

In the test case, the initial inventory of the product is 100 copies, and 200 users are simulated to buy it. According to the principle, only 100 copies can be purchased in the end.

If we want to simulate and test the difference between “locked” and “unlocked”, we must create the condition of “high parallelism”. Here, we should remember that it is “high parallelism”, not “high concurrency”. Because the purchase interface here is fast in response to a single request, in order to simulate the situation that multiple users call the interface at the same time, we should use multithreading locally.

I’ve tried a lot to simulate highly parallel environments. The first time is to write JUnit test cases, starting 200 threads to call the interface. The result is that the service will hang up as soon as there are many threads. Although the reason is not clear, JUnit should not use it in this way. The second time is to use postman to do concurrent interface test, but it is found that whether postman is a fake concurrent test or a single thread calls the interface 200 times in turn, there is no implementation of multithreading. Finally, JMeter was installed, which met my expectations.

When we start 200 threads to call the lock free and lock interface respectively, the test results are as follows:

contrast No lock It’s locked
Surplus stock 68 0
Purchase records 200 100
As you can see, there will be oversold in the case of no lock. Let’s take a look at the purchase code of no lock.

```java
/**
 *Purchase: no lock
 * @param productName
 */
 public synchronized void buyProduct(String productName) {
    Product product = productDao.findOneByName(productName);
    if (product.getAmount() > 0) {
        //Inventory minus 1
        product.setAmount(product.getAmount() - 1);
        productDao.save(product);
        //Log
        PurchaseHistory purchaseHistory = new PurchaseHistory();
        purchaseHistory.setProductName(productName);
        purchaseHistory.setAmount(1);
        purchaseHistoryDao.save(purchaseHistory);
    }
}
```

When multiple requests are called at the same timeproductDao.findOneByName(productName);
The result is the same. They all think that there is still inventory.
If they go to buy, the problem of oversold will appear.
To solve this problem, if it is a single process, through thesynchronized And so on.
If it is distributed multi node, we can consider the way of this paper and use redis distributed lock implementation.
