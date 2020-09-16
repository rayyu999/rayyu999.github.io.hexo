---
title: Java多线程
date: 2020-09-16 19:54:09
categories: Coding
tags: Java
---

----

<!--more-->

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
  Future<?> submit(Runnable task);	// 获取线程池仲的某一个线程对象，并执行
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