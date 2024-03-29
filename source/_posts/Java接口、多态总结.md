---
title: Java接口、多态总结
date: 2020-08-24 00:08:43
categories: Coding
tags: Java
---

----

<!--more-->



## 接口

接口是功能的集合，同样可看做是一种数据类型，是比抽象类更为抽象的”类”。
接口只描述所应该具备的方法，并没有具体实现，具体的实现由接口的实现类(相当于接口的子类)来完成。这样将功能的定义与实现分离，优化了程序设计。
请记住：一切事物均有功能，即一切事物均有接口。

接口的出现避免了单继承的局限性。父类中定义的事物的基本功能。接口中定义的事物的扩展功能。



### 接口的特点

1. 定义一个接口用 `interface` 关键字：

   ```java
   public interface 接口名 {
       抽象方法1;
       抽象方法2;
   }
   ```

2. 一个类实现一个接口，实现 `implements` 关键字：

   ```java
   class 类名 implements 接口名 {
       
   }
   ```

3. 接口不能直接创建对象

   通过多态的方式，由子类来创建对象，接口多态。



### 接口中的成员特点

* 成员变量：

  只能是 `final` 修饰的常量

  默认修饰符：`public static final`

* 构造方法：

  无

* 成员方法：

  只能是抽象方法

  默认修饰符：`public abstract`



### 类与类、类与接口、接口与接口之间的关系

* 类与类：

  继承关系，单继承，可以是多层继承

* 类与接口：

  实现关系，单实现，也可以多实现（因为没有方法冲突的问题）

* 接口与接口：

  继承关系，单继承，也可以是多继承

Java中的类可以继承一个父类的同时，实现多个接口



## 多态

多态是继封装、继承之后，面向对象的第三大特性。

现实事物经常会体现出多种形态，如学生，学生是人的一种，则一个具体的同学张三既是学生也是人，即出现两种形态。

Java作为面向对象的语言，同样可以描述一个事物的多种形态。如Student类继承了Person类，一个Student的对象便既是Student，又是Person。

Java中多态的代码体现在一个子类对象(实现类对象)既可以给这个子类(实现类对象)引用变量赋值，又可以给这个子类(实现类对象)的父类(接口)变量赋值。

如Student类可以为Person类的子类。那么一个Student对象既可以赋值给一个Student类型的引用，也可以赋值给一个Person类型的引用。

最终多态体现为父类引用变量可以指向子类对象。

多态的前提是必须有子父类关系或者类实现接口关系，否则无法完成多态。

在使用多态后的父类引用变量调用方法时，会调用子类重写后的方法。



### 多态使用的前提

1. 有继承或者实现关系
2. 要方法重写
3. 父类引用指向子类对象



### 多态的成员访问特点

方法的运行看右边（子类），其它都看左边（父类）。



### 多态的利弊

好处：提高了程序的扩展性

弊端：不能访问子类特有的功能



### 多态的分类

* 类的多态

  ```java
  abstract class Fu {
      public abstract void method();
  }
  class Zi extends Fu {
  	public void method(){
  		System.out.println(“重写父类抽象方法”);
  	}
  }
  //类的多态使用
  Fu fu= new Zi();
  ```

* 接口的多态

  ```java
  interface Fu {
  	public abstract void method();
  }
  class Zi implements Fu {
  	public void method(){
          System.out.println(“重写接口抽象方法”);
  }
  }
  //接口的多态使用
  Fu fu = new Zi();
  ```



## `instanceof` 关键字

格式：`对象名 instanceof 类名`

返回值：`true, false`

作用：判断指定的对象是否为给定类创建的对象

