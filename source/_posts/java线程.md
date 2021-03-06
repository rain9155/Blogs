---
title: java学习总结之线程
tags: 
- 线程
categories: java
date: 2019-07-19 12:25:50
---


## 前言

在java中，线程非常重要，我们要分清楚进程和线程的区别：进程是操作系统中**资源分配**的基本单位，进程是指一个内存中运行的应用程序，每个进程都拥有自己的一块独立的内存空间，进程之间的资源不共享；而线程是**CPU调度**的最小单元，一个进程可以有多个线程，线程之间的堆空间是共享的，但栈空间是独立的，java程序的进程至少包含主线程和后台线程(垃圾回收线程)。了解这些知识后，来看下文有关线程的知识。

<!--more-->

## 一、并发和并行

我们先来看一下概念：

* 并行：指两个或多个事件在**同一时刻点**发生
* 并发：指两个或多个事件在**同一时间段内发生**

对于单核CPU的计算机来说，它是不能并行的处理多个任务，它的每一时刻只能有一个程序执行时间片（时间片是指CPU分配给各个程序的运行时间），故在微观上这些程序只是**分时交替的运行**，所以在宏观看来在一段时间内有多个程序在同时运行，看起来像是并行运行。

对于多核CPU的计算机来说，它就可以并行的处理多个任务，可以做到多个程序在同一时刻同时运行。

同理对线程也一样，但系统只有一个CPU时，线程会以某种顺序执行，我们把这种情况称为线程调度，所以从宏观角度上看线程是并行运行的，但是从微观角度来看，却是串行运行，即一个线程一个线程的运行。

>一般来说，JVM的进程和线程都是与操作系统的进程和线程**一 一对应**的，这样做的好处是可以使操作系统来调度进程和线程，进程和线程调度是操作系统的核心模块，它的实现是非常复杂的，特别是考虑到多核的情况，因此，就完全没有必要在JVM中再提供一个进程和线程调度机制。

## 二、线程的创建与启动

有3种方式使用线程。

### 方式1：继承Thread类

定义一个类继承java.lang.Thread类，重写Thread类中的run方法，如下：

```java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}

//使用线程
public static void main(String[] args) {
    Thread thread = new MyThread();
     thread.start();
}
```

### 方式2：实现Runnable接口

#### 2.1：定义一个类实现Runnable接口

实现 Runnable只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用，如下：

```java
public class MyRunnable implements Runnable {
    public void run() {
        // ...
    }
}

//使用线程
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

#### 2.2、使用匿名内部类

这种方式只适用于这个线程只使用一次的情况，如下：

```java
public class MyRunnable implements Runnable {

//使用线程
public static void main(String[] args) {
    new Thread(new Runnable(){
        public void run(){
            // ...
        }
    }).start();
}
```

### 方式3：实现Callable接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装，所以在创建Thread时，要把FutureTask 传进去，如下：

```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}

//使用线程
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

### 继承与实现的区别

1、继承方式：

（1）java中类是单继承的，如果继承了Thread，该类就不能有其他父类了，但是可以实现多个接口

（2）从操作上分析，继承方式更简单，获取线程名字也简单

2、实现方式：

（1）java中类可以实现多接口，此时该类还可以继承其他类，并且还可以实现其他接口

（2）从操作上分析，实现方式稍复杂，获取线程名字也比较复杂，得通过Thread.currentThread来获取当前线程得引用

综上所述，实现接口会更好一些。

## 三、线程的中断与终止

### 1、interrupt()、isInterrupted()、interrupted()的作用

中断就是线程的一个标识位，它表示一个运行中的线程是否被其他线程调用了中断操作，其他线程可以通过调用线程的interrupt()方法对其进行中断操作，线程可以通过调用isInterrupted()方法判断是否被中断，线程也可以通过调用Thread的interrupted()静态方法对当前线程的中断标识位进行复位。

大家不要认为调用了线程的interrupt()方法，该线程就会停止，它只是做了一个标志位，如下：

```java
public class InterruptThread extends Thread{
    @Override
    public void run() {
        //一个死循环
        while (true){
            System.out.println("InterruptThread正在执行");
        }
    }
}

public static void main(String[] args) throws InterruptedException {
    InterruptThread interruptThread = new InterruptThread();
    interruptThread.start();
    interruptThread.interrupt();//调用线程的interrupt()
    System.out.println("interruptThread是否被中断，interrupt  = " + interruptThread.isInterrupted());//此时isInterrupted()方法返回true
}

输出结果：
interruptThread是否被中断，interrupt  = true
InterruptThread正在执行
InterruptThread正在执行
InterruptThread正在执行
//...
```

