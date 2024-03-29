# 计算机网络

## 基础篇

### TCP/IP网络模型

#### 存在即合理

对于同一台设备,在进行进程间的通信,可以采用管道,消息队列,信号,共享内存等多种方式,而对于不同设备间进程的通信,就需要进行网络通信,而由于设备的多样性,为了兼容不同类型的设备,所以需要设计出一套**通用的网络协议:即TCP/IP网络模型**

### 应用层

位于TCP/IP网络模型的最上层,是使用手机或电脑的用户能够直接体验到的一层,应用层负责将数据传输到下一层(传输层).

应用层专注于为用户提供相关的功能,比如HTTP,FTP,Telnet,DNS,SMTP等.

应用层不用关系数据是如何传输的,就像我们在挑选完商品后有快递公司负责派送,我们不需要关心商品具体是如何被派送的.

应用层位于操作系统的用户态,而传输层及其下面的层位于内核态.

### 传输层

传输层的设计理念是简洁,高效,专注.

传输层是为应用层**提供网络支持的**.专注于确定发送方和接收方的设备上某个具体的应用,为了达到这一目的.传输层通过一个叫"端口"的编号来区分设备上的不同应用.

传输层的两个主要的协议是TCP和UDP.

TCP的全称是传输控制协议.大部分应用使用的正是TCP协议,TCP相比UDP而言多了许多特性,包括流量控制,拥塞控制,超时重传等.依据这些特性保证了数据报可靠地传输到接收方.

UDP相较于TCP来说简单很多,只需要发送数据包,不需要考虑数据是否能够抵达对方,但它实时性相对更好,传输效率更高.而UDP实现可靠传输则需要在应用层实现TCP的这些特性(但这并不是一个简单的事情额...).

应用层的数据有时候会很大,如果直接传输既不容易控制,又会消耗大的成本(比如因为网络收敛导致发送的数据损坏了而需要重新发送时需要重新发送所有数据),当传输层的数据包大小超过MSS(TCP最大报文段长度),就要将数据包分块,而分块的好处是即使在传输过程中某个分块损坏需要重新发送也只需要发送这一个分块,在TCP协议中,这样一个分块称为一个TCP段

### 网络层

由于传输层专注于服务于应用层,作为应用层的一个媒介,因此,传输的主要功能则是由网络层实现.

网络层的主要功能分为两部分,一部分为寻址,另一部分为路由.不同设备之间通过复杂的网络连接,而网络由交换机,路由器等设备共同连接构成.在错综复杂的网络环境中,找到对应的路径和节点就叫做寻址(从一个节点应该朝哪个方向走),而路由则是从一台设备规划处一条路径到达下一个节点. **寻址就像是导航,而路由则是操纵方向盘**

网络层最常用的是IP协议,IP协议将传输层的部分作为数据,再加上IP报头组成IP报文.如果IP报文大小超过MTU(以太网中一般为1500字节)则会再次分片.

为了区分不同的设备,引出IP地址,IP地址有网络号和主机号组成.网络号标记设备位于哪个子网,主机号区分子网中的哪个主机.网络号和主机号由子网掩码进行区分(子网掩码和IP地址与运算得到网络号;取反后的子网掩码和IP地址与运算得到主机号)

### 网络接口层

将IP报文传输给网络接口层,在IP头部加上MAC头部,并封装成**数据帧**发送到网络上.

什么是以太网?以太网就是电脑上的以太网接口,WI-FI接口,路由器上的千兆,万兆以太网口.以太网就是在局域网内,将附近的设备相连起来可以进行通讯的一种技术.

在以太网中,无法通过IP地址去定位目标地址,而需要MAC地址.通过ARP协议获取对方的MAC地址.

### 键入网址到网页显示,期间发生了什么?

1. 首先浏览器解析URL,然后生成发送给WEB服务器的请求信息(当URL中没有文件名路径时,默认访问根目录下的index.html/default.html).解析完后生成请求报文/响应报文,HTTP报文

2. 在发送HTTP数据包之前,我们需要通过DNS查询服务器域名对应的IP地址.如www.baidu.com.,越靠右等级越高.域名的层级关系类似一个树形结构

   > 域名解析的工作流程:
   >
   > 1. 客户端发送一个DNS请求,请求某个域名A对应的IP地址,这个请求首先被发送到本地的DNS服务器上,如果能够在缓存里找到对应的IP地址,则直接返回;如果找不到,就问根域名服务器,根域名服务器会将负责次一级域名解析的服务器地址返回给本机,然后按照等级依次降低,直到找到存储该域名IP地址的DNS服务器,然后获取到对应域名的IP地址,本地DNS将IP地址返回给客户端,客户端与目标建立连接.这种找IP地址的方式类似于**找路:只指路不带路**

   **当然并非每次找域名的IP地址时候都经历一次DNS的搜索,而是依靠多个地方的缓存快速获取:首先是浏览器缓存,其次是操纵系统中的缓存,然后是hosts文件,如果都没有,才会传到本地DNS服务器**

3. 获取到IP地址后,HTTP的传输工作就交给了协议栈.协议栈中上面的给下面委托工作,下面的负责执行上层委托的工作.协议栈中上层的主要角色时TCP和UDP,负责接收应用层的数据,下层是IP负责控制收发数据包.

4. 可靠传输——TCP

   HTTP是基于TCP形成的,所以需要依靠TCP去传输HTTP数据包.TCP通过各种机制保证数据传输的可靠性,通过三次握手建立连接,通过源端口和目的端口来定位到发送方主机上的应用和接收方主机上的应用.HTTP报文作为数据,在加上TCP报头形成了TCP报文.然后交给网络层处理.MSS指的是TCP报文中数据的长度

5. 远程定位——IP

   通过传输层的TCP协议,我们知道了要发给谁,但是如何发呢?IP负责将数据封装成数据包然后规划好路线发送给接收方.当一台设备上存在多个网卡时,即存在多个IP地址,通过与子网掩码的与运算来得到相对应的IP地址的条目作为发送方的IP地址.TCP报文作为数据加上IP报头生成IP报文.

