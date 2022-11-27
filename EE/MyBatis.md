# MyBatis

1. 定义:MyBatis是一种持久性框架,它支持自定义SQL,存储过程(因为不能**调试**,所以 很少会用)和高级映射.MyBatis不需要JDBC和设置参数以及获取结果集的工作,可以通过简单的XML或注解来配置和映射原始类型.接口和Java POJO为数据库中的记录. MyBatis也是一个ORM(Object Related Mapping:对象关系映射)框架

2. MyBatis开发环境的配置

   1. 床架一个Spring项目,引入MyBatis的相关依赖![1658546064641](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658546064641.png)

      引入MyBatis的框架个数据库驱动

   2. 配置数据库连接字符串和MyBatis(保存的XML记录)

      1. 配置数据库连接字符串

         ![1658547519120](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658547519120.png)

      2. 配置MyBatis的XML的保存路径![1658547820367](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658547820367.png)

         因为各个环境下XML的路径是一致的,所以直接写到application.yml中

3. 使用MyBatis模式和语法操作数据库

   MyBatis模式包含两部分:接口(定义方法的声明);xml(实现接口中的方法),二者搭配可以形成一个数据库可以执行的SQL,并执行SQL将结果映射到程序的对象中

   1. 定义接口![1658549023817](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658549023817.png)

   2. 定义xml![1658549212301](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658549212301.png)

   3. 实现验证

      1. 利用Service将相关Bean对象引入Spring框架中

         ```java
         @Service
         public class UserService {
             @Resource
             private UserMapper userMapper;
         
             public UserInfo getUserById(Integer id){
                 return userMapper.getUserById(id);
             }
         }
         
         ```

      2. 通过Controller与前端页面交互,验证各个参数

         ```java
         @RestController
         @RequestMapping("/user")
         public class UserController {
             @Autowired
             private UserService userService;
         
             @RequestMapping("/getuserbyid")
             public UserInfo getUserById(Integer id){
                 if(id == null){
                     return null;
                 }
                 return userService.getUserById(id);
             }
         }
         
         ```

         为了简化验证,引出单元测试

   4. 增删改

      1. 改

         ![1658670192864](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658670192864.png)

         ![1658670182684](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658670182684.png)

         上述默认是会污染连接的数据库,要想不污染,只需要在测试方法前加一个注解@Transactional(本质是将事务rollback)	

      2. 删

         ![1658670155543](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658670155543.png)

         ![1658670166996](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658670166996.png)

      3. 增

         1. 

            ![1658670999940](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658670999940.png)

            ![1658670991918](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658670991918.png)

         2. ![1658671817859](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658671817859.png)

            ![1658671829781](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658671829781.png)

            ![1658671838154](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658671838154.png)

            ![1658671851282](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658671851282.png)

   5. #{} 和 ${}的区别

      > #是预处理查询,在得到具体参数之前已经加载完,然后只是将对应参数传进来
      >
      > $是即时查询,在得到具体的参数之后才进行加载,然后直接替换.
      >
      > #较$更安全,遇到的特殊情况更少
      >
      > 针对int参数 # 和 $都可以实现.而针对String参数 用$会报错,因为$是直接替换,替换的内容没有加单引号,所以找不到.
      >
      > 总结:
      >
      > 1. 定义不同:#{} 是预处理; ${} 是字符直接替换
      > 2. 使用不同:#{} 适用于所有类型的参数;${}适用于数值型的参数
      > 3. 安全性不同:#{} 性能高,并且没有安全性的问题; ${}存在SQL注入的安全问题
      >
      > ${}的适用场景:当输入的参数是SQL中的关键字(SQL命令),只能使用${},如果使用#{}.会认为传递的是一个普通的值,而非命令.**而当不得不使用${},就一定要提前在业务场景中对参数进行安全校验.**
      >
      > ${}的SQL注入问题
      >
      > ![1658796317253](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658796317253.png)
      >
      > ![1658796329915](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658796329915.png)
      >
      > ![1658796339748](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658796339748.png)
      >
      > ![1658796354634](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658796354634.png)
      >
      > 可以看到,返回了多个结果,发生了安全漏洞问题
      >
      > 当无法用#{}时,而用${}又无法穷举时,可以用concat函数搭配#{}使用:concat('%',#{username},'%'),concat函数是连接字符串函数

   6. resultMap的使用场景

      当返回的类对象的属性名和数据库中某个表中的属性名不一致时,无法做有效处理,为了处理这种名称不一致的映射,引出resultMap

      ```xml
      <!-- id是返回的对象名 -->
      <resultMap id = "BaseMap" type = "com.example.mybatis1.model.UserInfo">
          <!-- id是主键属性名称的映射,column是数据库中的属性名称,property是类对象的属性名称 -->
      	<id column = "id" property = "id"></id>
          <!-- result是普通属性的映射 -->
          <result column = "username" property = "name"></result>
      </resultMap>
      ```

      多表查询

      1. 一对一查询

         ![1658805119579](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658805119579.png)

         利用ResultMap可以实现多表查询,其中association是指的关联表的对象类,为了避免关联表中两个表中存在属性名称相同的两个不同属性的覆盖问题,要用columnPrefix为后一个表赋别名(**其实是在原来名称基础上加一个前缀**),在赋别名后,利用SQL进行查询时就不能用*,而需要分别为改了表名后的每个属性名分别赋别名(这个别名是columnPrefix标签中的值加上该属性原来的名,其他的别名是不行的),然后SQL查询也是多表查询,利用单表查询是不生效的

      2. 一对多查询

         ![1658806018608](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658806018608.png)

         相较于一对一查询,改变的只是将association变成了collection

   7. 动态SQL:是Mybatis的强大特性之一

      1. <if>标签:判断一个参数是否有值,如果没有则隐藏if语句内的SQL

         语法:<if test="username!=null"> username = #{username} </if>:含义是如果username不等于null则给username赋值,否则不执行if标签内的语句

      2. <trim>标签:用来忽略SQL语句前后多余的某个字符

         语法:<trim prefix = "(" suffix = ")" suffixOverrides=",">多个if标签</trim>

      3. <where>标签:where后的判断调教如果是非必须的则隐藏这一部分的SQLwhere判断语句,而且会自动忽略SQL语句**最前面的and字符串**(不能是最后面)

         语法:<where> 多个if标签 </where>

      4. <set>标签:动态的修改某个属性,也可以自动忽略SQL语句**最后面的","**

         语法:<set> 多个if标签</set>

      5. <foreach>标签:对集合进行循环

         语法:以批量删除为例,在SQL中的in()用<foreach>标签代替.

         <foreach collections = "集合名" open = "前缀符号" close = "后缀符号" item = "集合中每个元素名" separator = "分隔符">#{删除的属性名}</foreach>

   8. 

