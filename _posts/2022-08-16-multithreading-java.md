---
layout: post
title: '线程生命周期与Java中多线程实现与执行顺序'
subtitle: ''
date: 2022-08-16
author: Dave
cover: ''
tags: Java
---

# 线程生命周期与Java中多线程实现与执行顺序

## 线程生命周期

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/commonQuery/线程状态转移.png)

> 线程操作
>
> **线程创建：**Thread.sleep(long millis)方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。
>
> **线程睡眠：**Thread.sleep(long millis)方法（**注意,sleep()是不会释放持有的锁**），使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。
>
> **线程等待：**Object类的wait()方法（**wait()会释放持有的锁**），导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。唤醒线程后，就转为就绪（Runnable）状态。
>
> **线程让步：**Thread.yield() 方法（**注意,yield()是不会释放持有的锁**），暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。
>
> **线程加入：**join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。
>
> **线程I/O：**线程执行IO操作，需等待相关资源而进入了阻塞状态。
>
> **线程唤醒：**Object类中的notify()方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程，选择是任意性的，并在对实现做出决定时发生。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。



### 新建状态

线程初始状态，使用new关键字创建线程后，为新建状态；新建状态下仅由 JVM 分配内存，初始化成员变量。

### 就绪状态

线程对象调用start()方法后，线程为就绪状态；JVM为其创建方法调用栈、程序计数器，等待调度运行。

线程其余状态也可回退到就绪状态，

1） 运行状态下的线程，时间片用完，调用yield()方法进行让步；

2）sleep()、join()、I/O操作导致的阻塞状态下的线程，sleep()结束、join()结束、I/O操作完成，回到就绪状态；

3）同步阻塞状态（锁池状态），获取到锁，回到就绪状态；

### 运行状态

就绪状态的线程由系统调度获取CPU后，开始执行run()方法，线程为运行状态。若CPU时间片用完或调用yield()执行线程让步 或 再次系统调度，运行状态回到就绪状态。

### 阻塞状态

阻塞状态是线程放弃CPU使用，运行被暂停，线程被阻塞，等待重回就绪状态的中间状态。

1. **等待阻塞：**运行的线程执行wait()方法（**wait()会释放持有的锁**），JVM会把该线程放入等待池中。线程需要等待被显式地唤醒，否则会处于无限期等待的状态。
2. **同步阻塞：**运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池（同步阻塞状态）。
3. **其他阻塞：**运行的线程执行sleep()或join()方法，或发出I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。（**注意,sleep()是不会释放持有的锁**）。

### 死亡状态

线程的run()方法执行结束，或主线程的main()方法执行结束，可认定线程正常终止，死亡状态。

> 线程一旦终止了，就不能复生。

线程抛出一个未捕获的Exception或Error，线程异常终止；直接调用该线程的stop()方法，也可直接终止该线程，但容易导致死锁，不推荐。

## Java多线程实现方式

> Thread类 构造方法
>
> ```java
> public Thread() {
>  init(null, null, "Thread-" + nextThreadNum(), 0);
> }
> // Runnable构造器
> public Thread(Runnable target) {
>  init(null, target, "Thread-" + nextThreadNum(), 0);
> }
> // Runnable构造器
> Thread(Runnable target, AccessControlContext acc) {
>  init(null, target, "Thread-" + nextThreadNum(), 0, acc, false);
> }
> 
> public Thread(ThreadGroup group, Runnable target) {
>  init(group, target, "Thread-" + nextThreadNum(), 0);
> }
> // 线程name 默认 Thread-{数字} 可指定
> // this.getName() 继承Thread类可使用this获取当前线程对象
> // Thread.currentThread().getName() 实现Runnable接口需使用Thread.currentThread()获取当前线程对象
> public Thread(String name) {
>  init(null, null, name, 0);
> }
> 
> public Thread(ThreadGroup group, String name) {
>  init(group, null, name, 0);
> }
> // Runnable构造器
> public Thread(Runnable target, String name) {
>  init(null, target, name, 0);
> }
> 
> public Thread(ThreadGroup group, Runnable target, String name) {
>  init(group, target, name, 0);
> }
> 
> public Thread(ThreadGroup group, Runnable target, String name,
>            long stackSize) {
>  init(group, target, name, stackSize);
> }
> ```




### 1.显式创建：继承Thread类

> 显式创建线程；继承Thread类，重写run方法（无返回值）

**Thread 类实现了 Runable 接口。**

```java
public class Thread implements Runnable {
  //......
}
```

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

缺点：Java中继承为单继承，且run方法无返回值

实例：

```java
public class MyThreadTest {
    public static void main(String[] args) {
        System.out.println("======MyThreadTest======");
        MyThread myThread1 = new MyThread("线程1");
        MyThread myThread2 = new MyThread("线程2");
        MyThread myThread3 = new MyThread("线程3");
        myThread1.start();
        myThread2.start();
        myThread3.start();
    }
}
class MyThread extends Thread {
    private String myThreadName;
    public MyThread(String name) {
        // this获取线程对象，设置线程名
        this.setName(name);
        this.myThreadName = "myThreadName:" + name;
    }
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(this.getName() + "正在执行......" + i);
            //System.out.println(this.myThreadName + "正在执行......" + i);
        }
    }
}

/*
======MyThreadTest======
线程1正在执行......0
线程1正在执行......1
线程3正在执行......0
线程3正在执行......1
线程2正在执行......0
线程2正在执行......1
线程3正在执行......2
线程1正在执行......2
线程3正在执行......3
线程2正在执行......2
线程3正在执行......4
线程1正在执行......3
线程1正在执行......4
线程2正在执行......3
线程2正在执行......4
*/
// 线程间 执行乱序 线程内 顺序
```

