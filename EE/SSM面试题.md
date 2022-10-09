# SSM面试题(国庆)

1. Spring框架的好处

   1. 轻量:Spring的大小大概只有2MB
   2. 实现了松耦合:通过控制反转的方式实现了松耦合
   3. 面向切片编程(AOP),将应用逻辑和系统服务分开
   4. 通过容器来管理对象的生命周期
   5. 事务管理:Spring提供给一个持续的事务管理接口
   6. 异常处理:Spring提供方便的API将具体技术相关的异常转为为一致的运行异常处理

2. ApplicationContext和BeanFactory的区别

   1. ApplicationContext接口实现了BeanFactory接口.BeanFactory接口只实现了基本的访问Bean的方法,而ApplicationContext在BeanFactory的基础上对功能进行了拓展:通过实现MessageSource接口获得国际化的功能:通过ApplicationEvent和ApplicationListener两个接口实现事件的发布和传播的功能:即当通过ApplicationContext发布一个事件时,所以实现了ApplicationListener的Bean对象都会接收到这个事件;通过实现ResourceLoader接口来访问多个Resource
   2. BeanFactory接口通过懒加载的方式注入Bean,而ApplicationContext接口在容器启动时就创建好所有的Bean,使用的是饿汉式

3. Spring中Bean的生命周期

   1. Spring在启动时寻找并加载被管理的Bean对象,**执行Bean的实例化**
   2. Bean实例化后对Bean中的属性执行依赖注入
   3. 如果Bean实现了BeanNameAware接口,Spring就将Bean的id传给setBeanName方法
   4. 如果Bean实现了BeanFactoryAware接口,Spring就调用setBeanFactory方法,将BeanFactory容器的实例传入
   5. 如果Bean实现了ApplicationContextAware接口,Spring就调用Bean的setBeanFactory方法,将Bean所在应用的上下文传入
   6. 如果Bean实现了BeanPostProcessor接口,就调用postProcessBeforeInitialization方法
   7. 如果Bean实现了InitializingBean接口,就调用afterPropertiesSet方法,之后如果Bean使用init-method声明了初始化方法,就调用该方法
   8. 如果Bean实现了BeanPostProcesson接口,就调用postProcessAfterInitialization方法
   9. Bean初始化完成,可以被使用知道应用上下文被销毁
   10. 如果Bean实现了DisposableBean接口,Spring就调用destroy方法,之后如果Bean使用了destroy-method声明,就调用该销毁方法

   ```Java
   // Bean生命周期的示例
   
   // 1.Bean对象User类
   public class User implements BeanNameAware, BeanFactoryAware, ApplicationContextAware,
          InitializingBean, DisposableBean {
       private String username;
       private String password;
       public User(){
           System.out.println("User 正在初始化 ...");
       }
   
       @Override
       public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
           System.out.println("执行 setBeanFactory方法");
       }
   
       @Override
       public void setBeanName(String name) {
           System.out.println("执行 setBeanName方法");
       }
   
       @Override
       public void destroy() throws Exception {
           System.out.println("User 正在被销毁");
       }
   
       @Override
       public void afterPropertiesSet() throws Exception {
           System.out.println("User 正在通过afterPropertiesSet方法 执行 初始化");
       }
   
       @Override
       public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
           System.out.println("执行 setApplicationContext方法");
       }
   
   }
   
   // 2.自定义类MyPostProcessor实现BeanPostProcessor类中的初始化前后的方法
   public class MyBeanPostProcessor implements BeanPostProcessor {
       @Override
       public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
           System.out.println("执行初始化之前的方法:postProcessBeforeInitialization");
           return bean;
       }
   
       @Override
       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
           System.out.println("执行初始化之后的方法:postProcessAfterInitialization");
           return bean;
       }
   }
   
   // 3.将两个类添加到扫描路径中:spring-config.xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
   
       <bean id="user" class="com.beans.User"/>
       <bean class="com.beans.MyBeanPostProcessor"/>
   </beans>
   
   // 4.编写测试类
   public class Test {
       public static void main(String[] args) {
           //获取到Spring上下文对象
           ApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");
           User user = (User)context.getBean("user");
   
           ((ClassPathXmlApplicationContext) context).destroy();
   	}
   }
   
   // 5.运行结果
   /*
   User 正在初始化 ...
   执行 setBeanName方法
   执行 setBeanFactory方法
   执行 setApplicationContext方法
   执行初始化之前的方法:postProcessBeforeInitialization
   User 正在通过afterPropertiesSet方法 执行 初始化
   执行初始化之后的方法:postProcessAfterInitialization
   User 正在被销毁
   */
   ```

