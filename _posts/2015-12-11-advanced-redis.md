---
layout:     post
title:      Redis 进阶
date:       2015-12-11
categories: Development
---

### Redis常用数据类型
#### String
set,get,decr,incr,mget
#### Hash
hget,hset,hgetall
#### List
lpush,rpush,lpop,rpop,lrange
#### Set
sadd,spop,smembers,sunion
#### Sorted set
zadd,zrange,zrem,zcard

### Redis的持久化机制
#### 定时快照方式(snapshot)
#### 基于语句追加文件的方式(aof)
#### 虚拟内存(vm)
#### Diskstore方式

[Redis Documentation](http://redis.io/documentation)

[深入剖析 redis 主从复制](http://daoluan.net/blog/2014/04/22/decode-redis-replication/)

[更好的内存管理-jemalloc](http://wangkaisino.blog.163.com/blog/static/1870444202011431112323846/)
[jemalloc原理分析](http://club.alibabatech.org/article_detail.htm?articleId=36)

[jemalloc](https://github.com/jemalloc/jemalloc)
[MurmurHash])(https://en.wikipedia.org/wiki/MurmurHash)