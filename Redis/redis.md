# Redis

## NoSQL数据库简介

分布式服务器的Session问题:客户端与服务器1建立Session,然后再次访问服务器2如何得到对应的Session

1. 客户端缓存客户信息到Cookie   缺点是客户端缓存的安全性降低
2. 将Session复制到所有的负载均衡的服务器中.   缺点是空间利用率非常低
3. 将Session存储到服务器或数据库中  缺点是IO次数增加,效率降低
4. 将Session存储到NoSQL数据库中  可以将信息存储到内存中,避免IO

因此NoSQL可以解决**分布式服务器的Session问题**,同时NoSQL数据库也可以作为缓存解决数据库中数据过大的问题,可以将频繁读取的数据存储到NoSQL数据库中,提高访问速度,**减少IO操作**

### NoSQL的特点

1. NoSQL(Not Only SQL):不仅仅是SQL,泛指非关系型数据库.
2. NoSQL不依赖业务逻辑方式存储,而以简单的key-value模式存储,增加了数据库的扩展能力
3. NoSQL不遵循SQL标准
4. NoSQL不支持ACID
5. NoSQL远超于SQL的性能

### NoSQL适用场景

1. 对数据的高并发读写
2. 海量数据的读写
3. 要求实现高扩展性

### NoSQL不适用场景

1. 需要事务支持
2. 基于SQL的结构化存储查询.即当不需要使用SQL或使用SQL无法达到效果的情况下可以使用NoSQL

### 常见的NoSQL数据库 

1. Memcache
   1. 数据都存储在内存中,无法持久化
   2. 只支持简单的key-value模式(字符串)
2. Redis
   1. 覆盖了Memcache的所有功能
   2. 支持持久化
   3. 支持多种数据类型
3. MongoDB
   1. 属于高性能,开源,模式自由的文档型数据库

## Redis6概述和安装

### Redis6的安装

1. 官网下载安装Redis:https://redis.io/download/

2. 打开xshell,将redis的压缩包拖拽到某个目录下

3. 下载gcc环境(可以通过gcc --version来看一下是否已经下载过gcc):yum install gcc

   ![1665922198083](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1665922198083.png)

4. 解压redis压缩包:tar -zxvf redis-6.2.7.tar.gz

5. 编译完成后执行make install

6. 文件会默认下载到/usr/local/bin目录下![1665922407341](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1665922407341.png)

   redis-benchmark:性能测试工具

   redis-check-rdb:修复有问题的rdb文件

   redis-check-aof:修复有问题的aof文件

   reids-sentinel:Redis集群使用

   **redis-server:Redis服务器启动命令**:相当于前台启动(不推荐,随着窗口的关闭Redis就会关闭)

### Redis的启动

1. 前台启动:redis-server.  不推荐,Redis会随着窗口的关闭而关闭

   ![1665922651639](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1665922651639.png)

2. 后台启动

   1. 找到redis.conf文件
   2. 将redis.conf文件复制到其他路径下 : cp redis.conf /etc/redis.conf   (将redis.conf文件复制到etc路径下)
   3. 进入到etc路径下的redis.conf文件,将文件中的daemonize no 改为 daemonize yes(允许后天启动)
   4. 进入到/usr/local/bin目录下,使用redis-server /etc/redis.conf命令启动redis
   5. 通过ps -ef | grep redis命令查看redis的端口占用情况![1665923397752](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1665923397752.png)
   6. 通过redis-cli命令使客户端连接上redis
   7. 可以通过redis-cli shutdown或直接kill -9 (redis的进程号)来关闭redis

### Redis相关知识

1. Redis的默认占用端口号为6379
2. Redis底层是单线程+多路IO复用
3. Redis支持更多的数据类型
4. Redis支持持久化

## 常用五大数据类型

### key的操作

1. keys *  // 查看数据库的所有的key值
2. exists key值 // 查看key值是否存在(存在为1,不存在为0)
3. type key值 // 查看key值的类型
4. del key值 // 删除key值 直接删除key值
5. unlink key值 // 根据value选择非阻塞删除.即当前keyspace中没有了该key值,但实际上并没有删除,而是在之后合适的实际删除
6. expire key time // 给key值设定指定有效期time
7. ttl key // 查看key值还有多长时间过期. -1为永不过期,-2为已过期
8. select number // 切换到number号库
9. dbsize // 查看当前库中有多少个数量的key
10. flushdb // 清空当前库
11. flushall // 清空所有库

### string

string类型是二进制安全的,即string中可以包含任意类型的数据.在Redis中,一个string类型的value最大为512M

常用命令

1. set key value //添加<key,value>键值对.注意:相同的key后者会覆盖前者
2. get key // 得到key对应的value值
3. append key str // 将str添加到key对应的value的最后
4. strlen k1 // 获取k1对应的value的长度
5. setnx key value // 只有当key不存在时设置key的值
6. incr key // 当key对应的value为数值型时自动加1
7. decr key // 当key对应的value为数值型时自动减1
8. incrby/decrby key val // 为key对应的数值型value加/减val
9. mset key1 value1 key2 value2 // 同时设置多个键值对
10. mget key1 key2 // 同时获取多个key对应的value
11. msetnx key1 value1 key2 value2 // 如果有任意一个key存在,则该语句失效
12. getrange key from to // 获取到key对应的value的[from,to]的子串
13. setrange key 起始位置 value // 从key对应的value的起始位置开始将之后长度为value的子串改为value
14. setex key 过期时间 value // 在创建键值对时就设置键值对的过期时间
15. getset key value // 取出key对应的旧的value值,并将key的value值设置为新的value值

