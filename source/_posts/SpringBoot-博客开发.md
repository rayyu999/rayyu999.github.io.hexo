---
title: SpringBoot 博客开发
date: 2020-12-02 23:18:15
categories: Coding
tags: [Spring Boot, AdminLTE3, Java]
---



使用 Spring Boot 开发一个小而美的博客系统。

<!--more-->



## 登录模块



### 登录流程设计

登录的本质即身份验证和登录状态的保持，编码实现：

首先，在数据库中查询这条用户记录；

* 如果不存在这条记录则表示身份验证失败，登录流程终止；
* 如果存在这条记录，则表示身份验证成功，接下来则需要进行登录状态的存储和验证了。

用户登录成功后我们将用户信息放到 session 对象中，之后再实现一个拦截器，在访问项目时判断 session 中是否有用户信息，有则放行请求，没有就跳转到登录页面。



### AdminLTE3 模板整合

整合过程其实是把 AdminLTE3 代码压缩包中我们需要的样式文件、js 文件、图片等静态资源放入 Spring Boot 项目的静态资源目录下，比如 static 目录或者其他我们设置的静态资源目录。

这里将对应的文件放在 static 目录下的 admin 文件夹中：

![](http://images.yingwai.top/picgo/20201202222317.png)



### 实现

#### 前端页面

##### 登录页面

由于选用了 AdminLTE3 作为模板，就直接改造其登录页面即可，在 templates/admin 目录中新建 login.html 模板页面，模板引擎我们选择的是 Thymeleaf，代码如下：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>personal blog | Log in</title>
    <!-- Tell the browser to be responsive to screen width -->
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="shortcut icon" th:href="@{/admin/dist/img/favicon.png}" />
    <!-- Font Awesome -->
    <link rel="stylesheet" th:href="@{/admin/dist/css/font-awesome.min.css}" />
    <!-- Ionicons -->
    <link rel="stylesheet" th:href="@{/admin/dist/css/ionicons.min.css}" />
    <!-- Theme style -->
    <link rel="stylesheet" th:href="@{/admin/dist/css/adminlte.min.css}" />
    <style>
        canvas {
            display: block;
            vertical-align: bottom;
        }
        #particles {
            background-color: #f7fafc;
            position: absolute;
            top: 0;
            width: 100%;
            height: 100%;
            z-index: -1;
        }
    </style>
</head>
<body class="hold-transition login-page">
<div id="particles"></div>
<div class="login-box">
    <div class="login-logo" style="color: #007bff;">
        <h1>personal blog</h1>
    </div>
    <!-- /.login-logo -->
    <div class="card">
        <div class="card-body login-card-body">
            <p class="login-box-msg">your personal blog , enjoy it</p>
            <form th:action="@{/admin/login}" method="post">
                <div
                        th:if="${not #strings.isEmpty(session.errorMsg)}"
                        class="form-group"
                >
                    <div
                            class="alert alert-danger"
                            th:text="${session.errorMsg}"
                    ></div>
                </div>
                <div class="form-group has-feedback">
                    <span class="fa fa-user form-control-feedback"></span>
                    <input
                            type="text"
                            id="userName"
                            name="userName"
                            class="form-control"
                            placeholder="请输入账号"
                            required="true"
                    />
                </div>
                <div class="form-group has-feedback">
                    <span class="fa fa-lock form-control-feedback"></span>
                    <input
                            type="password"
                            id="password"
                            name="password"
                            class="form-control"
                            placeholder="请输入密码"
                            required="true"
                    />
                </div>
                <div class="row">
                    <div class="col-6">
                        <input
                                type="text"
                                class="form-control"
                                name="verifyCode"
                                placeholder="请输入验证码"
                                required="true"
                        />
                    </div>
                    <div class="col-6">
                        <img
                                alt="单击图片刷新！"
                                class="pointer"
                                th:src="@{/common/kaptcha}"
                                onclick="this.src='/common/kaptcha?d='+new Date()*1"
                        />
                    </div>
                </div>
                <div class="form-group has-feedback"></div>
                <div class="row">
                    <div class="col-8"></div>
                    <div class="col-4">
                        <button
                                type="submit"
                                class="btn btn-primary btn-block btn-flat"
                        >
                            登录
                        </button>
                    </div>
                </div>
            </form>
        </div>
    </div>
</div>
<script th:src="@{/admin/plugins/jquery/jquery.min.js}"></script>
<!-- Bootstrap 4 -->
<script
        th:src="@{/admin/plugins/bootstrap/js/bootstrap.bundle.min.js}"
