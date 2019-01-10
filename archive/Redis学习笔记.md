---
title: Redis学习笔记
date: 2017-07-29 11:04:21
tags: Redis
categories: 数据库
---
### Redis简介   
这两天在公司写全量脚本的时候，涉及到了Redis数据库，索性就系统的学习一下，Redis是一个key-value键值数据库，它是基于内存存储,通常用来存储结构化的数据，它和Memcached比较类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型），下面是有关一些基本的操作。
<!-- more -->
#### Redis的安装
```
sudo apt-get install redis-server
```
Redis安装完成后，会自动启动，通过以下命令来查看redis线程是否启动
```
ps -ef | grep 6379
```
通过下面的命令检查redis服务器的状态
```
sudo /etc/init.d/redis-server status
```
#### Redis客户端操作

1. 插入数字
```
// 插入一个数字记录
set nx_key 3

// 让数字自增
10.1.4.10:6379> INCR nx_key
(integer) 2

10.1.4.10:6379> INCR nx_key
(integer) 3

```
2. 插入字符串
```
// 插入字符串
10.1.4.10:6379> set nx_key2 "nanxuan"
OK
// 查询字符串
10.1.4.10:6379> get nx_key2
"nanxuan"
```
3. 增加一个列表
```
10.1.4.10:6379> LPUSH nx_str "hello"
(integer) 1

10.1.4.10:6379> RPUSH nx_str "world"
(integer) 2

10.1.4.10:6379> RPUSH nx_str "!"
(integer) 3
//打印列表
10.1.4.10:6379> LRANGE nx_str 0 4
1) "hello"
2) "world"
3) "!"

```
4. 增加一个key为nx_hashMap的Hash列表
```
// 向Hash表中插入名为id,值为1100
10.1.4.10:6379> HSET nx_hashMap id "1100"
(integer) 1
10.1.4.10:6379> HSET nx_hashMap name "nanxuan"
(integer) 1
// 在Hash表中获取name对应的value
10.1.4.10:6379> HGET nx_hashMap name
"nanxuan"
// 打印Hash表中所有的key和value
10.1.4.10:6379> HGETALL nx_hashMap
1) "id"
2) "1100"
3) "name"
4) "nanxuan"
```
5. 向一个key为nx_hashMap2的Hash表中批量插入
```
// 批量插入，依次是key和value
10.1.4.10:6379> HMSET nx_hashMap2 username nanxuan password 1234 email dt@com
OK

// // 打印Hash表中所有的key和value
10.1.4.10:6379> HGETALL nx_hashMap2
1) "username"
2) "nanxuan"
3) "password"
4) "1234"
5) "email"
6) "dt@com"

//获取key为username和email对的值
10.1.4.10:6379> HMGET nx_hashMap2 username email
1) "nanxuan"
2) "dt@com"

```
6. 查询所有的key
```
10.1.4.10:6379> keys *
```
7. 删除记录
```
// 删除key为nx_hashMap的记录
10.1.4.10:6379> del nx_hashMap
(integer) 1
```
8. 远程访问redis服务器
```
redis-cli -a redisredis -h 10.1.4.10
```

### 在Java代码中访问Redis
(注意除了Java，Redis还被C++等多种语言支持)

```
import org.junit.Test;

import redis.clients.jedis.Jedis;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Created by nanxuan on 17-7-29.
 */
public class JedisTest {



  public static void main(String[] args) {
    // 这里使用了Jedis来连接Redis,使用前需导入redis.clients包
    Jedis jedis;
    jedis = new Jedis("10.1.4.10",6379);

    jedis.set("nx_key01","hello"); //添加数据
    System.out.println(jedis.get("nx_key01")); //查询结果

    jedis.append("nx_key01"," world!");// 链接字符串
    System.out.println(jedis.get("nx_key01"));

    //设置多个键值对
    jedis.mset("username", "nanxuan", "password", "123", "age", "21");
    jedis.incr("age");
    System.out.println(jedis.get("username") + ";" + jedis.get("password") +
        ";" + jedis.get("age"));

    jedis.del("nx_key01");
    System.out.println(jedis.get("nx_key01"));

    /**
     *redis操作Map
     */

    Map<String,String> map = new HashMap<String, String>();
    map.put("id", "3783274");
    map.put("name", "nanxuan");
    map.put("password", "1234");
    //将map存入redis
    jedis.hmset("user",map);
    
    //取出一个map所有的value,注意结果是Value的list,第一个参数是map的key,后面是放入map的key
    List<String> vmap = jedis.hmget("user", "id", "name", "password");
    System.out.println(vmap);

  }
}

```
