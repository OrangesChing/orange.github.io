---
title: Servlet运行环境搭建
date: 2021-07-20 10:30:48
categories:
  - Java
  - Servlet
---

# 不借助IDE集成环境

通过手工的方式编写和发布Servlet程序，有助于理解整个Servlet发布流程

## 准备工作

1. 安装Java

2. 安装Tomcat

3. 将`Servlet-api.jar`包（在Tomcat的安装目录/lib目录）添加到java的classpath变量中（选做）

   **Windows系统**

   在CLASSPATH环境变量中增加`%CATALINA_HOME%\lib\servlet-api.jar`，注意是否配置了`CATALINA_HOME`环境变量

   ![image-20210624174243724](Servlet运行环境搭建\image-20210624174243724.png)

   ![image-20210624185101078](Servlet运行环境搭建\image-20210624185101078.png)


   **Linux系统**

   打开`/etc/profile`

   `CLASSPATH`追加`servlet-api`的路径，注意替换tomcat路径：

   `export CLASSPATH=.:$JAVA_HOME/lib/:/usr/local/soft/tomcat8081/lib/servlet-api.jar`

   重新加载配置文件`source /etc/profile`

   > classpath是jvm查找.class文件的路径

**注意：**

- 下述教程将使用`%CATALINA_HOME%`来代替Tomcat根目录，注意替换
- 若Tomcat安装在C盘则换个盘执行“编写相关代码”和“编译Serlet”步骤，否则在编译时可能没有写权限

## 构建Web项目

1、在`%CATALINA_HOME%\webapps`下，新建一个`helloServlet文件夹`，Tomcat会自动发布这个文件夹下的应用(若windows且tomcat装在C盘则随便找个非C盘的路径，不然后续编译可能有问题)

2、在`helloServlet`目录下新建一个子目录`WEB-INF`，`WEB-INF`是`JavaWEB`应用的安全目录，注意文件名要一模一样

3、在`helloServlet\WEB-INF`目录下新建两个子目录，为`classes`和`lib`

## 编写Servlet代码

在`helloServlet\WEB-INF\classes`目录下建立`helloServlet.java`文件，代码如下：

```java
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


public class HelloServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    public HelloServlet() {
        super();       //调用父类的构造方法
    }
    
    protected void doGet(HttpServletRequest request, HttpServletResponse response)  
      throws ServletException, IOException {
        doPost(request,response);
    }
    
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
     throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out=response.getWriter();
        out.println("<b>Hello Servlet!</b>");    //向客户端输出Hello Servlet
    }
}

```

## 编译Servlet

1. 打开cmd，进入`helloServlet\WEB-INF\classes`目录

2. 输入`javac`命令编译：

   若配置了classpath，使用命令：`javac HelloWorld.java`
若没有配置，则使用命令：
   
   Windows：`javac -classpath ".;%CATALINA_HOME%\lib\servlet-api.jar" HelloServlet.java`
   
   Linux：`javac -classpath ".:%CATALINA_HOME%/lib/servlet-api.jar" HelloServlet.java`
- `-classpath参数`用来配置环境变量classpath的值，` . `表示当前目录，`%CATALINA_HOME%\lib\servlet-api.jar`表示`servlet路径`
- `-d`参数用来设置编译后classes文件的目录，编译时也可不加参数，但一定要将编译后的.class文件放在WEB-INF目录下的classes文件夹里（连同所在的包）

## 部署Servlet

