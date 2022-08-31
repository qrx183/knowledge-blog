# Buffer Pool

## Buffer Pool的意义

数据库中的数据是存储在磁盘上的,我们在读写数据时需要和磁盘进行交互,而磁盘I/O的效率是较低的.因此为了提高效率,在InnoDB存储引擎中引入了缓存池:Buffer Pool.

## Buffer Pool介绍

Buffer Pool默认大小为128MB,可以通过innodb_buffer_pool_size参数来设置Buffer Pool的大小.

Buffer Pool中是以数据页为基本单位存储的.每个数据页的默认大小为16KB.

Buffer Pool中缓存的内容有:索引页,数据页,插入缓存页,undo页,自适应哈希索引,锁信息.

## 如何管理Buffer Pool

### 如何管理空闲页

使用链表管理空闲页,将空闲缓存页的控制块作为链表的节点,这个链表称为**Free链表**![1661914767270](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661914767270.png)

Free链表中存在一个头结点,头结点中记录了 Free链表的头指针和尾指针以及当前链表中节点的数量

当需要从Buffer Pool中寻找一个缓存空闲页时,根据Free链表快速找到缓存空闲页后将该缓存页从Free链表中消除

### 如何管理脏页

与空闲页类似,Buffer Pool同样使用链表来管理脏页.这个链表称为Flush链表![1661915098659](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661915098659.png)

### 如何提高缓存命中率

Buffer Pool的大小是有限的,因此Buffer Pool中存储的应该是频繁使用的数据,对于一些不常使用的数据在Buffer Pool快满时应该剔除掉.

因此利用LRU算法设计Buffer Pool:当访问的数据页在Buffer Pool中时,将数据页移动到LRU链表的头部;当访问的数据页不在Buffer Pool中时,将数据页加入LRU链表头部,并删除链表尾节点.

尽管LRU算法可以提高缓存命中率,但MySQL并没有使用LRU算法.因为LRU算法无法解决**预读失效和Buffer Pool污染**

> 预读失效
>
> 预读是MySQL为了减少磁盘I/O次数,提高效率的一种方式,是在读取某个数据页时将其相邻的数据页一同加载进Buffer Pool中.
>
> 而预读失效就是提前加载进来的数据页没有被访问.LRU会将预读页放入链表的头部,当预读失效,链表的头部节点是不会被淘汰掉的,因此降低了缓存命中率.
>
> 为了解决预读失效,MySQL对LRU算法进行了改进,将LRU链表分成了old和young区域,young区域在链表前半部分,old区域在链表后半部分.![1661916147635](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1661916147635.png)
>
> 预读页会被放入old区域的头部,只有真正被访问时才会移动到young区域的头部.



> Buffer Pool污染
>
> 当某个SQL语句扫描大量数据时会将Buffer Pool中的热点数据全部刷掉,而这次扫描的数据有可能是很少访问的数据,而热点数据被刷掉,因此缓存命中率下降,这就是Buffer Pool污染
>
> 解决方式:提高进入LRU链表中young区域的门槛.**MySQL对能够进入young区域的数据增加了一个停留在old区域的时间判断**:对某个处在old区域的数据进行第一次访问时,在控制块中记录下此次访问时间,后续再次访问到该数据的访问时间与第一次访问的访问时间之间的时间间隔如果大于**某个规定的值**则允许放入young区域
>
> 这个[规定的值]由参数innodb_old_block_times设置,默认为1000ms.
>
> 因此进入young区域的数据的条件为[被访问]+[在old区域停留时间不少于1s],同时MySQL规定young区域的前1/4数据被访问不会移动到头部,只有后3/4的数据被访问才会移动到头部

### 脏页什么时候会被刷入磁盘

1. redo log日志满后会主动触发刷盘
2. Buffer Pool空间不足时会优先将脏页刷新到磁盘上
3. MySQL空闲时,后台线程会定期将脏页刷新到磁盘上
4. MySQL正常关闭

