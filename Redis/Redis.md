Redis (Remote Dictionary Server)

## Install Redis in Linux

1. 下载redis. https://redis.io/

2. 解压安装包.程序放到opt目录下.  tar -zxvf <filename>

3. 进入redis的解压目录

4. 基本的环境安装

   ```shell
   yum install gcc-c++
   gcc -v  # 查看gcc 版本
   make  	# 编译
   make install # 安装(可选)
   
   
   # 编译
   [root@venhjuhost redis-6.2.6]# make
   cd src && make all
   make[1]: Entering directory '/opt/redis-6.2.6/src'
       CC Makefile.dep
   
   Hint: It's a good idea to run 'make test' ;)
   
   make[1]: Leaving directory '/opt/redis-6.2.6/src'
   
   # 安装
   [root@venhjuhost redis-6.2.6]# make install
   cd src && make install
   make[1]: Entering directory '/opt/redis-6.2.6/src'
   
   Hint: It's a good idea to run 'make test' ;)
   
       INSTALL redis-server
       INSTALL redis-benchmark
       INSTALL redis-cli
   make[1]: Leaving directory '/opt/redis-6.2.6/src'
   ```
   
   
   
5. redis的默认安装路径

   ```shell
   # redis 默认安装路径 /usr/local/bin
   [root@venhjuhost bin]# ls -l|grep redis
   -rwxr-xr-x 1 root root  6549120 Oct 29 09:24 redis-benchmark
   lrwxrwxrwx 1 root root       12 Oct 29 09:24 redis-check-aof -> redis-server
   lrwxrwxrwx 1 root root       12 Oct 29 09:24 redis-check-rdb -> redis-server
   -rwxr-xr-x 1 root root  6765632 Oct 29 09:24 redis-cli
   lrwxrwxrwx 1 root root       12 Oct 29 09:24 redis-sentinel -> redis-server
   -rwxr-xr-x 1 root root 12148696 Oct 29 09:24 redis-server
   ```

   

6. 将redis配置文件.复制到我们当前目录.

   ```shell
   [root@venhjuhost bin]# mkdir myconfig
   [root@venhjuhost bin]# cp /opt/redis-6.2.6/redis.conf ./myconfig/
   [root@venhjuhost bin]# ls myconfig/
   redis.conf
   ```

   

7. redis默认不是后台启动的.修改配置文件.将redis改为后台运行.(/usr/local/bin/myconfig)

   ![image-20211029110835716](Redis_images/image-20211029110835716.png)

8. 启动redis服务

   ```shell
   # 通过配置文件启动redis-server
   [root@venhjuhost bin]# pwd
   /usr/local/bin
   [root@venhjuhost bin]# redis-server myconfig/redis.conf
   
   # 启动client
   [root@venhjuhost bin]# redis-cli -p 6379    # -p 端口 -h host(默认本机)
   127.0.0.1:6379>
   127.0.0.1:6379> ping
   PONG                                        # 链接成功
   
   127.0.0.1:6379> set name peter              # 存入
   OK
   127.0.0.1:6379> get name
   "peter"
   
   127.0.0.1:6379> keys *                      # 查询所有key
   1) "name"
   ```

   

9. 查看redis的进程是否开启

   ```shell
   [root@venhjuhost ~]# ps -ef|grep redis
   root       36907       1  0 11:12 ?        00:00:00 redis-server 127.0.0.1:6379
   root       36948   36598  0 11:21 pts/0    00:00:00 redis-cli -p 6379
   root       37008   36958  0 11:24 pts/1    00:00:00 grep --color=auto redis
   
   ```
   
   
   
10. 关闭redis服务. shutdown

    ```shell
    # 关闭 redis服务
    127.0.0.1:6379> shutdown
    not connected> exit
    [root@venhjuhost bin]#
    
    # redis服务已关闭
    [root@venhjuhost ~]# ps -ef|grep redis
    root       37021   36958  0 11:28 pts/1    00:00:00 grep --color=auto redis
    ```

    

## 性能测试 redis-benchmark

```shell
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]

 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 2)
 --dbnum <db>       SELECT the specified db number (default 0)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
 

```

测试:

```shell
# 测试100个并发, 100000个请求
redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000

====== SET ======
  100000 requests completed in 2.64 seconds             # 100000 个请求
  100 parallel clients								 # 100 个并发
  3 bytes payload                                       # 每个请求3个byte
  keep alive: 1                                         # 只有一个链接(单机)
  host configuration "save": 3600 1 300 100 60 10000
  host configuration "appendonly": no
  multi-thread: no

Latency by percentile distribution:
0.000% <= 0.823 milliseconds (cumulative count 3)
50.000% <= 1.671 milliseconds (cumulative count 50424)
75.000% <= 1.999 milliseconds (cumulative count 75042)
87.500% <= 2.207 milliseconds (cumulative count 87638)

Summary:
  throughput summary: 37921.88 requests per second      # 每秒处理 37921.88个请求
  latency summary (msec):
          avg       min       p50       p95       p99       max
        2.000     0.816     1.671     2.591     3.607   272.127

```

