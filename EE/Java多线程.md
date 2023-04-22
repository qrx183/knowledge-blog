# Java多线程

## Synchronized和Volatile

1. volatile

   volatile是JVM提供的轻量级同步机制,是线程不安全的.volatile保证了可见性和有序性.

   volatile的作用

   1. 保证内存可见性

      假设变量a已经从主内存中存储到了工作内存中,那么在内存中对变量a的值进行修改,工作内存中的变量

      a不会立刻更新.而volatile修饰的变量可以保证工作内存中的值是最新的(和主内存中的变量值一致)

      volatile保证内存可见性的原理:JVM会对添加了volatile关键字修饰的变量添加一条**lock前缀指令**.lock前缀指令在多核处理器中会做2件事情:将当前工作内存中的变量的值写回到主内存中;写回操作会引起其他处理器中缓存了该数据的地址无效.

      因此当某个变量被volatile修饰时,该当前CPU中的最新的变量写回到主内存中,同时为了保证多个处理器的缓存一致,就会实现**缓存一致性协议**,每个处理器会通过**检查主内存中的数据来判断自己缓存的数据是否已经过期**,如果发现两个数据的地址**不一致**,则认为当前缓存的数据的地址失效,之后该处理器在处理该数据时由于该数据已经无效所以**会从主内存中读取该数据的最新值.**

      ```java
      static int a = 1;
      public static void main(String[] args){
          Thread t1 = new Thread(){
              @Override
              public void run(){
                  int i = 1;
                  while(a == 1){
                      i++;
                  }
              }
          }
          Thread t2 = new Thread(){
              @Override
              public void run(){
                  a = 2;
              }
          }
          t1.start();
          Thread.sleep(1000);
          t2.start();
      }
      
      // volatile适用于变量在一个线程中只读取,在另一个线程中修改的场景
      // 上述程序当不添加volatile关键字,程序会无限执行下去,原因是因为t2线程对a变量作的修改t1线程看不到,t1线程每次读取的仍是自己工作内存中存储的a的旧值1.
      // 当添加了volatile关键字后,t2线程对a变量作的修改会同步到主内存上,同时会让t1线程工作内存内存放a变量的地址失效,从而再下一次读取的时候从主内存中读取到a的最新值
      ```

   2. 禁止指令重排序

      volatile实现禁止指令重排序是依靠JMM在编译器和处理器层面限制指令重排序,而**JMM的内存屏障策略是保守策略**

      > LoadLoadBarriers:该屏障之前的语句中数据的读取优先于该屏障之后所有语句中数据的读取
      >
      > StoreStoreBarriers:该屏障之前的语句中数据的存储优先于该屏障之后所有语句中数据的存储
      >
      > LoadStoreBarriers:该屏障之前的语句中数据的读取优先于该屏障之后所有语句中数据的存储
      >
      > StoreLoadBarriers:该屏障之前的语句中数据的存储优先于该屏障之后所有语句中数据的读取

   volatile的使用情况

   1. 当存在两个及以上的线程访问同一个变量时使用volatile,当要访问的变量已在synchronized代码块中或已经是常量时无须用volatile修饰
   2. volatile修饰的变量禁止了编译器中对代码的优化,所以执行效率低,必要时再使用