### 2.显式创建：实现Runnable接口/Callable接口

> 显式创建线程；实现Runnable接口，重写run方法（无返回值）
>
> 与继承Thread类的异同：
>
> 本质上都是通过实现Runnable接口，重写run方法；但是，继承Thread类通过类的单继承的方式而实现Runnable接口（接口可多继承）更灵活。

缺点：run方法无返回值

实例：

```java
public class MyThreadTest {
    public static void main(String[] args) {
        System.out.println("======MyThreadTest======");
        Runnable myThread1 = new MyThread("线程1");
        Runnable myThread2 = new MyThread("线程2");
        Runnable myThread3 = new MyThread("线程3");
        new Thread(myThread1).start();
        new Thread(myThread2).start();
        new Thread(myThread3).start();
    }
}
class MyThread implements Runnable {
    private String myThreadName;
    public MyThread(String name) {
        this.myThreadName = "myThreadName:" + name;
    }
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + "正在执行......" + i);
            //System.out.println(this.myThreadName + "正在执行......" + i);
        }
    }
}
```



> 显式创建线程；实现Callable接口，重写call方法（有返回值且可以抛出异常）
>
> Thread构造方法不能直接通过Callable构造，但J.U.C中FutureTask实现了Runnable且接收Callable构造。

```java
// RunnableFuture 继承 Runnable, Future 接口
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}

// FutureTask 实现 RunnableFuture
public class FutureTask<V> implements RunnableFuture<V> {
  //......
		public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
  //......
}
```

实例：

```java
public class MyThreadTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");
        Callable<Integer> myThread1 = new MyThread("线程1");
        Callable<Integer> myThread2 = new MyThread("线程2");
        Callable<Integer> myThread3 = new MyThread("线程3");

        FutureTask<Integer> futureTask1 = new FutureTask<Integer>(myThread1);
        FutureTask<Integer> futureTask2 = new FutureTask<Integer>(myThread2);
        FutureTask<Integer> futureTask3 = new FutureTask<Integer>(myThread3);

        new Thread(futureTask1).start();
        new Thread(futureTask2).start();
        new Thread(futureTask3).start();
        System.out.println("======call return======");
        System.out.println("futureTask1:" + futureTask1.get());
        System.out.println("futureTask2:" + futureTask2.get());
        System.out.println("futureTask3:" + futureTask3.get());
    }
}
class MyThread implements Callable<Integer> {
    private String name;
    public MyThread(String name) {
        this.name = name;
    }
    @Override
    public Integer call() {
        try {
            int sum = 0;
            for (int i = 0; i < 5; i++) {
                System.out.println(name + "正在执行......" + i);
                sum += i;
            }
            return sum;
        } catch (Exception e) {
            e.printStackTrace();
            return -1;
        }
    }
}
/*
======MyThreadTest======
线程1正在执行......0
线程2正在执行......0
线程2正在执行......1
线程1正在执行......1
线程1正在执行......2
线程1正在执行......3
线程1正在执行......4
线程2正在执行......2
线程2正在执行......3
线程3正在执行......0
线程3正在执行......1
线程3正在执行......2
线程3正在执行......3
线程3正在执行......4
======call return======
futureTask1:10
线程2正在执行......4
futureTask2:10
futureTask3:10
*/
```

### 3.线程池

