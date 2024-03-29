# MySQL索引

## 执行一条select语句,期间发生了什么

MySQL架构主要分为两层:Server层和存储引擎层.

Server层负责建立连接,分析和执行SQL.MySQL的大部分功能都在Server层实现.所有的内置函数和跨存储引擎的功能(存储过程,视图,触发器等)也在Server层实现.

存储引擎层主要负责数据的存储和读取.支持InnoDB,MyISAM,Memory等多个存储引擎.**不同的存储引擎共用一个Server层**.从MySQL5.5开始,默认的存储引擎就是InnoDB,而**索引的数据结构是在存储引擎层实现的,InnoDB的索引的数据结构是B+树.**

![1660875781261](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660875781261.png)

#### 连接器

连接器负责MySQL的连接工作,因为MySQL是基于TCP实现的协议,所以首先需要经过TCP三次握手来**启动MySQL服务**,然后通过验证用户输入的用户名和密码,然后为此次连接的用于授予相应的权限.

查看MySQL服务被多少个客户端连接的命令:show processlist;

在MySQL中,空闲连接(建立好连接后不进行任何操作)是不能长期存在的,有wait_timeout参数控制,默认最大时长为8h.![1660877040499](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660877040499.png)

MySQL服务器的最大连接数有max_connections控制,![1660877018376](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660877018376.png)

MySQL中也存在长连接和短连接.长连接可以避免不必要的连接的资源消耗.但在长连接中,每次查询会使用内存连接管理对象,这些连接对象会在连接断开时释放,如果连接迟迟不断开,MySQL服务会占用过多内存资源.

解决方案

1. 定期关闭长连接
2. 客户端主动重连:MySQL5.7实现了mysql_reset_connection()接口,当连接中占用很多内存资源后,客户端会重置连接,将连接恢复到刚开始连接的状态(不需要重连和权限验证).

连接器做的工作:

> 1. 经过TCP三次握手启动MySQL服务
> 2. 验证用户输入的用户名和密码
> 3. 读取用户的权限并在连接中使用该权限

 #### 查询缓存

连接建立好以后,输入的SQL语句就可以发送到MySQL服务上MySQL服务接收到SQL语句后会解析SQL语句的第一个字段,如果是select的话,就会从[查询缓存]里查找数据,看是否能直接找到SQL语句对应的结果,[查询缓存]中的记录是 以key-value的形式存储的,key是SQL语句,value是SQL语句对应的结果

尽管[查询缓存]能够一定程度上提高查询的效率,但这种提升...微乎其微.因为[查询缓存]中缓存的记录会随着更新操作而清空.只要出现一个更新操作,[查询缓存]中的记录就会随之清空.因此在MySQL8.0中就将查询缓存删掉了,在MySQL8.0之前可以通过query_cache_type来手动关闭查询缓存

#### 解析SQL

MySQL服务在收到SQL语句后会对SQL语句解析.而解析的工作通过解析器来执行.解析器会做2件事情

1. 词法分析:将SQL语句中的关键字识别出来构建成SQL语法树.
2. 语法分析:根据词法分析的结果和语法规则判断SQL语句是否符合MySQL语法.如果语法不对,会在解析时将错误返回给客户端.**注意:表不存在或字段不存在的错误无法再解析时被检测出来**

#### 执行SQL

1. prepare阶段:预处理阶段

   1. 检查SQL语句中的表或字段是否存在
   2. 将select * 中的 * 扩展为表上的所有列

2. optimize阶段:优化阶段

   优化器主要负责确定SQL语句的执行方案,当表中有多个索引,选择查询成本最低的索引

   通过在查询语句前加explain字段可以查看MySQL在查询过程中使用了哪个索引(会显示一个key,key中即为使用的索引,如果key为null,则遍历整个表)

