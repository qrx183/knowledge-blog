# Spring事务

### 事务的必要性

事务的一致性是由日志来保证的.当事务进行到一半因为异常结束需要回滚时,系统会查看日志,当发现日志没有执行完时,会从出现问题的日志往回执行补偿机制.

### Spring中事务的实现

1. 编程式事务

   > 1. 开启事务
   > 2. 提交事务
   > 3. 回滚事务
   >
   > 利用DataSourceTransactionManager和TransactionDefinition实现事务.
   >
   > ```java
   > @RestController
   > @RequestMapping("/user")
   > public class UserController {
   > 
   >     @Autowired
   >     private UserService userService;
   > 
   >     @Autowired
   >     DataSourceTransactionManager transactionManager;
   > 
   >     @Autowired
   >     TransactionDefinition definition;
   > 
   >     @RequestMapping("/add")
   >     public int add(UserInfo userinfo){
   >         //开启事务
   >         TransactionStatus transactionStatus = transactionManager.getTransaction(definition);
   >         if(userinfo == null || !StringUtils.hasLength(userinfo.getUsername()) || !StringUtils.hasLength(userinfo.getPassword())){
   >             return 0;
   >         }
   >         int res = userService.add(userinfo);
   >         //回滚事务
   >     //    transactionManager.rollback(transactionStatus);
   >         //提交事务
   >         transactionManager.commit(transactionStatus);
   >         return res;
   >     }
   > }
   > 
   > ```

2. 声明式事务

   添加@Transactional注解:在方法之前,自动开始事务,方法结束之后,自动结束事务,当出现异常后回滚事务

   Transactional作用域:修饰方法时,只能修饰public方法;修饰类时,作用于该类内的所有方法

   Transactional 参数设置

   > 1. value/transactionManager:当配置了多个事务管理器时,可以使用该属性指定选择哪个事务管理器
   > 2. propagation:事务的传播行为,当存有嵌套事务时指定哪个事务生效
   > 3. isolation:事务的隔离级别
   > 4. timeout:如果时间超过timeout事务还没有完成,则自动回滚事务
   > 5. readOnly:指定事务是否为只读事务,默认为false
   > 6. rollbackFor/rollbackForClassName:用于指定可以触发事务回滚的异常类型,可以是多个.
   > 7. noRollBackFor/noRollBackForClassName:用于指定不触发事务回滚的异常类型,可以是多个.

   当Spring中事务的隔离级别与连接的数据库中的事务的隔离界别发生冲突时,以Spring为准

   **@Transactional注解是黑盒,遇到异常自动回滚事务,但当程序员在代码中处理异常后(try catch)后就不会触发事务回滚**

   解决方案:在try/catch中的catch中手动设置回滚事务:TransactionAspectSupport.currentTransactionStatus().setRollbackOnly().

### Spring事务传播机制

事务传播机制的定义:Spring事务传播机制定义了多个包含了事务的方法,相互调用时,事务是如何在这些方法间进行传递的.

为什么需要事务传播机制:为了保证稳定性

事务传播机制有哪些?

支持当前事务:

1. Propagation.REQUIRED:默认情况,如果当前存在事务,就加入到事务;如果没有事务,就创建一个事务.
2. Propagation.SUPPORTS:如果当前存在事务,就加入到事务;如果没有事务,就以非事务的形式运行
3. Propagation.MANDATORY:如果当前存在事务,就加入到事务;如果没有事务,就抛出异常

不支持当前事务:

1. Propagation.REQUIRES_NEW:创建一个新的事务,如果当前存在事务,则将事务挂起.无论外部方法是否开启事务,内部方法都会开启一个自己的新事务.**事务之间相互独立,互不干扰.**
2. Propagation.NOT_SUPPORTED:以非事务的方式运行.如果当前存在事务,则将当前事务挂起.
3. Propagation.NEVER:以非事务的方式运行.如果当前存在事务,则抛出异常.

嵌套事务:

1. Propagation.NESTED:如果当前存在事务,则创建一个事务作为当前事务的嵌套事务运行,如果没有事务就和Propagation.REQUIRED相同.