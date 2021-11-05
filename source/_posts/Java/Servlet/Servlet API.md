---
title: Servlet API
date: 2021-07-21 10:30:48
categories:
  - Java
  - Servlet
---

# Servlet API 概览
Servlet API 包含以下4个Java包：

- javax.servlet：servlet和servlet容器之间契约的类和接口

- javax.servlet.http：HTTP Servlet 和Servlet容器之间的关系

- javax.servlet.annotation：Servlet、Filter、Listener的标注。它还为被标注元件定义元数据
- javax.servlet.descriptor：提供程序化登录Web应用程序的配置信息的类型

# javax.servlet

## Servlet接口

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();
     
    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
     
    String getServletInfo();
     
    void destroy();
}
```

`init`、`service`、`destroy`对应了Servlet生命周期的三个阶段，Servlet容器会调用Servlet接口的这三个方法

## ServletRequset接口

Servlet容器对于接受到的每一个Http请求，都会创建一个`ServletRequest对象`，用于封装了关于这个请求的所有信息包括

- 请求行信息
- 请求头信息
- 请求正文
- 网络连接信息
- 请求域属性

ServletRequest接口的部分内容：
```
public interface ServletRequest {
    int getContentLength();//返回请求主体的字节数
    String getContentType();//返回主体的MIME类型
    String getParameter(String var1);//返回请求参数的值
}
```

 

 ## ServletResponse接口

Servlet容器对于接受到的每一个Http请求，都会创建一个`ServletResponse对象`，用于响应请求。ServletResponse隐藏了向浏览器发送响应的复杂过程。

重要方法如下：


```
public interface ServletResponse {
    // 用于发送二进制数据，返回一个二进制流对象
    ServletOutputStream getOutputStream() throws IOException;
    
    // 用于发送HTML数据，返回了一个可以向客户端发送文本的的Java.io.PrintWriter对象，改对象使用ISO-8859-1编码
    PrintWriter getWriter() throws IOException;
}
```



## ServletConfig接口

ServletConfig接口
    当Servlet容器初始化Servlet时，Servlet容器会给Servlet的init( )方式传入一个ServletConfig对象。

其中几个方法如下：

```
public interface ServletConfig {
    // 获取servlet在web.xml中配置的name值
    String getServletName();

    ServletContext getServletContext();
	// 获取初始化参数
    String getInitParameter(String var1);
	// 获取初始化参数名称
    Enumeration<String> getInitParameterNames();
}
```

 

## ServletContext接口
ServletContext对象表示Servlet应用程序。ServeltContext是一个上下文对象或者说全局对象，包含了有关应用的许多信息，它的数据供所有的应用程序共享，即应用的所有组件都可以从ServletContext获取当前应用的状态信息。一个应用程序有且只有一个ServeltContext，随着程序的启动而创建，随着程序的停止而销毁。

通过在ServletConfig中调用getServletContext方法，也可以获得ServletContext对象。

那么为什么要存在一个ServletContext对象呢？存在肯定是有它的道理，因为有了ServletContext对象，就可以共享从应用程序中的所有资料处访问到的信息，并且可以动态注册Web对象。前者将对象保存在ServletContext中的一个内部Map中。保存在ServletContext中的对象被称作属性。

ServletContext中的下列方法负责处理属性：

```
Object getAttribute(String var1);

Enumeration<String> getAttributeNames();

void setAttribute(String var1, Object var2);

