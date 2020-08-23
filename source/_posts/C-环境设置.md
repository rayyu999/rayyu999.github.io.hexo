---
title: C++环境设置
date: 2020-06-28 21:42:53
categories: Env
tags: [C++]
---

----



<!--more-->

# 安装GNU的C/C++编译器

## UNIX/Linux上的安装

首先在命令行使用以下命令确认系统是否安装了GCC：

```shell
$ g++ -v
```

如果系统已经安装了GNU编译器，会显示如下的消息：

```shell
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
Target: x86_64-redhat-linux
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
Thread model: posix
gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
```



若未安装，可以使用以下三条命令进行安装：

```shell
$ yum install gcc
$ yum install gcc-c++
$ yum install gdb
```



## Windows上的安装

首先访问www.mingw.org，右上方进入下载页面，下载最新版本的MinGW安装程序。

![](http://images.yingwai.top/picgo/gccf1.png)

安装完成后将 MinGW\bin 路径添加到系统环境变量里。

然后打开 MinGW Installation Manager，点击All Packages把想要安装的Package选中或点击Basic Setup选择mingw32-gcc-g++-bin：

![](http://images.yingwai.top/picgo/gccf2.png)

选择完后点击菜单栏Installation中的Apply changes选项，如果出现某种原因安装未能成功，选择review changes选项重新安装。

完成安装后可以在命令行中输入 `g++ -v`、 `gcc -v` 和 `mingw32-make -v` 检查是否安装成功。



# g++ 应用说明

helloworld.cpp 中包含如下简单代码：

```c++
#include <iostream>
using namespace std;
int main(){
    cout << "Hello, world!" << endl;
    return 0;
}
```

最简单的编译方式为：

```shell
$ g++ helloworld.cpp
```

由于命令行中未指定可执行程序的文件名，编译器采用默认的 a.out。程序可以这样来运行：

```shell
$ ./a.out
Hello, world!
```

通常我们使用 `-o` 选项指定可执行程序的文件名，以下实例生成一个 helloworld 的可执行文件：

```shell
$ g++ helloworld.cpp -o helloworld
```

执行 helloworld：

```shell
$ ./helloworld
Hello, world!
```

如果是多个 C++ 代码文件，如 ray1.cpp、ray2.cpp，编译命令如下：

```shell
$ g++ ray1.cpp ray2.cpp -o ray
```

生成一个 ray 可执行文件。



## g++ 常用命令选项

https://www.runoob.com/cplusplus/cpp-environment-setup.html

| 选项          | 解释                                                         |
| ------------- | ------------------------------------------------------------ |
| `-ansi`       | 只支持 ANSI 标准的 C 语法。这一选项将禁止 GNU C 的某些特色， 例如 asm 或 typeof 关键词。 |
| `-c`          | 只编译并生成目标文件。                                       |
| `-DMACRO`     | 以字符串"1"定义 MACRO 宏。                                   |
| `-E`          | 只运行 C 预编译器。                                          |
| `-g`          | 生成调试信息。GNU 调试器可利用该信息。                       |
| `-IDIRECTORY` | 指定额外的头文件搜索路径DIRECTORY。                          |
| `-LDIRECTORY` | 指定额外的函数库搜索路径DIRECTORY。                          |
| `-lLIBRARY`   | 连接时搜索指定的函数库LIBRARY。                              |
| `-m486`       | 针对 486 进行代码优化。                                      |
| `-o`          | FILE 生成指定的输出文件。用在生成可执行文件时。              |
| `-O0`         | 不进行优化处理。                                             |
| `-O`          | 或 `-O1` 优化生成代码。                                      |
| `-O2`         | 进一步优化。                                                 |
| `-O3`         | 比 `-O2` 更进一步优化，包括 inline 函数。                    |
| `-shared`     | 生成共享目标文件。通常用在建立共享库时。                     |
| `-static`     | 禁止使用共享连接。                                           |
| `-UMACRO`     | 取消对 MACRO 宏的定义。                                      |
| `-w`          | 不生成任何警告信息。                                         |
| `-Wall`       | 生成所有警告信息。                                           |

