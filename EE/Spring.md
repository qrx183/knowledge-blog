# SSM

复习策略:

1. 过笔试:剑指Offer+刷牛客网上的在线OJ热门TOP101
2. 过面试:学完比特的基础课程+刷面经/背八股文

投递简历的顺序:

1. 首先投知名公司
2. 然后投"中"字头开头的公司
3. 然后按照兴趣投相应的公司

## Spring

1. 定义:Spring是包含了众多工具方法的ioC容器

   > 容器:用来容纳某种物品的装置,如List/Map(数据存储容器),Tomcat(Web容器)
   >
   > ioC(Inversion of Control):控制反转,将存储的对象的生命周期交给容器而无需程序员自身关注和处理
   >
   > ioC的优点:实现代码的解耦,对象的生命周期交给ioC框架来维护,而程序员无须关注.

2. Spring ioC功能

   > 1. 将对象**存储**到Spring(容器)中
   > 2. 将对象从Spring(容器)中**取**出来

3. DI(Dependency Injection):依赖注入.

   > IoC和DI的区别
   >
   > > IoC是控制反转,DI是依赖注入
   > >
   > > IoC是一种思想,DI是一种具体的实现

4. Spring项目的创建

   > 1. 创建一个maven项目
   >
   > 2. 在Maven项目中添加spring框架的支持:spring-context,spring-beans
   >
   >    > 如果没办法正常导入,就需要引入国内源:在setting中的maven下重写并在相应文件夹下导入settings.xml文件.
   >    >
   >    > ![1657594932908](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657594932908.png)
   >    >
   >    > ![1657594969344](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657594969344.png)
   >    >
   >    > 在创建一个新的spring项目时,需要在"新项目设置"中重新加载settings.xml
   >    >
   >    > ![1657594983843](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657594983843.png)
   >    >
   >    > ![1657595005631](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657595005631.png)
   >
   > 3. 创建一个启动类并添加main方法

5. Spring的两个主要功能的实现

6. > 1. 将Bean(对象)存储到Spring中
   >
   >    1. 第一次添加需要先在Spring项目中添加配置文件,非第一次不需要这一步
   >
   >    2. 创建一个Bean对象类
   >
   >    3. 在配置文件中将需要存储到Spring中的bean对象注册到配置文件中
   >
   >       ![1657617842978](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657617842978.png)
   >
   >       id是key,claas是value
   >
   > 2. 将Bean对象从Spring中读取出来
   >
   >    1. 先得到Spring上下文对象
   >
   >       ![1657618310142](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657618310142.png)
   >
   >       和resources文件中的配置文件的文件名要相一致
   >
   >        
   >
   >       ![1657619159188](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657619159188.png)
   >
   >       以上两种方式都可以得到Spring的Context对象
   >
   >       ApplicationContext和BeanFactory的区别
   >
   >       1. ApplicationContext是BeanFactory的子类.BeanFactory只提供了基础访问Bean的方法,而ApplicationContext除了拥有BeanFactory的功能外,还提供了更多的方法实现,比如对国际化的支持,资源访问的支持,以及事件和传播等方面的支持.
   >       2. 从性能方面来看,BeanFactory是按需加载Bean,类似于懒加载,而ApplicationContext是饿汉方式,在创建时会将所有的Bean对象都加载起来,以备以后使用.
   >
   >    2. 通过上下文对象提供的方法来获取到需要的Bean对象
   >
   >       ![1657618380201](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657618380201.png)
   >
   >       按照id来获取Bean对象,这种获取对象存在的问题是当获取到的对象是NULL时,强转会发生异常.
   >
   >       ![1657629961470](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657629961470.png)
   >
   >       按照类来获取Bean对象,这种获取对象的方法存在问题:当配置文件中配置了多种User时,无法识别具体获取哪个User对象.
   >
   >       ![1657630141923](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657630141923.png)
   >
   >       推荐使用这种方式,获取到对象的方法更加健壮
   >
   >       较之于new出一个User对象,**这种控制反转的优势是解耦**
   >
   >    3. 使用Bean对象
   >
   >       ![1657618502719](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657618502719.png)
   >
   >       ![1657618508298](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657618508298.png)

