---
title: Tomcat原理
date: 2021-06-21 16:22:23
tags:
  - 原理篇
  - Web基础
categories:
  - 服务器配置与管理
  - Tomcat
---

# Tomcat结构图

![img](Tomcat原理/20180308114704839.png)

Tomcat主要组件：

- 服务器Server
- 服务Service
- 连接器Connector（核心）
- 容器Container（核心）

Tomcat 还有其它重要的组件，如安全组件 security、logger 日志组件、session、mbeans、naming 等其它组件。这些组件共同为 Connector 和 Container 提供必要的服务

一个Container容器和一个或多个Connector组合在一起，加上其他一些支持的组件共同组成一个Service服务，有了Service服务便可以对外提供能力了

但是Service服务的生存需要一个环境，这个环境便是Server，Server组件为Service服务的正常使用提供了生存环境，Server组件可以同时管理一个或多个Service服务

# 两大组件

## Connector

一个Connecter将在某个指定的端口上监听客户请求，接收浏览器的发过来的`TCP连接请求`，创建一个 `Request` 和 `Response` 对象分别用于和请求端交换数据，然后会产生一个线程来处理这个请求并把产生的 `Request` 和 `Response` 对象传给`Engine`(Container中的一部分，见Container组件的介绍)，从`Engine`处获得响应并返回客户

Tomcat中有两个经典的Connector

- 一个监听来自浏览器的HTTP请求。`HTTP/1.1 Connector`在端口8080处侦听来自客户端浏览器的HTTP请求

- 一个监听来自其他HTTP服务器请求。`AJP/1.3 Connector`在端口8009处侦听其他HTTP服务器的`Servlet`/`JSP`请求

Connector 最重要的功能就是接收连接请求，然后分配线程让Container来处理这个请求，所以这必然是多线程的，多线程的处理是 Connector 设计的核心

> **AJP**
>
> （1）为什么Tomcat需要监听其他HTTP服务器请求？
>
> 由于Tomcat的HTML和图片解析功能相对其他服务器如Apache、IIS等较弱。因此在实际应用中，常常把Tomcat与其他HTTP服务器集成。对于不支持Servlet/JSP的HTTP服务器，可以通过Tomcat服务器来运行`Servlet`/`JSP`组件。而Tomcat和其他服务器的集成，就是通过`AJP协议`来完成的。
>
> ![AJP与HTTP](Tomcat原理\AJP与HTTP.jpg)
>
> （2）AJP是什么？
>
> AJP是一个二进制的TCP传输协议，相比HTTP这种纯文本的协议来说，效率和性能更高，也做了很多优化。使WEB服务器可通过TCP连接与`Servlet`容器连接
>
> PS:
>
>    ​       浏览器并不能直接支持AJP13协议，只支持HTTP协议。所以实际情况是，通过Apache的`proxy_ajp`模块进行反向代理，暴露成http协议给客户端访问。其他支持AJP协议的代理服务器当然也可以用这种做法。但是实际情况是，支持AJP代理的服务器非常少，比如目前很火爆的Nginx就没这个模块
>    ​       因此Tomcat的配置大部分都是关闭AJP协议端口的，因为除了Apache之外别的HTTP服务器几乎都不能反代AJP13协议，自然就没太大用处了



## Container

![在这里插入图片描述](Tomcat原理\20200407023751702.png)

Container是容器的父接口，该容器的设计用的是典型的<u>责任链设计模式</u>，它由四个自容器组件构成：Engine、Host、Context、Wrapper。这四个组件是负责关系，存在包含关系

1. Engine可以管理多个主机，一个Service最多只能有一个Engine
2. Host是虚拟主机，通过配置Host就可以添加多台主机
3. Context就意味着一个WEB应用程序
4. Wrapper是Tomcat对Servlet的一层封装，通常一个Servlet Class对应一个Wrapper

**Engine 容器（顶层）**

 定义了一些基本的关联关系

**Host 容器（二层）**

一个 Host 在 Engine 中代表一个虚拟主机，这个虚拟主机的作用就是运行多个应用，它负责安装和展开这些应用，并且标识这个应用以便能够区分它们。它的子容器通常是 Context，它除了关联子容器外，还有就是保存一个主机应该有的信息

**Context 容器（三层）**

Context 代表 Servlet 的 Context，它具备了 Servlet 运行的基本环境，<u>理论上只要有 Context 就能运行 Servlet 了</u>。简单的 Tomcat 可以没有 Engine 和 Host

Context 最重要的功能就是管理它里面的 Servlet 实例，Servlet 实例在 Context 中是以 Wrapper 出现的

Context 如何才能找到正确的 Servlet 来执行它呢？ Tomcat5 以前是通过一个 Mapper 类来管理的，Tomcat5 以后这个功能被移到了 request 中

**Wrapper 容器（底层）**
Wrapper 代表一个 `Servlet`，它负责管理一个 `Servlet`，包括的 `Servlet` 的装载、初始化、执行以及资源回收
Wrapper 的实现类是 StandardWrapper，StandardWrapper 还实现了拥有一个 Servlet 初始化信息的 ServletConfig，由此看出 <u>StandardWrapper 将直接和 Servlet 的各种信息打交道</u>。

# HTTP请求过程

![img](Tomcat原理\20180308173032224.png)

1. 用户在浏览器中访问 `localhost:8080/test/index.jsp`，请求被发送到本机端口8080，被在那里监听的`Coyote HTTP/1.1 Connector`获得
2. Connector把该请求交给它所在的Service的Engine来处理，并等待Engine的回应
3. Engine获得请求`localhost/test/index.jsp`，匹配所有的虚拟主机Host。Engine匹配到名为`localhost`的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机），名为`localhost`的Host获得请求`/test/index.jsp`
4. 名为`localhost`的Host匹配它所拥有的所有的Context。Host匹配到路径为`/test`的Context（如果匹配不到就把该请求交给路径名为`“ ”`的Context去处理） 
5. `path=“/test”`的Context获得请求`/index.jsp`，在它的`mapping table`中寻找出对应的Servlet。Context匹配到URL模式为`*.jsp`的Servlet
6. 模式为`*.jsp`的Servlet对应于`JspServlet类`，构造`HttpServletRequest`对象和`HttpServletResponse`对象，作为参数调用`JspServlet`的`doGet()`或`doPost()`。执行业务逻辑、数据存储等程序
7. Context把执行后的`HttpServletResponse`对象返回给Host
8. Host把`HttpServletResponse`对象返回给Engine
9. Engine把`HttpServletResponse`对象返回Connector
10. Connector把`HttpServletResponse`对象返回给客户浏览器

参考整理自：

https://blog.csdn.net/u014231646/article/details/79482195

https://blog.csdn.net/z69183787/article/details/97244810