## 基础知识

redis默认有16个数据库 (/usr/local/bin/myconfig/redis.conf). 默认使用的是第0个,可以使用select进行数据库切换.

```shell
# /usr/local/bin/myconfig/redis.conf
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16

```

### 一些基本命令

#### select  <db idx>

```shell
127.0.0.1:6379> select 3     # 切换数据库
OK
127.0.0.1:6379[3]> dbsize    # database size
(integer) 0
127.0.0.1:6379[3]>

```

#### keys

```shell
127.0.0.1:6379> keys *    # 查看keys
1) "name"
2) "myhash"
3) "counter:__rand_int__"
4) "mylist"
5) "key:__rand_int__"

```

#### flushdb/flushall

```shell
127.0.0.1:6379> flushdb   # 清空当前库 / flushall 清空全部数据库
OK
127.0.0.1:6379> keys *
(empty array)

```

### Redis是单线程

redis是很快的,官方表示,redis是基于内存操作的.CPU不是redis的性能瓶颈.redis的瓶颈是根据机器的内存和网络带宽.既然可以使用单线程来实现,就使用了单线程.

Redis使用C语言来写的,官方提供的数据是100000+的QPS,这个不比memecahce差! 

**Redis为什么单线程还那么快?**

误区1: 高性能的服务器一定是多线程的.

误区2:多线程一定比单线程效率高.

**核心:**Redis是将所有的数据放在内存中的,所以使用单线程操作效率最高.(多线程,CPU上下文切换,耗时的操作).对于内存系统来说,如果没有上下文切换,效率就是最高的.



## 五大数据类型

> Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache, and message broker. Redis provides data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes, and streams. Redis has built-in replication, Lua scripting, LRU eviction, transactions, and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

###  Redis-Key 

- keys *   										查看所有key
- set key                                         设置key值
- exists <key>                                判断key是否存在
- move <key> <db idx>              移动key到目标数据库
- expire <key> <prieod sec>    设置key过期时间(sec)
- ttl <key>                                      查看key剩余时间
- type <key>                                  查看key数据类型

```shell
127.0.0.1:6379> keys *           # 查看所有的key
(empty array)
127.0.0.1:6379> set age 30       # set key
OK
127.0.0.1:6379> set name peter
OK
127.0.0.1:6379> keys *           # 查看所有的key
1) "name"
2) "age"
127.0.0.1:6379> exists name      # 判断key是否存在
(integer) 1
127.0.0.1:6379> exists name1
(integer) 0

127.0.0.1:6379> move name 1      # 移动key到指定数据库
(integer) 1
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> keys *        # name移动到数据库1
1) "name"

127.0.0.1:6379> expire age 10    # 设置key过期时间(in sec)
(integer) 1
127.0.0.1:6379> ttl age          # 查看key的剩余时间
(integer) 3
127.0.0.1:6379> ttl age
(integer) -2
127.0.0.1:6379> get age          # 得到key的值
(nil)
127.0.0.1:6379>
127.0.0.1:6379> type name        # 查看key数据类型
string
127.0.0.1:6379> type age
string

```

### String类型

#### 使用场景

1. 计数器
2. 统计数量
3. 对象缓存存储

#### 常用命令

- EXISTS <key>                                判断key是否存在

- APPEND <key> <str>                  追加字符串,如果key不存在,相当与set key

  

- STRLEN <key>                              返回字符串长度

- INCR/DECR <key>                        自增/自减

- INCRBY/DECRBY <key> <increment decrement>   增/减步长

  

- GETRANGE <key> <start> <end>  范围获取string   [0,-1]获取整个字符串

- SETRANGE <key> <offset> <repl>  替换从offset开始的字符

  

- SETEX <key> <seconds> <value>     设置key value并且设置expire 时间(sec)

- SETNX  <key> <value>                        当key不存在时,设置key value值.(防止覆盖已经存在的key值)

  

- MSET     <key> <value> [<key> <value> ...]  设置多个key value

- MGET <key> [<key> ...]                                      得到多个key

- MSETNX <key> <value> [<key> <value> ...]  设置多个key value(key不存在时成功) 原子操作,必须全部成功.否则全部失败

  