3. execute阶段:执行阶段

   在确定好SQL语句的执行方案后,由执行器负责执行SQL语句.并和存储引擎交互.交互过程可以分为3种

   1. 主键索引

      执行器将主键索引交给存储引擎,存储引擎在B+树中找到符合主键索引的记录返回给执行器,执行器再根据其他判断条件判断,如果符合则返回给客户端,否则跳过该记录继续让存储引擎查询,但因为主键索引的唯一性,所以第二次查找过程中会直接返回-1,执行器退出循环,结束查询.

   2. 全表扫描

      存储引擎会根据表中的记录从上到下读取每一条记录,然后返回给执行器,执行器判断记录是否符合条件,符合则返回,否则则跳过.直到存储引擎遍历完整张表.

   3. 索引下推

      索引下推能够减少二级索引的回表操作.索引下推指的是将执行器判断过程中依据的一些条件下推给存储引擎,由存储引擎负责判断,条件成立后再回表给执行器.![1660880486922](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660880486922.png)

![1660880619811](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660880619811.png)

总结在执行一条select语句的整个过程

1. 连接器:TCP三次握手建立连接,管理连接,校验用户身份,读取用户权限
2. 查询缓存:在缓存中查找是否存在结果,如果存在直接返回,否则继续执行.MySQL8.0删除了查询缓存
3. 解析SQL:经过词法分析得到SQL语句中的关键字并构建SQL语法树.根据语法分析分许SQL语句是否符合MySQL语法
4. 执行SQL
   1. 预处理阶段:判断表和字段是否存在,将*扩展为表上所有列
   2. 优化阶段:确定执行SQL语句的具体方案
   3. 执行阶段:执行SQL语句,与存储引擎交互,从存储引擎中读取记录,返回给客户端.



## 索引常见面试题

### 什么是索引

索引对于数据库而言就相当于目录对于书.能够提高查找的效率,相当于是利用空间换时间.索引的存在能够让存储引擎更快地获取到目标数据.

存储引擎的主要功能就是存储数据,为数据创建索引以及更新,查询数据.MySQL的存储引擎有InnoDB,MyISAM,Memory等,MySQL5.5之后的存储引擎默认为InnoDB.

### 索引的分类

> 按数据结构分类
>
> MySQL的索引可以分为B+Tree索引,Hash索引,Full-Text索引.
>
> ![1661043125386](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661043125386.png)
>
> InnoDB是MySQL5.5之后默认的存储引擎,在InnoDB中最常使用的是B+Tree索引.
>
> 在创建表时,InnoDB存储引擎会根据不同的场景选择不同的列作为索引.
>
> > 有主键,默认使用主键作为聚簇索引的key
> >
> > 没有主键,就选择非空的唯一列作为聚簇索引的key
> >
> > 否则InnoDB会自动生成一个隐式的自增id作为聚簇索引的key
> >
> > 除此之外,其他索引属于辅助索引(二级索引/非聚簇索引)
> >
> > **上述索引默认使用的都是B+Tree索引.**
>
> B+Tree的特点:除了叶子结点外的结点存储的都是索引值,父节点中的索引值在子节点中也所有体现.所有叶子节点按照顺序会连成一个单向链表![1661045192087](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661045192087.png)
>
> 主键索引和二级索引的区别是存储在叶子结点中的值不同,主键索引中的叶子结点存储的是表中的全部信息,而二级索引中的叶子结点存储的是主键的值,因此在用二级索引查询时,如果查询的结果不止是主键本身就需要在得到主键值后再在主键索引中查找到目标结果,也就是**回表**
>
> B+Tree存储千万级别的数据也只需要3-4层高度,因此查询效率非常高.
>
> > MySQL InnoDB选择B+Tree作为索引的数据结构的原因
> >
> > > 1. 与B+Tree相比,**BTree在非叶子节点存储的是全部信息**而非索引值,因此B+Tree的查询效率更高
> > > 2. 与二叉树相比,B+Tree查询千万级别的数据的时间复杂度也为O(3~4),而**二叉树的查询效率为O(logN)**,B+Tree的查询效率要高于二叉树
> > > 3. 与Hash相比,尽管Hash的查询效率为O(1),**但Hash不支持范围查询**.B+Tree支持范围查询,适用性更广.

