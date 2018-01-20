---
title: Java多线程编程基础
date: 2016-09-11 16:00:07
tags: Java
---

## 多线程的实现
在 Java 之中，如果要想实现多线程的程序，那么就必须依靠一个线程的**主体类**（就好比主类的概念一
样，表示的是一个线程的主类），但是这个线程的主体类在定义的时候也需要有一些特殊的要求，这个类
可以继承**Thread** 类或实现**Runnable** 接口来完成定义。
<!--more-->
### 继承 Thread 类实现多线程
**java.lang.Thread** 是一个负责线程操作的类，任何的类只需要继承了**Thread** 类就可以成为一个线程的主类，但是既然是主类必须有它的使用方法，而线程启动的主方法是需要覆写Thread类中的**run()方法**才可以。
```java
class MyThread extends Thread { // 线程的主体类
    private String title;
    public MyThread(String title) {
        this.title = title;
    }
    @Override
    public void run() { // 线程的主方法
        for (int x = 0; x < 50; x++) {
            System.out.println(this.title + "运行，x = " + x);
        }
    }
}
```
主类调用：
```java
public class TestDemo {
    public static void main(String[] args) throws Exception {
        MyThread mt1 = new MyThread("线程A") ;
        MyThread mt2 = new MyThread("线程B") ;
        MyThread mt3 = new MyThread("线程C") ;
        mt1.run() ;
        mt2.run() ;
        mt3.run() ;
    }
}
```
但是以上操作并没有真正启动多线程，，因为多个线程彼此之间的执行一定是交替的方式运行，而此时是顺序执行，即：每一个对象的代码执行完之后才向下继续执行。如果要想在程序之中真正的启动多线程，必须依靠Thread类的一个方法：`public void start()`，表示真正启动多线程，调用此方法后会间接调用`run()`方法:
```
public class TestDemo {
    public static void main(String[] args) throws Exception {
        MyThread mt1 = new MyThread("线程A") ;
        MyThread mt2 = new MyThread("线程B") ;
        MyThread mt3 = new MyThread("线程C") ;
        mt1.start() ;
        mt2.start() ;
        mt3.start() ;
    }
}
```
**要想启动线程必须依靠Thread类的start()方法执行，线程启动之后会默认调用了run()方法。**

那为什么调用 start() 方法就可以实现多线程呢？查看其定义如下：
```java
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();
    group.add(this);
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
        }
    }
}
private native void start0();
```
首先看到该方法会抛出一个异常：`IllegalThreadStateException()`，这个异常属于运行时异常。当一个线程对象被重复启动之后会抛出此异常，即：一个线程对象只能启动唯一的一次。

在`start()`方法之中有一个最为关键的部分就是`start0()`方法，而且这个方法上使用了一个**native**关键字的定义。
**native**关键字指的是**Java本地接口调用**（Java Native Interface），即：是使用Java调用本机操作系统的函数功能完成一些特殊的操作，而这样的代码开发在Java之中几乎很少出现，因为Java的最大特点是可移植性，如果一个程序只能在固定的操作系统上使用，那么可移植性就将彻底的丧失，所以，此操作一般只作为兴趣使用。

多线程的实现一定需要操作系统的支持，`start0()`方法和抽象方法类似，没有方法体，而这个方法体交给**JVM**去实现。即：在windows下的JVM可能使用**A**方法实现了`start0()`，而在linux下的JVM可能使用了**B**方法实现了`start0()`，但是在调用的时候并不会去关心具体是何方式实现了`start0()`方法，只会关心最终的操作结果，交给JVM去匹配了不同的操作系统。所以在多线程操作之中，使用`start()`方法启动多线程的操作是需要进行操作系统函数调用的。