></script>
<script th:src="@{/admin/dist/js/plugins/particles.js}"></script>
<script th:src="@{/admin/dist/js/plugins/login-bg-particles.js}"></script>
</body>
</html>
```

该页面时直接修改的 AdminLTE3 模板的登录页，将文案修改为中文，并微调了一下页面布局，同时增加了验证码的设计。

用户在输入账号、密码和验证码后，点击登录按钮后将会向后端发送登录请求，请求地址为 admin/login，请求类型为 post，在 form 表单中已经定义了登陆的请求路径：

```html
<form th:action="@{/admin/login}" method="post"></form>
```



##### 登录后的跳转页面

在同目录下新建 index.html 文件：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<header th:replace="admin/header::header-fragment"></header>
<body class="hold-transition sidebar-mini">
<div class="wrapper">
    <!-- 引入页面头header-fragment -->
    <div th:replace="admin/header::header-nav"></div>
    <aside th:fragment="sidebar-fragment(path)" class="main-sidebar sidebar-dark-primary elevation-4">
        <!-- Brand Logo -->
        <a th:href="@{/admin/index}" class="brand-link">
            <img th:src="@{/admin/dist/img/logo.png}" alt="ssm-cluster Logo" class="brand-image img-circle elevation-3"
                 style="opacity: .8">
            <span class="brand-text font-weight-light">my blog</span>
        </a>
        <!-- Sidebar -->
        <div class="sidebar">
            <!-- Sidebar user panel (optional) -->
            <div class="user-panel mt-3 pb-3 mb-3 d-flex">
                <div class="image">
                    <img th:src="@{/admin/dist/img/avatar5.png}" class="img-circle elevation-2" alt="User Image">
                </div>
                <div class="info">
                    <a href="#" class="d-block" th:text="${session.loginUser}"></a>
                </div>
            </div>
            <!-- Sidebar Menu -->
            <nav class="mt-2">
                <ul class="nav nav-pills nav-sidebar flex-column" data-widget="treeview" role="menu"
                    data-accordion="false">
                    <!-- Add icons to the links using the .nav-icon class
                         with font-awesome or any other icon font library -->
                    <li class="nav-header">首页</li>
                    <li class="nav-item">
                        <a th:href="@{/admin/index}" class="nav-link active">
                            <i class="nav-icon fa fa-dashboard"></i>
                            <p>
                                首页
                            </p>
                        </a>
                    </li>
                    <li class="nav-header">系统管理</li>
                    </li>
                </ul>
            </nav>
            <!-- /.sidebar-menu -->
        </div>
        <!-- /.sidebar -->
    </aside>
    <!-- Content Wrapper. Contains page content -->
    <div class="content-wrapper">
        <!-- Content Header (Page header) -->
        <div class="content-header">
            <div class="container-fluid">
            </div><!-- /.container-fluid -->
        </div>
        <!-- Main content -->
        <div class="content">
            <div class="container-fluid">
                <div class="card-header">
                    <h3 class="card-title">管理首页</h3>
                </div> <!-- /.card-body -->
                <div class="row" style="margin-top: 40px;border-top:0px;">
                    My Blog 后台管理系统首页
                </div>
            </div><!-- /.container-fluid -->
        </div>
        <!-- /.content -->
    </div>
    <!-- /.content-wrapper -->
    <!-- 引入页脚footer-fragment -->
    <div th:replace="admin/footer::footer-fragment"></div>
</div>
<!-- jQuery -->
<script th:src="@{/admin/plugins/jquery/jquery.min.js}"></script>
<!-- jQuery UI 1.11.4 -->
<script th:src="@{/admin/plugins/jQueryUI/jquery-ui.min.js}"></script>
<!-- Bootstrap 4 -->
<script th:src="@{/admin/plugins/bootstrap/js/bootstrap.bundle.min.js}"></script>
<!-- AdminLTE App -->
<script th:src="@{/admin/dist/js/adminlte.min.js}"></script>
</body>
</html>
```

这里引入的页头和页脚也要新建：