- GETSET <key> <value>                                    先get,后set. 不存在,返回nil. 存在,返回值, 设置新的值.

```shell
# STRLEN,APPEN
127.0.0.1:6379> get name
"Han"
127.0.0.1:6379> STRLEN name    				# 返回字符串长度
(integer) 3
127.0.0.1:6379> APPEND name 'name,nihao'  	 # 追加字符串
(integer) 13
127.0.0.1:6379> get name
"Hanname,nihao"
127.0.0.1:6379> STRLEN name
(integer) 13
##################################################################################
# INCR/DECR,INCRBY/DECRBY
127.0.0.1:6379> INCR views				    # 自增
(integer) 1
127.0.0.1:6379> INCR views
(integer) 2
127.0.0.1:6379> DECR views				    # 自减
(integer) 1
127.0.0.1:6379> INCRBY views 10              # 增加10
(integer) 11
127.0.0.1:6379> DECRBY views 5               # 减少10
(integer) 6
##################################################################################
# GETRANGE,SETRANGE
127.0.0.1:6379> get name
"Hanname,nihao"
127.0.0.1:6379> GETRANGE name 0 -1			# 获取整个字符串
"Hanname,nihao"
127.0.0.1:6379> GETRANGE name 1 4			# 获取位置 [1,4]
"anna"

127.0.0.1:6379> get name
"Hanname,nihao"
127.0.0.1:6379> SETRANGE name 3 " Jun"      # 替换位置3后的字符串
(integer) 13
127.0.0.1:6379> get name
"Han Jun,nihao"
##################################################################################
# SETEX,SETNX
127.0.0.1:6379> SETEX age 30 30         # 设置 age, 并且30秒后失效
OK
127.0.0.1:6379> get age
"30"
127.0.0.1:6379> ttl age                 # 查看失效时间
(integer) 25
127.0.0.1:6379> SETNX color red         # 设置 color, 当color不存在,设置成功.返回 1
(integer) 1
127.0.0.1:6379> get color
"red"
127.0.0.1:6379> SETNX color black		# 设置 color, 当color存在,设置失败.返回 0
(integer) 0
127.0.0.1:6379> get color			   # color没有被替换
"red"
127.0.0.1:6379> get age                 # age 已经失效
(nil)
127.0.0.1:6379> ttl age                 # 查看age剩余时间,-2表示已经失效
(integer) -2
##################################################################################
# MSET,MGET,MSETNX
127.0.0.1:6379> MSET k1 v1 k2 v2 k3 v3   # 设置多个key
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"
127.0.0.1:6379> mget k1 k2 k3            # 得到多个key
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> MSETNX k1 vv1 k4 vv4 k5 vv5    # k1已经存在,设置失败
(integer) 0
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"
127.0.0.1:6379> MSET k4 vv4 k5 vv5             # k4,k5不存在,设置成功
OK
127.0.0.1:6379> keys *
1) "k1"
2) "k5"
3) "k2"
4) "k3"
5) "k4"
# 存一个对象

127.0.0.1:6379> mset user:1 {name:peter,age:18}
OK
127.0.0.1:6379> get user:1
"{name:peter,age:18}"
127.0.0.1:6379> mset user:1:name peter user:1:age 18
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "peter"
2) "18"
##################################################################################
# GETSET
127.0.0.1:6379> GETSET sex male            # 不存在,返回nil.设置male
(nil)
127.0.0.1:6379> GETSET sex female          # 存在,返回值.设置female
"male"
127.0.0.1:6379> get sex
"female"

```



### List类型

#### 使用场景

List实际上是一个链表 <before> node <after>

1. 栈  LPUSH,LPOP
2. 队列  LPUSH,RPOP
3. 阻塞队列

#### 常用命令

- LPUSH <key> <element> [<element>...]   添加list元素到列表头部（左边插入）

- LRANGE <key> <start> <stop>                获取list元素

- RPUSH <key> <element> [<element>...]  添加list元素到列表尾部（右边插入）

  

- LPOP key [count]                                            左边弹出

- RPOP key [count]                                           右边弹出

  

- LINDEX <key> <index>                               按index取值

- LLEN <key>                                                   获得列表长度

  

- LREM <key> <count> <element>            移除count个指定element

- LTRIM <key> <start> <stop>                  截取list

  

-  RPOPLPUSH <source> <dest>               从source RPOP 并 LPUSH到 dest

  

- LSET <key> <index> <element>            将列表中存在的元素更新。不存在将报错。

- LINSERT <key> <before|after> <pivot> <element>   往某个元素的前面或后面添加元素

  

