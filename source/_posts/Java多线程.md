---
title: Java 多线程&并发编程
date: 2020-12-9 19:54:09
categories: Coding
tags: [Java, JUC]
---

----

<!--more-->

# Java 多线程



## 创建线程的方式

### 继承 Thread 线程类

1. 自定义类继承 `Thread` 线程类
2. 在自定义类中重写 `Thread` 类的 `run` 方法
3. 创建自定义类对象（线程对象）
4. 调用 `start` 方法，启动线程，通过 JVM，调用线程中的 `run` 方法



### 实现 Runnable 接口

1. 创建线程任务类实现 `Runnable` 接口
2. 在线程任务中重写接口中的 `run` 方法
3. 创建线程任务类对象
4. 创建线程对象，把线程任务类对象作为 `Thread` 类构造方法的参数使用
5. 调用 `start` 方法，启动线程，通过 JVM，调用线程中的 `run` 方法



## 线程池

线程池，其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。

![](http://images.yingwai.top/picgo/xianchengchi.png)

### 为什么要使用线程池

1. 在java中，如果每个请求到达就创建一个新线程，开销是相当大的。
2. 在实际使用中，创建和销毁线程花费的时间和消耗的系统资源都相当大，甚至可能要比在处理实际的用户请求的时间和资源要多的多。
3. 除了创建和销毁线程的开销之外，活动的线程也需要消耗系统资源。
   如果在一个jvm里创建太多的线程，可能会使系统由于过度消耗内存或“切换过度”而导致系统资源不足。
4. 为了防止资源不足，需要采取一些办法来限制任何给定时刻处理的请求数目，尽可能减少创建和销毁线程的次数，特别是一些资源耗费比较大的线程的创建和销毁，尽量利用已有对象来进行服务。

线程池主要用来解决线程生命周期开销问题和资源不足问题。通过对多个任务重复使用线程，线程创建的开销就被分摊到了多个任务上了，而且由于在请求到达时线程已经存在，所以消除了线程创建所带来的延迟。这样，就可以立即为请求服务，使用应用程序响应更快。另外，通过适当的调整线程中的线程数目可以防止出现资源不足的情况。



### 使用线程池的方式

通常，线程池都是通过线程池工厂创建，再调用线程池中的方法获取线程，再通过线程去执行任务方法。

#### Runnable 接口

* `Executors`：线程池创建工厂类

  ```java
  public static ExecutorService newFixedThreadPool(int nThreads);	// 返回线程池对象
  ```

* `ExecutorService`：线程池类

  ```java
  Future<?> submit(Runnable task);	// 获取线程池中的某一个线程对象，并执行
  ```

* `Future` 接口：用来记录线程任务执行完毕后产生的结果。



**使用线程池中线程对象的步骤：**

1. 创建线程池对象
2. 创建 `Runnable` 接口子类对象
3. 提交 `Runnable` 接口子类对象
4. 关闭线程池



**代码演示：**

```java
public class ThreadPoolDemo {
	public static void main(String[] args) {
		//创建线程池对象
		ExecutorService service = Executors.newFixedThreadPool(2);//包含2个线程对象
		//创建Runnable实例对象
		MyRunnable r = new MyRunnable();
		
		//自己创建线程对象的方式
		//Thread t = new Thread(r);
		//t.start(); ---> 调用MyRunnable中的run()
		
		//从线程池中获取线程对象,然后调用MyRunnable中的run()
		service.submit(r);
		//再获取个线程对象，调用MyRunnable中的run()
		service.submit(r);
		service.submit(r);
		//注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。将使用完的线程又归还到了线程池中

		//关闭线程池
		service.shutdown();
	}
}
```



#### Callable接口

* `Callable` 接口：与 `Runnable` 接口功能相似，用来指定线程的任务。其中的 `call()` 方法，用来返回线程任务执行完毕后的结果，`call` 方法可抛出异常。

* `ExecutorService`：线程池类

  ```java
  <T> Future<T> submit(Callable<T> task);	// 获取线程池中的某一个线程对象，并执行线程中的call()方法
  ```

* `Future` 接口：用来记录线程任务执行完毕后产生的结果。



**使用线程池中线程对象的步骤：**

1. 创建线程池对象
2. 创建 `Callable` 接口子类对象
3. 提交 `Callable` 接口子类对象
4. 关闭线程池



## 同步锁

多个线程想要保证线程安全，必须要使用同一个锁对象

* 同步代码块

  ```java
  synchronized (锁对象) {
      可能产生线程安全问题的代码
  }
  ```

  **同步代码块的锁对象可以是任意的对象。**

* 同步方法

  ```java
  public synchronized void method() {
      可能产生线程安全问题的代码
  }
  ```

  **同步方法中的锁对象是 this**

* 静态同步方法

  ```java
  public synchronized void method() {
      可能产生线程安全问题的代码
  }
  ```

  **静态同步方法中的锁对象是 类名.class**



## 多线程的几种实现方案

1. 继承 `Thread` 类
2. 实现 `Runnable` 接口
3. 通过线程池，实现 `Callable` 接口



## 同步的几种方式

1. 同步代码块
2. 同步方法
3. 静态同步方法



## run() 和 start() 的区别

启动一个线程是 `start()`

区别：

`start()`：启动线程，并调用线程中的 `run()` 方法

`run()`：执行该线程对象要执行的任务



## sleep() 和 wait() 方法的区别

`sleep()`：不释放锁对象，释放CPU使用权；在休眠的时间内，不能唤醒。

`wait()`：释放锁对象，释放CPU使用权；在等待的时间内，能唤醒。



----



# JUC

![](http://images.yingwai.top/picgo/20201209100316.png)



## 多线程代码模板

1. 高聚低合前提下，线程操作资源类
2. 判断/干活/通知
3. 多线程交互中，必须要防止多线程的虚假唤醒（判断只能用 `while`，不能用 `if`）
4. 标志位

### 使用 `synchronized`

首先回顾一下最基本的使用线程的两种方法（Thread、Runable）情况下的代码模板：

**高内聚低耦合：线程、操作、资源类**

```java
class 资源类 {
	...
	public synchronized void 操作 {
		...
	}
	...
}

public class ThreadDemo {
    public static void main(String[] args) {
        资源类 resource = new 资源类();
        // 线程
        // 匿名内部类方法
        new Thread(new Runable{
            @Overwrite
            public void run() {
                for (int i = 0; i < 10; ++i) {
                    resource.操作;
                    System.out.println(Thread.currentThread().getName + "\t" + "Hello Thread1!");
                }
            }
        }, "t1").start();
        // lambda表达式方法
        new Thread(() -> {for (int i = 0; i < 10; ++i) System.out.println(Thread.currentThread().getName + "\t" + "Hello Thread2!");}, "t2").start();
    }
}
```



#### 线程通信

`wait()`、`sleep()`、`notify()`、`notifyAll()`

多线程交互中，必须要防止多线程的虚假唤醒（判断只能用 `while`，不能用 `if`）



### 使用 `Lock`

```java
class 资源类 {
    private Lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
	...
	public void 操作 {
		lock.lock();
        try {
            while (条件) {
                condition.await();
            }
            ...
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
	}
	...
}

public class ThreadDemo {
    public static void main(String[] args) {
        资源类 resource = new 资源类();
        // 线程
        // 匿名内部类方法
        new Thread(new Runable{
            @Overwrite
            public void run() {
                for (int i = 0; i < 10; ++i) {
                    resource.操作;
                    System.out.println(Thread.currentThread().getName + "\t" + "Hello Thread1!");
                }
            }
        }, "t1").start();
        // lambda表达式方法
        new Thread(() -> {for (int i = 0; i < 10; ++i) System.out.println(Thread.currentThread().getName + "\t" + "Hello Thread2!");}, "t2").start();
    }
}
```



#### 线程通信

`condition.await()`、`condition.signal()`、`condition.signalAll()`

需要精确唤醒某一个线程时可以创建多个 `Condition` 对象。



## `synchronized` 和 `Lock` 的区别

|   类别   |                         synchronized                         |                             Lock                             |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 存在层次 |                  Java的关键字，在jvm层面 上                  |                          是一个接口                          |
| 锁的释放 | 1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁 |         在finally中必须释放锁，不然容易造成线程死锁          |
| 锁的获取 |  假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待  | 分情况而定，Lock有多个锁获取的方式，大致就是可以尝试获得锁，线程可以不用一直等待(可以通过tryLock判断有没有锁) |
|  锁状态  |                           无法判断                           |                           可以判断                           |
|  锁类型  |                    可重入 不可中断 非公平                    |               可重入 可判断 可公平（两者皆可）               |
|   性能   |                           少量同步                           | 大量同步 1. Lock可以提高多个线程进行读操作的效率。（可以通过readwritelock实现读写分离） 2. 在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态 3. ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。 |



### `synchronized` 的缺陷

synchronized是java中的一个关键字，也就是说是Java语言内置的特性。那么为什么会出现Lock呢？

如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，而这里获取锁的线程释放锁只会有两种情况：

1. 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；

2. 线程执行发生异常，此时JVM会让线程自动释放锁。

那么如果这个获取锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，其他线程便只能干巴巴地等待，试想一下，这多么影响程序执行效率。

因此就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间或者能够响应中断），通过Lock就可以办到。

再举个例子：当有多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作会发生冲突现象，但是读操作和读操作不会发生冲突现象。

但是采用synchronized关键字来实现同步的话，就会导致一个问题：

如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程只能等待无法进行读操作。

因此就需要一种机制来使得多个线程都只是进行读操作时，线程之间不会发生冲突，通过Lock就可以办到。

另外，通过Lock可以知道线程有没有成功获取到锁。这个是synchronized无法办到的。

总结一下，也就是说Lock提供了比synchronized更多的功能。但是要注意以下几点：

1. Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；

2. Lock和synchronized有一点非常大的不同，采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。



## 多线程8锁

现在有如下资源类：

```java
class Phone {
    public synchronized void sendEmail() throws Exception {
        System.out.println("----sendEmail");
    }
    public synchronized void sendSMS() throws Exception {
        System.out.println("----sendSMS");
    }
}
```

### 标准访问

```java
class Lock8 {
    public static void main(String[] args) throws Exception {
        Phone phone = new Phone();
        
        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "A").start();
        
        //Thread.sleep(100);
        
        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "B").start();
    }
}
```

先打印邮件还是短信？

都有可能，看操作系统调度。在某个线程访问资源类的 `synchronized` 方法时，得到的是**对象锁**，一个资源类同一时间下只能被一个线程访问该类其中一个 `synchronized` 方法。

> 一个对象里面如果有多个 `synchronized` 方法，某一时刻内，只要一个线程去调用其中的一个 `synchronized` 方法了，其它的线程都只能等待，换句话说，某一时刻内，只能有唯一一个线程去访问这些 `synchronized` 方法。
>
> 锁的是当前对象 `this`，被锁定后，其它的线程都不能进入到当前对象的其它的 `synchronized` 方法。

### 邮件方法暂停4秒

资源类的邮件方法作如下修改：

```java
class Phone {
    public synchronized void sendEmail() throws Exception {
        try { TimeUnit.SECONDS.sleep(4); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("----sendEmail");
    }
    ...
}
```

先打印邮件还是短信？

答案解释同*标准访问*。

### 新增一个普通方法

资源类新增一个方法：

```java
class Phone {
    ...
    public void sayHello() throws Exception {
        System.out.println("----hello");
    }
}
```

线程 B 调用 `sayHello()` 方法：

```java
class Lock8 {
    public static void main(String[] args) throws Exception {
        ...
        new Thread(() -> {
            try {
                phone.sayHello();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "B").start();
    }
}
```

先打印邮件还是 hello？

hello，因为 `sayHello()` 方法不需要锁即可进入。

### 两部手机

new 两个 Phone 对象分别给两个线程使用：

```java
class Lock8 {
    public static void main(String[] args) throws Exception {
        Phone phone = new Phone();
        Phone phone2 = new Phone();
        
        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "A").start();
        
        //Thread.sleep(100);
        
        new Thread(() -> {
            try {
                phone2.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "B").start();
    }
}
```

先打印邮件还是短信？

短信，两个线程使用不同的对象则不需要争抢资源，由于邮件方法会暂停，因此 B 线程的打印短信方法会先执行。

### 同一部手机两个静态同步方法

给资源类的方法加上静态关键字：

```java
class Phone {
    public static synchronized void sendEmail() throws Exception {
        try { TimeUnit.SECONDS.sleep(4); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("----sendEmail");
    }
    public static synchronized void sendSMS() throws Exception {
        System.out.println("----sendSMS");
    }
}
```

两个线程还是调用同一台手机的两个不同方法，先打印邮件还是短信？

答案解释同*标准访问*。

### 两部手机两个静态同步方法

在两个方法都是静态同步的基础上，两个线程使用两个不同的实例对象：

```java
class Lock8 {
    public static void main(String[] args) throws Exception {
        Phone phone = new Phone();
        Phone phone2 = new Phone();
        
        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "A").start();
        
        //Thread.sleep(100);
        
        new Thread(() -> {
            try {
                phone2.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "B").start();
    }
}
```

先打印邮件还是短信？

都有可能，此时就与*两部手机但方法不为静态*的情况有区别，此时线程获得的是类锁而不是对象锁，锁住的是类的方法（因为静态方法是属于类而不是属于对象的）而不是对象的方法，因此即使两个线程使用的是不同的对象实例，但还是需要抢夺资源。

### 同一部手机，一个普通同步方法和一个静态同步方法

资源类：

```java
class Phone {
    public static synchronized void sendEmail() throws Exception {
        try { TimeUnit.SECONDS.sleep(4); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("----sendEmail");
    }
    public synchronized void sendSMS() throws Exception {
        System.out.println("----sendSMS");
    }
}
```

先打印邮件还是短信？

短信，这里两个线程不需要抢夺资源，因为两个方法的锁不一样，静态同步方法是类锁，普通同步方法是对象锁，两者不冲突。这里静态同步方法由于有休眠，因此短信先打印。

### 两部手机，一个普通同步方法和一个静态同步方法

资源类：

```java
class Phone {
    public static synchronized void sendEmail() throws Exception {
        try { TimeUnit.SECONDS.sleep(4); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("----sendEmail");
    }
    public synchronized void sendSMS() throws Exception {
        System.out.println("----sendSMS");
    }
}
```

两个线程使用两个不同的实例对象：

```java
class Lock8 {
    public static void main(String[] args) throws Exception {
        Phone phone = new Phone();
        Phone phone2 = new Phone();
        
        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "A").start();
        
        //Thread.sleep(100);
        
        new Thread(() -> {
            try {
                phone2.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "B").start();
    }
}
```

先打印邮件还是短信？

答案解释类似*同一部手机，一个普通同步方法和一个静态同步方法*。



### 总结

* 所有的非静态同步方法用的都是同一把锁——实例对象本身。

* `synchronized` 实现同步的基础：Java 中的每一个对象都可以作为锁。

* 具体表现为以下三种形式：

  1. 对于普通同步方法，锁是当前实例对象
  2. 对于静态同步方法，锁是当前类的 `Class` 对象
  3. 对于同步方法块，锁是 `synchonized` 括号里配置的对象

* 当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。

  > 也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其他非静态同步方法必须等待获取锁的方法释放锁后才能获取锁，可是别的实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同的锁，所以毋须等待该实例对象已获取锁的非静态同步方法释放锁就可以获取他们自己的锁。

* 所有的静态同步方法用的也是同一把锁——类对象本身。

  >对象锁和类锁（`this/Class`）这两把锁是两个不同的对象，所以静态同步方法与非静态同步方法之间是不会有竟态条件的。但是一旦一个静态同步方法获取锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取锁，而不管是同一个实例对象的静态同步方法之间，还是不同的实例对象的静态同步方法之间，只要它们是同一个类的实例对象！



## 集合框架不安全类

### List 不安全

#### 解决方法

1. `Vector`
2. `Collections.synchronizedList(new ArrayList<>());`
3. `CopyOnWriteArrayList`



#### CopyOnWriteArrayList

![](http://images.yingwai.top/picgo/20201208153609.png)

写时复制：

CopyOnWrite容器即写时复制的容器。往一个容器添加元素的时候，不直接往当前容器`Object[]` 添加，而是先将当前容器 `Object[]` 进行Copy，复制出一个新的容器 `Object[] newElements`，然后在新的容器 `Object[] newElements` 里添加元素，添加完元素后，再将原容器的引用指向新的容器 `setArray(newElements);`。这样做的好处是可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    
    lock.lock();
    
    try {
        object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```



### Set 不安全

#### 解决方法

1. `Collections.synchronizedSet(new HashSet<>());`
2. `CopyOnWriteArraySet`



### Map 不安全

#### 解决方法

1. `Collections.synchronizedMap(new HashMap<>());`
2. `ConcurrentHashMap`



## Callable

Java 中第三种获得多线程的方式就是使用 Callable：

1. 首先新建类实现 Callable 接口，方法需要返回什么类型 `Callable<E>` 中的的泛型就填什么类型；
2. 创建 FutureTask 对象，将实现了 Callable 接口的对象作为参数传入其构造方法；
3. `new Thread(FutureTask对象, "线程名").start;`；
4. 在需要结果时调用 FutureTask 对象的 `get` 方法。

```java
class MyThread implements Runnable{
    @Override
    public void run() {

    }
}

class MyThread2 implements Callable<Integer>{

    @Override
    public Integer call() throws Exception {
        System.out.println("******come in call method()");
        return 1080;
    }
}

/**
 * 多线程中，第3种获得多线程的方式
 */
public class CallableDemo7 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask(new MyThread2());

        new Thread(futureTask,"A").start();

        Integer result = futureTask.get();
        System.out.println(result);
    }
}
```

FutureTask 的继承图：

![](http://images.yingwai.top/picgo/20201208153623.png)

## JUC 辅助类

### CountDownLatch

CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他6个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <= 6; ++i) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+"\t任务完成");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t任务A完成");
    }
}
```

#### 构造器

```java
public CountDownLatch(int count) {};	// 参数count为计数值
```

#### 方法

CountDownLatch 主要有两个方法：

* `await()`：当一个或多个线程调用 `await` 方法时，这些线程会阻塞；
* `countDown()`：其它线程调用 `countDown` 方法会将计数器减1（调用 `countDown` 方法的线程不会阻塞）；
* 当计数器的值变为0时，因 `await` 方法阻塞的线程会被唤醒，继续执行。



### CyclicBarrier

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

如果说 CountDownLatch 是倒着数，那么 CyclicBarrier 可以看做是正着数，数够了再执行：

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {System.out.println("****召唤神龙");});
        for (int i = 1; i <= 7; ++i) {
            final int tempInt = i;	// lambda表达式中变量需要用fianl修饰
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+"\t收集到第 " + tempInt + " 颗龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

#### 构造器

CyclicBarrier类位于java.util.concurrent包下，CyclicBarrier提供2个构造器：

```java
public CyclicBarrier(int parties) {};

public CyblicBarrier(int parties, Runnable barrierAction) {};
```

参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

#### 方法

CyclicBarrier中最重要的方法就是await方法，它有2个重载版本：

```java
public int await() throws InterruptedException, BrokenBarrierException {};

public int await(long timeout, TimeUnit unit) throws InterruptedException,BrokenBarrierException,TimeoutException {};
```

第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；

第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。



### Semaphore

```java
public class SemaphoreDemo {
    public static void main(String[] args) throws InterruptedException {
        Semaphore semaphore = new Semaphore(3);		// 模拟资源类，有3个空车位
        for (int i = 1; i <= 6; ++i) {
            new Thread(() -> {
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"\t抢占到了车位");
                    // 暂停一会线程
                    try { TimeUnit.SECONDS.sleep(3);} catch (InterruptedException e) { e.printStackTrace(); }
                    System.out.println(Thread.currentThread().getName()+"\t离开了车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

#### 构造器

```java
public CountDownLatch(int permits) {};	// 参数permits为资源可被同时访问的个数
```

#### 方法

在信号量上定义两种操作：

* `acquire`（获取）：当一个线程调用acquire操作时，它要么通过成功获取信号量（信号量减1），要么一直等下去，直到有线程释放信号量或超时；
* `release`（释放）：实际上会将信号量的值加1，然后唤醒等待的线程。

信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。



## ReadWriteLock

```java
class MyCache {
    //volatile:，保证可见性，不保证原子性，一个线程修改后，通知更新
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t--写入数据" + key);
            //暂停一会线程
            try {
                TimeUnit.MICROSECONDS.sleep(300);
            } catch (Exception e) {
                e.printStackTrace();
            }
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t--写入完成");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }

    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t读取数据");
            Object result = map.get(key);
            System.out.println(Thread.currentThread().getName() + "\t读取完成" + result);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}
/**
 * 多个线程同时读一个资源类没有任何问题，所以为了满足并发量，读取共享资源应该可以同时进行。
 * 但是，如果有一个线程想去写共享资源来，就不应该再有其他线程可以对改资源进行读或写
 * 小总结：
 *      读-读能共存
 *      读-写不能共存
 *      写-写不能共存
 */
public class ReadWriteLockDemo11 {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for (int i = 1; i <= 5; i++) {
            final int tempInt = i;
            new Thread(()->{
                myCache.put(tempInt+"",tempInt+"");
            },String.valueOf(i)).start();
        }
        for (int i = 1; i <= 5; i++) {
            final int tempInt = i;
            new Thread(()->{
                myCache.get(tempInt+"");
            },String.valueOf(i)).start();
        }
    }
}
```

ReadWriteLock 维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。

也就是说，允许多个线程同时读，但只要有一个线程在写，其他线程就必须等待：

|        |   读   |   写   |
| :----: | :----: | :----: |
| **读** |  允许  | 不允许 |
| **写** | 不允许 | 不允许 |



## BlockingQueue

### 阻塞队列

阻塞：必须要阻塞/不得不阻塞

阻塞队列是一个队列，在数据结构中起的作用如下图：

![](http://images.yingwai.top/picgo/20201209114108.bmp)

* 当队列是空的，从队列中获取元素的操作将会被阻塞；

* 当队列是满的，从队列中添加元素的操作将会被阻塞；
* 试图从空的队列中获取元素的线程将会被阻塞，直到其他线程往空的队列插入新的元素；
* 试图向已满的队列中添加新元素的线程将会被阻塞，直到其他线程从队列中移除一个或多个元素；
* 或者完全清空，使队列变得空闲起来并后续新增。



### 阻塞队列的用处

在多线程领域：所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤起。

为什么需要BlockingQueue？

* 好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BlockingQueue都给你一手包办了
* 在concurrent包发布以前，在多线程环境下，我们每个程序员都必须去自己控制这些细节，尤其还要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。



### 架构

![](http://images.yingwai.top/picgo/20201209114405.bmp)



### BlockingQueue 核心方法

| 方法类型 |  抛出异常   |   特殊值   |   阻塞   |          超时          |
| :------: | :---------: | :--------: | :------: | :--------------------: |
|   插入   |  `add(e)`   | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
|   移除   | `remove()`  |  `poll()`  | `take()` |   `poll(time, unit)`   |
|   检查   | `element()` |  `peek()`  |  不可用  |         不可用         |

* **抛出异常**
  * 当阻塞队列满时，再往队列里add插入元素会抛IllegalStateException:Queue full
  * 当阻塞队列空时，再往队列里remove移除元素会抛NoSuchElementException
* **特殊值**
  * 插入方法，成功返回ture失败返回false
  * 移除方法，成功返回出队列的元素，队列里没有就返回null
* **一直阻塞**
  * 当阻塞队列满时，生产者线程继续往队列里put元素，队列会一直阻塞生产者线程直到put数据or响应中断退出
  * 当阻塞队列空时，消费者线程试图从队列里take元素，队列会一直阻塞消费者线程直到队列可用
* **超时退出**
  * 当阻塞队列满时，队列会阻塞生产者线程一定时间，超过限时后生产者线程会退出



## ThreadPool

### 线程池的优势

线程池做的工作只要是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。

它的主要特点为：线程复用;控制最大并发数；管理线程。

1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的销耗。
2. 提高响应速度。当任务到达时，任务可以不需要等待线程创建就能立即执行。
3. 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会销耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。



### 线程池的使用

#### 架构说明

Java中的线程池是通过Executor框架实现的，该框架中用到了Executor，Executors，ExecutorService，ThreadPoolExecutor这几个类：

![](http://images.yingwai.top/picgo/20201209164831.bmp)

#### 编码实现

* `Executors.newFixedThreadPool(int nThreads)`：执行长期任务性能好，创建一个线程池，一池有n个固定的线程，有固定线程数的线程；
* `Executors.newSingleThreadExecutor()`：一个任务一个任务的执行，一池一线程；
* `Executors.newCachedThreadPool()`：执行很多短期异步任务，线程池根据需要创建新线程，但在先前构建的线程可用时将重用它们。可扩容，遇强则强。

```java
public class MyThreadPoolDemo {

    public static void main(String[] args) {
        //List list = new ArrayList();
        //List list = Arrays.asList("a","b");
        //固定数的线程池，一池五线程

//       ExecutorService threadPool =  Executors.newFixedThreadPool(5); //一个银行网点，5个受理业务的窗口
//       ExecutorService threadPool =  Executors.newSingleThreadExecutor(); //一个银行网点，1个受理业务的窗口
       ExecutorService threadPool =  Executors.newCachedThreadPool(); //一个银行网点，可扩展受理业务的窗口

        //10个顾客请求
        try {
            for (int i = 1; i <=10; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"\t 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }

    }
}
```



### ThreadPoolExecutor 底层原理

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                 0L, TimeUnit.MILLISECONDS,
                                 new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        		(new ThreadPoolExecutor(1, 1,
                            0L, TimeUnit.MILLISECONDS,
                            new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integre.MAX_VALUE,
                                 60L, TimeUnit.SECONDS,
                                 new SynchronousQueue<Runnable>());
}
```



#### 线程池的几个重要参数

![](http://images.yingwai.top/picgo/20201209171047.png)

* `corePoolSize`：线程池中的常驻核心线程数
* `maximumPoolSize`：线程池中能够容纳同时执行的最大线程数，此值必须大于1
* `keepAliveTime`：多余的空闲线程的存活时间当前池中线程数量超过`corePoolSize`时，当空闲时间达到`keepAliveTime`时，多余线程会被销毁直到只剩下`corePoolSize`个线程为止
* `unit`：`keepAliveTime`的单位
* `workQueue`：任务队列，被提交但尚未被执行的任务
* `threadFactory`：表示生成线程池中工作线程的线程工厂，用于创建线程，一般默认的即可
* `handler`：拒绝策略，表示当队列满了，并且工作线程大于等于线程池的最大线程数（`maximumPoolSize`）时如何来拒绝请求执行的`runnable`的策略



#### 线程池的主要处理流程

![](http://images.yingwai.top/picgo/20201210104107.bmp)

![](http://images.yingwai.top/picgo/20201210104116.bmp)



### 线程池的拒绝策略

等待队列已经排满了，再也塞不下新任务了同时，线程池中的max线程也达到了，无法继续为新任务服务。这个是时候我们就需要拒绝策略机制合理的处理这个问题。

#### JDK 内置的拒绝策略

* **AbortPolicy(默认)**：直接抛出RejectedExecutionException异常阻止系统正常运行
* **CallerRunsPolicy**：“调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量。
* **DiscardOldestPolicy**：抛弃队列中等待最久的任务，然后把当前任务加人队列中尝试再次提交当前任务。
* **DiscardPolicy**：该策略默默地丢弃无法处理的任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种策略。

以上内置拒绝策略均实现了 RejectedExecutionHandle 接口。



### 生产中如何设置合理参数

#### 工作中单一的/固定数的/可变的三种创建线程池的方法哪个用的多？

一个都不用：

![](http://images.yingwai.top/picgo/20201210104649.bmp)

一般定义 `maximumPoolSize` 为电脑cpu逻辑处理器数量+1。

> Java 如何看cpu逻辑处理器数量：`Runtime.getRuntime().availableProcessors();`



## 分支合并框架

### 原理

* Fork：把一个复杂的任务进行分拆，大事化小
* Join：把分拆任务的结果进行合并

其实就类似算法里面的分治法。



### 相关类

#### ForkJoinPool

![](http://images.yingwai.top/picgo/20201210174928.bmp)

分支合并池 *类比=>* 线程池

#### ForkJoinTask

![](http://images.yingwai.top/picgo/20201210175109.bmp)

ForkJoinTask *类比=>* FutureTask

#### RecursiveTask

![](http://images.yingwai.top/picgo/20201210175205.bmp)

递归任务：继承后可以实现递归（自己调自己）调用的任务

```java
class Fibonacci extends RecursiveTask<Integer> {
    final int n;
    Fibonacci(int n) { this.n = n; }
    Integer compute() {
        if (n <= 1)
            return n;
        Fibonacci f1 = new Fibonacci(n - 1);
        f1.fork();
        Fibonacci f2 = new Fibonacci(n - 2);
        return f2.compute() + f1.join();
    }
}
```



### 实例

```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;

class MyTask extends RecursiveTask<Integer> {
    private static final Integer ADJUST_VALUE = 10;
    private int begin;
    private int end;
    private int result;

    public MyTask(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if ((end - begin) <= ADJUST_VALUE){
           for(int i = begin; i <= end; i++){
                result = result + i;
           }
        } else {
            int middle = (begin + end) / 2;
            MyTask task01 = new MyTask(begin, middle);
            MyTask task02 = new MyTask(middle+1, end);
            task01.fork();
            task02.fork();
            result =  task01.join() + task02.join();
        }

        return result;
    }
}


/**
 * 分支合并例子
 * ForkJoinPool
 * ForkJoinTask
 * RecursiveTask
 */
public class ForkJoinDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        MyTask myTask = new MyTask(0, 100);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> forkJoinTask = forkJoinPool.submit(myTask);

        System.out.println(forkJoinTask.get());

        forkJoinPool.shutdown();
    }
}
```



## 异步回调

### 原理

![](http://images.yingwai.top/picgo/20201210185154.bmp)



### CompletableFutureDemo

```java
import java.util.concurrent.CompletableFuture;

public class CompletableFutureDemo {

    public static void main(String[] args) throws Exception {
        //同步，异步，异步回调

        //同步
		// CompletableFuture<Void> completableFuture1 = CompletableFuture.runAsync(()->{
//            System.out.println(Thread.currentThread().getName()+"\t completableFuture1");
//        });
//        completableFuture1.get();

        //异步回调
        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+"\t completableFuture2");
            int i = 10/0;
            return 1024;
        });

        completableFuture2.whenComplete((t, u)->{
            System.out.println("-------t="+t);
            System.out.println("-------u="+u);
        }).exceptionally(f->{
            System.out.println("-----exception:"+f.getMessage());
            return 444;
        }).get();

    }
}
```

在上面的代码中，

* 若 `completableFuture2` 中定义要运行的代码没有发生异常，则在获取结果时 `t` 的值就为计算的结果，`u` 为 null，且 `get()` 方法返回的是定义函数返回的结果（即 `supplyAsync()` 中定义的接口方法）；
* 若发生了异常，则 `t` 为 null 而 `u` 为异常信息，且 `get()` 方法返回的是 `.exceptionally()` 中定义的函数的返回结果。



### Future 和 CompletableFuture 区别

* 构建 FutureTask 并启动，虽然是一个异步任务，但主线程需要获取该任务的返回值时，还是需要一直阻塞等待该任务完成。

* CompletableFuture 能够将回调放到与任务不同的线程中执行，也能将回调作为继续执行的同步函数，在与任务相同的线程中执行。它避免了传统回调最大的问题，那就是能够将控制流分离到不同的事件处理器中。

* CompletableFuture 弥补了 Future 模式的缺点。在异步的任务完成后，需要用其结果继续操作时，无需等待。可以直接通过 `thenAccept`、`thenApply`、`thenCompose` 等方式将前面异步处理的结果交给另外一个异步事件处理线程来处理。