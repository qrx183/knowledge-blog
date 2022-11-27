# MySql概述

### SQL概述

1. 基本介绍

   结构化查询语言。程序员需要学习SQL语句，程序员通过编写SQL语句，然后DBMS负责执行SQL语句，最终来完成数据库中数据的增删改查操作。 SQL是一套标准，程序员主要学习的就是SQL语句，这个SQL在多个DBMS中使用。

### 数据库

1. 基本介绍

   数据库(Database)，简称DB。按照一定格式存储数据的一些文件的组合。顾名思义：存储数据的仓库，实际上就是一堆文件。这些文件中存储了具有特定格式的数据

### 数据库管理系统

1. 基本介绍

   数据库管理系统(DataBaseMangement)，简称DBMS。数据库管理系统是专门用来管理数据库中数据的，数据库管理系统可以对数据库当中的数据进行增删改查。

2. 常见的数据库管理系统

   > MySQL,Oracle,MS,SqlServer,DB2,sybase等

### SQL、数据库、数据库管理系统的关系

DBMS ----> 执行 -----> SQL ----> 操作 -----> DB

Windows操作系统中使用命令来启动和关闭mysql服务：

net stop 服务名称

net start 服务名称

注：其他服务的启停都可以采用以上的命令



mysql常用命令

退出mysql：exit

登录：-uroot -p密码，  隐藏密码可以在p之后回车

查看mysql中有哪些数据库：show databases; (末尾要用英文分号)

查看数据库中有哪些表：show tables;

使用某个数据库：use 数据库名;

创建一个数据库：create database 数据库名;

查看mysql数据库的版本号：select version();

查看当前使用的是那个数据库：select database();

中止一条命令：\c

注：命令大小写都可以，mysql是不见";"不执行



数据库中最基本的单元是表：table

数据库中是以表格的形式表示数据的，因为表比较直观

任何一张表都有行和列

行(row)：被称为数据/记录

列(column)：被称为字段，每一个字段都有其特有的属性，不能乱填。每一个字段都有：字段名、数据类型、约束等特性



### SQL语句的分类：

1. DQL：数据查询语言(凡是带有select关键字的都是查询语句)

   + select..

2. DML：数据操作语言(凡是对表当中的**数据**进行增删改的都是DML)

   + insert：改
   + add：增
   + delete：删

3. DDL：数据定义语言,凡是带有create、drop、alter的都是DDL

   DDL主要操作的是表的结构，不是表中的数据

   + create：新建
   + drop：删除
   + alter：修改

   这个增删改和DML不同，这个主要是对表结构进行操作

4. TCL：事务控制语言

   + 事务提交：commit
   + 事务回滚：rollback

5. DCL：数据控制语言

   + grant：授权

   + reboke：撤销权限



导入数据：

将sql文件中的数据导入数据库中

直接将sql文件的路径拖拽到命令窗口中即可

注：路径中不能有中文



查询表中数据：select * from 表名;

查询表中结构：不看具体数据： desc(describe的缩写) 表名;

查询一个字段：select 字段名 from 表名;

查询多个字段：select 字段名1,字段名2 from 表名;

查询所有字段：select * from 表名;(不建议这么做)



给查询的列起别名：select 旧字段名 as 新字段名 from 表名

注意：只是将现实的查询结果的列名显示为新的字段名，原列表名还是旧的字段名。**select语句永远不会进行修改操作的**

as 可以用空格代替，新的字段名中不能有空格，否则会报错，如果新的字段名中必须有空格需要用引号把字段名中引起来

注意：在所有的数据库当中，字符串统一使用单引号括起来，单引号是标准，双引号在oracle数据库当中用不了，但是在mysql数据库中可以使用





计算求和：**可以直接在字段上进行数学运算**，但是同时字段名显示的也为数学表达式，所以**同时需要起别名**



### 条件查询

+ 定义：不是将表中所有数据都查出来，是查询出来符合条件的。
+ 分类：
  1. =等于 ：select 字段名1,字段名2 from 表名 where 字段名3 = 某个值(字符串也可以查找)
  2. <>或!= 不等于：同1
  3. <小于
  4. \>大于
  5. <= 小于等于
  6. \>= 大于等于
  7. between 小数据 and 大数据 两个值之间，等同于>= and <=
  8. is null 或 null (is not null) 
  9. and 并且
  10. or 或者
  11. in 包含,相当于多个or  in(字段1,字段2,字段3)，in后面跟的是具体的值，而不是一个区间
  12. not 取非,常用在in或is中
  13. is null(not null) 查空(不空)    注意：在数据库中，null代表什么也没有，不用用等号，要用is 来表示。

条件查询中：and优先级高于or

​		 14. like 模糊查询,支持%或下划线匹配，%匹配任意个字符,下划线只匹配一个字符。如找出一个字段中含有某个字母或某些字母的：

select ename from emp where ename like %O%; //名字中含有O的

select ename from emp where ename like %K; //名字中以K开始

select ename from emp where ename like K%; //名字以K开始

select ename from emp where ename like _A%; //名字中第二个字母为A

注意：想要找到某个字段中有下划线或者%要用转义字符\





排序：select 字段1,字段2 from 表名 order by 字段1; //按照字段1进行排序，默认升序

select 字段1,字段2 from 表名 order by 字段1 desc; //按照字段1进行降序排序

升序：asc    降序：desc

按照多个字段排序：select 字段1,字段2 from 表名 order by 字段1 asc(desc),字段2 asc(desc);

