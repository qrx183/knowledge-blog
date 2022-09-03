# Java基础面试题

## Java的8种基本类型 装箱 拆箱

1. Java的8种基本类型:byte,char,short,int,long,float,double,boolean,占用字节数的大小为1,1,2,4,8,4,8,1

2. ```java
   short s1 = 1;
   s1 = s1 + 1;
   s1 += 1;
   
   //上述第一种会发生编译错误,原因是s1+1是整形运算,会将结果转为int,而将int赋给short会发生类型转换错误
   //+=是java的一种运算符,因此可以正确编译
   ```

3. Integer和int的区别

   1. int是Java中的一种基本数据类型,Integer是一种包装类
   2. int的默认值为0,而Integer的默认值为null,Integer可以区分未赋值和赋值为0.
   3. Integer是对对象的引用,int是直接存储数据值

   Integer和int的比较

   > 1. 两个new出来的Integer比较
   >
   >    ```java
   >    Integer a = new Integer(100);
   >    Integer b = new Integer(100);
   >    // a == b false
   >    // 原因是new出来的是两个地址,地址是不相等的
   >    ```
   >
   > 2. Integer变量和int变量比较
   >
   >    ```java
   >    Integer a = new Integer(100);
   >    int b = 100;
   >    // a == b true
   >    // 当Integer和int比较时,会将Integer对象自动拆箱,比较的实际上是值
   >    ```
   >
   > 3. 两个非new得到的Integer变量比较
   >
   >    ```java
   >    Integer a = 100;
   >    Integer b = 100;
   >    // a == b true
   >    Integer c = 200;
   >    Integer d = 200;
   >    // c == d false
   >    //原因是在自动装箱时,会调用Integer.valueOf():在-128~127之间会直接返回常量池中的值,否则会new一个新的对象
   >    
   >    
   >    
   >     public static Integer valueOf(int i) {
   >         if (i >= IntegerCache.low && i <= IntegerCache.high)
   >             return IntegerCache.cache[i + (-IntegerCache.low)];
   >         return new Integer(i);
   >     }
   >    ```
   >
   > 4. 一个new和一个非new的Integer变量比较
   >
   >    ```java
   >    Integer a = 100;
   >    Integer b = new(100);
   >    // a == b false
   >    //原因是a调用的是valueOf(),-128~127的会指向常量池,否则会new一个新的对象,无论是哪个,和b的new的新对象的地址都不一样
   >    ```
   >

4. 装箱和拆箱是基本数据类型和包装类之间的相互转化.装箱是基本数据类型转化为包装类,拆箱就是反过来.

## Java的面向对象的三种特性

Java的三个特性:封装 继承 多态

**封装**:将数据和访问数据的方法绑定起来,外部对数据的访问只能通过指定的接口.使用者在使用某个类时只能通过指定的方法去调用类中的数据.封装提高了程序的安全性,隐藏了程序的实现细节

**继承**:从已有类中得到继承信息并创建出新的类.提供继承信息的类是父类,超类,得到继承信息的类是子类,派生类

**多态**顾名思义就是一种东西的多个形态,这个东西可以是一个对象,也可以是一个方法.对于对象而言,对象的多态就是父类对子类的引用,比如Animal animal = new Cat()/new Dog()..,可以将编译类型为Animal的animal指向不同的实现了Animal的子类的对象引用.方法多态分为两种,一种是编译时的多态:方法重载,一种是运行时的多态:方法重写.

**方法重载**:是在同一个类中对一个方法的不同种描述.**重载的方法要求形参列表不同,名相同,对返回类型无要求**

**方法重写**:是在具有继承关系的子类中对父类的某个方法的重新实现.重写的方法要求和父类的方法名相同,参数列表相同,方法返回值相同,同时要求子类的访问权限要不小于父类的访问权限(当父类的方法为private或static时不能被重写)



创建Java对象的三种方法

1. new创建对象
2.  利用反射创建对象:使用Class类的newInstance方法
3. 利用clone()方法

## String StringBuilder StringBuffer

1. String StringBuffer StringBuilder的区别
   1. 三者都被final修饰,都不可被继承
   2. String是长度不可变的,而StringBuilder和StringBuffer是长度可变的
   3. StringBuffer是线程安全的,StringBuffer和StringBuilder的所有方法都相同.StringBuffer在StringBuilder的基础上添加了synchronized来保证线程安全,但同样的StringBuilder的性能更好
