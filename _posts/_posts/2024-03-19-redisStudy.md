---
layout: single
title: "[BigData]redis的一些使用"
categories: [DB]
last_modified_at: 2024-03-15
excerpt: "So how do you solve the concurrency contention problem with Redis?"
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---




### redis 基础


#### Redis通用命令  


```sql


KEYS pattern  
  
KEYS *  
  
TYPE k1  

-- Redis是一个开源的，基于内存的数据结构存储系统，它可以用作数据库、缓存和消息中间件。  
-- 它支持多种类型的数据结构，如字符串（strings）、散列（hashes）、列表（lists）、集合（sets）、有序集合（sorted sets）等。  
-- 其中key 一般是字符串，value 可以是字符串、散列、列表、集合、有序集合等数据结构。  
-- Redis还支持各种不同的功能，如事务、Pub/Sub、Lua脚本、键过期等。  
-- Redis的通用命令包括：DEL、EXISTS、EXPIRE、EXPIREAT、KEYS、MOVE、PERSIST、PEXPIRE、PEXPIREAT、PTTL、RANDOMKEY、RENAME、RENAMENX、TTL、TYPE等。  
-- 示例：  
  
-- 删除键  
DEL k1  
-- 检查键是否存在  
EXISTS k1  
-- 设置键的过期时间  
EXPIRE k1 10  
-- 设置键的过期时间（时间戳）  
EXPIREAT k1 1293840000  
-- 查找所有符合给定模式 pattern 的 keyKEYS *  
-- 将当前数据库的 key 移动到给定的数据库 db 当中  
MOVE k1 1  
-- 移除 key 的过期时间  
PERSIST k1  
```

  
#### Redis字符串命令  


```sql

-- 概述：value是字符串，根据格式分为string（普通字符串）和int（整数），float（浮点数）。  
-- Redis字符串命令包括：  
-- SET：添加或修改一个字符串键值对。  
SET k1 v1  
-- GET：获取一个字符串键的值。  
GET k1  
-- MSET: 添加或修改多个字符串键值对。  
MSET k2 v2 k3 v3  
-- MGET: 获取多个字符串键的值。  
MGET k2 k3  
-- INCR: 将键的值加1。(整数) | INCRBY: 将键的值加上指定的整数。(整数) | INCRBYFLOAT: 将键的值加上指定的浮点数。(浮点数)  
SET k4 1  
INCR k4  
GET k4  
-- DECR: 将键的值减1。(整数)  
-- SETNX: 设置键的值，仅当键不存在时。  
SETNX k5 v5  
-- SETEX: 设置键的值，并设置过期时间。  
SETEX k6 10 v6  
```
  
#### Redis的Key层级结构  


```sql

-- 概述：Redis的key是字符串，可以包含多个层级，层级之间使用冒号分隔。  
-- 格式：[层级1]:[层级2]:[层级3]:...:[层级n] | 项目：业务类型：编号  
-- 示例：格式：姓名：类型：编号  
SET Kasumi:Student:001 '{"id": "001", "name": "Kasumi", "age": 18}'  
SET Kasumi:Teacher:001 '{"id": "001", "name": "Kasumi", "age": 28}'  
SET Nota:Teacher:001 '{"id": "001", "name": "Nota", "age": 28}'  
-- 把值序列化成json字符串，有利于java、python等语言的解析。
```
  
  
#### Redis散列命令（hash） 


```sql

-- 概述：value是散列，散列是一个键值对集合，适合存储对象。可以把对象的字段独立存储，方便修改和查询。  
-- Redis散列命令包括：  
-- HSET key field value：设置散列键的字段值。  
HSET Kasumi:Student:003 name alice  
HSET Kasumi:Student:003 age 18  
-- HGET key field：获取散列键的字段值。  
HGET Kasumi:Student:003 name  
-- HMSET key field value [field value ...]：设置散列键的多个字段值。| HMGET key field [field ...]：获取散列键的多个字段值。  
-- HGETALL key：获取散列键的所有字段值。  
HGETALL Kasumi:Student:003  
-- HDEL key field [field ...]：删除散列键的字段值。  
HDEL Kasumi:Student:003 age  
--HKYES key：获取散列键的所有字段名。  
HKEYS Kasumi:Student:003  
-- HVALS key：获取散列键的所有字段值。  
HVALS Kasumi:Student:003  
```

  
#### Redis列表命令（list） 


```sql

-- 概述：value是列表，列表是一个有序的字符串集合，可以添加、删除、修改、查询。与java的linkedlist类似。看作双向链表。支持正反向遍历。  
-- Redis列表命令包括：  
-- LPUSH key value [value ...]：在列表头部添加一个或多个值。| RPUSH key value [value ...]：在列表尾部添加一个或多个值。  
LPUSH Kasumi:Student:003:Course math  
LPUSH Kasumi:Student:003:Course english chinese  
-- LPOP key：从列表头部删除一个值。| RPOP key：从列表尾部删除一个值。|| BLPOP key [key ...] timeout：从列表头部删除一个值，如果列表为空则阻塞等待。  
LPOP Kasumi:Student:003:Course  
-- LRANGE key start stop：获取列表的指定范围的值。0表示第一个元素，-1表示最后一个元素。  
LRANGE Kasumi:Student:003:Course 0 -1  
-- LINDEX key index：获取列表的指定索引的值。  
LINDEX Kasumi:Student:003:Course 0  
```

  
#### Redis集合命令（set）