6. 两点传输——MAC

   通过IP我们规划了数据包传输到接收方的路线,但即使我们知道整体怎么走,但如何传输到下一台设备以及如何定位下一台设备我们都不清楚.所以我们还需要加上MAC头部(记录发送和接收方的MAC地址信息).MAC报文的数据的长度为MTU(以太网中一般为1500字节).MAC地址通过ARP协议确认(首先在ARP缓存中寻找,没有的话利用ARP广播寻找下一台设备的MAC地址,如果下一台设备不再局域网中,就负责该子网的路由器的MAC地址作为下一台设备的MAC地址,由路由器转发到另外一个路由器上).因此我们可以得出在数据报整个的传输过程中,源IP和目的IP是不变的,发送变化的只是发送方的MAC地址和接收方的MAC地址

7. 出口——网卡

   上述阐述的数据包是存放在内存中的二进制数字信息.这种数据包通过网卡将数字信号转为电信号,才能在网线上进行传输.网卡驱动负责控制网卡,在报文头部加上报头和起始帧分界符(作为一个包开头的标识)以及在报文尾部加上FCS(校验数据包的完整和正确).

8. 送别者——交换机

   数据包发送到交换机(二层网络设备)上,交换机本身是没有MAC地址的,它只是负责接收所有发送到本机上的数据包然后解析数据包中的MAC地址并和本机中的MAC缓存表比较看是否有对应MAC地址发送的下一个端口,如果没有则将该数据包发送给所有端口.

9. 出境大门——路由器

   数据包在离开一个子网时需要发送到路由器(三层网络设备)上,路由器的每个端口都是有自己的MAC地址和IP地址,路由器端口负责将数据包中的接收方MAC地址和自己的MAC地址比较,如果一致,则保存到缓存表中,和交换机类似,通过目标IP地址找到下一台设备的IP地址,并通过ARP协议找到相对应的MAC地址

10. 互相扒皮——客户端和服务器

    数据包经过层层传输到达了服务器,服务器将电信号转化为数字信号,然后层层"扒皮"(MAC-IP-TCP),最终得到发送方的HTTP请求信息,服务器得到请求信息后根据请求信息得到响应并将响应作为HTTP数据,然后通过相同的过程发送到客户端,客户端经过层层"扒皮"然后得到服务器的响应信息,然后经由浏览器去进行相应功能的完善.然后客户端向服务器发起了四次挥手,至此双方的连接断开了.

### Linux是如何收发网络包的

#### 接收过程

首先网卡接收到数据包时,为了告诉操作系统网络包已经到达,网卡会中断操作系统.在高性能网络场景下,如果有很多个网络包时,网卡会一直中断操作系统,进而影响整个系统的处理效率.为此,Linux引入了NAPI机制,混合中断和轮询的方式去接收网络包.当有网络包到达网卡时,会执行网卡的中断处理函数,中断处理函数处理完后需要**暂时屏蔽中断(告诉后续接收的网络包不需要再中断,而是直接存到指定的内存中)**,然后唤醒**软中断**(因为硬中断 会消耗CPU资源,所以短暂的开启硬中断后转到软中断去处理网络包的接收)来轮询处理数据,将Ring Buffer中的数据拷贝到缓冲区中,然后交给网络协议栈处理

**软中断就像是寻找众数算法中的那个time**

#### 发送过程

与接收过程刚好相反,先由网络协议栈层层打包数据包,数据包传输到网卡上前,会触发软中断告诉网卡驱动程序,网卡驱动程序通过DMA,从发包队列中读取网络包,将其放入网卡的队列中,然后经由网卡将数据包发送出去

![1658933672261](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658933672261.png)

## HTTP

### HTTP面试题

#### HTTP基本概念

> 1. HTTP是什么?
>
>    HTTP是一个超文本传输协议
>
>    > 协议:可以分成协和议两部分来解释
>    >
>    > > 协:表示有两个及以上的参与者.
>    > >
>    > > 议:表示对参与者行为的一种约定和规范.
>    > >
>    > > HTTP是一个协议,它规定了计算机之间通信的规范以及相关的各种控制和错误处理方式.
>    >
>    > 传输:HTTP是一个双向的协议,既可以从A地传到B地,也可以从B地传到A地.在传输过程中,也可以有中间者进行中转或者接力.
>    >
>    > 超文本:传输的信息不仅仅是一些纯文本,还包括图片,音频,超链接等多种格式的数据.
>
>    总结而言,HTTP就是一个**在两点之间进行双向传输**图片,音频等**超文本**数据的一种**约定和规范**
>
> 2. HTTP常见的状态码有哪些?
>
>    > 1xx:提示信息,表示目前处于中间过程,后续还有其他操作
>    >
>    > ​	1xx的状态码表示当前处于中间过程,在实际应用中很少遇到.
>    >
>    > 2xx:成功,表示报文成功传输且被处理
>    >
>    > > 200:表示一切成功,且响应报文中有body数据
>    > >
>    > > 204:表示一切成功,但响应报文中无body数据
>    > >
>    > > 206:表示响应报文中的资源只是其中的一部分,并不是全部.
>    >
>    > 3xx:重定向,表示访问的资源位置发生了改变
>    >
>    > > 301:永久重定向,客户端需要用其他的url访问该资源
>    > >
>    > > 302:短暂重定向,资源的位置没变,但是暂时需要用其他url访问该资源
>    > >
>    > > 304:不是重定向,表示客户端可以继续使用缓存资源
>    >
>    > 4xx:客户端错误
>    >
>    > > 400:笼统的一个错误,只是表示客户端发生了错误.
>    > >
>    > > 403:表示服务器禁止访问资源,并不是客户端的错误
>    > >
>    > > 404:表示请求的资源在服务器上不能存在或未找到.
>    >
>    > 5xx:服务器错误
>    >
>    > > 500:笼统的一个错误,只是表示服务器发生了错误
>    > >
>    > > 501:表示客户端请求的功能还不支持
>    > >
>    > > 502:服务器作为网关或代理时表示后端的服务器发生了错误
>    > >
>    > > 503:表示服务器当前很忙,无法响应客户端
>
> 3. HTTP常见字段有哪些?
>
>    > Host字段:用来指定访问的服务器的域名
>    >
>    > Content-Length字段:表示服务器的响应报文中数据的长度
>    >
>    > Connection字段:客户端要求服务器要持久连接
>    >
>    > Content-Type字段:服务器回应时,响应报文中数据的格式
>    >
>    > Accept字段:客户端可以接受的数据格式
>    >
>    > Content-Encoding字段:服务器回应时,响应博文中数据的压缩方法
>    >
>    > Accept-Encoding:客户度可以接受的数据的压缩方法

