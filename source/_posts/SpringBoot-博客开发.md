---
title: Spring Boot 博客开发
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



### 登录拦截器

在上面实现了管理员的登陆功能，该功能已经完成，但是身份认证的整个流程并没有完善，该流程中应该包括登陆功能、身份认证、访问拦截、退出功能，我们仅仅完成了第一步，因此接下来将会对该流程进行完善。

#### 拦截器介绍

定义一个 Interceptor 非常简单方式也有几种，这里简单列举两种：

- 新建类要实现 Spring 的 HandlerInterceptor 接口
- 新建类继承实现了 HandlerInterceptor 接口的实现类，例如已经提供的实现了 HandlerInterceptor 接口的抽象类 HandlerInterceptorAdapter

HandlerInterceptor 方法介绍：

```java
    boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception;

    void postHandle(
            HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception;

    void afterCompletion(
            HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception;
```

- **preHandle**：在业务处理器处理请求之前被调用。预处理，可以进行编码、安全控制、权限校验等处理；
- **postHandle**：在业务处理器处理请求执行完成后，生成视图之前执行。
- **afterCompletion**：在 DispatcherServlet 完全处理完请求后被调用，可用于清理资源等，返回处理（已经渲染了页面）；



#### 定义拦截器

新建 intercepto 包，在包中新建 AdminLoginInterceptor 类，该类需要实现 HandlerInterceptor 接口，代码如下：

```java
package cn.yuyingwai.springbootblog.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 后台系统身份验证拦截器
 */
@Component
public class AdminLoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String uri = request.getRequestURI();
        if (uri.startsWith("/admin") && null == request.getSession().getAttribute("loginUser")) {
            request.getSession().setAttribute("errorMsg", "请登录");
            response.sendRedirect(request.getContextPath() + "/admin/login");
            return false;
        } else {
            request.getSession().removeAttribute("errorMsg");
            return true;
        }
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

`preHandle` 方法中读取当前 session 中是否存在 `loginUser` 对象，如果不存在则返回 false 并跳转至登录页面。



#### 配置拦截器

新建 config 包，之后新建 MyBlogWebMvcConfigurer 类并实现 WebMvcConfigurer 接口，代码如下：

```java
package cn.yuyingwai.springbootblog.config;

import cn.yuyingwai.springbootblog.interceptor.AdminLoginInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyBlogWebMvcConfigurer implements WebMvcConfigurer {
    
    @Autowired
    private AdminLoginInterceptor adminLoginInterceptor;
    
    public void addInterceptors(InterceptorRegistry registry) {
        // 添加一个拦截器，拦截以/admin为前缀的url路径
        registry.addInterceptor(adminLoginInterceptor)
                .addPathPatterns("/admin/**")
                .excludePathPatterns("/admin/login")
                .excludePathPatterns("/admin/dist/**")
                .excludePathPatterns("/admin/plugins/**");
    }
    
}
```

在该配置类中，添加刚刚新增的 AdminLoginInterceptor 登录拦截器，并对该拦截器所拦截的路径进行配置，由于后端管理系统的所有请求路径都以 /admin 开头，所以拦截的路径为 /admin/** ，但是登陆页面以及部分静态资源文件也是以 /admin 开头，所以需要将这些路径排除，配置如上。



## 用户模块

前面实现了登录功能，但用户模块不只是登录功能，还包括用户信息修改、安全退出的功能。

### 前端页面

#### 修改信息页面

在 resources/templates/admin/ 目录下新建 profile.html：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<header th:replace="admin/header::header-fragment"></header>
<body class="hold-transition sidebar-mini">
<div class="wrapper">
    <!-- 引入页面头header-fragment -->
    <div th:replace="admin/header::header-nav"></div>
    <!-- 引入工具栏sidebar-fragment -->
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
                        <a th:href="@{/admin/index}" class="nav-link">
                            <i class="nav-icon fa fa-dashboard"></i>
                            <p>
                                首页
                            </p>
                        </a>
                    </li>
                    <li class="nav-header">系统管理</li>
                    <li class="nav-item">
                        <a th:href="@{/admin/profile}" class="nav-link active">
                            <i class="fa fa-user-secret nav-icon"></i>
                            <p>修改密码</p>
                        </a>
                    </li>
                    <li class="nav-item">
                        <a th:href="@{/admin/logout}" class="nav-link">
                            <i class="fa fa-sign-out nav-icon"></i>
                            <p>安全退出</p>
                        </a>
                    </li>
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
        <section class="content">
            <div class="container-fluid">
                <div class="row">
                    <div class="col-md-6">
                        <div class="card card-primary card-outline">
                            <div class="card-header">
                                <h3 class="card-title">基本信息</h3>
                            </div> <!-- /.card-body -->
                            <div class="card-body">
                                <form role="form" id="userNameForm">
                                    <div class="form-group col-sm-8">
                                        <div class="alert alert-danger" id="updateUserName-info"
                                             style="display: none;"></div>
                                    </div>
                                    <!-- text input -->
                                    <div class="form-group">
                                        <label>登陆名称</label>
                                        <input type="text" class="form-control" id="loginUserName"
                                               name="loginUserName"
                                               placeholder="请输入登陆名称" required="true" th:value="${loginUserName}">
                                    </div>
                                    <div class="form-group">
                                        <label>昵称</label>
                                        <input type="text" class="form-control" id="nickName"
                                               name="nickName"
                                               placeholder="请输入昵称" required="true" th:value="${nickName}">
                                    </div>
                                    <div class="card-footer">
                                        <button type="button" id="updateUserNameButton" onsubmit="return false;"
                                                class="btn btn-danger float-right">确认修改
                                        </button>
                                    </div>
                                </form>
                            </div><!-- /.card-body -->
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="card card-primary card-outline">
                            <div class="card-header">
                                <h3 class="card-title">修改密码</h3>
                            </div> <!-- /.card-body -->
                            <div class="card-body">
                                <form role="form" id="userPasswordForm">
                                    <div class="form-group col-sm-8">
                                        <div class="alert alert-danger updatePassword-info" id="updatePassword-info"
                                             style="display: none;"></div>
                                    </div>
                                    <!-- input states -->
                                    <div class="form-group">
                                        <label class="control-label"><i class="fa fa-key"></i> 原密码</label>
                                        <input type="text" class="form-control" id="originalPassword"
                                               name="originalPassword"
                                               placeholder="请输入原密码" required="true">
                                    </div>
                                    <div class="form-group">
                                        <label class="control-label"><i class="fa fa-key"></i> 新密码</label>
                                        <input type="text" class="form-control" id="newPassword" name="newPassword"
                                               placeholder="请输入新密码" required="true">
                                    </div>
                                    <div class="card-footer">
                                        <button type="button" id="updatePasswordButton" onsubmit="return false;"
                                                class="btn btn-danger float-right">确认修改
                                        </button>
                                    </div>
                                </form>
                            </div><!-- /.card-body -->
                        </div>
                    </div>
                </div>
            </div><!-- /.container-fluid -->
        </section>
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
<!-- public.js -->
<script th:src="@{/admin/dist/js/public.js}"></script>
<!-- profile -->
<script th:src="@{/admin/dist/js/profile.js}"></script>
</body>
</html>
```

#### 登录后的跳转页面

同时要在 index.html 中的 `<li class="nav-header">系统管理</li>` 下面添加如下代码：

```html
<li class="nav-item">
    <a th:href="@{/admin/profile}" class="nav-link">
        <i class="fa fa-user-secret nav-icon"></i>
        <p>修改密码</p>
    </a>
</li>
<li class="nav-item">
    <a th:href="@{/admin/logout}" class="nav-link">
        <i class="fa fa-sign-out nav-icon"></i>
        <p>安全退出</p>
    </a>
</li>
```



### 后端实现

#### AdminController

* 用户主页面：

  ```java
  @GetMapping("/profile")
  public String profile(HttpServletRequest request) {
      Integer loginUserId = (int) request.getSession().getAttribute("loginUserId");
      AdminUser adminUser = adminUserService.getUserDetailById(loginUserId);
      if (adminUser == null) {
          return "admin/login";
      }
      request.setAttribute("path", "profile");
      request.setAttribute("loginUserName", adminUser.getLoginUserName());
      request.setAttribute("nickName", adminUser.getNickName());
      return "admin/profile";
  }
  ```

  上面的代码主要是将用户的信息返回给前端动态显示出来；

* 修改密码：

  ```java
  @PostMapping("/profile/password")
  @ResponseBody
  public String passwordUpdate(HttpServletRequest request,
                               @RequestParam("originalPassword") String originalPassword,
                               @RequestParam("newPassword") String newPassword) {
      if (StringUtils.isEmpty(originalPassword) || StringUtils.isEmpty(newPassword)) {
          return "参数不能为空";
      }
      Integer loginUserId = (int) request.getSession().getAttribute("loginUserId");
      if (adminUserService.updatePassword(loginUserId, originalPassword, newPassword)) {
          // 修改成功后清空session中的数据，前端控制跳转至登录页
          request.getSession().removeAttribute("loginUserId");
          request.getSession().removeAttribute("loginUser");
          request.getSession().removeAttribute("errorMsg");
          return "success";
      } else {
          return "修改失败";
      }
  }
  ```

* 修改昵称：

  ```java
  @PostMapping("/profile/name")
  @ResponseBody
  public String nameUpdate(HttpServletRequest request,
                           @RequestParam("loginUserName") String loginUserName,
                           @RequestParam("nickName") String nickName) {
      if (StringUtils.isEmpty(loginUserName) || StringUtils.isEmpty(nickName)) {
          return "参数不能为空";
      }
      Integer loginUserId = (int) request.getSession().getAttribute("loginUserId");
      if (adminUserService.updateName(loginUserId, loginUserName, nickName)) {
          return "success";
      } else {
          return "修改失败";
      }
  }
  ```

* 登出：

  ```java
  @GetMapping("/logout")
      public String logout(HttpServletRequest request) {
          request.getSession().removeAttribute("loginUserId");
          request.getSession().removeAttribute("loginUser");
          request.getSession().removeAttribute("errorMsg");
          return "admin/login";
      }
  ```

#### AdminUserService

添加以下方法：

```java
/**
 * 获取用户信息
 * @param loginUserId
 * @return
 */
AdminUser getUserDetailById(Integer loginUserId);

/**
 * 修改当前登录用户密码
 * @param loginUserId
 * @param originalPassword
 * @param newPassword
 * @return
 */
Boolean updatePassword(Integer loginUserId, String originalPassword, String newPassword);

/**
 * 修改当前登录用户的名称信息
 * @param loginUserId
 * @param loginUserName
 * @param nickName
 * @return
 */
Boolean updateName(Integer loginUserId, String loginUserName, String nickName);
```

##### AdminUserServiceImpl

```java
@Override
public AdminUser getUserDetailById(Integer loginUserId) {
    return adminUserDao.selectByPrimaryKey(loginUserId);
}

@Override
public Boolean updatePassword(Integer loginUserId, String originalPassword, String newPassword) {
    AdminUser adminUser = adminUserDao.selectByPrimaryKey(loginUserId);
    // 当前用户非空才可以进行更改
    if (adminUser != null) {
        String originalPasswordMd5 = MD5Util.MD5Encode(originalPassword, "UTF-8");
        String newPasswordMd5 = MD5Util.MD5Encode(newPassword, "UTF-8");
        // 比较原密码是否正确
        if (originalPasswordMd5.equals(adminUser.getLoginPassword())) {
            // 设置新密码并修改
            adminUser.setLoginPassword(newPasswordMd5);
            if (adminUserDao.updateByPrimaryKeySelective(adminUser) > 0) {
                // 修改成功则返回true
                return true;
            }
        }
    }
    return false;
}

@Override
public Boolean updateName(Integer loginUserId, String loginUserName, String nickName) {
    AdminUser adminUser = adminUserDao.selectByPrimaryKey(loginUserId);
    // 当前用户非空才可以进行更改
    if (adminUser != null) {
        // 设置新密码并修改
        adminUser.setLoginUserName(loginUserName);
        adminUser.setNickName(nickName);
        if (adminUserDao.updateByPrimaryKeySelective(adminUser) > 0) {
            // 修改成功则返回true
            return true;
        }
    }
    return false;
}
```

#### AdminUserDao

添加以下方法：

```java
AdminUser selectByPrimaryKey(Integer adminUserId);

int updateByPrimaryKeySelective(AdminUser record);
```

#### AdminUserMapper

```java
<select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from tb_admin_user
    where admin_user_id = #{adminUserId,jdbcType=INTEGER}
</select>
<update id="updateByPrimaryKeySelective" parameterType="cn.yuyingwai.springbootblog.entity.AdminUser">
        update tb_admin_user
    <set>
    <if test="loginUserName != null">
        login_user_name = #{loginUserName,jdbcType=VARCHAR},
</if>
    <if test="loginPassword != null">
        login_password = #{loginPassword,jdbcType=VARCHAR},
</if>
    <if test="nickName != null">
        nick_name = #{nickName,jdbcType=VARCHAR},
</if>
    <if test="locked != null">
        locked = #{locked,jdbcType=TINYINT},
</if>
    </set>
    where admin_user_id = #{adminUserId,jdbcType=INTEGER}
</update>
```



