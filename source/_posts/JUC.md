---
title: juc
date: 2023/07/15
categories:
  - coding
tags:
  - juc
  - 多线程
  - 并发编程
abbrlink: 42310
---
# 1、java多线程基本概念
## 1.1、进程和线程

1. **进程**
- 程序由指令和数据组成，但这些指令要运行，数据要读写，就必须将指令加载至 CPU，数据加载至内存。在指令运行过程中还需要用到磁盘、网络等设备。进程就是用来加载指令、管理内存、管理 IO 的 
- 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。 
- 进程就可以视为程序的一个实例。大部分程序可以同时运行多个实例进程（例如记事本、画图、浏览器等），也有的程序只能启动一个实例进程（例如网易云音乐、360 安全卫士等） 

2. **线程**
- 一个进程之内可以分为一到多个线程。 
- 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给 CPU 执行 
- **Java 中，线程作为最小调度单位，进程作为资源分配的最小单位。 在 windows 中进程是不活动的，只是作为线程的容器**
## 1.2、并发和并行
并发：线程轮流使用CPU
并行：多核cpu下，多个核同时调度运行线程

## 1.3、多线程的应用
### 1.3.1、异步调用
比如在项目中，视频文件需要转换格式等操作比较费时，这时开一个新线程处理视频转换，避免阻塞主线程 
### 1.3.2、提升效率
充分利用多核 cpu 的优势，提高运行效率。想象下面的场景，执行 3 个计算，最后将计算结果汇总。
```java
计算 1 花费 10 ms
计算 2 花费 11 ms
计算 3 花费 9 ms
汇总需要 1 ms
```

- 如果是串行执行，那么总共花费的时间是 10 + 11 + 9 + 1 = 31ms 
- 但如果是四核 cpu，各个核心分别使用线程 1 执行计算 1，线程 2 执行计算 2，线程 3 执行计算 3，那么 3 个 线程是并行的，花费时间只取决于最长的那个线程运行的时间，即 11ms最后加上汇总时间只会花费 12ms 

> 需要在多核 cpu 才能提高效率，单核仍然时是轮流执行

# 2、java线程
## 2.1、线程的创建
### 2.1.1、直接使用Thread
```java
// 创建线程对象
Thread t = new Thread() {
    public void run() {
        // 要执行的任务
    }
};
// 启动线程
t.start();
```
### 2.1.2、使用 Runnable 配合 Thread 
```java
Runnable runnable = new Runnable() {
    public void run(){
        // 要执行的任务
    }
};
// 创建线程对象
Thread t = new Thread( runnable );
// 启动线程
t.start();
```
java8可用lambda精简
```java
// 创建任务对象
Runnable task2 = () -> log.debug("hello");

// 参数1 是任务对象; 参数2 是线程名字，推荐
Thread t2 = new Thread(task2, "t2");
t2.start();
```
### 2.1.3、FutureTask 配合 Thread 
FutureTask 能够接收 Callable 类型的参数，用来处理有返回结果的情况

```java
// 创建任务对象
FutureTask<Integer> task3 = new FutureTask<>(() -> {
    log.debug("hello");
    return 100;
});

// 参数1 是任务对象; 参数2 是线程名字，推荐
new Thread(task3, "t3").start();

// 主线程阻塞，同步等待 task 执行完毕的结果
Integer result = task3.get();
log.debug("结果是:{}", result);
```
## 2.2、查看进程线程
### 2.2.1、windows

1. tasklist 查看进程 
2. taskkill 杀死进程 
3. netstat -ano|findstr 8080 根据端口查看进程
### 2.2.2、linux

1. ps -fe 查看所有进程 
2. kill 杀死进程
3.  top -Hp <PID> 查看某个进程（PID）的所有线程 
4. netstat -nlp|grep 8080 根据端口查看进程
### 2.2.3、JDK

1. jps 命令查看所有 Java 进程
2.  jstack <PID> 查看某个 Java 进程（PID）的所有线程状态
3.  jconsole 来查看某个 Java 进程中线程的运行情况（图形界面）

## 2.3、线程运行原理

1. **线程创建**

每个线程启动后，虚拟机就会为其分配一块栈内存。 每个栈由多个栈帧（Frame）组成
，栈帧对应着每次方法调用所占内存

2. **上下文切换**

因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码 
> 线程的 cpu 时间片用完 
> 垃圾回收 
> 有更高优先级的线程需要运行 
> 线程自己调用了 sleep、yield、wait、join、park、synchronized、lock 等方法 

当 Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java 中对应的概念就是程序计数器（Program Counter Register），它的作用是记住下一条 jvm 指令的执行地址，是线程私有的 

##  2.4、start与run

- 直接调用 run 是在主线程中执行了 run，没有启动新的线程 
- 使用 start 是启动新的线程，通过新的线程间接执行 run 中的代码

## 2.5、sleep 与 yield 

1. **sleep**
- 1. 调用 sleep 会让当前线程从 _Running_进入 _Timed Waiting _状态（阻塞） 
- 2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException 
- 3. 睡眠结束后的线程未必会立刻得到执行 