2. String不可变的原因
   1. 提高效率:因为String不可变,所以在创建值相同的String对象时,多个对象引用可以同时指向常量池中的一个对象,而无需创建多个对象.但如果String可变,多个对象引用同时指向一个对象的时候,一个对象引用发生改变就会造成其他的对象引用指向的对象发生改变,所以就要创建多个对象
   2. 安全:字符串常被作为url或文件路径,不可变标志存放资源的位置不会被篡改.
3. String不变性的理解
   1. String被final修饰,不可被继承
   2. 尽管有时我们会用+号来拼接字符串,但底层实现时调用的是StringBuilder的append方法
4. String的hashCode和equals方法
   1. 两个对象equals为true,则两个对象的hashCode相同
   2. 两个对象hashCode不相同,则两个对象的equals为false
   3. 两个对象hashCode相同,但两个对象的equal不一定为true(哈希冲突)
   4. 当重写了equals但没有重写hashCode时,对于散列函数Set,当在Set中先后存储两个值相同的对象时,由于hashCode没有重写,因此两个对象的hashCode不同,因此Set中会出现同时存在两个值相同的对象

## Java反射

反射机制是在**运行状态**中,对于任意一个类,都能够知道**这个类的所有方法和属性**,对于任意一个对象,都能够**调用它的任意一个方法和属性**

当我们在new一个对象时,JVM会进行类加载,从本地磁盘中找到对应类的.class文件,并根据.class文件生成对应的.class对象,反射就是根据.class对象来反向获取到类的信息

利用反射我们可以实现反编译,可以更便捷地写出一些非常灵活的代码.但反射也在一定程度上破坏了封装,使得安全性降低

获取.class的四种方法

```java
// 通过.class(类本身的class属性)
Class class1 = Student.class;

// 通过类.getClass()  运行时类的对象
Student stu = new Student();
Class class2 = stu.getClass();

// 通过Class.forName(className) Class的静态方法
Class class3 = Class.forName("Student");

// 通过类加载器
ClassLoader classLoader = this.getClass().getClassLoader();
Class class4 = classLoader.loadClass("student");
```

## static 和 final

1. static

   static表示全局和静态的意思,可以用来修饰属性,方法和代码块

   修饰属性:表示该变量在类加载就完成了初始化,内存中只有一个,所有类共享一个变量

   修饰方法:表示在类加载的时候实现,static方法必须实现,不能用abstract修饰

   修饰代码块:表示在类加载的时候执行

   执行顺序:父类静态代码块>子类静态代码块>父类非静态代码块>父类构造函数>子类非静态代码块>子类构造函数

2. final

   final表示无法改变,final可以修饰变量,方法,类,参数

   修饰变量:表示该变量是一个常量,不能够被改变

   修饰方法:表示该方法不可被重写

   修饰类:表示该类不可被继承

   修饰形参:表示形参不可变

3. static的注意事项

   1. 在static范围内不能使用非static的变量
   2. static变量是不安全的,因此涉及到static变量的在多线程下应该加锁(synchronized),但static方法是线程安全的,因为每个线程在使用static方法时都会创建新的变量
   3. static修饰的变量如果是一些基本数据类型或者常量(字符串常量)则在类加载的连接阶段赋值,其他的变量则类加载的初始化阶段赋值

4. 

```java
class A{
    public A() {
        System.out.println("A gouzao");
    }

        static A a = new A();
        static{
            System.out.println("static");
        }
        {
            System.out.println("A1");
        }
}
class B extends A{
    public B(){
        System.out.println("B");
    }
}
public class Inc {
    public static void main(String[] args) throws InterruptedException {
        /*
        执行顺序:
        1.首先执行static A a = new A();  执行A的普通代码块和A的构造器
        2.执行A的静态代码块
        3.执行A的普通代码块
        4.执行B的构造方法
         */
       B b = new B();
        /*
        A1
        A gouzao
        static
        A1
        A gouzao
        B
        */

    }
}
```

## Session 和 Cookie

1. 什么是Cookie?

   Cookie是服务器向客户端浏览器发送并保存到客户端本地的一小块数据.客户端在再次向相同的服务器发送请求时会携带上Cookie.服务器通过Cookie可以得知请求的来源.Cookie是无状态的HTTP协议也能够记录稳定的状态信息

2. 什么是Session?

   Session是客户端和服务器的一个会话,在服务器本地记录了某个用户会话的相关信息.通过Session可以使得在一次用户会话中用户的信息不会随着页面跳转等操作而丢失

