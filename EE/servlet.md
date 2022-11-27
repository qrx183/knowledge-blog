## Servlet

Tomcat Servlet和客户端.服务器的关系.

![1653361494705](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1653361494705.png)

其中HTTP Server就相当于Tomcat,接收浏览器(用户)发来的请求,并生成响应,这个响应的生成则是在Servlet接口中进行实现,同时Servlet接口还承接了HTTP服务器和数据库.而HTTP服务器和浏览器之间请求和响应的传输仍旧按照TCP/IP五层模型进行网络通信的.



Servlet中对应的获取到请求中JSON格式的信息

```java
ObjectMapper objectMapper = new ObjectMapper();

User user = objcetMapper.readValue(req.getInputStream(),User.class);
```





开发一个网站的基本步骤

1. 约定好前后端交互的接口(请求是啥,响应是啥,统一请求响应的数据格式)
2. 开发服务器代码
   1. 先编写servlet能够处理前端发来的请求
   2. 编写数据库代码,来存储/获取关键数据
3. 开发客户端代码
   1. 基于ajax能够构造请求以及解析响应(回调)
   2. 能够响应用户的操作

上述步骤又被称为MVC,其中M为Model(操作数据库存取数据的逻辑),V(View,给用户展示的界面(前端)),C(Controller 控制器,处理请求之后的关键逻辑.



Session(会话)

Servlet提供的Session的API:HttpSession.

```
getSession(true/false);
//true:会话不存在,则创建;会话存在,则获取
//false:会话不存在,则返回null;会话存在,则获取.
```





获取Post请求中的body部分的方法

1. body中的内容的格式为json

   ```java
   //服务器
   //首先需要引入对应版本的Jason
   private ObjectMapper objectMapper = new ObjectMapper();
       @Override
       protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
           resp.setContentType("application/json;charset=utf-8");
          	//将json格式的body内容转为java中的类
           Body b = objectMapper.readValue(req.getInputStream(),Body.class);
           resp.getWriter().write("b中的内容");
       }
   class Body{
       //body中不同内容的key
   }
   ```

   ```html
   <!--客户端-->
   <!--首先需要引入jQuery的cdn-->
   <script>jQuery cdn</script>
   <script>
       $.ajax({
           type:'post',
           url:'服务器中对应的WebServlet',
           contentType:'application/json;charset=utf-8',
           data:JSON.stringify(具体的json格式的数据),
           success : function(body){
               //可以将body中的内容显示到相应的页面上
               document.write(body);
           }
       })
   </script>
   
   ```

2. body中的内容的格式为原生的

    

   ```java
   private ObjectMapper objectMapper = new ObjectMapper();
       @Override
       protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
           //利用数据流读取post请求中的body部分的内容
           resp.setContentType("application/x-www-form-urlencoded;charset=utf-8");
           StringBuilder data = new StringBuilder();
           String line;
           BufferedReader reader;
           reader = req.getReader();
           while((line = reader.readLine()) != null){
               data.append(line);
           }
           resp.getWriter().write(new String(data));
       }
   ```

   ```html
   <!--客户端-->
   <!--首先引入jQUery-->
   
   <script>jQuery cdn</script>
   <script>
       $.ajax({
           type:'post',
           url:'servlet中对应的WebServlet',
           contentType:'application/x-www-form-urlencoded;charset=utf-8',
           data:'xx=xx & yy=yy'(form表单的格式),
           success:function(body){
               document.write(body);
           }
       })
   </script>
   ```