7. 利用更简单高效的方式实现Spring的两个主要的功能:利用注解

   > 在Spring的配置文件中设置Bean的扫描路径
   >
   > 1. 将Bean对象存储到Spring容器中
   >
   >    > 使用五大类注解
   >    >
   >    > 1. @Controller(控制器):负责前端参数的检验
   >    > 2. @Service(服务):负责数据组装和分配接口
   >    > 3. @Repository(仓库):负责和数据库直接交互
   >    > 4. @Configuration(配置):存储配置文件
   >    > 5. @Component(组件):存储工具文件
   >    >
   >    > 之所以分成5个注解,是为了提高代码的可读性,让程序员能够直接得到某个文件的大致含义
   >    >
   >    > 5大类注解之间的联系:@Controller,@Service,@Repository,@Configuration都是基于Component实现的.
   >    >
   >    > 使用@Bean注解:该注解添加在某个具体的方法中,但需要搭配五大类注解使用
   >    >
   >    > ![1657672270666](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657672270666.png)
   >    >
   >    > ![1657672309413](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657672309413.png)
   >    >
   >    > 从Spring中取出Bean对象时的id为方法名的小驼峰形式(这里的小驼峰是spring遵循了jdk的变量命名的规则:如果前两个字母均为大写则返回源变量名,否则把原变量名的第一个字符变为小写之后返回)
   >    >
   >    > ![1657672955842](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657672955842.png)
   >    >
   >    > 重命名Bean![1657673927348](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657673927348.png)
   >    >
   >    > 重命名Bean(一个Bean可以被重命名多个名称)以后,通过id获取对象时只能使用重命名的id名,**使用方法名不能正常获取到对象**
   >    >
   >    >  
   >    >
   >    > @Bean将一个类型注入多次引入的问题
   >    >
   >    > 在类中引入该类型的对象时容易发生识别出多个而不知选哪个的问题
   >    >
   >    > 解决方案
   >    >
   >    > > 1. 将变量名改成对应的Spring容器中存储的对象的id名称
   >    > > 2. 利用@Resource(name = "id名称")来代替@Autowired
   >    > > 3. 利用@Autowired+@Qualifier(value = "id名称")来使用
   >
   > 2. 从Spring中获取到Bean对象(对象装配/对象注入)
   >
   >    > 1. 通过@Autowired属性注入:将spring容器中的user对象赋值给UserController的user属性![1657675275782](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657675275782.png)
   >    >
   >    > 2. 通过构造方法实现对象注入(官方认证的对象注入方式)
   >    >
   >    >    ![1657675496650](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657675496650.png)
   >    >
   >    > 3. Setter注入
   >    >
   >    >    ![1657675758275](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657675758275.png)
   >    >
   >    > 属性注入,构造方法注入即setter注入的区别?
   >    >
   >    > 1. 属性注入:写法简单,但是通用性差,只能运行在IoC容器下
   >    > 2. setter注入:早起Spring推荐的写法,Setter注入通用性要差于构造方法注入(其他高级语言大多不再用java的这种get和set方法)
   >    > 3. 构造方法注入:现在官方推荐的写法,通用性最好
   >    >
   >    > **在使用属性注入和setter注入时,可以用@Resource来代替@Autowired,但构造方法注入不支持@Resource.@Resource是java提供的,而@Autowired是Spring提供的**
   >    >
   >    > @Resource和@Autowired的区别
   >    >
   >    > 1. 出身不同:@Resource来自于JDK,@Autowired来自于Spring
   >    > 2. 用法不同:@Resource支持属性注入和Setter注入,@Autowired支持属性注入,Setter注入和构造方法注入
   >    > 3. 支持的参数不同:@Resource支持更多的参数设置(name,type..),而@Autowired只支持required参数的设置
   >

8. Bean作用域和生命周期

   1. Bean的作用域:Bean在Spring整个框架中的某种行为

      > singleton:单例模式(默认):无状态(对象不发生更新)时使用单例模式
      >
      > prototype:原型模式(多例模式):有状态(对象会发生更新或者可能发生更新)时使用多例模式
      >
      > request:请求作用域(Spring MVC):一次请求和响应,类似于多例模式
      >
      > session:会化作用域(Spring MVC):共享会话,类似于单例模式
      >
      > application:全局作用域(Spring MVC)
      >
      > websocket:HTTP WebSocket的作用域(Spring WebSocket)
      >
      >
      >
      > 单例作用域(singleton)VS全局作用域(application)
      >
      > 1. 单例作用域作用于IoC容器,全局作用域作用于Servlet容器
      > 2. 单例作用域是Spring Core的作用域,全局作用域是Spring Web的作用域

   2. Bean的作用域的设置:利用@Scope

      > @Scope("prototype")
      >
      > @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)

   3. Bean的生命周期

      1. 实例化Bean(为Bean分配内存空间)
      2. 设置属性(对象注入)
      3. 初始化
         1. 执行各种Aware(通知)
         2. 执行初始化的前置方法
         3. 执行构造方法,一种是执行@PostConstruct,另一种是执行 init-method
         4. 执行初始化的后置方法
      4. 使用Bean
      5. 销毁Bean

## SpringBoot

1. 定义:SpringBoot是Spring框架的脚手架,目的是为了简化Spring的开发

2. 优势

   > 快速集成框架,Spring Boot 提供了启动添加依赖的功能,用于秒级集成各种框架
   >
   > 内置运行容器,无需配置Tomcat等Web容器,直接运行和部署程序.
   >
   > 快速部署项目,无需外部容器即可启动并运行项目
   >
   > 可以完全抛弃繁琐的XML,使用注解和配置的方式进行开发
   >
   > 支持更多的监控的指标,可以更好的了解项目的运行情况