> 线程池主要需要理解和使用 J.U.C 包下 JDK 1.5后引入的[Executor框架](https://www.cnblogs.com/MOBIN/p/5436482.html)
>
>
> 原理：内部使用线程池机制，控制线程的启动、执行和关闭，简化并发编程；
>
> 优点：任务的提交和执行解耦；更易管理、效率更好（用线程池实现，节约开销）；
>
> 有助于避免this逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象用Executor在构造器中。
>
> 流程： 
>
> 1. 提交一个Callable对象给ExecutorService（如最常用的线程池ThreadPoolExecutor），得到一个Future对象；
>
> 2. 调用Future对象的get方法等待执行结果；
>
> 提交方式：execute() 与 submit()
>
> - execute只能提交Runnable类型的任务，无返回值。submit既可以提交Runnable类型的任务，也可以提交Callable类型的任务，会有一个类型为Future的返回值，但当任务类型为Runnable时，返回值为null。
> - execute在执行任务时，如果遇到异常会直接抛出，而submit不会直接抛出，只有在使用Future的get方法获取返回值时，才会抛出异常。
>

Executor基于ThreadPoolExecutor实现了部分内置线程池

> **newCachedThreadPool**：创建可缓存的线程池，如果线程池中的线程在60秒未被使用就将被移除，在执行新的任务时，当线程池中有之前创建的可用线程就重用可用线程，否则就新建一条线程。
>
> **newFixedThreadPool**：创建可重用且固定线程数的线程池，如果线程池中的所有线程都处于活动状态，此时再提交任务就在队列中等待，直到有可用线程；如果线程池中的某个线程由于异常而结束时，线程池就会再补充一条新线程。
>
> **newSingleThreadExecutor**：创建一个单线程的Executor，如果该线程因为异常而结束就新建一条线程来继续执行后续的任务。
>
> ......(其余见源码)

#### 线程池重要参数

> 这篇写的蛮不错[阿里巴巴Java开发手册推荐线程池的创建方式](https://www.jianshu.com/p/94852bd1a283)

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) //后两个参数为可选参数
```

> **corePoolSize**：核心线程数，如果运行的线程少于corePoolSize，则创建新线程来执行新任务，即使线程池中的其他线程是空闲的
> **maximumPoolSiz**：最大线程数，可允许创建的线程数，corePoolSize和maximumPoolSize设置的边界自动调整池大小：
> **corePoolSize  < 运行的线程数 < ** **maximumPoolSize**：仅当队列满时才创建新线程
> **corePoolSize = 运行的线程数 = ** **maximumPoolSize**：创建固定大小的线程池
> **keepAliveTime**：如果线程数多于corePoolSize,则这些多余的线程的空闲时间超过keepAliveTime时将被终止
> **unit**：keepAliveTime参数的时间单位
> **workQueue**：保存任务的阻塞队列，与线程池的大小有关：
>
> 1. 当运行的线程数少于corePoolSize时，在有新任务时直接创建新线程来执行任务而无需再进队列；
> 2. 当运行的线程数等于或多于corePoolSize，在有新任务添加时则选加入队列，不直接创建线程；
> 3. 当队列满时，而仍未超过最大线程数时，有新任务就创建新线程；
>
> **threadFactory**：使用ThreadFactory创建新线程，默认使用defaultThreadFactory创建线程
> **handle**：定义处理被拒绝任务的策略，默认使用ThreadPoolExecutor.AbortPolicy,任务被拒绝时将抛出RejectExecutorException

ThreadPoolExecutor执行规则：

![](https://img-blog.csdnimg.cn/20200531162025694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMjAwMTYz,size_16,color_FFFFFF,t_70#pic_center)

> 阻塞队列
>
> ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
> LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
> PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
> DelayQueue： 一个使用优先级队列实现的无界阻塞队列。
> SynchronousQueue： 一个不存储元素的阻塞队列。
> LinkedTransferQueue： 一个由链表结构组成的无界阻塞队列。
> LinkedBlockingDeque： 一个由链表结构组成的双向阻塞队列。
>
> 拒绝策略
>
> 1.  ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出RejectedExecutionException异常。 (默认)
> 2.  ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
> 3.  ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务。（重复此过程）
> 4.  ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务。
>
> 提交任务
>
> - Callable
>   - 有返回值，可抛出异常；
>   - 通过submit函数提交，返回Future对象；
>   - Future对象通过get获取执行结果；
> - Runnable
>   - 只执行，无法返回，执行过程无法抛异常；
>   - 通过execute提交；
>
> 线程池顺序：**corePool -> workQueue -> maxPool**
>
> ​		1）当运行的线程数少于corePoolSize时，在有新任务时直接创建新线程来执行任务而无需再进队列；
>
> ​		2）当运行的线程数等于或多于corePoolSize，workQueue队列未满时，有新任务添加则选加入队列，不直接创建线程；
>
> ​		3）当队列满时，而运行的线程数小于maxPoolSize，在有新任务时就创建新线程（非核心线程），创建失败则拒绝任务；
>
> ​		4）队列满，且运行的线程数等于或大于maxPoolSize，拒绝策略；
>
> 现象：后进入线程池的任务有可能先开始执行，3）中的新任务 会比 当时workQueue队列 中的线程先执行；
>
> - 部分场景涉及到线程池顺序优化：**corePool->maxPool->workQueue**
>
>   线程操作涉及大量读写（IO密集），CPU未被充分利用，若workQueue参数设置偏大，而又有大量请求打进队列迟迟得不到执行，此时可优化线程池顺序，先进行3），当运行的线程数等于或大于maxPoolSize再进入workQueue队列。



#### 实例：线程池执行Runnable任务

```java
public class MyThreadTest {
    public static void main(String[] args) {
        System.out.println("======MyThreadTest======");
        ExecutorService executorService = Executors.newCachedThreadPool();
//      ExecutorService executorService = Executors.newFixedThreadPool(5);
//      ExecutorService executorService = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 5; i++) {
            executorService.execute(new MyThread("线程" + i));
            System.out.println("MyThread：" + i );
        }
        executorService.shutdown();
    }
}