4. Spring中Bean的作用域

   Spring中Bean的作用域有5种,默认为单例(Singleton)

   > 1. singleton:在Spring IOC中只存在一个Bean实例
   > 2. prototype:每次调用Bean时获取到的都是新生成的一个Bean实例
   > 3. request:每次Http请求时会获得一个Bean实例
   > 4. session:同一个Session公用一个Bean实例,不同的Session使用不同的Bean实例
   > 5. application:限定Bean在Servlet Context中使用.
   >
   > 注:后三者尽在Spring WebApplicationContext环境中使用

5. Spring IOC

   IoC,Inversion of Control.控制反转.即将创建对象和管理对象之间的依赖关系这一操作由程序猿转交给Spring容器,由Spring容器负责.**即控制对象的权利由对象引用本身转交给Spring**.IoC是一种设计思想,而实现这种思想的是DI,依赖注入.DI实现的原理是反射,在运行时动态获取到对象的所有方法和属性

6. Spring AOP

   AOP,Aspect oriented Program.面向切片编程,将被封装的对象切开,从中取出多个类共有的公共行为封装到一个可重用模块.

   AOP就是将与业务无关,却被业务模块共同调用的逻辑单独封装起来,减少项目的重复代码和代码的耦合度

   AOP将项目中的代码分为2个部分:核心关注点和横切关注点.业务处理的主要流程即为核心关注点,与业务关系不大的即为横切关注点.横切关注点往往穿插于核心关注点的多处,并在各处基本相似,如权限认证,日志,事务管理.AOP的作用就是将这些横切关注点从核心关注点中分离开来

   Spring AOP的实现原理

   ![1664787953360](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1664787953360.png)

   调用者在调用某段代码时,会被代理截胡,代理首先根据Advice的种类执行特定的Advice,之后再根据Advice执行的结果去决定是否执行目标方法,如果执行则由代理执行目标方法并将目标方法返回给调用者

   Spring AOP的两种实现方式

   1. 基于接口的动态代理

      Java的动态代理机制涉及到InvocationHandler接口和Proxy类

      每一个动态代理类必须要实现InvocationHandler接口并重写invoke方法

      invoke有3个参数,分别是proxy:所代理的那个真实对象,method:所调用的真是对象的method方法对象,args:方法对象对应的形参个数

      我们需要使用Proxy类中的newProxyInstance方法来创建一个代理对象的实例

      newProxyInstance方法有3个参数,分别是loader:代理对象对应的类加载器对象,interfaces:需要提供给代理对象的一组接口,h:一个InvocationHandler对象

      实现示例:

      ```java
      public interface Subject
      {
          public void rent();
          
          public void hello(String str);
      }
      public class RealSubject implements Subject
      {
          @Override
          public void rent()
          {
              System.out.println("I want to rent my house");
          }
          
          @Override
          public void hello(String str)
          {
              System.out.println("hello: " + str);
          }
      }
      
      
      public class DynamicProxy implements InvocationHandler
      {
          //　这个就是我们要代理的真实对象
          private Object subject;
          
          //    构造方法，给我们要代理的真实对象赋初值
          public DynamicProxy(Object subject)
          {
              this.subject = subject;
          }
          
          @Override
          public Object invoke(Object object, Method method, Object[] args)
                  throws Throwable
          {
              //　　在代理真实对象前我们可以添加一些自己的操作
              System.out.println("before rent house");
              
              System.out.println("Method:" + method);
              
              //    当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
              method.invoke(subject, args);
              
              //　　在代理真实对象后我们也可以添加一些自己的操作
              System.out.println("after rent house");
              
              return null;
          }
      
      }
      public class Client
      {
          public static void main(String[] args)
          {
              //    我们要代理的真实对象
              Subject realSubject = new RealSubject();
      
              //    我们要代理哪个真实对象，就将该对象传进去，最后是通过该真实对象来调用其方法的
              InvocationHandler handler = new DynamicProxy(realSubject);
      
              /*
               * 通过Proxy的newProxyInstance方法来创建我们的代理对象，我们来看看其三个参数
               * 第一个参数 handler.getClass().getClassLoader() ，我们这里使用handler这个类的ClassLoader对象来加载我们的代理对象
               * 第二个参数realSubject.getClass().getInterfaces()，我们这里为代理对象提供的接口是真实对象所实行的接口，表示我要代理的是该真实对象，这样我就能调用这组接口中的方法了
               * 第三个参数handler， 我们这里将这个代理对象关联到了上方的 InvocationHandler 这个对象上
               */
              Subject subject = (Subject)Proxy.newProxyInstance(handler.getClass().getClassLoader(), realSubject
                      .getClass().getInterfaces(), handler);
              
              System.out.println(subject.getClass().getName());
              subject.rent();
              subject.hello("world");
          }
      }
      
      // 输出
      /*
      $Proxy0
      
      before rent house
      Method:public abstract void com.xiaoluo.dynamicproxy.Subject.rent()
      I want to rent my house
      after rent house
      
      before rent house
      Method:public abstract void com.xiaoluo.dynamicproxy.Subject.hello(java.lang.String)
      hello: world
      after rent house
      */
      ```

   2. 基于子类化的CGLIB代理

      CGlib动态代理是通过继承业务类,生成的动态代理类是业务类的子类,通过重写业务方法进行代理.

      在运行时动态的生成一个被代理类的子类,子类重写了被代理类中所有非final的方法,通过方法拦截将横切的部分逻辑植入到重写的代理类的方法中

   Spring AOP大多数情况下使用的都是CGLIB代理,但在多例或者被代理类中含有final方法时要考虑jdk动态代理

