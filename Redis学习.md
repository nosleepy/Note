---
title: Redis学习
date: 2020-09-19 10:25:18
tags:
categories:
---

## Redis 简介

Redis 是完全开源的，遵守 BSD 协议，是一个高性能的 key-value 数据库。

Redis 与其他 key - value 缓存产品有以下三个特点：

+ Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
+ Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
+ Redis支持数据的备份，即master-slave模式的数据备份。

## Redis安装

[CENTOS7下安装REDIS](https://www.cnblogs.com/zuidongfeng/p/8032505.html)

[redis在Windows10下的安装](https://www.cnblogs.com/juncaoit/p/10122642.html)

[redis开启远程访问](https://www.cnblogs.com/liusxg/p/5712493.html)

**修改redis.conf配置文件**

```conf
# bind 127.0.0.1 // 所有ip可以访问
daemonize yes // 后台进程启动
protected-mode no // 关闭保护模式
requirepass 123456 // 设置密码
```

**重新运行redis**

```
./redis-server ../redis.conf //必须指定配置文件
```

**本地连接redis**

```
./redis-cli
auth 123456 // 输入密码
```

**使用远程连接**

```
./redis-cli -h 192.168.32.104 // 远程ip地址
auth 123456 // 输入密码
```

## Redis存储数据的结构

+ string：最常用的，一般用于存储一个值。
+ hash：存储一个对象数据的。
+ list：使用list结构实现栈和队列结构。
+ set：交集，差集和并集的操作。
+ zset：排行榜，积分存储等操作。

## Redis常用命令

**string常用命令**

```
#1 添加值
set key value

#2 取值
get key

#3 批量操作
mset key value [key value...]
mget key [key...]
```

**hash常用命令**

```
#1 存储数据
hset key field value

#2 获取数据
hget key field

#3 批量操作
hmset key field value [field value...]
hmget key field [field...]
```

**list常用命令**

```
#1 存储数据（从左侧插入数据，从右侧插入数据）
lpush key value [value...]
rpush key value [value...]

#2 存储数据（指定索引位置）
lset key index value

#3 弹栈方式获取数据（左侧弹出数据，右侧弹出数据）
lpop key
rpop key

#4 获取指定索引范围数据
lrange key start stop

#5 获取指定索引位置的数据
lindex key index
```

**set常用命令**

```
#1 存储数据
sadd key member [member...]

#2 获取数据（获取全部数据）
smembers key

#3 随机获取一个数据（获取的同时，移除数据）
spop key [count]
```

**zset常用命令**

```
#1 添加数据
zadd key score member [score member...]

#2 修改member的分数
zincrby key increment member

#3 查看指定的member的分数
zscore key member

#4 获取zset中数据的数量
zcard key

#5 根据score的范围查询member数量
zcount key min max

#6 删除zset中的成员
zrem key member [member...]
```

**key常用命令**

```
#1 查看Redis中的全部的key
keys pattern

#2 查看某一个key是否存在
exists key

#3 删除key
del key [key...]

#4 选择操作的库
select 0~15

#5 移动key到另外一个库中
move key db
```

**db常用命令**

```
#1 清空当前所在的数据库
flushdb

#2 清空全部的库
flushall

#3 查看当前数据库中有多少个key
dbsize

#4 查看最后一次操作的时间
lastsave

#5 实时监控Redis服务接收到的命令
monitor
```

## Java连接Redis

**Jedis连接Redis**

+ 创建maven项目。
+ 导入需要的依赖。
+ 进行单元测试。

```java
public class JedisTest01 {

    Jedis jedis = null;

    @Before
    public void init() {
        // 1.连接Redis
        jedis = new Jedis("192.168.186.130", 6379);
        jedis.auth("123456");
    }

    @Test
    public void set() {
        // 2.操作Redis
        jedis.set("name", "qrsx");
        // 3.释放资源
        jedis.close();
    }

    @Test
    public void get() {
        // 2.操作Redis
        String value = jedis.get("name");
        System.out.println(value);
        // 3.释放资源
        jedis.close();
    }
}
```

**Jedis存储一个对象到Redis以byte[]的形式**

```java
public class JedisTest02 {

    Jedis jedis = null;

    @Before
    public void init() {
        jedis = new Jedis("192.168.186.130", 6379);
        jedis.auth("123456");
        System.out.println("连接成功...");
    }

    // 存储对象 - 以byte[]形式存储在Redis中
    @Test
    public void setByteArray() {
        String key = "user";
        User value = new User(1, "echo1328", new Date());
        byte[] byteKey = SerializationUtils.serialize(key);
        byte[] byteValue = SerializationUtils.serialize(value);
        jedis.set(byteKey, byteValue);
        jedis.close();
    }

    // 获得对象 - 以byte[]形式在Redis中获取
    @Test
    public void getByteArray() {
        String key = "user";
        byte[] byteKey = SerializationUtils.serialize(key);
        byte[] byteValue = jedis.get(byteKey);
        User value = (User) SerializationUtils.deserialize(byteValue);
        System.out.println(value);
        jedis.close();
    }
}
```

**Jedis存储一个对象到Redis以String的形式**

```java
public class JedisTest03 {

    Jedis jedis = null;

    @Before
    public void init() {
        jedis = new Jedis("192.168.186.130", 6379);
        jedis.auth("123456");
        System.out.println("连接成功...");
    }

    // 存储对象 - 以String形式存储在Redis中
    @Test
    public void setString() {
        String stringKey = "user";
        User user = new User(1, "echo", new Date());
        String stringValue = JSON.toJSONString(user);
        jedis.set(stringKey, stringValue);
        jedis.close();
    }

    // 获得对象 - 以String形式在Redis中获取
    @Test
    public void getString() {
        String stringKey = "user";
        String stringValue = jedis.get(stringKey);
        System.out.println(stringValue);
        User user = JSON.parseObject(stringValue, User.class);
        System.out.println(user);
        jedis.close();
    }
}
```

**Jedis连接池的操作**

```java
public class JedisTest04 {

    // 简单的方式
    @Test
    public void pool01() {
        JedisPool jedisPool = new JedisPool("192.168.186.130", 6379);
        Jedis jedis = jedisPool.getResource();
        jedis.auth("123456");
        String value = jedis.get("user");
        System.out.println(value);
        jedis.close();
    }

    // 复杂的方式
    @Test
    public void pool02() {
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig(); // 连接池配置信息
        poolConfig.setMaxTotal(100); // 最大活跃数
        poolConfig.setMaxIdle(10); // 最大空闲数
        poolConfig.setMinIdle(5); // 最小空闲数
        poolConfig.setMaxWaitMillis(3000); // 超时时间
        JedisPool jedisPool = new JedisPool(poolConfig, "192.168.186.130", 6379);
        Jedis jedis = jedisPool.getResource();
        jedis.auth("123456");
        String value = jedis.get("user");
        System.out.println(value);
        jedis.close();
    }
}
```

**Redis的管道操作**

因为在操作Redis的时候，执行一个命令需要先发送请求到Redis服务器，这个过程需要经历网络的延迟，Redis还需要给客户端一个响应。如果我们需要一次性执行很多个命令，可以通过Redis的管道，先将命令放到客户端的一个PipeLine中，之后一次性的将全部命令都发送到Redis服务器，Redis服务器一次性的将全部的返回结果响应给客户端。

```java
public class JedisTest04 {

    // Redis未使用管道操作
    @Test
    public void pipeLine01() {
        JedisPool jedisPool = new JedisPool("192.168.186.130", 6379);
        Jedis jedis = jedisPool.getResource();
        jedis.auth("123456");
        long start = System.currentTimeMillis();
        for(int i = 0; i < 100000; i++) {
            jedis.incr("count");
        }
        long end = System.currentTimeMillis();
        System.out.println("time = " + (end - start)); // 45332
        jedis.close();
    }

    // Redis使用管道操作
    @Test
    public void pipeLine02() {
        JedisPool jedisPool = new JedisPool("192.168.186.130", 6379);
        Jedis jedis = jedisPool.getResource();
        jedis.auth("123456");
        Pipeline pipelined = jedis.pipelined();
        long start = System.currentTimeMillis();
        for(int i = 0; i < 100000; i++) {
            pipelined.incr("count");
        }
        pipelined.syncAndReturnAll();
        long end = System.currentTimeMillis();
        System.out.println("time = " + (end - start)); // 133
        jedis.close();
    }
}
```

## Redis其他配置及集群

**Redis的AUTH**

通过修改Redis的配置文件，实现Redis的密码校验。

```
# Redis的AUTH密码
requirepass 123456
```

**Redis的持久化机制**

+ RDB是Redis默认的持久化机制。

```
# RDB持久化机制的配置

# 代表RDB执行的时机
save 900 1 // 900秒之内，有1个key改变了，就执行RDB持久化
save 300 10 // 300秒之内，有10个key改变了，就执行RDB持久化
save 60 10000 // // 60秒之内，有10000个key改变了，就执行RDB持久化

# 开启RDB持久化的压缩
rdbcompression yes

# RDB持久化文件的名称
dbfilename dump.rdb
```

+ AOF持久化机制默认是关闭的。

```
#AOF持久化机制的配置

# 代表开启AOF持久化
appendonly no

# AOF文件的名称
appendfilename "appendonly.aof"

# AOF持久化执行的时机

#appendfsync always // 每执行一个写操作就持久化

appendfsync everysec // 每秒执行持久化

#appendfsync no // 在一定时间内执行持久化
```

**Redis的事务**

Redis的事务：一次事务操作，该成功的成功，该失败的失败。

先开启事务，执行一系列的命令，但是命令不会立即执行，会被放在一个队列中，如果执行事务，这个队列中的命令会全部执行，如果取消了事务，这个队列中的命令全部作废。

```
// 事务操作
1. 开启事务：multi
2. 输入要执行的命令，被放入到一个队列中
3. 执行事务：exec
4. 取消事务：discard
5. 开启监听：watch
```

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 21
QUEUED
127.0.0.1:6379> set name echo
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
```

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 21
QUEUED
127.0.0.1:6379> set name echo
QUEUED
127.0.0.1:6379> incr name
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) (error) ERR value is not an integer or out of range
```

```
// 窗口1
127.0.0.1:6379> watch name age
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name qrsx
QUEUED
127.0.0.1:6379> set age 18
QUEUED
127.0.0.1:6379> exec
(nil)

// 窗口2
127.0.0.1:6379> set name error // 改变name的值
OK
```

**Redis的主存架构**

单机版Redis存在读写瓶颈的问题。

+ Master：读和写。
+ Slave：读。
+ Slave：读。

**哨兵**

哨兵可以帮助我们解决主从架构中的单点故障问题。

**Redis的集群**

Redis集群在保证主从加哨兵的基本功能之外，还能够提升Redis存储数据的能力。

1. Redis集群是无中心的。
2. Redis集群有一个ping-pang机制。
3. 投票机制，Redis集群节点的数量必须是2n+1。
4. Redis集群中默认分配了16384个hash槽，在存储数据时，就会将key进行crc16的算法，并且对16384取余，根据最终的结果，将key-value存放到指定的Redis节点中，而且每一个Redis节点都在维护着相应的hash槽。
5. 为了保证数据的安全性，每一个集群的节点，至少要跟着一个从节点。
6. 单独的针对Redis集群中的某一个节点搭建主从。
7. 当Redis集群中，超过半数的节点宕机之后，Redis集群就瘫痪了。

## Redis常见问题

**Redis的删除策略**

key的生存时间到了，Redis会立即删除吗？不会立即删除。（定期删除，惰性删除）

**Redis的淘汰机制**

在Redis内存已经满的时候，添加一个新的数据，执行淘汰机制。

```
maxmemory-policy：具体策略 // 指定淘汰机制的方式
maxmemory <bytes> // 设置Redis的最大内存
```

**缓存的常见问题**

> 缓存穿透：查询的数据，Redis中没有，数据库中也没有。

1. 根据id查询时，如果id是自增的，将id的最大值放到Redis中，在查询数据库之前，直接比较一下id。
2. 如果id不是整形，可以将全部的id放到set中，在用户查询之前，在set中查看一下是否有这个id。
3. 获取客户端的id地址，可以将id的访问添加限制。

> 缓存击穿：缓存中的热点数据，突然到期了，造成了大量的请求都去访问数据库，造成数据库宕机。

1. 在访问缓存中没有的时候，直接添加一个锁，让几个请求去访问数据库，避免数据库宕机。
2. 热点数据的生存时间去掉。

> 缓存雪崩：当大量缓存同时到期时，最终大量的请求同时去访问数据库，导致数据库宕机。

1. 将缓存中的数据的生存时间，设置为30~60的一个随机时间。

> 缓存倾斜：热点数据放在了一个Redis节点上，导致Redis节点无法承受住大量的请求，最终Redis宕机。

1. 扩展主从架构，搭建大量的从节点，缓解Redis的压力。
2. 可以在Tomcat中做JVM缓存，在查询Redis之前，先去查询Tomcat中的缓存。