---
title: redis
date: 2023/08/07
categories:
  - coding
tags:
  - redis
  - 缓存中间件
  - 编程基础
abbrlink: 44296
---

# 1、基本概念

Redis诞生于2009年全称是Remote Dictionary Server，远程词典服务器，是一个基于内存的键值型NoSQL数据库。

## 1.1、redis特性

1. 键值（key-value）型，value支持多种不同数据结构，功能丰富
2. 单线程，每个命令具备原子性

> Redis的网络IO和键值对读写是由一个线程来完成的,但Redis的其他功能,例如持久化、异步删除、集群数据同步等操作依赖于其他线程来执行

3. 低延迟，速度快

>原因：基于内存、采用多路复用非阻塞I/O、单线程


1. 支持数据持久化
2. 支持主从集群、分片集群
3. 支持多语言客户端

## 1.2、数据类型

> Redis是一个key-value的数据库，key一般是String类型，不过value的类型多种多样

![redis数据类型](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202308071403029.png)


# 2、redis常见命令
## 2.1、String

String是Redis中最基本的数据类型，可以存储任何数据，包括二进制数据、序列化的数据、JSON化的对象甚至是图片。 

> String类型，也就是字符串类型，是Redis中最简单的存储类型
> 底层SDS结构。为什么不直接实用字符串？①C 语言字符数组最后一个元素总是 '\0'，而在Redis中\0可能会被判定为提前结束而识别不了字符串②获取字符串长度为O(n)，因为C字符串需要去遍历，开销较大，SDS对象有len属性直接获取

其value是字符串，不过根据字符串的格式不同，又可以分为3类：
（1）string：普通字符串
（2）int：整数类型，可以做自增、自减操作
（3）float：浮点类型，可以做自增、自减操作

| 命令 | 描述 |
| --- | --- |
| SET | 添加或者修改已经存在的一个String类型的键值对 |
| GET | 根据key获取String类型的value |
| MSET | 批量添加多个String类型的键值对 |
| MGET | 根据多个key获取多个String类型的value |
| INCR | 让一个整型的key自增1 |
| INCRBY | 让一个整型的key自增并指定步长，例如：incrby num 2 让num值自增2 |
| INCRBYFLOAT | 让一个浮点类型的数字自增并指定步长 |
| SETNX | 添加一个String类型的键值对，前提是这个key不存在，否则不执行 |
| SETEX | 添加一个String类型的键值对，并且指定有效期 |

> **Redis的key允许有多个单词形成层级结构，多个单词之间用” ：“隔开，格式如下：**

```java
项目名:业务名:类型:id
```
## 2.2、Hash

> Hash类型，也叫散列，底层是hashtable，其value是一个无序字典，类似于Java中的HashMap结构。

Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD

| **命令** | **描述** |
| --- | --- |
| HSET key field value | 添加或者修改hash类型key的field的值 |
| HGET key field | 获取一个hash类型key的field的值 |
| HMSET | hmset 和 hset 效果相同 ，4.0之后hmset可以弃用了 |
| HMGET | 批量获取多个hash类型key的field的值 |
| HGETALL | 获取一个hash类型的key中的所有的field和value |
| HKEYS | 获取一个hash类型的key中的所有的field |
| HVALS | 获取一个hash类型的key中的所有的value |
| HINCRBY | 让一个hash类型key的字段值自增并指定步长 |
| HSETNX | 添加一个hash类型的key的field值，前提是这个field不存在，否则不执行 |

## 2.3、List

> list列表的数据结构使用的是压缩列表ziplist和普通的双向链表linkedlist组成。元素少的时候会用ziplist，元素多的时候会用linkedlist
  ziplist是一种压缩链表，它的好处是更能节省内存空间，因为它所存储的内容都是在连续的内存区域当中的,当数据量较大的时候因为需要重新分配，开销较大

常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等

| **命令** | **描述** |
| --- | --- |
| LPUSH key element … | 向列表左侧插入一个或多个元素 |
| LPOP key | 移除并返回列表左侧的第一个元素，没有则返回nil |
| **RPUSH key element …** | 向列表右侧插入一个或多个元素 |
| RPOP key | 移除并返回列表右侧的第一个元素 |
| LRANGE key star end | 返回一段角标范围内的所有元素 |
| BLPOP和BRPOP | 与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil |

## 2.4、SET

> Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征。数据结构的底层实现有两种方式：Intset 和 Hashtable。当集合中的所有元素都是整数，并且元素数量较少时，Redis 会使用 Intset 作为底层实现。当集合中的元素不仅限于整数，或者元素数量较多时，Redis 会使用 Hashtable 作为底层实现
 

| **命令** | **描述** |
| --- | --- |
| SADD key member … | 向set中添加一个或多个元素 |
| SREM key member … | 移除set中的指定元素 |
| SCARD key | 返回set中元素的个数 |
| SISMEMBER key member | 判断一个元素是否存在于set中 |
| SMEMBERS | 获取set中的所有元素 |
| SINTER key1 key2 … | 求key1与key2的交集 |
| SDIFF key1 key2 … | 求key1与key2的差集 |
| SUNION key1 key2 … | 求key1和key2的并集 |

## 2.5、SortedSet

> Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能

| **命令** | **描述** |
| --- | --- |
| ZADD key score member | 添加一个或多个元素到sorted set ，如果已经存在则更新其score值 |
| ZREM key member | 删除sorted set中的一个指定元素 |
| ZSCORE key member | 获取sorted set中的指定元素的score值 |
| ZRANK key member | 获取sorted set 中的指定元素的排名 |
| ZCARD key | 获取sorted set中的元素个数 |
| ZCOUNT key min max | 统计score值在给定范围内的所有元素的个数 |
| ZINCRBY key increment member | 让sorted set中的指定元素自增，步长为指定的increment值 |
| ZRANGE key min max | 按照score排序后，获取指定排名范围内的元素 |
| ZRANGEBYSCORE key min max | 按照score排序后，获取指定score范围内的元素 |
| ZDIFF、ZINTER、ZUNION | 求差集、交集、并集 |

> **注意：所有的排名默认都是升序，如果要降序则在命令的Z后面添加REV即可**


# 3、java客户端
## 3.1、springboot整合redis
### 3.1.1、引入依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
### 3.1.2、redis基本配置

```properties
#redis基本配置
spring.redis.host=127.0.0.1
spring.redis.port=6379
```
### 3.1.3、redis固定配置

```java
@Configuration
public class RedisConfig {
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        // 配置连接池工厂
        template.setConnectionFactory(factory);

        // Jackson序列化配置
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        // 属性访问器为全部，作用域为全部
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 序列化输入类型必须是非final类型的
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        // String 的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}
```


> redis使用JDK提供的序列化功能。 优点是反序列化时不需要提供类型信息(class)，但缺点是需要实现Serializable接口，还有序列化后的结果非常庞大，是JSON格式的5倍左右，这样就会消耗redis服务器的大量内存
> 所以我们需要  使用Jackson库将对象序列化为JSON字符串。优点是速度快，序列化后的字符串短小精悍，易读

### 3.1.4、工具类


