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
   2. 事务的传播级别

8. Spring MVC