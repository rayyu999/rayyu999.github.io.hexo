---
title: Java常用API
date: 2020-08-31 17:35:36
categories: Coding
tags: Java
---

----

<!--more-->



## Object类

Object类是所有类的超类，祖宗类。java仲所有的类都直接或间接地继承这个类

### 方法

```java
public String toString() // 返回当前对象中的内容, 对于Object类默认操作来说，返回的对象的类型+@+内存地址值
public boolean equals(Object obj) // 比较两个对象内容是否相同，对于Object类默认操作来说,比较的是地址值
```



## String类

字符串类，字符串是常量；它们的值在创建之后不能更改

### 方法

```java
boolean equals(Object obj) // 判断两个字符串中的内容是否相同
boolean equalsIgnoreCase(String str)  // 判断两个字符串中的内容是否相同, 忽略大小写
boolean contains(String str) // 判断该字符串中 是否包含给定的字符串
boolean startsWith(String str) // 判断该字符串 是否以给定的字符串开头
boolean endsWith(String str) // 判断该字符串 是否以给定的字符串结尾
boolean isEmpty() // 判断该字符串的内容是否为空的字符串  ""
    
int length() // 获取该字符串的长度
char charAt(int index) // 获取该字符串中指定位置上的字符 
String substring(int start) // 从指定位置开始，到末尾结束，截取该字符串，返回新字符串
String substring(int start,int end) // 从指定位置开始，到指定位置结束，截取该字符串，返回新字符串 
int indexOf(int ch ) // 获取给定的字符，在该字符串中第一次出现的位置
int indexOf(String str) // 获取给定的字符串，在该字符串中第一次出现的位置
int indexOf(int ch,int fromIndex) // 从指定位置开始，获取给定的字符，在该字符
    
byte[] getBytes() // 把该字符串 转换成 字节数组
char[] toCharArray() // 把该字符串 转换成 字符数组
String replace(char old,char new) // 在该字符串中，将给定的旧字符，用新字符替换
String replace(String old,String new) // 在该字符串中， 将给定的旧字符串，用新字符串替换
String trim() // 去除字符串两端空格，中间的不会去除，返回一个新字符串
String toLowerCase() // 把该字符串转换成 小写字符串 
String toUpperCase() // 把该字符串转换成 大写字符串
int indexOf(String str,int fromIndex) // 从指定位置开始，获取给定的字符串，在该字符串中第一次出现的位置
```



## StringBuffer/StringBuilder类

### 方法

```java
public StringBuffer append(String str) // 在原有字符串缓冲区内容基础上，在末尾追加新数据
public StringBuffer insert(int offset,String str) // 在原有字符串缓冲区内容基础上，在指定位置插入新数据
public StringBuffer deleteCharAt(int index) // 在原有字符串缓冲区内容基础上，删除指定位置上的字符
public StringBuffer delete(int start,int end) // 在原有字符串缓冲区内容基础上，删除指定范围内的多个字符
public StringBuffer replace(int start,int end,String str) // 在原有字符串缓冲区内容基础上，将指定范围内的多个字符 用给定的字符串替换
    
public StringBuffer reverse() // 将字符串缓冲区的内容 反转  "abc"----"cba"
public String substring(int start) // 从指定位置开始，到末尾结束，截取该字符串缓冲区，返回新字符串
public String substring(int start,int end)  // 从指定位置开始，到指定位置结束，截取该字符串缓冲区，返回新字符串
```



## 正则表达式

用来定义匹配规则，匹配一系列符合某个句法规则的字符串。

### 正则表达式的匹配规则

| 字符 | 含义                 | 例子                                                         |
| :--- | -------------------- | ------------------------------------------------------------ |
| `x`  | 代表字符 `x`         | 匹配规则为 `"a"`，那么需要匹配的字符串内容就是 `"a"`         |
| `\\` | 代表反斜线字符 `'\'` | 匹配规则为 `"\\"` ，那么需要匹配的字符串内容就是 `"\"`       |
| `\t` | 制表符               | 匹配规则为 `"\t"`，那么对应的效果就是产生一个制表符的空间    |
| `\n` | 换行符               | 匹配规则为 `"\n"`，那么对应的效果就是换行，光标在原有位置的下一行 |
| `\r` | 回车符               | 匹配规则为 `"\r"`，那么对应的效果就是回车后的效果，光标来到下一行行首 |