可以看到当你调用了线程的interrupt()方法后，此时调用isInterrupted()方法会返回true，但是该线程还是会继续执行下去。所以怎么样才能终止一个线程的运行呢？

### 2、终止线程的运行

一个线程正常执行完run方法之后会自动结束，如果在运行过程中发生异常也会提前结束；所以利用这两种情况，我们还可以通过以下三种种方式安全的终止运行中的线程：

#### 2.1、利用中断标志位

前面讲到的中断操作就可以用来取消线程任务，如下：

```java
public class InterruptThread extends Thread{
    @Override
    public void run() {
        while (!isInterrupted()){//利用中断标记位
            System.out.println("InterruptThread正在执行");
        }
    }
}
```

当不需要运行InterruptThread线程时，通过调用InterruptThread.interrupt()使得isInterrupted()返回true，就可以让线程退出循环，正常执行完毕之后自动结束。

#### 2.2、利用一个boolean变量

利用一个boolean变量和上述方法同理，如下：

```java
public class InterruptThread extends Thread{
    
    private volatile boolean isCancel;

    @Override
    public void run() {
        while (!isCancel){//利用boolean变量
            System.out.println("InterruptThread正在执行");
        }
    }

    public void cancel(){
        isCancel = true;
    }
}

```

当不需要运行InterruptThread线程时，通过调用InterruptThread.cancel()使isCancel等于true，就可以让线程退出循环，正常执行完毕之后自动结束，这里要注意boolean变量要用volatile修饰保证内存的可见性。

#### 2.3、响应InterruptedException

通过调用一个线程的 interrupt() 来中断该线程时，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程，例如当你调用Thread.sleep()方法时，通常会让你捕获一个InterruptedException异常，如下:

```java
public class InterruptThread extends Thread{
    @Override
    public void run() {
        try{
            while (true){
                Thread.sleep(100);//Thread.sleep会抛出InterruptedException
                System.out.println("InterruptThread正在执行");
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

```

当不需要运行InterruptThread线程时，通过调用InterruptThread.interrupt()使得 Thread.sleep() 抛出InterruptedException，就可以让线程退出循环，提前结束。在抛出InterruptedException异常之前，JVM会把中断标识位复位，此时调用线程的isInterrupted()方法将会返回false。

## 四、线程的生命周期

### 1、线程的6种状态

线程也是有生命周期，也就是存在不同的状态，状态之间相互转换，线程可以处于以下的状态之一：

#### 1.1、NEW(新建状态)

使用new创建一个线程对象，但还没有调用线程的start方法，**Thread t = new Thread()**，此时属于新建状态。

#### 1.2、RUNNABLE(可运行状态)

但在新建状态下线程调用了start方法，**t.start()**，此时进入了可运行状态。可运行状态又分为两种状态：

* ready(就绪状态)：线程对象调用stat方法后，等待JVM的调度，此时线程并没有运行。
* running(运行状态)：线程对象获得JVM调度，此时线程开始运行，如果存在多个CPU，那么允许多个线程并行运行。

线程的start方法只能调用一次，否则报错（IllegalThreadStateException）。

#### 1.3、BLOCKED(阻塞状态)

正在运行的线程因为某些原因放弃CPU，暂时停止运行，就会进入阻塞状态，此时JVM不会给该线程分配CPU，直到线程重新进入就绪状态，才有机会转到运行状态，阻塞状态只能先进入就绪状态，不能跳过就绪状态直接进入运行状态。线程进入阻塞状态常见的情况有：

* 1、当A线程处于运行状态时，试图获取同步锁，却被B线程获取，此时JVM把当前A线程放到对象的锁池(同步队列)中，A线程进入阻塞状态，等待获取对象的同步锁。
* 2、当线程处于运行状态时，发出了IO请求，此时进入阻塞状态。

```
ps: 如果是使用Synchronize关键字，那么尝试获取锁的线程会进入BLOCKED状态；如果是使用java.util.concurrent 类库中的Lock，那么尝试获取锁的线程则会进入WAITING或TIMED WAITING状态，因为java.util.concurrent 类库中的Lock是使用LockSupport来进行同步的。
```

#### 1.4、WAITING(等待状态)