#### Get和Post

> 1. GET和POST有什么区别?
>
>    > 按照RFC规范
>    >
>    > 1. GET是从服务器上获取指定资源;POST是根据请求负荷对指定的资源做处理.
>    > 2. GET是安全,幂等的,可缓存的,POST是不安全,不幂等也不可缓存的
>
> 2. GET和POST方法都是安全和幂等的吗?
>
>    > 在HTTP协议中,**安全**指的是请求方法不会破坏服务器上的资源;**幂等**是多次相同的请求得到的结果是相同.
>    >
>    > **从RFC规范的定义来看**
>    >
>    > **GET方法是安全和幂等的,**因为GET方法是只读操作,只是从服务器上获取到对应的资源,多次执行相同的GET方法获取到的资源是相同的
>    >
>    > **POST方法是不安全和幂等的**,因为它会修改服务器上一些资源,且多次执行相同的请求会创建多个资源.
>    >
>    >  
>    >
>    > 上述阐述是基于RFC规范的,但在实际应用中,不一定会完全遵照RFC规范,因此GET请求也可以用于修改服务器上数据,也可能是不安全和不幂等的,POST请求也可以是只读操作,也可以是安全和幂等的.

#### HTTP特性

> HTTP/1.1的优点
>
> 1. 简单
>
>    HTTP报文由header+body两部分组成,header是简单的key-value内容,易于理解和学习.
>
> 2. 灵活 易拓展
>
>    字段的不限制使用使得开发者可以根据需求随意使用和补充,同时HTTP属于应用层,其下层可以随意变化.
>
> 3. 应用广泛和跨平台
>
>    应用HTTP的应用和软件非常广泛,同时天然具有跨平台的特性
>
> HTTP/1.1的缺点
>
> 1. 无状态
>
>    好处是服务器不需要花费额外的资源去记录状态信息;缺点是服务器没有记忆能力,在完成关联性操作时非常繁琐
>
>    常见的解决方法是引入cookie和session
>
> 2. 明文传输
>
>    好处是方便阅读,方便调试;缺点是重要信息是在传输过程中是暴露在外的,极容易被窃取和篡改
>
> 3. 不安全
>
>    是用明文通信,内容容易被窃听;不验证通信双方的身份,有可能遭遇伪装;无法证明报文的完整性,所以有可能已遭篡改
>
> HTTP/1.1的性能如何
>
> 1. 长连接
>
>    在HTTP/1,0中,一次通信需要建立连接,断开连接.为了避免频繁通信导致服务器频繁建立,断开连接的开销,HTTP/1.1引入了长连接;即在建立了连接后,不会马上断开,之后的通信可以直接展开,知道通信双方有一方主动断开连接.当然如果通信双方长时间没有数据交互,服务器会主动断开连接.![1659049292155](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659049292155.png)
>
> 2. 管道网络传输
>
>    在同一个TCP连接里,客户端可以发起多个请求,且之后的请求不需要等待前面请求响应后才能发送的限制,进而避免了前面连接在服务器异常时,后续请求无法继续发送的问题,也就是**解决了请求的队头阻塞**;**但并没有解决响应的队头阻塞**:服务器必须根据请求接收的顺序来发送响应
>
>    注意:实际应用中HTTP/1.1基本上没有支持管道网络传输
>
> 3. 队头阻塞
>
> 总结:HTTP/1.1相比HTTP/1.0性能有所提升,但整体性能一般般,后续的HTTP/2和HTTP/3就是在优化HTTP的性能

#### HTTP缓存技术

> 1. HTTP缓存有哪些实现方式
>
>    强制缓存和协商缓存
>
> 2. 什么是强制缓存?
>
>    强制缓存是指只要浏览器判断缓存没有过期,则直接使用浏览器本地的缓存
>
>    强制缓存的具体流程:
>
>    1. 浏览器第一次请求服务器的资源,服务器在响应报文中加上Cache-Control字段,设置缓存的过期时间
>    2. 浏览器在之后访问同样的资源时会和Cache-Control中缓存的过期时间对比,如果没有过期则直接使用缓存,否则访问服务器.
>    3. 在访问服务器后,服务器会在响应报文中添加Cache-Control设置缓存的过期时间
>
> 3. 什么是协商缓存?
>
>    与服务器协商后来判断是否使用本地缓存,协商缓存有两种实现方式
>
>    1. If-Modified-Since和Last-Modified
>
>       当资源过期以后,请求报文中的If-Modified-Since会带上Last-Modified的时间,服务器收到请求报文后会发现有If-Modified-Since字段后,会将请求报文中的Last-Modifed的时间与资源最近更新的时间作比较,如果前者比较大,则返回200(返回最新的资源),否则返回304,使用本地缓存
>
>    2. If-None-Match和Etag
>
>       当资源过期时,浏览器发现响应报文中有Etag字段后会再次发送请求报文,并将If-None-Match的值设置为Etag的值发送给服务器,服务器收到请求后会对比资源是否发生了修改,如果修改则返回200,否则返回304
>
>    协商缓存是在强制缓存未命中的基础上实施的.ETag的优先级要高于Last-Modified![1659020090570](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659020090570.png)

#### HTTPS与HTTP