| 字符类         | 含义                                              | 例子                                                         |
| -------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| `[abc]`        | 代表字符 `a, b` 或 `c`                            | 匹配规则为 `"[abc]"`，那么需要匹配的内容就是字符 `a` 或者字符 `b` 或者字符 `c` 中的一个 |
| `[^abc]`       | 代表除了字符 `a, b` 或 `c` 以外的任何字符         | 匹配规则为 `"[^abc]"`，那么需要匹配的内容就是不是字符 `a` 或者字符 `b` 或者字符 `c` 的任意一个字符 |
| `[a-zA-z]`     | 代表 `a` 到 `z` 或 `A` 到 `Z`，两头的字母包括在内 | 匹配规则为 `"[a-zA-Z]"`，那么需要匹配的是一个大写或者小写字母 |
| `[0-9]`        | 代表 `0` 到 `9` 数字，两头的数字包括在内          | 匹配规则为 `"[0-9]"`，那么需要匹配的是一个数字               |
| `[a-zZ-Z_0-9]` | 代表字符或者数字或者下划线（即单词字符）          | 匹配规则为 `"[a-zZ-Z_0-9]"`，那么需要匹配的是一个字符或者是一个数字或一个下划线 |

| 预定义字符类 | 含义                                                | 例子                                                         |
| ------------ | --------------------------------------------------- | ------------------------------------------------------------ |
| `.`          | 代表任何字符                                        | 匹配规则为 `"."`，那么需要匹配的是一个任意字符。如果只想匹配字符 `.` 的话，使用匹配规则 `"\\."` 来实现 |
| `\d`         | 代表0到9数字，相当于 `[0-9]`                        | 匹配规则为 `"\d"`，那么需要匹配的是一个数字                  |
| `\w`         | 代表的字母或者数字或者下划线，相当于 `[a-zA-Z_0-9]` | 匹配规则为 `\w`，那么需要匹配的是一个字母或者是一个数字或一个下划线 |

| 边界匹配器 | 含义         | 例子                                                         |
| ---------- | ------------ | ------------------------------------------------------------ |
| `^`        | 代表行的开头 | 匹配规则为 `^[abc][0-9]$`，那么需要匹配的内容从 `[abc]` 这个位置开始，相当于左双引号 |
| `$`        | 代表行的结尾 | 匹配规则为 `^[abc][0-9]$`，那么需要匹配的内容以 `[0-9]` 这个结束，相当于右双引号 |
| `\b`       | 代表单词边界 | 匹配规则为 `"\b[abc]\b"`，那么代表的是字母 `a` 或 `b` 或 `c` 的左右两边需要的是非单词字符（`[a-zA-Z_0-9]`） |

| 数量词   | 含义                                    | 例子                                                         |
| -------- | --------------------------------------- | ------------------------------------------------------------ |
| `X?`     | 代表 `X` 出现一次或一次也没有           | 匹配规则为 `"a?"`，那么需要匹配的内容是多个字符 `a`，或者一个 `a` 都没有 |
| `X*`     | 代表 `X` 出现零次或多次                 | 匹配规则为 `"a*"`，那么需要匹配的内容是多个字符 `a`，或者一个 `a` 都没有 |
| `X+`     | 代表 `X` 出现一次或多次                 | 匹配规则为 `"a+"`，那么需要匹配的内容是多个字符 `a`，或一个 `a` |
| `X{n}`   | 代表 `X` 恰好出现 n 次                  | 匹配规则为 `"a{5}"`，那么需要匹配的内容是5个字符 `a`         |
| `X{n,}`  | 代表 `X` 至少出现 n 次                  | 匹配规则为 `"a{5,}"`，那么需要匹配的内容是最少有5个字符 `a`  |
| `X{n,m}` | 代表 `X` 至少出现 n 次，但是不超过 m 次 | 匹配规则为 `"a{5,8}"`，那么需要匹配的内容是5到8个字符 `a`    |