```java
package com.kaka.redis;

import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import javax.annotation.Resource;
import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Component
public final class RedisUtils {

    @Resource
    private RedisTemplate<String, Object> redisTemplate;


    public Set<String> keys(String keys){
        try {
            return redisTemplate.keys(keys);
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 指定缓存失效时间
     * @param key 键
     * @param time 时间(秒)
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 根据key 获取过期时间
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }
    /**
     * 判断key是否存在
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 删除缓存
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete((Collection<String>) CollectionUtils.arrayToList(key));
            }
        }
    }
    /**
     * 普通缓存获取
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }
    /**
     * 普通缓存放入
     * @param key 键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 普通缓存放入, 不存在放入，存在返回
     * @param key 键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean setnx(String key, Object value) {
        try {
            redisTemplate.opsForValue().setIfAbsent(key,value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 普通缓存放入并设置时间
     * @param key 键
     * @param value 值
     * @param time 时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 普通缓存放入并设置时间,不存在放入，存在返回
     * @param key 键
     * @param value 值
     * @param time 时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean setnx(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().setIfAbsent(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 递增
     * @param key 键
     * @param delta 要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }
    /**
     * 递减
     * @param key 键
     * @param delta 要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }
    /**
     * HashGet
     * @param key 键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }
    /**
     * 获取hashKey对应的所有键值
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }
    /**
     * HashSet
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * HashSet 并设置时间
     * @param key 键
     * @param map 对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key 键
     * @param item 项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key 键
     * @param item 项
     * @param value 值
     * @param time 时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 删除hash表中的值
     * @param key 键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }
    /**
     * 判断hash表中是否有该项的值
     * @param key 键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }
    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     * @param key 键
     * @param item 项
     * @param by 要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }
    /**
     * hash递减
     * @param key 键
     * @param item 项
     * @param by 要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }
    /**
     * 根据key获取Set中的所有值
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
    /**
     * 根据value从一个set中查询,是否存在
     * @param key 键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 将数据放入set缓存
     * @param key 键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    /**
     * 将set数据放入缓存
     * @param key 键
     * @param time 时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0){
                expire(key, time);
            }
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    /**
     * 获取set缓存的长度
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    /**
     * 移除值为value的
     * @param key 键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    // ===============================list=================================
    /**
     * 获取list缓存的内容
     * @param key 键
     * @param start 开始
     * @param end 结束 0 到 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
    /**
     * 获取list缓存的长度
     * @param key 键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    /**
     * 通过索引 获取list中的值
     * @param key 键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0){
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 将list放入缓存
     *
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0){
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 根据索引修改list中的某条数据
     * @param key 键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 移除N个值为value
     * @param key 键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
}
```

5. 使用

```java
@RestController
public class Demo {
    @Autowired
    RedisUtils redisUtils;
    @RequestMapping("/redisTest")
    public String test(){
        redisUtils.set("test3:key3","hello,redis");
        return redisUtils.get("test3:key3").toString();
    }
}
```

# 4、进阶
## 4.1、redis持久化
### 4.1.1、rdb

把当前内存中的快照写入磁盘

（1）save：save指令执行会阻塞当前redis服务器，直到当前rdb过程执行完，可能造成长时间阻塞，线上环境不建议使用
（2）bgsave：调用fork函数生成子进程，解决了save的阻塞问题
（3）自动执行：（redis配置文件中配置）save 900 1   save 300 10   save 60 1000

### 4.1.2、aof

以日志的方式记录每次写命令，重启时再执行aof中的命令达到数据恢复的目的（是目前redis持久化的主流方式）

aof写数据策略：

（1）always：服务器每写入一个命令,就调用一次fdatasync（不会丢失数据）
（2）Everysec：服务器每一秒重调用一次fdatasync（数据同步），最多丢失1秒的数据
（3）NO：操作系统决定任何将缓冲区里面的命令写入磁盘里面，数据丢失量是不确定的

> 注：always策略持久化数据：先把写命令追加到aof buffer中，下一次进入事件循环循环后，再将buffer写到磁盘上。也就是说，这次写到磁盘上的内容是上一个事件循环产生的所以，即使设置为always，也会丢失一个循环的数据


### 4.1.3、对比

|  | rdb | aof |
| --- | --- | --- |
| 占用存储空间 | 小（数据级） | 大（指令级） |
| 恢复速度 | 快 | 慢（需要执行指令） |
| 数据安全性 | 可能会丢失最后一次持久化后的数据 | 根据策略决定 |

## 4.2、redis数据删除策略

### 4.2.1、定时删除
方式：创建一个定时器,当设置的key到达到期时间时,由定时器任务立即执行对key的删除操作

优缺点：
（1）节约内存,到时就删,快速释放掉不必要的内存占用 
（2）CPU压力变大,无论CPU此时负载量多高,均占用CPU

### 4.2.2、惰性删除
方式：数据到期时不做删除,等下次访问时进行删除

优缺点：
（1）节约cpu性能  
（2）若大量的key在超出超时时间后，很久一段时间内，都没有被获取过，那么可能发生内存泄露（无用的垃圾占用了大量的内存）

### 4.2.3、定期删除
方式：每隔一段时间主动检查一批过期键，并将其删除。这样可以保证过期键及时地从内存中释放

>（1）Redis 默认每秒进行 10 次过期扫描，此配置可通过 Redis 的配置文件 redis.conf 进行配置，配置健为
 hz 它的默认值是 hz 10。
 【注意】:Redis 每次扫描并不是遍历过期字典中的所有健，而是采用随机抽取判断并删除过期健的形式执行的。