> 1. HTTP与HTTPS的区别?
>
>    > 1. HTTP是明文传输;HTTPS中引入SSL/TLS安全协议,使得数据是加密传输
>    > 2. HTTP建立连接的过程相对简单,是在TCP三次握手之后建立连接;HTTPS在TCP三次握手之后还需要SSL/TLS的握手过程
>    > 3. HTTP的端口号是80,HTTPS的端口号是443
>    > 4. HTTPS需要向CA申请数字证书来证明服务器的身份是可信的
>
> 2. HTTPS解决了HTTP哪些问题?
>
>    1. 窃听风险
>    2. 篡改风险
>    3. 冒充风险
>
>    HTTPS引入SSL/TLS协议来解决HTTP中上述的不安全问题.
>
>    1. 信息加密:采用混合加密的方式
>    2. 校验机制:摘要算法来实现数据的完整性,为每个数据生成一个独有的指纹,指纹用于校验数据的完整性
>    3. 身份证书:将服务器公钥放到身份证书中.
>
>     
>
>    1. 混合加密
>
>       采用非对称加密的方式进行密钥的安全交换;采用对称加密的方式进行加密数据传输
>
>       对称加密只使用一个密钥,运算速度快,密钥必须保密,无法做到密钥的安全交换
>
>       非对称加密有两个密钥:公钥和私钥,公钥可以任意而私钥需要保密,可以解决密钥交换但速度慢
>
>    2. 摘要算法+数字签名
>
>       为了保证数据的完整性,会用摘要算法(哈希)计算出数据的一个指纹(哈希),服务器会对接收到的数据再次计算出一个指纹并与报文中数据的指纹做对比,如果相同则说明数据是完整的.
>
>       为了保证数据和哈希不被替换,需要用数字签名算法**对哈希进行加密**.数字签名主要采用的是非对称加密方式.非对称加密有两个密钥,一个公钥,一个私钥;二者是可以相互加密解密的.
>
>       1. 公钥加密,私钥解密:**保证数据传输的安全性**.私钥是保密的,只有持有私钥的人才能解密数据.
>       2. 私钥加密,公钥解密:**防止发送数据的人不会被冒充**.因为私钥是解密的,所以用公钥能够解密出正确数据说明发送方是持有私钥的.
>
>       数字签名就是采用[私钥加密,公钥解密]的方式保证不被冒充.服务器将公钥发送给客户端,然后用私钥加密数据,客户端如果能够用公钥正常解密,就说明发送的数据来源于服务器
>
>    3. 身份证书
>
>       为了保证服务器发送的公钥不被伪造,引入身份证书.服务器会向CA申请数字证书,并将服务器公钥放到身份证书中,只要证书是可信的,公钥就可信
>
> 3. HTTPS是如何建立连接的?期间交互了什么?
>
>    在经过TCP三次握手之后,进入SSL/TLS的四次握手过程:客户端向服务器索要并验证密钥,客户端和服务器协商产生**会话密钥**
>
>    SSL/TLS建立连接的过程
>
>    1.  客户端向服务器发起加密通信请求,信息:客户端支持的SSL/TLS版本+客户端生成的随机数A+客户端支持的密码套件列表
>    2. 服务器收到客户端的请求后,向客户端发出响应,信息:选择的SSL/TLS版本+服务器生成的随机数B+选择的密码套件+服务器的数字证书
>    3. 客户端收到服务器的响应后,首先通过浏览器或者操作系统得到CA的公钥,确认服务器证书的真实性,之后会取出证书里的服务器的公钥,然后使用它加密报文,信息:一个随机数(pre-master key)(被加密),加密通信算法改变通知,表示之后通信用会话密钥,客户端握手结束通知,并将之前的所有的数据作个摘要,供服务器校验
>    4. 服务器收到报文后,用私钥解密第三个随机数后,根据协商加密算法计算出会话密钥.信息:加密通信算法改变通知,之后通信用会话密钥,服务器握手结束通知,并将之前的所有数据作个摘要供客户端校验
>
> 4. HTTPS的应用数据是如何保证完整性的?
>
>    TLS在实现上分为握手协议和记录协议
>
>    记录协议负责数据的完整性和来源的验证.
>
>    具体过程:
>
>    1. 首先,消息被分割成多个较小的片段,然后分别对每个片段进行压缩.
>    2. 经过压缩的片段会加上消息认证码(MAC值,通过哈希算法生成),这是为了保证数据的完整性,并进行数据的认证.
>    3. 经过压缩的片段和消息认证码会通过对称密码加密
>    4. 经过加密的数据加上报头形成报文交给传输层TCP.

#### HTTP/1.1 HTTP/2 HTTP/3演变

> 1. HTTP/1.1相比HTTP/1.0提高了什么性能?
>
>    1. 使用长连接的方式改善了HTTP1.0的短连接造成的性能开销
>    2. 支持管道网络运输,支持一次发送多个请求报文,减少整体的响应时间
>
>    HTTP1.1性能瓶颈
>
>    1. 报文头部未经压缩就发送,首部信息越多延迟就越大.
>    2. 发送冗长的首部,每次互相发送相同的首部造成的浪费较多
>    3. 服务器是按照请求的顺序响应的,响应可能会造成队头阻塞
>    4. 没有请求优先级控制
>    5. 请求只能从客户端发出,服务器只能被动响应
>
> 2. HTTP/2做了什么优化?
>
>    HTTP/2是基于HTTPS的,在安全性上有保障
>
>    1. 头部压缩
>
>       HTTP/2会压缩头,如果你同时发出多个请求,它们的头是一样或相似的,协议会帮你消除掉重复的部分.
>
>       利用HPACK算法,在客户端和服务器各自维护一张头部信息表,所有字段都会存入这个表,生成一个索引号,以后发送同样的字段时只发送一个索引
>
>    2. 二进制格式
>
>       HTTP/2中的报文由HTTP/1.1的纯文本格式改成了二进制形式.统称为帧:头信息帧和数据帧.这样提高了计算机处理数据的效率.
>
>    3. 数据流
>
>       HTTP/2中的数据包不是顺序发送的,可以乱序发送.同一个连接里连续的数据包可能属于不同的回应.一个请求或响应中的所有数据包称为数据流,为了区分不同的数据包,每一个数据包都会标记一个Stream ID,一个数据流中的数据包的Stream ID是相通的.接收方可以根据Stream ID来组装数据包
>
>       客户端和服务器都可以建立Stream,为了区分,客户端的Stream ID是奇数,服务器是偶数
>
>       客户端可以数据流的优先级是服务器优先/靠后处理
>
>    4. 多路复用
>
>       一个连接中可以同时发送多个请求/响应(HTTP/1.1不能解决响应队头阻塞的问题).
>
>    5. 服务器推送
>
>       服务器可以主动向客户端发送信息,较少了消息传递的次数
>
>    HTTP/2虽然采用多路复用解决了响应的队头阻塞问题,但仍无法完全解决队头阻塞问题.问题处在TCP层,TCP是面向字节流的,所以TCP层接收到的数据要求是完整且连续的,这样才能从内核中将数据提供给HTTP.但当发生丢包现象后,由于要保证连续,所有包都需要等待丢包重传后才能传给HTTP,因此造成了阻塞.
>
> 3. HTTP/3做了什么优化?
>
>    HTTP/3完全解决了队头阻塞问题:将HTTP下层的TCP改成了UDP.
>
>    UDP对发送顺序和连续性没有要求,所以可以解决队头阻塞的问题.但UDP是不可靠的,但基于UDP的QUIC协议可以实现类似TCP的可靠传输
>
>    QUIC的3个特性
>
>    1. 无队头阻塞
>
>       QUIC同样采用HTTP/2中的多路复用与Stream的概念.与HTTP/2不同的是丢包后QUIC只会影响丢的包本身,而不会影响其他包
>
>    2. 更快的连接建立
>
>       在HTTP/3之前,TCP和TLS是分层的,分别属于传输层和表示层,所以需要分批实现,先TCP握手,然后TLS握手
>
>       HTTP/3中少了TCP握手,而TLS握手转变成了QUIC握手,由四次握手改变为了三次握手(2个RTT---->1个RTT),甚至可以是两次握手(0个RTT).握手的目的是确认双方的连接ID.
>
>    3. 连接迁移
>
>       基于TCP的HTTP协议是通过四元组(源IP,源端口,目的IP,目的端口)来确认连接的,当移动设备的网络环境发生变化(从流量切换到WIFi)时,对应的IP地址发生了改变,因此连接会发生断开再重连,重连需要经过TCP三次握手和TLS四次握手的时延,连接迁移的成本很大.
>
>       而QUIC是基于连接ID来确认连接.这样即使IP地址变化了,只要上下文信息仍保持,连接就无须重连,可以继续正常使用
>
>    总的来说,QUIC是一个在UDP之上结合了TCP+TLS+HTTP/2的多路复用的协议,但由于QUIC是新协议,许多网络设备会将其当做是UDP,而有些网络设备会丢弃UDP包,这样就会导致丢弃QUIC包.因此HTTP/3的普及速度非常缓慢.