class MyThread implements Runnable {
    private String myThreadName;
    public MyThread(String name) {
        this.myThreadName = "myThreadName:" + name;
    }
    @Override
    public void run() {
        System.out.println(this.myThreadName + "正在执行......");
    }
}
```

#### 实例：线程池执行Callable任务

```java
public class MyThreadTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");
        ExecutorService executorService = Executors.newCachedThreadPool();
        //ExecutorService executorService = Executors.newFixedThreadPool(5);
        //ExecutorService executorService = Executors.newSingleThreadExecutor();
        List<Future<Integer>> resultList = new ArrayList<>();

        // 创建任务并执行
        for (int i = 0; i < 5; i++) {
            //使用ExecutorService执行Callable类型的任务，并将结果保存在future变量中
            Future<Integer> future = executorService.submit(new MyThread("线程" + i));
            //将任务执行结果存储到List中
            resultList.add(future);
        }
        // 遍历任务的结果
        for (Future<Integer> fs : resultList) {
            try {
                // Future返回如果没有完成，则一直循环等待，直到Future返回完成
                while (!fs.isDone()) {
                    ;
                }
                System.out.println("======call return======");
                // 打印各个线程（任务）执行的结果
                System.out.println(fs.get());
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                //启动一次顺序关闭，执行以前提交的任务，但不接受新任务
                executorService.shutdown();
            }
        }
    }
}

class MyThread implements Callable<Integer> {
    private String name;
    public MyThread(String name) {
        this.name = name;
    }
    @Override
    public Integer call() {
        try {
            System.out.println(name + "正在执行......");
            return 123;
        } catch (Exception e) {
            e.printStackTrace();
            return -1;
        }
    }
}
/*
======MyThreadTest======
线程0正在执行......
线程1正在执行......
线程2正在执行......
线程3正在执行......
线程4正在执行......
======call return======
123
======call return======
123
======call return======
123
======call return======
123
======call return======
123
*/
```

#### 实例：自定义线程池执行Runnable任务

```java
public class MyThreadTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");
        //创建等待队列
        BlockingQueue<Runnable> blockQueue = new ArrayBlockingQueue<>(20);
        //创建线程池，池中保存的线程数为3，允许的最大线程数为5,
        ThreadPoolExecutor pool = new ThreadPoolExecutor(3, 5, 50, TimeUnit.MILLISECONDS, blockQueue);
        //创建5个任务
        Runnable myThread1 = new MyThread("线程1");
        Runnable myThread2 = new MyThread("线程2");
        Runnable myThread3 = new MyThread("线程3");
        Runnable myThread4 = new MyThread("线程4");
        Runnable myThread5 = new MyThread("线程5");
        //每个任务会在一个线程上执行
        pool.execute(myThread1);
        pool.execute(myThread2);
        pool.execute(myThread3);
        pool.execute(myThread4);
        pool.execute(myThread5);
        //关闭线程池
        pool.shutdown();
    }
}
class MyThread implements Runnable {
    private String myThreadName;
    public MyThread(String name) {
        this.myThreadName = "myThreadName:" + name;
    }
    @Override
    public void run() {
        System.out.println(this.myThreadName + "正在执行......");
    }
}
```

### 4.SpringBoot中的线程池

> SpringBoot中使用线程池
>
> 1. 创建线程池配置类（类型为 java.util.concurrent.TaskExecutor 或其子类的 bean），一般使用 Spring 提供的 ThreadPoolTaskExecutor 类；
> 2. 线程池配置类或SpringBoot入口类中添加注解`@EnableAsync`；
> 3. 在由Spring管理的对象的方法上使用`@Async`注解显式定义多线程任务，最好直接指明线程池；



> 阿里巴巴Java开发手册
> 【强制】创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。
> 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。
> 【强制】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

实例：

线程池配置类

```java
@Slf4j
@Configuration
public class ThreadPoolConfiguration {

    /**
     * 核心线程数（默认线程数）
     */
    @Value("${piccolo.threadPool.corePoolSize:10}")
    private Integer corePoolSize;
    /**
     * 最大线程数
     */
    @Value("${piccolo.threadPool.maxPoolSize:50}")
    private Integer maxPoolSize;
    /**
     * 允许线程空闲时间（单位：默认为秒）
     */
    @Value("${piccolo.threadPool.keepAliveTime:30}")
    private Integer keepAliveTime;
    /**
     * 缓冲队列大小
     */
    @Value("${piccolo.threadPool.queueCapacity:50000}")
    private Integer queueCapacity;
    /**
     * 线程前缀
     */
    @Value("${piccolo.threadPool.threadNamePrefix:common-threadPool-}")
    private String threadNamePrefix;

    @Bean("commonTaskExecutor")
    public ThreadPoolTaskExecutor commonTaskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(queueCapacity);
        executor.setKeepAliveSeconds(keepAliveTime);
        executor.setThreadNamePrefix(threadNamePrefix);
        executor.setTaskDecorator(new MdcTaskDecorator());
        // 线程池对拒绝任务的处理策略
        executor.setRejectedExecutionHandler((runnable, threadPoolExecutor) -> {
            log.warn("线程数超过线程池限制,进行拒绝执行处理逻辑,线程池信息:{}", threadPoolExecutor.toString());
            if (!threadPoolExecutor.isShutdown()) {
                runnable.run();
            }
        });
        // 初始化
        executor.initialize();
        return executor;
    }

}
```

接口注解

```java
//-----------------------接口类--------------------------
public interface UserService{
  	/**
  	 * 执行异步任务
  	 */
  	void saveRecord();
} 

