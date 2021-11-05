---
title: Servlet概述与基础
date: 2021-07-19 10:30:48
categories:
  - Java
  - Servlet
---

# 什么是Servlet

Servlet（Server Applet），全称Java Servlet，是用Java编写的服务器端程序

主要功能在于交互式地浏览和修改数据，生成动态Web内容

狭义的Servlet是指Java语言实现的一个接口

广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者

Servlet运行于支持Java的应用服务器中。从实现上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器

# Servlet 的工作机制

1. 客户端发送请求至服务器，请求经过Tomcat层层处理，到达Servlet容器
2. Servlet容器调用Servlet的`Service()`方法，并传入一个`ServletRequest对象`和一个`ServletResponse对象``ServletRequest对象`和`ServletResponse对象`都是由`Servlet容器`（例如TomCat）封装好的、可直接使用的两个对象。`ServletRequest`中封装了当前的Http请求，`ServletResponse`表示当前用户的Http响应。Servlet响应内容写入`ServletResponse`
3. 服务器将响应返回客户端

# Servlet 的生命周期

一个Servelt的生命周期包括：`创建init()`、`服务service()`、`销毁destroy()`

Servlet容器（如Tomcat）会根据下面的规则来调用这三个方法

- `创建init()`：<u>只在第一次</u>请求Servlet时执行
- `服务service()`：<u>每次</u>请求Servlet时执行
- `销毁destroy()`：只在卸载应用程序或者关闭Servlet容器时执行，一般在这个方法中会写一些清除代码

# Servlet的使用

Servlet技术的核心是Servlet接口，它是所有Servlet类必须直接或者间接实现的一个接口。Servlet接口定义了Servlet与servlet容器之间的契约。

这个契约是：Servlet容器将Servlet类载入内存，并产生Servlet实例和调用它具体的方法。但是要注意的是，在一个应用程序中，每种Servlet类型只能有一个实例。

在编写实现Servlet的Servlet类时，直接实现它。在扩展实现这个这个接口的类时，间接实现它