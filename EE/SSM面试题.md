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
4. Spring中Bean的作用域
5. Spring IOC
6. Spring AOP
7. 事务
8. Spring MVC

