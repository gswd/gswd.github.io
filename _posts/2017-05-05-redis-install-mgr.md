---
layout: post
title:  "2. Linux下安装redis以及简单管理"
categories: Redis笔记
tags:  Redis
excerpt: redis在linux下安装、错误处理以及简单的管理命令
---

* content
{:toc}


## 安装Redis

1. 下载redis安装包

[http://download.redis.io/releases/redis-3.2.8.tar.gz](http://note.youdao.com/)

2. 将压缩包拷贝到/opt目录下并且解压缩

```
[root@hm707 redis-3.2.8]# tar -zxvf redis-3.2.8.tar.gz
```

3. 进入到解压缩后的目录中，执行`make`命令

注意：执行`make`时需要有gcc环境，如果没有需要提前安装

```
[root@hm707 redis-3.2.8]# yum install gcc-c++ -y
```

再次执行`make`时还是会报错，错误如下

```
[root@hm707 redis-3.2.8]# make
cd src && make all
make[1]: Entering directory `/opt/redis-3.2.8/src'
    CC adlist.o
In file included from adlist.c:34:
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
zmalloc.h:55:2: error: #error "Newer version of jemalloc required"
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/opt/redis-3.2.8/src'
make: *** [all] Error 2
```
该错误是因为之前在没有gcc环境时执行make命令后生成了一些文件目录导致的，只需要执行`make distclean`之后在执行`make`

4. 最后执行`make install`

```
[root@hm707 redis-3.2.8]# make install
cd src && make install
make[1]: Entering directory `/opt/redis-3.2.8/src'

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install

```

## 简单管理

1. 目录的简单说明

安装完成后的可执行文件在`/usr/local/bin`目录下

```
[root@hm707 redis-3.2.8]# cd /usr/local/bin
[root@hm707 bin]# ll
total 26348
-rwxr-xr-x. 1 root root 5580327 Apr 27 23:09 redis-benchmark
-rwxr-xr-x. 1 root root   22217 Apr 27 23:09 redis-check-aof
-rwxr-xr-x. 1 root root 7829986 Apr 27 23:09 redis-check-rdb
-rwxr-xr-x. 1 root root 5709187 Apr 27 23:09 redis-cli
lrwxrwxrwx. 1 root root      12 Apr 27 23:09 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 7829986 Apr 27 23:09 redis-server
```

修改配置文件，让redis可以后台启动

```
[root@hm707 bin]# pwd
/opt/redis-3.2.8
[root@hm707 bin]# mkdir /myredis
[root@hm707 bin]# cp redis.conf /myredis/
[root@hm707 bin]# vim /myredis/redis.conf

```
daemonize改为yes

```
################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes

```

启动redis服务

```
[root@hm707 bin]# pwd
/usr/local/bin
[root@hm707 bin]# redis-server /myredis/redis.conf
[root@hm707 bin]# ps -ef | grep redis
root      6048     1  0 23:26 ?        00:00:00 redis-server 127.0.0.1:6379
root      6054  2438  0 23:27 pts/1    00:00:00 grep redis

```

使用`redis-cli -p 6379`连接redis服务.
`SHUTDOWN`关闭服务

```
[root@hm707 bin]# pwd
/usr/local/bin
[root@hm707 bin]# redis-cli -p 6379
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> get k1
"hello"
127.0.0.1:6379> SHUTDOWN
not connected> exit
[root@hm707 bin]# ps -ef|grep redis
root      6075  2438  0 23:31 pts/1    00:00:00 grep redis

```

使用`Redis-benchmark`测试redis性能

下面是简单的例子，具体参考 [http://www.redis.cn/topics/benchmarks.html](http://www.redis.cn/topics/benchmarks.html)

```
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]

-h <hostname>      Server hostname (default 127.0.0.1)
-p <port>          Server port (default 6379)
-s <socket>        Server socket (overrides host and port)
-c <clients>       Number of parallel connections (default 50)
-n <requests>      Total number of requests (default 10000)
-d <size>          Data size of SET/GET value in bytes (default 2)
-k <boolean>       1=keep alive 0=reconnect (default 1)
-r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
  Using this option the benchmark will get/set keys
  in the form mykey_rand:000000012456 instead of constant
  keys, the <keyspacelen> argument determines the max
  number of values for the random number. For instance
  if set to 10 only rand:000000000000 - rand:000000000009
  range will be allowed.
-P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
-q                 Quiet. Just show query/sec values 只显示每秒钟能处理多少请求数结果
--csv              Output in CSV format
-l                 Loop. Run the tests forever 永久测试
-t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
-I                 Idle mode. Just open N idle connections and wait.
```

```
//SET/GET 100 bytes 检测host为127.0.0.1 端口为6379的redis服务器性能
redis-benchmark -h 127.0.0.1 -p 6379 -q -d 100

//5000个并发连接，100000个请求，检测host为127.0.0.1 端口为6379的redis服务器性能
redis-benchmark -h 127.0.0.1 -p 6379 -c 5000 -n 100000
```
