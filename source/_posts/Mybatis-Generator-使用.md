---
title: Mybatis Generator 使用
date: 2020-11-20 22:11:17
categories: Coding
tags: [Mybatis, Maven]
---



MyBatis Generator工具用于将数据库中的表反转成对应的 Domain、XML文件 和 对应的 DAO  Java文件。

<!--more-->



## 引入依赖

在 pom.xml 中添加下面的配置：

```xml
<dependency>
	<groupId>org.mybatis.generator</groupId>
	<artifactId>mybatis-generator-core</artifactId>
	<version>1.4.0</version>
</dependency>
```



## 编辑配置文件

在项目目录下新建 mbg.xml 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <context id="DB2Tables" targetRuntime="MyBatis3">
        <!-- 配置注释的生成，这里配置不生成注释 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!-- 配置数据库连接 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/ssm_crud?serverTimezone=UTC&amp;characterEncoding=utf-8&amp;useSSL=false"
                        userId="root"
                        password="960914">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        
        <!-- 实体类生成配置 -->
        <javaModelGenerator targetPackage="cn.yuyingwai.crud.bean" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- mapper生成配置 -->
        <sqlMapGenerator targetPackage="mapper"  targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!-- dao类生成配置 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="cn.yuyingwai.crud.dao"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- 要生成的对应数据库中的表 -->
        <table tableName="tbl_emp" domainObjectName="Employee"></table>
        <table tableName="tbl_dept" domainObjectName="Department"></table>

    </context>
</generatorConfiguration>
```

* 数据库驱动要与mysql以及jdk的版本对应数据库连接要输入参数时，需要将 `&` 写成 `&amp;`；
* 生成配置中的 `targetPackage` 为需要生成的类或文件所在的包，`targetProject` 为需要生成的类或文件的路径。



## 通过编码和配置文件运行

在测试包下新建 MGBTest 类：

```java
public class MBGTest {

    public static void main(String[] args) throws IOException, XMLParserException, InvalidConfigurationException, SQLException, InterruptedException {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File("mbg.xml");	// 配置文件位置
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }

}
```

运行该类，即可生成对应的实体类以及mapper文件。

![](http://images.yingwai.top/picgo/20201119103520.png)

