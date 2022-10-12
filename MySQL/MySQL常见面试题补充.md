# MySQL常见面试题

1. 数据库的常用范式

   > 第一范式(1NF):表中的列不可再分,数据库中的每一列都是一个不可分割的数据项,同一列中不能有多个值
   >
   > 第二范式(2NF):在1NF的基础上,要保证表中有主键,且非主键列必须完全依赖于主键列,不能只依赖于主键列的一部分
   >
   > 第三范式(3NF):在2NF的基础上,消除非主键列对主键列的传递依赖
   >
   > BC范式(BCNF):在3NF的基础上,消除主属性对码(能够唯一标识记录的属性的集合)的传递依赖

2. 常用的存储引擎   InnoDB与MyISAM的区别

   常用的存储引擎有InnoDB,MyISAM,Memory

   > InnoDB和MyISAM的区别
   >
   > 1. InnoDB支持事务,MyISAM不支持事务
   > 2. InnoDB支持表级锁和行级锁,而MyISAM只支持表级锁
   > 3. InnoDB中默认需要使用主键(即使不显示设置主键,也会设置一个隐式的主键),主键要求非空且支持自增,MyISAM中允许一个表中没有主键,且不支持外键
   > 4. InnoDB和MyISAM中使用的都是B+树索引,但MyISAM的B+数索引中的叶子结点存储的是记录的地址,而InnoDB中主键索引中存储的是具体的记录,二级索引中存储的是记录的主键ID
   > 5. InnoDB在调用count函数时会进行遍历,而MyISAM因为不支持事务(并发)所以维护了一个count变量,在调用count函数时可以直接返回count变量

3. 数据库中的锁机制

4. explain的执行计划

   执行计划是SQL语句在执行查询分析器后作出的一个查询方案,通过explain关键字可以了解MySQL是如何执行SQL查询语句的

   ![1663421206097](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1663421206097.png)

   > id:SQL语句执行顺序的标识,id越大代表SQL语句执行的优先级越高
   >
   > select_type:用以区分是普通查询,联合查询,子查询等哪种查询
   >
   > table:查询的是哪张表
   >
   > type:访问类型,即MySQL中如何查找表中的行依次从好到差：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL，除了all之外，其他的 type 类型都可以使用到索引，除了 index_merge 之外，其他的type只可以用到一个索引。一般要求type为 ref 级别，范围查找需要达到 range 级别
   >
   > > system：表中只有一条数据匹配（等于系统表），可以看成 const 类型的特例
   > > const：通过索引一次就找到了，表示使用主键索引或者唯一索引
   > > eq_ref：主键或者唯一索引中的字段被用于连接使用，只会返回一行匹配的数据
   > > ref：普通索引扫描，可能返回多个符合查询条件的行。
   > > fulltext：全文索引检索，全文索引的优先级很高，若全文索引和普通索引同时存在时，mysql不管代价，优先选择使用全文索引。
   > > ref_or_null：与ref方法类似，只是增加了null值的比较。
   > > index_merge：表示查询使用了两个以上的索引，索引合并的优化方法，最后取交集或者并集，常见and ，or的条件使用了不同的索引。
   > > unique_subquery：用于where中的in形式子查询，子查询返回不重复值唯一值；
   > > index_subquery：用于in形式子查询使用到了辅助索引或者in常数列表，子查询可能返回重复值，可以使用索引将子查询去重。
   > > range：索引范围扫描，常见于使用>,<,between ,in ,like等运算符的查询中。
   > > index：索引全表扫描，把索引树从头到尾扫描一遍；
   > > all：遍历全表以找到匹配的行（Index与ALL虽然都是读全表，但index是从索引中读取，而ALL是从硬盘读取）
   >
   > possible_keys:查询时可能使用到的索引
   >
   > key:实际使用哪个索引来优化对表的访问
   >
   > key_lens:实际使用用于优化查询的索引长度
   >
   > ref:显示哪个字段或者常量与key一起使用
   >
   > rows:估算查询所需行数

5. MySQL的主从复制

   主从复制的原理:从服务器从主服务器中获取binlog日志文件,将日志文件解析成对应的SQL语句后在从服务器上重新执行一遍主服务器上的操作.主从复制是一种异步复制,只能保证数据最后的一致性.

   主从复制的流程

   1. master服务器在执行SQL语句后,记录在binlog日志文件中
   2. slave服务器的IO线程与master端建立连接,并请求读取binlog日志文件中的内容
   3. master服务器在接收到请求后,利用binlog dump线程件binlog日志文件中的内容返回给slave服务器
   4. slave服务器将得到的内容存储到relay log文件中并解析relay log文件中的内容,从而达到master端和slave端的数据一致性

   主从复制的好处

   1. 实现了读写分离,在主服务器上进行数据的写操作,在从服务器上进行数据的读操作,提高了主服务器的性能
   2. 提高了数据安全性,主服务器崩溃造成的数据丢失可以通过从服务器上的备份进行恢复