### HTTP/1.1如何优化

 #### 如何避免发送HTTP请求?

对于一些重复性的请求,我们可以通过本地缓存来避免发送重复的HTTP请求.在之后请求相同的资源时,可以直接从本地的缓存中获取(资源过期前).而缓存由强制缓存和协商缓存构成,具体可以参考[HTTP面试题:HTTP缓存技术].

#### 如何减少HTTP请求次数?

1. 减少重定向请求次数

   当发生重定向时,客户端需要再次用新的url去访问服务器,为了较少重定向的请求次数,可以通过服务器(代理服务器)用新的url去获取资源然后返回给客户端

2. 合并请求

   将访问多个小文件的请求合并成一个大的请求,减少了重复发送的HTTP头部.

   尽管HTTP/1.1支持长连接,但不同的请求都需要客户端向服务器发送一个请求头,将N个小文件合并成1个大文件就可以减少(N-1)RTT的时间

   合并请求的方式

   1. 将多个小图片合并成一个大图片(Css Image Sprites)
   2. 服务器使用webpack等打包工具将css,js等资源打包成大文件

   合并请求本质上是合并资源,合并请求也有不好的地方,比如当一个大文件中某个小文件资源发生变化就需要重新发送整个大文件,或者服务器对大文件的处理时间要长于合并请求节省的时间

3. 延迟发送请求

   每次请求的时候,只请求当必须获取的资源,对于一些不必要的资源可以延迟获取.

#### 如何减少HTTP响应的数据大小? 

对响应的数据压缩.减少响应报文的大小,提高报文的传输效率.

1. 无损压缩:资源经过压缩后,不会被破坏,解压后能够得到完整的数据.常见的无损压缩算法有gzip,Google的Brotli算法.
2. 有损压缩:资源经过压缩后,会造成小幅度的破坏,解压后能够得到和原数据非常相似的数据,通过降低压缩的质量来提高压缩比.常见的有损压缩:视频:H264,H265;音频:AAC,AC3

### HTTPS RSA握手解析

#### TLS握手过程

HTTPS引入SSL/TLS协议解决了HTTP不安全的问题:易被窃听,易被篡改,易被冒充.具体解决过程参考[HTTP面试题:HTTPS与HTTP:HTTPS解决了HTTP的哪些问题]

不同的密钥交换算法,TLS的握手过程也会发生一些变化.常见的密钥交换算法有RSA,ECDHE.

#### RSA握手过程

1.  TLS第一次握手

   客户端会发送一个[Client Hello]的消息,消息里面有随机数(用于之后会话密钥的生成),客户端支持的TLS版本号,支持的密码套件列表.

2. TLS第二次握手

   服务器收到客户端的[Client Hello]消息,会确认消息中的TLS版本号是否支持,从密码套件列表中选出一个密码套件,并生成一个随机数,服务器会向客户端发送响应报文,报文中的消息包括生成的随机数(用于之后会话密钥的生成),选择的版本号和选择的密码套件以及用于验证自身身份的身份证书.

   密码套件的基本形式:[密钥交换算法+签名算法+对称加密算法+摘要算法]

   如:“Cipher Suite: TLS_RSA_WITH_AES_128_GCM_SHA256”,表示的含义是密钥交换算法和签名算法采用RSA,对称加密算法选择AES,生成的密钥长度为128位,摘要算法为SHA256.

   **这个随机数也是经过公钥加密发送的.这个公钥时客户端的公钥**

   **具体过程:首先客户端是拥有服务器的公钥1的,然后随机生成一对公钥2和私钥2,然后客户端用公钥1加密公钥2发送给服务器,服务器用私钥1解密得到客户端的公钥2,随机数就是用公钥2加密发送的,客户端可以用私钥2解密得到随机数**

3. 客户端验证证书

   客户端验证服务器的身份证书参考[HTTP面试题:HTTPS与HTTP:HTTPS如何建立连接,期间发生了哪些交互]

4. TLS第三次握手

   客户端验证完证书后,会生成一个新的随机数(用于会话密钥的生成),并将该信息用RSA公钥加密发送给服务器,并将上述过程中的数据作个摘要(用会话密钥加密,以验证该会话密钥是否能够正常使用以及验证之前数据的完整性),用于服务器的校验以及告知服务器之后采用会话密钥通信

5. TLS第四次握手

   服务器用RSA私钥解密并生成会话密钥,之后做相同的动作:将上述数据作摘要(同样用会话密钥加密)供客户端校验,并告知客户端之后采用会话密钥通信.

   当服务器和客户端对会话密钥的验证无误后,之后就用会话密钥进行加解密数据传输