2. synchronized

   synchronized可以**保证线程的安全**,添加synchronized的代码块保证在同一时刻只能被一个线程执行,一旦一个线程执行持有synchronized的代码块后,其他线程想要处理该代码块就需要等待已经持有的线程释放锁,否则处于阻塞状态.除此之外,synchronized还可以保证持有该锁的线程对代码块的修改可以被其他线程共享,也就是synchronized可以完全代替volatile来**保证可见性**

   synchronized的原理

   在JVM中,每一个对象都有一个对象头,而对象头的Mark Word中记录着该对象持有的锁类型

   ![1662449802155](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662449802155.png)

   synchronized的对象锁是一把**重量级锁**时,该锁指向一个monitor对象(监视器锁),每一个对象都与一个monitor关联,当一个monitor被某个线程占有时,它便处于锁定状态.monitor有ObjectMonitor实现.当多个线程同时请求某个对象监视器时,对象监视器会将多个线程进行区分:

   1. Contension List:所有请求该对象监视器的线程首先都会进入到Contension竞争队列中

   2. Entry List:Contension List中有机会成为候选人的线程进入Entry队列中

   3. Wait Set:调用wait方法而阻塞的线程会被放入到Wait Set中

   4. OnDeck:抢占到锁的线程被称为OnDeck,同一时刻OnDeck只能有一个

   5. Owner:获得锁的线程

   6. !Owner:释放锁的线程

      ![1662450979818](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662450979818.png)

      同步代码块:多个线程同时访问一个代码块,这些线程首先会进入EntryList中,抢占到锁对象的monitor后通过OnDeck进入Owner区域,同时将monitor对象的count值+1(由0到1),执行代码块中的内容,若进入Owner区域的线程调用了wait方法则将count的值变为0,同时释放持有的monitor同时进入到WaitSet中

      同步方法:多个线程同时访问一个实例方法,线程会通过检查ACC_SYNCHRONIZED标志是否被设置,如果被设置就持有该方法的monitor,成功获取monitor后执行该方法,执行结束后释放monitor.

      注意:synchronized是可重入的,抢占到synchronized的线程会将维护的count变量值从0变为1,因为synchronized是可重入的,因此当当前线程继续占用该锁时,count变量会从1变为2,之后释放的时候也是从2---1---0

   synchronized加锁过程的优化

   为了提升加锁的效率,JVM将synchronized锁对象分成了无锁,偏向锁,自旋锁,重量级锁四种.

   当没有线程占用锁对象时处于无锁,当有一个线程占用该锁时变成偏向锁,偏向搜不是真的加锁,而是添加一种锁标记,如果之后没有发生锁竞争就无须开销锁资源,如果有锁竞争就添加自旋锁,自旋锁是一种轻量级锁,在锁冲突不激烈的情况下,在抢占不到锁时不会处于阻塞状态而会一致尝试抢占锁,当锁竞争激烈时,自旋锁不能马上获取到锁,就会膨胀为重量级锁,调用OS的mutex将没有抢占到锁的线程加入到阻塞队列中,重量级锁的具体执行过程如上述介绍

## CAS

CAS:Compare and Swap,比较并交换.CAS的原理是存在三个值内存中的值A,旧的预期值B和要修改成的值C,当A和B相等就将内存值A改为C,否则什么都不做

在Java中的AtomicInteger类中就运用CAS实现了线程安全的加法操作.

CAS的缺点就是无法解决**ABA问题**.

ABA问题就是将原本值为A的变量的值改为B,之后再把这个变量的值重新改为A.

CAS的Compare比较的是内存中的值和旧的预期的值,比较的值相等意味着内存中的值没有被改变过,而ABA问题显然改变了内存中的值.为了区分值相等是因为ABA还是没有发生过改变,可以通过**控制变量的版本**(版本自增)来保证CAS的正确性

## ReentrantLock

ReentrantLock是基于AQS实现的,AQS是基于CAS的一种队列.

在ReentrantLock有一个抽象类Sync实现了AbstractQueueSynchronizer(AQS)

ReentrantLock的构造方法中的参数决定了ReentrantLock是公平锁(true)还是非公平锁(false/默认),公平锁就是先到达的线程优先抢占到锁,非公平锁就是所有线程随机抢占到锁

ReentrantLock最常用的方法就是lock和unlock

**lock**

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

首先当当前锁空闲时,state值为0,第一个抢占到该锁的线程会执行if语句,利用CAS将当前线程的值设置为1,同时将AbstractQueueSynchronizer的Thread设置为当前线程

之后当当前锁被占用时,state值为1,之后竞争该锁的线程会进入else语句的acquire(1)

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

首先会继续尝试占用锁(tryAcquire),tryAcquire会执行Sync.nonfairTryAcquire方法

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

首先会继续尝试占用锁,之后如果是当前线程嵌套锁就给state的值+1(**ReentrantLock是可重入锁**),否则返回false

然后继续调用addWaiter方法:创建一个当前线程的Node然后加入到AQS队列中

然后调用acquireQueued方法将线程阻塞