> 按物理存储分类
>
> 索引分为主键索引和二级索引.区别就是二级索引在查询到的结果是目标的主键值,**可能需要进行回表操作**

> 按字段特性分类
>
> 索引分为主键索引,唯一索引,普通索引,前缀索引.
>
> > 主键索引
> >
> > > 主键索引就是以主键字段作为索引,**一张表中最多只有1个主键索引,且不为空**
> >
> > 唯一索引
> >
> > > 唯一索引是建立在unique字段上索引,一张表中可以有多个唯一索引,每一个唯一索引中的列必须唯一,但可以为空.
> >
> > 普通索引'
> >
> > > 普通索引就是建立在普通字段上的索引.
> >
> > 前缀索引
> >
> > > 前缀索引是指选取字符类型的字段的前几个字符作为索引.目的是减少索引占用的空间,提升查询效率

> 按字段个数分类
>
> 索引分为单列索引和联合索引.
>
> 单列索引就是以单个列作为索引
>
> 联合索引就是将多个字段组合起来构成一个索引.
>
> 联合索引遵循最左匹配原则,例如某一个联合索引(a,b,c),对a先排序,然后对b排序,最后对c排序.因此在使用联合索引时,如果忽略左面的索引而使用右面的索引时不生效的.因为但看后面的某一个索引,其顺序是无序的.![1661047613340](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661047613340.png)
>
> 因此,范围查询也可能导致联合索引失效.
>
> 那么联合索引在找到符合第一个索引的结果后是如何判断
>
> > MySQL5,6之前,是将得到的主键值进行回表,在主键索引中找到记录后判断后续条件是否符合
> >
> > MySQL5.6之后引入了**索引下推优化**,不需要大量的回表操作,在联合索引遍历的过程中,对满足条件的结果先判断后续条件是否符合,如何不符合则直接跳过,符合后再进行回表
>
> 建立联合索引时,根据最左匹配原则,选取索引的顺序是非常重要的.区分度大的索引应该放在靠左的位置.
>
> 区分度 = 单列中不同的值的个数/表中所有行

### 索引的适用场景和非适用场景

索引的优点就是查询效率高,但索引也是有缺点的

> 1. 需要占用额外的物理空间,数据越多,占用空间越大
> 2. 创建索引和维护索引需要消耗时间
> 3. 会降低增删改的效率.每次增删改索引时,B+Tree都需要动态维护.

1. 索引的适用场景

   > 适用于字段有唯一性限制,经常使用Where字段查询,经常用Group By 和 Order By(建立好索引后,就不需要再查询到结果后再一次排序了)

2. 索引的非适用场景

   > Where,Group By,Order By中用不到的字段.
   >
   > 字段中存在大量重复数据,比如性别.
   >
   > 表数据比较少
   >
   > 经常更新的字段

### 如何优化索引

1. 前缀索引优化

   使用前缀索引可以减小索引的大小,提升查询效率

   但前缀索引也有一定的局限性

   > 前缀索引不能用于Order By(严格排序)
   >
   > 前缀索引不能用于覆盖索引(覆盖索引返回的结果是一个唯一确定值而不是一个范围)

2. 覆盖索引优化

   覆盖索引是对使用二级索引是减少回表操作的一种优化.可以将返回的字段作成联合索引,然后在二级索引时找到符合条件的结果而无需返回整行记录的信息.

3. 主键索引最好是自增的

   主键索引如果是非递增的,那么在添加新纪录的过程中有可能主键对应的纪录会添加到某两个叶子结点之间,这也就意味着需要移动已经添加好的记录.而如果是自增的,每次添加的新记录都会按顺序放在已有记录的最后而无需移动已经添加好的记录.

4. 索引设置为NOT NULL

   当某一个值为NULL时,不同的字段对于NULL的处理是不一样的,因此NULL值对优化器而言更加复杂,难以优化.同时NULL值在更多情况下是一个没有意义的值,但会耗费物理空间.