正在运行的线程调用了无参数的wait方法，此时JVM把该线程放入对象的等待池（等待队列）中，此时线程进入等待状态，等待状态的线程只能被其他线程唤醒，否则不会被分配 CPU 时间片。下面是让线程进入等待状态的方法：

| 进入方法                          | 退出方法                             |
| --------------------------------- | ------------------------------------ |
| 无Timeout参数的Object.wait()      | Object.notify() / Object.notifyAll() |
| 无Timeout参数的Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法           | LockSupport.unpark(Thread)           |

#### 1.5、TIMED WAITING(计时等待状态)

正在运行的线程调用了有参数的wait方法，此时JVM把该线程放入对象的等待池中，此时线程进入计时等待状态，计时等待状态的线程被其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。下面是让线程进入等待状态的方法：

| 进入方法                            | 退出方法                                        |
| ----------------------------------- | ----------------------------------------------- |
| 调用Thread.sleep(int timeout) 方法  | 时间结束                                        |
| 有Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 有Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法        | LockSupport.unpark(Thread)                      |
| LockSupport.parkUntil() 方法        | LockSupport.unpark(Thread)                      |

```
ps：阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。
```

#### 1. 6、TREMINATED(终止状态)

又称死亡状态，表示线程的终止。线程进入终止状态的情况有：

* 1、正常执行完run方法，线程正常退出。
* 2、遇到异常而退出

线程一旦终止了，就不能再次启动，否则报错（IllegalThreadStateException）

### 2、线程的状态转换图

{% asset_img thread1.png thread1 %}

## 六、线程之间的通信

如果一个线程从头到尾的执行完一个任务，不需要和其他线程打交道的话，那么就不会存在安全性问题了，由于java内存模式的存在，如下：

{% asset_img thread2.png thread2 %}

每一个java线程都有自己的工作内存，线程之间要想协作完成一个任务，就必须通过主内存来通信，所以这里就涉及到对共享资源的竞争，在主内存中的东西都是线程之间共享，所以这里就必须通过一些手段来让线程之间完成正常通信。主要有以下两种方法：

### 1、wait() / notify()  notifyAll() 机制

它们都是Object类中的方法，它们的主要作用如下：

* wait()：执行该方法的线程对象释放同步锁（这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁），然后JVM把该线程存放在等待池中，等待其他线程唤醒该线程
* notify()：执行该方法的线程唤醒在等待池中等待的任意一个线程，把线程转到锁池中等待
* notifyAll()：执行该方法的线程唤醒在等待池中等待的所有线程，把线程转到锁池中等待

注意：上述方法只能在同步方法或者同步代码中使用，否则会报IllegalMonitorStateException异常，还有上述方法只能被同步监听锁对象来调用，不能使用其他对象调用，否则会报IllegalMonitorStateException异常。

假设A线程和B线程共同操作一个X对象，A、B线程可以通过X对象的wait方法和notify方法进行通信，流程如下：

1、当A线程执行X对象的同步方法时，A线程持有X对象的锁，则B线程没有执行同步方法的机会，B线程在X对象的锁池中等待。

2、A线程在同步方法中执行X.wait()时，A线程释放X对象的锁，进入X对象的等待池中。

3、在X对象的锁池中等待获取锁的B线程在这时获取X对象的锁，执行X对象的另一个同步方法。

4、B线程在同步方法中执行X.notify()或notifyAll()时，JVM把A线程从X对象的等待池中移到X对象的锁池中，等待获取锁。

5、B线程执行完同步方法，释放锁，A线程获取锁，从上次停下来的地方继续执行同步方法。

下面以一个ATM机存钱取钱的例子说明，ATM机要在银行把钱存进去后，其他人才能取钱，如果没钱取，只能先回家等待，等银行通知你有钱取了，再来取，如果有钱取，就直接取钱。

ATM机，存钱和取钱方法都是同步方法：