2. **yield**
- 1. 调用 yield 会让当前线程从 _Running _进入 _Runnable_就绪状态，然后调度执行其它线程 
- 2. 具体的实现依赖于操作系统的任务调度器 

## 2.6、join
join：t1调用t2的join方法，会先执行t2，然后执行t1
如果调用的是无参join方法，则等待thread执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的时间
```java
static int r = 0;
public static void main(String[] args) throws InterruptedException {
    test1();
}

private static void test1() throws InterruptedException {
    log.debug("开始");
    Thread t1 = new Thread(() -> {
        log.debug("开始");
        sleep(1);
        log.debug("结束");
        r = 10;
    });
    t1.start();
    // t1.join();
    log.debug("结果为:{}", r);
    log.debug("结束");
}
```
> 如果不加t1.join()结果为0，加上以后结果为1


## 2.7、interrupt
### 2.7.1、打断阻塞状态的线程
sleep，wait，join 的线程 这几个方法都会让线程进入阻塞状态 ，打断 这几个状态 的线程, 会中断
```java
private static void test1() throws InterruptedException {
    Thread t1 = new Thread(()->{
        sleep(1);
    }, "t1");
    t1.start();
    sleep(0.5);
    t1.interrupt();
    log.debug(" 打断状态: {}", t1.isInterrupted());
}
```
输出
```java
java.lang.InterruptedException: sleep interrupted
     at java.lang.Thread.sleep(Native Method)
     at java.lang.Thread.sleep(Thread.java:340)
     at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
     at cn.itcast.n2.util.Sleeper.sleep(Sleeper.java:8)
     at cn.itcast.n4.TestInterrupt.lambda$test1$3(TestInterrupt.java:59)
     at java.lang.Thread.run(Thread.java:745)
21:18:10.374 [main] c.TestInterrupt - 打断状态: false
```
### 2.7.2、打断正常运行的线程
打断正常运行的线程, 不会清空打断运行（此时我们需要通过打断状态来中止）
```java
private static void test2() throws InterruptedException {
    Thread t2 = new Thread(()->{
        while(true) {
            Thread current = Thread.currentThread();
            boolean interrupted = current.isInterrupted();
            if(interrupted) {
                log.debug(" 打断状态: {}", interrupted);
                break;
            }
        }
    }, "t2");
    t2.start();
    sleep(0.5);
    t2.interrupt();
}
```
输出
```java
20:57:37.964 [t2] c.TestInterrupt - 打断状态: true 
```

## 2.8、主线程与守护线程
默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。有一种特殊的线程叫做守护线程，只要其它非守护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束。 

例
```java
log.debug("开始运行...");
Thread t1 = new Thread(() -> {
    log.debug("开始运行...");
    sleep(2);
    log.debug("运行结束...");
}, "daemon");
// 设置该线程为守护线程
t1.setDaemon(true);
t1.start();

sleep(1);
log.debug("运行结束...");
```
输出
```java
08:26:38.123 [main] c.TestDaemon - 开始运行... 
08:26:38.213 [daemon] c.TestDaemon - 开始运行... 
08:26:39.215 [main] c.TestDaemon - 运行结束...
```

> **注意 **
> - 垃圾回收器线程就是一种守护线程 
> - Tomcat 中的 Acceptor 和 Poller 线程都是守护线程，所以 Tomcat 接收到 shutdown 命令后，不会等待它们处理完当前请求 


## 2.9、线程状态
| 状态   | 说明 |
| --- | --- |
| NEW | 初始状态:线程被创建，但还没有调用start()方法 |
| RUNNABLE | 运行状态:Java线程将操作系统中的就绪和运行两种状态笼统的称作"运行" |
| BLOCKED | 阻塞状态:表示线程阻塞于锁 |
| WAITING | 等待状态:表示线程进入等待状态，进入该状态表示当前线程需要等待其他线程做出一些特定动作(通知或中断) |
| TIMEWAITING | 超时等待状态:该状态不同于WAITIND，它是可以在指定的时间自行返回的 |
| TERMINATED | 终止状态:表示当前线程已经执行完毕 |

![线程状态](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230930532.jpg)


# 3、共享模型之管程
管程（monitor），管理共享变量以及对其的操作过程，让这个类是线程安全的
## 3.1、monitor
Monitor 被翻译为**监视器**或**管程**
每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的Mark Word 中就被设置指向 Monitor 对象的指针

### 3.1.1、Monitor结构
**结构**：owner  entryList  waitSet

![monitor结构](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230930118.jpg)


### 3.1.2、Monitor原理
（1）刚开始monitor中owner为null 
（2）当某个线程t1执行同步方法synchronized时，会将owner置为该线程 
（3）t1持有锁过程中，t2，t3来了就会进入entryList中阻塞 
（4）t1执行完会唤醒entrylist中的某个线程（不公平）
（5）调用wait方法，会将此线程放入到wait set中，然后放弃锁。直到有其它线程调用notify（），才会重新进入entrylist中，重新争夺锁的拥有权