void removeAttribute(String var1);
```

## ServletContextListener接口

`ServletContextListener`是对`ServeltContext`的一个监听

```java
public interface ServletContextListener extends EventListener {
    // servelt容器启动,serveltContextListener会调用contextInitialized方法
    void contextInitialized(ServletContextEvent var1);
    // servelt容器关闭,serveltContextListener会调用contextDestroyed方法
    void contextDestroyed(ServletContextEvent var1);
}
```

> Spring中的使用：
>
> 1. 结构分析：监听器类`org.springframework.web.context.ContextLoaderListener`实现了`ServletContextListener接口`。其中`contextInitialized(ServletContextEvent var1)`方法调用了`initWebApplicationContext(event.getServletContext())`
> 2. 当Servlet容器启动时，ServletContext对象被初始化，然后Servlet容器调用web.xml中注册监听器的`contextInitialized方法`，而在监听器中，调用了`initWebApplicationContext方法`，在这个方法中实例化了Spring IOC容器。即ApplicationContext对象。
>
> 因此，当ServletContext创建时我们可以创建applicationContext对象，当ServletContext销毁时，我们可以销毁applicationContext对象。这样applicationContext就和ServletContext“共生死了”

## GenericServlet抽象类 

前面介绍了各种接口，若使用实现Servlet接口的方式来编写代码，则必须要实现Servlet接口中定义的所有的方法，即使有一些方法中没有任何东西也要去实现，并且还需要自己手动的维护ServletConfig这个对象的引用。因此，这样去实现Servlet是比较麻烦的

`GenericServlet抽象类`的出现很好的解决了这个问题。`GenericServlet`实现了`Servlet接口`和`ServletConfig接口`

`GenericServlet抽象类`相比于直接实现`Servlet接口`，有以下几个好处：

1. 为Servlet接口中的所有方法提供了默认的实现
2. 提供方法，包围ServletConfig对象中的方法
3. 将init( )方法中的ServletConfig参数赋给了一个内部的ServletConfig引用从而来保存ServletConfig对象，不需要程序员自己去维护ServletConfig了

虽然GenricServlet是对Servlet一个很好的加强，但是也不经常用

# javax.servlet.http

## HttpServlet抽象类

HttpServlet继承了GenericServlet抽象类，是对GenericServlet的扩展
HttpServlet之所以运用广泛的原因是现在大部分的应用程序都要与HTTP结合起来使用。这意味着我们可以利用HTTP的特性完成更多更强大的任务

以下为HttpServlet对GenericServlet中service()方法的实现，可看出，HttpServlet将Servlet接口的输入ServletRequest 和ServletResponse转化为了HttpServletRequest和HttpServletResponse，然后根据不同的请求方式调用不用的接口，使用者只需实现具体的请求方法对应的接口如：doGet()，doPost()等

```java
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
    HttpServletRequest request;
    HttpServletResponse response;
    try {
        request = (HttpServletRequest)req;
        response = (HttpServletResponse)res;
    } catch (ClassCastException var6) {
        throw new ServletException(lStrings.getString("http.non_http"));
    }

    this.service(request, response);
}

protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getMethod();
        long lastModified;
        if (method.equals("GET")) {
            lastModified = this.getLastModified(req);
            if (lastModified == -1L) {
                this.doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader("If-Modified-Since");
                } catch (IllegalArgumentException var9) {
                    ifModifiedSince = -1L;
                }

                if (ifModifiedSince < lastModified / 1000L * 1000L) {
                    this.maybeSetLastModified(resp, lastModified);
                    this.doGet(req, resp);
                } else {
                    resp.setStatus(304);
                }
            }
        } else if (method.equals("HEAD")) {
            lastModified = this.getLastModified(req);
            this.maybeSetLastModified(resp, lastModified);
            this.doHead(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        } else if (method.equals("PUT")) {
            this.doPut(req, resp);
        } else if (method.equals("DELETE")) {
            this.doDelete(req, resp);
        } else if (method.equals("OPTIONS")) {
            this.doOptions(req, resp);
        } else if (method.equals("TRACE")) {
            this.doTrace(req, resp);
        } else {
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[]{method};
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(501, errMsg);
        }

    }