至此lock方法实现了某个线程独占锁,其他线程加入到AQS队列中阻塞

**unlock**

```java
public void unlock() {    sync.release(1);}
```

调用sync.release方法

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

首先尝试释放锁(tryRelease)

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

只有一个线程完全对一个ReentrantLock全部解锁后才能真正释放(即一个lock对应一个unlock),然后将占有当前锁的线程设置为空.然后走unparkSuccessor(h)方法

```java
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

将AQS队列中的第一个节点取出然后调用unpark方法令该线程占用锁,同时从AQS队列中删除该Node节点.AQS队列结构的调整得益于acquireQueued方法

```java
final boolean acquireQueued(final Node node, int arg) {
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } catch (RuntimeException ex) {
        cancelAcquire(node);
        throw ex;
    }
}
```

注意这里的**for循环是个无条件循环,且只有队列的第一个节点抢占到锁时才返回**,因此当队列中某个线程抢占到锁时会第一时间执行第一个if语句然后调整AQS队列的结构



ReentrantLock的底层实现

首先ReentrantLock是一把可重入锁,它是基于Java当中的一个类,是基于 AQS同步器实现的,而AQS是一个运用了大量CAS的队列.在ReentrantLock类中,有3个常用的方法lock,unlock和trylock,相比如synchronized而言,ReentrantLock的trylock更加灵活,它的含义是在抢占锁失败后不会马上进入阻塞队列,而是在规定时间内不断的尝试抢占锁,相当于是一个自旋锁.

而lock方法它会首先利用CAS操作去尝试将内存中的state的值从0改为1,如果成功说明第一个线程抢占到该锁,然后把AQS的线程设置为当前线程,如果CAS操作失败,就尝试将线程加入到AQS队列中,在加入的过程中会不断尝试锁资源是否被释放,如果没有释放,则会判断AQS队列是否为空,如果为空就创建队列将线程包装成Node加入到队列,否则直接加入到队列

而unlock方法则是调用AQS的release方法,release方法首先判断state的值是否为0,如果不为0说明存在可重入锁,释放失败,单纯的将state-1,如果state的值为0,则将队列中第一个节点的线程取出并占用锁资源,然后调整队列的结构.

ReentrantLock和synchronized的区别

1. synchronized是一个关键字,是JVM内部实现的;ReentrantLock是标准库的一个类,是基于Java实现的
2. synchronized使用时不需要手动释放锁,ReentrantLock需用手动调用unlock释放锁
3. synchronized只有抢占锁和阻塞两种状态,而ReentrantLock在这两者基础上还有一个更灵活的tryLock去在一定时间内尝试获取锁
4. synchronized是非公平锁,ReentrantLock默认是非公平锁,但通过传入true参数可以设置成公平锁

## 常用锁策略

1. 乐观锁 VS 悲观锁

   1. 乐观锁:假设锁冲突发生的概率比较低,在数据提交更新的时候检测是否发生并发冲突,如果发生则将这种错误返回给用户
   2. 悲观锁:假设锁冲突发生的概率比较高,因此每次拿到数据时都会为数据加锁,其他线程只能处于阻塞状态

2. 公平锁 VS 非公平锁

   1. 公平锁:根据线程的先后顺序来分布抢占到锁的优先级,先到先得.
   2. 非公平锁:所有线程抢占到锁的概率是随机的

3. 自旋锁:当获取线程失败后不会处于阻塞状态,而是一直尝试获取锁.自旋锁是一种轻量级锁的具体实现

4. 偏向锁:偏向锁不是真正的加锁,而是添加一个锁标记,之后如果没有发生锁竞争,就无须加锁,当发生锁竞争,就添加轻量级锁

5. 轻量级锁 VS 重量级锁

   1. 重量级锁:加锁机制重度依赖OS的mutex
   2. 轻量级锁:加锁机制尽可能在用户态代码完成,实在不行在使用OS的mutex

   重量级锁和轻量级锁其实是悲观锁和乐观锁的一种体现,重量级锁一般是在锁竞争比较激烈的场景使用,而轻量级锁一般是在锁竞争不激烈的场景使用

## 创建线程的方式

1. 继承Thread类

   ```java
   Thread t = new Thread(){
       @Override
       public void run() {
           System.out.println("通过继承Thread类实现线程创建");
       }
   }
   ```

2. 实现Runnable接口

   ```java
   Thread t = new Thread(new Runnable() {
       @Override
       public void run() {
           System.out.println("通过Runnable创建线程");
       }
   })
   ```

3. 创建线程池

   ```java
   static int k = 0;
   public static void main(String[] args) throws InterruptedException, ExecutionException {
       int taskNum = 3;
       ExecutorService pool = Executors.newFixedThreadPool(taskNum);
       List<Future> l = new ArrayList<>();
       for(int i = 0; i < taskNum; i++){
           Callable c = new Callable() {
               @Override
               public Object call() throws Exception {
                   return new String("该任务是第" + ++k + "个任务");
               }
           };
           Future f = pool.submit(c);
           l.add(f);
       }
       for(int i = 0; i < taskNum; i++){
           System.out.println(l.get(i).get().toString());
       }
   }
   ```

4. 实现Callable接口

   ```java
   Callable c = new Callable() {
       @Override
       public Object call() throws Exception {
           return new String("通过Callable创建线程");
       }
   };
   FutureTask<String> f = new FutureTask<>(c);
   Thread t = new Thread(f);
   t.start();
   System.out.println(f.get());
   ```

5. 四种创建线程方式的比较

   实现Runnable接口相较于Thread类可以更轻松的实现资源共享,且Runnable接口也弥补了单继承的机制.同样线程池中只支持存储Runnable和Callable对象.

   实现Callable接口可以得到有返回值的线程,剩下三种线程创建方式都是无返回值的

   综上所述:如果**有返回值则使用Callable**创建线程,**没有返回值则使用Runnable**创建线程.而Callable和Runnable实际上创建的是具体要执行的任务,把这些任务**存储到线程池中**由线程池管理要比单独创建Thread这样的野线程的效率要高同时也更方便管理

## 线程状态(线程生命周期)

![1662536842697](C:\Users\qiu\AppData\Roaming\Typora\typora-user-images\1662536842697.png)

线程的生命周期

首先new Thread进入初始状态(New),调用t.start进入可运行状态(就绪状态/Runnable),当线程被os调度选中进入运行中状态(Running).阻塞状态是由运行中的状态调用sleep或join方法进入阻塞状态,未抢占到锁的其他线程会进入到锁池中,而调用wait方法则会进入等待队列,等待被唤醒.线程结束运行意味着线程的生命周期结束

## 一般线程和守护线程的区别

一般线程即为在程序运行期间执行的线程,而守护线程是在程序运行期间**在后台提供一种通用服务**的线程,守护线程不属于程序中必要的部分.

一般线程和守护线程的区别在于一般线程在JVM虚拟机退出之前结束,而守护线程在JVM虚拟机退出之后结束.(守护线程负责保障和收尾)

一般线程可以通过Thread.setDaemon(true)来变成守护线程.但转变的前提是在启动start之前,一个运行中的一般线程是不能转化为用户线程的,用户线程中的所有线程都属于用户线程

## 多线程常用方法

1. sleep和wait的区别

   sleep是Thread类中的方法,含义是暂停一段时间,然后继续执行该线程.**sleep方法不会释放占用的对象锁**,sleep使当前线程进入阻塞状态

   wait是Object类的方法,调用wait方法会让该对象释放占用的锁,进入等待队列

   二者的区别:

   1. sleep是Thread类的方法,wait是Object类的方法
   2. 调用sleep方法不会释放对象锁,调用wait方法会释放对象锁
   3. sleep方法使用的时候必须处理异常(InterruptedException,防止该线程在休眠期间,其他线程调用该线程的interpret方法来中止该线程),wait方法不用处理
   4. sleep的作用范围为程序的任意位置,而wait方法只能在某个锁对象的同步代码块中

2. yield:将当前线程从Running状态变为Runnable状态,依次将执行机会让给具有更高级别或同等级别的其他线程,如果没有其他线程,yield方法将不起作用

3. join:在某一个线程内调用另一个线程,保证调用的线程先执行,结束后再执行本线程

4. notify:唤醒一个处于等待队列中的线程(若有多个线程处于等待队列中,具体哪个线程被唤醒取决于**操作系统的调度**)

## 中断线程

中断线程的方式

1. 使用标志位标志线程退出的条件,该标志为为了保证所有线程的可见通常用volatile关键字修饰
2. 调用t.interpret方法中断线程

## 多线程如何避免死锁

1. 什么是死锁

   死锁是指多个进程因竞争资源而造成的一种相互等待的僵局.

   死锁产生的4个必要条件

   1. 互斥条件:进程要求对所分配的资源进行**排他性控制**,即某段时间内一个资源只能被一个进程所占有,其他进程在请求该资源时只能等待
   2. 不剥夺条件:进程使用的资源不能被其他进行强行剥夺,资源的释放只能由使用者主动释放
   3. 请求和保持条件:一个进程在占有某个资源的同时(不释放),又申请了一个新的被别的进程占有的资源
   4. 循环等待条件:存在一条循环链,链中每个进程请求的资源被链中下一个进程占有

2. 如何避免死锁

   1. 规定加锁顺序

      设置一套加锁的顺序,所有进程在申请锁资源时要按照加锁的顺序进行,比如有三把锁A,B,C加锁顺序是A---B---C,那么对于任何一个进程在加锁C前需要先加锁B和锁A.

      这种方式的缺点是你必须事先知道加锁规则中的所有锁才能使用

   2. 设置加锁时限

      每个进程在尝试获取锁时要设定一定的超时时间,超过规定的这个时间就放弃获取该锁.

      这种方式的缺点是无法判别一个进程执行时间长的原因是因为发生死锁还是本身执行的逻辑导致,同时当多个线程同时竞争时,存在某个线程一直获取不到资源的情况

   3. 死锁检测

      死锁检测指的是当一个进程抢占某个资源时,在一个数据结构中记录先该进程,没有抢占资源的进程也作记录.这样当一个进程在请求锁时发现锁资源请求失败时,就可以在锁资源对应的数据结构上查找是否是某个线程已经占用了该锁,从而判断是否发生了死锁

      ```java
      // 死锁检测的逻辑
      import java.util.*;
      
      /**
       * @author qiu
       * @version 1.8.0
       */
      public class Main {
          static HashMap<Integer,SourceDivideList> map;
          static boolean flag;
          static List<Integer> isUsed;
          public static void main(String[] args) {
              flag = false;
              map = new HashMap<>();
              isUsed = new ArrayList<>();
              // 把当前正在被使用的锁资源更新到map中
              SourceDivideList s1 = new SourceDivideList();
              s1.usedId = 1;
              s1.waitIdList = new ArrayList<>();
              s1.waitIdList.add(2);
              SourceDivideList s2 = new SourceDivideList();
              s2.usedId = 2;
              s2.waitIdList = new ArrayList<>();
              s2.waitIdList.add(3);
              SourceDivideList s3 = new SourceDivideList();
              s3.usedId = 3;
              s3.waitIdList = new ArrayList<>();
              s3.waitIdList.add(1);
              map.put(s1.usedId,s1);
              map.put(s2.usedId,s2);
              map.put(s3.usedId,s3);
              Set<Integer> set = map.keySet();
              for(Integer id : set){
                  SourceDivideList s = map.get(id);
                  dfs(s);
                  if(flag){
                      System.out.println("发生了死锁");
                      return;
                  }
              }
          }
          private static void dfs(SourceDivideList s){
              if(isUsed.contains(s.usedId)){
                  flag = true;
              }
              if(flag){
                  return;
              }
              isUsed.add(s.usedId);
              for(Integer id : s.waitIdList){
                  if(map.containsKey(id)){
                      dfs(map.get(id));
                  }
              }
      
          }
          static class SourceDivideList{
              int sourceId;
              int usedId;
              List<Integer> waitIdList;
          }
          // 资源1--->usedId:1,waitIdList:2
          // 资源2--->usedId:2,waitIdList:1
      }
      
      ```

      当死锁检测成功后,针对死锁对应的循环链中,可以通过设置进程的优先级来让循环链中的某些线程主动释放锁,从而解除死锁

## 线程池

1. 线程池的好处

   线程池就是事先创建好几个线程存储到线程池中,这样之后使用线程就从线程池中取,用完直接放回线程池.而无需每次都创建和销毁线程,减少了资源的开销,同时利用Thread创建线程时线程和任务是绑定在一起的,灵活性很差,而线程池可以将任务分离出来处理然后再放入线程池,灵活性更强.

2. 线程池的submit的原理

   在AbstractExecutorService中实现了Submit方法

   ```java
   public Future<?> submit(Runnable task) {
       if (task == null) throw new NullPointerException();
       RunnableFuture<Void> ftask = newTaskFor(task, null);
       execute(ftask);
       return ftask;
   }
   ```

   但这个其实是个模板,真正执行的是ThreadPoolExecutor类中重写的execute方法

   ```java
   public void execute(Runnable command) {
       if (command == null)
           throw new NullPointerException();
       int c = ctl.get();
       if (workerCountOf(c) < corePoolSize) {
           if (addWorker(command, true))
               return;
           c = ctl.get();
       }
       if (isRunning(c) && workQueue.offer(command)) {
           int recheck = ctl.get();
           if (! isRunning(recheck) && remove(command))
               reject(command);
           else if (workerCountOf(recheck) == 0)
               addWorker(null, false);
       }
       else if (!addWorker(command, false))
           reject(command);
   }
   ```

   在execute方法中,首先判断当前线程数是否小于核心线程数目,如果小于则直接创建新的核心线程执行任务,否则将任务添加到任务队列中(如果队列满了就创建新的非核心线程来执行任务),添加成功后进行双重校验确保线程池正常运行.如果当前线程数大于了最大线程数,那么执行拒绝策略

3. 线程池的参数,拒绝策略是什么

   Java中的线程池的创建常用的类是Executors工厂类,而Executors实际上是对ThreadPoolExecutor类的一种封装,实际调用的是ThreadPoolExecutor.

   在ThreadPoolExecutor中,有一些重要的参数

   ```java
   public ThreadPoolExecutor(int corePoolSize,
                             int maximumPoolSize,
                             long keepAliveTime,
                             TimeUnit unit,
                             BlockingQueue<Runnable> workQueue,
                             ThreadFactory threadFactory,
                             RejectedExecutionHandler handler) {
       this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
            Executors.defaultThreadFactory(), defaultHandler);
   }
   ```

   > corePoolSize:线程池核心线程的数量
   >
   > maximumPoolSize:线程池可创建的最大线程数量
   >
   > keepAliveTime:当线程数量超过corePoolSize时,空闲线程的时间超过该值就会被销毁
   >
   > unit指定了keepAliveTime的单位
   >
   > workQueue:存储未执行的任务队列
   >
   > threadFactory:创建线程的工厂
   >
   > handler:拒绝策略,表示任务队列已满且没有空闲线程可以再执行任务时对新任务的处理策略
   >
   > > AbortPolicy():超过负荷,直接抛出异常
   > >
   > > CallerRunPolicy():由调用者负责处理
   > >
   > > DiscardOldestPolicy():丢弃队列中最老的任务
   > >
   > > DiscardPolicy():丢弃新来的任务

4. 线程池的类型

   1. 固定线程数的线程池:newFixedThreadPool
   2. 线程数目动态增长的线程池:newCachedThreadPool
   3. 只包含单个线程的线程池:newSingleThreadExecutor
   4. 设定可以延时执行的线程池:newScheduleedThreadPool

5. 一个线程池中的部分:线程管理器(池子),工作线程(Worker),任务接口(Task),任务队列(TaskQueue)

## juc包

juc是java.util.Concurrent包的简称,在juc包下大致可以分为以下几部分

> Atomic:AtomicInteger..
>
> Locks:Lock,Condition,ReadWriteLock,LockSupport,ReentrantLock
>
> Collections:Queue,ConcurrentHashMap
>
> Executor:Future,Callable,Executor
>
> Tools:CountDownLatch,CyclicBarrier,Semaphore,Executors

1. 原子类

   原子类就是具有原子性的一种类.原子类通过使用CAS来保证操作的原子性,进而保证了并发的安全性.

   AtomicInteger中的运算操作通过调用Unsafe类的CAS操作实现

   ```java
   public final int getAndAddInt(Object var1, long var2, int var4) {
       int var5;
       do {
           var5 = this.getIntVolatile(var1, var2);
       } while(!this.compareAndSwapInt(var1, var2, var5, var5 +var4));
   
       return var5;
   }
   // var1是当前对象,var2是预期的旧值,var5是内存中的值,当var2和var5相等时,将内存中的值进行更新
   ```

   原子类其实是一种乐观锁,适用于在锁冲突不激烈的情况下.因此在高并发的情况下AtomicInteger的性能比较差,差的原因是因为某个线程作出的CAS更新会因为volatile而让其他线程正在进行的CAS操作失效.因此在高并发下使用LongAdder方法来代替AtomicLong,LongAdder是每个线程独享一个cell数组,在竞争不激烈的情况下直接将结果更新到base上,竞争激烈各个线程对自己的cell数组中的某个对象操作,然后调用sum求和

2. ReadWriteLock

   ReadWriteLock是一种读写锁,对读和写操作分别加不同的锁,读锁是共享式的,多个线程可以同时共享一个资源,写锁是独占式的,一个资源同一时刻只能有一个写锁

   实现ReadWriteLock接口的类时ReentrantTReadWriteLock,而ReentrantReadWriteLock中核心的两个类是writeLock和readLock,wirteLock走的就是ReentrantLock的那一套,而readLock的模板其实和wirteLock一致,只是锁的类型从独占式(MODE)变为了共享式(SHARED)

3. CountDownLatch

   CountDownLatch适用于一个或多个线程需要等待其他线程均执行完后再执行相关逻辑

   CountDownLatch的主要方法为countDown和await,初始化CountDownLatch时会指定count的值,await释放的条件为count==0,而CountDwonLatch每次执行都会让count-1,当调用CountDownLatch的所有线程都执行完,count变为0,await被释放

   CountDownLatch的计数无法被重置,如果需要重置计数需要使用CyclicBarrier

4. CyclicBarrier

   CyclicBarrier是一种循环屏障,等到所有线程到达屏障之后再继续执行后续的代码.CountDownLatch需要**每一个线程**调用countDown来对count-1.而CyclicBarrier只需要在任何一个线程中调用await方法就可以阻塞,没有countDown这样的方法.并且CyclicBarrier可以对计数重置

5. Semaphore

   Semaphore是信号量,它可以控制多个线程同时访问一个资源,可以控制并发的线程数

   Semaphore有两个主要的方法:acquire和release

   acquire表示申请访问该资源,如果访问到Semaphore中资源个数-1;如果没有访问到则阻塞

   release表示申请到资源的线程释放资源

6. LockSupport

   LockSupport也是一种阻塞工具,通过park阻塞进程,通过unpark来唤醒进程,LockSupport引入了许可的概念,每次调用park时会检测当前许可是否有效,如果有效则消费掉该许可然后执行park,每次调用unpark获得许可.**LockSupport中的许可证最多有1个,不可堆积**

7. ArrayBlockingQueue和LinkedBlockQueue

   ArrayBlockingQueue是基于数组实现,保证并发安全性的是ReentrantLock和Condition

   put方法:利用ReentrantLock加锁,然后判断队列是否满,如果已满则通过await方法挂起,被其他线程唤醒后执行添加元素入队列的过程,然后通过signal唤醒take的线程.释放锁

   take方法:利用ReentrantLock加锁,然后判断对垒是否已空,如果为空则通过await方法挂起,被其他线程唤醒后执行删除元素,然后通过signal唤醒put的线程,释放锁

    

   LinkedBlockingQueue是基于双向链表实现,保证并发安全性的是ReentrantLock和Condition,同时LinkedBlockingQueue中提供了两把锁分别用来读和写,可以同时在首尾进行操作,并发的效率更高效.入元素和出元素的逻辑和ArrayBlockingQueue类似

    

   ArrayBlockingQueue和LinkedBlockingQueue的区别

   1. 队列大小不同,Array在初始化时必须指定队列大小,而Linked可以指定,可以不指定,在添加元素速度大于插入元素速度时Linked容易发生内存溢出问题
   2. 元素存储方式不同,Array中存储的元素是数组,Linked存储的元素是链表
   3. Array中只有一个锁,而Linked中的锁是分离的,添加元素用putLock,删除元素时用takeLock.并发时效率更高

## 单例模式

单例模式就是一个类在内部创建一个对象,调用者不能自己去创建该类的实例,只能通过指定方法来获取类中唯一的实例对象.

单例模式 VS 静态类

1. 单例可以继承和被继承,静态类不能继承
2. 静态类中的对象在执行结束后会被GC清理,无法存储在内存中
3. 静态类中的初始化在类加载的时候完成,单例模式可以实现延迟加载(懒汉)

单例模式有2种模式,一种是饿汉模式,一种是懒汉模式.

饿汉模式:每次在创建类的时候就对该类中的唯一对象实例化

```java
public class SingTon1{
    private static SingTon1 singTon1 = new SingTon1();
    private SingTon1(){

    }
    public SingTon1 getSingTon1(){
        return singTon1;
    }
}
```

懒汉模式:每次在创建类的时候不对该类中的唯一对象实例化,而是在外部调用时再对该对象实例化

```java
public class SingleTon {
    private static volatile SingleTon singleTon = null; // volatile保证禁止指令重排序
    private static final Object OBJECT = new Object();
    private SingleTon(){

    }
    public SingleTon getSingleTon(){

        if(singleTon == null) { // 防止无效锁资源的开销
            synchronized (OBJECT) { // 保证线程安全
                if (singleTon == null) {
                    singleTon = new SingleTon();
                }
            }
        }
        return singleTon;
    }
}
```

静态内部类实现单例模式.可以保证线程安全,也可以实现延迟加载.线程安全是由于静态内部类只会加载一次,在第一个调用getSingleTon的时候加载后之后就不再加载

```java
public class SingleTon {
    private SingleTon(){

    }
    // 不会再SingleTon类加载的时候加载,而是在调用getSingTon时加载
    private static class SingleTonInstance{
        private static final SingleTon singleTon = new SingleTon();
    }
    public SingleTon getSingleTon(){
    	return SingleTonInstance.singleTon;
    }
}
```

枚举类实现单例模式.可以保证线程安全(枚举类型本身是线程安全的,只会装载一次),但不能实现延迟加载,除了枚举类,其他形式的单例模式都可以通过反射来破坏,而反射在处理枚举时会抛出异常

```java
public class SingleTon{
    