1. 从`webapps\examples\WEB-INF`目录中复制`web.xml`文件到`HelloWorld\WEB-INF`目录下，只保留web-app标签和xml，其他内容删除，即：（保险起见不要自己复制创建，复制最好！！！）

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                         http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
     version="4.0"
     metadata-complete="true">
   
   
   </web-app>
   ```

   

2. 编写的`Servlet`配置访问路径，当访问该webapp配置的路径时(url-pattern)时，Tomcat会去classes目录找到对应的class执行响应。内容如下：
   
   
   
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                         http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
     version="4.0"
     metadata-complete="true">
   
       <!-- 定义Servlet本身的属性 -->
       <servlet>
           <!-- 声明Servlet的名称，随便写，就是安个名 -->
           <servlet-name>HelloServlet</servlet-name>
           <!-- 声明Servlet所对应的类名，有包的加上包名packageName.className -->
           <servlet-class>HelloServlet</servlet-class>
       </servlet>
       
       <!-- 用于进行Servlet映射 -->
       <servlet-mapping>
           <!-- 与上面的servlet-name名字一致 -->
           <servlet-name>HelloServlet</servlet-name>
           <!-- 用于指明Servlet的访问地址，最终在浏览器访问的地址 -->
           <url-pattern>/helloServlet</url-pattern>
       </servlet-mapping>
   </web-app>
   ```
   
   PS：在Servlet3.0规范之前需要编写上述web.xml文件对Servlet进行配置。在Servlet3.0版本规范中支持注解进行配置，即在源代码中加入注解`@WebServlet(urlPatterns={“/helloServlet”})`

3. 若不在`%CATALINA_HOME%\webapps`下，则将`HelloWorld`文件夹复制到Tomcat中的`webapps`文件夹`%CATALINA_HOME%\webapps`

## 验证结果

1. 打开cmd使用startup.bat启动Tomcat服务器

2. 在浏览器地址栏输入`http://localhost/helloServlet/helloServlet`
   链接中localhost后的helloServlet是项目所在的目录，/helloServlet是web.xml文件中（url-pattern）标签里的内容

![image-20210716161614491](Servlet运行环境搭建\image-20210716161614491.png)

# IDEA

## 构建/配置WEB项目

### 新建模块

#### 新建一个模块ServletLearn

![image-20210716172753962](Servlet运行环境搭建\image-20210716172753962.png)

![image-20210716173221114](Servlet运行环境搭建\image-20210716173221114.png)

使用该框架格式会给你创建WEB-INF文件夹和web.xml

![image-20210716173459747](Servlet运行环境搭建\image-20210716173459747.png)

#### 创建classes文件夹和lib文件夹

![image-20210716173710794](Servlet运行环境搭建\image-20210716173710794.png)

![image-20210716174140213](Servlet运行环境搭建\image-20210716174140213.png)

### 配置项目

#### 设置编译后的class文件放置的路径

![image-20210716174731001](Servlet运行环境搭建\image-20210716174731001.png)

#### 设置项目运行环境Tomcat

![image-20210716175553504](Servlet运行环境搭建\image-20210716175553504.png)

![image-20210716181226976](Servlet运行环境搭建\image-20210716181226976.png)

#### 配置部署方式

![image-20210716181324936](Servlet运行环境搭建\image-20210716181324936.png)

> Artifact详细解说：https://developer.aliyun.com/article/659686

### 测试配置结果

运行改项目，会弹出浏览器，显示的是index.jsp页面的内容

![image-20210716181740757](Servlet运行环境搭建\image-20210716181740757.png)

![image-20210716181715427](Servlet运行环境搭建\image-20210716181715427.png)

## 创建Servlet

### 导入servlet-api包

![image-20210716175200990](Servlet运行环境搭建\image-20210716175200990.png)

![image-20210716175234123](Servlet运行环境搭建\image-20210716175234123.png)

### 编写Servlet

```java
package com.servletLearn;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Date;

public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 向浏览器输出内容
        // 设置编码
        response.setContentType("text/html;charset=utf-8");
        response.getWriter().write("Hello Servlet  ");
        response.getWriter().write("当前系统时间是："+new Date());
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}

```

### 配置Servlet

在web.xml的`web-app`标签里，添加以下配置，即配置Servlet名、响应路径、响应类

```xml
<servlet>
	<servlet-name>HelloServlet</servlet-name>
	<servlet-class>com.servletLearn.HelloServlet</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>HelloServlet</servlet-name>
	<url-pattern>/HelloServlet</url-pattern>
</servlet-mapping>
```

## 测试最终效果

启动项目访问`http://localhost:8080/ServletLearn/HelloServlet`

![image-20210719092521276](Servlet运行环境搭建\image-20210719092521276.png)

## 注意点

Project SDK与Project language level要保持一致，否则会报`java: 无效的源发行版: 8`

![image-20210719092406471](Servlet运行环境搭建\image-20210719092406471.png)