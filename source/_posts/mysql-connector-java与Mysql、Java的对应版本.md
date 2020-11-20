---
title: mysql-connector-java与Mysql、Java的对应版本
date: 2020-11-20 22:12:58
categories: Env
tags: [Mysql, java]
---

-----

<!--more-->



## mysql-connector-java与Mysql对应版本

| **Connector/J version** | **Driver Type** | **JDBC version**   | **MySQL Server version** | **Status**                                |
| ----------------------- | --------------- | ------------------ | ------------------------ | ----------------------------------------- |
| 5.1                     | 4               | 3.0, 4.0, 4.1, 4.2 | 5.6*, 5.7*, 8.0*         | General availability                      |
| 8.0                     | 4               | 4.2                | 5.6, 5.7, 8.0            | General availability. Recommended version |

* MySQL Connector/J 8.0 is highly recommended for use with MySQL Server 8.0, 5.7, and 5.6. Please upgrade to MySQL Connector/J 8.0.
* 官方更推荐MySQL5.6以上使用connector/j 8.0



### mysql-connector-java与Java对应版本

| **Connector/J version** | **JRE Supported**             | **JDK required for compiling source code** |
| ----------------------- | ----------------------------- | ------------------------------------------ |
| 5.1                     | 1.5.x, 1.6.x, 1.7.x*, 1.8.x** | 1.5.x and 1.8.x ***                        |
| 8.0                     | 1.8.x                         | 1.8.x                                      |

* JRE 1.7 support requires Connector/J 5.1.21 and higher.
* JRE 1.7 需要connector/J 5.1.21 以上