### 实现Runnable接口实现多线程
使用 Thread类实现多进程最大的缺点就是单继承问题，可以使用 Runnable 接口实现多线程来弥补这个缺陷。
```java
public interface Runnable {
    public void run();
}
```
通过Runnable接口实现多线程:
```java
class MyThread implements Runnable { // 线程的主体类
    private String title;
    public MyThread(String title) {
        this.title = title;
    }
    @Override
    public void run() { // 线程的主方法
        for (int x = 0; x < 50; x++) {
            System.out.println(this.title + "运行，x = " + x);
        }
    }
}
```
由于使用接口，没有继承启动多线程要依靠 Thread类的 `start()` 方法，为了解决这个问题，，在Thread类中定义了一个构造方法：`public Thread(Runnable target)`，接收Runnable接口对象。
```java
public class TestDemo {
    public static void main(String[] args) throws Exception {
        MyThread mt1 = new MyThread("线程A");
        MyThread mt2 = new MyThread("线程B");
        MyThread mt3 = new MyThread("线程C");
        new Thread(mt1).start();
        new Thread(mt2).start();
        new Thread(mt3).start();
    }
}
```
### Thread类和Runnable接口实现多线程的区别
事实上，Thread类也是Runnable接口的子类，Thread 类的定义如下：
```java
public class Thread extends Object implements Runnable
```
这样的话，之前的程序结构就是下面的形式：
![代理设计模式][1]

这个时候所表现出来的代码模式**非常类似于代理设计模式**，但是它并不是严格意义上代理设计模式，因为从严格来讲代理设计模式之中，代理主题所能够使用的方法依然是接口中定义的run()方法，而此处代理主题调用的是start()方法，所以只能够说形式上类似于代理设计模式，但本质上还是有差别的。

除此之外，**使用Runnable接口可以更加方便的表示出数据共享的概念。**

### 线程的操作状态
每一个线程对象实际上都拥有属于自己的运行状态:

- 所有的线程对象都必须通过关键字**new**进行创建
- 线程如果要进行启动则一定会调用Thread类的**start()**方法，但是代码可能会分先后顺序。

```java
new Thread(mt).start();
new Thread(mt).start();
new Thread(mt).start();
```
以上启动了三个线程，虽然在代码上有先后调用start()方法的顺序，可是对于JVM而言，都表示着所有的线程将同时进入到就绪状态，等待执行。
- 进入到就绪状态之后，将等待着CPU进行资源的抢占，抢占到了资源之后，线程会进如到运行状态，开始执行run()方法体之中所定义的代码
- 每一个线程执行run()方法到一定的时间的时候会让出CPU资源，进入到阻塞状态，而后重新回到就绪状态等待下次资源调度并继续执行run()方法中的代码
- 如果全部方法执行完毕之后，将进入到线程的终止状态，并且不会再进入到就绪状态，直接结束。

线程的状态：
![线程的状态][2]

## 线程的主要操作方法
### 线程的命名
线程本身是属于不可见的运行状态的，所以如果要想在程序之中操作线程，唯一依靠的就是线程名称，而要想取得和设置线程的名称可以使用如下的方法：

- 构造方法：public Thread(Runnable target, String name)；
- 设置名字：public final void setName(String name)；
- 取得名字：public final String getName()。

取得当前线程对象的方法：`public static Thread currentThread()`

>如果为线程设置了名字，那么会使用用户定义的名字，而如果没有设置线程名称，会自动的为其分配一个名称。

每一次使用java命令执行一个类的时候就表示启动了一个**JVM的进程**，而主方法是这个进程上的一个线程。一个 JVM 进程启动的时候至少启动两个进程，一个是**main**，一个是**gc**。

### 线程的休眠
线程的休眠指的是让程序的执行速度变慢一些，方法：
```java
public static void sleep(long millis) throws InterruptedException
```

设置的休眠单位是毫秒。
```java
class MyThread implements Runnable { // 线程的主体类
    @Override
    public void run() { // 线程的主方法
        for (int x = 0; x < 100; x++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "，x = " + x);
        }
    }
}
public class TestDemo {
    public static void main(String[] args) throws Exception {
        MyThread mt = new MyThread();
        new Thread(mt, "线程A").start();
        new Thread(mt, "线程B").start();
        new Thread(mt, "线程C").start();
        new Thread(mt, "线程D").start();
        new Thread(mt, "线程E").start();
    }
}
```
### 线程的优先级
从理论上讲，线程的优先级越高，越有可能先执行。如果要想操作线程的优先级有如下两个方法：

- 设置线程的优先级：public final void setPriority(int newPriority)；
- 取得线程的优先级：public final int getPriority()；

发现设置和取得优先级的时候都是利用了一个int型数据的操作，而这个int型数据有三种取值：