删除流程：
（1）从过期字典随机取20个键
（2）删除这20个键中过期的键
（3）如果过期key的比例超过25%，重复步骤1

优缺点：
（1）分批处理，以避免对cpu产生过大的负载

## 4.3、redis内存淘汰策略

Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据

>LRU：淘汰最长时间没有被使用的
 LFU：一定时间内使用频次越低的
 random：随机
 ttl：越早过期的数据
 
|淘汰策略名称|策略含义|
|---|---|
|noeviction|默认策略，不淘汰数据；大部分写命令都将返回错误（DEL等少数除外）|
|allkeys-lru|从所有数据中根据 LRU 算法挑选数据淘汰|
|volatile-lru|从设置了过期时间的数据中根据 LRU 算法挑选数据淘汰|
|allkeys-random|从所有数据中随机挑选数据淘汰|
|volatile-random|从设置了过期时间的数据中随机挑选数据淘汰|
|volatile-ttl|从设置了过期时间的数据中，挑选越早过期的数据进行删除|
|allkeys-lfu|从所有数据中根据 LFU 算法挑选数据淘汰（4.0及以上版本可用）|
|volatile-lfu|从设置了过期时间的数据中根据 LFU 算法挑选数据淘汰（4.0及以上版本可用）|

  
## 4.4、redis工作模式（高可用）

### 4.4.1、单机模式

单机模式是最简单的 Redis 工作模式。在单机模式下，Redis 只运行在单个节点上，数据存储在该节点的内存中。这种模式适用于小规模应用或开发环境

### 4.4.2、主从复制模式

主从复制模式通过将数据从主节点复制到一个或多个从节点来提高数据的可靠性和读取性能。主节点负责处理写入操作，从节点复制主节点的数据，并可以处理读取操作。主从复制模式适用于需要读取扩展和数据冗余的场景

### 4.4.3、哨兵模式

主从复制的基础上，引入了哨兵节点来监控主节点的状态。当主节点发生故障时，哨兵节点会自动将一个从节点升级为新的主节点，并将其他从节点重新配置为复制新的主节点。这种模式提供了故障转移和自动主节点切换的功能

### 4.4.4、集群模式

即使使用哨兵，redis每个实例也是全量存储，每个redis存储的内容都是完整的数据。cluster是为了解决单机Redis容量有限的问题，将数据按一定的规则分配到多台机器，提高并发量

## 4.5、redis发布订阅机制

Redis 发布订阅（Pus/Sub）是一种消息通信模式：发送者通过 PUBLISH发布消息，订阅者通过 SUBSCRIBE 订阅接收消息或通过UNSUBSCRIBE 取消订阅。
发布者和订阅者属于客户端，Channel 是 Redis 服务端，发布者将消息发布到频道，订阅这个频道的订阅者则收到消息。从而实现消息的广播和实时通知


> Redis 的发布订阅机制是一种简单的消息传递方式，并不提供消息持久化和消息队列的功能。如果需要更高级的消息队列功能，可以考虑rabbitmq，kafka等

```shell
# A订阅频道
SUBSCRIBE channel1

# B向频道发送消息，A就可以收到消息
PUBLISH channel1 "Redis PUBLISH test"

```

## 4.6、缓存穿透、击穿、雪崩
###  4.6.1、缓存穿透

缓存穿透：某些不存在的数据，被大量的查询访问，缓存层中没有这些数据的缓存，请求就直达存储层，造成宕机

> 解决方法：
> 1.返回空对象，将该key的空值返回给缓存层，缓存层会直接返回空对象。
> 2.布隆过滤器：将所有的key都存在过滤器中，在访问缓存层的时候会首先访问过滤器，如果过滤器中不存在这个值，那么直接返回空值。 

> 布隆过滤器：它是一种类似哈希的数据结构，通过这个数据结构，可以快速的插入和查询，确定某个事件一定不存在或可能存在。特点是占用空间少，缺点是返回的结果是概率性
> 当一个元素加入集合时，就通过K个hash函数将这个映射成一个位数组中的K个点，把它们置为1。当查询时，只要检查这些点是否全为1，就能判断集合中是否可能存在。
> 如果k个点有任何一个0，则被检元素一定不在。如果都是1，则很可能存在，这个期望概率是可以设置