```shell
# LPUSH,LRANGE,RPUSH
127.0.0.1:6379> LPUSH mylist L1 L2 L3  # 添加list元素到列表头部   L3,L2,L1 (列表左边为头部)
(integer) 3
127.0.0.1:6379> LRANGE mylist 0 -1     # 获取全部元素
1) "L3"
2) "L2"
3) "L1"
127.0.0.1:6379> LRANGE mylist 0 0      # 获取第0个元素
1) "L3"
127.0.0.1:6379> LRANGE mylist 0 1      # 获取[0,1]元素
1) "L3"
2) "L2"
127.0.0.1:6379> RPUSH mylist L4 L5     # 添加list元素到列表尾部 L3,L2,L1,L4,L5 (列表右边为尾部)
(integer) 5
127.0.0.1:6379> LRANGE mylist 0 -1
1) "L3"
2) "L2"
3) "L1"
4) "L4"
5) "L5"
127.0.0.1:6379> LRANGE mylist 3 4
1) "L4"
2) "L5"
##################################################################################
# LPOP,RPOP
127.0.0.1:6379> LRANGE mylist 0 -1
1) "L3"
2) "L2"
3) "L1"
4) "L4"
5) "L5"
127.0.0.1:6379> LPOP mylist   # 左弹出
"L3"
127.0.0.1:6379> RPOP mylist   # 右弹出
"L5"
127.0.0.1:6379> LRANGE mylist 0 -1
1) "L2"
2) "L1"
3) "L4"
##################################################################################
# LINDEX,LLEN
127.0.0.1:6379> LINDEX mylist 0   # 通过下标获得值
"L2"
127.0.0.1:6379> LINDEX mylist 1
"L1"
127.0.0.1:6379> LINDEX mylist 10  # 越界返回nil
(nil)

127.0.0.1:6379> LLEN mylist       # 获得列表长度
(integer) 3
##################################################################################
# LREM,LTRIM
127.0.0.1:6379> LRANGE mylist 0 -1
1) "L2"
2) "L1"
3) "L4"
127.0.0.1:6379> LPUSH mylist L1 L2
(integer) 5
127.0.0.1:6379> LRANGE mylist 0 -1
1) "L2"
2) "L1"
3) "L2"
4) "L1"
5) "L4"
127.0.0.1:6379> LREM mylist 2 L1            # 移除2个L1
(integer) 2
127.0.0.1:6379> LRANGE mylist 0 -1
1) "L2"
2) "L2"
3) "L4"
127.0.0.1:6379> LREM mylist 1 L2           # 移除1个L2
(integer) 1
127.0.0.1:6379> LRANGE mylist 0 -1
1) "L2"
2) "L4"

127.0.0.1:6379> LRANGE mylist 0 -1
1) "L2"
2) "L4"
3) "L5"
4) "L6"
5) "L7"
127.0.0.1:6379> LTRIM mylist 1 3          # 截取 mylist中的[1,3], index 从0开始
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "L4"
2) "L5"
3) "L6"
##################################################################################
# RPOPLPUSH 
127.0.0.1:6379> LRANGE l1 0 -1
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> RPOPLPUSH l1 l2          # 从l1中 RPOP 并 LPUSH到 L2
"v3"
127.0.0.1:6379> LRANGE l2 0 -1
1) "v3"
##################################################################################
# LSET
127.0.0.1:6379> LRANGE l1 0 -1
1) "v1"
2) "v2"
127.0.0.1:6379> LSET l1 0 vv1            # 将 l1中 idx=0的元素更新为vv1
OK
127.0.0.1:6379> LSET l1 3 vv3            # l1中不存在 idx=3的元素，更新失败
(error) ERR index out of range
127.0.0.1:6379> LRANGE l1 0 -1
1) "vv1"
2) "v2"
##################################################################################
# LINSERT
127.0.0.1:6379> lrange l1 0 -1
1) "v3"
2) "v2"
3) "v1"
127.0.0.1:6379> LINSERT l1 before v2 ------   # 在 v2 前插入 ------
(integer) 4
127.0.0.1:6379> LINSERT l1 after v2 -------   # 在 v2 后插入 ------
(integer) 5
127.0.0.1:6379> lrange l1 0 -1
1) "v3"
2) "------"
3) "v2"
4) "-------"
5) "v1"

```

### Set类型

Set中的值是不能重复的。

#### 常用命令

- SADD <key> <member> [<member>...]            往set中添加元素

- SREM <key> <member> [<member>...]           移除set集合中的指定元素

- SMEMBERS <key>                                           查看set中所有member

- SISMEMBER <key> <member>                        查看member是否存在于set中

- SCARD <key>                                                   获取set中元素个数

  

- SRANDMEMBER <key> <count>                   从set中随机获取count个元素