string的数据结构为简单的动态字符串(二倍扩容,当字符串长度超过1M,每次只会扩容1M,最大长度为512M)'

### list

redis中list是一个双向链表

常用命令

1. lpush k1 v1 v2 v3 // 从左到右向列表k1中放入三个元素(头插法)
2. rpush k1 v1 v2 v3 // 从右到左向列表k1中放入三个元素(尾插法)
3. lrange k1 from to // 从k1中取出from到to的元素,[0,-1] 表示取出list中的所有元素
4. lpop/rpop k1 // 从左/右取出k1中的一个元素(pop消除)
5. rpoplpush k1 k2 // 将k1中最后一个元素取出放入到k2的头部
6. lindex k1 index //取出k1中下标为index的元素
7. llen k1 // 获得k1的长度
8. linsert k1 before/after v1 newv1 // 在k1的v1的前面/后面插入一个新的元素newv1
9. lrem k1 n v1 // 从左到右删除k1中的n个v1
10. lset k1 index v1 // 将k1中下标为index的元素改为v1 

### set

redis中set是一个哈希表,无序存储非重复的元素

常用命令

1. sadd k1 v1 v2 v3 // 向集合k1中存储3个元素
2. smembers k1 // 取出k1中的所有元素
3. sismember k1 v1 // 查看集合k1中是否存在v1元素
4. scard k1 // 查看k1中元素的个数
5. srem k1 v1 // 删除k1中的v1
6. smove k1 k2 v3 // 将k1中的v3放入到k2
7. sinter k1 k2 // 取k1 和 k2 的交集
8. sunion k1 k2 // 取k1 和 k2 的并集
9. sdiff k1 k2 // 取k1 和 k2 的差集

### Hash

常用命令

1. hset user : 1 id 1 // 向哈希user中创建key为1,id为1的映射
2. hget user : 1 id // 获取哈希user中key为1的值中id的值
3. hmset user : 1 id 1 name 'zhangsan' // 一次批量添加不同的值
4. hkeys user : 1 // 获取key为1对应的值中的所有的key
5. hvals user : 1 // 获取key为1对应的值中的所有的value
6. hexists key value // 查询key对应的value是否存在
7. hincrby key value n // 让key对应的value加n 

### zset

在set的基础上保证了元素的有序性

zadd key k1 v1 k2 v2 // 向集合key中添加

zrange key from to // 查看集合key中的from到to的元素

zrangebyscore key  min max // 查看集合key中得分在[min,max]中的所有元素

zrevrangebyscore key min max // 倒序查看集合key中得分在[min,max]中的所有元素

zrem key value // 删除集合key中的value元素

zcount key min max // 返回集合key中得分在[min,max]中的元素个数

zrank key value // 查看集合中value的得分排名(升序)

## Redis6配置文件

redis.conf为redis的配置文件.在/etc路径下

1. bind

   ![1667815795547](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1667815795547.png)

   表示127.0.0.1访问的本地主机

2. ![1667815838101](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1667815838101.png)

   表示开启保护模式,只能有本地进行访问redis,改为no即其他主机也可以进行访问

3. port

   ![1667815926517](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1667815926517.png)

   访问redis的默认端口号

4. ![1667815940893](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1667815940893.png)

   表示正在进行tcp三次握手的队列个数和已经完成tcp三次握手的队列个数的和

5. ![1667815996994](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1667815996994.png)

   表示当你长时间不操作时断开连接的时间(0为永不超时)

## Redis6的发布和订阅

发布:publish channel名 message

订阅:subscribe channel名

## Redis6新数据类型

### Bitmaps

位图

常用命令

1. setbit keys value 1/0 // 表示在位图keys中偏移量为value对应的是否存在(1为存在,0为不存在)
2. getbit keys value // 查看位图keys中偏移量为value对应的是否存在
3. bitcount keys from to // 返回位图keys中偏移量在[from,to]中的个数
4. bitop or/and destkey key1 key2 key... // 对位图进行位操作,将key1-keyn的元素位运算存储到destkey中

### Hyperloglog

Hpyerloglog是一种使用更小内存去存储更多的基数(不重复的数)的数据结构

常用命令

1. pfadd keys v1 v2 v3 // 向keys中添加v1,v2,v3(重复的元素不会添加成功)
2. pfcount keys // 查看keys中的元素个数
3. pfmerge destkey keys1 keys2 // 将keys1中的元素和keys2中的元素合并添加到destkey中

### Geospitial

设置地方的经纬度,方便计算两个地方的距离

常用命令

1. geoadd keys j w city ... // 向keys中添加经度为j,纬度为w的城市
2. geopos keys city // 从keys中取出city的经纬度
3. geodist keys city1 city2  dw// 从keys中计算city1和city2之间的直线距离,单位为dw
4. georadius keys j w radius dw // 以j为经度,w为纬度方圆radius,单位为dw范围内keys中的值取出来

## Jedis操作Redis6

## Redis6与Spring Boot整合

## Redis6的事务操作

## Redis6持久化之RDB

## Redis6持久化之AOF

## Redis6的主从复制 

## Redis6的集群

## Redis6新功能