3. Cookie和Session的区别

   1. 作用范围不同:Cookie作用在客户端,Session作用在服务器
   2. 存取方式不同:Cookie只能存储ASCII,Session可以存储任意类型的数据
   3. 有效期不同:Cookie可以被设置为长时间有效,比如默认的登录状态(CSDN),Session的有效期比较短,会随着客户端关闭或超时而失效
   4. 存储数据大小不同:Cookie存储的数据最大为4K,而Session可以存储更多的数据
   5. 安全性不同:Cookie存储在客户端,相较于Session而言安全性更低,更容易被窃取或篡改

4. 为什么需要Cookie和Session,它们是如何工作的

   由于HTTP协议是无状态的,因此在客户端和服务器通信时,客户端无法得知用户的身份信息和状态信息.为了获取到用户的信息,引入了Cookie和Session

   工作流程

   1. 用户第一次发送请求给服务器,服务器会将用户的信息保存到Session中,并将SessionID返回给客户端,客户端会将SessionID以及对应的域名保存到服务器
   2. 用户第二次请求时会查看对应的域名下是否有Cookie信息,如果有的话就将Cookie一起发送给服务器.服务器在收到Cookie后获取到SessionID后查看本地是否有对应的Session,如果有说明用户已登录,否则未登录.

5. 如果浏览器禁止了Cookie,如何验证用户是否登陆

   1. 每次请求时都携带一个SessionID
   2. 使用Token机制,Token是服务器生成的一个字符串(令牌).通过这个字符串代替SessionID来实现Cookie和Session的功能

6. 当服务器有多台时,用户对应的Session在服务器A上存储,当访问服务器B失效如何解决

   1. 规定一个客户端只能访问一个服务器
   2. 复制Session,当某个服务器上的Session发生变化,通过广播的形式将Session信息广播给其他服务器
   3. 共享Session,将所有服务器中存储的Session信息都存储到一个地方,保证所有服务器可以共享所有Session

7. Session的持久化

   服务器需要存储Session对象,这个Session对象即使失效也将存储在服务器中,以便于后续请求验证时能够被使用.当服务器存储了大量的Session对象时就会占据服务器大量空间,因此Session的持久化就是将服务器中一些未活跃但没有失效的Session对象存储到文件系统或数据库中,等使用的时候再取出来

## NIO BIO AIO

1. NIO

   **同步非阻塞**.客户端发送连接后直接返回,可以做自己的事情,但需要时不时的向服务器查询是否返回响应(轮询)

2. BIO

   **同步并阻塞**.客户端发送连接后处于等待过程直到服务器返回响应.

3. AIO

   **异步非阻塞**.客户端发送连接后直接返回,可以做自己的事情,当服务器返回响应后会通知客户端

同步与异步

同步:同步就是客户端向服务器发送请求后,服务器一直等到响应完成后才向客户端发送

异步:异步就是客户端向服务器发送请求后,服务器会立刻返回一个接收到请求的回应(此回应不是响应),客户端接收到回应后可以处理其他的请求,之后服务器通过回调等机制将响应返回给客户端

阻塞非阻塞的区别是在发起一个连接后是等待还是直接返回

## ThreadLocal

ThreadLocal提供了每个线程内部的局部变量.多线程环境下使用ThreadLocal维护变量时,会为每个线程生成该变量的副本,每个线程操作本线程中的变量副本.各个线程之间相互独立,互不干扰,从而保证了线程的安全.

在每个Thread中都维护了一个ThreadLocalMap,ThreadLocalMap中有一个Entry,其中key为ThreadLocal,value存放的是变量副本.

获取ThreadLocal中的value值,首先需要获取到当前线程的ThreadLocalMap,然后根据当前的ThreadLocal获取到对应的Entry,然后取出Entry中的value![1662104967663](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662104967663.png)

ThreadLocal中的主要方法

1. get():首先获取当前线程t,然后获取到当前线程t中的ThreadLocalMap,通过ThreadLocal(this)从ThreadLocalMap中找到对应的Entry,返回Entry的value

   ```java
   public T get() {
       Thread t = Thread.currentThread();
       ThreadLocalMap map = getMap(t);
       if (map != null) {
           ThreadLocalMap.Entry e = map.getEntry(this);
           if (e != null) {
               @SuppressWarnings("unchecked")
               T result = (T)e.value;
               return result;
           }
       }
       return setInitialValue();
   }
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
   private Entry getEntry(ThreadLocal<?> key) {
       int i = key.threadLocalHashCode & (table.length - 1);
       Entry e = table[i];
       if (e != null && e.get() == key)
           return e;
       else
           return getEntryAfterMiss(key, i, e);
   }
   ```