- SPOP <key> <count>                                     从set中随机移除count个元素

  

- SMOVE <source> <dest> <member>           将元素从source移动到dest

  

- SDIFF <key> [<key>...]                                   差集

- SINTER <key> [<key>...]                                交集

- SUNION <key> [<key>...]                               并集

  

  

```shell
# SADD,SREM,SMEMBER,SISMEMBER,SCARD
127.0.0.1:6379> SADD set S1 S2 S3       # 添加set元素
(integer) 3
127.0.0.1:6379> SMEMBERS set            # 查看set元素
1) "S2"
2) "S1"
3) "S3"
127.0.0.1:6379> SISMEMBER set S1       # S1 存在于set
(integer) 1
127.0.0.1:6379> SISMEMBER set S5       # S5 不存在于set
(integer) 0
127.0.0.1:6379> SCARD set              # 获取set中元素的个数
(integer) 3
127.0.0.1:6379> sadd set S4
(integer) 1
127.0.0.1:6379> SCARD set
(integer) 4
127.0.0.1:6379> SREM set S4            # 移除 S4
(integer) 1
127.0.0.1:6379> SCARD set
(integer) 3
127.0.0.1:6379> SMEMBERS set
1) "S2"
2) "S1"
3) "S3"
##################################################################################
# SRANDMEMBER,SPOP
127.0.0.1:6379> SRANDMEMBER set 1   # 从 set中随机获取一个
1) "S3"
127.0.0.1:6379> SRANDMEMBER set 1
1) "S1"
127.0.0.1:6379> SRANDMEMBER set 2   # 从 set中随机获取2个
1) "S4"
2) "S6"
127.0.0.1:6379> spop set 1          # 从 set中随机移除一个元素
1) "S4"
127.0.0.1:6379> spop set 1          # 从 set中随机移除一个元素
1) "S1"
##################################################################################
# SMOVE
127.0.0.1:6379> sadd s1 s1 s2 s3   
(integer) 3
127.0.0.1:6379> smove s1 s2 s1     # 将s1 从 集合s1移动到集合s2
(integer) 1
127.0.0.1:6379> SMEMBERS s1
1) "s3"
2) "s2"
127.0.0.1:6379> SMEMBERS s2
1) "s1"
##################################################################################
# SDIFF,SINTER,SUNION
127.0.0.1:6379> SADD set1 L1 L2 L3 L4 L5
(integer) 5
127.0.0.1:6379> SADD set2 L4 L5 L6 L7 L8
(integer) 5
127.0.0.1:6379> SADD set3 L1 L4 L5 L9 L10
(integer) 5
127.0.0.1:6379> sdiff set1 set2 set3             # 差集，set1中，不存在于set2,set3的元素
1) "L2"
2) "L3"
127.0.0.1:6379> sinter set1 set2 set3            # 交集
1) "L4"
2) "L5"
127.0.0.1:6379> sunion set1 set2 set3            # 并集
 1) "L3"
 2) "L6"
 3) "L10"
 4) "L8"
 5) "L1"
 6) "L7"
 7) "L5"
 8) "L9"
 9) "L2"
10) "L4"

```

### Hash类型

Map集合 key - map.值是map集合. Hash更适合对象的存储。

#### 常用命令

- HSET <key> <field><value>[<field><value>...]     设置hash map

- HDEL <key> <field><value>[<field><value>...]     删除field

- HGET <key> <field>                                                       得到field 的value值

- HGETALL <key>                                                                 得到所有的field - value值

  

- HLEN <key>                                                                        得到hash中键值对的个数

- HMSET <key> <field><value>[<field><value>...]  设置hash map

- HMGET <key> <field><value>[<field><value>...]  得到多个key的值

- HEXISTS <key> <field>                                                  判断hash中是否存在field

  

- HKEYS <key>                                                                     得到所有field

- HVALS <key>                                                                      得到所有value

  

- HINCRBY <key> <field> <increment>                       增加field by increment

- HSETNX  <key> <field> <value>                                 当field不存在时，设置field