//-----------------------接口实现类--------------------------
@Async("commonTaskExecutor")
@Override
public void saveRecord(String oaAccount, String oaName, Integer operationType, Long spaceId, Long contentId) {
	// ......
}
```

`@EnableAsync`注解添加（以SpringBoot启动类为例）

```java
@Slf4j
@SpringBootApplication(scanBasePackages = "com.piccolo")
@EnableDiscoveryClient
@EnableFeignClients
@EnableAsync
@MapperScan({"com.piccolo.dao"})
//.......
public class PiccoloApplication {
    public static void main(String[] args) {
        SpringApplication.run(PiccoloApplication.class, args);
        log.info("----------piccolo-server start success----------");
    }
}
```



## Java多线程的顺序执行

首先，思考一下多线程的运行状态，有以下特点：

1）线程无依赖的情况下，线程先启动（start），不能保证其 run() 方法先执行；

2）由线程的运行状态可见，线程经阻塞后重回就绪；

因此，对于线程顺序执行问题，主要思路：

1）变量监测：线程间存在执行依赖，通过监测变量进行顺序同步，比如设置锁、信号量监测、条件变量监测等；

2）线程阻塞：按顺序阻塞，按顺序执行；

3）单线程线程池：线程顺序入队顺序执行

这部分参考了[让线程按顺序执行8种方法](https://www.cnblogs.com/wenjunwei/p/10573289.html)

### 1.线程阻塞：join() 方法

#### 实例：主线程内顺序启动并join()线程

```java
public class MyThreadTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");
        //创建3个线程
        Runnable myThread1 = new MyThread("线程1");
        Thread thread1 = new Thread(myThread1);
        Runnable myThread2 = new MyThread("线程2");
        Thread thread2 = new Thread(myThread2);
        Runnable myThread3 = new MyThread("线程3");
        Thread thread3 = new Thread(myThread3);
      
        thread1.start();
        thread1.join();
        thread2.start();
        thread2.join();
        thread3.start();
        thread3.join();
    }
}
class MyThread implements Runnable {
    private String myThreadName;
    public MyThread(String name) {
        this.myThreadName = "myThreadName:" + name;
    }
    public MyThread(String name,Thread former) throws InterruptedException {
        former.join();
        this.myThreadName = "myThreadName:" + name;
    }
    @Override
    public void run() {

        System.out.println(this.myThreadName + "正在执行......");
    }
}
/*
======MyThreadTest======
myThreadName:线程1正在执行......
myThreadName:线程2正在执行......
myThreadName:线程3正在执行......
*/
```

#### 实例：创建线程时join()

```java
public class MyThreadTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");
        //创建3个线程
        Runnable myThread1 = new MyThread("线程1");
        Thread thread1 = new Thread(myThread1);
        Runnable myThread2 = new MyThread("线程2", thread1);
        Thread thread2 = new Thread(myThread2);
        Runnable myThread3 = new MyThread("线程3", thread2);
        Thread thread3 = new Thread(myThread3);

        thread1.start();
        thread2.start();
        thread3.start();
    }
}
class MyThread implements Runnable {
    private String myThreadName;
    public MyThread(String name) {
        this.myThreadName = "myThreadName:" + name;
    }
    public MyThread(String name,Thread former) throws InterruptedException {
        former.join();
        this.myThreadName = "myThreadName:" + name;
    }
    @Override
    public void run() {

        System.out.println(this.myThreadName + "正在执行......");
    }
}
/*
======MyThreadTest======
myThreadName:线程1正在执行......
myThreadName:线程2正在执行......
myThreadName:线程3正在执行......
*/
```



### 2.线程阻塞：wait() 方法

线程的等待阻塞状态

```java
public class MyThreadTest {
    private static Object myLock1 = new Object();
    private static Object myLock2 = new Object();

