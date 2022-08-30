# MySQL日志

## 执行一条update语句会发生什么

1. 客户端首先通过连接器建立连接,对用户的身份信息进行校验
2. update语句不会进入查询缓存,但会将查询缓存中的内容清空
3. 语句经过词法解析得到语句中的关键字并构造语法树,然后经过语法分析
4. 通过预处理器判断表和字段是否存在
5. 通过优化器构造更新的执行方案
6. 由执行器负责和存储引擎交互进行更新

更新语句会涉及到undo log(回滚日志),redo log(重做日志),binlog(归档日志)三个日志

> undo log:是InnoDB存储引擎生成的日志,实现了事务中的原子性,主要用于**事务回滚和MVCC**(多版本并发控制,指的是一条记录有多个版本)
>
> redo log:是InnoDB存储引擎生成的日志,实现了事务中的持久性,主要用于**掉电等故障恢复**
>
> binlog:是Server层生成的日志,主要用于**数据备份和主从复制**

## undo log

1. 事务回滚

   当对数据进行更新时,MySQL会默认开启事务,在语句执行完后自动提交事务.undo log能够保证事务的原子性:当事务未提交而发生崩溃时可以根据undo log中的数据信息对事务进行回滚.undo log会在执行事务前将未更新的数据信息存储起来,回滚时对undo log中对应的数据作相反的操作

   对于update而言,其在undo log中的记录有trx_id和roll_pointer两个指针,trx_id负责记录该记录是被哪个事务修改的,roll_pointer则将之前的记录连成一个链表(版本链)

2. 实现MVCC

   undo log和read view可以实现可重复读和读已提交隔离级别下的**快照读**

## Buffer Pool

数据库中的记录都是存储在磁盘中的.因此当对数据库中的记录进行更新时需要先从磁盘中取出记录,然后在内存中进行操作,并将更新后的数据存储到磁盘上.这样每个记录都需要1次磁盘I/O.

如果在存储引擎中对操作后的记录进行缓存,那么之后在查询相同的记录时就无须再和磁盘交互,直接从Buffer Pool中存取,![1661826872711](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661826872711.png)

在Buffer Pool中,

当读取数据时,首先会从Buffer Pool中查找,如果存在直接读取Buffer Pool中的数据,否则在去磁盘中读取

当修改数据时,如果数据存在于Buffer Pool中,直接修改Buffer Pool中数据所在的页,并将页设置为**脏页**,同时脏页会由后台选择一个合适的时机写入磁盘

Buffer Pool存在于存储引擎中,而InnoDB存储引擎上是以页为单位存储数据的.

在Buffer Pool中存储了索引页,数据页,插入缓存页,undo页,自适应哈希索引,锁信息![1661827303500](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661827303500.png)



## redo log

redo log是物理日志,记录了数据作了哪些修改.在事务提交时,只需要将redo log持久化到磁盘上而无需等待将Buffer Pool上的脏页持久化到磁盘.**系统崩溃时,可以根据redo log将数据库恢复到最新内容**

redo log和undo log的区别

1. redo log记录了此次事务**完成后**的状态,记录的是**更新之后的值**
2. undo log记录了此次事务**完成前**的状态,记录的是**更新之前的值**

redo log中的内容也不是直接写到磁盘上,而是缓存到自己的redo log buffer中,之后再将read log buffer中的内容持久化到磁盘上

redo log什么时候刷盘

1. MySQL正常关闭时
2. 当read log buffer中记录的数量超过其空间的一半时
3. InnoDB后台线程每隔1s,将redo log buffer中的内容持久化到磁盘上
4. 事务提交时(根据innodb_flush_log_at_trx_commit参数控制:0为事务提交不刷盘,1为刷盘,2位将redo log buffer中的内容写到redo log文件中)

## **binlog**

binlog 是server层生成的,记录了一个事务对数据的修改和表的变更的信息.不记录查询信息.

binlog和redo log的区别

1. 适用对象不同

   binlog是server层实现的日志,所有存储引擎都可以使用

   redo log是InnoDB存储引擎实现的日志

2. 文件格式不同

   binlog有3中格式,statement,row,mixed

   statement格式是对每一个修改数据的SQL语句进行记录.缺点是当使用**动态函数**时,主库上的执行结果和从库上的复制结果不一致.

   row格式是对每一个变化的数据行进行记录.缺点是记录的数据量较大,会使binlog文件过大.

   mixed格式就是row格式和statement格式混用

   redo log是物理日志,记录的是某个数据页发生了什么修改

3. 写入方式不同

   binlog是追加写,写满一个文件会创建一个新的文件.不会覆盖原来的日志

   redo log是循环写.

4. 用途不同

   binlog主要用于数据备份,主从复制

   redo log主要用于掉电等故障恢复



### 主从复制的实现

MySQL主从复制的实现依赖于binlog日志,binlog日志中记录了MySQL上的所有变化并以二进制形式存储到磁盘上.从库通过复制主库binlog日志内容实现主从复制

![1661841774491](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661841774491.png)

主从复制可以分为3个过程

1. 写入binlog:主库接收客户端的请求,在提交事务前将数据更新记录存储到binlog日志中

2. 同步binlog:从库创建一个I/O线程连接主库的log dump线程将主库的binlog文件中数据存储到relay log中继日志中.

3. 回放binlog:从库回放binlog,并更新存储引擎中的数据

   主从复制的过程是异步的,实现主从复制后就可以实现读和写的分离.有主库负责写,从库负责读![1661842445788](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661842445788.png)

MySQL主从复制的模型

1. 同步复制:MySQL主库在等待所有从库主从复制成功后返回客户端响应.实用性很差
2. 异步复制:默认的一种模式.MySQL主库不会等待从库主从复制就返回客户端响应.缺点是主库宕机会造成数据丢失
3. 半同步复制:主库会等待某一部分从库主从复制成功的响应之后再返回客户端响应.是对同步复制和异步复制的一种结合.



执行一条update语句到执行器

1. 首先执行器会从存储引擎中找到对应的记录:如果存储引擎的Buffer Pool中有,则直接取,否则将磁盘中对应的数据页存到Buffer Pool中.
2. 执行器拿到要更新的记录后会对比更新前后记录是否发生了改变,如果没有改变则直接结束.否则将记录的更新任务交给存储引擎
3. 存储引擎拿到更新的记录后,先将更新前的旧值存储到undo log中,也就是修改Buffer Pool中的undo页,在修改undo页之前,先记录修改undo页的redo log.
4. 存储引擎开始更新记录,先记录更新记录的redo log,然后更新Buffer Pool中对应的数据页,该数据页成为脏页后并不会马上存储到硬盘,后台线程会选择一个合适的时机将脏页存储到磁盘
5. 语句执行结束后,会将修改记录存储到binlog中,binlog中的内容会存储到binlog cache中,然后随着事务的提交将binlog cache中的内容刷新到磁盘上

## 两阶段提交



## 优化MySQL磁盘I/O