# SpringBoot单元测试

1. 单元测试:是指对软件中的最小可测单元(方法)进行检查和验证的过程叫单元测试

2. 单元测试的优势

   > 1. 如果中途改变了代码,在项目打包的时候会发现错误,因为打包的时候会自动启动单元测试,单元测试错误就会被发现
   > 2. 可以快速的知道某个方法的正确性
   > 3. 使用单元测试可以在验证功能的基础上不会污染连接的数据库(而通过Service,Controller测试一些数据更新的操作时会同时更新数据库中的数据)

3. 单元测试的实现

   > 1. 确认项目中内置了测试框架(高版本的SpringBoot会内置测试框架)![1658635789278](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658635789278.png)
   >
   > 2. 生成单元测试的类
   >
   >    在对应的测试的方法中生成测试![1658635939248](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658635939248.png)
   >
   >    ![1658636219394](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658636219394.png)
   >
   >
   >
   >    ![1658636253660](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658636253660.png)
   >
   >    生成了对应方法的测试方法
   >
   > 3. 配置单元测试的类并添加@SpringBootTest注解,并添加Test方法代码
   >
   >    ![1658636379405](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658636379405.png)
   >
   >    ​	直接运行无论结果是否正确都显示的是测试成功,因此我们需要"断言测试".
   >
   >    ​	Assertions.assertNull/assertNotNull/assertTrue/assertFalse/assertSame/assertNotSame/
   >
   >    ​	
   >
   >    ```java
   >    @SpringBootTest
   >    class UserControllerTest {
   >    
   >        @Resource
   >        private UserMapper userMapper;
   >        @Test
   >        void getUserById() {
   >            UserInfo userInfo = userMapper.getUserById(1);
   >            Assertions.assertNull(userInfo);
   >        }
   >    }
   >    ```
   >
   >    ![1658666271510](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1658666271510.png)