    /**
     * 为什么要加这两个标识状态?
     * 如果没有状态标识，当t1已经运行完了t2才运行，t2在等待t1唤醒导致t2永远处于等待状态
     */
    private static Boolean t1Run = false;
    private static Boolean t2Run = false;

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");
        final Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (myLock1) {
                    System.out.println("线程1正在执行...");
                    t1Run = true;
                    myLock1.notify();
                }
            }
        });

        final Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (myLock1) {
                    try {
                        if (!t1Run) {
                            System.out.println("线程2等待中...");
                            myLock1.wait();
                        }
                        synchronized (myLock2) {
                            System.out.println("线程2正在执行...");
                            t2Run = true;
                            myLock2.notify();
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });

        Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (myLock2) {
                    try {
                        if (!t2Run) {
                            System.out.println("线程3等待中...");
                            myLock2.wait();
                        }
                        System.out.println("线程3正在执行...");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        
        thread2.start();
        thread1.start();
        thread3.start();
    }
}
/* 2 1 3输出顺序下有多种情况，因线程进入顺序有变化。
======MyThreadTest======
线程2等待中...
线程1正在执行...
线程2正在执行...
线程3正在执行...

======MyThreadTest======
线程2等待中...
线程3等待中...
线程1正在执行...
线程2正在执行...
线程3正在执行...

======MyThreadTest======
线程2等待中...
线程1正在执行...
线程3等待中...
线程2正在执行...
线程3正在执行...


*/
```

### 3.线程阻塞：FutureTask

> FutureTask调用get()方法会阻塞获取返回值

```java
public class MyThreadTest {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        System.out.println("======MyThreadTest======");
        // FutureTask可以阻塞获取返回值
        FutureTask<String> task1 = new FutureTask<>(new MyCallable());
        Thread thread1 = new Thread(task1, "线程1");
        FutureTask<String> task2 = new FutureTask<>(new MyCallable());
        Thread thread2 = new Thread(task2, "线程2");
        FutureTask<String> task3 = new FutureTask<>(new MyCallable());
        Thread thread3 = new Thread(task3, "线程3");
        thread1.start();
        task1.get();
        thread2.start();
        task2.get();
        thread3.start();
        task3.get();
    }
}
class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println(Thread.currentThread().getName() + " 正在执行...");
        return Thread.currentThread().getName() + " : 返回值";
    }
}
```


### 4.JDK8特性：CompletableFuture

> 在JDK8中, 新增加了一个包含50个方法左右的类: CompletableFuture，结合了Future的优点，提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，提供了函数式编程的能力，可以通过回调的方式处理计算结果，并且提供了转换和组合CompletableFuture的方法。
> supplyAsync表示创建带返回值的异步任务的，相当于ExecutorService submit(Callable<T> task) 方法，runAsync表示创建无返回值的异步任务，相当于ExecutorService submit(Runnable task)方法，这两方法的效果跟submit是一样的，
> ```java
> // JDK8新增加的异步执行能力，thenXXXX(param)形式，支持函数式编程，使得顺序执行更加方便。
> // 1.单任务顺序
> // 1.1 thenApply系列 可获取前一个任务结果，且有返回结果
> 		// thenApply 前置子线程执行成功才执行，后续使用前置子线程执行，可获取前一个任务结果，且有返回结果
>     public <U> CompletableFuture<U> thenApply(
>         Function<? super T,? extends U> fn) {
>         return uniApplyStage(null, fn);
>     }
> 		// thenApplyAsync 一个入参，前置子线程执行成功，后续使用守护线程执行，有返回结果
>     public <U> CompletableFuture<U> thenApplyAsync(
>         Function<? super T,? extends U> fn) {
>         return uniApplyStage(asyncPool, fn);
>     }
> 		// thenApplyAsync 两个入参，前置子线程执行成功，后续使用指定executor的自定义线程执行，有返回结果
>     public <U> CompletableFuture<U> thenApplyAsync(
>         Function<? super T,? extends U> fn, Executor executor) {
>         return uniApplyStage(screenExecutor(executor), fn);
>     }
> // 1.2 thenAccept系列 可获取前一个任务结果，无返回结果
> 		// thenAccept 前置子线程运行成功才执行，后续使用前置的子线程执行，可获取前一个任务结果，但无返回结果
>     public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
>         return uniAcceptStage(null, action);
>     }
> 		// thenAcceptAsync 一个入参，前置子线程执行成功，后续使用守护线程执行，无返回结果
>     public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
>         return uniAcceptStage(asyncPool, action);
>     }
> 		// thenAcceptAsync 两个入参，前置子线程执行成功，后续使用指定executor的自定义线程执行，无返回结果
>     public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,
>                                                    Executor executor) {
>         return uniAcceptStage(screenExecutor(executor), action);
>     }
> // 1.3 thenRun系列 无法获取前一个任务结果，无返回结果
> 		// thenRun 前置子线程执行成功才执行，后续使用前置的子线程执行，无法获取前一个任务结果，且无返回结果
>     public CompletableFuture<Void> thenRun(Runnable action) {
>         return uniRunStage(null, action);
>     }
> 		// thenRunAsync 一个入参，前置子线程执行成功，后续使用守护线程执行，无法获取前置返回的结果，无返回结果
>     public CompletableFuture<Void> thenRunAsync(Runnable action) {
>         return uniRunStage(asyncPool, action);
>     }
> 		// thenRunAsync 两个入参，前置执行成功，后续使用指定executor的自定义线程执行，无法获取前置返回的结果，无返回结果
>     public CompletableFuture<Void> thenRunAsync(Runnable action,
>                                                 Executor executor) {
>         return uniRunStage(screenExecutor(executor), action);
>     }
> 
> // 2.多任务组合处理
> // 2.1 将两个CompletableFuture组合起来处理，只有两个任务都正常完成时，进行下阶段任务
> 		// thenCombine 将两个任务的执行结果作为所提供函数的参数，该方法有返回值
>     public <U,V> CompletableFuture<V> thenCombine(
>         CompletionStage<? extends U> other,
>         BiFunction<? super T,? super U,? extends V> fn) {
>         return biApplyStage(null, other, fn);
>     }
> 
> 		// thenAcceptBoth 将两个任务的执行结果作为方法入参，无返回值
>     public <U> CompletableFuture<Void> thenAcceptBoth(
>         CompletionStage<? extends U> other,
>         BiConsumer<? super T, ? super U> action) {
>         return biAcceptStage(null, other, action);
>     }
> 		// runAfterBoth 没有入参，也没有返回值
>     public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other,
>                                                 Runnable action) {
>         return biRunStage(null, other, action);
>     }
> // 2.2 将两个CompletableFuture组合起来处理，当有一个任务正常完成时，进行下阶段任务
>   	// applyToEither 将已经完成任务的执行结果作为所提供函数的参数，该方法有返回值
>     public <U> CompletableFuture<U> applyToEither(
>         CompletionStage<? extends T> other, Function<? super T, U> fn) {
>         return orApplyStage(null, other, fn);
>     }
> 		// acceptEither 将已经完成任务的执行结果作为方法入参，无返回值
>     public CompletableFuture<Void> acceptEither(
>         CompletionStage<? extends T> other, Consumer<? super T> action) {
>         return orAcceptStage(null, other, action);
>     } 
> 		// runAfterEither 没有入参，也没有返回值
>     public CompletableFuture<Void> runAfterEither(CompletionStage<?> other,
>                                                   Runnable action) {
>         return orRunStage(null, other, action);
>     }
> ```
>

实例：thenRun()


```java
public class MyThreadTest {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        System.out.println("======MyThreadTest======");

