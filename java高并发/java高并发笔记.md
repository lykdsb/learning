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

---

如果有多个线程调用wait，**则会进入一个等待队列**，如果有线程调用notify则会**随机选择一个队列中的线程**

---
还可以使用notifyAll会**激活等待队列中所有等待的线程**而不是随机选择一个

---

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