header.html：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:fragment="header-fragment">
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>my personal blog | 后台管理系统</title>
    <!-- Tell the browser to be responsive to screen width -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- Font Awesome -->
    <link rel="shortcut icon" th:href="@{/admin/dist/img/favicon.png}"/>
    <link rel="stylesheet" th:href="@{/admin/dist/css/font-awesome.min.css}">
    <!-- Ionicons -->
    <link rel="stylesheet" th:href="@{/admin/dist/css/ionicons.min.css}">
    <link rel="stylesheet" th:href="@{/admin/dist/css/main.css}">
    <link rel="stylesheet" th:href="@{/admin/plugins/bootstrap/css/bootstrap.css}"/>
    <link rel="stylesheet" th:href="@{/admin/plugins/sweetalert/sweetalert.css}"/>
    <link rel="stylesheet" th:href="@{/admin/plugins/jqgrid-5.3.0/ui.jqgrid-bootstrap4.css}"/>
    <!-- Theme style -->
    <link rel="stylesheet" th:href="@{/admin/dist/css/adminlte.min.css}">
</head>
<!-- Navbar -->
<nav class="main-header navbar navbar-expand bg-white navbar-light border-bottom" th:fragment="header-nav">
    <!-- Left navbar links -->
    <ul class="navbar-nav">
        <li class="nav-item">
            <a class="nav-link" data-widget="pushmenu" href="#"><i class="fa fa-bars"></i></a>
        </li>
        <li class="nav-item d-none d-sm-inline-block">
            <a th:href="@{/admin/index}" class="nav-link">Dashboard</a>
        </li>
    </ul>
    <!-- Right navbar links -->
    <ul class="navbar-nav ml-auto">
        <li class="nav-item dropdown">
            <a class="nav-link" data-toggle="dropdown" href="#">
                <i class="fa fa-user">&nbsp;&nbsp;作者</i>
            </a>
            <div class="dropdown-menu dropdown-menu-lg dropdown-menu-right">
                <div class="dropdown-divider"></div>
                <a href="#" class="dropdown-item">
                    <i class="fa fa-user-o mr-2"></i> 姓名
                    <span class="float-right text-muted text-sm">十三 / 13</span>
                </a>
                <div class="dropdown-divider"></div>
                <a href="#" class="dropdown-item">
                    <i class="fa fa-user-secret mr-2"></i> 身份
                    <span class="float-right text-muted text-sm">Java开发工程师</span>
                </a>
                <div class="dropdown-divider"></div>
                <a href="#" class="dropdown-item">
                    <i class="fa fa-address-card mr-2"></i> 邮箱
                    <span class="float-right text-muted text-sm">2449207463@qq.com</span>
                </a>
            </div>
        </li>
    </ul>
</nav>
<!-- /.navbar -->
</html>
```

footer.html：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<footer class="main-footer" th:fragment="footer-fragment">
    <strong>Copyright &copy; 2018 <a href="##">13blog.site</a>.</strong>
    All rights reserved.
    <div class="float-right d-none d-sm-inline-block">
        <b>my personal blog #Version</b> 1.0
    </div>
</footer>
</html>
```



#### 表结构设计

博客系统正式的用户模块表结构设计如下：

