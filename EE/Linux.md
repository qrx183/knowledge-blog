## Linux

### 命令

1. ls:查询当前目录的文件和子目录(list)
   1. ls -l <==> ll :以一种更详细的方式展示当前目录的文件和子目录
2. pwd:查询当前路径
3. cd:切换目录,在cd之后加一个路径(可以是绝对路径也可以是相对路径)
4. Ctrl C表示取消当前操作
5. Ctrl Insert为复制,Shift Insert为粘贴
6. Ctrl L清屏
7. touch: 创建一个空文件
8. cat:显示文件内容
9. echo:打印内容到控制台,也可以用于写文件
10. mkdir : 创建目录
11. yum:相当于"应用商店",称为"包管理器".  yum install tree --- 安装tree命令,tree可以直观查看目录结构
12. rm:删除.既可以删除文件,也可以删除目录. 后面跟一个文件名或目录名.不能直接删除目录.要用rm -r来删除目录,表示递归删除.在删除目录的每个文件时,都会提示是否删除.想直接删除可以用 rm -rf来删除.**rm命令要慎用,在Linux中,文件删除后很难再恢复文件**   **rm -rf/*(rm -rf/)这个命令就是把整个系统的文件删除掉(一键还原)** .   这个命令这辈子少碰几次额...呵呵.
13. cp:复制命令 cp  待拷贝文件名  拷贝的位置,如果想要复制文件夹,需要加一个r,即cp -r
14. mv:移动命令 mv 待移动文件名 移动到的位置,除了移动以外,mv还有重命名的功能
15. man:查看其他命令的帮助手册  man  某个具体命令   退出man手册按'q'
16. less:读取文件内容(内置了翻页功能)  less 文件名,用less读大文件可以秒开(懒加载),与man类似,按'q'退出less
17. vim:技能读取文件内容,也能编写文件
    1. 创建或打开文件:vim 文件名(无则创建,有则打开)
    2. 编辑文件内容:首先按i从普通模式转为insert模式,然后和notepad一样对文本进行编译即可
    3. 保存退出:首先按esc从insert模式退出到普通模式,然后输入":wq"(也可以用:x)保存退出,不保存退出":q！"
18. grep:对字符串进行过滤 grep "字符串,筛选出含有该"字符串"的输出
19. |:将|前面命令的输出作为|后面命令的输入
20. su:从普通用户切换到管理员(root)
21. unzip 压缩文件:对压缩文件进行压缩

### Linux权限

![1656231369350](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1656231369350.png)

r:可读,w:可写,x:可执行,-:不能执行相关操作

上述截图中对应三种角色:例如rwx r-x r-x,其中第一个是文件拥有者的权限:可读可写可执行,第二个是同组用户的权限:可读不可写可执行 第三个是其他用户:可读不可写可执行

### Linux上部署Java Web程序

1. 通过yum list | grep jdk找到如下版本的jdk

![1656233230120](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1656233230120.png)

2. 通过yum instatll (该版本的jdk名)安装jdk

3. 在Tomcat官网下载Tomcat8.5.zip包

4. 将.zip包拖拽到xshell中(这一过程可能需要安装lszrz命令 yum install lszrz)

5. unzip Tomcat的zip包(可能需要安装unzip yum install unzip)

6. 切换到Tomcat的bin目录下

7. 利用chmod +x .*sh使所有.sh文件拥有可运行权限(在Linux中运行Tomcat的是 startup.sh)

8. sh startup.sh:运行Tomcat

9. 如何验证Tomcat正常运行

   1. 通过ps aux|grep tomcat命令去查看Tomcat进程的运行情况

      ![1656236100410](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1656236100410.png)

   2. 查看Tomcat的端口(8080)是否被某个进程绑定:netstat -anp | grep 8080

      ![1656236113026](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1656236113026.png)

   3. 看能否访问Tomcat的访问页面:在浏览器输入服务器的外网IP加对应端口号

      第一次访问会失效的原因是没有配置防火墙(防火墙在腾讯云服务器官网的管理页面中的防火墙中设置)

      ![1656236850721](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1656236850721.png)

10. 安装MySQL(Mariadb)并作一些参数修改:参考https://zhuanlan.zhihu.com/p/49046496.然后将本地数据库中对应的表和相应的数据写到Mariadb中.在java程序的DBUtil中将密码改为""(云服务器上的Mariadb没有设置登录密码).然后利用maven重新打包,打的包要达成war包(具体的格式在pom.xml中修改:<packing>war</packing>,<build><finalName>包名</finalName></build>).
11. 然后在Linux上将路径修改到新安装的Tomcat文件下的webapp路径,将war包拖到文件夹里.
12. 在浏览器上输入对应的外网IP以及相应的路径.
13. 显示异常的解决方法
    1. 抓包去看请求和响应报文中的报错信息
    2. 查看Tomcat的日志:在Tomcat文件夹下的logs文件夹中(按日期前后来看)![1656468190110](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1656468190110.png)

