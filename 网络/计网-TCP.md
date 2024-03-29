# TCP

## TCP三次握手与四次挥手面试题

### TCP基本认识

1. 与连接有关的TCP头部字段

   > 序列号,确认应答号以及标志位中的SYN,ACK,RST,FIN.
   >
   > 序列号:在建立连接时由计算机生成的随机数作为初始值传递给接收方,之后每发送一次数据序列号就增加这个数据的大小,**可以解决网络包乱序问题**.
   >
   > 确认应答号:指下一次期望收到的网络包序列号.发送端可以依据确认应答号来判断对方正常接受信息,**可以解决网络丢包问题.**
   >
   > SYN:为1时表示希望建立连接,并初始化序列号的值
   >
   > ACK:为1时表示确认应答有效,规定除了最初建立连接的SYN包其他包中该位必须为1
   >
   > RST:为1时表示出现异常而强制中断连接.
   >
   > FIN:为1时表示希望断开连接,用于TCP连接断开.

2. 为什么需要TCP协议?TCP工作在哪一层?

   > TCP工作在传输层.
   >
   > 在网络层中IP只关注网络包的发送,不能保证网络包一定发送到接收端,更不能保证网络包以一种完整,有序的方式到达接收端.因此需要其上层传输层TCP来保证数据包的正确到达.
   >
   > TCP是一个可靠的传输层协议,它能确保接收端接收的数据包是无损坏,无间隔,无冗余且有序的.

3. 什么是TCP?

   > **TCP是一种面向有连接,可靠的,面向字节流的传输层协议**.
   >
   > 面向有连接:在通信前发送方和接收方要先建立好连接.且通信是一对一的(当然也可以多对一,多个发送端).不能像UDP那样一个主机同时向多个主机发送消息.
   >
   > 可靠:TCP能够保证数据包正确被接收方接收.
   >
   > 面向字节流:数据在发送时,应用层的数据会在传输层被切分成一个一个的段,**TCP段是有序的**,因此当接收方前一个TCP段没有接收到,后面的TCP段即使被接收到也不能传递给应用层处理,**同时重复的TCP报文会被丢弃.**

4. 什么是TCP连接?

   > 用于保障可靠性和流量控制维护的某些状态信息,这些信息包括Socket,序列号和窗口大小称为连接.
   >
   > Socket:有IP地址和端口号组成
   >
   > 序列号:用于解决乱序问题等
   >
   > 窗口大小:用于流量控制

5. 有一个IP的服务器监听了一个端口,它的TCP的最大连接数是多少?

   > 服务器通常固定在某一个本地端口上监听,等待客户端的连接请求.
   >
   > 由于客户端的IP地址和端口号是可变的,因此理论上TCP的最大连接数=客户端的IP数 X 客户端的端口数
   >
   > IPv4中,客户端的IP数最多为2^32,客户端的端口数最多为2^16.
   >
   > 尽管理论上服务器上TCP的最大连接数是2^48,但在实际应用中要远低于这个数,会受各种因素的影响:
   >
   > 1. 文件描述符限制:每个TCP连接都是一个文件,如果文件描述符被占满了,会发生too many open  file.
   > 2. 内存限制:每个TCP连接都要占用一定内存,当操作系统的内存被占满后,会发生OOM(Out Of Memory)

6. UDP和TCP的区别是什么呢?各自的应用场景有哪些?

   > UDP不提供复杂的控制机制,依靠IP实现无连接的通信.
   >
   > TCP和UDP的区别
   >
   > 1. 连接
   >
   >    > TCP是面向有连接的协议
   >    >
   >    > UDP是面向无连接的协议
   >
   > 2. 服务对象
   >
   >    > TCP传输中接收方只能有一个,只能是一对一,多对一的通信
   >    >
   >    > UDP传输中接收方可以是多个,可以实现一对多的通信.
   >
   > 3. 可靠性
   >
   >    > TCP是可靠传输,数据包会无损,完整,有序的到达接收方
   >    >
   >    > UDP是不可靠传输,不保证接收方能否接收到数据以及接收到的数据是否正确
   >
   > 4. 传输过程
   >
   >    > TCP依靠拥塞控制,流量控制等机制保证传输在安全可靠的前提下尽可能的快
   >    >
   >    > UDP传输不受网络限制,网络拥堵不影响UDP的传输速率
   >
   > 5. 首部开销
   >
   >    > TCP首部长度要长于UDP,且当首部含有选项时,长度会变长且不确定
   >    >
   >    > UDP首部长度为8个字节,是固定的
   >
   > 6. 传输方式
   >
   >    > TCP是面向字节流传输的,属于流式传输,没有边界,但可以保证数据的有序和完整.
   >    >
   >    > UDP是面向数据报传输的,有边界,但可能会丢包或乱序
   >
   > 7. 分片方式不同
   >
   >    > 应用层的数据到TCP传输中会被切分成一个个最大大小为MSS的段进行发送
   >    >
   >    > 应用层的数据到UDP传输中不会进行切分,到达IP层会切分成一个个最大大小为MTU的包进行发送.
   >
   > TCP和UDP的应用场景
   >
   > > TCP是面向连接,保证可靠的传输.可以用于HTTP/HTTPS以及FTP文件传输
   > >
   > > UDP是面向无连接的,但传输速度快,可以用于包总数比较少的通信,如DNS,SNMP等;视频,音频等多媒体通信;广播通信;实时性要求比较高的通信.

7. 为什么UDP头部没有首部长度,而TCP头部有首部长度?

   > 因为UDP头部的大小是固定的,为8个字节.而TCP头部因为选项的存在,长度不固定,所以需要首部长度字段去记录首部的大小

8. 为什么UDP头部有包长度,TCP头部没有包长度?

   > TCP数据长度等于IP总长度-IP首部长度-TCP首部长度. IP总长度与IP首部长度在IP层可知,TCP首部长度在TCP层可知,所以TCP数据长度可以被计算出来.
   >
   > 而虽然UDP数据长度也可以用相同的方式计算出来,但为了方便处理首部长度应该是4的整数倍,而UDP首部的包长度可能是为了补齐4的整数倍

### TCP连接建立

1. TCP三次握手过程和状态变迁

   > ![1659495332700](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659495332700.png)
   >
   > 初始状态:开始客户端和服务器都处于**CLOSED**状态,服务器主动监听一个端口,处于**LISTEN**状态
   >
   > 第一次握手:之后客户端请求建立连接:客户端会生成一个随机值作为序列号的初始值,然后将标志位中的SYN设置为1,将报文发送给服务器,之后客户端处于**SYN_SENT**状态.
   >
   > 第二次握手:服务器收到SYN报文后,也会生成一个随机值作为自己端序列号的初始值,并将标志位中的SYN设置为1;同时会用客户端发送的SYN包中的序号+1作为确认序号返回给客户端,同时将标志位中的ACK设置为1,之后服务器处于**SYN_RCVD**状态.
   >
   > 第三次握手:客户端接收到服务器报文后,向服务器返回一个应答报文,应答报文中的确认序号为第二次握手中报文中的序号+1,同时将标志位中的ACK设置为1,**这次发送过程是可以携带应用层数据的**.之后客户端处于**ESTABLISHED**状态
   >
   > 服务器收到客户端的应答报文后,也会进入**ESTABLISHED**状态
   >
   > **前两次握手中不能携带数据,第三次握手时可以携带数据**
   >
   >  
   >
   > 在Linux3.7中提供了快速连接的功能,对于普通的连接建立,HTTP想要进行一次完整的HTTP请求交互最少需要2个RTT(TCP连接消耗1.5RTT+数据随着第三次握手发送+0.5RTT数据响应的发送),且之后的连接过程都是消耗至少2个RTT.而快速连接的第一次完整HTTP请求交互需要2个RTT,但在之后的连接过程中只需要1个RTT.
   >
   > 快速连接的流程
   >
   > 1. 在第一次建立连接的过程,服务器会在第二次握手过程中产生一个经过加密的Cookie同SYN+ACK报文发送给客户端.客户端在本地缓存这个Cookie.第一次连接完成一次完整的HTTP请求交互仍需要2个RTT.
   > 2. 在下次建立连接时,客户端会将SYN包和Cookie以及数据请求一同发送给服务器,服务器通过解析Cookie获取到TCP连接的信息.然后服务器返回SYN+ACK以及数据响应.因此整个过程只需要消耗1个RTT.![1659755767545](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659755767545.png)

2. 在Linux中如何查看TCP状态

   > 使用命令:netstat -napt

3. 为什么是三次握手?不能是两次,为什么不是四次?

   > 1. 三次握手才可以阻止重复历史连接的初始化造成混乱(主要原因)
   >
   >    考虑一个场景,客户端发送了一个SYN(seq = 50)的包,发送后客户端宕机了,然后因为网络收敛包也没有传输到服务器,客户端重启后发送了一个新的SYN(seq = 70)的包,这时网络恢复正常,之前的SYN(seq = 50)的包优先于新的SYN包到达服务器,服务器会针对seq=50的请求报文做出确认应答ACK(ack = 51)的报文给客户端.
   >
   >    假设只有两次握手,此时服务器发送完报文后处于ESTABLISHED状态,也就是说可以发送数据给客户端了,这时因为网络收敛客户端迟迟收不到服务器的ACK报文,然后服务器又发送了数据给客户端,网络恢复,数据报文优先于ACK报文到客户端,尽管客户端可以通过上下文丢弃异常的ACK报文(因为客户端要的ACK报文的ack=71)而发送RST中断连接,但服务器发送的数据已经到达客户端,所以造成了资源的浪费.
   >
   >    而三次握手则可以处理历史连接的问题,第二次握手的ACK报文达到客户端后,服务器处于SYN_PRVD状态,客户端在接收到错误的ack后会发送RST报文终止异常连接,之后就可以通过正常的SYN(seq = 70)建立正确的连接了.
   >
   > 2. 三次握手才可以同步双方的序列号
   >
   >    TCP协议通信的双方都需要维护一个序列号,序列号可以保证
   >
   >    1. 接收方可以去除掉重复的数据
   >    2. 接收方可以保证接收的数据包有序
   >    3. 序列号和确认序号可以告知发送方接收方接收到了哪些数据
   >
   >    **而一次交互可以让接收方获知发送方的序列号,所以需要两次交互,而中间两次交互可以优化成一次,所以是三次握手,四次握手也可以获取到对方的序列号,但会浪费不必要的资源,而两次握手只能获取到一方的序列号**
   >
   > 3. 三次握手才可以避免资源浪费
   >
   >    两次握手在面临客户端超时重传发送多次连接请求时会让服务器创建冗余的无效连接(由于是两次握手,服务器每次接收到请求报文后发送报文,然后都会建立一个新的连接),进而浪费服务器的资源
   >
   >    四次握手可以优化成三次握手,所以使用四次握手也会浪费不必要的资源