## 3.2、java对象结构

1. **对象头**

包括：对象头：Mark Word（标记字段）、Class Pointer（类型指针）,数组长度（如果是数组）

![java对象头](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230930015.jpg)


2. **实例数据**

实例数据：对象真正存储的有效信息，存放类的属性数据信息，包括父类的属性信息

3. **对齐填充**

对齐填充：由于虚拟机要求 对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐。
## 3.3、synchronized升级
### 3.3.1、偏向锁
> 使用场景：如果只有一个线程，就不需要每次的申请释放锁

只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有 
### 3.3.2、轻量级锁
> 使用场景：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化

![轻量级锁](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230930202.jpg)



1. 创建 锁记录（Lock Record）对象，内部存储锁记录地址和状态00，还有对象引用指向锁对象
2. 让锁记录中引用 指向锁对象，并尝试用 cas 替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录
3. 如果 cas 替换成功，对象头中存储了 锁记录地址和状态 00 ，表示由该线程给对象加锁
4. 如果 cas 失败，有两种情况  

（1）如果是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入锁膨胀过程 
（2）如果是自己执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数（取值为null）

5. 当退出 synchronized 代码块（解锁时）如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示重入计数减一
6. 当退出 synchronized 代码块（解锁时）锁记录的值不为 null，这时使用 cas 将 Mark Word 的值恢复给对象头 

（1）成功，则解锁成功 
（2）失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程
### 3.3.3、重量级锁
> 使用场景：如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁

![重量级锁](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230931871.jpg)


1. 当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁
2. 这时 Thread-1 加轻量级锁失败，进入锁膨胀流程 

（1）为 Object 对象申请 Monitor 锁，让 Object 指向重量级锁地址 
（2）然后自己进入 Monitor 的 EntryList阻塞队列

3. 当 Thread-0 退出同步块解锁时，使用 cas 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级解锁流程，即按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中阻塞线程（不公平）

> 调用wait方法，会将此线程放入到wait set中，然后放弃锁。直到有其它线程调用notify（），才会重新进入entrylist中，重新争夺锁的拥有权

### 3.3.4、自旋锁
重量级锁竞争的时候，还可以使用自旋(循环尝试获取重量级锁)来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞。 (进入阻塞再恢复,会发生上下文切换,比较耗费性能)

## 3.4、wait/notify
Owner 线程发现条件不满足，调用 wait 方法，即可进入 WaitSet 变为 WAITING 状态 。BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片 。BLOCKED 线程会在 Owner 线程释放锁时唤醒 。WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入EntryList 重新竞争

### 3.4.1、sleep和wait的区别？

1.  sleep 是 Thread 方法，而 wait 是 Object 的方法 
2.  sleep 不需要强制和 synchronized 配合使用，但 wait 需要和 synchronized 一起用 
3.  sleep 在睡眠的同时，不会释放对象锁的，但 wait 在等待的时候会释放对象锁 

## 3.5、park/unpark
它们是 LockSupport 类中的方法
```java
// 暂停当前线程
LockSupport.park(); 
// 恢复某个线程的运行
LockSupport.unpark(暂停线程对象)
```

### 3.5.1、与 Object 的 wait & notify 相比 

- wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必
- park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll是唤醒所有等待线程，就不那么【精确】 
- park & unpark 可以先 unpark，而 wait & notify 不能先 notify 

### 3.5.2、原理
每个线程都有自己的一个(C代码实现的) Parker 对象，由三部分组成 _counter ， _cond 和_mutex 

核心部分是counter，我们可以理解为一个标记位。
当调用park时会看counter是否为0，为0则进入阻塞队列。为1则继续运行并将counter置为0。
当调用unpark时，会将counter置为1，若之前的counter值为0，还唤醒阻塞的线程。
## 3.6、死锁
多个线程，比如A持有1资源，B持有2资源，A要获取2资源，B要获取1资源。但两个线程都不释放他们当前持有的线程，就会导致死锁
### 3.6.1、死锁的必要条件

1. 互斥条件：一个资源一次只能被一个进程使用
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得资源保持不放
3. 不剥夺条件：进程获得的资源，在未完全使用完之前，不能强行剥夺
4. 循环等待条件：若干进程之间形成一种头尾相接的环形等待资源关系
### 3.6.2、死锁的实现
```java
/**
 * 实现一个死锁
 * 如果把lock(target, owner);放到上面则不会死锁
 */
public class DeadLock {
    public static void main(String[] args) throws InterruptedException {
        final Object owner = new Object();
        final Object target = new Object();
        //开启一个新线程
        new Thread(() -> {
            try {
                lock(owner, target);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        //主线程
        lock(target, owner);
    }
    public static void lock(Object owner, Object target) throws InterruptedException {
        synchronized (owner) {
            Thread.sleep(1000);
            synchronized (target) {
                System.out.println("success");
            }
        }
    }
}
```
### 3.6.3、定位死锁
检测死锁可以使用 jconsole工具，或者使用 jps 定位进程 id，再用 jstack 定位死锁
### 3.6.4、哲学家就餐问题