```sql

-- 概述：value是集合，集合是一个无序的字符串集合，可以添加、删除、修改、查询。与java的hashset类似。无序，不重复，支持交集、并集、差集。  
-- Redis集合命令包括：  
-- SADD key member [member ...]：向集合添加一个或多个成员。| SREM key member [member ...]：从集合删除一个或多个成员。  
SADD Kasumi:Student:004:Sport football basketball  
-- SCARD key：获取集合的成员数量。  
SCARD Kasumi:Student:004:Sport  
-- SMEMBERS key：获取集合的所有成员。  
SMEMBERS Kasumi:Student:004:Sport  
-- SISMEMBER key member：判断成员是否在集合中。  
SISMEMBER Kasumi:Student:004:Sport football  
  
SADD Kasumi:Student:004:Sport2 pingpong basketball  
-- SINTER key [key ...]：获取多个集合的交集。  
SINTER Kasumi:Student:004:Sport Kasumi:Student:004:Sport2  
-- SUNION key [key ...]：获取多个集合的并集。  
SUNION Kasumi:Student:004:Sport Kasumi:Student:004:Sport2  
-- SDIFF key [key ...]：获取多个集合的差集。  
SDIFF Kasumi:Student:004:Sport Kasumi:Student:004:Sport2  
  
#### Redis有序集合命令（sorted set）  
-- 概述：value是有序集合，有序集合是一个有序的字符串集合，可以添加、删除、修改、查询。与java的treeset类似。有序，不重复，支持按照分数排序。常用做排行榜。  
-- Redis有序集合命令包括：  
-- ZADD key score member [score member ...]：向有序集合添加一个或多个成员。-- ZREM key member [member ...]：从有序集合删除一个或多个成员。  
ZADD Kasumi:Student:005:Score 100 math 90 english 80 chinese  
-- ZCARD key：获取有序集合的成员数量。  
ZCARD Kasumi:Student:005:Score  
-- ZRANGE key start stop [WITHSCORES]：获取有序集合的指定范围的成员。| ZREVRANGE key start stop [WITHSCORES]：获取有序集合的指定范围的成员，逆序。  
ZREVRANGE Kasumi:Student:005:Score 0 -1 WITHSCORES  
-- ZSCORE key member：获取有序集合的成员的分数。  
ZSCORE Kasumi:Student:005:Score math  
-- ZRANK key member：获取有序集合的成员的排名。  
ZRANK Kasumi:Student:005:Score chinese  

```

#### Other


```sql

-- ## Redis事务命令  
-- 概述：Redis事务是一组命令的集合，可以一次性执行多个命令。事务中的命令要么全部执行，要么全部不执行。  
-- Redis事务命令包括：  
-- MULTI：开启事务。| EXEC：执行事务。| DISCARD：取消事务。  
-- MULTI  
-- SET k7 v7  
-- SET k8 v8  
-- EXEC  
  
-- ## Redis发布订阅命令  
-- 概述：Redis发布订阅是一种消息通信模式，包括发布者、订阅者、频道。发布者发布消息到频道，订阅者订阅频道，当发布者发布消息到频道时，订阅者会接收到消息。  
-- Redis发布订阅命令包括：  
-- SUBSCRIBE channel [channel ...]：订阅一个或多个频道。| UNSUBSCRIBE [channel [channel ...]]：取消订阅一个或多个频道。| PUBLISH channel message：发布消息到频道。  
SUBSCRIBE Kasumi:Channel  
  
-- ## Redis脚本命令  
-- 概述：Redis脚本是一段Lua脚本，可以一次性执行多个命令。脚本中的命令要么全部执行，要么全部不执行。  
-- Redis脚本命令包括：  
-- EVAL script numkeys key [key ...] arg [arg ...]：执行Lua脚本。| EVALSHA sha1 numkeys key [key ...] arg [arg ...]：执行Lua脚本。  
-- SCRIPT LOAD script：加载Lua脚本。| SCRIPT FLUSH：清空Lua脚本。  
-- EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
```
  


---


### redis Java Client 编程

#### Jedis

官方推荐的Java连接开发工具。类似与JDBC-SQLDB


```xml
<dependencies>  
    <!--jedis-->  
    <dependency>  
        <groupId>redis.clients</groupId>  
        <artifactId>jedis</artifactId>  
        <version>5.0.0</version>  
    </dependency>    
    <!--junit测试-->  
    <dependency>  
        <groupId>org.junit.jupiter</groupId>  
        <artifactId>junit-jupiter</artifactId>  
        <version>5.7.0</version>  
        <scope>test</scope>  
    </dependency>  
</dependencies>
```