4. 为什么每次建立TCP连接时生成的序列号都不一样?

   > 1. 防止历史报文被下一次相同的连接(四元组相同)接收
   >
   >    假设每一次初始生成的序列号都相同,在上一次连接中的数据报文因为网络拥塞和接收端宕机等原因被遗弃到了网络中,而当下一次连接建立好后,因为生成的序列号相同,所以有很大的概率接收到上一次连接中的数据报文.
   >
   >    而当每次生成的序列号都不相同时,历史连接中遗留的报文大概率就会因为序列号的不同而无法在新的连接中被接收端接收到.
   >
   > 2. 防止黑客伪造序列号发送恶意报文
   >
   >    同理.

5. 初始化序列号ISN是如何生成的?

   > 初始ISN是基于时钟产生的,每4微妙+1,4.55个小时完成一圈
   >
   > 随机算法:ISN = M + F(localhost,localport,remotehost,remoteport),其中M是一个计时器,每4微妙+1,F是一个哈希算法,根据四元组生成的一个hash值

6. 既然IP层会分片,那为什么TCP层还需要MSS?

   > MTU:**一个网络包**的最大长度
   >
   > MSS:**一个TCP段中数据**(不包括TCP首部的长度)的最大长度
   >
   > 假设TCP层不用MSS,而是交给IP进行分片.这样一个TCP数据包在IP层因为超过MTU而被分片.当这些分片传输到接收方时会重组成一个数据包.但如果传输到接收方中的某个分片丢包需要重传,因为IP层没有重传值,而TCP层重传时则需要将这个TCP数据包重新发送.这样就造成了重复传输.因此为了提升效率,在TCP中引入了MSS,在TCP层分片后的报文长度在IP层就不会因为超过MTU再分片,这样即使需要重传,也是以MSS为单位重传而不需要重传整个数据段.

7. 第一次握手丢失了,发生了什么?

   > 首先第一次握手过程是客户端向服务器发送请求连接的SYN报文,在这之后,如何客户端迟迟收不到服务器的确认应答报文(ACK),就会触发客户端的**超时重传**机制,客户端会重新发送一份相同的SYN报文.
   >
   > 不同版本的操作系统触发超时重传的时间不同,1s,maybe 3s
   >
   > 对于重发次数而言,在Linux中,客户端的最大重传次数由tcp_syn_retries参数控制.默认一般是5.
   >
   > 通常第一次重传的等待时间是1s,之后每次重传的等待时间是前一次的2倍(2.4.8.16.32..)在重传5次后,如果仍接收不到ACK报文,则客户端会主动断开连接.所以重传总耗时约为1min(1+2+4+8+16+32=63)

8. 第二次握手丢失了,发生了什么?

   > 第二次握手过程中服务器向客户端发送ACK报文和SYN报文.因此当第二次握手的报文丢失后会导致客户端和服务器都触发超时重传机制.客户端因为没有收到服务器的ACK报文而触发,服务器因为没有接收到客户端的ACK报文而触发.

9. 第三次握手丢失了,发生了什么?

   > 第三次握手过程中,客户端发送ACK报文给服务器后客户端处于**ESTLABLISH**,可以向服务器发送数据.当服务器接收不到ACK报文时会触发超时重传机制.重传SYN-ACK报文.
   >
   > 当服务器发送多次后仍接收不到ACK报文就会中断连接.
   >
   > 对于客户端而言,客户端存在两种情况
   >
   > 1. 客户端向服务器发送数据.
   >
   >    由于服务器已经断开连接所以会触发客户端的超时重传,在建立了连接后,超时重传的次数和没有建立连接的重传次数不同,前者是15次,后者是5次,所以重传15次后会断开连接
   >
   > 2. 客户端不发送数据.会触发保活机制.多次发送保活探测报文后仍接收不到响应则中断连接

10. 什么是SYN攻击?如何避免SYN攻击?

    >  SYN攻击就是第三方在短时间内模拟不同的IP地址给服务器发送SYN报文.服务器收到后发送SYN-ACK报文无法被正确接收,于是服务器的半连接队列就会被占满,因此服务器就无法提供正常的服务.
    >
    >  避免SYN攻击的方式
    >
    >  1. **修改Linux内核参数**.当接收端处理数据包的速度小于发送端发送数据包的速度且接收端的队列已经满时直接发送RST报文,丢弃连接:net.ipv4.tcp_abort_on_overflow
    >
    >    控制队列的最大值:net.core.netdev_max_backlog
    >
    >    SYN_PRVD状态连接的最大数:net.ipv4.tcp_max_syn_backlog
    >
    >  2. ![1659580091747](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659580091747.png)
    >
    >    服务器将客户端的SYN报文放入内核的半连接队列中;接着返回给客户端ACK+SYN报文;在接收到客户端的ACK报文后,**会从半连接队列中删除对应的连接然后创建一个新的连接放到全连接队列中**;应用通过调用accept()方法从全连接队列中取出连接.
    >
    >    当收到SYN攻击时,半连接队列会因为服务器收到多个SYN报文而服务器又无法接收到客户端的ACK报文而无法将连接从半连接队列移动到全连接队列.造成的结果就是**半连接队列队满而全连接队列队空**
    >
    >    可以通过调用命令net.ipv4.tcp_syncookies=1来应对SYN攻击.**当半连接队列队满时,再次接收到的SYN报文不会再放入半连接队列中,而是计算出一个cookie值放入SYN_ACK报文中的序列号字段发送给客户端,服务器验证客户端返回的ACK报文是否有效,如果有效才会删除半连接队列中的连接然后创建一个新的连接放入全连接队列中**. 
    >
    >  3. 减少SYN+ACK重传次数.
    >
    >     第二次握手丢失会造成客户端和服务器均重传报文,当重传次数超过某个限制后会断开连接,因此我们可以控制减少SYN+ACK重传次数,让客户端重传的SYN次数的速度加快,更快的断开连接.

### TCP连接断开