### 正则表达式的常用方法

```java
public boolean matches(String regex) // 判断字符是否匹配给定的规则
public String[] split(String regex) // 根据给定的正则表达式的匹配规则，拆分此字符串
public String replaceAll(String regex,String replacement)	// 将符合规则的字符串内容，全部替换为新字符串 
```



## Date类

日期/时间类。

### 构造方法

```java
public Date() // 系统当前日期时间
public Date(long date) // 得到距离 1970年1月1日0点 date毫秒的日期时间
```



### 方法

```java
public long getTime() // 获取日期所对应的毫秒值
```



## DateFormat类

是日期/时间格式化子类的抽象类，使用其子类 `SimpledateFormat` 实例化

### 构造方法

```java
public SimpleDateFormat() // 默认的格式化操作
public SimpleDateFormat(String pattern) // 按照指定的格式，进行日期初始化
```

**日期和时间模式：**

| 符号 | 含义                   |
| ---- | ---------------------- |
| `y`  | 年                     |
| `M`  | 年中的月份             |
| `d`  | 月份中的天数           |
| `H`  | 一天中的小时数（0-23） |
| `m`  | 小时中的分钟数         |
| `s`  | 分钟中的秒数           |
| `S`  | 毫秒数                 |

### 方法

```java
public final String format(Date date) // 把日期格式化成字符串
public Date parse(String source) // 把日期字符串转换成日期对象
```



## Calendar类

日历类，可获取日期中指定字段的值

### 方法

```java
public static Calendar getInstance() // 获取日期对象
public int get(int field)	// 获取时间字段值
public void add(int field,int amount)	// 指定字段增加某值
public final void set(int field,int value) // 设置指定字段的值
public final Date getTime()	// 获取该日历对象转成的日期对象
```



## 基本类型包装类

### 8种基本类型对应的包装类

| 基本类型 | 包装类    |
| -------- | --------- |
| byte     | Byte      |
| short    | Short     |
| int      | Integer   |
| long     | Long      |
| float    | Float     |
| double   | Double    |
| char     | Character |
| boolean  | Boolean   |



### 自动装箱、自动拆箱

* 自动装箱：基本数值转成对象（`int -> Integer`）
* 自动拆箱：对象转成基本数值（`Integer -> int`）



### 常用方法

```java
public int parseInt(String str)		// 把字符串转成基本类型int
public static String toString(int x)	// 把基本类型int转成字符串
public static Integer valueOf(int x)	// 把基本类型i字符串转成Integer对象
public int intValue()	// 以int类型返回该包装类对象的值
```



## System类

系统属性信息工具类

### 方法

```java
public static long currentTimeMillis()	// 获取当前系统时间与1970年01月01日00:00点之间的毫秒差值
public static void exit(int status)		// 用来结束正在运行的Java程序。参数传入一个数字即可。通常传入0记为正常状态，其他为异常状态
public static void gc()		// 用来运行JVM中的垃圾回收器，完成内存中垃圾的清除。
public static String getProperties()	// 用来获取指系统属性信息
```



## Arrays类

数组操作工具类

### 方法

```java
public static void sort()	// 用来对指定数组中的元素进行排序（元素值从小到大进行排序）
public static String toString()		// 用来返回指定数组元素内容的字符串形式
public static void binarySearch(要找的元素)	// 在指定数组中，查找给定元素值出现的位置。若没有查询到，返回位置为-插入点-1。要求该数组必须是个有序的数组
```



## Math类

数学运算工具类

### 方法

* `abs(int a)`方法,结果都为正数
* `ceil(int a)`方法，结果为比参数值大的最小整数的double值
* `floor(int a)`方法，结果为比参数值小的最大整数的double值
* `max(int a, int b)`方法，返回两个参数值中较大的值
* `min(int a, int b)`方法，返回两个参数值中较小的值
* `pow(int a, int p)`方法，返回第一个参数的第二个参数次幂的值
* `round(int a)`方法，返回参数值四舍五入的结果
* `random()`方法，产生一个大于等于0.0且小于1.0的double小数