        Thread thread1 = new Thread(new MyRunnable(), "线程1");
        Thread thread2 = new Thread(new MyRunnable(), "线程2");
        Thread thread3 = new Thread(new MyRunnable(), "线程3");
        CompletableFuture.runAsync(thread1::start)
                .thenRun(thread2::start)
                .thenRun(thread3::start);
    }
}
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 正在执行...");
    }
}
```




### 5.变量监测：Condition(条件变量)方法

**Condition（条件变量）**：通常与一个锁关联。需要在多个Contidion中共享一个锁时，可以传递一个Lock/RLock实例给构造方法，否则它将自己生成一个RLock实例。

- Condition中**await()**方法类似于Object类中的wait()方法。
- Condition中**await(long time,TimeUnit unit)**方法类似于Object类中的wait(long time)方法。
- Condition中**signal()**方法类似于Object类中的notify()方法。
- Condition中**signalAll()**方法类似于Object类中的notifyAll()方法。

**应用场景**：Condition是一个多线程间协调通信的工具类，使得某个，或者某些线程一起等待某个条件（Condition）,只有当该条件具备( signal 或者 signalAll方法被带调用)时 ，这些等待线程才会被唤醒，从而重新争夺锁。

```java
public class MyThreadTest {
    private static Lock lock = new ReentrantLock();
    private static Condition condition1 = lock.newCondition();
    private static Condition condition2 = lock.newCondition();

    /**
     * 为什么要加这两个标识状态?
     * 如果没有状态标识，当t1已经运行完了t2才运行，t2在等待t1唤醒导致t2永远处于等待状态
     */
    private static Boolean t1Run = false;
    private static Boolean t2Run = false;

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");

        final Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                lock.lock();
                System.out.println("线程1正在执行...");
                t1Run = true;
                condition1.signal();
                lock.unlock();
            }
        });

        final Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                lock.lock();
                try {
                    if (!t1Run) {
                        System.out.println("线程2等待中...");
                        condition1.await();
                    }
                    System.out.println("线程2正在执行...");
                    t2Run = true;
                    condition2.signal();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                lock.unlock();
            }
        });

        Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                lock.lock();
                try {
                    if (!t2Run) {
                        System.out.println("线程3等待中...");
                        condition2.await();
                    }
                    System.out.println("线程3正在执行...");
                    lock.unlock();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        thread2.start();
        thread1.start();
        thread3.start();
    }
}

/* 2 1 3输出顺序下有多种情况，因线程进入顺序有变化。
======MyThreadTest======
线程2等待中...
线程1正在执行...
线程3等待中...
线程2正在执行...
线程3正在执行...
*/
```



### 6.变量监测：CountDownLatch(倒计数)方法

**CountDownLatch**:位于java.util.concurrent包下，利用它可以实现类似计数器的功能。

**应用场景**:比如有一个任务C，它要等待其他任务A,B执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

```java
public class MyThreadTest {
    /**
     * 用于判断线程一是否执行，倒计时设置为1，执行后减1
     */
    private static CountDownLatch c1 = new CountDownLatch(1);

    /**
     * 用于判断线程二是否执行，倒计时设置为1，执行后减1
     */
    private static CountDownLatch c2 = new CountDownLatch(1);

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");


        final Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("线程1正在执行...");
                //对c1倒计时-1
                c1.countDown();
            }
        });

        final Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //等待c1倒计时，计时为0则往下运行
                    c1.await();
                    System.out.println("线程2正在执行...");
                    //对c2倒计时-1
                    c2.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //等待c2倒计时，计时为0则往下运行
                    c2.await();
                    System.out.println("线程3正在执行...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        thread2.start();
        thread1.start();
        thread3.start();
    }
}

/*
======MyThreadTest======
线程1正在执行...
线程2正在执行...
线程3正在执行...
*/
```



### 7.变量监测：CyclicBarrier(回环栅栏)方法

**CyclicBarrier(回环栅栏)**:通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

**应用场景**:公司组织春游,等待所有的员工到达集合地点才能出发，每个人到达后进入barrier状态。都到达后，唤起大家一起出发去旅行。

```java
public class MyThreadTest {
    static CyclicBarrier barrier1 = new CyclicBarrier(2);
    static CyclicBarrier barrier2 = new CyclicBarrier(2);