3. SpringBoot项目的创建

   1. idea创建SpringBoot项目

      ![1657722169055](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657722169055.png)

      选择和本地的jdk对应的版本号(jdk8

      ![1657722225966](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657722225966.png)

      勾选对应的依赖

   2. 网页版创建Spring Boot项目

      访问https://start.spring.io/网站![1657722381153](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657722381153.png)



      配置好依赖后点击"GENERATE"得到一个demo.zip,解压zip项目并用idea打开项目![1657722488053](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657722488053.png)

4. SpringBoot的设计思想:约定大于配置.**默认扫描路径是扫描启动类所在包下的所有类和包**

5. SpringBoot配置文件

   1. 配置文件的分类

      > 系统的配置文件:比如连接字符串,日志的相关设置.是系统已经定义好的.
      >
      > 用户自定义的配置文件:

   2. 配置文件的格式   注:SpringBoot约定配置文件的名称必须为application,在SpringBoot中,如果出现两种配置文件,会默认使用版本更老的.properties的配置文件

      > .properties
      >
      > 基本格式:key=value(不要有空格,容易产生歧义)
      >
      > 注释以#开头
      >
      > 读取配置文件的项:利用@Value注解将对应的value值属性注入.
      >
      > ![1657763704239](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657763704239.png)
      >
      > 注意:@Value括号中的字符串要用$和{}修饰(SpringBoot约定)
      >
      > .properties的缺点:配置文件中的key值存在较多的冗余信息,改善方式就是用.yml配置文件
      >
      >
      >
      > .yml
      >
      > 基本格式: key:​ value,key和value之间使用一个":"和一个" ''分割
      >
      > .yml支持多级显示(减少冗余)![1657764486182](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657764486182.png)
      >
      > 单双引号的问题:在双引号中转义字符会自动发生转义,而在单引号中不会自动发生转义
      >
      > yml还支持配置对象或集合,对象和集合不能用@Value注解去获取到,而是通过@ConfigurationProperties(prefix = "对象名")来获取到
      >
      > 对象格式![1657787868926](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657787868926.png)
      >
      > ![1657787896133](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657787896133.png)
      >
      > 配置对象的读取
      >
      > ![1657788023451](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657788023451.png)
      >
      > 将配置对象属性注入给java中的一个具体对象,因为需要在SpringBoot启动的时候获取到Student类,所以需要加上五大注解中的一个.
      >
      > ![1657788107572](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657788107572.png)
      >
      > 在controller中利用@Autowired来获取到SpringBoot容器中的对象.
      >
      >  
      >
      > 集合格式:![1657788740303](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657788740303.png)
      >
      > 注:集合用**中括号**括起来
      >
      > 集合的读:与对象类似,只是@ConfigurationProperties后括号的内容变为集合名:@ConfigurationProperties("dbtypes")

   3. properties VS yml

      1. 语法不同,properties是"key=value",yml是"key: value"
      2. properties为早期并且默认的配置文件格式,但存在冗余数据的问题,yml较properties而言更简洁,易读性更好
      3. yml通用性更好,支持更多的编程语言
      4. yml支持更多的数据类型

   4. SpringBoot读取配置文件的方式

      1. 使用@Value
      2. 使用@ConfigurationProperties
      3. 使用Environment变量
      4. 使用@PropertySource(该注解只是描述读取的配置文件的路径,还需要搭配@Value来读取配置文件中的项)
      5. 使用原生方式

   5. SpringBoot日志文件

      1. 日志是程序的重要组成部分,日志可以帮助我们排除和定位问题

      2. 日志的主要功能

         > 记录用户登录日志,方便分析用户是正常登录还是恶意破解用户
         >
         > 记录系统的操作日志,方便数据恢复和定位操作
         >
         > 记录程序的执行时间,方便以后优化程序提供数据支持

      3. SpringBoot自定义日志的打印![1657802885032](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657802885032.png)

      4. 日志级别:将日志进行分类,帮助程序员去更精确的定位到对应的日志信息

         > 1. trace
         > 2. debug:调试日志
         > 3. info:普通信息日志
         > 4. warn:警告日志
         > 5. error:错误日志
         > 6. fatal:致命日志(不支持自定义打印,由系统输出)
         >
         > 日志级别从上到下依次升高,级别低的日志可以看到级别高的日志(日志级别越低,打印的日志信息越多)
         >
         >  
         >
         > 设置日志级别
         >
         > ![1657804750130](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657804750130.png)
         >
         > 局部范围的设置就是将"root"改成对应的"java"文件夹下某个具体的文件路径
         >
         > 全局范围和局部范围的日志设置均存在时,局部范围优先于全局范围
         >
         >
         >
         > 日志持久化(将日志永久地存储到磁盘的某个位置上)
         >
         > 1. 在配置文件中设置日志的保存路径
         >
         >    ![1657805226333](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657805226333.png)
         >
         > 2. 在配置文件中设置日志的名称(一般设置文件的全路径以及文件的名称)
         >
         >    ![1657805403749](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1657805403749.png)
         >
         >    日志默认情况下是以追加的方式进行的,且每个日志的默认最大大小为10MB,超过10MB会自动建立新的日志文件(名称为原日志文件名称后面+1,+2...)
         >
         >  
         >
         > 更简单的实现日志打印

      5. 