## 分类模块

### 持久层相关

#### 表结构设计

在进行接口设计和具体的功能实现前，首先将表结构确定下来，每篇文章都会被归类到一个类别下，一个类别下会有多篇文章，分类实体与文章实体的关系是一对多的关系，因此在表结构设计时，在文章表中设置一个分类关联字段即可，分类表只需要将分类相关的字段定义好，分类实体与文章实体的关系交给文章表来维护即可（后续讲到文章表时再介绍），分类表的 SQL 设计如下，直接执行如下 SQL 语句即可：

```sql
USE `my_blog_db`;

/*Table structure for table `tb_blog_category` */

CREATE TABLE `tb_blog_category` (
  `category_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '分类表主键',
  `category_name` varchar(50) NOT NULL COMMENT '分类的名称',
  `category_icon` varchar(50) NOT NULL COMMENT '分类的图标',
  `category_rank` int(11) NOT NULL DEFAULT '1' COMMENT '分类的排序值 被使用的越多数值越大',
  `is_deleted` tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否删除 0=否 1=是',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`category_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



#### BlogCategory 实体类

```java
package cn.yuyingwai.springbootblog.entity;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import java.util.Date;

@Data
public class BlogCategory {

    private Integer categoryId;

    private String categoryName;

    private String categoryIcon;

    private Integer categoryRank;

    private Byte isDeleted;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createTime;

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(getClass().getSimpleName());
        sb.append(" [");
        sb.append("Hash = ").append(hashCode());
        sb.append(", categoryId=").append(categoryId);
        sb.append(", categoryName=").append(categoryName);
        sb.append(", categoryIcon=").append(categoryIcon);
        sb.append(", categoryRank=").append(categoryRank);
        sb.append(", isDeleted=").append(isDeleted);
        sb.append(", createTime=").append(createTime);
        sb.append("]");
        return sb.toString();
    }

}
```



#### BlogCategoryDao.java

```java
package cn.yuyingwai.springbootblog.dao;

import cn.yuyingwai.springbootblog.entity.BlogCategory;
import cn.yuyingwai.springbootblog.util.PageQueryUtil;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

@Mapper
public interface BlogCategoryDao {

    int deleteByPrimaryKey(Integer categoryId);

    int insert(BlogCategory record);

    int insertSelective(BlogCategory record);

    BlogCategory selectByPrimaryKey(Integer categoryId);

    BlogCategory selectByCategoryName(String categoryName);

    int updateByPrimaryKeySelective(BlogCategory record);

    int updateByPrimaryKey(BlogCategory record);

    List<BlogCategory> findCategoryList(PageQueryUtil pageUtil);

    List<BlogCategory> selectByCategoryIds(@Param("categoryIds") List<Integer> categoryIds);

    int getTotalCategories(PageQueryUtil pageUtil);

    int deleteBatch(Integer[] ids);

}
```



#### BlogCategoryMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.yuyingwai.springbootblog.dao.BlogCategoryDao">
    <resultMap id="BaseResultMap" type="cn.yuyingwai.springbootblog.entity.BlogCategory">
        <id column="category_id" jdbcType="INTEGER" property="categoryId"/>
        <result column="category_name" jdbcType="VARCHAR" property="categoryName"/>
        <result column="category_icon" jdbcType="VARCHAR" property="categoryIcon"/>
        <result column="category_rank" jdbcType="INTEGER" property="categoryRank"/>
        <result column="is_deleted" jdbcType="TINYINT" property="isDeleted"/>
        <result column="create_time" jdbcType="TIMESTAMP" property="createTime"/>
    </resultMap>
    <sql id="Base_Column_List">
        category_id, category_name, category_icon, category_rank, is_deleted, create_time
    </sql>
    <select id="findCategoryList" parameterType="Map" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from tb_blog_category
        where is_deleted=0
        order by category_rank desc,create_time desc
        <if test="start!=null and limit!=null">
            limit #{start},#{limit}
        </if>
    </select>

    <select id="getTotalCategories" parameterType="Map" resultType="int">
        select count(*)  from tb_blog_category
        where is_deleted=0
    </select>

    <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from tb_blog_category
        where category_id = #{categoryId,jdbcType=INTEGER} AND is_deleted = 0
    </select>
    <select id="selectByCategoryIds" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from tb_blog_category
        where category_id IN
        <foreach collection="categoryIds" item="item" index="index"
                 open="(" separator="," close=")">#{item}
        </foreach>
        AND is_deleted = 0
    </select>

    <select id="selectByCategoryName" parameterType="java.lang.String" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from tb_blog_category
        where category_name = #{categoryName,jdbcType=INTEGER} AND is_deleted = 0
    </select>
    <update id="deleteByPrimaryKey" parameterType="java.lang.Integer">
        UPDATE tb_blog_category SET  is_deleted = 1
        where category_id = #{categoryId,jdbcType=VARCHAR} AND is_deleted = 0
    </update>
    <update id="deleteBatch">
        update tb_blog_category
        set is_deleted=1 where category_id in
        <foreach item="id" collection="array" open="(" separator="," close=")">
            #{id}
        </foreach>
    </update>
    <insert id="insert" parameterType="cn.yuyingwai.springbootblog.entity.BlogCategory">
        insert into tb_blog_category (category_id, category_name, category_icon,
                                      category_rank, is_deleted, create_time
        )
        values (#{categoryId,jdbcType=INTEGER}, #{categoryName,jdbcType=VARCHAR}, #{categoryIcon,jdbcType=VARCHAR},
                #{categoryRank,jdbcType=INTEGER}, #{isDeleted,jdbcType=TINYINT}, #{createTime,jdbcType=TIMESTAMP}
               )
    </insert>
    <insert id="insertSelective" parameterType="cn.yuyingwai.springbootblog.entity.BlogCategory">
        insert into tb_blog_category
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="categoryId != null">
                category_id,
            </if>
            <if test="categoryName != null">
                category_name,
            </if>
            <if test="categoryIcon != null">
                category_icon,
            </if>
            <if test="categoryRank != null">
                category_rank,
            </if>
            <if test="isDeleted != null">
                is_deleted,
            </if>
            <if test="createTime != null">
                create_time,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="categoryId != null">
                #{categoryId,jdbcType=INTEGER},
            </if>
            <if test="categoryName != null">
                #{categoryName,jdbcType=VARCHAR},
            </if>
            <if test="categoryIcon != null">
                #{categoryIcon,jdbcType=VARCHAR},
            </if>
            <if test="categoryRank != null">
                #{categoryRank,jdbcType=INTEGER},
            </if>
            <if test="isDeleted != null">
                #{isDeleted,jdbcType=TINYINT},
            </if>
            <if test="createTime != null">
                #{createTime,jdbcType=TIMESTAMP},
            </if>
        </trim>
    </insert>
    <update id="updateByPrimaryKeySelective" parameterType="cn.yuyingwai.springbootblog.entity.BlogCategory">
        update tb_blog_category
        <set>
            <if test="categoryName != null">
                category_name = #{categoryName,jdbcType=VARCHAR},
            </if>
            <if test="categoryIcon != null">
                category_icon = #{categoryIcon,jdbcType=VARCHAR},
            </if>
            <if test="categoryRank != null">
                category_rank = #{categoryRank,jdbcType=INTEGER},
            </if>
            <if test="isDeleted != null">
                is_deleted = #{isDeleted,jdbcType=TINYINT},
            </if>
            <if test="createTime != null">
                create_time = #{createTime,jdbcType=TIMESTAMP},
            </if>
        </set>
        where category_id = #{categoryId,jdbcType=INTEGER}
    </update>
    <update id="updateByPrimaryKey" parameterType="com.site.blog.my.core.entity.BlogCategory">
        update tb_blog_category
        set category_name = #{categoryName,jdbcType=VARCHAR},
            category_icon = #{categoryIcon,jdbcType=VARCHAR},
            category_rank = #{categoryRank,jdbcType=INTEGER},
            is_deleted = #{isDeleted,jdbcType=TINYINT},
            create_time = #{createTime,jdbcType=TIMESTAMP}
        where category_id = #{categoryId,jdbcType=INTEGER}
    </update>
</mapper>
```



### 分页数据的格式及获取

分页使用到了 JqGrid 插件，在 JqGrid 整合中有如下代码：

```json
jsonReader: {
  root: "data.list", //数据列表模型
  page: "data.currPage", //当前页码
  total: "data.totalPage", //数据总页码
  records: "data.totalCount" //数据总记录数
  }
```

这里定义的是 jsonReader 对象如何对后端返回的 json 数据进行解析，比如数据列表为何读取 "data.list"，当前页码为何读取 "data.currPage"，这些都是由后端返回的数据格式所决定的，因此我们需要对后端返回的数据格式进行定义。

#### 定义后端响应结果的数据格式

```java
package cn.yuyingwai.springbootblog.util;

import lombok.Data;

import java.io.Serializable;

@Data
public class Result<T> implements Serializable {

    private static final long serialVersionUID = 1L;
    private int resultCode;
    private String message;
    private T data;

    public Result() {
    }

    public Result(int resultCode, String message) {
        this.resultCode = resultCode;
        this.message = message;
    }

    @Override
    public String toString() {
        return "Result{" +
                "resultCode=" + resultCode +
                ", message='" + message + '\'' +
                ", data=" + data +
                '}';
    }
    
}
```

#### 定义工具类生成响应结果

ResultGenerator.java 对各种生成响应结果的方法进行了封装：

```java
package cn.yuyingwai.springbootblog.util;

import org.springframework.util.StringUtils;

/**
 * 响应结果生成工具
 */
public class ResultGenerator {

    private static final String DEFAULT_SUCCESS_MESSAGE = "SUCCESS";
    private static final String DEFAULT_FAIL_MESSAGE = "FAIL";
    private static final int RESULT_CODE_SUCCESS = 200;
    private static final int RESULT_CODE_SERVER_ERROR = 500;

    public static Result genSuccessResult() {
        Result result = new Result();
        result.setResultCode(RESULT_CODE_SUCCESS);
        result.setMessage(DEFAULT_SUCCESS_MESSAGE);
        return result;
    }

    public static Result genSuccessResult(String message) {
        Result result = new Result();
        result.setResultCode(RESULT_CODE_SUCCESS);
        result.setMessage(message);
        return result;
    }

    public static Result genSuccessResult(Object data) {
        Result result = new Result();
        result.setResultCode(RESULT_CODE_SUCCESS);
        result.setMessage(DEFAULT_SUCCESS_MESSAGE);
        result.setData(data);
        return result;
    }

    public static Result genFailResult(String message) {
        Result result = new Result();
        result.setResultCode(RESULT_CODE_SERVER_ERROR);
        if (StringUtils.isEmpty(message)) {
            result.setMessage(DEFAULT_FAIL_MESSAGE);
        } else {
            result.setMessage(message);
        }
        return result;
    }

    public static Result genErrorResult(int code, String message) {
        Result result = new Result();
        result.setResultCode(code);
        result.setMessage(message);
        return result;
    }
    
}
```

#### 定义分页结果集的数据格式

```java
package cn.yuyingwai.springbootblog.util;

import lombok.Data;

import java.io.Serializable;
import java.util.List;

@Data
public class PageResult implements Serializable {

    //总记录数
    private int totalCount;
    //每页记录数
    private int pageSize;
    //总页数
    private int totalPage;
    //当前页数
    private int currPage;
    //列表数据
    private List<?> list;

    /**
     * 分页
     *
     * @param list       列表数据
     * @param totalCount 总记录数
     * @param pageSize   每页记录数
     * @param currPage   当前页数
     */
    public PageResult(List<?> list, int totalCount, int pageSize, int currPage) {
        this.list = list;
        this.totalCount = totalCount;
        this.pageSize = pageSize;
        this.currPage = currPage;
        this.totalPage = (int) Math.ceil((double) totalCount / pageSize);
    }

}
```

#### 对前端的请求参数进行封装

PageQueryUtil.java 将前端的请求参数封装到一个 map 中，方便持久层进行查询：

```java
package cn.yuyingwai.springbootblog.util;

import lombok.Data;

import java.util.LinkedHashMap;
import java.util.Map;

@Data
public class PageQueryUtil extends LinkedHashMap<String, Object> {

    //当前页码
    private int page;
    //每页条数
    private int limit;

    public PageQueryUtil(Map<String, Object> params) {
        this.putAll(params);

        //分页参数
        this.page = Integer.parseInt(params.get("page").toString());
        this.limit = Integer.parseInt(params.get("limit").toString());
        this.put("start", (page - 1) * limit);
        this.put("page", page);
        this.put("limit", limit);
    }

