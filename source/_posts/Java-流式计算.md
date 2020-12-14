---
title: Java 流式计算
date: 2020-12-14 22:14:58
categories: Coding
tags: [Java, Stream]
---



![](http://images.yingwai.top/picgo/20201210171601.png)

<!--more-->

## 函数式接口

### java.util.function

![](http://images.yingwai.top/picgo/20201210171817.bmp)



### java 内置核心四大函数式接口

|            函数式接口            | 参数类型 | 返回类型  |                             用途                             |
| :------------------------------: | :------: | :-------: | :----------------------------------------------------------: |
|  `Consumer<T>`<br />消费型接口   |   `T`    |  `void`   |    对类型为T的对象应用操作，包含方法：`void accept(T t);`    |
|  `Supplier<T>`<br />供给型接口   |    无    |    `T`    |         返回类型为 `T` 的对象，包含方法：`T get();`          |
| `Function<T, R>`<br />函数型接口 |   `T`    |    `R`    | 对类型为 `T` 的对象应用操作，并返回结果。结果是 `R` 类型的对象。包含方法：`R apply(T t);` |
|  `Predicate<T>`<br />断定型接口  |   `T`    | `boolean` | 确定类型为 `T` 的对象是否满足某约束，并返回 `boolean` 值。包含方法：`boolean test(T t);` |



### 实例

```java
//R apply(T t);函数型接口，一个参数，一个返回值
Function<String,Integer> function = t -> { return t.length(); };
System.out.println(function.apply("abcd"));

//boolean test(T t);断定型接口，一个参数，返回boolean
Predicate<String> predicate = t -> { return t.startsWith("a"); };
System.out.println(predicate.test("a"));

// void accept(T t);消费型接口，一个参数，没有返回值
Consumer<String> consumer = t -> {
    System.out.println(t);
};
consumer.accept("javaXXXX");

//T get(); 供给型接口，无参数，有返回值
Supplier<String> supplier = () -> { return UUID.randomUUID().toString(); };
System.out.println(supplier.get());
```



## Stream 流

### 流（Stream）是什么

是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。

> 集合讲的是数据，流讲的是计算！



### 特点

* Stream 自己不会存储元素
* Stream 不会改变源对象。相反，他们会返回一个持有结果的新 Stream
* Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行



### Stream 使用

1. 创建一个 Stream：一个**数据源**（数组、集合）
2. 中间操作：一个中间操作，**处理**数据源数据
3. 终止操作：一个终止操作，**执行**中间操作链，产生结果

*源头 => 中间流水线 => 结果*



### StreamDemo

```java
/**
 * 按照给出数据，找出同时满足
 * 偶数ID且年龄大于24且用户名转为大写且用户名名字倒排序
 * 最后只能给出一个用户名字
 */
public class StreamDemo {

    public static void main(String[] args) {
        User u1 = new User(11, "a", 23);
        User u2 = new User(12, "b", 24);
        User u3 = new User(13, "c", 22);
        User u4 = new User(14, "d", 28);
        User u5 = new User(15, "e", 26);

        List<User> list = Arrays.asList(u1, u2, u3, u4, u5);
        
        list.stream().filter(u -> {
            return u.getId() % 2 == 0;
        }).filter(u -> {
            return u.getAge() > 24;
        }).map(u -> {
            return u.getUserName().toUpperCase();
        }).sorted((u1, u2) -> {
            return u2.compareTo(u1);
        }).limit(1).forEach(System.out::println);
    }
    
}
```