根据字段的位置排序：select 字段1,字段2 from 表名 order by 2; //根据表中第二列的字段进行排序，但不推荐这样写，因为列的顺序可能发生变化

注意：综合应用时，关键字的顺序不能变：

select ... from ... where... order by... //from这个表中的数据且满足where中条件的数据被select出来按照order by的顺序进行排序

+ 数据处理函数

  + 基本介绍：数据处理函数又被称为单行处理函数

  + 单行处理函数的特点：一个输入对应一个输出。和单行处理函数相对的是：多行处理函数(多个输入对应一个输出)。

  +  常见的单行处理函数

    1. lower  //转小写

       ```mysql
       select lower(ename) from emp;
       //最终字段名会变成lower(ename)，可以用as进行字段名修改
       ```

    2. upper //转换大写

       ```mysql
       select upper(ename) as name from emp;
       ```

    3. substr //取子串 (substr(被截取的字符串,起始下标,截取的长度))

       ```mysql
       select substr(ename,1,1) as ename from emp;
       //注意：下标从1开始，没有0
       select ename from emp where substr(ename,1,1) == 'A';
       ```

    4. concat //字符串拼接(concat(str1,str2))

       ```mysql
       select concat(str1,str2) as ename from empl;
       //将str1,str2连接起来
       ```

    5. length //取长度

       ```mysql
       select length(ename) as length from emp;
       ```

    6. trim //去空格

       ```mysql
       select trim(ename) as ename1 from emp;
       ```

       ```mysql
       //注意：在数据库中
       select 字面量(如1个数字或者1个字符串) from emp;
       //这样写的结果会是把emp中的原有数据全部更换为字面量
       ```

    7. round //四舍五入(round(数字,小数的位数))

       ```mysql
       round(1234.2359,2) //结果为1234.24，保留2位小数 
       //当保留的位数为负数时，按顺序往前递增即可
       //如round(1234.2359,-1),结果为1230
       ```

    8. rand //生成随机数

       ```mysql
       select rand() from emp;
       //1以内的随机数
       ```

    9. ifnull //可以将null转换成一个具体值

       ```mysql
       //在所有数据库中，只要有null参与数学计算最终结果一定为null
       //ifnull 可以把空值转换成一个具体的数值
       ifnull(ename,0); //将ename字段中的所有空值转为0
       ```

    10. case..when..then..when..then..else..end

        ```mysal
        select ename,job,sal as oldsal,(case job when 'MANAGER' then sal*1.1 when 'SALEMAN' then sal*1.2 else sal end) as newsal from emp;
        //case..when..then..when..then..else..end最终会在指定表中形成一个新的字段
        ```

+ 分组函数

  + 基本介绍：分组函数又被称为多行处理函数

  + 特点：输入多行，最终输出一行

  + 常用函数：

    1. count //计数

       ```mysql
       select count(job) as job_num from emp;
       //统计工作数量
       ```

       

    2. max  //最大值

       ```mysql
       select min(sal) as min_sal from emp;
       ```

    3. min  //最小值

       ```mysql
       select max(sal) as max_sal from emp;
       ```

    4. avg //求平均值

       ```mysql
       select avg(sal) as avg_sal from emp;
       ```

    5. sum //求和

       ```mysql
       select sum(sal) as sum_sql from emp;
       ```

  + 注意事项

    1. 分组函数会自动忽略null值
    2. 如果你没有对数据进行分组，整张表默认为一组
    3. 分组函数中count(*)表示统计整张表中的所有行数，count(某个字段)表示统计该字段下非空数据个数，在数据空中不存在某一行的记录全为null
    4. 分组函数不能直接使用在where字句中
    5. 分组函数可以组合使用





# 博客：索引、事务(面试常考)

 事务的四个基本特性：原子性(打包成一个整体，要么全部执行，要么都不执行)、一致性(在事务执行前和执行后，数据均为合理合法的)、持久性(事务执行后调整的数据会被存入硬盘中，不会改变)、隔离性(并发时加锁，防止读取错误数据造成最终数据的错误)并发和隔离是不能兼得的，但两者不是绝对的限制，两者可以同时出现，通过不同的场景来调节隔离程度。MySql中提供的隔离级别：read uncommitted:允许读取未提交的数据，并发最高，隔离最低；read committed:只允许读取提交之后的数据，解决了脏读，但还有不可重复读和幻读；repeatable read:相当于给读和写都加锁，可以解决脏读和不可重复读，但还有幻读；serializable：串行化。隔离程度最高，并发程度最低 



## alter

对表结构进行操作(一般是列(属性))

1. 增

   ```mysql
   alter table 表名 add 属性名 属性数据类型 添加的位置(first,after 某一列之后);
   #如：
   alter student id int first;
   alter student name varchar(20) after student;
   ```

   

2. 删

   ```mysql
   alter table 表名 drop 属性名;
   #如：
   alter student drop id;
   ```

   

3. 改字段

   ```mysql
   修改字段的类型
   alter table 表名 modify 字段名 字段的新类型;
   #如：
   alter student modify id int auto_increment primary key;
   ```

   

   ```mysql
   修改字段的名称
   alter table 表名 change 字段名 新字段名 新字段的类型;
   #如：
   alter 表名 change id sId int;
   ```

   

4. 改表名

   ```mysql
   alter table 表名 rename to 新表名;
   #如：
   alter table student rename to student_SS;
   ```

   