    public static void main(String[] args) {
        System.out.println("======MyThreadTest======");
        final Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("线程1正在执行...");
                    //放开栅栏1
                    barrier1.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        });

        final Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //放开栅栏1
                    barrier1.await();
                    System.out.println("线程2正在执行...");
                    //放开栅栏2
                    barrier2.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        });

        final Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //放开栅栏2
                    barrier2.await();
                    System.out.println("线程3正在执行...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        });

        thread2.start();
        thread1.start();
        thread3.start();
    }
}
```



### 8.变量监测：监测信号方法

#### Semaphore(信号量)

Semaphore 通常我们叫它信号量， 可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。

通常用于那些资源有明确访问数量限制的场景，常用于限流 。

比如：数据库连接池，同时进行连接的线程有数量限制，连接不能超过一定的数量，当连接达到了限制数量后，后面的线程只能排队等前面的线程释放了数据库连接才能获得数据库连接。

比如：停车场场景，车位数量有限，同时只能容纳多少台车，车位满了之后只有等里面的车离开停车场外面的车才可以进入。

```java
// Semaphore常用方法
acquire()  //获取一个令牌，在获取到令牌、或者被其他线程调用中断之前线程一直处于阻塞状态。
acquire(int permits)  //获取一个令牌，在获取到令牌、或者被其他线程调用中断、或超时之前线程一直处于阻塞状态。  
acquireUninterruptibly() //获取一个令牌，在获取到令牌之前线程一直处于阻塞状态（忽略中断）。  
tryAcquire()//尝试获得令牌，返回获取令牌成功或失败，不阻塞线程。
tryAcquire(long timeout, TimeUnit unit)//尝试获得令牌，在超时时间内循环尝试获取，直到尝试获取成功或超时返回，不阻塞线程。
release()//释放一个令牌，唤醒一个获取令牌不成功的阻塞线程。
hasQueuedThreads()//等待队列里是否还存在等待线程。
getQueueLength()//获取等待队列里阻塞的线程数。
drainPermits()//清空令牌把可用令牌数置为0，返回清空令牌的数量。
availablePermits()//返回可用的令牌数量。
```

```java
public class MyThreadTest {
    private static Semaphore semaphore1 = new Semaphore(1);
    private static Semaphore semaphore2 = new Semaphore(1);
    private static Semaphore semaphore3 = new Semaphore(1);

    public static void main(String[] args) throws InterruptedException {
        System.out.println("======MyThreadTest======");
        final Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    semaphore1.acquire();
                    System.out.println("线程1正在执行...");
                    semaphore2.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        final Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    semaphore2.acquire();
                    System.out.println("线程2正在执行...");
                    semaphore3.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    semaphore3.acquire();
                    thread2.join();
                    semaphore1.release();
                    System.out.println("线程3正在执行...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        // 先获取2 3 令牌，使2 3 阻塞，让 1 首先运行
        semaphore2.acquire();
        semaphore3.acquire();
        thread2.start();
        thread1.start();
        thread3.start();
    }
}
```

#### 其余信号形式

实例：顺序List

```java
public class MyThreadTest {
    private static List<Integer> semaphoreList;

    public static void main(String[] args) throws InterruptedException {
        System.out.println("======MyThreadTest======");
        semaphoreList = new ArrayList<Integer>() {{
            add(1);
            add(2);
            add(3);
        }};
        final Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    while (semaphoreList.get(0) != 1) {
                        System.out.println("线程1等待中...");
                        Thread.sleep(100);
                    }
                    System.out.println("线程1正在执行...");
                    semaphoreList.remove(0);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        final Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    while (semaphoreList.get(0) != 2) {
                        System.out.println("线程2等待中...");
                        Thread.sleep(100);
                    }
                    System.out.println("线程2正在执行...");
                    semaphoreList.remove(0);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    while (semaphoreList.get(0) != 3) {
                        System.out.println("线程3等待中...");
                        Thread.sleep(100);
                    }
                    System.out.println("线程3正在执行...");
                    semaphoreList.remove(0);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        thread2.start();
        thread1.start();
        thread3.start();
    }
}
/*
======MyThreadTest======
线程2等待中...
线程1正在执行...
线程3等待中...
线程3等待中...
线程2正在执行...
线程3正在执行...
*/
```

### 9.线程池

线程池仅保留一个线程

- 单线程化线程池(newSingleThreadExecutor)
- 可控最大并发数线程池(newFixedThreadPool)
- 可回收缓存线程池(newCachedThreadPool)
- 支持定时与周期性任务的线程池(newScheduledThreadPool)

实例：单线程化线程池(newSingleThreadExecutor)

```java
public class MyThreadTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("======MyThreadTest======");
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        //创建3个任务
        Runnable myThread1 = new MyThread("线程1");
        Runnable myThread2 = new MyThread("线程2");
        Runnable myThread3 = new MyThread("线程3");

        //每个任务会在一个线程上执行
        executorService.execute(myThread1);
        executorService.execute(myThread2);
        executorService.execute(myThread3);
        //关闭线程池
        executorService.shutdown();
    }
}

class MyThread implements Runnable {
    private String myThreadName;

    public MyThread(String name) {
        this.myThreadName = "myThreadName:" + name;
    }

    public MyThread(String name, Thread former) throws InterruptedException {
        former.join();
        this.myThreadName = "myThreadName:" + name;
    }

    @Override
    public void run() {

        System.out.println(this.myThreadName + "正在执行......");
    }
}
/*
======MyThreadTest======
myThreadName:线程1正在执行......
myThreadName:线程2正在执行......
myThreadName:线程3正在执行......
*/
```



