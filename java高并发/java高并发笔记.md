- [基本概念](#基本概念)
  - [同步和异步](#同步和异步)
  - [并发和并行](#并发和并行)
  - [临界区](#临界区)
  - [阻塞和非阻塞](#阻塞和非阻塞)
  - [死锁、活锁、饥饿](#死锁活锁饥饿)
  - [并发级别](#并发级别)
    - [阻塞](#阻塞)
    - [无饥饿](#无饥饿)
    - [无障碍](#无障碍)
    - [无锁](#无锁)
    - [无等待](#无等待)
  - [原子性、可见行、有序性](#原子性可见行有序性)
    - [原子性](#原子性)
    - [可见性](#可见性)
    - [有序性](#有序性)
- [java并行程序基础](#java并行程序基础)
  - [线程](#线程)
  - [线程的创建](#线程的创建)
  - [线程中断](#线程中断)
  - [wait和notify](#wait和notify)
  - [join和yield](#join和yield)
  - [volatile](#volatile)
  - [守护线程](#守护线程)
  - [优先级](#优先级)
  - [synchronized](#synchronized)
- [JDK并发包](#jdk并发包)
  - [重入锁](#重入锁)
    - [重入锁的使用](#重入锁的使用)
    - [公平锁](#公平锁)
    - [Condition](#condition)
  - [信号量](#信号量)
  - [读写锁](#读写锁)
  - [CountDownLach](#countdownlach)
  - [线程池](#线程池)
    - [线程池的概念](#线程池的概念)
    - [线程池的种类](#线程池的种类)
    - [计划任务](#计划任务)
    - [线程池的实现](#线程池的实现)
    - [线程数量的选择](#线程数量的选择)
  - [并发容器(待补完)](#并发容器待补完)
    - [ConcurrentHashMap](#concurrenthashmap)
- [锁优化](#锁优化)
  - [提高锁的性能的几种方式](#提高锁的性能的几种方式)
    - [减少持有时间](#减少持有时间)
    - [减少锁的粒度](#减少锁的粒度)
    - [使用读写分离锁](#使用读写分离锁)
    - [锁分离](#锁分离)
    - [锁粗化](#锁粗化)
  - [JDK的锁优化](#jdk的锁优化)
    - [锁偏向](#锁偏向)
    - [轻量级锁](#轻量级锁)
    - [自旋锁](#自旋锁)
    - [锁消除](#锁消除)
  - [ThreadLocal](#threadlocal)
  - [无锁](#无锁-1)
    - [CAS](#cas)
    - [AtomicInteger](#atomicinteger)
    - [Unsafe类](#unsafe类)
  - [并行模式和算法](#并行模式和算法)
    - [单例模式](#单例模式)
# 基本概念

## 同步和异步

同步和异步一般用于形容一次方法调用

* 同步方法一旦开始，那么调用者必须等待方法返回之后才能够进行进行后续的行为
* 异步方法开始后会立即进行返回，调用者可以进行之后的操作。而调用的方法会在另一个线程中执行

## 并发和并行

并发和并行都是**多个任务同时进行**，但是偏重点不一样：
* 并发：**偏重于多个任务交替执行，多个任务之间是可以串行的**
* 并行：**是真正的同时执行**

对于单核系统来说，多进程或者多线程一定是并发的不可能是并行的

并行只可能发生在多核系统中

## 临界区
临界区表示一种公共资源或者说共享数据，可以被多个线程使用，但是同时只能有一个线程进行使用。

## 阻塞和非阻塞

阻塞和非阻塞用来表示线程之间的相互干扰

* 阻塞：线程如果占用率临界区资源，那么其他线程必须等待，等待导致线程挂起称为**阻塞**
* 非阻塞：没有线程能够妨碍其他线程执行，所有的线程都会尝试不断向前执行

## 死锁、活锁、饥饿

死锁是指两个或者两个以上的线程因为争夺资源而进入相互等待的状态，如果发生死锁不加以外部作用是不能解除的

死锁发生有四个条件

1. 互斥：对于资源的访问是互斥的
2. 请求与保持：持有资源的被请求资源后会继续保持
3. 不可剥夺：持有的资源不可被其他线程剥夺
4. 环路等待 ：等待形成环路


饥饿：一个或者多个线程由于优先级较低导致一直无法执行

活锁：两个线程都可以使用资源，但是两个线程相互谦让导致谁都无法使用资源

## 并发级别
并发级别可以分为以下几种
* 阻塞
* 无饥饿
* 无障碍
* 无锁
* 无等待

### 阻塞

其他线程释放资源之前，当前线程无法继续执行

使用synchronized或者可重入锁的时候就是**阻塞**的级别

### 无饥饿

使用公平的锁的时候，不管来的线程的优先级怎么样都需要进行排队，这样就不会发生饥饿的问题

### 无障碍

所有线程都可以同时进入临界区，如果发生冲突的话，会立即对于自己做出的修改进行回滚来确保数据的安全性

可以说这种同步的方式是一种**乐观**的策略

### 无锁

无锁与无障碍类似，但是无锁能够保证至少有一个线程能够执行下去，在修改中“胜出”

但是其存在一个问题，可能有一直在临界区中竞争失败的线程，会不断进行尝试导致线程停滞不前
```java
while(!atomicVar.compareAndSet(localVar,localVar+1))
{
    localVar=atomicVar.get();
}
```

### 无等待
要求所有线程必须在有限步内完成，这样不会导致长时间的等待

比较典型的设计是RCU：
* 读线程的时候是无等待的，因为其不会引起任何冲突
* 写数据时先取得原始数据的副本。接着只修改副本数据，之后在合适的时机将副本中的数据写回

## 原子性、可见行、有序性

### 原子性

原子性表示一个操作是**不可中断的**，即便是多个线程一起执行的时候也不会别其他线程干扰

对于一个静态全局变量，两个线程对其赋值，最终其值要么是A线程赋的，要么是B线程赋的，不存在相互作用

但是对于long型变量不会是这样（32位虚拟机）
```java
new Thread(new changeT(111L)).start();
new Thread(new changeT(222L)).start();
new Thread(new ReadT()).start();
```
如果这样去读的话，结果是
```
4294966962
```
这是因为long的读写都不是原子性的（二进制码发生串位）

### 可见性
可见性指的是如果一个线程修改一个变量的值，其他线程未必知道这个值发生变化
主要有以下的原因
* 缓存优化
* 指令重排
* 编辑器优化

### 有序性
有序性在并发的时候指的是指令的执行未必是按顺序的
比如
```java
public void writer()
{
    a=1;
    flag=true;
}
public void reader()
{
    if(flag)
    {
        int i=a+1;
        ...
    }
}
```
如果线程a调用writer，线程b调用reader可能实际执行起来就成了

writer先调用flag=true，然后还没执行a=1,就进入了reader里面的`int i=a+1`,导致完全不一样的结果

# java并行程序基础

## 线程
线程的所有状态都在Thread的State中进行定义

```java
public enum State
{
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED
}
```

* NEW 表示线程刚刚创建，还没有执行，**等到start方法调用的时候才开始执行**
* RUNNABLE 表示线程已经执行
* BLOCKED 表示线程进入临界区进入阻塞状态
* WAITING 表示无时间限制等待状态（等待针对的是notify方法）
* TIMED_WAITING 表示有时间限制等待状态
* TERMINATED 线程执行完毕表示结束

***  
从NEW出发后不能够再回到NEW状态，处于TERMINATED状态的也不能回到RUNNABLE状态
***

## 线程的创建
新建一个线程只要new一个线程对象然后使用start进行运行即可  
**不要使用run，run只是单独运行这一方法**

还可以使用实现Runnable方法的方式进行线程的创建（最好使用Runnable，**因为java的继承资源是有限的**），能用接口尽量使用接口的方式进行线程的创建

> 注意Thread有一个重要的构造方法
> ```java
> public Thread(Runnable target)
> ```
> 这也是Thread执行run的默认方式

## 线程中断

线程中断不是直接让线程退出，而是向线程发出一个通知，至于线程接到通知之后如何处理完全由目标线程自行决定（**实际上只是打上一个标记**）。**因为如果无条件退出可能会导致比较严重的问题**

线程可以使用interrupted方法进行中断，但是最好和中断处理代码配合使用  
如下  
```java
Thread thread1 = new Thread(new Runnable(){
            @Override
            public void run(){
                while(true)
                {
                    if(Thread.currentThread().isInterrupted())
                    {
                        System.out.println("Interrupted");
                        break;
                    }
                    Thread.yield();
                }
            }
        });
thread1.start();
try 
{

     Thread.sleep(2000);

} 
catch (InterruptedException e)
{

    e.printStackTrace();

}

thread1.interrupt();
```

使用sleep方法可以让当前线程休眠一段时间，**调用这个方法会抛出一个InterruptedException**，程序必须捕获并且处理这个异常

```java
try
{

Thread.sleep(2000);

}
catch(InterruptedException e)
{

System.out.println("Thread is sleeping");

}
```

## wait和notify

wait 和notify 这两个方法不是在Thread中的，而是在Object中的  
当在一个对象实例上使用wait方法之后，线程就会在这个对象上等待，直到其他线程在这个对象上调用notify


如果有多个线程调用wait，**则会进入一个等待队列**，如果有线程调用notify则会**随机选择一个队列中的线程**

还可以使用notifyAll会**激活等待队列中所有等待的线程**而不是随机选择一个


⚠️注意：wait/notify必须在对应的synchronized语句中

可以使用以下例子说明使用的方式
```java
public class Solution {
    final static Object object=new Object();
    public static class T1 extends Thread
    {
        @Override
        public void run()
        {
            synchronized (object) {
                System.out.println(System.currentTimeMillis() + ":T1 Starts");
                try {
                    System.out.println(System.currentTimeMillis()+":T1 waits");
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis()+":T1 ends");
            }
        }
    }
    public static class T2 extends Thread
    {
        @Override
        public void run()
        {
            synchronized (object)
            {
                System.out.println(System.currentTimeMillis() + ":T2 Starts");
                System.out.println(System.currentTimeMillis() + ":T2 notifies");
                object.notify();
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }
    }
    public static void main(String[]args)
    {
        Thread t1=new T1();
        Thread t2=new T2();
        t1.start();
        t2.start();
    }
}
```
可以发现不是执行notify后等待队列马上就可以执行，而是**需要等待当前线程释放锁之后**才可以执行

⚠️：wait和sleep的区别就是wait会释放目标的锁，而sleep不会释放任何资源

## join和yield

join表示进行等待，会阻塞当前线程直到目标线程执行完毕，也可以在其中传入一个最大等待时间
```java
public class Solution {
    public volatile static int i=0;
    public static class T1 extends Thread
    {
        @Override
        public void run() {
            for(i=0;i<10000;i++);

        }
    }
    public static void main(String[]args)
    {
        Thread t1=new T1();
        t1.start();
        try {
            t1.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(i);
    }
}
```

如果不使用join，可能输出会很小，但是使用join之后结果一定是10000

而yield会使当前线程让出CPU，但是这不代表该线程不执行了。**调用该方法后，该线程会重新参与资源的争夺**

## volatile
volatile关键字可以保证线程对于一个变量的修改能够让所有线程知道

## 守护线程
守护线程是系统的守护者，在后台完成一些系统性的服务，与之相对应的是用户线程  
⚠️：所有用户线程结束之后守护线程也会自然结束  

可以使用以下语句设定守护线程
```java
t.setDaemon(true);
```
**设定守护线程一定要在调用start方法之前**

## 优先级
可以使用`setPriority`设定线程执行的优先级，一般优先级高的会比较容易执行（**但是不是优先级高的一定优先执行**）

## synchronized
synchronized的作用是实现线程之间的同步，保证每一次只有一个线程进入同步块  
synchronized一般有三种用法
* 作用于加锁对象：对给定的对象加锁
* 作用于实例方法：对于当前实例加锁
* 作用于静态方法：相当于对当前类进行加锁

# JDK并发包

## 重入锁
### 重入锁的使用
可重入锁需要使用显式的加锁、释放的过程来进行控制

```java
public static ReentrantLock lock =new ReentrantLock();
lock.lock();
//同步的代码块
lock.unlock();
```
可重入锁的出现主要是解决这样一个问题：
要允许一个线程连续获得同一把锁，**否则将会产生死锁**，程序就会卡死在第二次申请锁的过程中

使用可重入锁可以指定限时等待的过程，让一个线程如果拿不到锁的话，超过时间自动放弃
```java
if(lock.tryLock(5,TimeUnit.SECONDS))
{
    System.out.println("got lock");
}
else
{
    System.out.println("get lock failed");
}
```

同时可以使用不带参数的方式
```java
lock.tryLock();
```
会直接尝试获取锁，如果获取失败则返回false
### 公平锁

ReentrantLock可以指定锁的申请的是否公平
```java
public ReentrantLock(boolean fair);
```
* 公平性：公平的锁会按照时间的先后顺序，保证先到的线程优先进行处理，这样不会发生饥饿的现象，只要排队就能够获得资源
* 非公平性：不能够保证线程的执行顺序和线程来的顺序的不同

公平锁开起来比较优美，但是**实现公平锁需要维护一个队列，因此性能比较低**

### Condition
Condition作用和wait/notify大致相同，但是wait/notify只能够和synchronized进行合作使用，而Condition是与重入锁相关的

创建一个Condition是使用锁中的`newCondition()`方法，这样可以创建一个**与这个锁绑定的condition**

* condition.await()==object.wait()
* condition.signal()==object.notify()
* condition.signalAll()==object.notifyAll()

调用这些方法也要在持有锁的情况下才能够进行调用

## 信号量
使用信号量可以控制多个线程对于一个资源的访问，而不是只允许一个线程对于资源进行访问

```java
public Semaphore(int permits);
public Semaphore(int permits,boolean fair);
```
permits表示该信号量的准入数
使用`acquire`,`release`控制获取一个准入的许可和释放一个准入的许可

```java
public class Solution {
    public static class Sema implements Runnable
    {
        final Semaphore semaphore=new Semaphore(5);
        @Override
        public void run()
        {
            try {
                semaphore.acquire();
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getId()+"done!");
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }

    public static void main(String[] args) {
        ExecutorService exec= Executors.newFixedThreadPool(20);
        final Sema sema=new Sema();
        for(int i=0;i<100;i++)
            exec.submit(sema);
    }
}
```

运行上面的程序可以发现是每2s打印5条语句，说明每次同时有五个线程获取了访问权限

一定要注意`acquire()`和`release()`的配套使用，要不然会导致可访问的权限数量越来越少

## 读写锁
可以使用`ReadWriteLock`来作为读写锁  
读写锁的出现主要是因为**如果使用可重入锁的话，如果有读操作和写操作，同时读操作会相互阻塞，这是不合理的，因为读操作不会对于数据进行修改**

如果系统中的读操作远远大于写操作的时候，读写锁的作用可以说是非常好的

读写锁可以使用以下的方式进行使用(大致使用方式和可重入锁的差不多)
```java
ReadWriteLock RWLock=new ReadWriteLock();
RWLock.readLock().lock();
RWLock.readLock().unlock();
RWLock.writeLock().lock();
RWLock.writeLock().unlock();
```

## CountDownLach
`CountDownLach`是一个倒计时器，可以对于线程进行阻塞，然后在倒计时完成之后允许线程继续执行
* 使用`countDown`将计数减一
* 使用`await`使线程进行等待

```java
public class Solution {
    static final CountDownLatch end=new CountDownLatch(10);
    static final CountDownDemo demo=new CountDownDemo();
    public static class CountDownDemo implements Runnable
    {


        @Override
        public void run()
        {
            try {
                Thread.sleep(2000);
                System.out.println("System Op Completed");
                end.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }

    public static void main(String[] args) {

        ExecutorService exec= Executors.newFixedThreadPool(10);
        for(int i=0;i<10;i++)
            exec.submit(demo);
        try {
            end.await();
            System.out.println("Fire");
            exec.shutdown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }


    }
}
```
## 线程池
### 线程池的概念
如果过多使用线程会浪费很多系统资源，如果对于每一个任务都创建一个线程，**可能创建线程的时间已经超过任务执行的时间**  
> 所以线程的数量必须得到控制

解决这个问题的方式就是线程池，即：使用时将线程从线程池中取出，使用完毕后放回线程池  

### 线程池的种类  
* `newFixedThreadPool`:固定线程数量的的线程池，如果有任务来，线程池中如果有空闲线程，则使用空闲线程执行，否则进入等待队列
有线程空闲的时候执行等待队列中的线程
* `newSingleThreadExecutor`：只有一个线程的线程池，若多余一个的线程提交，会进入等待队列中，如果线程空闲，则按照先入先出的方式顺序执行任务
* `newCachedThreadPool`:：一个根据实际情况调整线程池大小的小城池，如果有线程空闲，则使用线程池中空闲的线程，否则创建新的线程，执行完毕后进入线程池
* `newScheduledThreadPool`:给定时间进行执行的线程池
* `newSingleScheduledThreadPool`：给定时间进行执行的单一线程池
### 计划任务
`newScheduledThreadPool`主要有以下几种方法
* scheduleAtFixRate：
    ```java
    public ScheduledFuture<> scheduleAtFixRate(
        Runnable command，
        long initialDelay,
        long period，
        TimeUnit unit
    )；
    ```
    任务在初始延迟之后开始执行，后续任务按一定周期执行，**与上一个任务的结束时间无关**
* scheduleAtFixDelay:
    ```java
    public ScheduledFuture<> scheduleAtFixDelay(
        Runnable command，
        long initialDelay,
        long period，
        TimeUnit unit
    )；
    ```
    任务在初始延迟之后开始执行，**后续任务在前一个任务结束后的固定时间之后执行**

### 线程池的实现
所有线程池的种类的构造方法都是`ThreadPoolExecutor`的封装
```java
public ThreadPoolExecutor(
    int corePoolSize,
    int maxmumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedRejectedExecutionHandler handler
)
```
* corePoolSize：核心池线程数量
* maxmumPoolSize：最大线程数量
* keepAliveTime：线程数量超过核心池数量时，多余线程的存活时间
* unit：时间单位
* BlockQueue：阻塞队列
* ThreadFatory：线程工厂，一般使用默认的
* handler：拒绝策略

阻塞队列：
阻塞队列可以选择以下几种类型  
*  直接提交的队列：`SynchronousQueue`提交的任务不会被真实保存，总是会直接提交给线程池执行，一般需要配合比较大的maxmumPoolSize
*  有界的任务队列：`ArrayBlockingQueue`需要传入最大容量，是有最大容量的任务队列
*  无界的任务队列：`LinkedBlockingQueue`没有最大容量，线程数量永远不会超过corePoolSize
*  优先任务队列：有优先级的任务队列

拒绝策略：
jdk内置四种拒绝策略
* AbortPolicy：抛出异常
* CallerRunsPolicy：在调用者线程中运行被抛弃的线程
* DiscardOldestPolicy：丢弃最老的请求，再次提交
* DiscardPolicy：丢弃所有无法处理的任务

ThreadFactory：
线程池通过`ThreadFactory`进行接口的创建
```java
Thread newThread(Runnable r);
```
可以使用ThreadFactory对于线程进行一些统一的操作

### 线程数量的选择
一般线程池中的线程数量不能够太大也不能够太小，一般可以使用以下公式进行计算

$Nthreads=Ncpu*Ucpu*(1+W/C)$
* Ncpu是CPU的数量
* Ucpu是目标CPU的使用率
* W/C是等待时间与计算时间的比率

## 并发容器(待补完)
常用的并发容器主要有以下几种
* ConcurrentHashMap
* CopyOnWriteArrayList：在读多写少的情况下效率非常高
* ConcurrentLinkedQueue：高效的并发队列
* BlockingQueue：阻塞队列
* ConcurrentSkipListMap：跳表的实现

### ConcurrentHashMap
因为HashMap本身是线程不安全的，如果直接使用synchronized关键字等进行同步的话会导致性能比较低

# 锁优化
## 提高锁的性能的几种方式
### 减少持有时间
应该尽可能减少线程持有锁的时间，如果线程中的某一步操作不需要进行同步就不要加锁
```java
public synchronized void syncMethod()
{
    othercode1();
    mutextMethod();
    othercode2();
}
public void syncMethod()
{
    othercode1();
    synchronized(this){
    mutextMethod();
    }
    othercode2();
}
```
在这里显然下面的写法会比上面的写法好很多

### 减少锁的粒度
类似于1.7里面的ConcurrentHashMap会对于一个segment进行加锁而不是对于一整个数组进行加锁，明显提高了并发效率

### 使用读写分离锁
对于大多数情况下，特别是读多写少的情况下，应该尽量使用读写锁进行加锁

### 锁分离
锁分离是读写锁思想的延伸,比如对于一个队列进行操作，take()和put()一个发生在队首，一个发生在队尾，一般不会发生冲突，这时分别上锁会有更好的效率

### 锁粗化
对于一个锁的请求是需要时间的，如果对于一个锁不断进行请求或者释放会浪费很多时间，因此可以将多个请求合并为一个进行上锁
如
```java
for(int i=0;i<1000;i++)
{
    synchronized(lock)
    {
        //code
    }
}
```
应该被替换为
```java
synchronized(lock)
{
        for(int i=0;i<1000;i++)
        {
            //code
        }
}
```

## JDK的锁优化
### 锁偏向
锁偏向是一种针对加锁操作的优化手段  
核心思想是
> 如果线程获得了锁，那么锁就进入偏向模式。当线程再次请求锁的时候就不需要进行任何同步操作

这种优化方式对于几乎没有锁竞争的情况下效果比较好，如果竞争激烈的话效果则不佳  
可以使用虚拟机参数`-XX:UseBiasedLocking`开启偏向锁
### 轻量级锁
顾名思义，轻量级锁是相对于重量级锁而言的。使用轻量级锁时，不需要申请互斥量，仅仅将指针头中的部分字节CAS更新指向线程栈中的Lock Record，如果更新成功，则轻量级锁获取成功，记录锁状态为轻量级锁；否则，说明已经有线程获得了轻量级锁，目前发生了锁竞争（不适合继续使用轻量级锁），接下来膨胀为重量级锁。

### 自旋锁
锁膨胀之后，虚拟机为了避免线程被真实地挂起，虚拟机会尝试自旋锁。也就是让其循环等待，直到拿到锁资源

### 锁消除
java虚拟机在编译的过程中，通过对于上下文的扫描，去除不可能存在共享资源竞争的锁，通过锁消除可以节省无意义的请求锁的时间

## ThreadLocal
ThreadLocal是线程的一个局部变量
一般按照下面的方式进行使用
```java
ThreadLocal<Integer> t1 = new ThreadLocal<Integer>();
t1.get();
t1.set(3);
```
`ThreadLocal`底层是通过`ThreadLocalMap`实现的，`ThreadLocalMap`是Thread的内部成员，保证了是与每一个Thread相关的

ThreadLocal 常用于这样的情况：  
比如如果每一个线程都需要使用一个Random进行随机数的生成，如果这些线程共用一个Random对象的话，会导致这些线程对于Random的访问有并发控制，但是实际上随机数的生成并不需要这一步，而只是需要每一个线程都有一个本地的Random进行随机数的生成就可以了，这样效率可以得到显著的提升

## 无锁
无锁是一种乐观策略，即假定每一次访问都不会发生冲突，因此不需要在访问的时候进行上锁而**只需要在访问结束之后对于值进行检查即可**

### CAS
CAS有三个参数CAS(V,E,N)
只有V值等于E值的时候，才会将V值更新为N，否则什么都不做

### AtomicInteger
AtomicInteger核心是通过
```java
private volatile int value;
```
实现的，volatile保证了所有的线程读取的一致性

一般CAS()操作需要通过自旋锁的形式进行，如果不一致则进入自旋，直到能够将值进行设定再继续

### Unsafe类
CAS的实现大致如下
```java
public final boolean compareAndSet(int expect,int update)
{
    return unsafe.compareAndSwapInt(this,valueOffset,expect,update);
}
```
CAS实际上使用`Unsafe`类进行实现的，`Unsafe`类实际上是对于指针的控制  
但是不是任何情况都能够使用`Unsafe`类
```java
public static Unsafe getUnsafe(){
    Class cc = Reflection.getCallerClass();
    if(cc.getClassLoader()!=null)
    {
        throw new SecurityException("Unsafe");
    }
    return theUnsafe;
}
```
即在获得Unsafe对象的时候会检查调用`getUnsafe`方法的类，如果其ClassLoader不为null就会抛出异常（通过Bootstrap进行加载的）

## 并行模式和算法

### 单例模式