2. set():首先尝试获取到对应线程中的ThreadLocalMap,如果获取不到就创建一个ThreadLocalMap并写入第一个Entry,如果获取到在map中找到对应key相同的Entry,将新的value值复制,如果找不到,就在map中创建一个新的Entry并进行扩容

   ```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
   private void set(ThreadLocal<?> key, Object value) {
   
       // We don't use a fast path as with get() because it is at
       // least as common to use set() to create new entries as
       // it is to replace existing ones, in which case, a fast
       // path would fail more often than not.
   
       Entry[] tab = table;
       int len = tab.length;
       int i = key.threadLocalHashCode & (len-1);
   
       for (Entry e = tab[i];
            e != null;
            e = tab[i = nextIndex(i, len)]) {
           ThreadLocal<?> k = e.get();
   
           if (k == key) {
               e.value = value;
               return;
           }
   
           if (k == null) {
               replaceStaleEntry(key, value, i);
               return;
           }
       }
   
       tab[i] = new Entry(key, value);
       int sz = ++size;
       if (!cleanSomeSlots(i, sz) && sz >= threshold)
           rehash();
   }
   void createMap(Thread t, T firstValue) {
       t.threadLocals = new ThreadLocalMap(this, firstValue);
   }
   ```

3. remove():获取线程对应的ThreadLocalMap,然后找到符合条件的entry并删掉,并对map中的entry结构作出一些调整

   ```java
   public void remove() {
       ThreadLocalMap m = getMap(Thread.currentThread());
       if (m != null)
           m.remove(this);
   }
   private void remove(ThreadLocal<?> key) {
       Entry[] tab = table;
       int len = tab.length;
       int i = key.threadLocalHashCode & (len-1);
       for (Entry e = tab[i];
            e != null;
            e = tab[i = nextIndex(i, len)]) {
           if (e.get() == key) {
               e.clear();
               expungeStaleEntry(i);
               return;
           }
       }
   }
   
   ```

ThreadLocalMap利用**线性探测**解决哈希冲突

```java
private static int nextIndex(int i, int len) {
	return ((i + 1 < len) ? i + 1 : 0);
}
```

ThreadLocal的内存泄露问题

使用完ThreadLocal后,需要手动调用remove()方法,否则容易使value值发生内存泄漏

造成ThreadLocal内存泄漏的原因是Thread中的ThreadLocalMap的生命周期和当前线程的生命周期相同,且ThreadLocal实例使用完没有手动删除.

![1662107156292](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662107156292.png)

在Entry中key是弱引用,因此当使用完ThreadLocalRef后回收ThreadLocalRef后,因为key是弱引用,所以ThreadLocal也会被回收(GC机制),因此此时的key为null,但CurrentThread还在使用,因此无法访问到value,造成内存泄露

之所以key是弱引用,是为了将ThreadLocal的生命周期和Thread的生命周期解绑,同时弱引用意味着当ThreadLocal被回收后key为null,而ThreadLocal中的get,set方法针对key为null的情况作了优化,会认为该value值无效,从而避免内存泄漏

## finalize  finally

1. finalize

   垃圾回收器GC回收某个对象时,调用的是该对象的finalize方法.

2. finally

   在try catch finally代码块中,finally会在try或catch语句执行完后执行.finalliy中的语句是一定会执行的

   注意:尽管catch语句中有return语句,也不影响finally代码块中的语句执行,但finally代码块语句中的执行不会对catch的return结果造成影响

## public protected default private

![1662108185170](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662108185170.png)

## Object中有哪些方法

常用的方法有:equals,hashCode,wait,notify,clone,finalize,toString

![1662108233839](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662108233839.png)

## equals和==的区别

对于基本数据类型而言,没有equals一说,==比较的是两个数据的值

对于引用数据类型而言,equals比较的是两个数据的值,而==比较的是两个数据的地址

注意:StringBuffer StringBuilder特殊,==和equals均为比较地址(没有重写equals方法,调用的是Object的equals)

## 异常

1. Java异常体系

   异常就是指在程序执行期间发生的错误.异常可以分为两种Error和Exception

   1. Error:表示程序本身无法阻止的一种错误,这类错误一般比较严重,与代码本身无关.比如系统崩溃,内存异常等等.这类异常交由JVM处理,一般情况下会中断线程
   2. Exception:表示程序出现的一种可以被克服或执行的错误.这种异常又分为编译时异常和运行时异常.

