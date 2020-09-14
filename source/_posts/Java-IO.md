---
title: Java IO
date: 2020-09-14 22:12:58
categories: Coding
tags: Java
---

----

<!--more-->

## File

文件和目录路径名的抽象表示形式

### 构造方法

```java
public File(String pathname) // 通过给定的文件或文件夹的路径，来创建对应的File对象
public File(String parent, String child) // 通过给定的父文件夹路径，与给定的文件名称或目录名称来创建对应的File对象
public File(File parent, String child)	// 通过给定的File对象的目录路径，与给定的文件夹名称或文件名称来创建对应的File对象
```



### 路径的分类

* 绝对路径，带盘符

  `E:\Workspace\day20_File\abc.txt`

* 相对路径，不带盘符

  `day20_File\abc.txt`

* 注意：当指定一个文件路径的时候，如果采用的是相对路径，默认的目录为项目的根目录



### 方法

* ```java
  public boolean createNewFile()	// 创建文件
  ```

  * 返回值为 `true`，说明创建文件成功
  * 返回值为 `false`，说明文件已存在，创建文件失败

* ```java
  public boolean mkdir()	// 创建单层文件夹
  ```

  * 创建文件夹成功，返回 `true`
  * 创建文件夹失败，返回 `false`

* ```java
  public boolean mkdirs()	// 创建多层文件夹
  ```

* ```java
  public boolean delete()
  ```

  * 删除此抽象路径名表示的文件或目录
  * 如果此路径名表示一个目录，则该目录必须为空才能删除

* ```java
  public boolean isDirectory()	// 判断是否为文件夹
  ```

* ```java
  public boolean isFile()	// 判断是否为文件
  ```

* ```java
  public boolean exists()	// 判断File对象对应的文件或文件夹是否存在
  ```

* ```java
  public String getAbsolutePath()	// 获取当前File的绝对路径
  ```

* ```java
  public String getName()	// 获取当前File对象的文件或文件夹名称
  ```

* ```java
  public long length()	// 获取当前File对象的文件或文件夹的大小（字节）
  ```

* ```java
  public File[] listFiles()	// 获取File所代表目录中所有文件或文件夹的绝对路径
  ```



## IO 流

### IO 流分类

```java
|- 字节流
	|- 字节输入流 InputStream 抽象类
		|- FileInputStream 操作文件的字节输入流
    	|- BufferedInputStream 缓冲输入流
    |- 字节输出流 OutputStream 抽象类
    	|- FileOutputStream 操作文件的字节输出流
    	|- BufferedOutputStream 缓冲输出流
|- 字符流
    |- 字符输入流 Reader 抽象类
    	|- InputStreamReader 输入操作的转换流
    		|- FileReader 用来操作文件的字符输入流（简便的流）
    	|- BufferedReader 缓冲字符输入流
    |- 字符输出流 Writer 抽象类
    	|- OutputStreamWriter 输出操作的转换流
    		|- FileWriter 用来操作文件的字符输出流（简便的流）
    	|- BufferedWriter 缓冲字符输出流
```



### 流的操作规律

IO流中对象很多，这里对解决问题（处理设备上的数据时）到底该用哪个对象进行了总结：

* 明确要操作的数据是数据源还是数据目的。

  源：`InputStream`、`Reader`

  目的：`OutputStream`、`Writer`

  先根据需求明确要读，还是要写。

* 明确要操作的数据是字节还是文本。

  源：

  * 字节：`InputStream`
  * 文本：`Reader`

  目的：

  * 字节：`OutputStream`
  * 文本：`Writer`

  已经明确了具体的体系上。

* 明确数据所在的具体设备。

  源设备：

  * 硬盘：文件，`File` 开头
  * 内存：数组，字符串
  * 键盘：`System.in`
  * 网络：`Socket`

  目的设备：

  * 硬盘：文件，`File` 开头
  * 内存：数组，字符串
  * 屏幕：`System.out`
  * 网络：`Socket`

  完全可以明确具体要使用哪个流对象。

* 明确是否需要额外功能

  额外功能：

  * 转换流：`InputStreamReader`、`OutputStreamWriter`
  * 缓冲区对象：`BufferedXXX`



### 方法

#### 读数据方法

* ```java
  read()	// 一次读一个字节或字符的方法
  ```

* ```java
  read(byte[]  char[])	// 一次读一个数组数据的方法
  ```

* ```java
  readLine()	// 一次读一行字符串的方法（BufferedReader类特有方法）
  ```

* ```java
  readObject()	// 从流中读取对象（ObjectInputStream特有方法）
  ```



#### 写数据方法

* ```java
  write(int)	// 一次写一个字节或字符到文件中
  ```

* ```java
  write(byte[] char[])	// 一次写一个数组数据到文件中
  ```

* ```java
  write(String)	// 一次写一个字符串到文件中
  ```

* ```java
  writeObject(Object)	// 写对象到流中（ObjectOutputStream类特有方法）
  ```

* ```java
  newLine()	// 写一个换行符号（BufferedWriter类特有方法）
  ```



### 向文件中写入数据的过程

1. 创建输出流对象
2. 写数据到文件
3. 关闭输出流



### 从文件中读数据的过程

1. 创建输入流对象
2. 从文件中读数据
3. 关闭输入流



### 文件复制的过程

1. 创建输入流（数据源）
2. 创建输出流（目的地）
3. 从输入流仲读数据
4. 通过输出流，把数据写入目的地
5. 关闭流



## Properties

Map 集合中的一种，它是 Hashtable 集合的子集合，它的键与值都是 String 类型，它是唯一能与 IO 流结合使用的集合。



### 方法

* ```java
  load(InputStream in)	// 从流所对应的文件中，读数据到集合中
  ```

* ```java
  load(Reader in)		// 从流所对应的文件中，读数据到集合中
  ```

* ```java
  store(OutputStream out, String message)	// 把集合中的数据，写入到流所对应的文件中
  ```

* ```java
  store(Writer out, String message)	// 把集合中的数据，写入到流所对应的文件中
  ```



## 实现文件内容的自动追加

* 构造方法

  ```java
  FileOutputStream(File file, boolean append)
  ```

  ```java
  FileOutputStream(String fileName, boolean append)
  ```

  ```java
  FileWriter(File file, boolean append)
  ```

  ```java
  FileWriter(String fileName, boolean append)
  ```



## 实现文件内容的自动刷新

* 构造方法

  ```java
  PrintStream(OutputStream out, boolean autoFlush)
  ```

  ```java
  PrintWriter(OutputStream out, boolean autoFlush)
  ```

  ```java
  PrintWriter(Writer out, boolean autoFlush)
  ```



## Commons-IO

### 方法

* ```java
  readFileToString(File file)	// 读取文件内容，并返回一个String
  ```

* ```java
  writeStringToFile(File file, String content)	// 将内容content写入到file中
  ```

* ```java
  copyDirectoryToDirectory(File srcDir, File destDir)	// 文件夹复制
  ```

* ```java
  copyFileToDirectory(File srcFile, File destFile)	// 文件复制
  ```