##### junit单元测试


```java
package com.kasumi.test;  
  
import org.junit.jupiter.api.AfterEach;  
import redis.clients.jedis.Jedis;  
import org.junit.jupiter.api.BeforeEach; //用于在每个测试方法之前执行一些初始化操作  
  
public class JedisTest {  
    private Jedis jedis;  
  
    @BeforeEach  
    public void init() {  
        jedis = new Jedis("localhost", 6379); // 连接本地的 Redis 服务  
    }  
  
    // @Test  
    public void test() {  
        jedis.set("name", "kasumi");  
        System.out.println("GET:"+jedis.get("name"));  
    }  
  
    //释放资源  
    @AfterEach  
    public void destroy() {  
        if (jedis != null) {  
            jedis.close();  
        }  
    }  
  
    public static void main(String[] args) {  
        JedisTest jedisTest = new JedisTest();  
        jedisTest.init();  
        jedisTest.test();  
        jedisTest.destroy();  
    }  
  
}
```

##### Jedis连接池

```java

import redis.clients.jedis.Jedis;  
import redis.clients.jedis.JedisPool;  
import redis.clients.jedis.JedisPoolConfig;  
  
public class JeConectFac { //连接池  
    private static final JedisPool jedispool;  
    static {  
        JedisPoolConfig poolConfig = new JedisPoolConfig(); //连接池配置  
        poolConfig.setMaxTotal(9); //最大连接数  
        poolConfig.setMaxIdle(9); //最大空闲连接数  
        poolConfig.setMinIdle(1); //最小空闲连接数  
        poolConfig.setBlockWhenExhausted(true); //连接耗尽时是否阻塞，false报异常，true阻塞直到超时，默认true  
        poolConfig.setMaxWaitMillis(5000); //连接耗尽时等待超时时间，单位毫秒  
  
  
        jedispool = new JedisPool(poolConfig, "localhost", 6379, 10000); //连接池对象  
    }  
  
    public static Jedis getJedis() {  
        return jedispool.getResource(); //获取连接  
    }  
}
```

对应测试文件：``jedis = JeConectFac.getJedis();``

---

#### SpringDataRedis

Spring Data Redis 是一个基于 Spring 的模块，它提供了对 Redis 的数据访问支持。Spring Data Redis 提供了丰富的 Redis 操作模板，它是一个高级的 Redis 客户端，使用它可以非常方便地操作 Redis。  Spring Data Redis 的主要特性包括： 

- 连接支持：提供了对多种 Redis 客户端库的支持，包括 Jedis、Lettuce 和 Redisson。  
- 数据操作：提供了丰富的数据操作 API，包括对 key-value、list、set、zset、hash 等数据结构的操作。  
- 序列化：提供了多种序列化策略，包括 String、JDK、JSON、XML 等。  
- Repository 支持：通过简单的注解，可以实现对 Redis 数据的 CRUD 操作。  
- Pub/Sub：支持 Redis 的发布订阅模式。  
- 集群：支持 Redis 集群和哨兵模式。  

> 引入依赖 -> 配置 -> 注入实例


.properties配置如下：


```shell
spring.application.name=SpringBoot  
  
# redis  
spring.data.redis.host=localhost  
spring.data.redis.port=6379  
  
# pool  
spring.data.redis.lettuce.pool.max-active=8  
spring.data.redis.lettuce.pool.max-idle=8  
spring.data.redis.lettuce.pool.max-wait=-1  
spring.data.redis.lettuce.pool.min-idle=0
```

测试文件：

```java

import org.junit.jupiter.api.Test;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.test.context.SpringBootTest;  
import org.springframework.data.redis.core.RedisTemplate;  
  
@SpringBootTest  
class ApplicationTests {  
  
    @Autowired  
    private RedisTemplate <String, String> redisTemplate;  
    //为redisTemplate字段注入一个RedisTemplate<String, String>类型的实例。这个实例由Spring框架创建并管理，不需要手动创建。所谓"依赖注入"/"控制反转"的设计模式。
  
    @Test  
    void testString() {  
        redisTemplate.opsForValue().set("Spring", "summer");  
        String value = redisTemplate.opsForValue().get("Spring");  
        System.out.println("GET: "+value);  
    }  

	@Test  
	void testList() {  
	    redisTemplate.opsForList().leftPush("list", "Spring");  
	    redisTemplate.opsForList().leftPush("list", "Summer");  
	    redisTemplate.opsForList().leftPush("list", "Autumn");  
	    String value = redisTemplate.opsForList().leftPop("list");  
	    System.out.println("POP: "+value);  
	}  
	  
	@Test  
	void testHash() {  
	    redisTemplate.opsForHash().put("hash", "Spring", "1");  
	    redisTemplate.opsForHash().put("hash", "Summer", "2");  
	    String value = (String) redisTemplate.opsForHash().get("hash", "Spring");  
	    System.out.println("GET: "+value);  
	    System.out.println("GET: "+redisTemplate.opsForHash().entries("hash"));  
	}
  
}
```