```java
public class ATM {

    private int money;
    private boolean isEmpty = true;//标志ATM是否有钱的状态

    /**
     * 往ATM机中存钱
     */
    public synchronized void push(int money){

        try{
            //ATM中有钱，等待被人把钱取走
            while (!isEmpty){
                this.wait();
            }

            //ATM中没钱了，开始存钱
            System.out.println(Thread.currentThread().getName() + ":" + "发现ATM机没钱了，存钱中...");
            Thread.sleep(2000);
            this.money = money;
            System.out.println(Thread.currentThread().getName() + ":" + "存钱完毕，存了" + money + "元");

            //存钱完毕，把标志置为false
            isEmpty = false;

            //ATM中有钱了，通知别人取钱
            this.notify();

        }catch (InterruptedException e){
            e.printStackTrace();
        }

    }

    /**
     * 从ATM机中取钱
     */
    public synchronized void pop(){
            try {

                //ATM中没钱取，等待通知
                while (isEmpty){
                    System.out.println(Thread.currentThread().getName() + ":" + "ATM机没钱，等待中...");
                    this.wait();
                }

                //ATM中有钱了，开始取钱
                System.out.println(Thread.currentThread().getName() + ":" + "收到通知，取钱中...");
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + ":"+ "取出完毕，取出了" + this.money + "钱");

                //取钱完毕，把标志置为true
                isEmpty = true;

                //ATM没钱了，通知银行存钱
                this.notify();

            } catch (InterruptedException e) {
                e.printStackTrace();
        }
    }

}
```

银行， 需要传入同一个ATM示例：

```java
public class Blank implements  Runnable  {

    private ATM mAtm;//共享资源

    public Blank(ATM atm){
        this.mAtm = atm;
    }

    @Override
    public void run() {
        //银行来存钱
        for(int i = 0; i < 2; i++){
            mAtm.push(100);
        }
    }
}
```

小明， 需要传入同一个ATM示例：

```java
public class Person implements Runnable{

    private ATM mAtm;//共享资源

    public Person(ATM atm){
        this.mAtm = atm;
    }

    @Override
    public void run() {
        //这个人来取钱
        mAtm.pop();
    }
}
```

客户端操作，我特地让小明提前来取钱，此时ATM机中是没钱的，小明要等待：

```java
 public static void main(String[] args){

        //创建一个ATM机
        ATM atm = new ATM();
        //小明来取钱
        Thread tPerson = new Thread(new Person(atm), "XiaoMing");
        tPerson.start();
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //银行来存钱
        Thread tBlank = new Thread(new Blank(atm), "Blank");
        tBlank.start();
    }

```

输出结果：

```java
XiaoMing:ATM机没钱，等待中...
Blank:发现ATM机没钱了，存钱中...
Blank:存钱完毕，存了100元
XiaoMing:收到通知，取钱中...
XiaoMing:取出完毕，取出了100钱
Blank:发现ATM机没钱了，存钱中...
Blank:存钱完毕，存了100元
```