- 最高优先级：public static final int MAX_PRIORITY，10；
- 中等优先级：public static final int NORM_PRIORITY，5；
- 最低优先级：public static final int MIN_PRIORITY，1；

主线程的优先级是5，是**中等级别**。

### 等待与唤醒

等待：public final void wait() throws InterruptedException；
唤醒第一个等待线程：public final void notify()；
唤醒全部等待线程：public final void notifyAll()。

>对于唤醒的两个操作：
>notify()是**按照等待顺序**进行了唤醒，而使用了notifyAll()则表示所有等待的线程都会被唤醒，那个线程的优先级高，那个线程就先执行。

**sleep()和wait()的区别？**

- sleep()是Thread类定义的static方法，表示线程休眠，休眠到一定时间后自动唤醒；
- wait()是Object类定义的方法，表示线程等待，一直到执行了notify()或notifyAll()之后才结束等待。

## 线程的同步与死锁
### 同步
所谓的同步问题指的是多个线程操作同一资源时所带来的信息的安全性问题。同步具体的实现思想是指多个操作在同一时间段内只能有一个线程进行，其他线程要等待此线程完成之后才可以继续执行。

实现同步有两种方式，一种是**同步代码块**，另外一种就是**同步方法**。

① **同步代码块**，使用`synchronized`关键字定义的代码块就称为同步代码块，但是在进行同步的操作之中必须设置一个要同步的对象，而这个对象应该理解为当前对象：this。
```java
class MyThread implements Runnable { // 线程的主体类
    private int ticket = 6;
    @Override
    public void run() { // 线程的主方法
        for (int x = 0; x < 10; x++) {
            synchronized (this) { // 同步代码块
                if (this.ticket > 0) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()
                    + "卖票，ticket = " + this.ticket--);
                }
            }
        }
    }
}
public class TestDemo {
    public static void main(String[] args) throws Exception {
        MyThread mt = new MyThread();
        new Thread(mt, "票贩子A").start();
        new Thread(mt, "票贩子B").start();
        new Thread(mt, "票贩子C").start();
        new Thread(mt, "票贩子D").start();
        new Thread(mt, "票贩子E").start();
    }
}
```
② 同步方法
```java
class MyThread implements Runnable { // 线程的主体类
    private int ticket = 6;
    @Override
    public void run() { // 线程的主方法
        for (int x = 0; x < 10; x++) {
            this.sale() ;
        }
    }
    public synchronized void sale() {
        if (this.ticket > 0) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()
            + "卖票，ticket = " + this.ticket--);
        }
    }
}
```
**加入同步之后明显比不加入同步慢许多，所以同步的代码性能会很低，但是数据的安全性会高。**

### 死锁
同步就是指一个线程要等待另外一个线程执行完毕才会继续执行的一种操作形式，但是如果在一个操作之中都是在互相等着的话，那么就会出现**死锁**问题。

## 线程间的经典操作案例
在多线程的开发之中存在一种称为“**生产者和消费者的程序**”，这个程序的主要功能是生产者负责生产一些内容，每当生产完成之后，会由消费者取走全部内容。
```java
class Message {
    private String title ;
    private String content ;
    private boolean flag = true ;
    // flag == true：表示可以生产，但是不能取走
    // flag == false：表示可以取走，但是不能生产
    public synchronized void set(String title,String content) {
        if (this.flag == false) { // 已经生产过了，不能生产
            try {
                super.wait() ;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.title = title ;
        try {
            Thread.sleep(200) ;
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.content = content ;
        this.flag = false ;
        super.notify() ;
    }
    public synchronized void get() {
        if (this.flag == true) { // 不能取走
            try {
                super.wait() ;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        try {
            Thread.sleep(100) ;
        } catch (InterruptedException e) {
                e.printStackTrace();
        }
        System.out.println(this.title + " --> " + this.content);
        this.flag = true ; // 已经取走了，可以继续生产
        super.notify() ;
    }
}
```

[1]: http://static.zybuluo.com/guoxs/6gpf6y99uovm1vxxw29wcf9g/%E7%BB%98%E5%9B%BE1.png
[2]: http://static.zybuluo.com/guoxs/sj5p34wp84ry6ln8oul2tcz6/%E8%BF%9B%E7%A8%8B.png
