---
title: Redisson 框架中的分布式锁
date: 2023/06/01
updated: 2023/06/02
categories:
  - coding
tags:
  - redisson
  - 分布式锁
  - 技术文章
abbrlink: 46453
---


实现分布式锁通常有三种方式：数据库、Redis 和 Zookeeper。我们比较常用的是通过 Redis 和 Zookeeper 实现分布式锁。Redisson 框架中封装了通过 Redis 实现的分布式锁，下面分析一下它的具体实现
# 1、关键点

1. 原子性：要么都成功，要么都失败
2. 过期时间：如果锁还没来得及释放就遇到了服务宕机，就会出现死锁的问题。给 Redis 的 key 设置过期时间，即使服务宕机了超过设置的过期时间锁会自动进行释放。
3. 锁续期： 因为给锁设置了过期时间而我们的业务逻辑具体要执行多长时间可能是变化和不确定的，如果设定了一个固定的过期时间，可能会导致业务逻辑还没有执行完，锁被释放了的问题。锁续期能保证锁是在业务逻辑执行完才被释放。
4. 正确释放锁： 保证释放自己持有的锁，不能出现 A 释放了 B 持有锁的情况。

# 2、实现

## 2.1、引入依赖

```xml
<!--  pom.xml文件-->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.17.7</version>
</dependency>

```

版本依赖：

|redisson-spring-data module name|Spring Boot version|
|---|---|
|redisson-spring-data-16|1.3.y|
|redisson-spring-data-17|1.4.y|
|redisson-spring-data-18|1.5.y|
|redisson-spring-data-2x|2.x.y|
|redisson-spring-data-3x|3.x.y|

## 2.2、配置yml

```yml
spring:
  redis:
    redisson:
      config:
        singleServerConfig:
          address: redis://127.0.0.1:6379
          database: 0
          password: null
          timeout: 3000
```

## 2.3、代码中使用

```java
@Service
public class Lock {
    @Resource
    private RedissonClient redissonClient;

    public void lock() {
        // 写入redis的key值
        String lockKey = "lock-test";
        // 获取一个Rlock锁对象
        RLock lock = redissonClient.getLock(lockKey);
        // 获取锁，并为其设置过期时间为10s
        lock.lock(10, TimeUnit.SECONDS);
        try {
            // 执行业务逻辑....
            System.out.println("获取锁成功！");
        } finally {
            // 释放锁
            lock.unlock();
            System.out.println("释放锁成功！");
        }
    }

}
```


# 3、底层剖析

## 3.1、lock()

```java
    <T> RFuture<T> tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
        return commandExecutor.syncedEval(getRawName(), LongCodec.INSTANCE, command,
                "if ((redis.call('exists', KEYS[1]) == 0) " +
                       "or (redis.call('hexists', KEYS[1], ARGV[2]) == 1)) then " +
                        "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                        "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                        "return nil; " +
                    "end; " +
                    "return redis.call('pttl', KEYS[1]);",
                Collections.singletonList(getRawName()), unit.toMillis(leaseTime), getLockName(threadId));
    }
```


- `RFuture<T>`：表示返回一个异步结果对象，其中泛型参数 T 表示结果的类型。
- `tryLockInnerAsync` 方法接受以下参数：
    - `waitTime`：等待时间，用于指定在获取锁时的最大等待时间。
    - `leaseTime`：租约时间，用于指定锁的持有时间
    - `unit`：时间单位，用于将 leaseTime 转换为毫秒
    - `threadId`：线程 ID，用于标识当前线程
    - `command`：Redis 命令对象，用于执行 Redis 操作
- 方法体中的代码使用 Lua 脚本来实现分布式锁的逻辑。

    > - if ((redis.call('exists', KEYS[1]) == 0) or (redis.call('hexists', KEYS[1], ARGV[2]) == 1)): 如果键不存在或者哈希表中已经存在对应的线程ID，则执行以下操作：
    >     - redis.call('hincrby', KEYS[1], ARGV[2], 1): 将哈希表中对应线程ID的值加1。
    >     - redis.call('pexpire', KEYS[1], ARGV[1]): 设置键的过期时间为租约时间。
    >     - return nil: 返回nil表示成功获取锁。
    > - else: 如果键存在且哈希表中不存在对应的线程ID，则执行以下操作：    
    >     - return redis.call('pttl', KEYS[1]): 返回键的剩余生存时间。
    
- `commandExecutor.syncedEval`：表示同步执行 Redis 命令
- `LongCodec.INSTANCE`：用于编码和解码长整型数据
- `Collections.singletonList(getRawName())`：创建一个只包含一个元素的列表，元素为锁的名称
- `unit.toMillis(leaseTime)`：将租约时间转换为毫秒
- `getLockName(threadId)`：根据线程 ID 生成锁的名称


```java
// 省去了那些无关重要的代码
private void lock(long leaseTime, TimeUnit unit, boolean interruptibly) throws InterruptedException {
    long threadId = Thread.currentThread().getId();
    // tryAcquire就是上面分析的lua完整脚本
    Long ttl = tryAcquire(-1, leaseTime, unit, threadId);
    // 返回null就代表上锁成功。
    if (ttl == null) {
        return;
    }
    // 如果没成功，也就是锁的剩余时间不是null的话，那么就执行下面的逻辑
    // 其实就是说 如果有锁（锁剩余时间不是null），那就死循环等待重新抢锁。
    try {
        while (true) {
            // 重新抢锁
            ttl = tryAcquire(-1, leaseTime, unit, threadId);
            // 抢锁成功就break退出循环
            if (ttl == null) {
                break;
            }
            // 省略一些代码
        }
    } finally {}
}
```