![哲学家就餐问题](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202307230931868.jpg)


有五位哲学家，围坐在圆桌旁。 他们只做两件事，思考和吃饭，思考一会吃口饭，吃完饭后接着思考。 吃饭时要用两根筷子吃，桌上共有 5 根筷子，每位哲学家左右手边各有一根筷子。 如果筷子被身边的人拿着，自己就得等待 

1. 筷子类

```java
class Chopstick {
    String name;
    
    public Chopstick(String name) {
        this.name = name;
    }
    
    @Override
    public String toString() {
        return "筷子{" + name + '}';
    }
}
```

2. 哲学家类

```java
class Philosopher extends Thread {
    Chopstick left;
    Chopstick right;
    
    public Philosopher(String name, Chopstick left, Chopstick right) {
        super(name);
        this.left = left;
        this.right = right;
    }
    
    private void eat() {
        log.debug("eating...");
        Sleeper.sleep(1);
    }
    
    @Override
    public void run() {
        while (true) {
            // 获得左手筷子
            synchronized (left) {
                // 获得右手筷子
                synchronized (right) {
                    // 吃饭
                    eat();
                }
                // 放下右手筷子
            }
            // 放下左手筷子
        }
    }
}
```

3. 就餐

```java
Chopstick c1 = new Chopstick("1");
Chopstick c2 = new Chopstick("2");
Chopstick c3 = new Chopstick("3");
Chopstick c4 = new Chopstick("4");
Chopstick c5 = new Chopstick("5");

new Philosopher("苏格拉底", c1, c2).start();
new Philosopher("柏拉图", c2, c3).start();
new Philosopher("亚里士多德", c3, c4).start();
new Philosopher("赫拉克利特", c4, c5).start();
new Philosopher("阿基米德", c5, c1).start();
```
## 3.7、活锁
两个线程互相改变对方的结束条件导致谁也无法结束
> eg：共享变量count为10000, t1线程while count > 0, count-- ;t2线程while count < 20000, count++ .两个线程同时运行，这样count的值一直无法达到结束循环的条件。两个线程一直在执行


## 3.8、**ReentrantLock**
相对于 synchronized 它具备如下特点 

1. 可中断 
2. 可以设置超时时间 
3. 可以设置为公平锁 （默认不公平）
4. 支持多个条件变量 

与 synchronized 一样，都支持可重入 
# 4、共享模型之内存
## 4.1、java内存模型（jmm）
Java内存模型(即Java Memory Model，简称JMM) 。它是一个规范，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本
## 4.2、线程安全性的三个体现
**原子性**：原子性指的是一个或多个操作要么全部执行成功要么全部执行失败（一个操作CPU不可被中断）（Atomic、CAS算法、synchronized、Lock）
**可见性**：可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值（synchronized、volatile）
**有序性**：即程序执行的顺序按照代码的先后顺序执行（happens-before原则）
# 5、共享模型之无锁
## 5.1、CAS
### 5.1.1、CAS基本概念
CAS是所有原子类的底层原理，乐观锁主要采用CAS算法。
CAS，比较并交换，是JDK提供的非阻塞原子性操作，CAS的思想很简单：三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。
> 1. CAS利用了处理器的CMPXCHG指令，该指令操作的内存区域就会加锁，导致其他处理器不能同时访问它，保证原子性
> 2. CAS 必须借助 volatile 才能读取到共享变量的最新值来实现【比较并交换】的效果


### 5.1.2、CAS问题
多线程环境，线程1读取了数据A，然后线程2将数据A变为B再变为A，线程1进行cas操作时就会认为数据没有被修改过，其实数据是被修改过的。

**解决方案：**
使用AtomicStampReference 
AtomicStampReference在cas的基础上增加了一个标记stamp，使用这个标记可以用来觉察数据是否发生变化，给数据带上了一种实效性的检验

## 5.2、volatile
### 5.2.1、如何保证可见性和有序性

1. **保证可见性（缓存一致性协议MESI）**

首先cpu会根据共享变量是否带有Volatile字段，来决定是否使用**MESI协议**保证缓存一致性。
如果有Volatile，汇编层面会对变量加上Lock前缀，当一个线程修改变量的值后，会马上经过store、write等原子操作修改主内存的值（如果不加Lock前缀不会马上同步），为什么监听到修改会马上同步呢？cpu**总线嗅探机制**监听到这个变量被修改，就会把其他线程的变量副本由共享S置为无效I，当其他线程在使用变量副本时，发现其已经无效，就回去主内存中拿一个最新的值

> **M 修改** (Modified) 代表该缓存行中的内容被修改了，并且该缓存行只被缓存在该CPU中
**E 独享、互斥** (Exclusive) E代表该缓存行对应内存中的内容只被该CPU缓存.该缓存可以在任何其他CPU读取该缓存对应内存中的内容时变成S状态。或者本地处理器写该缓存就会变成M状态。
**S 共享** (Shared) 当多个线程都拿到了共享变量，此时为共享状态.当有一个CPU修改该缓存行对应的内存的内容时会使该缓存行变成 I 状态
**I 无效** (Invalid) 线程丢弃了自己工作内存中的变量，为无效状态