#### RSA算法的缺陷

使用RSA算法生成会话密钥的最大问题是不支持前向保密.

客户端给服务器传递用公钥加密的随机数时,一旦服务器的私钥泄露,第三方就可以通过私钥去解密之前拦截的所有数据密文,为了解决这个缺陷引出了ECDHE算法(现在大多数网站使用的一种算法)

### HTTPS ECDHE握手解析

#### DH算法

假设客户端和服务器各自随机生成一对公钥和私钥,分别标记为A,a,B,b.每一方的公钥A/B = G^a/b % P,其中G是底数,P是模数(通常是一个比较大的质数).当我们知道a/b,G,P时是很容易计算出A/B的,但反过来知道A/B,G,P是很难计算出a/b的.A,B是公开的,所以客户端和服务器交换公钥后,客户端通过B^a%P计算出会话密钥K,服务器通过A^b%P计算出会话密钥K,由于交换律二者计算的会话密钥是相等的

#### DHE算法

在DHE算法出现之前,还有一种DH算法叫做static DH算法.static DH算法是指客户端每次随机生成公钥而服务器的公钥是固定不变的.这样当交互海量数据后,黑客就可以通过这些数据中的有效信息暴力破解出服务器的私钥,进而可以计算出会话密钥.因此static DH算法不具备前向安全性.

DHE算法就是每次会话客户端和服务器随机生成一种公钥,这样即使黑客破解出这次通信的会话密钥,也不会影响其他的通信.通信过程之间是独立的,保证了前向安全.

#### ECDHE算法

 由于DHE算法需要计算幂,所以需要计算大量的乘法.为了提升DHE算法的性能推出了ECDHE算法

ECDHE算法是利用ECC椭圆的特性可以用更少的计算量计算出公钥.

具体过程:

1. 双方事先规定好使用哪一个椭圆曲线,并在椭圆曲线上选择一个基点G
2. 双方各自随机生成一个随机数作为私钥a,并与基点相乘得到公钥aG
3. 双方交换各自的公钥并与自己的私钥相乘,并将结果映射到椭圆曲线的一个点上,两个点的x坐标是一致的,这个x坐标就是会话密钥

#### ECDHE握手过程

大致过程和RSA 握手过程是相似的.

不同的是第二次握手时,由于采用ECDHE密钥协商算法,所以服务器还需要选择一条椭圆曲线以及基点G,并通过生成随机数确定私钥,以及私钥和基点确定公钥,并将椭圆曲线利用RSA签名算法签名后发送给客户端.

第三次握手时客户端再验证了服务器的身份后,根据椭圆曲线和基点G生成自己的公钥,并结合服务器的公钥计算出会话密钥(x坐标),并将自身生成的公钥发送给服务器,服务器由此生成会话密钥

最终的会话密钥是由客户端生成的随机数+服务器生成的随机数+坐标x共同构成



RSA和EDCHE的区别

1. RSA不支持前向保密,EDCHE支持前向保密
2. RSA必须在四次握手之后才能发送数据,EDCHE的客户端不需要等服务器的第四次握手既可以发送数据
3. 使用EDCHE在第二次握手时还需要发送Server Key Change:椭圆曲线+基点G

### HTTPS如何优化

相比于HTTP协议,HTTPS协议保证了安全性.因此对HTTPS的优化是非常必要的.

#### 分析性能损耗

产生性能消耗的两个环节

1. TLS握手过程

   > 1. 握手过程中增加了网络延时(最多花费2RTT).
   > 2. ECDHE密钥协商算法,握手过程中客户端和服务器都需要临时生成椭圆曲线公私钥
   > 3. 客户端验证证书时,需要用CA公钥去解密身份证书
   > 4. 双方都需要计算pre-master,也就是对称加密密钥.

2. 握手后的对称加密报文传输:主要的是加密算法上的优化.但现在的技术已经可以让这一个环节的性能消耗变得非常小了

#### 硬件优化

玩游戏最快战胜对方的方式就是[充钱].软件都是在硬件上跑的,想要提高软件的效率,最直接的方法就是提高硬件的质量. 因为HTTPS是计算密集型,所以主要提升的硬件应该是CPU.

#### 软件优化

软件升级.但升级大量服务器花费的人力物力成本也是不容小觑的.因此软件优化的重心是**协议优化**

#### 协议优化

##### 密钥交换算法优化

用ECDHE算法代替RSA算法,既能提高安全性(前向安全),也能减少1个RTT(2RTT-->1RTT).

在选用ECDHE算法时,可以选择x25519椭圆曲线(速度最快),如果对性能要求不高,对称加密算法可以选择AES-128-GCM,它生成的会话密钥长度更短,速度更快.

##### TLS升级

将TLS从1.2升级为1.3

TLS1.2握手过程中,需要4次握手:Client Hello(1),Server Hello(2)协商出需要使用的加密算法,互换公钥(3,4),需要花费2个RTT;TLS1.3握手过程中,将hello和交换公钥进行了合并,只需要1个RTT.

TLS1.3握手过程:首先客户端在Client Hello消息里带上支持的椭圆曲线,基点以及自己根据支持椭圆曲线生成的公钥;服务器收到消息以后选取一条椭圆曲线,然后根据该椭圆曲线生成自己的公钥并发送给客户端.至此,客户端和服务器都具备了生成会话密钥的材料,于是客户端就可以开始利用会话密钥进行通信了.![1659310439221](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659310439221.png)

除此之外,TLS1.3的密码套件列表进行了缩减,对于密钥交换算法,只支持ECDHE算法,对于对称加密和签名算法只支持最安全的几个.

 #### 证书优化

证书指的是服务器用以验证自身身份的身份证书

##### 证书传输优化

通过减少证书的大小来提高证书传输的速度,因此选取椭圆曲线证书来代替RSA证书,因为椭圆曲线公钥的长度要远小于RSA公钥.

##### 证书验证优化

客户端在验证服务器的身份证书时,为了验证证书是否被CA吊销,需要去访问CA,下载**CRL**或者**OCSP数据**来确认证书的有效性

CRL:CA维护的一张证书吊销的列表,当一张证书存在于CRL中时,说明该证书已被吊销,不可用.

CRL的问题:

1. 实时性差:因为CRL是CA定期更新的一张表,所以客户端在验证身份证书时,该证书可能当前并不存在于CRL中,但已经被CA吊销了.
2. 效率低:首先客户端必须访问CA,然后查询CRL,但随着吊销证书的增加,CRL表的增大,从CRL中查询出想要的结果的资源耗费是比较大的.

OCSP:在线证书状态协议.通过向CA发送查询请求,让CA返回证书的有效状态.

因为是实时查询,所以解决了CRL实时性差的问题,但查询的效率依旧不高.

OCSP Stapling:服务器定期向CA查询证书的状态并将带有时间戳(验证有效期)和签名(防治服务器篡改)的结果缓存到本地,这样客户端在验证证书是否被吊销时,服务器直接返回一个结果就可以.

#### 会话复用

HTTP连接时会生成一个对称加密密钥,这个密钥每次连接都会生成.通过缓存首次HTTP连接生成的对称加密密钥,直接**复用**该密钥,减少TLS握手性能的损耗.

##### Session ID

客户端和服务器首次建立连接时生成的对称加密密钥会被缓存到本地,并用一个唯一映射该密钥的Session ID来标识.这样在之后的连接中,客户端只需要发送自己的Session ID给服务器,就可以直接开始通信.为了安全,缓存中的对称加密密钥会定期失效.

Session ID的缺点:

1. 服务器需要保存每个客户端的会话密钥,随着客户端的增多,服务器的内存压力会增大.
2. 当多台服务器负载均衡提供服务时,客户端发送的Session ID很难命中之前建立连接的服务器,也就是说,这次请求到达的服务器很有可能不存在该Session ID,因此还需要重新建立连接生成新的会话密钥.

##### Session Ticket

Session Ticket将任务交给了客户端,类似于HTTP的cookie.

客户端和服务器首次建立连接生成的会话密钥,服务器会对其加密生成Ticket发送给客户端,客户端再建立连接时发送该Ticket给服务器,服务器解密Ticket并验证其有效性,若有效,则用该会话密钥进行通信.

对于集群服务器而言,要保证每台服务器加密会话密钥是一致的,这样就会解决Session ID的第2个问题.

Session Ticket的缺点:**不具备前向安全性**,当会话密钥被破解后,之前的很多通信数据都会被破译.同时也很难应对重放攻击.

重放攻击:中间人在拿到Session ID/Ticket时,通过监听客户端和服务器之间的对话,能够冒充客户端与服务器进行重复的相同或类似的通信使服务器中的数据并不是客户端想要的结果

解决重放攻击的方式就是给会话密钥设置有效时间

##### Pre-shared Key

Session ID/Ticket都需要1RTT才能恢复会话,而Pre-shared Key可以实现0RTT .

原理和Ticker类似,只是在发送的时候将Ticket和请求一同发送给服务端,这种方式交Pre-shared Key.



**使用会话重用是会可能受到重放攻击的,因此要给缓存的会话密钥一个有效期,或者只针对安全的HTTP请求,如Get使用会话重用,**

### HTTP/2

#### HTTP/1.1协议的性能问题

1. 延迟难以下降:随着带宽的增加,已经达到了延迟下降的下限.
2. 并发连接有限:软件对并发数量的限制,而且每个连接都要经过TCP和TLS握手耗时,以及TCP慢启动过程给流量带来的影响.
3. 队头阻塞问题:同一连接中只能在完成一个事务(请求和响应)后,才能进行下一个事务
4. 头部巨大且重复:因为HTTP是无状态的,所以每个数据包都需要携带头部.
5. 服务器不能主动推送:服务器不能主动向客户端发送消息,客户端需要获取通知时,还需要客户端主动向服务器定时发送请求.

#### 兼容HTTP/1.1

HTTP/2兼容了HTTP/1.1,在用户使用方面,url的格式没有发生改变,只是在背后升级协议,使协议进行了平滑的升级;只在应用层作了改变,在传输层还是基于TCP传输的,在应用层又分为语义和语法两部分,**语义不改动,大幅度改动了语法部分**

#### 头部压缩

HTTP协议是由Header+Body两部分组成,Body部分可以通过字段Content-Encoding提供的压缩方法进行压缩,但并没有压缩头部.

HTTP/1.1报文中Header部分存在的问题

1. 含有很多固定的字段,这些字段应该被压缩
2. 请求和响应报文中有很多重复的字段,应该避免重复字段占用冗余空间
3. 字段是ASCII编码,应该改为二进制编码

HTTP/2开发了HPACK算法来对报文的头部进行压缩.HPACK算法包括三部分:

1. 静态字典

2. 动态字典

3. Huffman编码

   客户端和服务器会维护字典,用一个长度较小的索引去代替重复的字符串,再用Huffman压缩数据.

##### 静态表编码

HTTP/2为高频出现在头部的字段和字符串建立了一张静态表,这个表不会变化,共有61组,有Index(索引),Header Name(字段名),Header Value(字段值)

对于静态表中只有字段名而无字段值的数据,会根据Huffman编码动态生成一个数据.

##### 动态表编码

不在静态表中出现的字段会在动态表中构建,动态表是实时更新的,索引从62开始.

对于一个需要重复发送的字段,在第一次在动态表构建好后,客户端和服务器的动态表中都会生成该字段的索引,在之后再使用该字段及该字段的值时会利用索引代替,这样就避免了大量重复的字符串去占用空间的问题.

但动态表的大小不能无线扩大,否则会造成服务器性能变差.因此服务器会提供一种配置来限制动态表的最大大小.



**利用静态表,动态表和Huffman编码来对HTTP报文的头部进行压缩.**

#### 二进制数据帧

HTTP/2将HTTP/1.1中的纯文本格式改进为二进制格式的数据,提高了HTTP传输效率.

HTTP/2中数据包被划分为两种类型的帧:首部和消息负载.![1659402073395](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659402073395.png)

其中,帧头只有9个字节,其中前3个字节表示数据帧的长度,后面的帧类型表示数据帧的传输类型:数据帧/控制帧,标志位用于携带简单的控制信息,流标识符就是Stream ID,用以区分一个数据包属于哪个数据流,其中首位保留不用.

帧数据中存放的是利用HPACK算法压缩的头部和数据.

#### 并发传输

HTTP/1.1中存在响应报文队头阻塞的问题,HTTP/2解决了HTTP/1.1中的队头阻塞问题,可以实现在一个连接中并发传输.