    @Override
    public String toString() {
        return "PageUtil{" +
                "page=" + page +
                ", limit=" + limit +
                '}';
    }

}
```



### 分类模块接口设计及实现

为了让页面体验更加友好，就不采用传统的 MVC 跳转模式，一个功能一个页面，这种交互感觉有些浪费，翻页的时候，翻一页跳转一次也比较繁琐，添加或者新增的时候也要进行页面跳转，所以这些功能的实现就采用通过 Ajax 异步与后端交互数据，当使用者点击了页面上的元素，此时触发响应的 js 事件，进而通过 Ajax 的方式向后端请求数据，前端再根据后端返回的数据内容去进行响应的展示逻辑，在前面的个人信息修改中其实用到的就是这种方式。

分类模块在后台管理系统中有5个接口，分别是：

- 分类列表分页接口
- 添加分类接口
- 根据 id 获取单条分类记录接口
- 修改分类接口
- 删除分类接口



#### 分类列表分页接口

列表接口负责接收前端传来的分页参数，如 `page` 、`limit` 等参数，之后将数据总数和对应页面的数据列表查询出来并封装为分页数据返回给前端。

##### 控制层代码

```java
@Controller
@RequestMapping("/admin")
public class CategoryController {

    @Resource
    private CategoryService categoryService;

    @GetMapping("/categories")
    public String categoryPage(HttpServletRequest request) {
        request.setAttribute("path", "categories");
        return "admin/category";
    }

    /**
     * 分类列表
     * @param params
     * @return
     */
    @RequestMapping(value = "/categories/list", method = RequestMethod.GET)
    @ResponseBody
    public Result list(@RequestParam Map<String, Object> params) {
        if (StringUtils.isEmpty(params.get("page")) || StringUtils.isEmpty(params.get("limit"))) {
            return ResultGenerator.genFailResult("参数异常！");
        }
        PageQueryUtil pageUtil = new PageQueryUtil(params);
        return ResultGenerator.genSuccessResult(categoryService.getBlogCategoryPage(pageUtil));
    }

}
```

##### 业务层代码

```java
@Service
public class CategoryServiceImpl implements CategoryService {

    @Autowired
    private BlogCategoryDao blogCategoryDao;

    /**
     * 利用PageQueryUtil封装的前端参数，获得当前页的分页数据，
     * 并将其封装为PageResult，方便前端的JqGrid使用
     * @param pageUtil
     * @return
     */
    @Override
    public PageResult getBlogCategoryPage(PageQueryUtil pageUtil) {
        List<BlogCategory> categoryList = blogCategoryDao.findCategoryList(pageUtil);
        int total = blogCategoryDao.getTotalCategories(pageUtil);
        PageResult pageResult = new PageResult(categoryList, total, pageUtil.getLimit(), pageUtil.getPage());
        return pageResult;
    }
    
   /**
     * 获得总分类数
     * @return
     */
    @Override
    public int getTotalCategories() {
        return blogCategoryDao.getTotalCategories(null);
    }
}
```



#### 添加分类接口

添加接口负责接收前端的 POST 请求并处理其中的参数，接收的参数为 categoryName 字段和 categoryIcon 字段，categoryName 为分类名称，categoryIcon 字段为分类的图标字段。

##### 控制层代码

在 CategoryController.java 中添加如下方法：

```java
    /**
     * 分类添加
     * @param categoryName
     * @param categoryIcon
     * @return
     */
    @RequestMapping(value = "/categories/save", method = RequestMethod.POST)
    @ResponseBody
    public Result save(@RequestParam("categoryName") String categoryName,
                       @RequestParam("categoryIcon") String categoryIcon) {
        if (StringUtils.isEmpty(categoryName)) {
            return ResultGenerator.genFailResult("请输入分类名称！");
        }
        if (StringUtils.isEmpty(categoryIcon)) {
            return ResultGenerator.genFailResult("请选择分类图标！");
        }
        if (categoryService.saveCategory(categoryName, categoryIcon)) {
            return ResultGenerator.genSuccessResult();
        } else {
            return ResultGenerator.genFailResult("分类名称重复");
        }
    }
```

##### 业务层代码

在 CategoryServiceImpl.java 中添加如下方法：

```java
    /**
     * 给定类名和类图标保存分类
     * @param categoryName
     * @param categoryIcon
     * @return
     */
    @Override
    public Boolean saveCategory(String categoryName, String categoryIcon) {
        BlogCategory temp = blogCategoryDao.selectByCategoryName(categoryName);
        if (temp == null) {
            BlogCategory blogCategory = new BlogCategory();
            blogCategory.setCategoryName(categoryName);
            blogCategory.setCategoryIcon(categoryIcon);
            return blogCategoryDao.insertSelective(blogCategory) > 0;
        }
        return false;
    }
```

添加接口中，首先会对参数进行校验，之后交给业务层代码进行操作，在 `saveCategory()` 方法中，首先会根据名称查询是否已经存在该分类，之后才会进行数据封装并进行数据库 insert 操作。



#### 删除分类接口

删除接口负责接收前端的分类删除请求，处理前端传输过来的数据后，将这些记录从数据库中删除，这里的“删除”功能并不是真正意义上的删除，而是逻辑删除，将接受的参数设置为一个数组，可以同时删除多条记录，只需要在前端将用户选择的记录 id 封装好再传参到后端即可。

##### 控制层代码

```java
    /**
     * 分类删除
     * @param ids
     * @return
     */
    @RequestMapping(value = "/categories/delete", method = RequestMethod.POST)
    @ResponseBody
    public Result delete(@RequestBody Integer[] ids) {
        if (ids.length < 1) {
            return ResultGenerator.genFailResult("参数异常！");
        }
        if (categoryService.deleteBatch(ids)) {
            return ResultGenerator.genSuccessResult();
        } else {
            return ResultGenerator.genFailResult("删除失败");
        }
    }
```

##### 业务层代码

```java
    /**
     * 根据id删除分类数据
     * @param ids
     * @return
     */
    @Override
    @Transactional
    public Boolean deleteBatch(Integer[] ids) {
        if (ids.length < 1) {
            return false;
        }
        //删除分类数据
        return blogCategoryDao.deleteBatch(ids) > 0;
    }
```

接口的请求路径为 /categories/delete，并使用 `@RequestBody` 将前端传过来的参数封装为 id 数组，参数验证通过后则调用 `deleteBatch()` 批量删除方法进行数据库操作，否则将向前端返回错误信息。



#### 其它

还有根据 id 获取详情的接口，路径为 categories/info/{id}，请求方法为 GET；分类修改接口，路径为 categories/update，请求方法为 POST。

##### 控制层代码

```java
    /**
     * 分类修改
     * @param categoryId
     * @param categoryName
     * @param categoryIcon
     * @return
     */
    @RequestMapping(value = "/categories/update", method = RequestMethod.POST)
    @ResponseBody
    public Result update(@RequestParam("categoryId") Integer categoryId,
                         @RequestParam("categoryName") String categoryName,
                         @RequestParam("categoryIcon") String categoryIcon) {
        if (categoryId == null || categoryId < 1) {
            return ResultGenerator.genFailResult("非法参数！");
        }
        if (StringUtils.isEmpty(categoryName)) {
            return ResultGenerator.genFailResult("请输入分类名称！");
        }
        if (StringUtils.isEmpty(categoryIcon)) {
            return ResultGenerator.genFailResult("请选择分类图标！");
        }
        if (categoryService.updateCategory(categoryId, categoryName, categoryIcon)) {
            return ResultGenerator.genSuccessResult();
        } else {
            return ResultGenerator.genFailResult("分类名称重复");
        }
    }

    /**
     * 详情
     * @param id
     * @return
     */
    @GetMapping("/categories/info/{id}")
    @ResponseBody
    public Result info(@PathVariable("id") Integer id) {
        if (id == null || id < 1) {
            return ResultGenerator.genFailResult("非法参数！");
        }
        BlogCategory category = categoryService.selectById(id);
        return ResultGenerator.genSuccessResult(category);
    }
```

##### 业务层代码

```java
    /**
     * 根据id，给定类名和类图标更新类别信息
     * @param categoryId
     * @param categoryName
     * @param categoryIcon
     * @return
     */
    @Override
    @Transactional
    public Boolean updateCategory(Integer categoryId, String categoryName, String categoryIcon) {
        BlogCategory blogCategory = blogCategoryDao.selectByPrimaryKey(categoryId);
        if (blogCategory != null) {
            blogCategory.setCategoryIcon(categoryIcon);
            blogCategory.setCategoryName(categoryName);
            return blogCategoryDao.updateByPrimaryKeySelective(blogCategory) > 0;
        }
        return false;
    }

    @Override
    public BlogCategory selectById(Integer id) {
        return blogCategoryDao.selectByPrimaryKey(id);
    }
```



### 前端页面实现

#### category.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<header th:replace="admin/header::header-fragment">
</header>
<body class="hold-transition sidebar-mini">
<div class="wrapper">
    <!-- 引入页面头header-fragment -->
    <div th:replace="admin/header::header-nav"></div>
    <!-- 引入工具栏sidebar-fragment -->
    <div th:replace="admin/sidebar::sidebar-fragment(${path})"></div>
    <!-- Content Wrapper. Contains 图标content -->
    <div class="content-wrapper">
        <!-- Content Header (图标header) -->
        <div class="content-header">
            <div class="container-fluid">
            </div><!-- /.container-fluid -->
        </div>
        <!-- Main content -->
        <div class="content">
            <div class="container-fluid">
                <div class="card card-primary card-outline">
                    <div class="card-header">
                        <h3 class="card-title">分类管理</h3>
                    </div> <!-- /.card-body -->
                    <div class="card-body">
                        <div class="grid-btn">
                            <button class="btn btn-info" onclick="categoryAdd()"><i
                                    class="fa fa-plus"></i>&nbsp;新增
                            </button>
                            <button class="btn btn-info" onclick="categoryEdit()"><i
                                    class="fa fa-pencil-square-o"></i>&nbsp;修改
                            </button>
                            <button class="btn btn-danger" onclick="deleteCagegory()"><i
                                    class="fa fa-trash-o"></i>&nbsp;删除
                            </button>
                        </div>
                        <br>
                        <table id="jqGrid" class="table table-bordered">
                        </table>
                        <div id="jqGridPager"></div>
                    </div><!-- /.card-body -->
                </div>
            </div><!-- /.container-fluid -->
        </div>
        <!-- /.content -->
        <div class="content">
            <!-- 模态框（Modal） -->
            <div class="modal fade" id="categoryModal" tabindex="-1" role="dialog" aria-labelledby="categoryModalLabel">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                        <div class="modal-header">
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span
                                    aria-hidden="true">&times;</span></button>
                            <h6 class="modal-title" id="categoryModalLabel">Modal</h6>
                        </div>
                        <div class="modal-body">
                            <form id="categoryForm" onsubmit="return false;">
                                <div class="form-group">
                                    <div class="alert alert-danger" id="edit-error-msg" style="display: none;">
                                        错误信息展示栏。
                                    </div>
                                </div>
                                <input type="hidden" class="form-control" id="categoryId" name="categoryId">
                                <div class="form-group">
                                    <label for="categoryName" class="control-label">分类名称:</label>
                                    <input type="text" class="form-control" id="categoryName" name="categoryName"
                                           placeholder="请输入分类名称" required="true">
                                </div>
                                <div class="form-group">
                                    <label for="categoryIcon" class="control-label">分类图标:</label>
                                    <input type="hidden" class="form-control" id="categoryIcon" name="categoryIcon">
                                    <div class="col-sm-4">
                                        <img id="categoryIconImg" src="/admin/dist/img/img-upload.png"
                                             style="height: 64px;width: 64px;">
                                        <button class="btn btn-secondary" style="margin-top: 5px;margin-bottom: 5px;"
                                                id="categoryIconButton"><i
                                                class="fa fa-random"></i>&nbsp;图标切换
                                        </button>
                                    </div>
                                </div>
                            </form>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-default" data-dismiss="modal">取消</button>
                            <button type="button" class="btn btn-primary" id="saveButton">确认</button>
                        </div>
                    </div>
                </div>
            </div>
            <!-- /.modal -->
        </div>
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
<!-- jqgrid -->
<script th:src="@{/admin/plugins/jqgrid-5.3.0/jquery.jqGrid.min.js}"></script>
<script th:src="@{/admin/plugins/jqgrid-5.3.0/grid.locale-cn.js}"></script>
<!-- sweetalert -->
<script th:src="@{/admin/plugins/sweetalert/sweetalert.min.js}"></script>
<script th:src="@{/admin/dist/js/public.js}"></script>
<script th:src="@{/admin/dist/js/category.js}"></script>
</body>
</html>
```

#### 功能按钮

分类管理模块也设计了常用的几个功能：分类信息增加、分类信息编辑、分类信息删除，因此在页面中添加对应的功能按钮以及触发事件，代码如下：

```html
<div class="grid-btn">
  <button class="btn btn-info" onclick="categoryAdd()">
    <i class="fa fa-plus"></i>&nbsp;新增
  </button>
  <button class="btn btn-info" onclick="categoryEdit()">
    <i class="fa fa-pencil-square-o"></i>&nbsp;修改
  </button>
  <button class="btn btn-danger" onclick="deleteCagegory()">
    <i class="fa fa-trash-o"></i>&nbsp;删除
  </button>
</div>
```