> 涉及到的指令
lock(锁定)：将一个变量标识为被一个线程独占状态
store(存储)：作用于工作内存的变量,将变量传输到主内存中
write(写入)：将store入主内存的变量,放入到主内存的变量中


2. **保证有序性（禁止指令重排优化）**

多线程环境下，有序性问题产生的主要原因就是执行重排优化，而Volatile的另一个作用就是禁止指令重排优化。具体是通过对Volatile修饰的变量增加内存屏障来完成的
内存屏障的主要工作原理为：通过在指令间插入一条内存屏障并禁止cpu对Volatile修饰的变量进行重排序

## 5.3、原子类
| 类型   | 具体类 |
| --- | --- |
| Atomic 基本类型原子类   | AtomicInteger AtomicLong AtomicBoolean |
| AtomicArray 数组类型原子类 | AtomicIntegerArray  AtomicLongArray AtomicReferenceArray |
| AtomicReference 引用类型原子类 | AtomicReference AtomicStampedReference AtomicMarkableReference |
| AtomicFieldUpdate 升级类型原子类 | AtomicIntegerFieldupdater AtomicLongFieldUpdater AtomicReferenceFieldUpdater |


# 6、共享模式之工具
## 6.1、线程池
### 6.1.1、**ThreadPoolExecutor**

1. **构造方法**

通过new ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler)自定义创建

> 1. corePoolSize：核心线程池的大小，如果核心线程池有空闲位置，新的任务就会被核心线程池新建一个线程执行，执行完毕后不会销毁线程，线程会进入缓存队列等待再次被运行。
> 2. maximunPoolSize：线程池能创建最大的线程数量。如果核心线程池和缓存队列都已经满了，新的任务进来就会创建救急线程来执行。但是数量不能超过maximunPoolSize，否侧会采取拒绝接受任务策略，我们下面会具体分析。
> 3. keepAliveTime：救急线程线程能够空闲的最长时间，超过时间，线程终止。这个参数默认只有在线程数量超过核心线程池大小时才会起作用。只要线程数量不超过核心线程大小，就不会起作用。
> 4. unit：空闲线程存活时间单位 创建一个新线程时使用的工厂，可以用来设定线程名、是否为daemon线程等等
> 5. workQueue：缓存队列，用来存放等待被执行的任务。
> 6. threadFactory 线程工厂
> 7. handler：拒绝策略
（1）abortPolicy：抛出异常（默认）
（2）discardPolicy：放弃本次任务
（3）discardoldestPolicy：放弃队列中最早的任务，本任务取代
（4）callerrunPolicy：让调用者运行任务


2. **工作原理**

如果当前线程池中正在执行的线程数目小于corePoolSize，则每来一个任务，就会创建一个线程去执行这个任务；
如果当前线程池中正在执行任务的的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程(救急线程)去执行这个任务；
如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止，直至线程池中的线程数目不大于corePoolSize；
如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理
### 6.1.2、Executors类中提供的工厂方法
根据上面的ThreadPoolExecutor这个构造方法，JDK Executors类中提供了众多工厂方法来创建各种用途的线程池

1. **newFixedThreadPool**

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

> 特点：
> - 核心线程数 == 最大线程数（没有救急线程被创建），因此也无需超时时间 
> - 阻塞队列是无界的，可以放任意数量的任务 
> 
评价：
> 适用于任务量已知，相对耗时的任务


2. **newCachedThreadPool**

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

> 特点 
> - 核心线程数是 0，最大线程数是 Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，意味着 
>    - 全部都是救急线程（60s 后可以回收）
>    - 救急线程可以无限创建 
> - 队列采用了 SynchronousQueue 实现特点是，它没有容量，没有线程来取是放不进去的（一手交钱、一手交货）
> 
评价：
> 整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完，空闲 1分钟后释放线程
> 适合任务数比较密集，但每个任务执行时间较短的情况


3. newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```
使用场景： 
希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。
任务执行完毕，这唯一的线程也不会被释放。 

### 6.1.3、创建多大的线程池

- 过小会导致程序不能充分地利用系统资源、容易导致饥饿 
- 过大会导致更多的线程上下文切换，影响性能

1. **CPU 密集型运算 **

通常采用 `cpu 核数 + 1` 能够实现最优的 CPU 利用率，+1 是保证当线程由于页缺失故障（操作系统）或其它原因 导致暂停时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费 

2. **I/O密集型**

CPU 不总是处于繁忙状态，例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程 RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。 
经验公式如下 ：
`线程数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间` 

## 6.2、锁
### 6.2.1、AQS

1. 基本概念

AbstractQueuedSynchronizer抽象的队列式同步器。是除了java自带的synchronized关键字之外的锁机制。这个类在java.util.concurrent.locks包。AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch...

2. 原理 

它维护了一个volatile int state（代表共享资源）和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列），核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并将共享资源设置为锁定状态，如果被请求的共享资源被占用，那么就将暂时获取不到锁的线程加入到队列中
AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）

3. 实现

自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：
> **isHeldExclusively()**：该线程是否正在独占资源。只有用到condition才需要去实现它。
**tryAcquire(int)**：独占方式。尝试获取资源，成功则返回true，失败则返回false。
**tryRelease(int)**：独占方式。尝试释放资源，成功则返回true，失败则返回false。
**tryAcquireShared(int)**：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
**tryReleaseShared(int)**：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。


以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的

### 6.2.2、ReentrantLock
**实现原理**

1. 首先，ReentrantLock实现Lock接口，这样他就向外提供了加锁，解锁，中断锁的基本功能
2. 它内部维护了一个sync同步器，继承了AQS。它加锁，解锁的实现其实就是调用这个同步器的方法
3. 加锁的时候用CAS尝试去修改state从0变为1，修改成功则把owner线程改成当前线程。修改失败说明已经有线程获取锁，判断当前持有锁的线程是否是该线程，是的话state+1，冲入。否则进入等待队列
4. 持有锁的线程释放时，找到队列中离 head 最近的一个 Node，unpark 恢复其运行，tryAcquire尝试获取锁。如果在默认的非公平状态下，这时如果又有新的线程获取锁，它仍有可能获取失败。如果是公平状态，该线程可以顺利拿到锁（新来的线程会添加到队列尾部）

**条件变量——Condition**
Condition 是一个多线程协调通信的工具类，作用类似于synchronized中的waitset。可以让某些线程一起等待某个条件（condition），只有满足条件时，线程才会被唤醒。两个重要的方法：await:把当前线程阻塞挂起 signal:唤醒阻塞的线程
### 6.2.3、**ReentrantReadWriteLock**
#### 4.2.3.1、ReentrantReadWriteLock基本概念
ReadLock和WriteLock是ReentrantReadWriteLock的两个内部类，Lock的上锁和释放锁都是通过一个AQS同步器sync来实现的。将 state 的 高 16 位和低 16 位拆开表示读写锁。其中高 16 位表示读锁，低 16 位表示写锁。读锁，允许共享；写锁，是独占锁。适合在读多写少的场景中使用
#### 4.2.3.2、锁获取过程

1. 获取读锁

如果写锁没有被另一个线程持有，则获取读锁并立即返回。     
 	如果写锁由另一个线程持有，则出于线程调度目的，当前线程将被禁用（unpark），并处于休眠状态，直	到获取读锁为止。

4. 获取写入锁

如果没有其他线程持有读锁或写锁，会直接返回，并将写锁计数设置为1。     *
如果当前线程持有写锁，则将写锁计数 +1，然后返回
如果锁正在被其他线程持有，则当前线程将被禁用，并处于休眠状态，直到获取读锁并将写锁计数设置为1。

#### 4.2.3.3、常见问题

1. **读锁和写锁的可重入性**

在加锁的时候，判断是否为当前线程，如果是当前线程，则直接累加计数。值得注意的是：读锁重入计数使用的 ThreadLocal 在线程中缓存计数，而写锁则直接用的 state 进行累加

2. **当前线程获取锁失败，被阻塞的后续操作是什么？**

获取失败，会放到 AQS 等待队列中，在队列中不断循环，监视前一个节点是否为 head ，是的话，会重新尝试获取锁

3. **锁降级是怎么降级的？**

在获取读锁的时候，如果当前线程持有写锁，是可以获取读锁的。这块就是指锁降级，比如线程 A 获取到了写锁，当线程 A 执行完毕时，它需要获取当前数据，假设不支持锁降级，就会导致 A 释放写锁，然后再次请求读锁。而在这中间是有可能被其他阻塞的线程获取到写锁的。从而导致线程 A 在一次执行过程中数据不一致（脏读）

## 6.3、工具
### 6.3.1、Semaphore

1. **概念**

Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源（Semaphore可以用于做流量控制，特别是公用资源有限的应用场景，比如数据库连接）
```java
public class DataSourcePool {
 
    private final CopyOnWriteArrayList<Connection> list;
    
    //用到了信号量
    private final Semaphore semaphore;
 
    public DataSourcePool(int size) throws SQLException {
        list = new CopyOnWriteArrayList<>();
        semaphore = new Semaphore(size);
        for (int i=0;i<size;i++){
            Connection connection = DriverManager.getConnection("url");
            list.add(connection);
        }
    }
    
    //使用同步方法获取
    public synchronized Connection getConnection() throws InterruptedException {
        //先将当前信号量-1，如果为0，将阻塞
        semaphore.acquire();
        return list.remove(0);
    }
 