上面代码实现了一个分布式锁的功能。它使用了Lua脚本来尝试获取锁，并在成功获取锁后返回锁的剩余时间（ttl）。如果获取锁失败，则进入一个死循环，不断尝试重新获取锁，直到成功为止。

## 3.2、unlock()

关键代码

```java
    protected RFuture<Boolean> unlockInnerAsync(long threadId) {
        return evalWriteAsync(getRawName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
              "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                        "return nil;" +
                    "end; " +
                    "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
                    "if (counter > 0) then " +
                        "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                        "return 0; " +
                    "else " +
                        "redis.call('del', KEYS[1]); " +
                        "redis.call(ARGV[4], KEYS[2], ARGV[1]); " +
                        "return 1; " +
                    "end; " +
                    "return nil;",
                Arrays.asList(getRawName(), getChannelName()),
                LockPubSub.UNLOCK_MESSAGE, internalLockLeaseTime, getLockName(threadId), getSubscribeService().getPublishCommand());
    }
```

- `RFuture<Boolean>`: 表示返回一个异步结果对象，其中泛型参数Boolean表示结果的类型。
- `unlockInnerAsync`方法接受以下参数：
    - `threadId`: 线程ID，用于标识当前线程。
- 方法体中的代码使用Lua脚本来实现分布式锁的解锁逻辑。以下是对Lua脚本的解释：
    - `if (redis.call('hexists', KEYS[1], ARGV[3]) == 0)`: 如果哈希表中不存在对应的线程ID，则返回nil表示无法解锁。
    - `local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1)`: 将哈希表中对应线程ID的值减1，并将结果赋值给变量counter。
    - `if (counter > 0)`: 如果counter大于0，表示还有其他线程持有锁，执行以下操作：
        - `redis.call('pexpire', KEYS[1], ARGV[2])`: 设置键的过期时间为租约时间。
        - `return 0`: 返回0表示锁仍然被其他线程持有。
    - `else`: 如果counter等于0，表示当前线程是最后一个持有锁的线程，执行以下操作：
        - `redis.call('del', KEYS[1])`: 删除键，释放锁。
        - `redis.call(ARGV[4], KEYS[2], ARGV[1])`: 调用发布命令，通知其他线程锁已经释放。
        - `return 1`: 返回1表示成功释放锁。
    - `return nil`: 如果前面的条件都不满足，返回nil表示无法解锁。
- `evalWriteAsync`方法用于执行Lua脚本并返回异步结果对象。
- `getRawName()`: 获取锁的名称。
- `LongCodec.INSTANCE`: 用于编码和解码长整型数据。
- `RedisCommands.EVAL_BOOLEAN`: 指定Lua脚本的返回类型为布尔值。
- `Arrays.asList(getRawName(), getChannelName())`: 创建一个包含两个元素的列表，元素分别为锁的名称和频道名称。
- `LockPubSub.UNLOCK_MESSAGE`: 发布消息的内容。
- `internalLockLeaseTime`: 锁的租约时间。
- `getLockName(threadId)`: 根据线程ID生成锁的名称。
- `getSubscribeService().getPublishCommand()`: 获取发布命令。

## 3.3、锁续期

watchDog

核心工作流程是定时监测业务是否执行结束，没结束的话在看你这个锁是不是快到期了（超过锁的三分之一时间），那就重新续期。这样防止如果业务代码没执行完，锁却过期了所带来的线程不安全问题。

Redisson 的 watchDog 机制底层不是调度线程池，而是直接用的 netty 事件轮。

Redisson的WatchDog机制是用于自动续期分布式锁和监控对象生命周期的一种机制，确保了分布式环境下锁的正确性和资源的及时释放。

1. 自动续期：当Redisson客户端获取了一个分布式锁后，会启动一个WatchDog线程。这个线程负责在锁即将到期时自动续期，保证持有锁的线程可以继续执行任务。默认情况下，锁的初始超时时间是30秒，每10秒钟WatchDog会检查一次锁的状态，如果锁依然被持有，它会将锁的过期时间重新设置为30秒。
2. 参数配置：可以通过设置lockWatchdogTimeout参数来调整WatchDog检查锁状态的频率和续期的超时时间。这个参数默认值是30000毫秒（即30秒），适用于那些没有明确指定leaseTimeout参数的加锁请求。
3. 重连机制：除了锁自动续期外，WatchDog机制还用作Redisson客户端的自动重连功能。当客户端与Redis服务器失去连接时，WatchDog会自动尝试重新连接，从而恢复服务的正常运作。
4. 资源管理：WatchDog也负责监控Redisson对象的生命周期，例如分布式锁。当对象的生命周期到期时，WatchDog会将其从Redis中删除，避免过期数据占用过多内存空间。
5. 异步加锁：在加锁的过程中，WatchDog会在RedissonLock#tryAcquireAsync方法中发挥作用，该方法是进行异步加锁的逻辑所在。通过这种方式，加锁操作不会阻塞当前线程，提高了系统的性能。