```sql
CREATE DATABASE /*!32312 IF NOT EXISTS*/`my_blog_db` /*!40100 DEFAULT CHARACTER SET utf8 */;
USE `my_blog_db`;
/*Table structure for table `tb_admin_user` */
DROP TABLE IF EXISTS `tb_admin_user`;
CREATE TABLE `tb_admin_user` (
  `admin_user_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '管理员id',
  `login_user_name` varchar(50) NOT NULL COMMENT '管理员登陆名称',
  `login_password` varchar(50) NOT NULL COMMENT '管理员登陆密码',
  `nick_name` varchar(50) NOT NULL COMMENT '管理员显示昵称',
  `locked` tinyint(4) DEFAULT '0' COMMENT '是否锁定 0未锁定 1已锁定无法登陆',
  PRIMARY KEY (`admin_user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*Data for the table `tb_admin_user` */
insert  into `tb_admin_user`(`admin_user_id`,`login_user_name`,`login_password`,`nick_name`,`locked`) values (1,'admin','e10adc3949ba59abbe56e057f20f883e','十三',0);
```

新增了一张表，并在表中新增了一条用户数据，之后我们在演示登陆功能时会用到。



#### 后端功能实现

##### 添加管理员用户实体类

```java
package cn.yuyingwai.springbootblog.entity;

import lombok.Data;

@Data
public class AdminUser {

    private Integer adminUserId;

    private String loginUserName;

    private String loginPassword;

    private String nickName;

    private Byte locked;

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(getClass().getSimpleName());
        sb.append(" [");
        sb.append("Hash = ").append(hashCode());
        sb.append(", adminUserId=").append(adminUserId);
        sb.append(", loginUserName=").append(loginUserName);
        sb.append(", loginPassword=").append(loginPassword);
        sb.append(", nickName=").append(nickName);
        sb.append(", locked=").append(locked);
        sb.append("]");
        return sb.toString();
    }

}
```



##### 添加mybatis以及kaptcha(用于生成验证码)的依赖

pom.xml：

```xml
<!-- mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
<!-- 验证码 -->
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```



##### DAO层

AdminUserDao.java：

```java
package cn.yuyingwai.springbootblog.dao;

import cn.yuyingwai.springbootblog.entity.AdminUser;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.springframework.stereotype.Component;

@Mapper
@Component
public interface AdminUserDao {

    int insert(AdminUser record);

    int insertSelective(AdminUser record);

    /**
     * 登陆方法
     * @param userName
     * @param password
     * @return
     */
    AdminUser login(@Param("userName") String userName, @Param("password") String password);

}
```

这里的 `login()` 方法就是根据从前端接受的用户名以及密码，到数据库中查找对比，下面是对应的 xml 映射文件：

AdminUserMapper.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.yuyingwai.springbootblog.dao.AdminUserDao">
    <resultMap id="BaseResultMap" type="cn.yuyingwai.springbootblog.entity.AdminUser">
        <id column="admin_user_id" jdbcType="INTEGER" property="adminUserId" />
        <result column="login_user_name" jdbcType="VARCHAR" property="loginUserName" />
        <result column="login_password" jdbcType="VARCHAR" property="loginPassword" />
        <result column="nick_name" jdbcType="VARCHAR" property="nickName" />
        <result column="locked" jdbcType="TINYINT" property="locked" />
    </resultMap>
    <sql id="Base_Column_List">
    admin_user_id, login_user_name, login_password, nick_name, locked
  </sql>

    <select id="login" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List" />
        from tb_admin_user
        where login_user_name = #{userName,jdbcType=VARCHAR} AND login_password=#{password,jdbcType=VARCHAR} AND locked = 0
    </select>
    
</mapper>
```



##### Service层

首先定义一个接口规定方法：

```java
package cn.yuyingwai.springbootblog.service;

import cn.yuyingwai.springbootblog.entity.AdminUser;

public interface AdminUserService {

    AdminUser login(String userName, String password);

}
```

然后是该接口的实现类：

```java
package cn.yuyingwai.springbootblog.service.impl;

import cn.yuyingwai.springbootblog.dao.AdminUserDao;
import cn.yuyingwai.springbootblog.entity.AdminUser;
import cn.yuyingwai.springbootblog.service.AdminUserService;
import cn.yuyingwai.springbootblog.util.MD5Util;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class AdminUserServiceImpl implements AdminUserService {

    @Resource
    private AdminUserDao adminUserDao;

    @Override
    public AdminUser login(String userName, String password) {
        String passwordMd5 = MD5Util.MD5Encode(password, "UTF-8");
        return adminUserDao.login(userName, passwordMd5);
    }
}
```

因为密码在数据库中是以密文的形式存储的，所以这里 `login()` 方法首先将收到的密码用工具类进行加密后再交给持久层进行比对。

**密文工具类** MD5Util.java：

```java
package cn.yuyingwai.springbootblog.util;

import java.security.MessageDigest;

public class MD5Util {

    private static String byteArrayToHexString(byte b[]) {
        StringBuffer resultSb = new StringBuffer();
        for (int i = 0; i < b.length; i++)
            resultSb.append(byteToHexString(b[i]));

        return resultSb.toString();
    }

    private static String byteToHexString(byte b) {
        int n = b;
        if (n < 0)
            n += 256;
        int d1 = n / 16;
        int d2 = n % 16;
        return hexDigits[d1] + hexDigits[d2];
    }

    public static String MD5Encode(String origin, String charsetname) {
        String resultString = null;
        try {
            resultString = new String(origin);
            MessageDigest md = MessageDigest.getInstance("MD5");
            if (charsetname == null || "".equals(charsetname))
                resultString = byteArrayToHexString(md.digest(resultString.getBytes()));
            else
                resultString = byteArrayToHexString(md.digest(resultString.getBytes(charsetname)));
        } catch (Exception exception) {
        }
        return resultString;
    }

    private static final String hexDigits[] = {"0", "1", "2", "3", "4", "5",
            "6", "7", "8", "9", "a", "b", "c", "d", "e", "f"};

}
```



##### Controller层

```java
package cn.yuyingwai.springbootblog.controller.admin;

import cn.yuyingwai.springbootblog.entity.AdminUser;
import cn.yuyingwai.springbootblog.service.AdminUserService;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.annotation.Resource;
import javax.servlet.http.HttpSession;

@Controller
@RequestMapping("/admin")
public class AdminController {

    @Resource
    private AdminUserService adminUserService;

    @GetMapping({"/login"})
    public String login() {
        return "admin/login";
    }

    @PostMapping(value = "/login")
    public String login(@RequestParam("userName") String userName,
                        @RequestParam("password") String password,
                        @RequestParam("verifyCode") String verifyCode,
                        HttpSession session) {
        if (StringUtils.isEmpty(verifyCode)) {
            session.setAttribute("errorMsg", "验证码不能为空");
            return "admin/login";
        }
        if (StringUtils.isEmpty(userName) || StringUtils.isEmpty(password)) {
            session.setAttribute("errorMsg", "用户名或密码不能为空");
            return "admin/login";
        }
        String kaptchaCode = session.getAttribute("verifyCode") + "";
        if (StringUtils.isEmpty(kaptchaCode) || !verifyCode.equals(kaptchaCode)) {
            session.setAttribute("errorMsg", "验证码错误");
            return "admin/login";
        }
        AdminUser adminUser = adminUserService.login(userName, password);
        if (adminUser != null) {
            session.setAttribute("loginUser", adminUser.getNickName());
            session.setAttribute("loginUserId", adminUser.getAdminUserId());
            // session过期时间设置为7200秒 即两小时
            session.setMaxInactiveInterval(60 * 60 * 2);
            return "redirect:/admin/index";
        } else {
            session.setAttribute("errorMsg", "登陆失败");
            return "admin/login";
        }
    }

    @GetMapping({"", "/", "/index", "/index.html"})
    public String index() {
        return "admin/index";
    }

    @GetMapping("/header")
    public String header() {
        return "admin/header";
    }

    @GetMapping("/footer")
    public String footer() {
        return "admin/footer";
    }

}
```



##### 使用 Kaptcha 生成验证码

###### Kaptcha 设置

在 `controller.common` 包下新建 KaptchaConfig.java：

```java
package cn.yuyingwai.springbootblog.controller.common;

import com.google.code.kaptcha.impl.DefaultKaptcha;
import com.google.code.kaptcha.util.Config;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.util.Properties;

@Component
public class KaptchaConfig {

    @Bean
    public DefaultKaptcha getDefaultKaptcha() {
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        Properties properties = new Properties();
        // 图片边框
        properties.put("kaptcha.border", "no");
        // 字体颜色
        properties.put("kaptcha.textproducer.font.color", "black");
        // 图片宽
        properties.put("kaptcha.image.width", "160");
        // 图片高
        properties.put("kaptcha.image.height", "40");
        // 字体大小
        properties.put("kaptcha.textproducer.font.size", "30");
        // 验证码长度
        properties.put("kaptcha.textproducer.char.space", "5");
        // 字体
        properties.setProperty("kaptcha.textproducer.font.names", "宋体,楷体,微软雅黑");

        Config config = new Config(properties);
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }

}
```

这里主要是对于生成的验证码的一些设置。



###### 为前端页面提供验证码图片

在设置好后，还需要为前端页面提供验证码，需要与前端请求的路径对应：

```java
package cn.yuyingwai.springbootblog.controller.common;

import com.google.code.kaptcha.impl.DefaultKaptcha;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

import javax.imageio.ImageIO;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.awt.image.BufferedImage;
import java.io.ByteArrayOutputStream;

@Controller
public class CommonController {

    @Autowired
    private DefaultKaptcha captchaProducer;

    @GetMapping("/common/kaptcha")
    public void defaultKaptcha(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        byte[] captchaOutputStream;
        ByteArrayOutputStream imgOutputStream = new ByteArrayOutputStream();
        try {
            // 生成验证码字符串并保存到session中
            String verifyCode = captchaProducer.createText();
            httpServletRequest.getSession().setAttribute("verifyCode", verifyCode);
            BufferedImage challenge = captchaProducer.createImage(verifyCode);
            ImageIO.write(challenge, "jpg", imgOutputStream);
        } catch (IllegalArgumentException e) {
            httpServletResponse.sendError(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        captchaOutputStream = imgOutputStream.toByteArray();
        httpServletResponse.setHeader("Cache-Control", "no-store");
        httpServletResponse.setHeader("Pragma", "no-cache");
        httpServletResponse.setDateHeader("Expires", 0);

        httpServletResponse.setContentType("image/jpeg");
        ServletOutputStream responseOutputStream = httpServletResponse.getOutputStream();

        responseOutputStream.write(captchaOutputStream);
        responseOutputStream.flush();
        responseOutputStream.close();
    }

}
```