```shell
# HSET,HGET,HGETALL
127.0.0.1:6379> HSET mymap name1 peter name2 "han jun" name3 "gao shuai"   # 设置hash值
(integer) 3
127.0.0.1:6379> HGET mymap name2    # 得到 name2的值
"han jun"
127.0.0.1:6379> HGET mymap name3
"gao shuai"
127.0.0.1:6379> HGETALL mymap       # 得到所有hash值
1) "name1"
2) "peter"
3) "name2"
4) "han jun"
5) "name3"
6) "gao shuai"
127.0.0.1:6379> HDEL mymap name2    # 删除 name2
(integer) 1
127.0.0.1:6379> HGETALL mymap
1) "name1"
2) "peter"
3) "name3"
4) "gao shuai"
##################################################################################
# HLEN,HMSET,HMGET,HEXISTS
127.0.0.1:6379> HMSET mm K1 V1 K2 V2 K3 V3 K4 V4   # 设置 hash
OK
127.0.0.1:6379> HMGET mm K1 K2 K3 K4               # 得到多个k的值
1) "V1"
2) "V2"
3) "V3"
4) "V4"
127.0.0.1:6379> HLEN mm                            # 得到hash中键值对的个数
(integer) 4
127.0.0.1:6379> HEXISTS mm K1                      # 判断map中是否存在 K1
(integer) 1
127.0.0.1:6379> HEXISTS mm K6
(integer) 0
##################################################################################
# HKEYS,HVALS
127.0.0.1:6379> HKEYS mm
1) "K1"
2) "K2"
3) "K3"
4) "K4"
127.0.0.1:6379> HVALS mm
1) "V1"
2) "V2"
3) "V3"
4) "V4"
##################################################################################
# HINCRBY,HSETNX
127.0.0.1:6379> HSET mm k1 1 k2 2
(integer) 2
127.0.0.1:6379> HINCRBY mm k1 3      # k1 增加 3
(integer) 4
127.0.0.1:6379> HSETNX mm k3 3       # k3不存在，set k3
(integer) 1
127.0.0.1:6379> HKEYS mm
1) "k1"
2) "k2"
3) "k3"
127.0.0.1:6379> HSETNX mm k3 4      # k3存在，set k3失败
(integer) 0

```

### Zset类型（有序结合）

在set的基础上，增加了一个值，用来排序

#### 常用命令

- ZADD <key> [NX|XX] [GT|LT] [CH] [INCR] <score> <member> [<score><member>...]  增加

- ZRANGE <ley><min><max> [BYSCORE|BYLEX] [REV] [LIMITOFFSET COUNT]           显示

- ZREVRANGE / ZRANGEBYSCORE / ZRANGEBYLEX

  

- ZREM <key><member>[<member>...]                 移除元素

- ZCARD  <key>                                                      获取元素个数

- ZCOUNT <key> <min> <max>                             统计区间个数      (min   开区间      

```shell
# ZADD,ZRANGE
127.0.0.1:6379> ZADD myz 1 one 2 two 3 three 4 four  # 添加多个值
(integer) 4
127.0.0.1:6379> ZRANGE myz 0 4   # 显示全部值
1) "one"
2) "two"
3) "three" 
4) "four"

##################################################################################
# 默认排序
127.0.0.1:6379> ZADD salary 1000 p1 3000 p3 4000 p4 5000 p5 2000 p2  # 添加 Zset 元素
(integer) 5
127.0.0.1:6379> ZRANGE salary 0 -1   # 显示全部元素 (默认排序by score升序)
1) "p1"
2) "p2"
3) "p3"
4) "p4"
5) "p5"
127.0.0.1:6379> ZRANGE salary 0 -1 REV # 显示全部元素（by score降序）
1) "p5"
2) "p4"
3) "p3"
4) "p2"
5) "p1"
127.0.0.1:6379> ZRANGE salary 2000 3000 BYSCORE # 显示 [2000,3000]的元素
1) "p2"
2) "p3"
127.0.0.1:6379> ZRANGE salary (2000 3000 BYSCORE # 显示 (2000,3000]的元素
1) "p3"


127.0.0.1:6379> zadd scores 100 peter 98 gao 90 shuai 60 mao 59 hans  
(integer) 5
127.0.0.1:6379> zrange scores -inf +inf byscore  # -inf 负无穷 +inf 正无穷
1) "hans"
2) "mao"
3) "shuai"
4) "gao"
5) "peter"
127.0.0.1:6379> zrange scores -inf +inf byscore LIMIT 0 2  # 显示 offset=0,count=2 前2个
1) "hans"
2) "mao"
##################################################################################
# ZREM,ZCARD,ZCOUNT
127.0.0.1:6379> zrange salary 0 -1
1) "peter"
2) "han"
3) "mao"
4) "shuai"
5) "gao"
6) "chen"
127.0.0.1:6379> zrem salary chen    # 删除 chen 元素
(integer) 1
127.0.0.1:6379> zrange salary 0 -1  
1) "peter"
2) "han"
3) "mao"
4) "shuai"
5) "gao"
127.0.0.1:6379> zcard salary      # 显示元素个数
(integer) 5
127.0.0.1:6379> zcount salary 1000 3000   # 统计[1000,3000]的元素个数
(integer) 3
127.0.0.1:6379> zcount salary (1000 3000  # 统计(1000,3000]的元素个数
(integer) 2
127.0.0.1:6379> zcount salary (1000 (3000 # 统计(1000,3000)的元素个数
(integer) 1
```

## 三种特殊数据类型

### geospatial类型

#### 使用场景

底层原理： 是Zset,可以使用Zset命令来操作。

地理位置，朋友定位，附近的人，打车距离计算。

Redis的Geo在Redis3.2的版本推出了，这个功能可以推算地理位置的信息，两地之间的距离，方圆几里的人。

查询城市经纬度： http://www.jsons.cn/lngcode/

#### 常用命令

- GEOADD  key [NX|XX] [CH] longitude latitude member [longitude latitude member ...] 添加地理位置

- GEOPOS key member [member ...]                     查询地理位置

- GEODIST key member1 member2 [m|km|ft|mi]   查询两个地点的距离

- GEOHASH key member [member ...]                   返回元素坐标经纬度的11位的hash字符串，原长度为52位。这里会损失一些经度。

  ​                                                                              但仍然指向同一地区。

- GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC|DESC] [STORE key] [STOREDIST key]                           返回给定坐标半径内的地理位置

- GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC|DESC] [STORE key] [STOREDIST key]     返回给定元素半径内的地理位置

- GEOSEARCH                                                       替换GEORADIUS,GEORADIUSBYMEMBER

  ​																			  https://redis.io/commands/geosearch                               

- GEOSEARCHSTORE                                           替换GEORADIUS,GEORADIUSBYMEMBER 

  ​                                                                              https://redis.io/commands/geosearchstore

```shell
# GEOADD
# 地球两极无法直接添加
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai    # 添加地理位置
(integer) 1
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 113.28 23.12 guangzhou
(integer) 1
127.0.0.1:6379> geoadd china:city 114.08 22.54 shenzhen
(integer) 1
127.0.0.1:6379> geoadd china:city 120.61 31.29 suzhou
(integer) 1
127.0.0.1:6379> geoadd china:city 114.49 36.61 handan
(integer) 1
##################################################################################
# GEOPOS
127.0.0.1:6379> GEOPOS china:city beijing shanghai    # 获取指定的地理位置 经度，纬度
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
2) 1) "121.47000163793563843"
   2) "31.22999903975783553"
##################################################################################
127.0.0.1:6379> GEODIST china:city shanghai suzhou km   # 上海，苏州的距离(km)
"82.0395"
127.0.0.1:6379> GEODIST china:city shanghai beijing km
"1067.3788"
##################################################################################
# GEORADIUS
127.0.0.1:6379> GEORADIUS china:city 120 30 300 km    # 以120,30为中心，300km内的城市
1) "suzhou"
2) "shanghai"
127.0.0.1:6379> GEORADIUS china:city 120 30 2000 km WITHCOORD WITHDIST COUNT 3   # 以120，30为中心，2000km以内
1) 1) "suzhou"                                                                   # WITHCOORD 显示坐标
   2) "154.9004"                                                                 # WITHDIST 显示距离
   3) 1) "120.60999959707260132"                                                 # COUNT 3 从近到远（升序）前3
      2) "31.29000095904170564"
2) 1) "shanghai"
   2) "196.2512"
   3) 1) "121.47000163793563843"
      2) "31.22999903975783553"
3) 1) "handan"
   2) "895.6274"
   3) 1) "114.48999792337417603"
      2) "36.61000046432025812"
##################################################################################
# GEORADIUSBYMEMBER
127.0.0.1:6379> GEORADIUSBYMEMBER china:city shanghai 500 km WITHDIST    # 上海半径500km内的城市
1) 1) "suzhou"
   2) "82.0395"
2) 1) "shanghai"
   2) "0.0000"
##################################################################################
# GEOHASH
127.0.0.1:6379> GEOHASH china:city beijing shanghai    # 返回经纬度，11长度的hash字符串
1) "wx4fbxxfke0"                                       # 将二维经纬度，转换为一维字符串
2) "wtw3sj5zbj0"
##################################################################################
# 使用Zset命令来操作
127.0.0.1:6379> ZRANGE china:city 0 -1
1) "nanjing"
2) "shenzhen"
3) "guangzhou"
4) "suzhou"
5) "shanghai"
6) "handan"
7) "beijing"
127.0.0.1:6379> ZREM china:city nanjing    # 移除nanjing
(integer) 1
127.0.0.1:6379> ZRANGE china:city 0 -1
1) "shenzhen"
2) "guangzhou"
3) "suzhou"
4) "shanghai"
5) "handan"
6) "beijing"

```