可以看到，小明总是在收到ATM的通知后才来取钱，如果通过这个存钱取钱的例子还不了解wait/notify机制的话，可以看看这个[修厕所的例子](https://mp.weixin.qq.com/s/OriB-ouTDuCzquoFmjv9Lg)。

```
ps: wait() 和 sleep() 的区别是什么，首先wait()是Object的方法，而sleep()是Thread的静态方法，其次调用wait()会释放同步锁，而sleep()不会，最后一点不同的是调用`wait`方法需要先获得锁，而调用`sleep`方法是不需要的。
```

### 2、await()  / signal() signalAll()机制

从java5开始，可以使用Lock机制取代synchronized代码块和synchronized方法，使用java.util.concurrent 类库中提供的Condition 接口的await / signal() signalAll()方法取代Object的wait() / notify()  notifyAll() 方法。

下面使用Lock机制和Condition 提供的方法改写上面的那个例子，如下：

ATM2：

```java
public class ATM2 {

    private int money;
    private boolean isEmpty = true;//标志ATM是否有钱的状态

    private Lock mLock = new ReentrantLock();//新建一个lock
    private Condition mCondition = mLock.newCondition();//通过lock的newCondition方法获得一个Condition对象

    /**
     * 往ATM机中存钱
     */
    public void push(int money){
        mLock.lock();//获取锁
        try{
            //ATM中有钱，等待被人把钱取走
            while (!isEmpty){
                mCondition.await();
            }

            //ATM中没钱了，开始存钱
            System.out.println(Thread.currentThread().getName() + ":" + "发现ATM机没钱了，存钱中...");
            Thread.sleep(2000);
            this.money = money;
            System.out.println(Thread.currentThread().getName() + ":" + "存钱完毕，存了" + money + "元");

            //存钱完毕，把标志置为false
            isEmpty = false;

            //ATM中有钱了，通知别人取钱
            mCondition.signal();

        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            mLock.unlock();//释放锁
        }

    }

    /**
     * 从ATM机中取钱
     */
    public void pop(){
        mLock.lock();//获取锁
        try {

            //ATM中没钱取，等待通知
            while (isEmpty){
                System.out.println(Thread.currentThread().getName() + ":" + "ATM机没钱，等待中...");
                 mCondition.await();
            }

            //ATM中有钱了，开始取钱
            System.out.println(Thread.currentThread().getName() + ":" + "收到通知，取钱中...");
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + ":"+ "取出完毕，取出了" + this.money + "钱");

            //取钱完毕，把标志置为true
            isEmpty = true;

            //ATM没钱了，通知银行存钱
            mCondition.signal();

        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            mLock.unlock();//释放锁
        }
    }
}
```

可以看到ATM2改写ATM后，把方法的synchronized去掉，因为Lock机制没有同步锁的概念，然后获取lock锁，在finally里释放lock锁，还把原本Object.wait()用Condition.await()代替，原本Object.notify()用Condition.signal()代替。

客户端操作只是把ATM换成ATM2，输出结果和上面的一样，就不在累述。

### 3、死锁

多线程通信的时候很容易造成死锁，死锁一旦发生，只能通过外力解决。

**死锁是什么？**

当A线程等待获取由B线程持有的锁，而B线程正在等待获取由A线程持有的锁，发生死锁现象，JVM既不检测也不会避免这种情况，所以程序员必须保证不导致死锁。

> 官方定义：如果一组进程中的每一个进程都在等待仅由该进程组中的其他进程才能引发的事件，那么该组进程就是死锁。

**产生死锁的原因？**

多个线程对**不可抢占性资源**或**可消耗性资源**的进行争夺时，可能会产生死锁。

**产生死锁的必要条件？**

同时满足以下4个条件，就会产生死锁：

1、 互斥条件：线程对分配到的资源进行排他性使用；

2、 请求和保持条件：线程已经保持了至少一个资源，又提出了新的资源请求;

3、 不可抢占条件: 线程已获得的资源在未使用完之前不能被抢占；

4、 循环等待条件：发生死锁时，一定存在一个线程-资源的循环链。

**如何预防死锁？**

在程序运行之前，可以通过以下3点来预防死锁：

1、破坏必要条件中的一个或几个就行；

2、当多个线程都要访问共享资源A、B、C时，保证每一个线程都按照相同的顺序去访问去访问他们，比如先访问A，接着访问B，最后访问C；

3、不要使用Thread类中过时的方法，因为容易导致死锁，所以被废弃，例如A线程获得对象锁，正在执行一个同步方法，如果B线程调用A线程的suspend()，此时A线程暂停运行，放弃CPU，但是不会放弃锁，所以B就永远不会得到A持有的锁。

> 在操作系统中，还可以在程序运行时，通过**银行家算法**来避免死锁。

**解决死锁的办法？**

1、从一个或多个线程中，抢占足够数量的资源分配给死锁线程，解决死锁状态；

2、终止或撤销系统中一个或多个线程，直到打破死锁状态。

> 上面对死锁讨论的所有情况，同样适用于进程，线程就是""轻量级进程""。

### 4、 Thread类中过时的方法

由于线程安全问题，被弃用，如下：

- void suspend()：暂停当前线程。
- void resume()：恢复当前线程。
- void stop()：结束当前线程

suspend()方法在调用之后不会释放已经占有的资源(锁)，然后进入睡眠状态，这样很容易导致死锁； stop()方法直接终止线程，不会保证线程资源的正常释放，导致程序处于不确定状态。对于suspend()和 resume()可以用上面提到的等待/通知机制代替，而 stop()方法可以用上面提到的终止线程运行的3种方式代替。

## 七、线程的控制操作

下面来看一些可以控制线程的操作。

### 1、线程休眠

让执行的线程暂停等待一段时间，进入计时等待状态，使用如下：

```java
public static void main(String[] args){
    Thread.sleep(1000);
}
```

调用sleep()后，当前线程放弃CPU，在指定的时间段内，sleep所在的线程不会获得执行的机会，在此状态下该线程不会释放同步锁。

### 2、联合线程

在线程中调用另一个线程的 join() 方法，会将当前线程置于阻塞状态，等待另一个线程完成后才继续执行，原理就是等待/通知机制，使用如下：

```java
public class JoinThread extends Thread {

    @Override
    public void run() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("JoinThread执行完毕！");
    }
}

 public static void main(String[] args) throws InterruptedException {
        JoinThread joinThread = new JoinThread();
        joinThread.start();
        System.out.println("主线程等待...");
        joinThread.join();//主线程等join线程执行完毕后才继续执行
        System.out.println("主线程执行完毕");
    }

输出结果：
主线程等待...
JoinThread执行完毕！
主线程执行完毕
```

对于以上代码，主线程会等join线程执行完毕后才继续执行，因此最后的结果能保证join线程的输出先于主线程的输出。

### 3、后台线程

顾名思义，在后台运行的线程，其目的是为其他线程提供服务，也称“守护线程”，JVM的垃圾回收线程就是典型的后台线程，通过**t.setDaemon(true)**把一个线程设置为后台线程，如下：

```java
public class DeamonThread extends Thread {
    @Override
    public void run() {
        System.out.println(getName());
    }
}

 public static void main(String[] args) throws InterruptedException {
        //主线程不是后台线程，是前台线程
        DeamonThread deamonThread = new DeamonThread();
        deamonThread.setDaemon(true);//设置子线程为后台线程
        deamonThread.start();
        //通过deamonThread.isDaemon()判断是否是后台线程
        System.out.println(deamonThread.isDaemon());
    }

输出结果：true
```

后台线程有以下几个特点：

1、若所有的前台线程死亡，后台线程自动死亡，若前台线程没有结束，后台线程是不会结束的。

2、前台线程创建的线程默认是前台线程，可以通过setDaemon(true)设置为后台线程，在后台线程创建的新线程，新线程是后台线程。

注意：t.setDaemon(true)方法必须在start方法前调用，否则会报IllegalMonitorStateException异常

### 4、线程优先级

当线程的时间片用完时就会发生线程调度，而线程优先级就是决定线程需要多或少分配一些CPU时间片的线程属性，在java中，通过一个成员变量priority来控制优先级，在线程构建时可以通过setPriority(int)方法来修改线程的优先级，如下：

```java
public class PriorityThread extends Thread{
    @Override
    public void run() {
        super.run();
    } 
}

public static void main(String[] args) throws InterruptedException {
    PriorityThread priorityThread = new PriorityThread();
    priorityThread.setPriority(Thread.MAX_PRIORITY);//10
    priorityThread.start();
}
```

优先级范围从1到10，默认是5，优先级高的线程分配的时间片数量要多于优先级低的线程。

```
ps: 在不同的JVM以及操作系统上，线程优先级规划会有差异，有些操作系统会忽略对线程优先级的设定，所以线程优先级不能作为程序正确性的依赖保证，因为操作系统可以完全不用理会线程优先级的设定
```

### 5、线程礼让

对静态方法 Thread.yield() 的调用，声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。如下：

```java
public class YieldThread extends Thread {
    @Override
    public void run() {
        System.out.println("已经完成重要部分，可以让其他线程获取CPU时间片");
        Thread.yield();
    }
}
```

该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。也就是说，就算你执行了这个方法，该线程还是有可能继续运行下去。

### 6、线程组

java.lang.ThreadGroup类表示线程组，可以对一组线程进行集中管理，当用户在创建线程对象时，可以通过构造器指定其所属的线程组：Thread(ThreadGroup group, String name)。

如果A线程创建B线程，如果没有设置B线程的分组，那么B线程加入到A线程的线程组，一旦线程加入某个线程组，该线程就一直存在于该线程组中直到线程死亡，不能在中途修改线程的分组。

当java程序运行时，JVM会创建名为main的线程组，在默认情况下，所以的线程都属于该线程组。

## 结语

本文到这里就结束了，在平时开发中我们一般都只会使用线程，但却很少去了解线程的生命周期、通信机制等，但我们不要忽略掉这些知识点，它们都是**面试常客**，也是非常的重要，在java中，一般不推荐你直接**new**一个线程使用，如果你需要创建的线程数量非常多的话，这时就需要使用[**线程池**](https://blog.csdn.net/Rain_9155/article/details/90757694)来帮助你管理线程的创建，在线程的内部中，还有一个用于存储数据的**Map集合**，java提供了一个[**ThreadLocal**](https://blog.csdn.net/Rain_9155/article/details/86768467)类来操作这些集合，ThreadLocal在多线程环境下可以很好的保证了这些数据只能为本线程使用，从而避免了并发问题。

以上就是我对线程的总结，希望对大家有所帮助。