    public synchronized void  close(Connection connection){
        //信号量+1
        semaphore.release();
        list.add(connection);
    }
}
```

2. **原理**

Semaphore的实现原理主要是通过内部类Sync来实现的，内部类Sync是AQS的子类，主要是通过重写AQS的共享式获取和释放同步状态方法来实现的

1、把初始令牌数量赋值给同步队列的state状态，state的值就代表当前所剩余的令牌数量。
2、 semaphore.acquire(); 当前线程会尝试去同步队列获取一个令牌，获取令牌的过程也就是使用原子的操作去修改同步队列的state ,获取一个令牌则修改为state=state-1。state<0,令牌数量不足，加入阻塞队列。>=0则获取成功
3、semaphore.release() ，释放令牌的过程也就是把同步状态的state修改为state=state+1的过程。释放令牌成功之后，同时会唤醒同步队列的所有阻塞节共享节点线程
### 6.3.2、CountdownLatch

1. **概念**

CountDownLatch允许一个或多个线程等待其他线程完成操作。await（）用来等待计数归0，countDown（）用来让计数减少一
```java
public static void main(String[] args) throws InterruptedException {
    CountDownLatch latch = new CountDownLatch(3);
    ExecutorService service = Executors.newFixedThreadPool(4);
    
    service.submit(() -> {
        log.debug("begin...");
        sleep(1);
        latch.countDown();
        log.debug("end...{}", latch.getCount());
    });
    
    service.submit(() -> {
        log.debug("begin...");
        sleep(1.5);
        latch.countDown();
        log.debug("end...{}", latch.getCount());
    });
    
    service.submit(() -> {
        log.debug("begin...");
        sleep(2);
        latch.countDown();
        log.debug("end...{}", latch.getCount());
    });
    
    service.submit(()->{
        try {
            log.debug("waiting...");
            latch.await();
            log.debug("wait end...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    
}
```
```java
18:52:25.831 c.TestCountDownLatch [pool-1-thread-3] - begin... 
18:52:25.831 c.TestCountDownLatch [pool-1-thread-1] - begin... 
18:52:25.831 c.TestCountDownLatch [pool-1-thread-2] - begin... 
18:52:25.831 c.TestCountDownLatch [pool-1-thread-4] - waiting... 
18:52:26.835 c.TestCountDownLatch [pool-1-thread-1] - end...2 
18:52:27.335 c.TestCountDownLatch [pool-1-thread-2] - end...1 
18:52:27.835 c.TestCountDownLatch [pool-1-thread-3] - end...0 
18:52:27.835 c.TestCountDownLatch [pool-1-thread-4] - wait end...
```

2. **原理**

CountDownLatch的实现原理主要是通过内部类Sync来实现的，内部类Sync是AQS的子类，主要是通过重写AQS的共享式获取和释放同步状态方法来实现的
1、初始化CountDownLatch实际就是设置了AQS的state为计数的值
2、调用CountDownLatch的countDown方法时实际就是调用AQS的relase方法，每调用一次就自减一次state值
3、调用await方法实际就调用AQS的共享式获取同步状态state，当AQS的state值为0时，await方法才会执行成功，否则就会一直处于死循环中不断重试

3. **和join的区别？**

CountDownLatch：控制力度更细，比如可以在子线程执行一部分后coutdown，就不一定要等到线程执行完成

### 6.3.3、CyclicBarrier

1. **概念**

CyclicBarrier可以理解为一个循环同步屏障，定义一个同步屏障之后，当一组线程都全部达到同步屏障之前都会被阻塞，直到最后一个线程达到了同步屏障之后才会被打开，其他线程才可继续执行
实现王者荣耀10个人都加载完才开始游戏
```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        int count = 10;
        CyclicBarrier cb = new CyclicBarrier(count, new Runnable() {
            @Override
            public void run() {
                System.out.println("全部加载完毕");
            }
        });
        ExecutorService executorService = Executors.newFixedThreadPool(count);
        for (int x = 0; x < count; x++) {
            executorService.execute(new Worker(cb));
        }
    }
}

class Worker extends Thread {
    CyclicBarrier cyclicBarrier;
    public Worker(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 已加载完");
        try {
            cyclicBarrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}
```
```java
pool-1-thread-1 已加载完
pool-1-thread-5 已加载完
pool-1-thread-4 已加载完
pool-1-thread-3 已加载完
pool-1-thread-6 已加载完
pool-1-thread-2 已加载完
pool-1-thread-7 已加载完
pool-1-thread-8 已加载完
pool-1-thread-9 已加载完
pool-1-thread-10 已加载完
全部加载完毕

```

2. **原理**

1、创建CyclicBarrier时定义了CyclicBarrier对象需要达到的线程数count
2、每当一个线程执行了await方法时，需要先通过ReentrantLock进行加锁操作，然后对count进行自减操作，操作成功则判断当前count是否为0；
3、如果当前count不为0则调用Condition的await方法使当前线程进入等待状态；
4、如果当前count为0则表示同步屏障已经完全，此时调用Condition的signalAll方法唤醒之前所有等待的线程，并开启循环的下一次同步屏障功能；
5、唤醒其他线程之后，其他线程继续执行剩余的逻辑。

## 6.4、线程安全集合类
### 6.4.1、概述
						  
线程安全实现类有三类：

1. 遗留的线程安全集合如 Hashtable ， Vector 
2. 使用 Collections 装饰的线程安全集合（调用所有方法时加synchronized修饰）

>    - Collections.synchronizedCollection 
>    - Collections.synchronizedList 
>    - Collections.synchronizedMap 
>    - Collections.synchronizedSet 
>    - Collections.synchronizedNavigableMap 
>    - Collections.synchronizedNavigableSet
>    - Collections.synchronizedSortedMap 
>    - Collections.synchronizedSortedSet 

4. JUC下的安全集合: Blocking、CopyOnWrite、Concurrent 

> - Blocking 大部分实现基于锁，并提供用来阻塞的方法 (Lock)
> - CopyOnWrite 之类容器修改开销相对较重 (修改时拷贝)
> - Concurrent 类型的容器 （内部很多操作使用cas优化）
>    - 内部很多操作使用 cas 优化，一般可以提供较高吞吐量 
>    - 弱一致性 
>       - 遍历时弱一致性，例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续进行遍历，这时内容是旧的 
>       - 求大小弱一致性，size 操作未必是 100% 准确 
>       - 读取弱一致性 

> 遍历时如果发生了修改，对于非安全容器来讲，使用 **fail-fast **机制也就是让遍历立刻失败，抛出ConcurrentModifificationException，不再继续遍历

### 6.4.2、concurrentHashMap
**JDK1.8前**
ConcurrentHashMap使用分段锁技术，数据结构：ReentrantLock+segement+hashEntry。一个segement中包含一个hashentry数组（hashentry结构类似hashmap：数组+链表）。

元素查询：使用二次hash，第一次定位到segement，第二次hash定位到元素所在链表的头部

锁：segement继承了reentrantLock，锁定操作的segement，其它segement不受影响，并发度为segement个数


> ConcurrentHashMap 与HashMap和Hashtable 最大的不同在于：put和 get 两次Hash到达指定的HashEntry，第一次hash到达Segment,第二次到达Segment里面的Entry,然后在遍历entry链表


**JDK1.8**

在JDK8中，ConcurrentHashMap的底层数据结构与HashMap一样，也是采用“数组+链表+红黑树”的形式。同时，它又采用锁定头节点的方式降低了锁粒度，以较低的性能代价实现了线程安全

1. 初始化数组或头节点时，ConcurrentHashMap并没有加锁，而是CAS的方式进行原子替换（原子操作，基于Unsafe类的原子操作API）。 
2. 插入数据时会进行加锁处理，但锁定的不是整个数组，而是槽中的头节点。所以，ConcurrentHashMap中锁的粒度是槽，而不是整个数组，并发的性能很好。 
3. 扩容时会进行加锁处理，锁定的仍然是头节点。并且，支持多个线程同时对数组扩容，提高并发能力。每个线程需先以CAS操作抢任务，争抢一段连续槽位的数据转移权。抢到任务后，该线程会锁定槽内的头节点，然后将链表或树中的数据迁移到新的数组里。 
4. 查找数据时并不会加锁，所以性能很好。另外，在扩容的过程中，依然可以支持查找操作。如果某个槽还未进行迁移，则直接可以从旧数组里找到数据。如果某个槽已经迁移完毕，但是整个扩容还没结束，则扩容线程会创建一个转发节点，在这个节点里面记录的是新的 ConcurrentHashMap 的引用，从新数组中找到目标数据。



### 6.4.3、BlockingQueue
主要的两个实现ArrayBlockingQueue 和 LinkedBlockingQueue 

1. 区别

（1）内部实现：ArrayBlockingQueue 使用数组；LinkedBlockingQueue 使用单链表
（2）锁的个数：ArrayBlockingQueue只有一把锁（最多只允许一个线程，生产者或消费者二选一）；	LinkedBlockingQueue 有两把锁：takeLock、putLock（可以允许两个线程同时执行，一个生产者，一个消费者）
（3）支持公平锁：ArrayBlockingQueue 支持；LinkedBlockingQueue 不支持，因为有两把锁，没法实现

### 6.4.4、ConcurrentLinkedQueue 
ConcurrentLinkedQueue 的设计与 LinkedBlockingQueue 非常像，也是 两把【锁】，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）

### 6.4.5、CopyOnWriteArrayList

1. 首先CopyOnWriteArrayList的内部也是通过数组来实现的，在向CopyOnWriteArrayList添加元素时，会复制一个新的数组，写操作在新的数组上进行，读操作在原数组上进行
2. 写数据时会加ReentLocak锁，防止并发写入丢失数据的问题
3. 写操作结束后会把原数组指向新数组
4. CopyOnWriteArrayList允许在写操作时来读取数据，大大提高了读的性能，因此适合读多写少的场景。但是CopyOnWriteArrayList比较占用内存，同时可能督导的数据不是实时最新的数据，所以不适合实时性要求很高的场景