5. 防止索引失效

   索引失效的场景

   1. 使用左或左右模糊匹配("%xx","%xx%")时,索引会失效(索引是根据前缀有序排列的,%无法对应)
   2. 查询条件中对索引列作了计算,函数或类型转换操作,索引会失效.(索引存储的是原始值)
   3. 联合索引时需要遵循最左匹配原则,否则会失效.
   4. Where子句中,如果Or前是索引列,Or后是非索引列,索引会失效(Or要求满足一个即可,所以Or后面的要进行全表扫描)

## 从数据页的角度看B+树

#### InnoDB是如何存储数据的?

记录是按照行来存储的,但在读的过程中不是每次读出一行,而出一次性读出一页.因此InnoDB是按照**数据页**读写数据的.**数据库的I/O操作的最小单位是页**.

InnoDB数据页默认大小是16KB.因此每次最少从磁盘中读取16KB数据到内存,每次从内存刷新16KB数据到磁盘

数据页的结构:

![1661221101844](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661221101844.png)

其中文件头表示页的信息,页头表示页的状态信息,最大最小记录为两个虚拟的伪记录,表示数据页中的最大最小记录,用户记录即为存储的行记录内容,空间空间为页中还没使用的空间,页目录存储用户记录的相对位置,对记录起到索引作用,文件尾用于校验页是否完整.

在文件头中有两个指针分别指向前一个数据页和后一个数据页,从而构成一个双向链表.在数据页中,记录按照主键顺序存储在一个单向链表中.在数据页中页目录充当一个记录的索引,**指向每个槽中的最后一个记录**.方便更快的找到某个记录.

![1661221805666](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661221805666.png)

​	**当查找某条具体的记录时利用二分精确到具体的页目录,然后在该槽中通过双向指针查找某个具体的记录,为了提高查找的效率,InnoDB规定第一个槽中存储1条记录,最后一个槽中存储1~8条记录,剩下的槽中存储4~8条记录.**

#### B+树是如何查询数据的

在B+树中,每个节点都是一个数据页.只有叶子节点中存放数据.所有节点按照索引键大小排序,构成一个双向链表

![1661223674296](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661223674296.png)

#### 聚簇索引与二级索引

### 为什么MySQL采用B+树作为索引

索引的目的是为了提高查询的效率.因此设计索引的数据结构的**查询效率一定要尽可能的高且要尽可能满足各种查询情况.**

> 1. 要尽可能的在少的磁盘I/O次数下完成查询
> 2. 既要支持单个查询,也要满足范围查询

首先当存储的数据是有序时,我们可以通过使用二分法来大大降低查询的时间复杂度.

1. 与平衡树,红黑树相比.当数据量比较大时,平衡树的高度会随着数据量的增大而变高,而B+树即使在数据量很大的情况下也会维持一个较低的树高.
2. 与哈希表比,哈希表不支持范围查找,而B+树的叶子结点通过链表连接可以实现范围查找.
3. 与B树相比,B树在非叶子节点中也存储了大量的用户数据,浪费了大量的空间,在查找某个叶子结点时,需要从磁盘中取出大量非目标的用户数据,浪费了资源.



MySQL采用B+树作为索引的原因

1. B+树只有叶子节点中存有记录,,在非叶子节点中只存放索引,因此相比于既存放索引又存放数据的B树而言,B+树的非叶子节点可以存储更多的索引,因此B+树可以更矮胖
2. B+树中非叶子节点存有大量的冗余节点.因此在更新树中的节点时只需要更改叶子结点而无需动整个树.
3. B+树的叶子节点之间用链表连接,因此实现范围查询的效率更高.

### 索引失效有哪些

1. 使用模糊查询的左模糊查询("%xx")或左右模糊查询("%xx%")

   索引B+树是按照**索引值**进行有序排列存储的,因此只能根据**前缀**进行比较,而上述两种模糊查询的**前缀是不确定的**

2. Where子句中 Or前为索引,Or后为非索引

   Or的含义是实现前后者中的一个即可.因此即使Or前的条件走索引,Or后的非索引条件也会进行全表查询

