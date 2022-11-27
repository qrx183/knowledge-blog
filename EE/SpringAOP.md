# SpringAOP

## AOP

1. AOP:面向切片编程(Aspect Oriented Programing)

2. AOP组成

   1. 切面:由切点和通知组成.定义了AOP是针对哪个统一的功能,这个功能就叫做一个切面.比如用户登录验证,统一处理异常等等.
   2. 连接点:所有可能触发AOP的点都被称为连接点.比如一个项目中有100个方法,其中98个方法都需要进行用户登录的验证(AOP),这98个需要嵌入用户登录验证AOP的方法都是连接点
   3. 切点:提供一组规则来匹配相对应的连接点.切点是一组连接点的集合.
   4. 通知:切面的工作被称为通知,规定了AOP执行的时机和执行的方法
      1. 前置通知:在执行某个方法之前执行的通知:用户登录验证
      2. 后置通知:再执行某个方法之后执行的通知:记录日志
      3. 抛出异常后通知:在抛出异常后执行的通知
      4. 返回数据之后通知:在return之后执行的通知:记录代码的执行结束时间
      5. 环绕通知:讲方法包起来,在前后都可执行的通知

3. AOP实现

   1. 导入对应的Spring-boot-aop依赖![1658972389294](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658972389294.png)
   2. 设置切面类![1658972434036](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658972434036.png)
   3. 设置切点![1658972580285](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658972580285.png)
   4. 设置通知
      1. 前置通知![1658977092675](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658977092675.png)
      2. 后置通知![1658977101045](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658977101045.png)
      3. 返回数据后通知![1658977158703](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658977158703.png)
      4. 抛出异常后通知![1658977793759](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658977793759.png)
      5. 环绕通知![1658977956581](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658977956581.png)
      6. 使用环绕通知实现统计某个方法的执行时间![1658979575759](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658979575759.png)
      7. ![1658977948403](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658977948403.png)

4. SpringAOP的实现原理

   SpringAOP是基于动态代理实现的.这种动态代理能够形成一个拦截器,为后端的一类方法去拦截前端传递来的参数(比如用户登录校验),通过动态代理,错误的前端参数会不经过后端直接从动态代理这返回前端一个错误信息.而这种动态代理的实现有2种实现方式

   1. JDK Proxy(JDK 动态代理)
   2. CGLIB Proxy

   默认情况下Spring AOP都是用CGLIB Proxy实现动态代理.CGLIB Proxy是通过继承代理对象生成代理对象的子类来实现动态代理(子类拥有父类的所有方法).因此CGLIB Proxy不能动态代理最终类(被final修饰的类),在这种场景下使用JDK Proxy