7. 事务

   1. 事务的实现方式

      事务的实现方式有2种:编程式事务和提交式事务

      1. 编程式事务:通过调用beginTransaction(),commit(),rollback()方法管理事务

         ```java
         @RestController
         @RequestMapping("/user")
         public class UserController {
         
             @Autowired
             private UserService userService;
         
             @Autowired
             DataSourceTransactionManager transactionManager;
         
             @Autowired
             TransactionDefinition definition;
         
             @RequestMapping("/add")
             public int add(UserInfo userinfo){
                 //开启事务
                 TransactionStatus transactionStatus = transactionManager.getTransaction(definition);
                 if(userinfo == null || !StringUtils.hasLength(userinfo.getUsername()) || !StringUtils.hasLength(userinfo.getPassword())){
                     return 0;
                 }
                 int res = userService.add(userinfo);
                 //回滚事务
             //    transactionManager.rollback(transactionStatus);
                 //提交事务
                 transactionManager.commit(transactionStatus);
                 return res;
             }
         }
         ```

      2. 提交式事务

         通过@Transactional注解来管理事务.

         Transactional注解的参数说明

         > 1. value/transactionManager:当配置了多个事务管理器时,可以使用该属性指定选择哪个事务管理器
         > 2. propagation:事务的传播行为,当存有嵌套事务时指定哪个事务生效
         > 3. isolation:事务的隔离级别
         > 4. timeout:如果时间超过timeout事务还没有完成,则自动回滚事务
         > 5. readOnly:指定事务是否为只读事务,默认为false
         > 6. rollbackFor/rollbackForClassName:用于指定可以触发事务回滚的异常类型,可以是多个.
         > 7. noRollBackFor/noRollBackForClassName:用于指定不触发事务回滚的异常类型,可以是多个.

   2. 事务的传播级别

      事务的传播机制就是确定当多个设置了方法的事务在相互调用时,事务在方法间如何传递的一种机制

      事务的传播机制一共有7种级别

      1. Propagation.REQUIRED:默认的Spring事务传播级别.该级别的特点是如果当前上下文中存在事务,就将新的方法加入到该事务中执行;如果没有则创建新事务执行.

      2. Propagation.SUPPORTS:特点是如果当前上下文中存在事务,就将新的方法加入到该事务中执行;如果没有则按照非事务的方式执行

      3. Propagation.MANDATORY:特点是要求当前上下文中必须存在事务,否则抛出异常

      4. Propagation.REQUIRED_NEW:每次执行都会创建一个新的事务,如果当前上下文中已经存在事务,就将当前事务挂起,等新事务执行结束后再执行已经存在的事务

      5. Propagation.NOT_SUPORTED:特点是如果当下上下文中存在事务,就将该事务挂起,待逻辑执行结束后再恢复该事务

      6. Propagation.NEVER:特点是如果当前上下文中存在事务就抛出异常

      7. Propagation.NESTED:特点是如果当前上下文中存在事务就嵌套事务执行,否则创建新的事务

         嵌套事务指的是新创建的事务作为子事务嵌套在父事务中执行.在执行过程中父事务会在子事务的开始点处标记回滚点save point.当子事务回滚,父事务会回到回滚点,回滚点之前的操作不会受影响,然后继续执行回滚点之后的操作;父事务回滚会造成子事务也回滚;子事务在父事务之前提交,但在父事务执行结束前,子事务不会发生提交