分别是添加按钮，对应的触发事件是 `categoryAdd()` 方法，修改按钮，对应的触发事件是 `categoryEdit()` 方法，删除按钮，对应的触发事件是 `deleteCagegory()` 方法。

#### 分页信息展示区域

页面中已经引入 JqGrid 的相关静态资源文件，需要在页面中展示分页数据的区域增加如下代码：

```html
<table id="jqGrid" class="table table-bordered"></table>
<div id="jqGridPager"></div>
```

此时只是静态效果展示，并没有与后端进行数据交互，接下来将结合 Ajax 和后端接口实现具体的功能。



### 分页模块前端功能实现

#### 分页功能

在 resources/static/admin/dist/js 目录下新增 category.js 文件，并添加如下代码：

```javascript
$(function () {
  $('#jqGrid').jqGrid({
    url: '/admin/categories/list',
    datatype: 'json',
    colModel: [
      {
        label: 'id',
        name: 'categoryId',
        index: 'categoryId',
        width: 50,
        key: true,
        hidden: true,
      },
      {
        label: '分类名称',
        name: 'categoryName',
        index: 'categoryName',
        width: 240,
      },
      {
        label: '分类图标',
        name: 'categoryIcon',
        index: 'categoryIcon',
        width: 120,
        formatter: imgFormatter,
      },
      {
        label: '添加时间',
        name: 'createTime',
        index: 'createTime',
        width: 120,
      },
    ],
    height: 560,
    rowNum: 10,
    rowList: [10, 20, 50],
    styleUI: 'Bootstrap',
    loadtext: '信息读取中...',
    rownumbers: false,
    rownumWidth: 20,
    autowidth: true,
    multiselect: true,
    pager: '#jqGridPager',
    jsonReader: {
      root: 'data.list',
      page: 'data.currPage',
      total: 'data.totalPage',
      records: 'data.totalCount',
    },
    prmNames: {
      page: 'page',
      rows: 'limit',
      order: 'order',
    },
    gridComplete: function () {
      //隐藏grid底部滚动条
      $('#jqGrid').closest('.ui-jqgrid-bdiv').css({ 'overflow-x': 'hidden' });
    },
  });

  $(window).resize(function () {
    $('#jqGrid').setGridWidth($('.card-body').width());
  });
});
```

以上代码的主要功能为分页数据展示、字段格式化 jqGrid DOM 宽度的自适应，在页面加载时，调用 JqGrid 的初始化方法，将页面中 id 为 jqGrid 的 DOM 渲染为分页表格，并向后端发送请求，之后按照后端返回的 json 数据填充表格以及表格下方的分页按钮。

#### 按钮事件及 Modal 框实现

添加和修改两个按钮分别绑定了触发事件，需要在 category.js 文件中新增 `categoryAdd()` 方法和 `categoryEdit()` 方法，两个方法中的实现为打开信息编辑框，下面实现信息编辑框和两个触发事件，代码如下：

```html
<div class="content">
  <!-- 模态框（Modal） -->
  <div
    class="modal fade"
    id="categoryModal"
    tabindex="-1"
    role="dialog"
    aria-labelledby="categoryModalLabel"
  >
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <button
            type="button"
            class="close"
            data-dismiss="modal"
            aria-label="Close"
          >
            <span aria-hidden="true">&times;</span>
          </button>
          <h6 class="modal-title" id="categoryModalLabel">Modal</h6>
        </div>
        <div class="modal-body">
          <form id="categoryForm" onsubmit="return false;">
            <div class="form-group">
              <div
                class="alert alert-danger"
                id="edit-error-msg"
                style="display: none;"
              >
                错误信息展示栏。
              </div>
            </div>
            <input
              type="hidden"
              class="form-control"
              id="categoryId"
              name="categoryId"
            />
            <div class="form-group">
              <label for="categoryName" class="control-label">分类名称:</label>
              <input
                type="text"
                class="form-control"
                id="categoryName"
                name="categoryName"
                placeholder="请输入分类名称"
                required="true"
              />
            </div>
            <div class="form-group">
              <label for="categoryIcon" class="control-label">分类图标:</label>
              <input
                type="hidden"
                class="form-control"
                id="categoryIcon"
                name="categoryIcon"
              />
              <div class="col-sm-4">
                <img
                  id="categoryIconImg"
                  src="/admin/dist/img/img-upload.png"
                  style="height: 64px;width: 64px;"
                />
                <button
                  class="btn btn-secondary"
                  style="margin-top: 5px;margin-bottom: 5px;"
                  id="categoryIconButton"
                >
                  <i class="fa fa-random"></i>&nbsp;图标切换
                </button>
              </div>
            </div>
          </form>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-default" data-dismiss="modal">
            取消
          </button>
          <button type="button" class="btn btn-primary" id="saveButton">
            确认
          </button>
        </div>
      </div>
    </div>
  </div>
</div>
```

`categoryAdd()` 方法和 `categoryEdit()` 方法实现如下：

```javascript
function categoryAdd() {
    reset();
    $('.modal-title').html('分类添加');
    $('#categoryModal').modal('show');
}

function categoryEdit() {
    reset();
    var id = getSelectedRow();
    if (id == null) {
        return;
    }
    $('.modal-title').html('分类编辑');
    $('#categoryModal').modal('show');
    //请求数据
    $.get("/admin/categories/info/" + id, function (r) {
        if (r.resultCode == 200 && r.data != null) {
            //填充数据至modal
            $("#categoryIconImg").attr("src", r.data.categoryIcon);
            $("#categoryIconImg").attr("style", "width:64px ;height: 64px;display:block;");
            $("#categoryIcon").val(r.data.categoryIcon);
            $("#categoryName").val(r.data.categoryName);
        }
    });
    $("#categoryId").val(id);
}
```

添加方法仅仅是将 Modal 框显示，修改功能则多了一个步骤，需要将选择的记录回显到编辑框中以供修改，因此需要请求 categories/info/{id} 详情接口获取被修改的分类数据信息。

#### 添加功能和编辑功能

在信息录入完成后可以点击信息编辑框下方的**确认**按钮，此时会进行数据的交互，js 实现代码如下：

```javascript
//绑定modal上的保存按钮
$('#saveButton').click(function () {
    var categoryName = $("#categoryName").val();
    if (!validCN_ENString2_18(categoryName)) {
        $('#edit-error-msg').css("display", "block");
        $('#edit-error-msg').html("请输入符合规范的分类名称！");
    } else {
        var params = $("#categoryForm").serialize();
        var url = '/admin/categories/save';
        var id = getSelectedRowWithoutAlert();
        if (id != null) {
            url = '/admin/categories/update';
        }
        $.ajax({
            type: 'POST',//方法类型
            url: url,
            data: params,
            success: function (result) {
                if (result.resultCode == 200) {
                    $('#categoryModal').modal('hide');
                    swal("保存成功", {
                        icon: "success",
                    });
                    reload();
                }
                else {
                    $('#categoryModal').modal('hide');
                    swal(result.message, {
                        icon: "error",
                    });
                }
                ;
            },
            error: function () {
                swal("操作失败", {
                    icon: "error",
                });
            }
        });
    }
});
```

由于传参和后续处理逻辑类似，为了避免太多重复代码因此将两个方法写在一起了，通过 id 是否大于 0 来确定是修改操作还是添加操作，方法步骤如下：

1. 前端对用户输入的数据进行简单的正则验证
2. 封装数据
3. 向对应的后端接口发送 Ajax 请求
4. 请求成功后提醒用户请求成功并隐藏当前的信息编辑框，同时刷新列表数据
5. 请求失败则提醒对应的错误信息

`getSelectedRowWithoutAlert()` 方法（在 `public.js` 中）：

```javascript
/**
 * 获取jqGrid选中的一条记录(不出现弹框)
 * @returns {*}
 */
function getSelectedRowWithoutAlert() {
    var grid = $("#jqGrid");
    var rowKey = grid.getGridParam("selrow");
    if (!rowKey) {
        return;
    }
    var selectedIDs = grid.getGridParam("selarrrow");
    if (selectedIDs.length > 1) {
        return;
    }
    return selectedIDs[0];
}
```

#### 删除功能

删除按钮的点击触发事件为 `deleteCagegory()`，在 category.js 文件中新增如下代码：

```javascript
function deleteCagegory() {
  var ids = getSelectedRows();
  if (ids == null) {
    return;
  }
  swal({
    title: '确认弹框',
    text: '确认要删除数据吗?',
    icon: 'warning',
    buttons: true,
    dangerMode: true,
  }).then((flag) => {
    if (flag) {
      $.ajax({
        type: 'POST',
        url: '/admin/categories/delete',
        contentType: 'application/json',
        data: JSON.stringify(ids),
        success: function (r) {
          if (r.resultCode == 200) {
            swal('删除成功', {
              icon: 'success',
            });
            $('#jqGrid').trigger('reloadGrid');
          } else {
            swal(r.message, {
              icon: 'error',
            });
          }
        },
      });
    }
  });
}
```

`getSelectedRows()` 方法（`public.js`）：

```javascript
/**
 * 获取jqGrid选中的多条记录
 * @returns {*}
 */
function getSelectedRows() {
    var grid = $("#jqGrid");
    var rowKey = grid.getGridParam("selrow");
    if (!rowKey) {
        swal("请选择一条记录", {
            icon: "warning",
        });
        return;
    }
    return grid.getGridParam("selarrrow");
}
```

获取用户在 jqgrid 表格中选择的需要删除的所有记录的 id，之后将参数封装并向后端发送 Ajax 请求，请求地址为 categories/delete。



## 侧边栏抽取

sidebar.html 这个模板文件是抽取出来的左侧导航栏文件，由于每个页面都需要加上侧边导航栏的代码，为了精简代码就将这部分代码提取出来作为公共代码，代码简化的同时，也方便维护和修改，代码如下：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
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
                <li class="nav-header">Dashboard</li>
                <li class="nav-item">
                    <a th:href="@{/admin/index}" th:class="${path}=='index'?'nav-link active':'nav-link'">
                        <i class="nav-icon fa fa-dashboard"></i>
                        <p>
                            Dashboard
                        </p>
                    </a>
                </li>
                <li class="nav-header">管理模块</li>
                <li class="nav-item">
                    <a th:href="@{/admin/categories}" th:class="${path}=='categories'?'nav-link active':'nav-link'">
                        <i class="fa fa-bookmark nav-icon" aria-hidden="true"></i>
                        <p>
                            分类管理
                        </p>
                    </a>
                </li>
                <li class="nav-item">
                    <a th:href="@{/admin/tags}" th:class="${path}=='tags'?'nav-link active':'nav-link'">
                        <i class="fa fa-tags nav-icon" aria-hidden="true"></i>
                        <p>
                            标签管理
                        </p>
                    </a>
                </li>
                <li class="nav-header">系统管理</li>
                <li class="nav-item">
                    <a th:href="@{/admin/profile}"
                       th:class="${path}=='profile'?'nav-link active':'nav-link'">
                        <i class="fa fa-user-secret nav-icon"></i>
                        <p>修改密码</p>
                    </a>
                </li>
                <li class="nav-item">
                    <a th:href="@{/admin/logout}" class="nav-link">
                        <i class="fa fa-sign-out nav-icon"></i>
                        <p>安全退出</p>
                    </a>
                </li>
                </li>
            </ul>
        </nav>
        <!-- /.sidebar-menu -->
    </div>
    <!-- /.sidebar -->
</aside>
</html>
```

接下来解释一下具体的实现逻辑，首先，以上这部分代码如果不进行抽取的话，在添加其他模块的时候需要在每一个模块的页面代码中添加一遍，但是基本上所有的代码都是重复的，只有一处不同，那就是导航栏中当前模块的选中状态，比如在分类管理页面中，左侧导航栏中的“分类管理”即为选中状态，其他页面与此相同。

这里的实现方式是通过添加一个 path 变量来控制当前导航栏中的选中状态，在模板文件中的 Thymeleaf 判断语句中通过 path 字段来确定是哪个功能模块，并对应的将左侧导航栏上当前模块的 css 样式给修改掉，判断语句如下：

```html
th:class="${path}=='categories'?'nav-link active':'nav-link'"
```

如果当前的 path 字段值为 'categories'，那么“分类管理”这个选项的 css 样式就修改为选中状态，如果当前的 path 字段值为 'tags'，那么“标签管理”这个选项的 css 样式就修改为选中状态，其他模块依次类推，path 字段的值是在哪里进行赋值的呢？答案是 admin 包下的 Controller 类中，在进行页面跳转时，会分别将对应的 path 字段进行赋值，代码如下：

- TagController：

```java
    @GetMapping("/tags")
    public String tagPage(HttpServletRequest request) {
        request.setAttribute("path", "tags");
        return "admin/tag";
    }
```

- CategoryController：

```java
    @GetMapping("/categories")
    public String categoryPage(HttpServletRequest request) {
        request.setAttribute("path", "categories");
        return "admin/category";
    }
