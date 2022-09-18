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

6. 读写分离

7. 分库分表

8. 分区

9. 什么是视图

10. 什么是存储过程