###  Hyperloglog类型

#### 基数统计

基数统计通常用来统计一个集合中不重复的元素个数。例如统计某个网站的访问人数，或者用户搜索网站的关键词数量。数据分析、网络监控及数据库优化等领域都会涉及到基数计数的需求。

传统的方式是通过set来保存用户id,然后计算set中用户个数来作为判断。但是保留了大量用户id,浪费内存。

基数统计用于解决这个问题。目前大多使用hyperloglog方法来实现：

> 关于hyperloglog， wiki的[hyperloglog](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/HyperLogLog)词条给出了以下解释：
>
> HyperLogLog算法的基础是观察到可以通过计算集合中每个数字的二进制表示中的前导零的最大数目来估计均匀分布的随机数的多重集的基数。如果观察到的前导零的最大数目是n，则集合中不同元素的数量的估计是2^n
> 在HyperLogLog算法中，将哈希函数应用于原始多集中的每个元素，以获得具有与原始多集相同基数的均匀分布的随机数的多集。然后可以使用上述算法来估计该随机分布集合的基数。
> 使用上述算法获得的基数的简单估计具有很大差异的缺点。在HyperLogLog算法中，通过将多集合分成多个子集，计算这些子集中每个子集中的数字中的前导零的最大数量，并使用调和平均数 Harmonic mean将每个子集的这些估计值合并为全集的基数。

hyperloglog算法带来的好处是：

1，利用尽可能少的内存空间来实现大数据集的基数统计实现。2^64元素的基数统计，只需要12KB的内存。

2，使用随机设定的bias来提高整体的计算精度。 0.81%的错误率。

其劣势也很显然：

1，由于使用随机的bias使得在小数据集的情况下导致的计算错误率太大

2，针对超大数据集因为使用了连续空间分配策略导致内存分配和回收不合理而效率低下。（超大数据集为>=1billion多重集的基数）。

#### 常用命令

- PFADD key [element [element ...]]                        添加元素
- PFCOUNT key [key ...]                                          基数统计
- PFMERGE destkey sourcekey [sourcekey ...]      合并集合

```shell
# PFADD,PFCOUNT,PFMERGE
127.0.0.1:6379> PFADD views p1 p2 p3 p3 p4 p4 p5 p6 p6 p7 p7 p7       # 添加元素
(integer) 1
127.0.0.1:6379> PFCOUNT views                                         # 基数统计
(integer) 7
127.0.0.1:6379> PFADD views2 p7 p7 p8 p8 p9 p10 p10 p11 p12 p13 p14 p14 p15  # 添加元素
(integer) 1
127.0.0.1:6379> PFCOUNT views2                                        # 基数统计
(integer) 9
127.0.0.1:6379> PFMERGE view3 views views2                            # 合并集合                  
OK
127.0.0.1:6379> PFCOUNT view3                                         # 查看并集基数统计
(integer) 15

```

### Bitmaps类型

bitmap 存储的是连续的二进制数字（0 和 1），通过 bitmap, 只需要一个 bit 位来表示某个元素对应的值或者状态，key 就是对应元素本身 。我们知道 8 个 bit 可以组成一个 byte，所以 bitmap 本身会极大的节省储存空间。

#### 使用场景

适合需要保存状态信息（比如是否签到、是否登录...）并需要进一步对这些信息进行分析的场景。比如用户签到情况、活跃用户情况、用户行为统计（比如是否点赞过某个视频）。

#### 常用命令

- SETBIT key offset value         设置bitmap

- GETBIT key offset                  查看bitmap

  

- BITCOUNT key [start end]     Count the number of set bits (population counting) in a string. 

  ​                                               这里start end是指的byte.[1,3]表示下标从1-3的byte

```shell
# SETBIT,GETBIT
127.0.0.1:6379> SETBIT sign 0 1    # 设置一周签到状态
(integer) 0
127.0.0.1:6379> SETBIT sign 1 0
(integer) 0
127.0.0.1:6379> SETBIT sign 2 1
(integer) 0
127.0.0.1:6379> SETBIT sign 3 1
(integer) 0
127.0.0.1:6379> SETBIT sign 4 0
(integer) 0
127.0.0.1:6379> SETBIT sign 5 1
(integer) 0
127.0.0.1:6379> SETBIT sign 6 1
(integer) 0

127.0.0.1:6379> GETBIT sign 4       # 查看bitmap
(integer) 0
127.0.0.1:6379> GETBIT sign 5
(integer) 1
##################################################################################
# BITCOUNT
127.0.0.1:6379> BITCOUNT sign       # 统计sign中1的位数
(integer) 5

```