```

通过这种方式，以后如果需要在系统中新增一个模块，就可以对应的增加一个导航栏按钮在 sidebar.html 文件中，并在后端的控制器方法中赋值对应的 path 字段即可，比如博客管理、配置管理等之后的功能模块。



## 标签管理模块

### 标签模块简介

标签是一种更为灵活、更有趣的分类方式，在书写博客时可以为每篇文章添加一个或多个标签，在博客系统中，文章的标签设计被广泛应用，我们可以看到大部分的博客网站中都会有标签设计，因此，在设计 personal-blog 这个项目时，也将标签运用了进来。

标签最明显的作用有如下两点：

1. 传统意义上分类的作用，类似分类名称

2. 对文章内容进行一定程度的描述，类似于关键词

虽然与分类设计类似，但是标签和分类还有一些细区别：

- 同一篇文章标签可以用多个，但通常只能属于一个分类
- 标签一般是在写作完成后，根据文章内容自行添加的内容
- 标签可以把文章中重点词语提炼出来，有关键词的意义，但是分类没有
- 标签通常更为主观，其内容相较于分类来说更加具体一些

与分类的功能和设计思想类似，但是又有一定的不同，标签可以算是分类的细化版本，同时，一篇博客的分类最好只有一个，但是在设计的时候，一篇博客的标签是可以有多个的，标签设计的介绍就到这里，接下来是功能开发的讲解。



### 持久层相关

#### 标签与文章的关系

一篇文章可以有多个标签字段，一个标签字段也可以被标注在多个文章中，这个情况与分类设计是有一些差别的，标签实体与文章实体的关系是多对多的关系，因此在表结构设计时不仅仅需要标签实体和文章实体的字段映射，还需要存储二者之间的关系数据，本系统采用的方式是新增一张关系表来维护二者多对多的关联关系。

#### 表结构设计

标签表以及标签文章关系表的 SQL 设计如下：

```sql
USE `my_blog_db`;

DROP TABLE IF EXISTS `tb_blog_tag`;

