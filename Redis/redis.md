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

5. 进入redis-6.2.7中执行make命令进行编译

6. 编译完成后执行make install

7. 文件会默认下载到/usr/local/bin目录下![1665922407341](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1665922407341.png)

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
   8. 最好在一开始就修改好redis.conf配置文件
      1. daemonize no 改为 daemonize yes(允许后台启动)
      2. bind 127.0.0.1 这一行注释掉,允许其他服务器访问
      3. protected-mode yes 改为no ,取消保护模式,否则其他服务器没办法正常访问
      4. 第2条和第3条的设置是为了云服务器的访问,但也会造成黑客通过开放的6379端口进入你的服务器挖矿...惨痛的教训.   为了防止这种现象的发生一定给redis设置密码. 搜索到requirepass foo..这一行.这一行本来是一个注释,将注释符取消掉,然后将foo...改为自己的密码.
      5. 保存后退出
   9. 注意 当给redis设置了密码时,每次启动redis客户端,都需要auth 密码,然后才可以正常访问

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

jedis是在java中连接redis的一种工具

连接步骤(idea上利用jedis连接云服务器上的redis)

1. 首先需要在云服务器上将对应端口的防火墙打开![1667874068961](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1667874068961.png)

2. 在xshell上打开redis.conf文件 vim/redis.conf所在的文件夹/redis.conf

   ![1667874211296](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1667874211296.png)

   将bind 127.0.0.1 这行注释掉

   将protected-mode 改为yes

   保存退出

3. 重启redis

   redis-cli shutdown

4. 在idea上创建maven项目引入jedis的依赖

   ```xml
     <dependency>
              <groupId>redis.clients</groupId>
               <artifactId>jedis</artifactId>
               <version>3.2.0</version>
           </dependency>
   ```

5. 写测试函数检验是否连接成功

   ```java
   public class JedisDemo {
       public static void main(String[] args) {
           Jedis jedis = new Jedis("43.138.26.181",6379);
           String value = jedis.ping();
           System.out.println(value);
       }
   }
   ```

   ![1667874378133](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1667874378133.png)

## Redis6与Spring Boot整合

创建springboot项目引入redis的相关依赖

```xml
	    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.6.0</version>
        </dependency>
```

在配置文件中配置redis的相关配置

```properties
spring.redis.host=43.138.26.181
spring.redis.port=6379
spring.redis.database= 0
spring.redis.timeout=1800000
spring.redis.lettuce.pool.max-active=20
spring.redis.lettuce.pool.max-wait=-1
spring.redis.lettuce.pool.max-idle=5
spring.redis.lettuce.pool.min-idle=0
```

写config代码

```java
package com.example.demo5.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setConnectionFactory(factory);
//key序列化方式
        template.setKeySerializer(redisSerializer);
//value序列化
        template.setValueSerializer(jackson2JsonRedisSerializer);
//value hashmap序列化
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
//解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
// 配置序列化（解决乱码的问题）,过期时间600秒
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }
}
```

写测试代码

```java
package com.example.demo5.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/redisTest")
public class RedisTestController {
    @Autowired
    private RedisTemplate redisTemplate;

    @GetMapping
    public String testRedis() {
        //设置值到redis
        redisTemplate.opsForValue().set("name","mary");
        //从redis获取值
        String name = (String)redisTemplate.opsForValue().get("name");
        return name;
    }
}


```

常见问题：可能因为某个依赖的版本不兼容导致项目启动不起来,通过报错信息修改相关依赖的版本

## Redis6的事务操作

Redis中的事务是一个单独的隔离操作,事务中的所有命令都会被序列化,按顺序的执行,事务在执行的过程中,不会被其他客户端发送来的命令请求打断.

1. multi 进行组队,将多个操作放入到一个事务队列中
2. exec 进行执行,对事务队列中的多个操作按顺序执行,相当于事务提交
3. discard 表示组队中断,相当于事务回滚

注意事项

1. 在组队阶段,如果有一个操作语法错误,则该事务队列中的所有操作都不执行
2. 在组队阶段成功放入队列中的操作不一定在执行阶段执行成功,但即使某个操作没有执行成功也不会影响事务队列中其他操作的正常进行

#### 事务冲突

在并发操作中发生的冲突

##### 悲观锁

每次操作之前都需要先对操作的数据进行加锁

##### 乐观锁

不是在每次操作之前进行加锁,而是通过引入版本号的方式来执行,每次对数据进行操作都需要增加版本号,通过比较版本号来判断是否发生了事务冲突,如果冲突则再加锁

redis采用乐观锁来解决事务冲突

watch key1 key2 ... keyn // 监视key1-n多个键,在开启事务之前监视多个key

unwatch key1 key2 .. keyn // 曲线监视key1-n多个键

#### 事务特性

1. 单独的隔离操作

2. 没有隔离级别的概念

3. 不保证原子性

   事务中如果有一条命令执行失败,则不会影响该事务中其他操作正常执行

## Redis6持久化之RDB

## Redis6持久化之AOF

## Redis6的主从复制 

## Redis6的集群

## Redis6新功能