8. Spring MVC

   1. Spring MVC是一个基于MVC架构的用来简化web应用程序开发的应用开发框架,是Spring中的一个模块.MVC即为Model View Controller,通过将Web分离成MVC三部分从而简化了Web的开发

   2. Spring MVC的执行流程

      首先DispatcherServlet接收前端用户传递来的请求,DispatcherServlet调用HandlerMapping处理器映射器;HandlerMapping根据url获取到该Handler配置的所有对象(包括Handler对象以及Handler对象对应的拦截器),并将对象以处理器执行链的方式返回给DispatcherServlet;DispatcherServlet通过HandlerAdapter(处理器适配器)来执行处理器的请求并返回ModelAndView,DispatcherServlet将ModelAndView传给ViewResolver视图解析器;ViewResolver解析后将结果返回给DispatcherServlet,DispatcherServlet对View进行渲染视图.![1664884062411](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1664884062411.png)

   3. Spring  MVC的加载流程

      1. Servlet加载(执行Servlet的init()方法)
      2. 加载配置文件
      3. 从ServletContext中拿到Spring初始化Spring MVC相关对象
      4. 放入ServletContext中

9. Spring MVC与Servlet的区别和联系

   Spring MVC是对Servlet的一个封装,在Spring MVC中有一个核心类DispatcherServlet就是实现的Servlet接口.

10. 常见的ORM框架有哪些

    ORM,Object Relational Mapping.对象关系映射,是一种为了解决面向对象和关系数据库之间互不匹配的现象的技术

    常见的ORM框架有MyBatis,Hibernate.

    > Hibernate:Hibernate框架是一个全表映射的框架,开发者只需要定义持久化对象和关系数据库之间的映射关系即可完成持久层操作
    >
    > Hibernate框架会根据编制的存储逻辑自动生成SQL语句,并调用JDBC接口执行.
    >
    > Hibernate的缺点
    >
    > 1. 多表查询时,SQL查询性能低
    > 2. 不支持存储过程
    > 3. 不能通过优化SQL的方式提高性能
    >
    > MyBatis:MyBatis框架是一个半自动映射的框架.相对于Hibernate而言,MyBatis需要手动设置POJO(简单的Java对象),SQL和映射关系.

11. IoC/DI的理解

    IoC即为Spring的核心思想之一.IoC,,Inversion of Control.控制反转.即将管理对象的生命周期以及对象之间的依赖关系的权利由对象引用本身,也可以说是开发人员转交给Spring容器.

    通过IoC可以实现对象的创建和使用代码的解耦,即开发人员在使用对象前只需要从Spring容器中将对象取出,然后直接使用即可.IoC的实现同时也使得开发流程变得更加简单.

    DI,Dependency Injection,依赖注入.是IoC思想的实现方式.在Java中注入的方式分为三种,分别是属性注入,构造方法注入和setter方法注入.

    1. 属性注入:写法简单,但是通用性差,只能运行在IoC容器下
    2. setter注入:早起Spring推荐的写法,Setter注入通用性要差于构造方法注入(其他高级语言大多不再用java的这种get和set方法)
    3. 构造方法注入:现在官方推荐的写法,通用性最好

    在进行依赖注入时,可以使用@Autowired注解或@Resources注解.二者的区别如下

    1. 出身不同:@Resource来自于JDK,@Autowired来自于Spring
    2. 用法不同:@Resource支持属性注入和Setter注入,@Autowired支持属性注入,Setter注入和构造方法注入
    3. 支持的参数不同:@Resource支持更多的参数设置(name,type..),而@Autowired只支持required参数的设置

12. Spring中单例Bean的线程安全问题

    在Spring中默认的Bean的作用域是单例,也就意味着在多线程的环境下如果对某个变量进行写的操作可能会造成线程不安全.

    针对线程不安全问题,我们可以通过设置@Scope来讲Bean的作用域从单例改为原型,也可以通过将共享变量放入ThreadLocal中,对ThreadLocal中的变量副本进行操作

    ```java
    // ThreadLocal 实例
    	static int a = 10;
        static ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>(){
            @Override
            protected Integer initialValue() {
                return 0;
            }
        };
        public static void main(String[] args) {
            for(int i = 0; i < 5; i++) {
                Thread t = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Integer n = threadLocal.get();
                        n += 1;
                        threadLocal.set(n);
                        System.out.println(threadLocal.get());
                    }
                });
                t.start();
            }
        }
    
    /*
    运行结果:
    1
    1
    1
    1
    1
    */
    // 通过运行结果我们可以看到ThreadLocal中变量副本在每个线程内是独立存在的
    ```