CREATE TABLE `tb_blog_tag` (
  `tag_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '标签表主键id',
  `tag_name` varchar(100) NOT NULL COMMENT '标签名称',
  `is_deleted` tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否删除 0=否 1=是',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_blog_tag_relation`;

CREATE TABLE `tb_blog_tag_relation` (
  `relation_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '关系表id',
  `blog_id` bigint(20) NOT NULL COMMENT '博客id',
  `tag_id` int(11) NOT NULL COMMENT '标签id',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '添加时间',
  PRIMARY KEY (`relation_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

在关系表中有一个 blog_id 字段，是文章表的主键 id，这张表存储的就是标签记录对应的文章记录，以多对多的方式进行记录的，把表结构导入到数据库中即可；另外标签表以及关系表的大部分的实现逻辑是在后续管理模块中进行调用和实现的。

#### BlogTag 实体类

```java
package cn.yuyingwai.springbootblog.entity;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import java.util.Date;

@Data
public class BlogTag {

    private Integer tagId;

    private String tagName;

    private Byte isDeleted;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createTime;

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(getClass().getSimpleName());
        sb.append(" [");
        sb.append("Hash = ").append(hashCode());
        sb.append(", tagId=").append(tagId);
        sb.append(", tagName=").append(tagName);
        sb.append(", isDeleted=").append(isDeleted);
        sb.append(", createTime=").append(createTime);
        sb.append("]");
        return sb.toString();
    }

}
```

#### BlogTagRelation 实体类

```java
package cn.yuyingwai.springbootblog.entity;

import lombok.Data;

import java.util.Date;

@Data
public class BlogTagRelation {

    private Long relationId;

    private Long blogId;

    private Integer tagId;

    private Date createTime;

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(getClass().getSimpleName());
        sb.append(" [");
        sb.append("Hash = ").append(hashCode());
        sb.append(", relationId=").append(relationId);
        sb.append(", blogId=").append(blogId);
        sb.append(", tagId=").append(tagId);
        sb.append(", createTime=").append(createTime);
        sb.append("]");
        return sb.toString();
    }

}
```

#### BlogTagDao.java

```java
package cn.yuyingwai.springbootblog.dao;

import cn.yuyingwai.springbootblog.entity.BlogTag;
import cn.yuyingwai.springbootblog.util.PageQueryUtil;

import java.util.List;

public interface BlogTagDao {

    int deleteByPrimaryKey(Integer tagId);

    int insert(BlogTag record);

    int insertSelective(BlogTag record);

    BlogTag selectByPrimaryKey(Integer tagId);

    BlogTag selectByTagName(String tagName);

    int updateByPrimaryKeySelective(BlogTag record);

    int updateByPrimaryKey(BlogTag record);

    List<BlogTag> findTagList(PageQueryUtil pageUtil);

    int getTotalTags(PageQueryUtil pageUtil);

    int deleteBatch(Integer[] ids);

}
```

#### BlogTagMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.yuyingwai.springbootblog.dao.BlogTagDao">
    <resultMap id="BaseResultMap" type="cn.yuyingwai.springbootblog.entity.BlogTag">
        <id column="tag_id" jdbcType="INTEGER" property="tagId"/>
        <result column="tag_name" jdbcType="VARCHAR" property="tagName"/>
        <result column="is_deleted" jdbcType="TINYINT" property="isDeleted"/>
        <result column="create_time" jdbcType="TIMESTAMP" property="createTime"/>
    </resultMap>

    <sql id="Base_Column_List">
        tag_id, tag_name, is_deleted, create_time
    </sql>

    <select id="findTagList" parameterType="Map" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from tb_blog_tag
        where is_deleted=0
        order by tag_id desc
        <if test="start!=null and limit!=null">
            limit #{start},#{limit}
        </if>
    </select>

    <select id="getTotalTags" parameterType="Map" resultType="int">
        select count(*)  from tb_blog_tag
        where is_deleted=0
    </select>

    <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from tb_blog_tag
        where tag_id = #{tagId,jdbcType=INTEGER} AND is_deleted = 0
    </select>

    <select id="selectByTagName" parameterType="java.lang.String" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from tb_blog_tag
        where tag_name = #{tagName,jdbcType=VARCHAR} AND is_deleted = 0
    </select>

    <update id="deleteByPrimaryKey" parameterType="java.lang.Integer">
        update tb_blog_tag set is_deleted = 1
        where tag_id = #{tagId,jdbcType=INTEGER}
    </update>

    <update id="deleteBatch">
        update tb_blog_tag
        set is_deleted=1 where tag_id in
        <foreach item="id" collection="array" open="(" separator="," close=")">
            #{id}
        </foreach>
    </update>

    <insert id="insert" parameterType="cn.yuyingwai.springbootblog.entity.BlogTag">
        insert into tb_blog_tag (tag_id, tag_name, is_deleted,
                                 create_time)
        values (#{tagId,jdbcType=INTEGER}, #{tagName,jdbcType=VARCHAR}, #{isDeleted,jdbcType=TINYINT},
                #{createTime,jdbcType=TIMESTAMP})
    </insert>
    <insert id="insertSelective" parameterType="cn.yuyingwai.springbootblog.entity.BlogTag">
        insert into tb_blog_tag
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="tagId != null">
                tag_id,
            </if>
            <if test="tagName != null">
                tag_name,
            </if>
            <if test="isDeleted != null">
                is_deleted,
            </if>
            <if test="createTime != null">
                create_time,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="tagId != null">
                #{tagId,jdbcType=INTEGER},
            </if>
            <if test="tagName != null">
                #{tagName,jdbcType=VARCHAR},
            </if>
            <if test="isDeleted != null">
                #{isDeleted,jdbcType=TINYINT},
            </if>
            <if test="createTime != null">
                #{createTime,jdbcType=TIMESTAMP},
            </if>
        </trim>
    </insert>
    <update id="updateByPrimaryKeySelective" parameterType="cn.yuyingwai.springbootblog.entity.BlogTag">
        update tb_blog_tag
        <set>
            <if test="tagName != null">
                tag_name = #{tagName,jdbcType=VARCHAR},
            </if>
            <if test="isDeleted != null">
                is_deleted = #{isDeleted,jdbcType=TINYINT},
            </if>
            <if test="createTime != null">
                create_time = #{createTime,jdbcType=TIMESTAMP},
            </if>
        </set>
        where tag_id = #{tagId,jdbcType=INTEGER}
    </update>
    <update id="updateByPrimaryKey" parameterType="cn.yuyingwai.springbootblog.entity.BlogTag">
        update tb_blog_tag
        set tag_name = #{tagName,jdbcType=VARCHAR},
            is_deleted = #{isDeleted,jdbcType=TINYINT},
            create_time = #{createTime,jdbcType=TIMESTAMP}
        where tag_id = #{tagId,jdbcType=INTEGER}
    </update>
</mapper>
```

#### BlogTagRelationDao.java

```java
package cn.yuyingwai.springbootblog.dao;

import cn.yuyingwai.springbootblog.entity.BlogTagRelation;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

@Mapper
public interface BlogTagRelationDao {

    int deleteByPrimaryKey(Long relationId);

    int insert(BlogTagRelation record);

    int insertSelective(BlogTagRelation record);

    BlogTagRelation selectByPrimaryKey(Long relationId);

    List<Long> selectDistinctTagIds(Integer[] tagIds);

    int updateByPrimaryKeySelective(BlogTagRelation record);

    int updateByPrimaryKey(BlogTagRelation record);
    
}
```

#### BlogTagRelationMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.yuyingwai.springbootblog.dao.BlogTagRelationDao">
    <resultMap id="BaseResultMap" type="cn.yuyingwai.springbootblog.entity.BlogTagRelation">
        <id column="relation_id" jdbcType="BIGINT" property="relationId"/>
        <result column="blog_id" jdbcType="BIGINT" property="blogId"/>
        <result column="tag_id" jdbcType="INTEGER" property="tagId"/>
        <result column="create_time" jdbcType="TIMESTAMP" property="createTime"/>
    </resultMap>
    <sql id="Base_Column_List">
        relation_id, blog_id, tag_id, create_time
    </sql>
    <select id="selectByPrimaryKey" parameterType="java.lang.Long" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from tb_blog_tag_relation
        where relation_id = #{relationId,jdbcType=BIGINT}
    </select>

    <select id="selectDistinctTagIds" resultType="java.lang.Long">
        select
        DISTINCT(tag_id)
        from tb_blog_tag_relation
        where tag_id in
        <foreach item="id" collection="array" open="(" separator="," close=")">
            #{id}
        </foreach>
    </select>

    <delete id="deleteByPrimaryKey" parameterType="java.lang.Long">
        delete from tb_blog_tag_relation
        where relation_id = #{relationId,jdbcType=BIGINT}
    </delete>

    <insert id="insert" parameterType="cn.yuyingwai.springbootblog.entity.BlogTagRelation">
        insert into tb_blog_tag_relation (relation_id, blog_id, tag_id,
                                          create_time)
        values (#{relationId,jdbcType=BIGINT}, #{blogId,jdbcType=BIGINT}, #{tagId,jdbcType=INTEGER},
                #{createTime,jdbcType=TIMESTAMP})
    </insert>
    <insert id="insertSelective" parameterType="cn.yuyingwai.springbootblog.entity.BlogTagRelation">
        insert into tb_blog_tag_relation
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="relationId != null">
                relation_id,
            </if>
            <if test="blogId != null">
                blog_id,
            </if>
            <if test="tagId != null">
                tag_id,
            </if>
            <if test="createTime != null">
                create_time,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="relationId != null">
                #{relationId,jdbcType=BIGINT},
            </if>
            <if test="blogId != null">
                #{blogId,jdbcType=BIGINT},
            </if>
            <if test="tagId != null">
                #{tagId,jdbcType=INTEGER},
            </if>
            <if test="createTime != null">
                #{createTime,jdbcType=TIMESTAMP},
            </if>
        </trim>
    </insert>

    <update id="updateByPrimaryKeySelective" parameterType="cn.yuyingwai.springbootblog.entity.BlogTagRelation">
        update tb_blog_tag_relation
        <set>
            <if test="blogId != null">
                blog_id = #{blogId,jdbcType=BIGINT},
            </if>
            <if test="tagId != null">
                tag_id = #{tagId,jdbcType=INTEGER},
            </if>
            <if test="createTime != null">
                create_time = #{createTime,jdbcType=TIMESTAMP},
            </if>
        </set>
        where relation_id = #{relationId,jdbcType=BIGINT}
    </update>
    <update id="updateByPrimaryKey" parameterType="cn.yuyingwai.springbootblog.entity.BlogTagRelation">
        update tb_blog_tag_relation
        set blog_id = #{blogId,jdbcType=BIGINT},
            tag_id = #{tagId,jdbcType=INTEGER},
            create_time = #{createTime,jdbcType=TIMESTAMP}
        where relation_id = #{relationId,jdbcType=BIGINT}
    </update>
</mapper>
```



### 标签模块接口设计及实现

#### 标签列表分页接口

列表接口负责接收前端传来的分页参数，如 page 、limit 等参数，之后将数据总数和对应页面的数据列表查询出来并封装为分页数据返回给前端。

##### 控制层

TagController.java：接口的映射地址为 /tags/list，请求方法为 GET，代码如下：

```java
package cn.yuyingwai.springbootblog.controller.admin;

import cn.yuyingwai.springbootblog.service.TagService;
import cn.yuyingwai.springbootblog.util.PageQueryUtil;
import cn.yuyingwai.springbootblog.util.Result;
import cn.yuyingwai.springbootblog.util.ResultGenerator;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import java.util.Map;

@Controller
@RequestMapping("/admin")
public class TagController {

    @Resource
    private TagService tagService;

    @GetMapping("/tags")
    public String tagPage(HttpServletRequest request) {
        request.setAttribute("path", "tags");
        return "admin/tag";
    }

    @GetMapping("/tags/list")
    @ResponseBody
    public Result list(@RequestParam Map<String, Object> params) {
        if (StringUtils.isEmpty(params.get("page")) || StringUtils.isEmpty(params.get("limit"))) {
            return ResultGenerator.genFailResult("参数异常！");
        }
        PageQueryUtil pageUtil = new PageQueryUtil(params);
        return ResultGenerator.genSuccessResult(tagService.getBlogTagPage(pageUtil));
    }

}
```

##### 业务层

TagServiceImpl.java：

```java
package cn.yuyingwai.springbootblog.service.impl;

import cn.yuyingwai.springbootblog.dao.BlogTagDao;
import cn.yuyingwai.springbootblog.dao.BlogTagRelationDao;
import cn.yuyingwai.springbootblog.entity.BlogTag;
import cn.yuyingwai.springbootblog.service.TagService;
import cn.yuyingwai.springbootblog.util.PageQueryUtil;
import cn.yuyingwai.springbootblog.util.PageResult;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.List;

@Service
public class TagServiceImpl implements TagService {

    @Autowired
    private BlogTagDao blogTagDao;

    @Autowired
    private BlogTagRelationDao relationDao;

    /**
     * 查询标签的分页数据
     * @param pageUtil
     * @return
     */
    @Override
    public PageResult getBlogTagPage(PageQueryUtil pageUtil) {
        List<BlogTag> tags = blogTagDao.findTagList(pageUtil);
        int total = blogTagDao.getTotalTags(pageUtil);
        PageResult pageResult = new PageResult(tags, total, pageUtil.getLimit(), pageUtil.getPage());
        return pageResult;
    }
}
```



#### 添加标签接口

添加接口负责接收前端的 POST 请求并处理其中的参数，接收的参数为 tagName 字段，tagName 为标签名称。

##### 控制层

```java
    @PostMapping("/tags/save")
    @ResponseBody
    public Result save(@RequestParam("tagName") String tagName) {
        if (StringUtils.isEmpty(tagName)) {
            return ResultGenerator.genFailResult("参数异常！");
        }
        if (tagService.saveTag(tagName)) {
            return ResultGenerator.genSuccessResult();
        } else {
            return ResultGenerator.genFailResult("标签名称重复");
        }
    }
```

##### 业务层

```java
    /**
     * 添加标签
     * @param tagName
     * @return
     */
    @Override
    public Boolean saveTag(String tagName) {
        BlogTag temp = blogTagDao.selectByTagName(tagName);
        if (temp == null) {
            BlogTag blogTag = new BlogTag();
            blogTag.setTagName(tagName);
            return blogTagDao.insertSelective(blogTag) > 0;
        }
        return false;
    }
```

#### 删除标签接口

删除接口负责接收前端的标签删除请求，处理前端传输过来的数据后，将这些记录从数据库中删除，这里的“删除”功能并不是真正意义上的删除，而是逻辑删除，我们将接受的参数设置为一个数组，可以同时删除多条记录，只需要在前端将用户选择的记录 id 封装好再传参到后端即可。

接口的请求路径为 /tags/delete，并使用 @RequestBody 将前端传过来的参数封装为 id 数组，参数验证通过后则调用 deleteBatch() 批量删除方法进行数据库操作，否则将向前端返回错误信息。

##### 控制层

```java
    @PostMapping("/tags/delete")
    @ResponseBody
    public Result delete(@RequestBody Integer[] ids) {
        if (ids.length < 1) {
            return ResultGenerator.genFailResult("参数异常！");
        }
        if (tagService.deleteBatch(ids)) {
            return ResultGenerator.genSuccessResult();
        } else {
            return ResultGenerator.genFailResult("有关联数据请勿强行删除");
        }
    }
```

##### 业务层

```java
    /**
     * 删除没有关联关系的标签
     * @param ids
     * @return
     */
    @Override
    public Boolean deleteBatch(Integer[] ids) {
        // 已存在关联关系不删除
        List<Long> relations = relationDao.selectDistinctTagIds(ids);
        if (!CollectionUtils.isEmpty(relations)) {
            return false;
        }
        // 删除tag
        return blogTagDao.deleteBatch(ids) > 0;
    }
```

在业务方法实现中，需要判断该标签是否已经与文章表中的数据进行了关联，如果已经存在关联关系，就不进行删除操作，这是其中的一种处理方式，因为在添加文章数据时，也会对应的向数据库中新增标签数据和关系数据，因此在数据删除时需要进行确认以免造成数据混乱，当然也可以使用另外一种处理方法，就是在删除标签记录时，将标签记录以及对应的关系表中所有与此标签有关联的记录删除掉，这样也是可以的，本系统选择的是第一种方式。



### 前端页面实现

#### tag.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<header th:replace="admin/header::header-fragment"></header>
<body class="hold-transition sidebar-mini">
<div class="wrapper">
    <!-- 引入页面头header-fragment -->
    <div th:replace="admin/header::header-nav"></div>
    <!-- 引入工具栏sidebar-fragment -->
    <div th:replace="admin/sidebar::sidebar-fragment(${path})"></div>
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
                <div class="card card-primary card-outline">
                    <div class="card-header">
                        <h3 class="card-title">标签管理</h3>
                    </div> <!-- /.card-body -->
                    <div class="card-body">
                        <div class="grid-btn">
                            <input type="text" class="form-control col-1" id="tagName" name="tagName"
                                   placeholder="标签名称" required="true">&nbsp;&nbsp;&nbsp;
                            <button class="btn btn-info" onclick="tagAdd()"><i
                                    class="fa fa-plus"></i>&nbsp;新增
                            </button>
                            <button class="btn btn-danger" onclick="deleteTag()"><i
                                    class="fa fa-trash-o"></i>&nbsp;删除
                            </button>
                        </div>
                        <table id="jqGrid" class="table table-bordered">
                        </table>
                        <div id="jqGridPager"></div>
                    </div><!-- /.card-body -->
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
<!-- jqgrid -->
<script th:src="@{/admin/plugins/jqgrid-5.3.0/jquery.jqGrid.min.js}"></script>
<script th:src="@{/admin/plugins/jqgrid-5.3.0/grid.locale-cn.js}"></script>
<!-- sweetalert -->
<script th:src="@{/admin/plugins/sweetalert/sweetalert.min.js}"></script>
<script th:src="@{/admin/dist/js/public.js}"></script>
<script th:src="@{/admin/dist/js/tag.js}"></script>
</body>
</html>
```

##### 功能按钮及信息编辑框

在页面中添加对应的功能按钮以及触发事件，由于标签信息的字段并不多，因此在按钮区新增了一个标签名称的输入框，代码如下：

```html
<div class="grid-btn">
  <input
    type="text"
    class="form-control col-1"
    id="tagName"
    name="tagName"
    placeholder="标签名称"
    required="true"
  />&nbsp;&nbsp;&nbsp;
  <button class="btn btn-info" onclick="tagAdd()">
    <i class="fa fa-plus"></i>&nbsp;新增
  </button>
  <button class="btn btn-danger" onclick="deleteTag()">
    <i class="fa fa-trash-o"></i>&nbsp;删除
  </button>
</div>
```

##### 分页信息展示区域

页面中已经引入 JqGrid 的相关静态资源文件，需要在页面中展示分页数据的区域增加如下代码：

```html
<table id="jqGrid" class="table table-bordered"></table>
<div id="jqGridPager"></div>
```

#### sidebar.html

在菜单中添加以下代码：

```html
<li class="nav-item">
    <a th:href="@{/admin/tags}" th:class="${path}=='tags'?'nav-link active':'nav-link'">
        <i class="fa fa-tags nav-icon" aria-hidden="true"></i>
        <p>
            标签管理
        </p>
    </a>
</li>
```



### 前端功能实现

#### 分页功能

在 resources/static/admin/dist/js 目录下新增 tag.js 文件，并添加如下代码：

```javascript
$(function () {
  $('#jqGrid').jqGrid({
    url: '/admin/tags/list',
    datatype: 'json',
    colModel: [
      {
        label: 'id',
        name: 'tagId',
        index: 'tagId',
        width: 50,
        key: true,
        hidden: true,
      },
      { label: '标签名称', name: 'tagName', index: 'tagName', width: 240 },
      {
        label: '添加时间',
        name: 'createTime',
        index: 'createTime',
        width: 120,
      },
    ],
    height: 560,
    rowNum: 10,
    rowList: [10, 20, 50],
    styleUI: 'Bootstrap',
    loadtext: '信息读取中...',
    rownumbers: false,
    rownumWidth: 20,
    autowidth: true,
    multiselect: true,
    pager: '#jqGridPager',
    jsonReader: {
      root: 'data.list',
      page: 'data.currPage',
      total: 'data.totalPage',
      records: 'data.totalCount',
    },
    prmNames: {
      page: 'page',
      rows: 'limit',
      order: 'order',
    },
    gridComplete: function () {
      //隐藏grid底部滚动条
      $('#jqGrid').closest('.ui-jqgrid-bdiv').css({ 'overflow-x': 'hidden' });
    },
  });
  $(window).resize(function () {
    $('#jqGrid').setGridWidth($('.card-body').width());
  });
});

/**
 * jqGrid重新加载
 */
function reload() {
    var page = $("#jqGrid").jqGrid('getGridParam', 'page');
    $("#jqGrid").jqGrid('setGridParam', {
        page: page
    }).trigger("reloadGrid");
}
```

以上代码的主要功能为分页数据展示、字段格式化 jqGrid DOM 宽度的自适应，在页面加载时，调用 JqGrid 的初始化方法，将页面中 id 为 jqGrid 的 DOM 渲染为分页表格，并向后端发送请求，之后按照后端返回的 json 数据填充表格以及表格下方的分页按钮。

#### 添加功能

在标签名称输入框中输入完成后可以点击右侧的**添加**按钮，此时会触发 `tagAdd()` 方法进行数据的交互，js 实现代码如下：

```javascript
function tagAdd() {
  var tagName = $('#tagName').val();
  if (!validCN_ENString2_18(tagName)) {
    swal('标签名称不规范', {
      icon: 'error',
    });
  } else {
    var url = '/admin/tags/save?tagName=' + tagName;
    $.ajax({
      type: 'POST', //方法类型
      url: url,
      success: function (result) {
        if (result.resultCode == 200) {
          $('#tagName').val('');
          swal('保存成功', {
            icon: 'success',
          });
          reload();
        } else {
          $('#tagName').val('');
          swal(result.message, {
            icon: 'error',
          });
        }
      },
      error: function () {
        swal('操作失败', {
          icon: 'error',
        });
      },
    });
  }
}
```

按钮点击后会触发对应的 js 方法，在该方法中首先会对用户输入的数据进行简单的正则验证，之后会封装数据并向对应的后端接口发送 Ajax 请求添加标签数据，之后根据后端返回的结果进行提示。

#### 删除功能

删除按钮的点击触发事件为 `deleteTag()`，在 tag.js 文件中新增如下代码：

```js
function deleteTag() {
  var ids = getSelectedRows();
  if (ids == null) {
    return;
  }
  swal({
    title: '确认弹框',
    text: '确认要删除数据吗?',
    icon: 'warning',
    buttons: true,
    dangerMode: true,
  }).then((flag) => {
    if (flag) {
      $.ajax({
        type: 'POST',
        url: '/admin/tags/delete',
        contentType: 'application/json',
        data: JSON.stringify(ids),
        success: function (r) {
          if (r.resultCode == 200) {
            swal('删除成功', {
              icon: 'success',
            });
            $('#jqGrid').trigger('reloadGrid');
          } else {
            swal(r.message, {
              icon: 'error',
            });
          }
        },
      });
    }
  });
}
```

获取用户在 jqgrid 表格中选择的需要删除的所有记录的 id，之后将参数封装并向后端发送 Ajax 请求，请求地址为 tags/delete。



## 文章编辑模块

### 富文本编译器

在 form 表单中通常会用 input 标签和 textarea 标签，简单的如登录信息的获取可能使用 input 标签即可，字数多一些的会用 textarea 标签来获取用户输入的内容，而博客文章排版比较丰富，各种内容和元素都会出现，此时就出现了问题，需要复杂排版的图文混合的内容或者更多内容录入的时候，这两个标签显然就无法满足需求。

#### 什么是富文本编译器？

> 富文本编辑器，是一种可内嵌于浏览器，所见即所得的文本编辑器。 富文本编辑器不同于文本编辑器(如 textarea 标签、input 标签)，也可以叫做图文编辑器，在富文本编辑器里可以编辑非常丰富的内容，如文字、图片、表情、代码……应有尽有，满足你的大部分需求。 像一些新闻排版，基本是以图文排版为主，而淘宝京东这些电商的商品详情页，基本都是多张已经排版好的设计图拼接而来的，富文本编辑器可以很完美的支持者两种需求。

目前的富文本编辑器主要有 markdown 版本和非 markdown 版本的编辑器，一般企业开发中使用非 markdown 版本比较多，常见的有 UEditor 和 KindEditor 等，因为运营人员可能不太懂 markdown 语法，而博客文章的编辑通常是使用 markdown 编辑器（即 md 编辑器），因为这部分人员掌握 markdown 语法也很快，所以大部分博客网站都会默认使用 markdown 编辑器作为用户的文章编辑器。

#### 为什么要使用富文本编辑器

以下是使用富文本编辑器的原因，也是富文本编辑器的优点：

- 需求变更导致，业务方提出的编辑需求越来越复杂
- 编辑的内容变得越来越复杂、越来越丰富
- 比起编辑 html，富文本编辑器更灵活
- 富文本编辑器功能丰富，满足大部分需求



### 文章编辑页面制作

#### 导航栏

首先在左侧导航栏中新增编辑页的导航按钮，在 sidebar.html 文件中新增如下代码（管理模块上面）：

```html
<li class="nav-item">
  <a
    th:href="@{/admin/blogs/edit}"
    th:class="${path}=='edit'?'nav-link active':'nav-link'"
  >
    <i class="nav-icon fa fa fa-pencil-square-o"></i>
    <p>
      发布博客
    </p>
  </a>
</li>
```

点击后的跳转路径为 /admin/blogs/edit，之后新建 Controller 来处理该路径并跳转到对应的页面。

#### Controller 处理跳转

首先在 controller/admin 包下新建 BlogController.java，之后新增如下代码：

```java
package cn.yuyingwai.springbootblog.controller.admin;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;

@Controller
@RequestMapping("/admin")
public class BlogController {

    @GetMapping("/blogs/edit")
    public String edit(HttpServletRequest request) {
        request.setAttribute("path", "edit");
        return "admin/edit";
    }

}
```

该方法用于处理 /admin/blogs/edit 请求，并设置 path 字段，之后跳转到 admin 目录下的 edit.html 中。



#### edit.html 页面制作

接下来就是博客编辑页面的模板文件制作了，在 resources/templates/admin 目录下新建 edit.html，并引入对应的 js 文件和 css 样式文件，代码如下：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <header th:replace="admin/header::header-fragment"></header>
  <body class="hold-transition sidebar-mini">
    <div class="wrapper">
      <!-- 引入页面头header-fragment -->
      <div th:replace="admin/header::header-nav"></div>
      <!-- 引入工具栏sidebar-fragment -->
      <div th:replace="admin/sidebar::sidebar-fragment(${path})"></div>
      <!-- Content Wrapper. Contains page content -->
      <div class="content-wrapper">
        <!-- Content Header (Page header) -->
        <div class="content-header">
          <div class="container-fluid"></div>
          <!-- /.container-fluid -->
        </div>
        <!-- Main content -->
        <div class="content">
          <div class="container-fluid">
            <div class="card card-primary card-outline">
              <div class="card-header">
                <h3 class="card-title">发布文章</h3>
              </div>
              <div class="card-body">
                编辑页面
              </div>
            </div>
          </div>
          <!-- /.container-fluid -->
        </div>
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
    <script
      th:src="@{/admin/plugins/bootstrap/js/bootstrap.bundle.min.js}"
    ></script>
  </body>
</html>
```



### Editor.md 编辑器整合

#### 整合步骤

1. 下载 Editor.md 插件代码并放进项目的 plugins 目录

2. 在 html 代码中引入 Editor.md 相关文件

   代码如下，首先是 css 文件：

   ```html
   <link
     th:href="@{/admin/plugins/editormd/css/editormd.css}"
     rel="stylesheet"
   />
   ```

   之后是引入编辑器的 js 文件：

   ```html
   <!-- editor.md -->
   <script th:src="@{/admin/plugins/editormd/editormd.min.js}"></script>
   ```

3. 添加编辑框 DOM 元素

   ```html
   <div class="card-body">
     <form id="blogForm" onsubmit="return false;">
       <div class="form-group" id="blog-editormd">
         <textarea style="display:none;"></textarea>
       </div>
       <div class="form-group">
         <!-- 按钮 -->
         &nbsp;<button
           class="btn btn-info float-right"
           style="margin-left: 5px;"
           id="confirmButton"
         >
           保存文章
         </button>
       </div>
     </form>
   </div>
   ```

   我们会在这里初始化 Editor.md 编辑器，这里首先定义将要初始化时的 id 名称为 blog-editormd，之后调用 Editor.md 插件的方法在这里将编辑器生成出来。

4. 初始化 Editor.md 对象

   添加如下 js 代码：

   ```html
   <script type="text/javascript">
     var blogEditor;
     $(function () {
       blogEditor = editormd('blog-editormd', {
         width: '100%',
         height: 640,
         syncScrolling: 'single',
         path: '/admin/plugins/editormd/lib/',
         toolbarModes: 'full',
       });
     });
   </script>
   ```

   通过调用 `editormd()` 方法并传入前文中定义的 DOM id，之后再次重启项目就能够看到编辑器的效果了。

#### 获取文档内容

在输入完成后，我们需要将 Editor.md 编辑器中输入的文字内容取出来，并传给后端以进行逻辑处理，提供了 `getMarkdown()` 方法来获取其中的内容，添加如下代码：

```javascript
$('#confirmButton').bind('click', function () {
  console.log(blogEditor.getMarkdown());
  alert(blogEditor.getMarkdown());
});
```

这里是添加了“保存文章”按钮的点击事件，点击该按钮后，会将编辑器中的内容给打印或者 alert 出来。

#### Editor.md 编辑器图片上传功能完善

在整合之后，默认是不可以上传图片的，需要略作配置修改，在 Editor.md 编辑器初始化时新增如下配置项：

```js
    /**图片上传配置*/
    imageUpload: true,//开启图片上传
    imageFormats: ["jpg", "jpeg", "gif", "png", "bmp", "webp"], //图片上传格式
    imageUploadURL: "/admin/blogs/md/uploadfile",//图片上传的后端路径
    onload: function (obj) { //上传成功之后的回调
    }
```

配置项的相关参数及参数释义已经给出，之后需要在后台 Controller 代码中新增一个方法用于接收图片上传请求并返回图片路径给 Editor.md 编辑器。

在 BlogController.java 中新增如下代码用于文件上传：

```java
    /**
     * 接收图片上传请求并返回图片路径给 Editor.md 编辑器
     * @param request
     * @param response
     * @param file
     * @throws IOException
     * @throws URISyntaxException
     */
    @PostMapping("/blogs/md/uploadfile")
    public void uploadFileByEditormd(HttpServletRequest request,
                                     HttpServletResponse response,
                                     @RequestParam(name = "editormd-image-file", required = true) MultipartFile file) throws IOException, URISyntaxException {
        String FILE_UPLOAD_DIC = "D:\\upload\\";   // 上传文件的默认url前缀，根据部署设置自行修改
        String fileName = file.getOriginalFilename();
        String suffixName = fileName.substring(fileName.lastIndexOf("."));
        // 生成文件名称通用方法
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd_HHmmss");
        Random r = new Random();
        StringBuilder tempName = new StringBuilder();
        tempName.append(sdf.format(new Date())).append(r.nextInt(100)).append(suffixName);
        String newFileName = tempName.toString();
        // 创建文件
        File destFile = new File(FILE_UPLOAD_DIC + newFileName);
        String fileUrl = MyBlogUtils.getHost(new URI(request.getRequestURI() + "")) + "/upload/" + newFileName;
        File fileDirectory = new File(FILE_UPLOAD_DIC);
        try {
            if (!fileDirectory.exists()) {
                if (!fileDirectory.mkdir()) {
                    throw new IOException("文件夹创建失败，路径为：" + fileDirectory);
                }
            }
            file.transferTo(destFile);
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            request.setCharacterEncoding("utf-8");
            response.setHeader("Content-Type", "text/html");
            response.getWriter().write("{\"success\": 1, \"message\":\"success\",\"url\":\"" + fileUrl + "\"}");
        } catch (UnsupportedEncodingException e) {
            response.getWriter().write("{\"success\":0}");
        } catch (IOException e) {
            response.getWriter().write("{\"success\":0}");
        }
    }
```

**MyBlogUtils.java：**

```java
package cn.yuyingwai.springbootblog.util;

import java.net.URI;

public class MyBlogUtils {

    public static URI getHost(URI uri) {
        URI effectiveURI = null;
        try {
            effectiveURI = new URI(uri.getScheme(), uri.getUserInfo(), uri.getHost(), uri.getPort(), null, null, null);
        } catch (Throwable var4) {
            effectiveURI = null;
        }
        return effectiveURI;
    }

}
```

之后在 MyBlogWebMvcConfigurer.java 中新增拦截器，代码如下：

```java
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/upload/**").addResourceLocations("file:D:\\upload\\");
    }
```



### 持久层相关

#### 表结构设计

这里以 CSDN 平台的文章编辑模块为例，来确定一下文章表的字段设计，编辑模块如下图所示：

![](http://images.yingwai.top/picgo/20201214211057.jpg)

通过上图可以得出以下字段：

- 文章标题
- 文章内容
- 文章标签
- 文章分类
- 发布状态

以上是字段是博客文章实体应该具有的基础字段，不管是哪个博客平台都会存在这些字段，本博客系统上在此基础上增加了几个字段：

- 文章封面图(为了页面美观)
- 阅读量(博客文章的基本字段)
- 是否允许评论(有评论模块，可以控制评论模块的开放和关闭)

文章表的 SQL 设计如下，直接执行如下 SQL 语句即可：

```sql
USE `my_blog_db`;

DROP TABLE IF EXISTS `tb_blog`;

CREATE TABLE `tb_blog` (
  `blog_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '博客表主键id',
  `blog_title` varchar(200) NOT NULL COMMENT '博客标题',
  `blog_sub_url` varchar(200) NOT NULL COMMENT '博客自定义路径url',
  `blog_cover_image` varchar(200) NOT NULL COMMENT '博客封面图',
  `blog_content` mediumtext NOT NULL COMMENT '博客内容',
  `blog_category_id` int(11) NOT NULL COMMENT '博客分类id',
  `blog_category_name` varchar(50) NOT NULL COMMENT '博客分类(冗余字段)',
  `blog_tags` varchar(200) NOT NULL COMMENT '博客标签',
  `blog_status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '0-草稿 1-发布',
  `blog_views` bigint(20) NOT NULL DEFAULT '0' COMMENT '阅读量',
  `enable_comment` tinyint(4) NOT NULL DEFAULT '0' COMMENT '0-允许评论 1-不允许评论',
  `is_deleted` tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否删除 0=否 1=是',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '添加时间',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`blog_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

在 tb_blog 表中，设计了一个 is_deleted 字段，用于逻辑删除的标志位，由于 is_deleted 的字段设计，对表中数据的删除都是软删除，因为是个人博客，这么做的目的主要也是为了防止误删。

#### Blog 实体类

```java
package cn.yuyingwai.springbootblog.entity;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import java.util.Date;

@Data
public class Blog {

    private Long blogId;

    private String blogTitle;

    private String blogSubUrl;

    private String blogCoverImage;

    private Integer blogCategoryId;

    private String blogCategoryName;

    private String blogTags;

    private Byte blogStatus;

    private Long blogViews;

    private Byte enableComment;

    private Byte isDeleted;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createTime;

    private Date updateTime;

    private String blogContent;

    public void setBlogTitle(String blogTitle) {
        this.blogTitle = blogTitle == null ? null : blogTitle.trim();
    }

    public void setBlogSubUrl(String blogSubUrl) {
        this.blogSubUrl = blogSubUrl == null ? null : blogSubUrl.trim();
    }

    public void setBlogCoverImage(String blogCoverImage) {
        this.blogCoverImage = blogCoverImage == null ? null : blogCoverImage.trim();
    }

    public void setBlogCategoryName(String blogCategoryName) {
        this.blogCategoryName = blogCategoryName == null ? null : blogCategoryName.trim();
    }

    public void setBlogTags(String blogTags) {
        this.blogTags = blogTags == null ? null : blogTags.trim();
    }

    public void setBlogContent(String blogContent) {
        this.blogContent = blogContent == null ? null : blogContent.trim();
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(getClass().getSimpleName());
        sb.append(" [");
        sb.append("Hash = ").append(hashCode());
        sb.append(", blogId=").append(blogId);
        sb.append(", blogTitle=").append(blogTitle);
        sb.append(", blogSubUrl=").append(blogSubUrl);
        sb.append(", blogCoverImage=").append(blogCoverImage);
        sb.append(", blogCategoryId=").append(blogCategoryId);
        sb.append(", blogCategoryName=").append(blogCategoryName);
        sb.append(", blogTags=").append(blogTags);
        sb.append(", blogStatus=").append(blogStatus);
        sb.append(", blogViews=").append(blogViews);
        sb.append(", enableComment=").append(enableComment);
        sb.append(", isDeleted=").append(isDeleted);
        sb.append(", createTime=").append(createTime);
        sb.append(", updateTime=").append(updateTime);
        sb.append(", blogContent=").append(blogContent);
        sb.append("]");
        return sb.toString();
    }

}
```

#### BlogDao.java

```java
package cn.yuyingwai.springbootblog.dao;

import cn.yuyingwai.springbootblog.entity.BlogCategory;
import cn.yuyingwai.springbootblog.util.PageQueryUtil;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

@Mapper
public interface BlogDao {

    int deleteByPrimaryKey(Integer categoryId);

    int insert(BlogCategory record);

    int insertSelective(BlogCategory record);

    BlogCategory selectByPrimaryKey(Integer categoryId);

    BlogCategory selectByCategoryName(String categoryName);

    int updateByPrimaryKeySelective(BlogCategory record);

    int updateByPrimaryKey(BlogCategory record);

    List<BlogCategory> findCategoryList(PageQueryUtil pageUtil);

    List<BlogCategory> selectByCategoryIds(@Param("categoryIds") List<Integer> categoryIds);

    int getTotalCategories(PageQueryUtil pageUtil);

    int deleteBatch(Integer[] ids);
    
}
```

#### BlogMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.yuyingwai.springbootblog.dao.BlogDao">
    <resultMap id="BaseResultMap" type="cn.yuyingwai.springbootblog.entity.Blog">
        <id column="blog_id" jdbcType="BIGINT" property="blogId"/>
        <result column="blog_title" jdbcType="VARCHAR" property="blogTitle"/>
        <result column="blog_sub_url" jdbcType="VARCHAR" property="blogSubUrl"/>
        <result column="blog_cover_image" jdbcType="VARCHAR" property="blogCoverImage"/>
        <result column="blog_category_id" jdbcType="INTEGER" property="blogCategoryId"/>
        <result column="blog_category_name" jdbcType="VARCHAR" property="blogCategoryName"/>
        <result column="blog_tags" jdbcType="VARCHAR" property="blogTags"/>
        <result column="blog_status" jdbcType="TINYINT" property="blogStatus"/>
        <result column="blog_views" jdbcType="BIGINT" property="blogViews"/>
        <result column="enable_comment" jdbcType="TINYINT" property="enableComment"/>
        <result column="is_deleted" jdbcType="TINYINT" property="isDeleted"/>
        <result column="create_time" jdbcType="TIMESTAMP" property="createTime"/>
        <result column="update_time" jdbcType="TIMESTAMP" property="updateTime"/>
    </resultMap>
    <resultMap extends="BaseResultMap" id="ResultMapWithBLOBs" type="cn.yuyingwai.springbootblog.entity.Blog">
        <result column="blog_content" jdbcType="LONGVARCHAR" property="blogContent"/>
    </resultMap>
    <sql id="Base_Column_List">
        blog_id, blog_title, blog_sub_url, blog_cover_image, blog_category_id, blog_category_name, 
    blog_tags, blog_status, blog_views, enable_comment, is_deleted, create_time, update_time
    </sql>
    <sql id="Blob_Column_List">
        blog_content
    </sql>
    <select id="selectByPrimaryKey" parameterType="java.lang.Long" resultMap="ResultMapWithBLOBs">
        select
        <include refid="Base_Column_List"/>
        ,
        <include refid="Blob_Column_List"/>
        from tb_blog
        where blog_id = #{blogId,jdbcType=BIGINT} and is_deleted = 0
    </select>
    <insert id="insert" parameterType="cn.yuyingwai.springbootblog.entity.Blog">
        insert into tb_blog (blog_id, blog_title, blog_sub_url,
                             blog_cover_image, blog_category_id, blog_category_name,
                             blog_tags, blog_status, blog_views,
                             enable_comment, is_deleted, create_time,
                             update_time, blog_content)
        values (#{blogId,jdbcType=BIGINT}, #{blogTitle,jdbcType=VARCHAR}, #{blogSubUrl,jdbcType=VARCHAR},
                #{blogCoverImage,jdbcType=VARCHAR}, #{blogCategoryId,jdbcType=INTEGER}, #{blogCategoryName,jdbcType=VARCHAR},
                #{blogTags,jdbcType=VARCHAR}, #{blogStatus,jdbcType=TINYINT}, #{blogViews,jdbcType=BIGINT},
                #{enableComment,jdbcType=TINYINT}, #{isDeleted,jdbcType=TINYINT}, #{createTime,jdbcType=TIMESTAMP},
                #{updateTime,jdbcType=TIMESTAMP}, #{blogContent,jdbcType=LONGVARCHAR})
    </insert>
    <insert id="insertSelective" useGeneratedKeys="true" keyProperty="blogId"
            parameterType="cn.yuyingwai.springbootblog.entity.Blog">
        insert into tb_blog
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="blogId != null">
                blog_id,
            </if>
            <if test="blogTitle != null">
                blog_title,
            </if>
            <if test="blogSubUrl != null">
                blog_sub_url,
            </if>
            <if test="blogCoverImage != null">
                blog_cover_image,
            </if>
            <if test="blogCategoryId != null">
                blog_category_id,
            </if>
            <if test="blogCategoryName != null">
                blog_category_name,
            </if>
            <if test="blogTags != null">
                blog_tags,
            </if>
            <if test="blogStatus != null">
                blog_status,
            </if>
            <if test="blogViews != null">
                blog_views,
            </if>
            <if test="enableComment != null">
                enable_comment,
            </if>
            <if test="isDeleted != null">
                is_deleted,
            </if>
            <if test="createTime != null">
                create_time,
            </if>
            <if test="updateTime != null">
                update_time,
            </if>
            <if test="blogContent != null">
                blog_content,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="blogId != null">
                #{blogId,jdbcType=BIGINT},
            </if>
            <if test="blogTitle != null">
                #{blogTitle,jdbcType=VARCHAR},
            </if>
            <if test="blogSubUrl != null">
                #{blogSubUrl,jdbcType=VARCHAR},
            </if>
            <if test="blogCoverImage != null">
                #{blogCoverImage,jdbcType=VARCHAR},
            </if>
            <if test="blogCategoryId != null">
                #{blogCategoryId,jdbcType=INTEGER},
            </if>
            <if test="blogCategoryName != null">
                #{blogCategoryName,jdbcType=VARCHAR},
            </if>
            <if test="blogTags != null">
                #{blogTags,jdbcType=VARCHAR},
            </if>
            <if test="blogStatus != null">
                #{blogStatus,jdbcType=TINYINT},
            </if>
            <if test="blogViews != null">
                #{blogViews,jdbcType=BIGINT},
            </if>
            <if test="enableComment != null">
                #{enableComment,jdbcType=TINYINT},
            </if>
            <if test="isDeleted != null">
                #{isDeleted,jdbcType=TINYINT},
            </if>
            <if test="createTime != null">
                #{createTime,jdbcType=TIMESTAMP},
            </if>
            <if test="updateTime != null">
                #{updateTime,jdbcType=TIMESTAMP},
            </if>
            <if test="blogContent != null">
                #{blogContent,jdbcType=LONGVARCHAR},
            </if>
        </trim>
    </insert>
    <update id="updateByPrimaryKeySelective" parameterType="cn.yuyingwai.springbootblog.entity.Blog">
        update tb_blog
        <set>
            <if test="blogTitle != null">
                blog_title = #{blogTitle,jdbcType=VARCHAR},
            </if>
            <if test="blogSubUrl != null">
                blog_sub_url = #{blogSubUrl,jdbcType=VARCHAR},
            </if>
            <if test="blogCoverImage != null">
                blog_cover_image = #{blogCoverImage,jdbcType=VARCHAR},
            </if>
            <if test="blogContent != null">
                blog_content = #{blogContent,jdbcType=LONGVARCHAR},
            </if>
            <if test="blogCategoryId != null">
                blog_category_id = #{blogCategoryId,jdbcType=INTEGER},
            </if>
            <if test="blogCategoryName != null">
                blog_category_name = #{blogCategoryName,jdbcType=VARCHAR},
            </if>
            <if test="blogTags != null">
                blog_tags = #{blogTags,jdbcType=VARCHAR},
            </if>
            <if test="blogStatus != null">
                blog_status = #{blogStatus,jdbcType=TINYINT},
            </if>
            <if test="blogViews != null">
                blog_views = #{blogViews,jdbcType=BIGINT},
            </if>
            <if test="enableComment != null">
                enable_comment = #{enableComment,jdbcType=TINYINT},
            </if>
            <if test="isDeleted != null">
                is_deleted = #{isDeleted,jdbcType=TINYINT},
            </if>
            <if test="createTime != null">
                create_time = #{createTime,jdbcType=TIMESTAMP},
            </if>
            <if test="updateTime != null">
                update_time = #{updateTime,jdbcType=TIMESTAMP},
            </if>
            <if test="blogContent != null">
                blog_content = #{blogContent,jdbcType=LONGVARCHAR},
            </if>
        </set>
        where blog_id = #{blogId,jdbcType=BIGINT}
    </update>
    <update id="updateByPrimaryKeyWithBLOBs" parameterType="cn.yuyingwai.springbootblog.entity.Blog">
        update tb_blog
        set blog_title = #{blogTitle,jdbcType=VARCHAR},
            blog_sub_url = #{blogSubUrl,jdbcType=VARCHAR},
            blog_cover_image = #{blogCoverImage,jdbcType=VARCHAR},
            blog_category_id = #{blogCategoryId,jdbcType=INTEGER},
            blog_category_name = #{blogCategoryName,jdbcType=VARCHAR},
            blog_tags = #{blogTags,jdbcType=VARCHAR},
            blog_status = #{blogStatus,jdbcType=TINYINT},
            blog_views = #{blogViews,jdbcType=BIGINT},
            enable_comment = #{enableComment,jdbcType=TINYINT},
            is_deleted = #{isDeleted,jdbcType=TINYINT},
            create_time = #{createTime,jdbcType=TIMESTAMP},
            update_time = #{updateTime,jdbcType=TIMESTAMP},
            blog_content = #{blogContent,jdbcType=LONGVARCHAR}
        where blog_id = #{blogId,jdbcType=BIGINT}
    </update>
    <update id="updateByPrimaryKey" parameterType="cn.yuyingwai.springbootblog.entity.Blog">
        update tb_blog
        set blog_title = #{blogTitle,jdbcType=VARCHAR},
            blog_sub_url = #{blogSubUrl,jdbcType=VARCHAR},
            blog_cover_image = #{blogCoverImage,jdbcType=VARCHAR},
            blog_category_id = #{blogCategoryId,jdbcType=INTEGER},
            blog_category_name = #{blogCategoryName,jdbcType=VARCHAR},
            blog_tags = #{blogTags,jdbcType=VARCHAR},
            blog_status = #{blogStatus,jdbcType=TINYINT},
            blog_views = #{blogViews,jdbcType=BIGINT},
            enable_comment = #{enableComment,jdbcType=TINYINT},
            is_deleted = #{isDeleted,jdbcType=TINYINT},
            create_time = #{createTime,jdbcType=TIMESTAMP},
            update_time = #{updateTime,jdbcType=TIMESTAMP}
        where blog_id = #{blogId,jdbcType=BIGINT}
    </update>

    <update id="deleteByPrimaryKey" parameterType="java.lang.Long">
        UPDATE tb_blog SET is_deleted = 1
        where blog_id = #{blogId,jdbcType=BIGINT} and is_deleted = 0
    </update>
</mapper>
```

通过以上代码可以看出，在删除操作时并不是执行 delete 语句，而是将需要删除的文章记录的 is_deleted 字段修改为 1，这样就表示该文章已经被执行了删除操作，那么其他的 select 查询语句就需要在查询条件中添加 is_deleted = 0 将“被删除”的记录给过滤出去。



### 编辑页面完善

接下来，把编辑页面按照字段来完善一下，将其他需要输入内容的字段填充到页面 DOM 中，目前编辑页面只有一个编辑框来输入文章字段。某些字段只需要一个 input 框即可，比如文章标题字段，而其他一些字段的输入则需要一些前端插件来完成，比如标签、博客封面图，仅仅是 input 框肯定是无法满足需求的，比如标签字段和分类字段。

#### 引入相关依赖

编辑页面中有如下字段需要使用插件来完善交互：

- 标签字段
- 分类字段
- 文章内容字段(已实现)
- 封面图字段

以上字段所需要的插件也是使用的比较常用的开源插件，插件如下：

- tagsinput（标签）
- select2（分类）
- Editor.md（文章内容）
- ajaxupload（图片上传）

引入插件首先需要把这些依赖文件放到 resources/static/admin/plugins 目录下，目录结构如下：