2. Java异常类型,两种类型的区别

   Java异常可以分为受检异常和非受检异常,其中受检异常又称为编译时异常,非受检异常又称为运行时异常.

   受检异常的异常诊断发生在编译期,受检异常如果不作异常处理就无法通过编译

   非受检异常不影响程序的编译,会在运行时由JVM发出异常诊断

3. throw和throws的区别

   throw关键字是在程序中明确抛出异常,而throws是指当前方法无法处理异常,将异常抛给上一层去处理

   ```java
    public static void main(String[] args) {
        try {
            int num = 0;
            System.out.println(1/num);
        }catch (Exception e){
            throw new RuntimeException("发生了异常错误");
        }
   
    }
   ```

4. 常见的异常种类![1662109611613](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662109611613.png)

## 序列化

序列化:将实现了Serializable接口的对象转换成一个字节序列,并且能够将字节序列恢复成原来的对象.序列化能够弥补不同操作系统的差异.

序列化的实现:实现Serializable接口,可以选择重写/不重写readObject和writeObject方法.

为了保证安全性,对于一些不可以序列化的属性添加transient关键字,因为在反序列化时private的属性也可以被直接得到

反序列化的实现:实现Serializable接口

```java
//将对象序列化到文件中
public static void mySerialize(Object obj,String fileName) throws IOException {
    OutputStream out=new FileOutputStream(fileName);
    ObjectOutputStream objOut=new ObjectOutputStream(out);
    objOut.writeObject(obj);
    objOut.close();
}
/**
     * 从指定文件中反序列化对象
     * @param fileName
     * @return
     */
public static Object myDeserialize(String fileName) throws IOException,
ClassNotFoundException {
    InputStream in=new FileInputStream(fileName);
    ObjectInputStream objIn=new ObjectInputStream(in);
    Object obj=objIn.readObject();
    return obj;
}
```

注意:

1. 被static修饰的属性不会被序列化
2. 对象的类名和属性可以被序列化,但方法不可以序列化

## Comparable接口和comparator接口

1. Comparable接口

   在调用Arrays或Collections的sort方法时,待排序对象需要实现Comparable接口下的compareTo方法去指定使用对象的哪个属性来排序

2. Comparator接口

   在调用Arrays或Collietions的sort方法时,我们可以通过重写Comparator接口的compare方法来实现自定义排序

   二者的区别就是Comparable接口是在类内实现排序规则,Comparator接口是在类外排序时指定排序规则

## 接口和抽象类

1. 接口和抽象类的区别

   1. 抽象类可以有构造函数,而接口中不可以有构造函数

   2. 抽象类中可以有具体的方法,而接口中不可以有具体的方法(当然可以有default修饰的具体方法)

      ![1662195455231](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662195455231.png)

   3. 抽象类中可以有普通的成员变量,而接口中不能有普通的成员变量,但接口可以有常量

      ![1662195563782](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662195563782.png)

   4. 抽象类中的抽象方法对修饰符没有限制,而接口中要求抽象方法的修饰符必须是public(默认)

   5. 抽象类内可以包含静态方法,接口中不可以

   6. 抽象类和接口中都可以包含静态成员变量,但接口中的静态成员变量默认是public static final修饰.而抽象类中无要求

      ![1662195889745](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662195889745.png)

   7. 一个类可以实现多个接口,但只能实现一个抽象类

2. 抽象类中构造函数的意义

   首先确定一点抽象类中是可以写构造函数的.尽管抽象类不能被直接实例化.但在其他类继承了抽象类后,在子类中的构造方法中会执行super的构造函数来初始化抽象类中的一些成员变量.

3. 抽象类实现接口时的注意事项

   因为是抽象类,所以在实现接口时不需要全部实现接口中的抽象方法.接口相当于弥补了Java中没有多继承的缺陷.好的做法是提供一个抽象类来被继承,提供一个接口来生命类型,例如Java.util中的List和AbstractList

## Runtime

Runtime:运行时,是一个封装了JVM的类,每个Java程序都会启动一个JVM进程.而每个JVM进程都会对应一个Runtime实例.通过Runtime实例我们可以去指定JVM进程的某些行为.但Runtime实例不能直接获得,需要调用Runtime类的静态方法getRuntime(单例模式)![1662173801089](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662173801089.png)

## 泛型中?与T的区别

泛型:

泛型中?和T的区别

T指定了使**某一种类型**的数据,而?则不限于指定某一种类型的数据,也可以是**某些类型**的数据

## 字节流 字符流

字节流与字符流的区别



## 静态内部类 匿名类