13. FactoryBean和BeanFactory

    BeanFactory是Spring中的顶级接口,负责管理Spring中的所有Bean对象.

    FactoryBean是一个获取Bean对象的接口.对象类通过实现FactoryBean接口并重写接口下的getObject方法来返回对象.  例如当我们想要获取一个第三方库的某个对象时,一般情况下我们无法通过new得到对应的对象,此时如果该对象重写了FactoryBean接口,那么我们就可以通过调用getObject方法来获取到对象实例.

    ```
    
    ```

14. Spring三级缓存的理解

    在Spring中,Bean对象是保存在缓存中的.一个Bean的生命周期概括为实例化Bean,属性注入,初始化Bean,使用Bean,销毁Bean.

    在Spring中一共有3级缓存,每个缓存的本质其实是一个哈希表,key为Bean对象的名字,value为对应的Bean对象.在1级缓存中存储的是已经初始化好的Bean对象,2级缓存中存储的是属性未完全注入的Bean对象,如果是经过AOP代理,则存储的是Bean的代理对象,3级缓存中存储的是Bean的工厂类(ObjectFactory)

    每个Bean对象在创建的过程中只能存在于某一个缓存中,不能同时存在于两个及以上的缓存中.而创建Bean对象的顺序是从3级-->1级,而查询Bean对象则是通过1级--->3级

    Spring中三级缓存很好的解决了循环依赖的问题.循环依赖即存在两个对象A和B,A中含有属性B,B中含有属性A.导致在初始化A,B时造成一种僵局.

    Spring三级缓存解决循环依赖的过程

    首先实例化对象A,将A放入到3级缓存中,然后执行A的属性注入,从上到下查询缓存发现没有B对象,于是先把对象A放入到2级缓存中,然后再实例化对象B,将B放入到3级缓存中,然后执行B的属性注入,从上到下在2级缓存中查询到对象A,对象B的属性注入完成,执行初始化然后放到1级缓存中,此时轮到对象A的属性注入,在1级缓存中找到对象B,对象A的属性注入完成,执行初始化然后放到1级缓存中.

15. Spring中事务的隔离级别

    1. ISOLATION_DEFAULT:使用数据库的默认隔离级别.MySQL中默认的隔离级别是可重复读
    2. ISOLATION_READ_UNCOMMITTED:读未提交,允许读取尚未提交的数据
    3. ISOLATION_READ_COMMITTED:读已提交,允许读取已经提交的数据
    4. ISOLATION_REPEATABLE_READ:可重复读,保证多次读取同一数据的结果是相同的
    5. ISOLATION_SERIALIZABLE:串行化,相当于是单线程下执行事务,一个线程只有等待另一个线程事务结束后才能对数据库中的数据进行操作

16. MyBatis中#和$的区别

    1. #{}注入本质上是预处理,jdbc占位符的替换,当传入的是一个字符串时,会替代成加引号的字符串;${}注入本质上是SQL字符串的拼接,当传的是一个字符串时,结果就是在字符串的字面量值
    2. #{}适用于所有类型的参数,${}适用于数值型的参数
    3. #{}的安全性更好,${}存在SQL注入的安全问题,但如果输入的是SQL中的关键字,则只能使用${}

17. MyBatis如何实现一对一,一对多关联

    一对一关联:在xml配置文件中使用association标签,在association标签中存在三个子标签分别是property:标记绑定的实体类的类名,resultMap:关联结果集的路径,columnPrefix:防止两个表同名字段的覆盖

    ```xml
     <resultMap id="BaseMap" type="com.example.mybatis1.model.UserInfo">
            <id column="id" property="id"></id>
            <result column="username" property="username"></result>
            <result column="password" property="password"></result>
            <result column="photo" property="photo"></result>
            <result column="createtime" property="createtime"></result>
            <result column="updatetime" property="updatetime"></result>
            <result column="state" property="state"></result>
            <collection property="aList"
                        resultMap="com.example.mybatis1.mapper.ArticleMapper.BaseMap"
                        columnPrefix="a_"
            ></collection>
        </resultMap>
    ```

    一对多关联:在xml配置文件中使用collection标签,在collection标签中的三个字标签和association标签中的一样

    ```xml
    <resultMap id="BaseMap" type="com.example.mybatis1.model.ArticleInfo">
            <id column="id" property="id"></id>
            <result column="title" property="title"></result>
            <result column="content" property="content"></result>
            <result column="createtime" property="createtime"></result>
            <result column="updatetime" property="updatetime"></result>
            <result column="uid" property="uid"></result>
            <result column="rcount" property="rcount"></result>
            <result column="state" property="state"></result>
            <association property="userInfo"
                         resultMap="com.example.mybatis1.mapper.UserMapper.BaseMap"
                         columnPrefix="u_"></association>
        </resultMap>
    ```

18. SpringBoot的自动配置原理