    private SingleTon(){
        
    }
    private enum SingleTonInstance{
        INSTANCE;
        private SingleTon instance;
        
        SingleTonInstance(){
            instance = new SingleTon();
        }
        
        private SingleTon getInstance(){
            return instance;
        }
    }
    public static SingleTon getInstance(){
        return SingleTonInstance.INSTANCE.getInstance();
    }
}
```

## 进程,线程

1. 概念

2. 1. 进程是并发执行的程序在执行过程中分配和管理资源的基本单位,是一个动态概念,竞争计算机系统的基本单位
   2. 线程是进程的一个执行单元,是进程内的一个调度实体.
   3. 协程是一种比线程更加轻量级的存在,一个线程也可以拥有多个协程.

2. 进程和线程的区别
   1. 进程是操作系统分配和管理资源的基本单位,而线程是操作系统调度的基本单位
   2. 资源:线程共享一个进程内的资源,如内存,I/O,cpu等,不利于资源的管理和保护,而进程之间的资源是独立的,能很好的进行资源管理和保护
   3. 从健壮性方面考虑,多进程要比多线程健壮,一个进程崩溃后不会对其他进程造成影响,但一个线程崩溃会造成进程内的其他线程都崩溃
3. 使用场景
   1. 当对资源的管理和保护要求比较高,对资源开销和效率要求比较低时考虑使用多进程
   2. 当频繁进行切换,对效率要求比较高,对资源的管理保护要求比较低时考虑使用多线程