### 4.6.2、缓存击穿？

一份热点数据，在它缓存失效期间，大量的请求直接命中存储层

> 解决方法：
 1.设置热点数据永不过期的策略。
 2.加互斥锁，在一个请求访问时另一个不能访问，这样，在这个请求访问过后，缓存重建，其他线程就可以访问了

### 4.6.3、缓存雪崩？

当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，会给后端系统带来很大压力。导致系统崩溃。

> 解决方法：
>1.不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀
>2.在系统启动或低峰期(比如系统刚启动)，提前加载热门数据到缓存中，避免在高峰期大量请求同时访问导致缓存失效


## 4.7、redis实现分布式锁

在分布式的环境下,会发生多个server并发修改同一个资源的情况,这种情况下,由于多个server是多个不同的JRE环境,而Java自带的锁局限于当前JRE,所以Java自带的锁机制在这个场景下是无效的,那么就需要我们自己来实现一个分布式锁

1. 通过`set...nx...`命令,将加锁、过期命令编排到一起,把他们变成原子操作。完整命令：set key random-value nx ex seconds

> 其实目前通常所说的Setnx命令，并非单指Redis的setnx key value这条命令。
> 一般代指Redis中对set命令加上nx参数进行使用

> （1）nx  ex 是set指令的两个参数： ex过期时间    nx只有key不存在时设置新的key/value
> （2）key设置成随机数，避免一个线程过期时间内没释放掉锁，过期后有另一个线程获取到锁，该线程执行完后释放掉另一个线程获取的锁
> （3）设置过期时间（EX）作用：如果客户端忘记解锁,那么这种情况就很有可能造成死锁
> （4）NX的作用：避免重复获取锁


2. 解锁的时候进行判断,是自己持有的锁才能释放,否则不能释放。另外判断,释放这两步需要保持原子性，所以通过Lua脚本将两个命令编排在一起,而整个Lua脚本的执行是原子的

> if redis.call("get",KEYS[1]) == ARGV[1] then return redis.call("del",KEYS[1]) else return 0 end

> **这里为什么要用原子操作？**
> 主要是怕误将其他客户端的锁解开。比如客户端A加锁，一段时间之后客户端A解锁，在进入unlock后执行jedis.del()之前，锁突然过期了，此时客户端B尝试加锁成功，然后客户端A再执行del()方法，则将客户端B的锁给解除

```java
import redis.clients.jedis.Jedis;

public class RedisLock {
    private Jedis jedis;

    public RedisLock(Jedis jedis) {
        this.jedis = jedis;
    }

    /**
     * 尝试获取锁
     * @param lockKey 锁的名称
     * @param requestId 请求标识，用于释放锁
     * @param expireTime 锁的过期时间
     * @return 是否成功获取锁
     */
    public boolean tryLock(String lockKey, String requestId, int expireTime) {
        String result = jedis.set(lockKey, requestId, "NX", "EX", expireTime);
        if ("OK".equals(result)) {
            return true;
        }
        return false;
    }

    /**
     * 释放锁
     * @param lockKey 锁的名称
     * @param requestId 请求标识，用于判断是否是同一个客户端
     * @return 是否成功释放锁
     */
    public boolean releaseLock(String lockKey, String requestId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        // jedis.eval是Jedis客户端提供的一个用于执行Lua脚本的方法
        Long result = (Long) jedis.eval(script, 1, lockKey, requestId);
        if (result == 1) {
            return true;
        }
        return false;
    }
}

```

> 另外可以通过Redisson框架，它的底层原理其实也是这个setnx


## 4.8、redis和数据库的双写一致性
更新数据库和更新redis不是一个原子操作。所以根据业务场景有两种方案：

1.保证最终一致性（可以接受数据短期不一致）

先更新数据库，再更新redis。第二步更新redis失败的请求异步写入mq消息队列，利用mq的重试机制进行更新

![image.png|500](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311141021779.png)


2.强一致性保证

使用读写锁，在数据更新的时候，其它任何请求都无法访问缓存中的数据

![image.png|500](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311141022655.png)