6. 分库分表

   分库分表可以解决数据库的存储瓶颈,提高查询效率

   1. 垂直拆分
      1. 垂直分表:将一个表按照字段分成多个表,每个表存储一部分字段,常见的是将常用的字段放到一张表中,将不常用的字段放到另一张表中.
      2. 垂直分库:按照业务模块的不同,将表拆分到不同的数据库中,适合业务之间的耦合度非常低,业务逻辑清晰的系统
   2. 水平拆分
      1. 水平分表:将一个表中的数据按照某种规则拆分到多张表中
      2. 水平分库:将一个表中的数据按照某种规则拆分到不同的数据库中

7. 什么是视图

   视图是从一个表或多个表中导出的表,其内容由查询定义.视图是一张虚拟表,在数据库中只存储视图的定义而不存储视图的数据.

   视图的优缺点

   > 优点
   >
   > 1. 简化了操作,可以把经常使用的数据定义成视图
   > 2. 提高了安全性,用户只能操作可见的数据
   > 3. 逻辑上实现了独立.屏蔽了真实表结构的影响
   >
   > 缺点
   >
   > 1. 性能差.在查询视图时需要转为对基本表的查询,尤其当视图是由一个多表查询所定义

8. 数据库的类型

   1. 关系型数据库

      采用关系模型来组织数据的数据库.关系模型即为二维表格模型.

      常见的关系型数据库有MySQL,SqlServer,Oracle

   2. 非关系型数据库(NoSql)

      NoSql放弃了传统sql的强事务保证和关系模型,重点放在数据库的高可用性和高扩展性

      常见的NoSql有Redis,MongoDB,HBase

   3. 高性能数据库

      高性能数据库在NoSql的基础上仍基于关系模型,同时也保留了成熟的SQL作为查询语言,保证了事务的ACID

      常见的NewSql有lustrix,GenieDB

9. SQL的优化

   SQL的优化可以分为4个部分:SQL语句以及索引,数据库表结构,系统配置和硬件.优化效果依次下降

   select语句的执行顺序:from ---> on ---> join ---> where ---> group by ---> having ---> select ---> distinct ---> order by ---> limit

   SQL优化的核心

   1. 最大化利用索引
   2. 尽可能避免全表扫描
   3. 减少无效数据的查询

   **避免不走索引的场景**

   1. 避免使用左模糊查询,导致数据库引擎走全表扫描

      ```mysql
      select a.name from a where a.name like "%邱"
      ```

   2. 尽量避免使用in和not in. 当属于是范围查询的时候可以用between来代替,当属于是子查询的时候可以用exists来代替

      ```mysql
      select a.id from a where a.id in(1,2,3)
      # 使用between代替
      select a.id from a where a.id between 1 and 3
      
      select a.id from a where a.id in(select ....)
      # 使用exists代替
      select a.id from a where a.id exists(select ....)
      ```

   3. 尽量避免使用or导致数据库引擎走全表扫描.可以用union代替

      ```mysql
      select a.id from a where a.id = 1 or a.id = 2;
      # 使用union代替
      select a1.id from a a1 where a1.id = 1
      union
      select a2.id from a a2 where a2.id = 2;
      ```

   4. 尽量避免进行null值的判断导致数据库引擎走全表扫描,可以用0或其他的数字代替.同时在MySQL中不同的函数对null值的处理也不一样.count(可为null的字段)会忽略掉为null的记录,count(*)会包含null

   5. 避免在条件查询中对索引条件使用<>和!=等判断条件

   6. 使用联合索引时避免破坏最左匹配原则

   7. 避免对索引使用函数或表达式以及索引的隐式类型转换

   **select语句的其他优化**

   1. 尽量避免使用select *. select * 会导致优化器无法执行索引覆盖这一类优化措施,导致得到的查询方案可能不是最优的.
   2. 多表关联查询时,小表在前,大表在后. 从左到右查询先对小表进行全表扫描
   3. 用where语句去代替having语句. where是在查询结果之前对条件进行筛选,having是在查询结果之后对条件进行查询
   4. 用join去代替子查询

   **增删改语句的优化**

   1. 使用批量插入代替分条插入

      ```mysql
      insert into a values(1,2);
      insert into a values(1,3);
      insert into a values(1,4);
      # -------
      insert into a values(1,2),(1,3),(1,4)
      ```

      批量插入可以减少SQL语句的解析,同时SQL语句短可以提升网络传输的效率

   2. 当大批量增删改语句时,可以适当的使用commit定期提交

   **建表的优化**

   1. 在表中建立索引,主键索引的id最好是自增的

   2. 尽量使用数字型字段来代替字符型字段.因为在比较时数字只需要比较一次,而字符型字段需要一个字符一个字符的比较

   3. 用varchar/nvarchar代替char/nchar.变长字段的存储空间更小.查询效率更高

   4. 在使用日期类型时,使用timestamp代替date,前者的存储空间只是后者的一半

   5. 尽量避免一张表建立的太大,可以适当拆分

   6. 在建表时尽量将字段设置为not null

      1. 在判断是否为null时会走全表查询
      2. null的字段会占用额外的存储空间
      3. 不同的函数对null字段的处理可能不一致
   7. 