```

## HttpServletRequest接口

HttpServletRequest对象表示Http环境中的Servlet请求。HttpServletRequest接口继承了ServletRequest接口，是对ServletRequest接口的拓展

可以通过ServletRequest接口分别获得HTTP请求的请求行，请求头和请求体

![img](Servlet API\20180513130638615)

### 获取请求行

获得客户端的请求方式：

`String getMethod()`

获得请求的URL：

`String getRequestURI()`

`StringBuffer getRequestURL()`

获取请求的web应用的名称：

`String getContextPath()` 

获取URL地址后的参数字符串

`String getQueryString()` 

### 获取请求头

请求头都是key-value形式

`long getDateHeader(String name)`

`String getHeader(String name)`

`Enumeration getHeaderNames()`

`Enumeration getHeaders(String name)`

`int getIntHeader(String name)`

### 获取请求体

请求体中的内容是通过post提交的请求参数

`String getParameter(String name)`

`String[] getParameterValues(String name)`

`Enumeration getParameterNames()`

`Map<String,String[]> getParameterMap()`

 注意：get请求方式的请求参数 上述的方法一样可以获得

## HttpServletResponse接口

HttpServletRequest对象表示Http环境中的Servlet响应。HttpServletResponse接口继承了ServletResponse接口，是对ServletRequest接口的拓展

由于HTTP请求消息分为状态行，响应消息头，响应消息体三部分，因此，在HttpServletResponse接口中定义了向客户端发送响应状态码，响应消息头，响应消息体的方法

![img](Servlet API\20180513203525684)

### 响应头设置

`addDateHeader(String name, long date)`

`addHeader(String name, String value)`

`addIntHeader(String name, int value)`

`setHeader(String name, String value)`

`setDateHeader(String name, long date)`

`setIntHeader(String name, int value)`

设置Cookie推荐使用：`addCookie(Cookie var1)`

### 设置响应状态码

`setStatus(int sc)`

 ###  设置响应数据

所有响应数据将写入Response缓存，Tomcat组装HTTP响应时会从这个缓存中取数据

`PrintWriter getWriter()`

获得字符流，通过字符流的write(String s)方法可以将字符串设置到response缓冲区中，随后Tomcat会将response缓冲区中的内容组装成Http响应返回给浏览器端

`ServletOutputStream getOutputStream()`

获得字节流，通过该字节流的write(byte[] bytes)可以向response缓冲区中写入字节，再由Tomcat服务器将字节内容组成Http响应返回给浏览器

**注意：**虽然response对象的`getOutSream()`和`getWriter()`方法都可以发送响应消息体，但是他们之间相互排斥，不可以同时使用，否则会发生异常

## 数据乱码问题

**乱码原因：**数据是通过二进制的形式传输、存储的，所以当数据传输时，会发生字符和字节之间的转换，而码表记录了字符与字节的对应，不同的编码(如：UTF-8, GBK等)对应不同的码表。<u>如果编码和解码使用的码表不一致，就会导致乱码</u>

### 服务器接收数据乱码问题

**问题分析：**由于在service中的默认解码方式为ISO-8859-1（不支持中文），而浏览器发送数据由UTF-8编码。若没有特殊设置，将使用UTF-8编码，ISO-8859-1解码，所以导致乱码

**解决方法：**手动设置解码为UTF-8

- post提交方式的乱码：`request.setCharacterEncoding("UTF-8");`
- get提交方式的乱码：`parameter = newString(parameter.getbytes("iso8859-1"),"utf-8");`

### 服务器发送数据乱码问题

**问题分析：**由于response缓存区默认编码方式为ISO-8859-1（不支持中文），而接收端浏览器使用GB2312解码。若没有特殊设置，将使用ISO-8859-1编码，GB2312解码，所以导致乱码

**解决方法：**手动设置response解码方式为UTF-8，通知浏览器使用UTF-8解码

方式一：

1. 设置response解码方式：`response.setCharacterEncoding("UTF-8");`
2. 通知浏览器使用编码：`response.setHeader("Content-Type", "text/html;charset=utf-8")`

方式二：

一句设置代替方式一的两句：`response.setContentType("text/html;charset=utf-8");`

## 总结：HTTP请求响应步骤

1. Tomcat解析客户端请求封装为ServletRequest对象，并创建一个ServletResponse对象
2. 调用service方法，传入ServletRequest、ServletResponse对象
3. 转换为HttpServletRequest对象和HttpServletResponse对象，根据请求方式调用响应函数
4. 响应函数使用`getWriter().write("hello servlet")`将数据写入response缓存中
5. Tomcat获取ServletResponse对象和response缓存数据，组装为HTTP响应报文，发送到客户端

参考：

https://blog.csdn.net/qq_19782019/article/details/80292110