HTTP/2实现并发传输的关键是Stream,一个连接中可以并发传输多个Stream,一个Stream中可以有多个Message,一个Message中含有多个Frame,Frame是以二进制格式压缩的内容(头部和数据).一个连接中,不同Stream的数据帧时可以乱序发送的,由Stream ID用以区分不同的数据帧,其中客户端数据帧的Stream ID是奇数,服务器是偶数.同时Stream ID是递增的,而不会发生重复.所以当Stream ID递增到某一个上界时,会通过一个控制帧来关闭该连接,同时HTTP/2中还可以为不同的数据帧设置优先级(在帧的帧头的标志位中描述)

**不同的Stream中的帧(Frame)是可以乱序发送的,因为有Stream ID保障,但同一个Stream中的帧必须是严格有序的**,即发送的时候是一个请求的头部就必须在接收端第一个被接收,发送的时候是一个请求的尾部就必须在接收端最后一个被接收.

#### 服务器推送

在HTTP/2中可以实现服务器主动推送消息.具体实现为:客户端向服务器发送请求,服务器在返回响应的同时会告诉客户端接下来要推送的消息的具体的Stream ID,然后并发的发送要推送给客户端的消息.

### HTTP/3

#### HTTP/2的不足

HTTP/2从头部压缩,二进制帧,并发传输,服务器推送等方面大幅度提升了HTTP/1.1的特性,但在队头阻塞,TCP与TLS握手延迟,网络迁移需要重新连接方面存有缺陷.

##### 队头阻塞

HTTP/2中多个请求是可以在一个TCP连接中传输的,当TCP发生丢包重传时,整个TCP连接里的请求都需要等待丢的包重传后才能被HTTP层接收.

因为TCP是面向字节流的,所以TCP接收到的数据必须是完整且有序的,当序号较小的TCP包发生丢包后,序号较大的TCP包即使被接收HTTP层也无法从内核中获取到,必须等丢掉的包到达后才能整体被HTTP接收.**这种阻塞影响的不仅仅是同一个流,而是同一个连接中的所有流.**

![1659410154365](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659410154365.png)

​	**因此HTTP/2中队头阻塞问题是发生在TCP传输层的.**

##### TCP与TLS握手延迟

发送HTTP数据前,需要经历**TCP三次握手和TLS四次握手**之后才能正常通信,整个过程需要消耗3RTT.另外由于TCP的拥塞控制的特性,还需要等待**TCP的一个慢启动**的过程.

##### 网络迁移需要重新连接

HTTP/2中是基于四元组[源IP,源端口,目的IP,目的端口]来定位接收方的,在数据传输过程中,四元组是不发生改变的,这意味着当通信一方如果网络发生迁移(如从流量到WIFi),建立的连接就会因为IP地址的改变而断开,本次连接中发送的数据就无法到达接收方,需要重新发送.而重新连接有需要经历TCP与TLS握手的延迟.这个弊端是基于TCP建立在有连接的基础上进行通信的,因此要想优化这个弊端,就需要更换传输层协议,HTTP/3就是如此.

#### QUIC协议

HTTP/3采用了基于UDP的QUIC协议来代替TCP协议.UDP是面向无连接的,不可靠的一个传输层协议.且UDP是面向数据报的,对数据的顺序是无要求的.QUIC在UDP的基础上又实现了类似于TCP的连接管理,拥塞控制,流量控制等特性来实现可靠传输.

QUIC协议就可以解决HTTP/2协议中的那些弊端

##### 无队头阻塞

因为QUIC协议是基于UDP的,UDP对数据的顺序是没有要求的,所以对于不同Stream中的帧,即使发生丢包现象,也只会影响这一个Stream中的数据的到达(因为同一个Stream中的帧是要求严格有序的),而不会影响其他Stream中数据的到达.

##### 更快的连接建立

对于HTTP/3之前的HTTP协议,TCP和TLS是分层的,分别属于内核实现的传输层和openssl库实现的表示层,因此需要先TCP握手,再TLS握手

而HTTP/3在传输前需要进行QUIC握手,这个握手只耗费1个RTT,为了确定客户端和服务器的[**连接ID**],而因为HTTP/3中每个帧中会携带TLS的记录,且HTTP/3采用的是TLS1.3,所以QUIC握手和TLS握手同时进行,**耗费1RTT**,甚至在第二次连接时**采用会话复用实现0RTT**建立连接

##### 连接迁移

HTTP/3之前的协议是基于四元组确认TCP连接,这种连接会随着网络迁移而断开,QUIC协议是基于连接ID来确认连接的.因此即使因为网络环境变化导致IP地址变化,只要上下文信息不会令连接ID改变,连接就不会断开.

#### HTTP/3协议

HTTP/3采用的也是二进制数据帧,与HTTP/2不同的是,HTTP/3可以直接使用QUIC协议提供的Stream,而不需要在帧头中添加关于Stream的标识![1659412745467](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1659412745467.png)

在进行头部压缩时,HTTP/3对头部压缩算法进行了升级:QPACK ,QPACK也采用了静态表,动态表和Huffman编码,与HTTP/2不同的是,HTTP/3中QPACK的静态表中的选项从61扩展到了91项,且动态表的使用也与HTTP/2有所差异.

HTTP/2在首次发送协议后客户端和服务器会将静态表中没有包含的字段更新到各自的动态表中,后续就会使用该字段对应的索引数字.但这样存在时序性问题,当首次发送的协议发生丢包时,服务器并没有更新动态表,而客户端在之后的发送过程中使用的数字索引就无法被服务器识别.

因此HTTP/3改善了这一弊端:QUIC有两个特殊的单向流,只允许一方发送消息给另一方.数据传输时使用的是双向流,这两个单向流用以更新动态表.

一个单向流负责将字典传递给对方,客户端可以将不属于静态表的字典发送给服务器

一个单向流负责响应对方,服务器通过这个单向流告知客户端自己的动态表是否已经更新.

打个比方:

HTTP/2相当于我直接给你发信息,信息里面含有我之后发信息的暗号表,之后我就直接用暗号给你发信息,没有考虑第一次对方接收不到信息的情况.

HTTP/3相当于我在通信前先把暗号表发送给你并等待你的确认通知,对方确认收到后会发送确认通知给我,当确认双方都有暗号表之后,我再用暗号发送信息给你.