---
title: RESTful api 设计规范
date: 2020-12-23 22:48:07
categories: Coding
tags: [RESTful]
---



目前比较流行的一套接口规范就是 RESTful api，REST（Representational State Transfer）,中文翻译叫"表述性状态转移",它首次出现在 2000 年 Roy Fielding 的博士论文中，Roy Fielding 是 HTTP 规范的主要编写者之一。他在论文中提到："我这篇文章的写作目的，就是想在符合架构原理的前提下，理解和评估以网络为基础的应用软件的架构设计，得到一个功能强、性能好、适宜通信的架构。REST 指的是一组架构约束条件和原则。"如果一个架构符合 REST 的约束条件和原则，我们就称它为 RESTful 架构，REST 其实并没有创造新的技术、组件或服务，在我的理解中，它更应该是一种理念、一种思想，利用 Web 的现有特征和能力，更好地诠释和体现现有 web 标准中的一些准则和约束。

<!--more-->



## 基本原则

### 基本原则一：URI

- 应该将 api 部署在专用域名之下。
- URL 中尽量不用大写。
- URI 中不应该出现动词，动词应该使用 HTTP 方法表示但是如果无法表示，也可使用动词，例如：search 没有对应的 HTTP 方法,可以在路径中使用 search，更加直观。
- URI 中的名词表示资源集合，使用复数形式。
- URI 可以包含 queryString，避免层级过深。



### 基本原则二：HTTP 动词

对于资源的具体操作类型，由 HTTP 动词表示，常用的 HTTP 动词有下面五个：

- GET：从服务器取出资源（一项或多项）。
- POST：在服务器新建一个资源。
- PUT：在服务器更新资源（客户端提供改变后的完整资源）。
- PATCH：在服务器更新资源（客户端提供改变的属性）。
- DELETE：从服务器删除资源。

还有两个不常用的 HTTP 动词：

- HEAD：获取资源的元数据。
- OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

例子：

```
用户管理模块：

1. [POST]   http：//lou.springboot.tech/users   // 新增
2. [GET]    http：//lou.springboot.tech/users?page=1&rows=10 // 列表查询
3. [PUT]    http：//lou.springboot.tech/users/12 // 修改
4. [DELETE] http：//lou.springboot.tech/users/12 // 删除
```



### 基本原则三：状态码（Status Codes）

处理请求后，服务端需向客户端返回的状态码和提示信息。

常见状态码**(状态码可自行设计，只需开发者约定好规范即可)**：

- 200：SUCCESS 请求成功。
- 401：Unauthorized 无权限。
- 403：Forbidden 禁止访问。
- 410：Gone 无此资源。
- 500：INTERNAL SERVER ERROR 服务器发生错误。 ...



### 基本原则四：错误处理

如果服务器发生错误或者资源不可达，应该向用户返回出错信息。



### 基本原则五：服务端数据返回

后端的返回结果最好使用 JSON 格式，且格式统一。



### 基本原则六：版本控制

- 规范的 api 应该包含版本信息，在 RESTful api 中，最简单的包含版本的方法是将版本信息放到 url 中，如：

  ```
  [GET]    http：//lou.springboot.tech/v1/users?page=1&rows=10
  [PUT]    http：//lou.springboot.tech/v1/users/12
  ```

- 另一种做法是，使用 HTTP header 中的 accept 来传递版本信息。



## 安全原则

以下为接口安全原则的注意事项：

### 安全原则一：Authentication 和 Permission

Authentication 指用户认证，Permission 指权限机制，这两点是使 RESTful api 强大、灵活和安全的基本保障。

常用的认证机制是 Basic Auth 和 OAuth，RESTful api 开发中，除非 api 非常简单，且没有潜在的安全性问题，否则，**认证机制是必须实现的**，并应用到 api 中去。Basic Auth 非常简单，很多框架都集成了 Basic Auth 的实现，自己写一个也能很快搞定，OAuth 目前已经成为企业级服务的标配，其相关的开源实现方案非常丰富。

### 安全原则二：CORS

CORS 即 Cross-origin resource sharing，在 RESTful api 开发中，主要是为 js 服务的，解决调用 RESTful api 时的跨域问题。

由于固有的安全机制，js 的跨域请求时是无法被服务器成功响应的。现在前后端分离日益成为 web 开发主流方式的大趋势下，后台逐渐趋向指提供 api 服务，为各客户端提供数据及相关操作，而网站的开发全部交给前端搞定，网站和 api 服务很少部署在同一台服务器上并使用相同的端口，js 的跨域请求时普遍存在的，开发 RESTful api 时，通常都要考虑到 CORS 功能的实现，以便 js 能正常使用 api。

目前各主流 web 开发语言都有很多优秀的实现 CORS 的开源库，我们在开发 RESTful api 时，要注意 CORS 功能的实现，直接拿现有的轮子来用即可。