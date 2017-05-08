---
layout: post
title:  "1. Redis概述"
categories: Redis笔记
tags:  Redis
excerpt: Redis的全称是 REmote DIctionary Server(远程字典服务器)是完全开源免费的，用C编写并遵守BSD协议。是一个高性能(Key/Value)分布式内存数据库，基于内存运行，并支持持久化的NoSQL数据库，
---

* content
{:toc}


## 一些有用的网站

- redis官网：[http://redis.io](http://redis.io)
- redis中文官网：[http://redis.cn/](http://redis.cn/)
- redis命令参考文档：[http://redisdoc.com/](http://redisdoc.com/)  或 [http://doc.redisfans.com/](http://doc.redisfans.com/)

## Redis是什么

> Redis的全称是 REmote DIctionary Server(远程字典服务器)
>
> 是完全开源免费的，用C编写并遵守BSD协议。是一个高性能(Key/Value)
> 分布式内存数据库，基于内存运行，并支持持久化的NoSQL数据库，
> 也被称为数据结构服务器。

## redis特点：

1. 支持数据持久化
2. 支持多种数据类型，比如：list set zset hash string等数据结构的存储
3. 支持数据的备份，即master-slave模式的数据备份


## redis的基础介绍

### 1. 单进程

redis使用单进程模型来处理客户端的请求。对读写等事件的响应是通过对`epoll`函数的包装来做到的。
Redis的实际处理速度完全依靠主进程的执行效率。

`epoll`是Linux内核为处理大批量文件描述符而做了赶紧的`epoll`，是Linux下多路复用IO接口`select/poll`的增强版本，
它能显出提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率。

### 2.默认有16个数据库，0~15号库

下面是redis.conf的配置
```
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16

```

可以使用`SELECT`命令切换库，使用`dbsize` 查看当前库key的数量。默认是在0号库上。下面是演示库的切换。

```
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> set k1 v1
OK
127.0.0.1:6379[1]> keys *
1) "k1"
127.0.0.1:6379[1]> DBSIZE
(integer) 1
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> keys *
(empty list or set)

```

## 3. 清理数据库

`FLUSHDB`删除当前库的所有key

`FLUSHALL`删除所有库的key

## 4. 统一密码管理
16个库都是同样的密码，要么都能连上，要么一个也连不上。