3. 联合索引时不遵循最左匹配原则

   联合索引就是将多个字段组合在一起创建的索引.如(a,b,c).联合索引要遵循最左匹配原则,也就是先实现a的有序,再实现b的有序,最后实现c的有序.因此如果违反最左匹配原则,就违背了索引的有序性,所以索引会失效.

4. 对索引使用函数,表达式,隐式类型转换等运算

   1. 对索引使用函数

      如select * from user where length(userId) > 1;上述查询过程中就不会使用userId这个主键索引.原因是索引中存储的是**原始的索引值**,通过函数计算得到的值是不能够走索引的.

      但MySQL8.0中已经实现了函数索引,支持经过函数计算的值实现索引.

   2. 对索引使用表达式

      如select * from user where userId + 1 = 10;

      理由跟上述使用函数相同

   3. 对索引使用隐式类型转换

      整数转换为字符串:select * from user where userPhone = 12345;  #userPhone是varchar类型

      字符串转换为整数:select * from user where userPhone = "12345"; #userPhone是int类型

      上述第一种情况索引会失效,而第二种可以走索引.原因是因为MySQL的类型转换支持字符串转为整形但不支持反向转换,**也就是说当字符串和数字比较时,会将字符串转换为数字**

### MySQL使用like "%x",索引一定会失效吗

大部分情况下使用左模糊查询"%x"索引会失效的.**但如果一张表中的属性均为索引时,使用左模糊查询索引也会生效**

例如:user表中有id和name两个属性,id是主键索引,name是一个二级索引.当执行select  * from user where name like '%xx'时,由于此时的*代表的是id和name,而这两个属性因为都是索引所以在二级索引树的叶子节点中存在(覆盖索引),所以无须进行回表操作.因此MySQL优化器认为此时遍历整个二级索引树的效率要高于全表查询.**注意这里的遍历二级索引树不是有序的遍历,而是整个遍历**.

如果查询的结果中有非索引字段,那么使用左模糊查询索引会失效,原因是因为此时的情况在遍历完二级索引树找到对应叶子节点后还需要进行回表操作遍历聚簇索引树.MySQL优化器认为这样的遍历方式的效率要低于全表查询,所以不走索引.

### count(*)和count(1)有什么区别,哪个性能更好

#### 哪种count性能最好

count(*)=count(1)>count(主键字段)>count(非主键字段)

其中count(1)的含义与count(*)相同,是求整个表中行记录的个数.而count(字段)是记录这个字段中非空值的个数.

> count(主键字段)执行过程
>
> MySQL中server层会维护一个count变量,循环向存储引擎读取记录,然后判断是否为null,不为null则count+1,直到循环结束.
>
> 当表中只有主键索引时,会遍历聚簇索引树,如果有二级索引,就遍历二级索引树.因为二级索引树比聚簇索引树小,磁盘I/O次数更少



> count(1)执行过程
>
> 与count(主键字段)类似,区别在于count(1)相较于count(主键字段)而言不需要去读取记录中的值,不需要作判断,直接返回count+1即可



> count(*)执行过程
>
> 在InnODB中,count(*)会被优化处理成count(0),而count(0)的处理过程与count(1)的处理过程基本是一致的.



> count(其他字段)执行过程
>
> 在执行这个函数时,会进行全表扫描

#### 为什么要通过遍历的方式来计数

在MyISAM中,在没有任何查询条件下,count(\*)的时间复杂度是O(1),原因是MyISAM中的每个表信息都存储了有多少行记录的信息.而InnoDB因为支持事务,由于并发操作导致无法维护一个具体的用户记录(脏读).

如果存在查询条件(Where),那么InnoDB和MyISAM都是通过遍历计数的

#### 如何优化count(*)

当表中数据量很大时,如果执行count(*)的效率也是比较低的.

改进措施

1. 用估计值代替近似值:使用explain命令或show table status估计表中记录个数
2. 额外保存表来记录表中个数:每次进行更新操作时,对表中的计数器进行相应的增减