1. TCP四次握手和状态变迁

   > ![1659581284915](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659581284915.png![1659581388669](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659581388669.png)
   >
   > 四次握手过程
   >
   > 1. 初始时客户端和服务器都处于**ESTABLISHED**状态.
   > 2. 第一次挥手:客户端向服务器发送FIN报文,报文首部的标识位FIN值设置为1.之后客户端进入**FIN_WAIT1**状态.
   > 3. 第二次挥手:服务器接收到FIN报文后会反馈一个ACK报文给客户端,之后服务器进入**CLOSED_WAIT**状态.客户端接收到ACK报文后进入**FIN_WAIT2**状态.
   > 4. 第三次挥手:服务器向客户端发送FIN报文,报文首部的标识位FIN值设置为1,之后服务器进入**LAST_ACK**状态.
   > 5. 第四次挥手:客户端收到FIN报文后会反馈一个ACK报文,之后进入**TIME_WAIT**状态.服务器收到ACK报文后进入**CLOSE**状态,客户端从发送报文后等待2MSL时间后进入**CLOSE**状态.
   >
   > 需要注意的是**主动关闭连接的一方才有TIME_WAIT状态**

2. 为什么挥手需要四次,不能像握手一样三次吗?

   > 和三次握手相比,四次挥手好像是将中间两次发送报文分开了,而没有合并.之所以不合并是因为此刻服务器还是可以向客户端发送数据.主动发送FIN报文给对方的一端表示在之后不再发送数据给对方,而不能确定对方也不再发送数据给自己.所以**服务器的ACK报文和FIN报文之间服务器可能还会向客户端发送一些数据报文,所以不能合并**.所以握手需要四次.

3. 第一次挥手丢失会发生什么?

   > 第一次握手后客户端会处于**FIN_WAIT1**状态,等待服务器的ACK报文.当第一次握手丢失后,客户端不能接收到服务器的ACK报文,所以会触发客户端的**超时重传**机制,所以客户端会重新发送FIN报文.当重传一定次数后仍不能收到服务器的ACK报文后会直接进入**CLOSE**状态

4. 第二次挥手丢失会发生什么?

   > 第二次挥手服务器会向客户端发送ACK报文,之后服务器进入CLOSED_WAIT状态,客户端接收到ACK报文后进入FIN_WAIT2状态.当第二次挥手丢失,客户端因为接收不到服务器的ACK报文而触发**超时重传**,重新发送FIN报文,当重传次数超过一定次数后会直接进入**CLOSE**状态
   >
   > 因为close函数关闭连接,所以无法再发送数据,因此客户端在FIN_WAIT2状态不能呆太长时间,通过**tcp_fin_timeout**控制**FIN_WAIT2**状态最多持续60s.
   >
   > 因此客户端在60s内如果接收不到服务器的FIN报文,客户端也会直接进入**CLOSE**状态.
   >
   > 上述是在利用close函数关闭连接的基础上的情况,**如果使用shutdown函数关闭连接,且只关闭发送方.此时如果发送方无法接收到第三次挥手的FIN报文,则会一直在FIN_WAIT2状态持续(tcp_fin_timeout无法影响shutdown关闭连接)**

5. 第三次挥手丢失会发生什么?

   > 服务器接收到客户端的FIN报文后,内核会自动回复ACK,然后进入**CLOSE_WAIT**状态,等待应用进程调用close函数,之后又进程调用close函数而发送FIN报文给客户端.如果第三次挥手丢失,服务器会触发**超时重传**重新发送FIN报文,发送一定次数仍接收不到ACK则会直接进入**CLOSE**状态.

6. 第四次挥手丢失会发生什么?

   > 第四次挥手客户端发送ACK报文给服务器,之后进入**TIME_WAIT**状态,然后等待2MSL后进入CLOSE状态.当ACK报文丢失后,服务器会触发超时重传重新发送FIN报文,当重传多次仍接收不到ACK后会进入**CLOSE**状态.

7. 为什么TIME_WAIT状态等待的时间是2MSL?

   > MSL是报文最大生存时间.它指的是一个报文在网络中可以停留的最长时间,超过这个时间报文就会被丢弃.
   >
   > 之所以等待2MSL,是因为主动关闭方需要等待被动关闭方发送FIN报文,然后反馈给被动关闭方ACK报文,一来一回正好是2MSL
   >
   > 2MSL实际上是允许ACK报文丢包一次的,客户端从接收到FIN报文到发送ACK报文后开始计时,当ACK报文丢失后,服务器会重传FIN报文,这整个过程恰好是2MSL.
   >
   > 之所以不是3MSL,4MSL,是因为丢包率本来在网络中是很低的,连续丢两次包的概率就更低了,所以只考虑一次丢包的性价比更高.
   >
   > Linux中规定1个MSL的时间是30s,所以TIME_WAIT状态等待的时间是60s.

8. 为什么需要TIME_WAIT状态?

   > 1.  防止历史连接中的数据被后面的相同连接错误接收.
   >
   >    首先初始序列号ISN的生成不是无限递增的,它也会发生一个回绕为原来值的情况(计时器是依靠时钟进行递增的).
   >
   >    因此如果TIME_WAIT状态没有或等待时间很短,则再下一次建立相同连接(四元组一致)后,历史连接中遗留的报文就有可能因为序列号匹配而被新的连接中的某一端接收.
   >
   >    设置TIME_WAIT为2MSL足以保证接收方和发送方发送的报文随着此次连接的关闭而全部被丢弃,保证了在下一次连接中的数据包绝不会是历史连接中遗留的数据包.
   >
   > 2. 保证被动关闭一方能够正常关闭连接.
   >
   >    当客户端发送ACK报文进入TIME_WAIT状态后,假如发送的ACK报文丢包了,此时就会触发服务器的超时重传机制重传FIN包,如果没有TIME_WAIT状态,则客户端在发送完ACK报文后就进入CLOSE状态了,当再次接收到服务器的FIN报文后,会发送RST报文给服务器,服务器接收到RST报文后会认为此次关闭是一种异常关闭.所以为了尽量避免这种状况的发生,引入了TIME_WAIT状态.

9. TIME_WAIT状态过多有什么危害?

   > 1. 占用系统资源
   >
   >    > 当被动发起方的TIME_WAIT过多时,服务器上会同时建立多个连接,尽管服务器可以同时和多个客户端进行通信,但过多的连接会占用系统资源而影响服务器的服务效率
   >
   > 2. 占用端口资源
   >
   >    > 当主动发起方的TIME_WAIT过多时,占用过多的端口是,客户端就没有可用的源端口,就无法再次和相同的服务器建立连接,进行通信.

10. 如何优化TIME_WAIT?

    > 1. 打开net.ipv4.tcp_tw_reuse和net.ipv4.tcp_timetemps选项
    >
    >    打开这两个参数后,可以复用处于TIME_WAIT状态的socket为新的连接所用.
    >
    >    net.ipv4.tcp_tw_reuse参数只能用于客户端.开启该功能后,在调用connect()函数时,会随机找一个TIME_WAIT状态超过1s的连接给新的连接复用
    >
    > 2. net.ipv4.tcp_max_tw_buckets
    >
    >    这个值默认为18000.当系统中处于TIME_WAIT状态的值超过这个数时,系统就会将TIME_WAIT状态的连接重置.
    >
    > 3. 程序中使用SO_LINGER
    >
    >    当开启SO_LINGER后,调用close函数后,会立刻发送一个RST给对端,该连接会跳过四次挥手状态直接关闭.

11. 如果已经建立了连接,但是客户端突然出现了故障怎么办?

    > TCP中有一个保活机制:在某个时间段内,如果通信双方没有再进行任何通信,服务器就会发送一个探测报文给对方,这个报文包含非常少的数据.如果连续多个探测报文都没有得到响应,则认为该连接已经死亡.
    >
    > Linux中可以设置报活时间,保活探测次数,保活探测时间间隔
    >
    > > net.ipv4.tcp_keepalive_time=7200    2h无活动后触发保活机制
    > >
    > > net.ipv4.tcp_keepalive_intvl=75     保活探测间隔为75s
    > >
    > > net.ipv4.tcp_keepalive_probes=9   保活次数达到9次后中断连接
    >
    > TCP保活情况
    >
    > 1. 对端正常工作,给予了保活探测报文正常的响应,则保活时间会被重置.
    > 2. 对端程序崩溃并重启,给予了保活探测报文错误的响应,则会发送RST报文表示连接发生异常
    > 3. 对端程序崩溃,无法基于正常的响应,多次发送保活探测报文后会中断本次连接.

12. 如果已经建立了连接,服务器的进程崩溃会发生什么?

    > 服务器会发送FIN报文,与客户端进行四次挥手

### Socket编程

1. 针对TCP应该如何Socket编程?

   > ![1659663458712](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659663458712.png)
   >
   > 1. 服务器和客户端初始化Socket,得到文件描述符.
   > 2. 服务器调用**bind**,绑定IP地址和端口.
   > 3. 服务器调用**listen**,监听某个端口.
   > 4. 服务器调用**accept**,等待客户端连接
   > 5. 客户端调用**connect**,向服务器的地址和端口发起连接请求
   > 6. 服务器**accept**返回用于传输的文件描述符
   > 7. 客户端调用**write**写入数据,服务器调用**read**读取数据(服务器调用write写入数据,客户端调用read读取数据)
   > 8. 客户端调用close断开连接.服务器在read时读取到了EOF,中断读取,待处理完数据后,服务器调用close断开连接
   >
   > 注意:**服务器listen时的socket和accept后得到的用于数据传输的socket是两个socket**

2. Listen时参数backlog的意义?

   > 在建立连接时,服务器内核中会建立两个队列:半连接队列(SYN队列)和全连接队列(Accept队列),具体过程可以参考[TCP连接建立(SYN攻击)章节]
   >
   > 参数backlog在早期Linux内核中是SYN队列的大小,在Linux2.2之后是Accept队列的大小

3. accept发生在三次握手的哪一步?

   > accept是在建立连接前开启,在第三次握手后成功返回.

4. 客户端调用close后,连接断开的流程是什么样的?

   > ![1659665257924](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659665257924.png)

## TCP重传,滑动窗口,拥塞控制,流量控制

为了保证TCP传输的可靠性,需要防止数据的破坏,丢包,重复以及分片乱序等问题.为此引入了众多机制.

### 重传机制

在TCP中,通过**序列号和确认应答**来保证数据被对方接收到.发送方发送的请求报文中会携带序列号,接收方为了让发送方知道自己已经接收到信息,会返回一个确认应答,其中报文中的确认序号为请求报文中序列号+1.

但在传输过程中,由于网络的收敛,数据包很可能发生丢包.因此引入**重传机制**

#### 超时重传

在发送方发送数据时设定一个计时器,当超过指定的时间后,仍没有接收到ACK报文,则重传请求报文.这就是**超时重传**

触发超时重传的有2种情况

1. 发送的请求报文没有到达对方
2. 对方发送的ACK报文没有接收到

对于超时重传机制,最为关键的是确定超时重传的时间.也就是RTO.

在介绍RTO之前,先介绍RTT(往返时延).RTT指的是发送请求到接收到请求的确认应答这一整个过程耗费的时间.

当RTO较长时,我们会浪费时间,降低了网络传输效率;当RTO较短时,重传的报文可能在接收到ACK报文前就被发送,则会造成不必要的重传报文的发送.

因此可以得出结论:RTO应该略高于RTT.但RTT的值是受网络质量影响而不断变化的,因此RTO也是一个动态值.

Linux计算RTO的流程

> 估计往返时间:
>
> 1. 利用TCP采样RTT的时间,然后进行加权平均,得到一个平滑的RTT的值.
> 2. 采样RTT的波动范围,避免RTT发生大的波动后无法被监测到.
>
> ![1659667031510](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659667031510.png)
>
> 其中**SRTT是平滑RTT,DevRTT是计算平滑RTT与最新RTT的差距**.
>
> 在Linux中,**α = 0.125，β = 0.25， μ = 1，∂ = 4**
>
> 在重传后之后每次重传的时间间隔都是前一次重传时间间隔的2倍

#### 快速重传

超时重传面临的一个问题就是超时重传等待的时间较长,为了更快的重传,引入了快速重传机制

快速重传机制不以时间为驱动,而是以数据驱动为驱动.![1659667371899](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659667371899.png)

如上图,在发送数据时,由于数据报文(Seq=2)的丢失,导致其之后的确认应答的报文都是数据报文(Seq=2)的确认应答报文,当发送方收到三个同样的ACK时就会触发重发机制.

快速重传解决了重传周期长的问题,但其面临一个新的问题:重传1个还是重传所有?

假设发送方发送的数据报文(Seq=2)和数据报文(Seq=3)都丢失,在之后发送方收到的Ack都是数据报文(Seq=2)的Ack,此时如果发送方只重传Seq=2的数据报文,那么Seq=3的数据报文还需要在之后等待3个Ack之后再重传,重传的效率很低;而如果重传所有报文(Seq=2~6),Seq(4~6)的报文已经被服务器成功接收过1次,相当于重复发送了相同的请求,浪费了资源.

为了解决快速重传几个的问题,引出了SACK方法.

#### SACK方法

SACK:选择性确认(Selected Acknowledge)

这种重传方式需要在TCP头部[选项]字段里加一个SACK,它可以将已收到数据的信心发送给发送方,通过SACK发送方就可以知道哪些数据收到了,哪些数据没有收到.这样发送方就可以将需要重传的所有数据一次性重传.

SACK要求通信双方都支持才可以正常使用.Linux中通过net.ipv4.tcp_sack参数开启.

#### Duplicate SACK

Duplicate SACK又称为D-SACK,主要作用是告诉发送方哪些数据被重复接收了.

发送重复数据情况:

1. 接收方的ACK报文丢失,发送方触发重传机制重传相同的报文
2. 网络收敛导致之前发送的请求报文没有达到接收端,之后发送方重传相同报文时之前没有到达的请求报文也到达了接收端.

当接收方发现接收了相同的数据后,就会利用D-SACK告知发送方哪个段的数据已经接受过了.

**通过ACK与SACK比较可以得知重传的具体原因是请求报文丢失(ACK与SACK一致)还是确认应答报文丢失(ACK>SACK且触发的重传是超时重传)还是因为请求报文被网络延迟(ACK>SACK且触发的重传是快速重传)了**.

在Linux中可以通过net.ipv4.tcp_dsack参数开启.

### 滑动窗口

在通信过程中,发送方每发送一个请求给接收方,需要接收到接收方的确认应答后才能继续发送下一个请求.这种通信方式很容易造成队头阻塞.也就是说数据包的往返时间越长,通信的效率就越低.为此引入了**滑动窗口**机制

窗口大小:无需等待确认应答,可以发送数据的最大值.窗口大小一般是由接收方的窗口大小来确定的.TCP头部有[窗口大小]的字段,这个字段就是接收方根据自己缓存的剩余空间来确认一个合适的滑动窗口的值发送给发送方.**这样就就可以避免发送方发送数据速度过快而导致接收端处理不过来.**

窗口的实现实际上是操作系统开辟的一块缓存空间,缓存空间中存储的是请求报文,请求报文在接收到确认应答后会从缓存中清除

> 发送方的滑动窗口
>
> ![1659671251983](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659671251983.png)
>
> 滑动窗口由4个部分组成
>
> 1. 已发送并收到ACK确认的数据
> 2. 已发送但未收到ACK确认的数据
> 3. 未发送但大小在接收方处理范围内(可用窗口)
> 4. 未发送且大小不再接收方处理范围内
>
> 注:2,3共同构成发送窗口:2是正在使用的,3是未使用的.
>
> 当有数据接收到ACK确认时,滑动窗口会发生偏移,也就可以继续发送新的数据.
>
> ![1659671415051](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659671415051.png)
>
> 利用三个指针来跟踪四个传输类别中每一个类别中的字节.
>
> ![1659671645331](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659671645331.png)
>
> 1. SND.UNA:指向发送窗口的第一个字节
> 2. SND.NXT:指向可用窗口的第一个字节
> 3. SND.WND:描述发送窗口的大小.指向#4的第一个字节的位置可以通过SND.UNA+SND.WND得到.

> 接收方的滑动窗口
>
> 接收方的滑动窗口由3部分组成
>
> ![1659671788913](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659671788913.png)
>
> 1. 已成功接收并确认的数据
> 2. 未收到数据但可以接收的数据
> 3. 未收到数据且不可以接收的数据
>
> 相应的利用2个指针来划分
>
> 1. RCV.WND:描述接收窗口的大小
> 2. RCV.NXT:指向接收窗口的第一个字节
>
> 接收窗口和发送窗口的大小关系是约等于.因为发送窗口的大小需要接收端根据自己接收窗口的大小决定.因此两者的大小关系存在一定的时延(即接收窗口大小先更新,发送窗口大小后更新)

### 流量控制

发送方不能无脑的发送数据给接收方,要考虑接收方数据处理能力.

如果无脑发送数据的速率超过了接收方处理的速率,会触发发送方的重发机制导致资源的浪费.为此引入了流量控制的机制:发送方会根据接收方的实际接受能力控制发送的数据量

流量控制的具体流程:

1. **接收端会将自己的接收缓冲区的大小放入TCP首部的"窗口"字段,通过ACK反馈给发送端**. 这个"窗口"也就是TCP首部的"16位窗口大小",当然这个缓冲区的大小不止65535位.在TCP首部40字节选项中还包含了一个窗口扩大因子M,实际缓冲区大小是窗口字段的值**左移M位**.
2. 当接收端发现自己的缓冲区中的数据**快满了**,就会**减小窗口的值**然后反馈给发送端,发送端依据此放慢自己的发送速度
3. 当接收端发现自己的缓冲区**已经满了**,就会将窗口的值设置为0,发送端根据这个值变为0后**将不再发送数据,但会定期的发送一个窗口探测的数据段用来得到接收区中缓冲区的剩余大小.当接收端中缓冲区大小有剩余时,会向发送端发送一个窗口更新通知**.

#### 操作系统缓冲区与滑动窗口的关系

发送方和接收方的滑动窗口中存放的字节数实际上都是放在操作系统内存缓冲区的.

当应用进程没办法即使读取缓冲区中的内容时,会对缓冲区造成影响,进而会对窗口造成影响.

当服务器资源十分紧张时会缩小接收缓冲区的大小.因此当客户端发送数据包给服务器时,服务器由于资源紧张减少接收缓冲区的大小,并且应用层无法读取数据,进一步减少了接收窗口的可用大小.也就是说此刻发送窗口的大小是大于接收窗口的(因为接收窗口因为资源紧张减少了大小),在接收方返回最新窗口之前,假设发送了一个超过此时接收方窗口大小的数据,则会造成服务器丢包,同时客户端中的发送窗口的大小会变成负值.

因此,为了防止这种情况发生,TCP规定不允许同时减少缓存且收缩窗口.而是先收缩窗口,之后再减少缓存,避免丢包现象的发生.

#### 窗口关闭

当接收方的窗口大小变为0时,意味着窗口关闭,告诉发送方在这一段时间内不要再发送数据.当接收方数据处理了一部分以后,会主动发送一个ACK报文给发送方告知发送方可以发送数据了.

这里存在一个危险:当接收端窗口大小从0变为非0后,传递给发送方的ACK报文如果发生丢包,发送方和接收方就会处于一种死等的状态.![1659702573200](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659702573200.png)

为了解决接收窗口为0后发送的ACK报文的丢包造成的死锁问题,TCP为每一个连接设定一个**持续定时器**,这个持续定时器从发送方接收到接收方发送的窗口大小为0的通知开始计时.当计时器超时,发送方就会发送一个**窗口探测报文**,而对方在收到窗口探测报文后会给出自己当前窗口的大小:如果为0,则重启定时器;如果不为0,则打破死锁.窗口探测报文在**连续探测3次后收到的窗口大小认为0**,TCP一般就会发送RST报文来中断连接

#### 糊涂窗口综合症

如果接收方太忙了,来不及取走接收窗口里的数据,那么发送方的发送窗口会越来越小.到最后,窗口中只剩下很小的可用区域,然后每次还得发送接收窗口的大小给发送方,然后发送方还会顽固地发送很少字节数的数据,这就是糊涂窗口综合症.

用TCP+IP头部一共40个字节的报文去发送数据只有几个字节的数据会耗费太多不必要的资源

要解决糊涂窗口综合症,就需要避免接收方将小窗口发送给发送方以及避免发送方发送小的数据给接收方.

1. 避免接收方放松小窗口给发送方

   > 接收方发送窗口的策略:当窗口大小小于min(MSS,缓存空间/2)时,会通告一个窗口大小为0的数据报文给发送方.当接收方的可用窗口大小大于min(MSS,缓存空间/2)后把窗口的真实大小返回给发送方.

2. 避免发送方发送小的数据给接收方

   > 发送方发送的策略:使用Nagle算法延时处理,当不满足下述任意一个条件时则不发送数据
   >
   > 1. 要等到窗口大小>=MSS或发送数据大小>=MSS
   > 2. 收到之前发送数据的ack包
   >
   > 由此可知Nagle算法是依靠避免接收方发送小窗口给发送方

   所以避免糊涂窗口综合症的方案是:不通告小窗口给对方+Nagle算法

   Nagle算法默认是打开的,但对于某些特殊的场景需要发送小数据给对方时,则需要手动关闭Nagle算法(设置TCP_NODELAY)

### 拥塞控制

流量控制可以保证发送方发送的数据填满接收方的缓存,但并不能对网络拥堵等情况作出相应的反应.当网络拥堵时,发送的数据包丢包,时延的概率就更大.因此发送方就会重传数据,重传的数据会加剧网络拥堵,然后会更容易丢包,时延.因此会造成一种恶性循环. 因此引出了拥塞控制:当网络发生拥堵时要减少发送的数据量

为了在发送方动态调节要发送的数据量,需要依靠一个拥塞窗口.

拥塞窗口是发送方维护的一个状态量,根据网络拥塞程度动态变化.因此滑动窗口的大小就等于min(拥塞窗口,接收窗口)

当网络中没有拥塞时,拥塞窗口会增大;当网络中出现拥塞时,拥塞窗口会减小**.网络是否拥塞通过发送方是否能在规定时间内接收到ACK报文来衡量.**

拥塞控制由4个算法构成

> 1. 慢启动
> 2. 拥塞避免算法
> 3. 拥塞发生
> 4. 快速恢复

#### 慢启动

TCP在刚建立连接完成后,会有一个慢启动的过程.这个慢启动就是一点一点增加发送包的数量.

慢启动的规则:当发送方每接收到1个ACK,拥塞窗口的大小就会+1.(这个1指的是1个MSS大小的数据包)

因此,拥塞窗口的大小随着接收ACK数量的增多,1,2,4,8...呈指数趋势增长.慢启动过程有一个慢启动门限ssthresh,当拥塞窗口大小小于ssthresh时,采用**慢启动算法**,当拥塞窗口大小大于等于ssthresh时,采用**拥塞避免算法**.一般来说ssthresh的大小为66535字节.

#### 拥塞避免算法

当拥塞窗口大小大于等于ssthresh时,会采用**拥塞避免算法**.

拥塞避免算法的规则:当发送方没接收到1个ACK,拥塞窗口的大小就会增加(1/拥塞窗口大小)

我们从慢启动的指数增长中退出到拥塞避免中,在拥塞避免中,拥塞窗口的大小会呈线性增长.

之后当拥塞窗口的大小增长到一定程度后,也会发生网络堵塞.此时就会触发发送方的重传机制,因此也就进入了**拥塞发生**

#### 拥塞发生

当拥塞窗口的大小增长到发生网络堵塞触发发送方的重传机制时采用拥塞发生算法.

拥塞发生算法的使用取决于采用哪种重传机制

1. 超时重传

   使用拥塞发生算法:ssthresh的值变为拥塞窗口大小的一般,拥塞窗口的大小变为拥塞窗口的初始值.之后就继续慢启动.

   **这种拥塞发生算法过于激进,相当于一个断崖式下降,会造成网络卡顿**

2. 快速重传

   当触发快速重传机制时,TCP会认为这种情况丢包现象不严重,因此拥塞窗口变为原来的一半,ssthresh变为新的拥塞窗口大小,之后进入快速恢复算法.

#### 快速恢复

快速恢复是在快速重传的前提下才发生的.快速恢复算法认为既然能够触发快速重传接收到3个重复的ACK包,说明网络拥塞情况不严重,不需要向超时重传的拥塞发生算法那么激烈.

快速恢复算法的流程

1. 拥塞窗口大小加3(3指的是快速重传中接收到的3个相同的ACK包)
2. 重传丢失的数据包.
3. 如果收到重复的ACK包拥塞窗口的大小+1(这个过程是尽可能重传丢失的数据包)
4. 当收到新的ACK包时,说明之前丢失的包已经全部发送完.至此快速恢复算法结束.拥塞窗口大小变为ssthresh的值,然后进入拥塞避免算法.

快速恢复算法相当于是对慢启动算法的优化.进入快速优化算法说明当前拥塞窗口的大小较大,会引发网络拥堵,所以要减小拥塞窗口的大小,但网络拥堵情况又没有太严重所以不需要像超时重传的拥塞发生算法那样下降的那么剧烈.

![1659749665684](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659749665684.png)

上图是触发超时重传的拥塞控制过程

下图是触发快速重传的拥塞控制过程

![1659749688985](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659749688985.png)

### 延迟确认

在糊涂窗口综合症的解决方案中提到了Nagle算法,Nagle算法+接收方避免通告小窗口可以解决糊涂窗口综合症,

但即使开启Nagle算法,也可能会在最开始的时候发送较小数据的报文.

**延迟确认**则是为了解决ACK传输效率低的问题.即ACK报文中的数据很少甚至没有仍需要发送的问题

延迟确认的策略:

1. 当有响应数据发送时,ACK会随着响应数据一起立刻发送给对方.
2. 当没有响应数据要发送时,ACK会延迟一段时间,以等待是否可以搭载响应报文的顺风车一起发送
3. 如果在延迟等待发送期间,对方的第二个数据报文已经到达了,则立刻发送ACK报文.

当同时使用延迟确认和Nagle算法时会产生新的问题:首先发送方发送一个小数据给接收方,接收方因为没有要发送的数据,根据延迟确认策略会进行等待,而发送方因为Nagle算法要接收到对方的ACK报文才可以继续发送数据.这样就造成了发送方和接收方中在延迟确认等待的这一段时间内毫无动作,降低了传输效率.

解决方案就是关闭延迟确认和Nagle算法中的一个.

##  TCP半连接队列和全连接队列

### 什么是TCP半连接队列和全连接队列

TCP三次握手的过程中,服务器调用listen时,内核会维护两个队列:半连接队列和全连接队列.当服务器接收到SYN报文后,连接会被存储到半连接队列中,服务器发送SYN+ACK报文给客户端,当服务器接收到ACK报文后,内核会把连接从半连接队列中移除,然后创建一个新的连接存放到全连接队列中,等待进程调用accpet方法取出连接.

半连接队列和全连接队列都有大小限制,当半连接队列队满时,再接收到SYN报文会直接丢弃或走syncookie那一套;当全连接队列队满时收到ACK会回复RST报文或直接丢弃.

### TCP全连接队列溢出

1. 全连接队列溢出后会发生什么?

   > 1. 直接丢弃
   > 2. 回复RST报文
   >
   > 通过tcp_abort_on_overflow的值来决定处理策略,为0则直接丢弃ack报文,为1则回复RST报文中断连接.
   >
   > 一般情况下设置为0传输效率更高,除非服务器长期处于繁忙状态导致全连接队列长期溢出才设置为1.

2. 如何增大TCP全连接队列?

   > TCP全连接队列的大小取决于somaxconn和backlog中的最小值.
   >
   > somaxconn的默认值是128,可以通过/proc/sys/net/core/somaxconn来设置其值.
   >
   > backlog是listen(int sockfd,int backlog)函数中backlog的大小.

### TCP半连接队列溢出

全连接队列的最大长度是sk_max_ack_backlog变量,是min(somaxconn,backlog).

半连接队列的最大长度是max_qlen_log变量.有max_syn_backlog和somaxconn,backlog共同决定,max_qlen_log=max(max_syn_backlog\*2,min(somaxconn,backlog)\*2).

max_qlen_log只是半连接队列理论上的最大值.但不代表服务器处于SYN_REVC状态的连接的最大个数.

第一次握手中SYN包被丢弃的条件

> 1. 半连接队列队满且没有开启syncookie
> 2. 全连接队列队满,服务器会丢掉客户度的ACK包从而触发SYN+ACK包重传,重传一定次数后仍接收不到会丢弃SYN包.
> 3. 没有开启syncookie,且max_syn_backlog减去当前半连接队列长度小于max_syn_backlog>>2,则会丢弃.

由SYN包被丢弃的条件可以得出服务器处于SYN_REVC状态的连接的最大个数可以分为2种情况

1. 连接个数小于max_qlen_log但大于max_syn_backlog-max_syn_backlog>>2,实际最大的连接个数是max_syn_backlog-max_syn_backlog>>2
2. 连接个数大于max_qlen_log,实际最大的连接个数是max_qlen_log.



半连接队列溢出后会发生什么?

> 1. 直接丢弃
> 2. 开启syncookie,利用传来的SYN计算出一个cookie值,然后和SYN_ACK报文一同传给发送方,接受方接收到ACK报文后验证ACK报文的有效性后创建新的连接存储到全连接队列中.



如何防御SYN攻击?

> 1. 增大半连接队列
>
> 2. 开启syncookie功能
>
> 3. 减少SYN+ACK重传次数
>
>    当遭受SYN攻击时,会有大量连接处于SYN_REVC状态,因此服务器发送的SYN_ACK报文会丢失而触发服务器重传,因此当减少SYN+ACK重传次数后可以加速服务器断开连接.

## 如何优化TCP?

### TCP三次握手的性能提升

#### 客户端优化

当客户端发送SYN报文后会进入SYN_SENT状态,正常情况下服务器会在毫秒内返回ACK+SYN,但当发生丢包触发客户端的重传机制时,客户端需要等待1..3..7..15..31..63s,也就是需要在重传5次之后才会断开连接(重传最大次数有tcp_syn_retries参数决定),因此**可以根据网络的稳定性和服务器的繁忙程度修改SYN重传次数**

#### 服务端优化

1. 半连接队列满

   >  增大半连接队列大小:增大tcp_max_syn_backlog和somaxconn以及backlog参数的大小,后两者的最小值是全连接队列的大小
   >
   > 开启tcp_syncookies机制.
   >
   > 减少ACK+SYN报文重传次数:修改tcp_synack_retries

2. 全连接队列满

   > 丢弃/回复RST报文(设置tcp_abort_on_overflow:0/1)
   >
   > 增大全连接队列长度:增大somaxconn和backlog参数的大小.

#### 如何绕过三次握手

引入快速连接(TCP fast open)机制.

传统的TCP连接需要消耗最少一个RTT后才能开始发送HTTP请求.但快速连接可以在建立一次连接之后,之后每次建立连接耗费的时间都是0RTT.

TFO过程:

第一次建立连接时:客户端发送SYN报文并在SYN报文中添加空的Cookie选项;服务器接收到SYN报文后会生成Cookie然后保存在SYN+ACK报文发送给客户端,客户端将Cookie缓存到本地;之后就客户端就可以发送数据了.

第二次建立连接时:客户端发送的SYN中包含数据和Cookie,服务器接收后验证Cookie的有效性,如果有效就返回响应和ACK报文.至此可以节省一个RTT.如果Cookie失效则会只发送ACK报文.

Cookie的值是保存在option选项中的.在Linux中,可以通过调整tcp_fastopn参数开始快速连接机制(1:客户端开启TFO,2:服务器开启TFO,3:任意一方开启TFO)

### TCP四次挥手的性能提升

#### 发送方优化

关闭连接的方式分为两种:分别是RST报文关闭和FIN报文关闭.其中前者属于暴力关闭,不经历四次握手的过程,安全的连接方式需要通过四次握手,由进程调用close函数或shutdown函数发起FIN报文.

> close函数意味着完全关闭连接,即调用close函数的一方之后既不能发送数据,也不能接收数据.
>
> shutdown函数可以通过参数来控制关闭的程度
>
> > RD:关闭连接的读.即不能再接收数据.会将接收缓冲区中的数据丢弃,即使对端发送了数据报文也会在回复ACK后将报文丢弃.
> >
> > WR:关闭连接的写.即不能再发送数据.如果发送缓冲区中有数据会立即发送,并发送一个FIN报文.
> >
> > RDWR:关闭连接的读和写,即不能接收数据和发送数据.



> FIN_WAIT1的优化
>
> 发送方发送了FIN报文后会进入FIN_WAIT1状态,在接收到接收方的ACK报文后会迅速进入到FIN_WAIT2状态.但当发生丢包时会触发发送方重传FIN报文,因此可以通过减少重传次数来加速发送方断开连接,即减少tcp_orphan_retries参数的值(默认是8).
>
> 但有时候当遇到恶意攻击时会导致连重发的FIN报文也无法发送(恶意攻击导致接收方窗口大小为0致使发送方无法发送数据,而发送方的发送缓冲区中的数据没有发送完,FIN报文需要在数据发送完后再发送).这时候需要调整tcp_max_orphans参数来降低最大的孤儿连接数量.当孤儿连接数量大于tcp_max_orphans的值时,新的孤儿连接会直接回复RST报文.

> FIN_WAIT2的优化
>
> 当发送方接收到接收方的ACK报文后会进入FIN_WAIT2状态.如果调用的是shutdown函数,因为可能还可以继续发送或接收数据,所以发送方可以一直处于FIN_WAIT2状态.但如果调用的是close函数,那么则会处于孤儿连接,因此在FIN_WAIT2状态不能持续太长时间,这个时间有tcp_fin_timeout参数决定,通常是60s(超过60s会直接关闭连接.),和TIME_WAIT一致.

> TIME_WAIT状态的优化
>
> > 1. 通过tcp_max_tw_buckets参数来控制TIME_WAIT的最大连接,当超过这个值,新关闭的连接将不经过TIME_WAIT状态而直接关闭.
> > 2. 在建立连接时,复用在TIME_WAIT状态的连接超过1s的连接.通过打开tcp_tw_reuse参数(注意:该参数只能由客户端发起,因为是在调用connect方法时发挥作用的.),同时也要将时间戳打开(通信双方都需要打开)
> > 3. 通过tcp_tw_recycle参数回收TIME_WAIT状态的连接(不推荐).当开启recycle和timestamp时,就会开启per-host的PAWS(判断TCP报文中的时间戳是否失效).per-host是对对端IP做PAWS检查,这就面临一个问题:当对端是通过NAT网关后进行通信,不同的客户端经过相同的NAT网关后的IP地址是相同,per-host无法判断对方的时间戳是哪一个客户端的,容易发生误判.优化方式就是对四元组做PAWS检查.
> > 4. 通过设置socket参数来跳过TIME_WAIT状态:将l_onoff设置为非0,l_linger设置为0,在调用close函数会直接回复RST报文来中断连接.

#### 接收方优化

接收方在发送了FIN报文后会进入LAST_ACK状态,如果收不到ACK会触发重传,与发送方一样,可以通过设置tcp_orphan_retries来加速连接的断开.

注意:**接收方发送ACK报文和FIN报文之间如果没有数据发送,则ACK报文和FIN报文可能会同时发送.**



当通信双方同时关闭连接时,即同时给对方发送FIN报文后,会同时进入FIN_WAIT1状态,所以上述的优化策略仍然成立.之后会给对方传输ACK报文,传输后会进入新的状态CLOSEING,在对方接收到ACK后会进入TIME_WAIT状态,等待2MSL时间后关闭连接.![1660030055802](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660030055802.png)

### TCP数据传输的性能提升

> 1. 扩大滑动窗口大小:tcp_window_scaling.可以让滑动窗口从默认的最大值64KB调整为1GB
> 2. 内核缓冲区决定了滑动窗口的大小,将内核缓冲区的上限设置为带宽时延积,带宽时延积是网络中能够承载的数据的最大数量,通过带宽与RTT相乘得到.tcp_wmem是调整发送缓冲区的大小,tcp_rmem是调整接收缓冲区的大小.
> 3. 发送缓冲区的大小是自动动态调节的,而接收缓冲区的动态调节功能可以通过tcp_moderate_rcvbuf打开.

## TCP面向字节流

TCP是面向字节流传输数据的,而UDP是面向数据报传输数据的.

对应UDP而言,当应用层的一个消息传输到传输层UDP协议时,UDP协议不对消息做任何处理,直接将这个完整的消息数据下发给网络层,由网络层进行切片处理发送.因此接收方在UDP协议中接收到的数据是一个完整的数据,应用层读取到的UDP报文中的数据就是一个完整的消息.接收方会将接收到的UDP报文存储到队列中,当应用程序调用时会从队列中取出一个元素.

对于TCP而言,应用层的一个消息传输到TCP中时会被作分片处理(MSS),分片后的数据段在传输到网络层时就无须进行分片处理.因此接收方在TCP中接收到的也是一个一个的数据分片.因此如果不再应用层对一个完整的消息作处理,接收方在接收到TCP段时是无法进行组装的,这也就是常见的粘包问题(即发送方将两个消息的部分数据组装到了一个分片中而接收方无法判断一个消息的长度)

### 如何解决粘包问题

粘包问题的根源在于接收方不知道**一个消息的边界是什么**,因此需要发送方在应用层中对一个完整的消息作特殊处理

1. 固定消息的长度

   > 在应用层发送每个消息时,在消息头部加上这个消息的长度的信息.

2. 用一些特殊字符作为边界

   > 在两个用户消息之间加上一个特殊的字符/字符串作为边界.HTTP请求报文中就是以这种方式处理的(请求头中各个信息之间用空格分割,请求行中字段和字段之间用回车+换行作为分割)
   >
   > 需要注意的是当消息中存在某些字符和边界字符相同时需要对消息中的特殊字符作转义处理

3. 自定义消息的结构

   > 将一个完整的消息发到自定义结构中,自定义结构中存有该消息的长度字段.接收方在接收时解析结构后能够得到一个完整消息的长度.

## 为什么每次建立连接的初始序列号都是不一样的

主要原因是为了防止历史连接中遗失的数据包在下一次相同连接中被错误的接收到.

假设每次建立连接时的初始序列号是一样的.那么当某次连接中因为一些异常原因导致数据包遗留在网络中且关闭连接时没有经过TCP的四次挥手过程(TIME_WAIT状态等待的2MSL可以保证连接中发送的所有数据随着连接的关闭而被丢弃).那么在下一次建立相同连接时,因为初始序列号是相同的,所以在这次连接中,在历史连接中遗留的数据包就有很大的概率在此次连接中被某一端错误的接收到.而每次连接的初始序列号随机生成,即使在此次连接中接收到历史连接中的数据报文,那么也大概率会因为序列号不同的原因而被丢弃.



即使随机生成初始序列号,也有一定的概率使初始序列号发生回绕

初始序列号的生成算法是时钟值+Hash(四元组),其中时钟值是每4微妙+1,4.55小时转完一圈.因此,对于相同的两次连接,前者和后者的初始序列号的值也可能相同.为了避免初始序列号发生回绕,引入了时间戳(timestamps)

通过时间戳,即使接收到历史连接中的报文,也会因为时间戳与Recent TSval值相比不是递增而被丢弃.

## SYN报文在什么情况下会被丢弃

1. 开启了tcp_tw_recycle参数后,使用NAT发送数据包时造成SYN包丢弃

   > 在TCP四次握手过程中,主动请求关闭连接的一方会进入到TIME_WAIT状态,在TIME_WAIT状态需要等待2MSL后才能进入CLOSE状态.tcp_tw_recycle的初衷就是减少TIME_WAIT等待的时间,将处于TIME_WAIT状态的连接进行快速回收.但这个机制在NAT环境下是不安全的.当开启recycle时,需要搭配时间戳使用.时间戳的目的是判断报文的序列号是否发生了回绕.而当recycle和时间戳同时开启后,会触发per-host的PAWS机制.该机制是对**对端IP**的时间戳进行检查,从而丢弃掉过期的数据包.
   >
   > 但在NAT环境下,处于同一个子网的不同的设备使用的是同一个IP地址.当客户端B和客户端A同时发送SYN包且客户端B的时间戳小于客户端A时,当服务器先与客户端A建立连接后服务器主动关闭并快速回收TIME_WAIT状态的连接后,当服务器再次接收到客户端B的SYN包后会因为PAWS检查(时间戳与Recent TSval比较不是递增)而被丢弃.

2. 半连接队列满后,造成的SYN包丢弃

   > 当服务器遭受到SYN攻击后半连接队列满后,如果不开启syncookies机制,则会就后续接收到的SYN包被丢弃.

3. 全连接队列满后,造成的SYN包丢弃

   > 当全连接队列较小时或应用进程没有及时调用accept方法造成全连接队列满后,会造成后续接收到的SYN包被丢弃

## 已经建立连接的TCP,在接受到SYN报文时会发生什么

当客户端和服务器建立好连接后,客户端突然宕机,之后客户端重新发送SYN报文

1. 客户端重新发送的SYN包与历史连接中的源端口不同,客户端和服务器会再建立一个不同的连接

2. 客户端重新发送的SYN包与历史连接中的源端口相同

   1. 发送的SYN包的初始序列号与历史连接的初始序列号不同(因为是随机生成的,所以大概率是不同的),服务器在接收到SYN报文后会回复ACK报文(包含Seq和正常情况下应该回复的ACK),客户端接收到ACK报文后发现ACK不是自己想要的ACK,所以会发送RST报文中断连接. 

      这种方式也可以作为关闭连接的方式:杀掉进程对于服务器而言会关闭与该端口建立连接的所有连接.所以通过伪造发送SYN包给服务器从而收到正确的ACK序号后回复RST报文给服务器可以只关闭这条连接.

   2. 发送的SYN包的初始序列号与历史连接的初始序列号相同(瞎猫碰上死耗子是吧),客户端会重新进入ESTABLISHED状态.

## 四次挥手时收到乱序的FIN包会如何处理

客户端关闭连接时调用的shutdown函数的只写.那么服务器在二三次挥手之间发送的数据因为网络延迟慢于FIN包到达客户端.

此时收到的FIN包属于乱序FIN包,客户端在收到乱序FIN包时会将乱序包放入乱序队列中,当因为网络延迟发送的数据到达客户端后,客户端在读取数据后,会从乱序队列中查看是否有数据包,如果有数据包判断该数据包的序列号是否匹配,如果匹配,则查看数据包中是否存在FIN标志,如果存在才会从FIN_WAIT2状态转换成TIME_WAIT状态.

如何判断数据包是否是乱序的呢?根据序列号来判断,正常顺序的数据包的序号应该是连续的.(x(序列号):100(数据包大小)--->x+100:200--->x+300:140)

![1660185929605](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660185929605.png)

如上图:客户端在接收到ACK报文后得知此时发送的数据包序列号为y,数据长度为0,因此下一次接收到数据报文的序列号还是y,但下次接收到的数据报文的序列号是y+100,说明该报文是乱序报文.

## 在TIME_WAIT状态的TCP连接,收到SYN报文后会发生什么

分成2种情况:合法的SYN和非法的SYN

合法的SYN:SYN报文的序列号要比TIME_WAIT状态下的一方期望收到的序列号大(如果开启了时间戳,还要保证时间戳比期望收到的时间戳大)

非法的SYN:SYN报文的序列号比TIME_WAIT状态下的一方期望收到的序列号小(如果开启了时间戳,SYN报文的时间戳比期望收到的时间戳小也是非法的).



假设处于TIME_WAIT状态的是服务器

收到合法的SYN时,服务器会跳过2MSL等待时间,与客户端建立新的连接

收到非法的SYN时,服务器会回复一个上次收到的ack报文,客户端在收到ack报文后发现ack不匹配后会回复RST报文给服务器.  处于TIME_WAIT状态下的服务器默认情况下接收到RST报文后会跳过等待2MSL时间直接关闭连接,但如果将参数net.ipv4.tcp.rfc1337设置为1(默认为0)时,会将RST报文丢弃

客户端因为收不到匹配的ACK触发重传,重传达到一定次数后会中断连接.

## TCP连接中,一端断电和进程崩溃有什么区别

1. 存在数据交互

   > 进程崩溃和一端断电的处理方式相同
   >
   > 发送方崩溃:接收方收到报文后会发送ACK和响应报文,然后也会因为接收不到ACK重传,之后会中断连接.
   >
   > 接收方崩溃:因为接收不到ACK会重传,当重传达到一定次数后会中断连接

2. 不存在数据交互但开启了保活机制

   > 进程崩溃和一端断电的处理方式相同
   >
   > 发送方和接收方一致:未崩溃的一方会发送保活探测报文给对方,因为对方无法给出相应,在发送几次探测报文都没有响应后会中断此次连接

3. 不存在数据交互且没有开启保活机制

   > 进程崩溃:进程崩溃可以被操作系统得知,操作系统会发送FIN报文通过四次握手关闭此次连接
   >
   > 一端断电:操作系统来不及得知进程崩溃的消息.所以未崩溃的一方会处于僵持状态,知道进程重启.

## 拔掉网线后,原来的TCP连接还会存在吗

将通信一端的网线拔掉,几秒后再插上后TCP连接会发生什么变数?(假设是客户端网线拔掉)

1. 有数据传输. 

   1.  服务器发送数据报文后,接收不到ACK报文会触发重传(在已经建立好连接后重传的上限默认是15次,),在重传过程中,对端网线插回去了.

      因为拔掉网线不会影响TCP的连接状态(TCP连接状态等信息存储在Linux中的一个struct socket结构中,拔掉网线不会让操作系统对struct socket中的内容作任何改变),所以客户端可以正常接收到重传报文然后回复ACK后然后正常通信.

   2. 服务器发送数据报文后,接收不到ACK报文会触发重传,在重传达到最大次数后服务器会中断连接,客户端在之后插上网线后向服务器发送数据,服务器因为已经断开连接所以会回复一个RST报文给客户端使客户端断开连接.

2. 没有数据传输.

   1. 有保活机制会执行保活机制的一套
   2. 没有保活机制,客户端拔掉网线后会和服务器一直保持连接的状态.

## tcptwreuse为什么默认是关闭的?

tcptwreuse参数的开启(需要搭配时间戳)可以快速复用在TIME_WAIT状态下超过1s的连接,属于一种TIME_WAIT状态的优化(跳过了2个MSL).而默认是关闭的原因就是因为跳过2个MSL会发生一些问题.

1. 首先等待2个MSL可以保证此次连接中传输的所有报文随着此次连接的关闭而全部被丢弃.如果跳过则可能会发生当前连接中接收到历史连接中的数据报文.但因为添加了时间戳,所以大部分历史数据报文都是可以被判断出来的.但有一个报文,时间戳对其是无效的.就是RST报文,也就是说.即使接收到过期的RST报文,该RST报文也可以发挥作用中断连接.
2. 当第四次挥手的ACK丢包时跳过2个MSL的话可能会造成被动关闭的一方不正常的关闭.假设使用了tcptwreuse后处于TIME_WAIT状态的连接被快速复用,但此时ACK报文丢包导致第三次挥手的FIN包重传,但另一端已经处于CLOSE状态于是就会回复RST报文给被动关闭一方,从而导致被动一方异常关闭连接.

## HTTPS中TCP和TLS可以同时握手吗?

传统的HTTPS中TCP和TLS是不可以同时握手的.因为HTTPS是基于TCP实现的可靠传输协议,而TCP和TLS又分别处于传输层和表示层,所以二者不能同时握手.

但如果是在TFO(TCP FAST OPEN)和1.3版本的TLS协议的情况下,在经过第一次连接后,之后的连接过程中可以实现TCP和TLS同时握手.

在第一次连接过程中TFP可以将FAST OPEN 的Cookie值缓存到客户端本地,而TLS协议可以得到对应的Socket Ticket/ID.在之后的连接中可以实现TCP和TLS同时握手.

## TCP keepalive 和 HTTP的Keep-Alive有什么联系吗?

除了名字像以外没有啥别的联系.

HTTP的Keep-Alive是对于HTTP1.1对于HTTP1.0中的短连接的一种优化.短连接中,一次连接中只能进行一次数据交互,之后再想进行数据交互就需要再次建立连接.多次连接的建立会浪费过多资源.Keep-Alive长连接允许在一个TCP连接中进行多次数据交互.但每个长连接都有自己的最大连接时限.

TCP keepalive是保活机制,当通信双方在某一段时间内没有数据交互时就会触发保活机制,发送探测报文给另一方,如果能得到对方的响应就正常通信,当发送多次探测报文没有响应后就会认为此次TCP连接已经死亡,会中断连接.

## TCP协议的缺陷

1. TCP升级困难

   TCP是基于内核实现的.升级TCP协议需要升级内核.而内核涉及到底层软件和运行库.而底层的升级是非常麻烦的.因此即使TCP已经提出了大量优化方式,但真正实现新特性的并不多.																											

2. 队头阻塞

   TCP是一种面向字节流的传输协议.所以要保证数据是完整且有序的,在接收方接收TCP段时,如果前面某一个段丢包,那么即使后面的段被接收到也无法被应用层获取到,需要等待丢包的片段重传.

3. TCP连接建立存在较大的延迟

   首先TCP连接需要经历三次握手,甚至基于安全性考虑,还要经历TLS四次握手.连接过程存在较大的延迟.

   即使TFO能够解决TCP三次握手连接的延迟问题,但实现TFO需要通信双方都支持TCP Fast Open机制.

   同时由于TLS属于应用层,所以它的加密范围不包括TCP的头部,这意味着TCP头部是明文传输的,这也就引发了RST攻击,黑客通过仿真数据包的序列号来向服务器发送"合法"(服务器需要的序列号)的RST报文来异常中断连接.

4. 网络迁移需要重新建立连接

   TCP连接是通过四元组确定的.这意味着当四元组中的某一个值发生改变时连接就会断开然后重新建立连接.常见的比如客户端的IP因为从WiFi切换到了热点而改变造成了TCP断开连接,然后重连.重连的过程需要TCP三次握手,TLS四次握手以及TCP慢启动过程.会造成较大的延迟.

## 如何基于UDP协议实现可靠传输?

结论就是在应用层上下功夫:参考TCP保证可靠传输的各种机制并且要改进TCP协议的缺陷(4个).事实上,已经出现了基于UDP协议实现可靠传输的协议:QUIC. HTTP/3就是基于QUIC实现的应用层协议.

1. QUIC是如何实现可靠传输的

   ![1660358379154](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660358379154.png)

   实现可靠传输的两个字段分别是QUIC packet header和QUIC frame header

    1. QUIC packet header

       Packet header分成两种:建立连接时和传输数据时

       1. 建立连接时:源ID,目的ID.

          QUIC建立连接主要是为了确认双方的连接ID.

       2. 传输数据时:目的ID,包编号,数据.

          在传输数据时只需要目的ID就可以发送数据.每个packet的包编号都是唯一的,且严格递增.之所以这么设计是为了解决TCP重传的歧义问题和队头阻塞问题.

          在QUIC中,即使是重传的报文,编号也是递增的,而不是和原报文编号相同.

          在TCP中,重传的报文和原报文的序列号是相同的,这样会造成一种歧义:客户端在接收到ACK时无法得知这个ACK是重传报文的ACK还是原报文的ACK,这就会导致无法精确计算RTT,进而无法计算出合适的额RTO(重传等待时间).  而QUIC中因为编号是递增的,所以根据编号就可以得知得到的报文是重传还是原报文.

          在TCP中,当收到的某个段丢失时,整个连接中的请求都需要等待重传的段到达后才可以被应用层读取.  而QUIC重传的报文的编号也是递增的,所以丢失的报文不会影响其他的报文,滑动窗口可以正常滑动,只要在之后接收到编号更大,数据和丢包数据相同的包就可以了(UDP对包的有序性无要求)

    2. QUIC frame header

       因为QUIC packet header中编号是递增的,即使重传的报文也是递增的,那么如何确定重传的报文和丢失报文中的数据相同呢?

       通过QUIC frame header中的Stream ID和offset(类似TCP中的序列号)实现数据的有序性,如果两个数据包的Stream ID和offset都相同,那么两个包的数据相同.

   通过QUIC packet header可以实现在传输层乱序确认数据包;通过QUIC frame header可以保证应用层可以有序的组装数据.

2. QUIC对TCP协议缺陷的改进

   1. 解决队头阻塞问题

      1. TCP队头阻塞

         1. 发生窗口的队头阻塞

            当发送窗口中可用窗口的数据都接收到ACK后,滑动窗口可以向后移动.但如果位于滑动窗口前面的数据的ACK报文丢失,即使后面的数据接收到ACK报文也需要等待前面数据接收到ACK后,滑动窗口才能移动.

         2. 接收窗口的队头阻塞

            与发送窗口类似.

      2. HTTP/2的队头阻塞

         HTTP/2定义了Stream概念,在同一个连接中,不同Stream中的帧是可以乱序发送的,同一个Stream中的帧要严格保证有序.可以实现并发传输.

         但HTTP/2是在同一个TCP连接中的,所以用的是同一个滑动窗口,根据TCP队头阻塞,当发生数据丢失时,该连接中的所有请求都要阻塞.

      3. QUIC解决队头阻塞

         首先QUIC是基于UDP实现的,所以允许数据乱序.同时QUIC同样实现了HTTP/2中的Stream,且QUIC为每一个Stream单独分配了一个滑动窗口,这样就保证Stream之间相互独立,不会影响别的Stream.

   2. 实现流量控制

      QUIC实现流量控制的方式:

      + 通过window_update帧来告诉对方自己可以接受的字节数
      + 通过BlockFrame告诉对方由于流量控制被阻塞了,无法发送数据

      QUIC将队头阻塞问题缩小到了一个Stream中.

      QUIC实现了2种级别的流量控制

      1. Stream级别的流量控制:一个连接中有多个Stream,对Stream做流量控制可以保证单个Stream不会消耗掉整个连接中所有的接收缓冲

         Stream级别的流量控制![1660361462100](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660361462100.png)

         接收窗口的左边界取决于收到的最大偏移字节数.当图中绿色部分超过最大接收窗口的一半后,最大接收窗口的右边界会右移,同时接收窗口的右边界也会右移.

         QUIC支持乱序确认,超时重传的数据包的编号不会和丢包的编号相同,而是会在已接收包的最大编号上加1.

      2. Connection级别的流量控制:防止发送方发送的数据超过连接的最大接收缓冲.

         就是将一个Connection中的所有Stream的最大可用窗口数求和.

   3. 改进拥塞控制

      首先QUIC协议默认使用TCP的拥塞控制机制,QUIC协议属于应用层,因此相比于传输层的TCP而言,QUIC协议升级迭代的代价更小,更灵活.TCP的拥塞控制机制是适用于所有应用层的,而QUIC协议可以针对不同的应用设置不同类型的拥塞控制机制,灵活性更强.

   4. 更快的连接建立

      QUIC协议可以实现1RTT,甚至在第二次连接之后实现0RTT建立连接.QUIC内部包含了TLS,QUIC协议本身的连接需要1RTT去确认通信双方的连接ID,因为内嵌的TLS是1.3版本,所以TLS也可以实现1RTT建立连接.而两个连接可以同时实现.因此连接只需要1RTT.(传统的建立连接至少2个RTT:TCP1个+TLS1个).

   5. 实现网络迁移

      传统的TCP连接是根据[四元组]确定的,当通信双方中的一方发生网络环境变化,那么建立的连接就会断开然后重连.而QUIC是基于连接ID确认连接的,只要连接ID不发生改变,连接就不会因为网络环境变化断开.

## TCP和UDP可以使用同一个端口吗?

### TCP和UDP可以同时使用同一个端口吗

答案是可以.

TCP和UDP都需要绑定端口,服务器都会调用bind()方法.对于TCP而言,还需要进行监听(listen)动作.在发送数据报文时,IP协议会在头部的[协议号]字段标记传输层使用的协议名,这样即使TCP和UDP同时绑定一个端口,接收方在接收数据时也可以通过协议号来区分不同协议的数据报文.

TCP和UDP协议在内核中是两个完全独立的软件模块.而[端口]只是为了区分同一台主机上不同的应用程序,对于传输数据时的协议无要求.

### 多个TCP进程服务进程可以绑定同一个端口吗

当多个TCP服务进程绑定的不是同一个IP地址时,即使端口相同也可以绑定成功,但如果其中一个进程绑定的是0.0.0.0地址时在执行bind()时会出错,原因是0.0.0.0地址代表任何地址.

但当多个TCP服务进程绑定的是同一个IP地址和端口号时,在执行bind()方法时会报错:"Address already in use".



> 重启TCP服务进程时为什么会报"Address already in user".
>
> 重启意味着先通过正常四次挥手关闭连接然后重新建立连接.在关闭的过程中,处于主动关闭方的一段会进入TIME_WAIT状态然后等待2个MSL后才能关闭连接.而此时处于TIME_WAIT状态的连接也被认为是一个有效的连接,所以此时如果重启在调用bind()方法时会出错.
>
>  
>
> 为了避免重启报错,可以在重启的时候使用SO_REUSEADD属性:当当前进程绑定的ID+PORT和处于TIME_WAIT状态的进程绑定的ID+PORT相同时,新启动的进程可以绑定成功.该属性也可以避免使用0.0.0.0地址造成的bind()报错.

### 客户端的端口可以重复使用吗

当某个IP地址和端口正在使用时,如果新建立的连接的IP地址不同,端口相同,端口是可以重复使用的,如果IP地址也相同,则不能重复使用.

在客户端通常不调用bind方法绑定端口(也可以使用),而是使用connect方法去从net.ipv4.ip_local_port_range中选取一个端口.



如果客户端处于TIME_WAIT状态的连接过多,那么当和不同的服务器建立连接时,端口是可以重复使用的,如果是和相同的服务器建立连接,则无法再与该服务器建立连接

因此通过tcp_tw_reuse参数来复用TIME_WAIT连接:当TIME_WAIT状态的连接超过1s后会被复用.

## 服务器端没有listen,客户端发送建立连接的请求会发生什么?

服务器在接收到报文后因为无法处理报文所以会回复一个RST报文给客户端.



> 没有listen可以建立连接吗?
>
> 特殊情况下是可以的,比如客户端自连接,两个客户端同时向对方建立连接.
>
> listen方法调用后内核会维护两个队列:半连接队列和全连接队列.用于存放建立连接的信息.
>
> 虽然客户端不调用listen方法,没有队列.但在内核中有一个全局hash表存放socket的信息.因此当客户端调用connect方法时会从hash表中取出socket,通过回环地址再将socket存放到hash表中,一来一回建立连接.

## 没有accept能建立TCP连接吗?

可以建立连接.accept只是从全连接队列中取出一个连接来使用,和TCP建立连接本身没有关系.

## 用了TCP协议,数据一定不会丢失吗?

不一定.

1. 建立连接时丢包

   > 建立连接时如果服务器的半连接队列或全连接队列满了以后且没有开启额外的处理机制服务器会丢掉报文.

2. 接收缓冲区丢包

   > 发送方的发送窗口是由接收方的接收窗口决定的,如果因为网络时延导致发送窗口没有及时更新,那么接收方也可能因为接收窗口中可用窗口太小而丢弃发送方发送的数据报文.

3. 网卡丢包

   > RingBuffer过小导致丢包
   >
   > > 接收到的数据会暂时存储到RingBuffer中,当RingBuffer过小时,也会将数据包丢弃.
   >
   > 网卡性能不足丢包
   >
   > > 当发送数据的速度过快超过网卡的最大性能时也会发生丢包

4. 网络丢包

   > 数据包的传输需要经过多个路由器等设备的中转,在中转设备上也极容易因为网络问题发生丢包.
   >
   > 通过mtr命令可以看到每条链路上的丢包情况

5. 如何解决丢包?

   > 使用TCP协议,利用TCP协议中的超时重传
   >
   > 但TCP协议只能保证传输层的可靠性.而对于传输层到应用层之间的可靠性无法保证.
   >
   > 应用层在读取传输层的数据时如果因为异常发生了数据的丢失,发送方也无法感知到.
   >
   > 解决办法就是从两端通信变成三端通信.![1660622479592](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1660622479592.png)
   >
   > 引入服务器后,发送方可以定时的和服务器同步信息去得知信息是否发送成功.接收方可以通过与服务器通信去得知自己少得到了那条数据,然后让服务器重发.
   >
   > 引入服务器的优势:
   >
   > 1. 客户端不需要建立多个连接,只需要和服务器建立1个连接即可.
   > 2. 更加安全.服务器上可以更方便地做一些鉴权功能.
   > 3. 解决版本不兼容问题.当通信双方的版本差过大时,直接通信会因为版本不兼容而无法通信,而引入服务器后可以在服务器上做软件兼容设置.





