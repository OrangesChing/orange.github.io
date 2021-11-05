---
title: Java操作关系型数据库
date: 2020-08-25 08:59:43
tags:
  - 概述
  - ORM框架
  - JDBC
categories:
  - Java
  - Java与数据库
---

JAVA的数据库操作可视为以下演变图，ORM框架中可以加入池化技术，ORM框架的底层都是JDBC实现，只是为了方便开发和解耦封装。

![数据库操作演变](Java操作关系型数据库/数据库操作演变.png)

# 原生JDBC

JDBC是Java数据库连接，全称是Java Database Connectivity，简称JDBC，是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，提供了诸如查询和更新数据库中数据的方法。JDBC也是Sun Microsystems的商标。我们通常说的JDBC是面向关系型数据库的

## 开发步骤

1. 注册驱动.，告知JVM使用的是哪一个数据库的驱动

   -参数：“驱动程序类名”  

   -`Class.forname("驱动程序类名")`

2. 获得连接.，使用JDBC中的类，完成对MySQL数据库的连接

   需要三个参数：`url`, `username`, `password `- 连接到数据库

3. 获得语句执行平台，通过连接对象获取对SQL语句的执行者对象

   -`conn.getStatemnent()`方法创建对象用于执行SQL语句

4. 执行SQL语句，使用执行者对象，向数据库执行SQL语句 获取到数据库的执行后的结果

   -`execute(ddl)`执行任何SQL，常用于执行DDL、DCL

   -`executeUpdate(dml)`执行DML语句，如：`insert`，`update`，`delete`

   -`executeQuery(dql) `执行DQL语句，如：`select`

5. 处理结果

   -`execute(ddl)`如果没有异常则成功 boolean

   -`executeUpdate(dml)`返回数字，表示更新“行”数量，抛出异常则失败  int

   -`executeQuery(dql)`返回`ResultSet`（结果）对象，代表2维查询结果， `ResultSet`使用`for`遍历处理，如果查询失败抛出异常

6. 释放连接

   -`conn.close();`

注意写代码之前，要导入数据库驱动包，连接不同厂商的数据库要用不同的驱动包

![JDBC](Java操作关系型数据库/JDBC.png)

示例代码如下：

```java
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Properties;

import com.mysql.jdbc.ResultSetMetaData;

/**
 * 
 * @author dgm
 * @describe "原生jdbc"
 * @date 2020年4月13日
 */
public class MysqlTest {
    // JDBC 驱动名及数据库 URL
    static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    static final String DB_URL = "jdbc:mysql://192.168.8.200:3306/bdrackdemo?useUnicode=true&characterEncoding=utf8&autoReconnect=true";
    // 数据库的用户名与密码，需要根据自己的设置
    static final String USER = "root";
    static final String PASS = "cstorfs";
    static Properties prop = new Properties();

    //读取数据库配置文件
    static void readDBSetting(String path) {
        Properties prop = new Properties();
        // 读取属性文件mysql.properties
        InputStream in = null;
        try {
            in = new BufferedInputStream(new FileInputStream(path));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        // 加载属性列表
        try {
            prop.load(in);
        } catch (IOException e) {
            e.printStackTrace();
        } 
        Iterator<String> it = prop.stringPropertyNames().iterator();
        while (it.hasNext()) {
            String key = it.next();
            System.out.println(key + "=" + prop.getProperty(key));
        }
        try {
            in.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        // 读取mysql 配置信息
        readDBSetting("conf/mysql.properties");

        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        try {
            // 注册 JDBC 驱动
            Class.forName(prop.getProperty("dbDriver")).newInstance();

            // 打开链接
            System.out.println("连接数据库...");
            conn = DriverManager
                    .getConnection(
                            "jdbc:mysql://"
                                    + prop.getProperty("mysqlhost")
                                    + ":"
                                    + prop.getProperty("mysqlport")
                                    + "/"
                                    + prop.getProperty("dbname")
                                    + "?useUnicode=true&characterEncoding=utf8&autoReconnect=true",
                            prop.getProperty("mysqluser"),
                            prop.getProperty("mysqlpwd"));

            // 执行查询
            System.out.println(" 实例化Statement对象...");
            stmt = conn.createStatement();
            String sql = "SELECT id, username, number FROM student";
            rs = stmt.executeQuery(sql);

            // 展开结果集数据库
            while (rs.next()) {
                // 输出数据
                System.out.print("用户id: " + rs.getInt("id"));
                System.out.print(", 用户名: " + rs.getString("username"));
                System.out.print(", 学号: " + rs.getString("number"));
                System.out.print("\n");
            }
        } catch (SQLException se) {
            se.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 关闭资源
            try {
                if (rs != null)
                    rs.close();
            } catch (SQLException se3) {
            }
            try {
                if (stmt != null)
                    stmt.close();
            } catch (SQLException se2) {
            }
            try {
                if (conn != null)
                    conn.close();
            } catch (SQLException se) {
                se.printStackTrace();
            }
        }
    }
}
```

早期开发将上述代码自己封装成JDBC工具，传入SQL语句。这种模式在规模不大的情况下，使用起来挺好

## 开发技巧

1. 使用PrearedStatement，可以通过预编译的方式避免在拼接SQL时造成SQL注入。
2. 使用ConnectionPool
3. 禁用自动提交
   这个最佳实践在我们使用JDBC的批量提交的时候显得非常有用，将自动提交禁用后，你可以将一组数据库操作放在一个事务中，而自动提交模式每次执行SQL语句都将执行自己的事务，并且在执行结束提交。
4. 使用Batch Update：批量更新/删除，比单个更新/删除，能显著减少数据传输的往返次数，提高性能。
5. 使用列名获取ResultSet中的数据，从而避免invalidColumIndexError
   JDBC中的查询结果封装在ResultSet中，我们可以通过列名和列序号两种方式获取查询的数据，当我们传入的列序号不正确的时候，就会抛出invalidColumIndexException，例如你传入了0，就会出错，因为ResultSet中的列序号是从1开始的。另外，如果你更改了数据表中列的顺序，你也不必更改JDBC代码，保持了程序的健壮性。有一些Java程序员可能会说通过序号访问列要比列名访问快一些，确实是这样，但是为了程序的健壮性、可读性，我还是更推荐你使用列名来访问。
6. 使用变量绑定而不是字符串拼接
   使用PreparedStatment可以防止注入，而使用`?`或者其他占位符也会提升性能，因为这样数据库就可以使用不同的参数执行相同的查询，提示性能，也防止SQL注入。
7. 关闭Connection 等资源，确保资源被释放；
8. 选择合适的JDBC驱动，参考前文，选择第四种；
9. 尽量使用标准的SQL语句，从而在某种程度上避免数据库对SQL支持的差异
   不同的数据库厂商的数据库产品支持的SQL的语法会有一定的出入，为了方便移植，推荐使用标准的ANSI SQL标准写SQL语句。
10. 使用正确的getXXX()方法
    当从ResultSet中读取数据的时候，虽然JDBC允许你使用getString()和getObject()方法获取任何数据类型，推荐使用正确的getter方法，这样可以避免数据类型转换。

## 连接步骤

JAVA操作关系型数据库![JDBC连接](Java操作关系型数据库/JDBC连接.png)

上图大致画出以访问MySQL为例，执行一条 SQL 命令的流程：

1. TCP建立连接的三次握手；
2. MySQL认证的三次握手；
3. 真正的SQL执行；
4. MySQL的关闭；
5. TCP的四次握手关闭；

## 优缺点

由上图可知，为了执行一条SQL，有大量的网络交互，性价比低
**优点**：实现简单
**缺点**：

1. 网络IO较多
2. 数据库的负载较高
3. 响应时间较长及QPS较低
4. 应用频繁的创建连接和关闭连接，导致临时对象较多，GC频繁
5. 在关闭连接后，会出现大量TIME_WAIT 的TCP状态(在2个MSL之后关闭)。
6. 大项目中使用很复杂
7. 没有封装
8. 难以实现 MVC 的概念
9. 很大的编程成本

# 池化数据库连接的JDBC

原生JDBC的瓶颈在于数据库连接，通过池化技术可解决

## 池化连接

> 池化技术简单点说就是预先连接好一定数量的连接，等需要时随意从中选择一个进行操作。比如线程池。

连接池负责：连接建立、连接释放、连接管理、连接分配。

**数据库连接池运行机制**
系统初始化时创建连接池，程序操作数据库时从连接池中获取空闲连接，程序使用完毕将连接归还到连接池中，系统退出时，断开所有数据库连接并释放内存资源。

**使用池化连接的好处**

1. 性能。连接到数据库既昂贵又缓慢。池化连接可以保持与数据库的物理连接，并在需要数据库访问的各个组件之间共享。这样一来，连接成本便一次性支付，并在所有消耗组件中摊销。
2. 诊断。如果您有一个子系统负责连接到数据库，则诊断和分析数据库连接使用情况将变得更加容易。
3. 可维护性。同样，如果您有一个子系统负责分发数据库连接，则与每个组件都连接到数据库本身相比，代码将更易于维护。

**为什么池化连接能带来性能上的提高**

因为无论何时请求一个连接，池数据源会从可用的连接池获取新连接。仅当没有可用的连接而且未达到最大的连接数时连接池将创建新的连接。`close()`方法把连接返回到连接池而不是真正地关闭它

重用数据库连接最明显的原因：

- 减少应用程序和数据库管理系统创建/销毁TCP连接的OS I/O开销
- 减少JVM对象垃圾
- 缓冲安全：连接池是即将到来的连接请求的有界缓冲区。如果出现瞬间流量尖峰，连接池会平缓这一变化，而不是使所有可用数据库资源趋于饱和。
- 等待步骤和超时机制，可有效防止数据库服务器过载。如果一个应用消耗太多数据库流量，为防止它将数据库服务器压垮，连接池将减少它对数据库的使用。

## 工具

- C3P0：开源JDBC连接池，实现数据源和JNDI绑定，包括实现jdbc3和jdbc2扩展规范说明的Connection 和Statement 池的DataSources 对象。单线程，性能较差，适用于小型系统，代码600KB左右。
- BoneCP：开源的快速的 JDBC 连接池。只有四十几K（运行时需要log4j和Google Collections的支持）。另外个人觉得 BoneCP 有个缺点是，JDBC驱动的加载是在连接池之外的，这样在一些应用服务器的配置上就不够灵活。官方说法BoneCP是一个高效、免费、开源的Java数据库连接池实现库。设计初衷就是为了提高数据库连接池性能，完美集成到一些持久化产品如Hibernate和DataNucleus中。特色：高度可扩展，快速；连接状态切换的回调机制；允许直接访问连接；自动化重置能力；JMX支持；懒加载能力；支持XML和属性文件配置方式；较好的Java代码组织。
- DBCP：Database Connection Pool，一个依赖Jakarta commons-pool对象池机制的数据库连接池，单独使用dbcp需要3个包：`common-dbcp.jar,common-pool.jar,common-collections.jar`，预先将数据库连接放在内存中，应用程序需要建立数据库连接时直接到连接池中申请一个就行，用完再放回。单线程，并发量低，性能不好，适用于小型系统。
- Tomcat Jdbc Pool：Tomcat在7.0以前都是使用common-dbcp做为连接池组件，但是dbcp是单线程，为保证线程安全会锁整个连接池，性能较差，dbcp有超过60个类，也相对复杂。Tomcat从7.0开始引入新增连接池模块叫做Tomcat jdbc pool，基于Tomcat JULI，使用Tomcat日志框架，完全兼容dbcp，通过异步方式获取连接，支持高并发应用环境，超级简单核心文件只有8个，支持JMX，支持XA Connection。
- Druid：阿里出品，淘宝和支付宝专用数据库连接池，但它不仅仅是一个数据库连接池，它还包含一个ProxyDriver，一系列内置的JDBC组件库，一个SQL Parser。支持所有JDBC兼容的数据库。Druid针对Oracle和MySQL特别优化，比如Oracle的PS Cache内存占用优化，MySQL的ping检测优化。
- HikariCP：新一代数据库连接池，性能相当优异，spring boot 2 默认使用的 dbcp 从之前的 tomcat-pool 换成HikariCP 。

## 开发步骤

1. 导入连接池jar
2. 创建连接池对象
3. 设置数据库必须的连接参数
4. 设置可选的连接池管理策略参数
5. 从连接池中获得活动的数据库连接
6. 使用连接范围数据量
7. 使用以后关闭数据库连接，这个关闭不再是真的关闭连接，而是将使用过的连接归还给连接池

```java
public class DBUtils {
    private static String driver;
    private static String url;
    private static String username;
    private static String password;
    private static int initSize;
    private static BasicDataSource ds; //连接池就一个
 
    static {
        ds = new BasicDataSource();
        Properties cfg = new Properties();
        //初始化静态属性
        //利用properties 读取配置文件
        //从配置文件中查找相应参数
        try {            //load天生有异常
            InputStream in = DBUtils.class.getClassLoader()
                    .getResourceAsStream("db.properties");
            cfg.load(in);
 
            driver = cfg.getProperty("jdbc.driver");
            url = cfg.getProperty("jdbc.url");
            username = cfg.getProperty("jdbc.username");
            password = cfg.getProperty("jdbc.pasaword");
            initSize = Integer.parseInt(cfg.getProperty("initSize"));
            in.close();
            //初始化连接池
            ds.setDriverClassName(driver);
            ds.setUrl(url);
            ds.setUsername(username);
            ds.setPassword(password);
            //设置连接池的管理策略参数
            ds.setInitialSize(initSize);
 
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }
 
    public static Connection getConnection() {
        try {
            //getConnection()从连接池获取重用的连接，如果连接池满了，则等待
            //如果连接归还，则获取重用的连接; 连接池的线程阻塞方法
            Connection conn = ds.getConnection();
            return conn;
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
 
    }
 
    public static void close(Connection conn) {
        try {
            //将用过的连接归还到连接池
            if (conn != null) {
                conn.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
 
    public static void rollback(Connection conn) {
        try {
            //将用过的连接归还到连接池
            if (conn != null) {
                conn.rollback();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
 
}
```



# Hibernate

JDBC的另一方面的缺点是不易于大型项目的开发，Hibernate的出现就是为了弥补这一缺陷

Hibernate是一个持久化框架和ORM框架，持久化和ORM是两个有区别的概念，持久化注重对象的存储方法是否随着程序的退出而消亡，ORM关注的是如何在数据库表和内存对象之间建立关联。

Hibernate使用POJO来表示Model，使用XML配置文件来配置对象和表之间的关系，它提供了一系列API来通过对对象的操作而改变数据库中的过程。

Hibernate更强调如何对单条记录进行操作，对于更复杂的操作，它提供了一种新的面向对象的查询语言：HQL。

> 当我们工作在一个面向对象的系统中时，存在一个对象模型和关系数据库不匹配的问题。RDBMSs 用表格的形式存储数据，然而像 Java 或者 C# 这样的面向对象的语言它表示一个对象关联图。这时就需要ORM
>
> ORM 表示 Object-Relational Mapping (ORM)，是一个方便在关系数据库和类似于 Java， C# 等面向对象的编程语言中转换数据的技术。一个 ORM 系统相比于普通的 JDBC 有以下的优点。
>
> - 使用业务代码访问对象而不是数据库中的表
> - 从面向对象逻辑中隐藏 SQL 查询的细节
> - 基于 JDBC 的 'under the hood'
> - 没有必要去处理数据库实现
> - 实体是基于业务的概念而不是数据库的结构
> - 事务管理和键的自动生成
> - 应用程序的快速开发

## 架构

| ![架构](Java操作关系型数据库/hibernate_architecture.jpg) | ![高层次架构](Java操作关系型数据库/hibernate_high_level.jpg) |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| 详细的 Hibernate 应用程序体系结构                        | 非常高水平的 Hibernate 应用程序架构                          |

Hibernate 架构是分层的，作为数据访问层，你不必知道底层 API 。Hibernate 利用数据库以及配置数据来为应用程序提供持续性服务（以及持续性对象）

Hibernate 使用不同的现存 Java API，比如 JDBC，Java 事务 API（JTA），以及 Java 命名和目录界面（JNDI）。JDBC 提供了一个基本的抽象级别的通用关系数据库的功能， Hibernate 支持几乎所有带有 JDBC 驱动的数据库。JNDI 和 JTA 允许 Hibernate 与 J2EE 应用程序服务器相集成。

## 开发步骤

### 准备Hibernate环境

导入Hibernate必须的jar包：导入hibernate-release-5.0.2.Final\lib\required下的jar包

加入数据库驱动的jar包

### 创建核心配置文件

Hibernate 需要事先知道在哪里找到映射信息，这些映射信息定义了 Java 类怎样关联到数据库表。Hibernate 也需要一套相关数据库和其它相关参数的配置设置。

所有这些信息通常是作为一个标准的 Java 属性文件提供的，名叫 `hibernate.properties`。又或者是作为 XML 文件提供的，名叫 `hibernate.cfg.xml`

例子：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
    
        <!-- 配置连接数据库的基本信息 -->
        <property name="connection.username">root</property>
        <property name="connection.password">ds756953242</property>
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql:///hibernate</property>
        
        <!-- 配置 hibernate 的基本信息 -->
        <!-- hibernate 所使用的数据库方言 
        <property name="dialect">org.hibernate.dialect.MySQLMyISAMDialect</property>        
        <property name="dialect">org.hibernate.dialect.MySQLDialect</property>        -->
        <property name="dialect">org.hibernate.dialect.MySQL5InnoDBDialect</property>        
        
        <!-- 执行操作时是否在控制台打印 SQL -->
        <property name="show_sql">true</property>
    
        <!-- 是否对 SQL 进行格式化 -->
        <property name="format_sql">true</property>
    
        <!-- 指定自动生成数据表的策略 -->
        <property name="hbm2ddl.auto">update</property>
        
        <!-- 指定关联的 .hbm.xml 文件 -->
        <mapping resource="helloWorld/customer.hbm.xml"/>
    
    </session-factory>
</hibernate-configuration>
```

### 创建持久化Java类

在 Hibernate 中，其对象或实例将会被存储在数据库表单中的 Java 类被称为持久化类。若该类遵循一些简单的规则或者被大家所熟知的 Plain Old Java Object (POJO) 编程模型。

**持久化类的要求**

1. 提供一个无参的构造器：使Hibernate可以使用`Constructor.newInstance()` 来实例化持久化类
2. 为了使对象能够在 Hibernate 和数据库中容易识别，所有类都需要包含一个 ID。此属性映射到数据库表的主键列。
3. 为类的持久化类字段声明访问方法(`get/set`): Hibernate对JavaBeans 风格的属性实行持久化。
4. 所有将被持久化的属性都应该声明为 private
5. 使用非 final 类: 在运行时生成代理是 Hibernate 的一个重要的功能. 如果持久化类没有实现任何接口, Hibnernate 使用 CGLIB 生成代理. 如果使用的是 final 类, 则无法生成 CGLIB 代理.
6. 重写 eqauls 和 hashCode 方法: 如果需要把持久化类的实例放到 Set 中(当需要进行关联映射时), 则应该重写这两个方法
7. Hibernate 不要求持久化类继承任何父类或实现接口，这可以保证代码不被污染。这就是Hibernate被称为低侵入式设计的原因

例：

```java
public class Employee {
   private int id;
   private String firstName; 
   private String lastName;   
   private int salary;  

   public Employee() {
      firstName=null;
      lastName=null;
      salary =0;
   }
   public Employee(String fname, String lname, int salary) {
      this.firstName = fname;
      this.lastName = lname;
      this.salary = salary;
   }
   public int getId() {
      return id;
   }
   public void setId( int id ) {
      this.id = id;
   }
   public String getFirstName() {
      return firstName;
   }
   public void setFirstName( String first_name ) {
      this.firstName = first_name;
   }
   public String getLastName() {
      return lastName;
   }
   public void setLastName( String last_name ) {
      this.lastName = last_name;
   }
   public int getSalary() {
      return salary;
   }
   public void setSalary( int salary ) {
      this.salary = salary;
   }
}
```



### 创建对象-关系映射文件

Hibernate 采用 XML 格式的文件来指定对象和关系数据之间的映射. 在运行时 Hibernate 将根据这个映射文件来生成各种 SQL 语句。映射文件的扩展名为 `.hbm.xml`。

有很多工具可以用来给先进的 Hibernate 用户生成映射文件。这样的工具包括 **XDoclet**, **Middlegen** 和 **AndroMDA**

下面是一个Mapping映射文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
 "-//Hibernate/Hibernate Mapping DTD//EN"
 "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd"> 

<hibernate-mapping>
   <class name="Employee" table="EMPLOYEE">
      <meta attribute="class-description">
         This class contains the employee detail. 
      </meta>
      <id name="id" type="int" column="id">
         <generator class="native"/>
      </id>
      <property name="firstName" column="first_name" type="string"/>
      <property name="lastName" column="last_name" type="string"/>
      <property name="salary" column="salary" type="int"/>
   </class>
</hibernate-mapping>
```

`<class>`标签定义从一个 Java 类到数据库表的特定映射。Java 的类名使用 **name** 来表示，数据库表明用 **table** 来表示。

`<meta>` 标签是一个可选元素，可以被用来修饰类。

`<id>` 标签将类中独一无二的 ID 属性与数据库表中的主键关联起来。id 元素中的 **name** 对应类的属性，**column** 对应数据库表的列。**type** 保存 Hibernate 映射的类型，这个类型会将从 Java 转换成 SQL 数据类型。

 id 元素中的 `<generator>` 标签用来自动生成主键值。设置 generator 标签中的 **class** 可以设置 **native** 使 Hibernate 可以使用 **identity**, **sequence** 或 **hilo** 算法根据底层数据库的情况来创建主键。

`<property>` 标签用来将 Java 类的属性与数据库表的列匹配。标签中 **name** 引用的是类的属性，**column** 引用的是数据库表的列。**type** 保存 Hibernate [映射的类型](https://www.w3cschool.cn/hibernate/fzum1iem.html)，这个类型会将从 Java 转换成 SQL 数据类型。


### 通过 Hibernate API 编写访问数据库

#### 创建会话Session

Session 用于获取与数据库的物理连接。 Session 对象是轻量级的，并且设计为在每次需要与数据库进行交互时被实例化。持久态对象被保存，并通过 Session 对象检索找回。

该 Session 对象不应该长时间保持开放状态，因为它们通常不能保证线程安全，而应该根据需求被创建和销毁。Session 的主要功能是为映射实体类的实例提供创建，读取和删除操作。

#### 创建事务

一个典型的事务应该使用以下语法：

```java
Session session = factory.openSession();
Transaction tx = null;
try {
   tx = session.beginTransaction();
   // do some work
   ...
   tx.commit();
}
catch (Exception e) {
   if (tx!=null) tx.rollback();
   e.printStackTrace(); 
}finally {
   session.close();
}
```

如果 Session 引发异常，则事务必须被回滚，该 session 必须被丢弃。

#### Session 接口方法

| 序号 | Session 方法及说明                                           |
| ---- | :----------------------------------------------------------- |
| 1    | **Transaction beginTransaction()** 开始工作单位，并返回关联事务对象。 |
| 2    | **void cancelQuery()** 取消当前的查询执行。                  |
| 3    | **void clear()** 完全清除该会话。                            |
| 4    | **Connection close()** 通过释放和清理 JDBC 连接以结束该会话。 |
| 5    | **Criteria createCriteria(Class persistentClass)** 为给定的实体类或实体类的超类创建一个新的 Criteria 实例。 |
| 6    | **Criteria createCriteria(String entityName)** 为给定的实体名称创建一个新的 Criteria 实例。 |
| 7    | **Serializable getIdentifier(Object object)** 返回与给定实体相关联的会话的标识符值。 |
| 8    | **Query createFilter(Object collection, String queryString)** 为给定的集合和过滤字符创建查询的新实例。 |
| 9    | **Query createQuery(String queryString)** 为给定的 HQL 查询字符创建查询的新实例。 |
| 10   | **SQLQuery createSQLQuery(String queryString)** 为给定的 SQL 查询字符串创建 SQLQuery 的新实例。 |
| 11   | **void delete(Object object)** 从数据存储中删除持久化实例。  |
| 12   | **void delete(String entityName, Object object)** 从数据存储中删除持久化实例。 |
| 13   | **Session get(String entityName, Serializable id)** 返回给定命名的且带有给定标识符或 null 的持久化实例（若无该种持久化实例）。 |
| 14   | **SessionFactory getSessionFactory()** 获取创建该会话的 session 工厂。 |
| 15   | **void refresh(Object object)** 从基本数据库中重新读取给定实例的状态。 |
| 16   | **Transaction getTransaction()** 获取与该 session 关联的事务实例。 |
| 17   | **boolean isConnected()** 检查当前 session 是否连接。        |
| 18   | **boolean isDirty()** 该 session 中是否包含必须与数据库同步的变化？ |
| 19   | **boolean isOpen()** 检查该 session 是否仍处于开启状态。     |
| 20   | **Serializable save(Object object)** 先分配一个生成的标识，以保持给定的瞬时状态实例。 |
| 21   | **void saveOrUpdate(Object object)** 保存（对象）或更新（对象）给定的实例。 |
| 22   | **void update(Object object)** 更新带有标识符且是给定的处于脱管状态的实例的持久化实例。 |
| 23   | **void update(String entityName, Object object)** 更新带有标识符且是给定的处于脱管状态的实例的持久化实例。 |

#### HQL语法

Hibernate 查询语言（HQL）是一种面向对象的查询语言，类似于 SQL，但不是去对表和列进行操作，而是面向对象和它们的属性。 HQL 查询被 Hibernate 翻译为传统的 SQL 查询从而对数据库进行操作。

尽管你能直接使用本地 SQL 语句，但我还是建议你尽可能的使用 HQL 语句，以避免数据库关于可移植性的麻烦，并且体现了 Hibernate 的 SQL 生成和缓存策略。

在 HQL 中一些关键字比如 SELECT ，FROM 和 WHERE 等，是不区分大小写的，但是一些属性比如表名和列名是区分大小写的。

#### 例子

```java
import java.util.List; 
import java.util.Date;
import java.util.Iterator; 

import org.hibernate.HibernateException; 
import org.hibernate.Session; 
import org.hibernate.Transaction;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class ManageEmployee {
   private static SessionFactory factory; 
   public static void main(String[] args) {
      try{
         factory = new Configuration().configure().buildSessionFactory();
      }catch (Throwable ex) { 
         System.err.println("Failed to create sessionFactory object." + ex);
         throw new ExceptionInInitializerError(ex); 
      }
      ManageEmployee ME = new ManageEmployee();

      /* Add few employee records in database */
      Integer empID1 = ME.addEmployee("Zara", "Ali", 1000);
      Integer empID2 = ME.addEmployee("Daisy", "Das", 5000);
      Integer empID3 = ME.addEmployee("John", "Paul", 10000);

      /* List down all the employees */
      ME.listEmployees();

      /* Update employee's records */
      ME.updateEmployee(empID1, 5000);

      /* Delete an employee from the database */
      ME.deleteEmployee(empID2);

      /* List down new list of the employees */
      ME.listEmployees();
   }
   /* Method to CREATE an employee in the database */
   public Integer addEmployee(String fname, String lname, int salary){
      Session session = factory.openSession();
      Transaction tx = null;
      Integer employeeID = null;
      try{
         tx = session.beginTransaction();
         Employee employee = new Employee(fname, lname, salary);
         employeeID = (Integer) session.save(employee); 
         tx.commit();
      }catch (HibernateException e) {
         if (tx!=null) tx.rollback();
         e.printStackTrace(); 
      }finally {
         session.close(); 
      }
      return employeeID;
   }
   /* Method to  READ all the employees */
   public void listEmployees( ){
      Session session = factory.openSession();
      Transaction tx = null;
      try{
         tx = session.beginTransaction();
         List employees = session.createQuery("FROM Employee").list(); 
         for (Iterator iterator = 
                           employees.iterator(); iterator.hasNext();){
            Employee employee = (Employee) iterator.next(); 
            System.out.print("First Name: " + employee.getFirstName()); 
            System.out.print("  Last Name: " + employee.getLastName()); 
            System.out.println("  Salary: " + employee.getSalary()); 
         }
         tx.commit();
      }catch (HibernateException e) {
         if (tx!=null) tx.rollback();
         e.printStackTrace(); 
      }finally {
         session.close(); 
      }
   }
   /* Method to UPDATE salary for an employee */
   public void updateEmployee(Integer EmployeeID, int salary ){
      Session session = factory.openSession();
      Transaction tx = null;
      try{
         tx = session.beginTransaction();
         Employee employee = 
                    (Employee)session.get(Employee.class, EmployeeID); 
         employee.setSalary( salary );
         session.update(employee); 
         tx.commit();
      }catch (HibernateException e) {
         if (tx!=null) tx.rollback();
         e.printStackTrace(); 
      }finally {
         session.close(); 
      }
   }
   /* Method to DELETE an employee from the records */
   public void deleteEmployee(Integer EmployeeID){
      Session session = factory.openSession();
      Transaction tx = null;
      try{
         tx = session.beginTransaction();
         Employee employee = 
                   (Employee)session.get(Employee.class, EmployeeID); 
         session.delete(employee); 
         tx.commit();
      }catch (HibernateException e) {
         if (tx!=null) tx.rollback();
         e.printStackTrace(); 
      }finally {
         session.close(); 
      }
   }
}
```



# MyBatis

MyBatis和Hibernate一样是一个持久化框架和ORM框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以对配置和原生Map使用简单的 XML 或注解，将接口和 Java 的 POJOs映射成数据库中的记录

## 架构

1. API接口层：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。
2. 数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。
3. 基础支撑层：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

## 开发步骤

### 准备MyBatis环境

导入Hibernate必须的jar包：导入mybatis-3.2.2.jar包

加入数据库驱动的jar包mysql-connector-java-5.1.10-bin.jar 

### 创建核心配置文件

Mybatis需要事先知道在哪里找到映射信息。Mybatis也需要一套相关数据库和其它相关参数的配置设置。

所有这些信息通常是作为一个标准的 Java 属性文件提供的。又或者是作为 XML 文件提供的

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- 配置开发环境，可以配置多个，在具体用时再做切换 -->
    <environments default="test">
        <environment id="test">
            <transactionManager type="JDBC"></transactionManager>  <!-- 事务管理类型：JDBC、MANAGED -->
            <dataSource type="POOLED">    <!-- 数据源类型：POOLED、UNPOOLED、JNDI -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/test?characterEncoding=utf-8" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>

    <!-- 加载映射文件 mapper -->
    <mappers>
        <!-- 路径用 斜线（/） 分割，而不是用 点(.) -->
        <mapper resource="yeepay/payplus/mapper/UserMapper.xml"></mapper>
    </mappers>
</configuration>
```



### 创建持久化Java类

遵循POJO类要求

```java
package yeepay.payplus;

/**
 * Created by 维C果糖 on 2017/2/2.
 */
public class Person {
    private Integer id;
    private String name;
    private Integer age;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```



### 创建对象-关系映射、定义SQL语句

MyBatis采用 XML 格式的文件来指定对象和关系数据之间的映射. MyBatis3加入了注解形式

#### XML形式

**文件中的一些属性**

设置 `namespace` 命名空间，目的是为了区分映射文件中的方法

结果集 `resultMap` 是 MyBatis 最大的特色，对象的 ORM 就由其来转换

- 在结果集中，包括主键 `id` 和 普通属性 `result`；
- 在结果集中，常用的两个属性分别为：`property`，表示实体的属性；`column`，表示 SQL 查询的结果集的列。

在映射文件中，常用的标签有四个，分别为： `select`、`insert`、`update` 和 `delete`

- 每个标签中都有 id 属性，在同一个 mapper 文件中 id 不允许重复；
- 参数 `parameterMap` 已经被废弃，现在其存在的目的就是为了兼容前期的项目；
- 参数 `parameterType` 支持很多的类型，例如 `int`、`Integer`、`String`、`Double`、`List`、`Map` 或者实体对象等；
- 返回值 `resultType` 用于简单的类型；
- 返回值 `resultMap` 用于复杂的类型；
- 当参数和返回值是集合的时候，其声明的是集合中的元素类型；
- SQL 语句不区分大小写，它默认使用 `PrepareStatement`，预编译，可以防止 SQL 注入。

获取参数的方法为 #{ 字段名 }**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="yeepay.payplus/mapper.UserMapper">   <!-- 命名空间，名字可以随意起，只要不冲突即可 -->
    <!-- 对象映射，可以不写 -->
    <resultMap type="yeepay.payplus.Person" id="personRM">
        <!-- property="id"，表示实体对象的属性；column="ID"，表示结果集字段 -->
        <id property="id" column="ID"/>
        <result property="Name" column="NAME"/>
        <result property="age" column="AGE"/>
    </resultMap>
    
    <!-- 查询功能，resultType 设置返回值类型 -->
    <select id="findAll" resultType="yeepay.payplus.Person">  <!-- 书写 SQL 语句 -->
        SELECT * FROM person
    </select>

    <!-- 通过 ID 查询 -->
    <select id="get" parameterType="Integer" resultMap="personRM">  <!-- 书写 SQL 语句 -->
        SELECT * FROM person
        WHERE id = #{id}
    </select>

    <!-- 新增功能，在SQL语句中有参数，并以实体来封装参数 -->
    <insert id="insert" parameterType="yeepay.payplus.Person">
        INSERT INTO person (id,name,age) VALUES (#{id},#{name},#{age})
    </insert>

    <!-- 修改功能 -->
    <update id="update" parameterType="yeepay.payplus.Person">
        UPDATE person set name=#{name},age=#{age}
        WHERE id = #{id}
    </update>

    <!-- 删除功能 -->
    <delete id="deleteById" parameterType="integer">
        DELETE FROM person
        WHERE id = #{id}
    </delete>
    
    
</mapper>
```

##### 定义动态SQL

动态SQL支持根据不同条件拼接 SQL 语句，可用标签：

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

```xml
<!-- 动态SQL，parameterType 设置参数类型，resultType 设置返回值类型 -->
    <select id="findAll" parameterType="yeepay.payplus.Person" resultType="Person">  
        SELECT id,name,age FROM person
        <where>
            <if test="name =! null">
                name = #{name}
            </if>
            <if test="age =! null">
                and age = #{age}
            </if>
        </where>
    </select>
    
<!-- 批量删除，Map 类型 -->
    <delete id="deleteMap" parameterType="Map">
        DELETE FROM person WHERE id IN
        <foreach collection="ids" item="id" open="(" close=")" separator=",">
            #{id}
        </foreach>
        AND  age = #{age}
    </delete>
<!-- 批量删除，List 类型 -->
    <delete id="deleteList" parameterType="integer">
        DELETE FROM person WHERE id IN
        <foreach collection="list" item="id" open="(" close=")" separator=",">
            #{id}
        </foreach>
    </delete>
```

##### 1:N映射关系定义

```xml
    <!-- 配置关联关系 1:N -->
    <resultMap type="yeepay.payplus.domain.Customer" id="customerOrdersRM" extends="customerRM">
        <!-- 配置多的（N），property 属性就是实体中的 List 对象属性名称，ofType 属性就是集合元素的类型 -->
        <collection property="orders" ofType="yeepay.payplus.domain.Orders">
            <id property="id" column="ID"/>
            <result property="sn" column="SN"/>
            <result property="remark" column="REMARK"/>
        </collection>
    </resultMap>
```

#### 注解形式

因为最初设计时,MyBatis 是一个 XML 驱动的框架。配置信息是基于 XML 的,而且 映射语句也是定义在 XML 中的。而到了 MyBatis 3,有新的可用的选择了。MyBatis 3 构建 在基于全面而且强大的 Java 配置 API 之上。这个配置 API 是基于 XML 的 MyBatis 配置的 基础,也是新的基于注解配置的基础。注解提供了一种简单的方式来实现简单映射语句,而 不会引入大量的开销。

注意 不幸的是,Java 注解限制了它们的表现和灵活。尽管很多时间都花调查,设计和 实验上,最强大的 MyBatis 映射不能用注解来构建,那并不可笑。C#属性(做示例)就没 有这些限制,因此 MyBatis.NET 将会比 XML 有更丰富的选择。也就是说,基于 Java 注解 的配置离不开它的特性。

**注解有下面这些:**

| 注解                                                         | 目标        | 相对应的 XML                                |
| ------------------------------------------------------------ | ----------- | ------------------------------------------- |
| `@CacheNamespace`                                            | `class`     |                                             |
| `@CacheNamespaceRef`                                         | `class`     |                                             |
| `@ConstructorArgs`                                           | `Method`    |                                             |
| `@Arg`                                                       | `Method`    |                                             |
| `@TypeDiscriminator`                                         | `Method`    |                                             |
| `@Case`                                                      | `Method`    |                                             |
| `@Results`                                                   | `Method`    |                                             |
| `@Result`                                                    | `Method`    |                                             |
| `@One`                                                       | `Method`    |                                             |
| `@Many`                                                      | `Method`    |                                             |
| `@MapKey`                                                    | `Method`    |                                             |
| `@Options`                                                   | `Method`    |                                             |
| `@Insert`,`@Update`, `@Delete`, `@Select`                    | `Method`    | `<insert>`,`<update>`,`<delete>`,`<select>` |
| `@InsertProvider,@UpdateProvider,@DeleteProvider,@SelectProvider` | `Method`    | `<insert>`,`<update>`,`<delete>`,`<select>` |
| `@Param`                                                     | `Parameter` |                                             |
| `@SelectKey`                                                 | `Method`    | `<selectKey>`                               |
| `@ResultMap`                                                 | `Method`    |                                             |
| `@ResultType`                                                | `Method`    | N                                           |

##### 映射注解申明sql样例

这些函数名相当于XML中的id

```java
public interface ClusterMessageMapper {

    // Insert
    @Insert("insert into cluster_manager(cluster_name, cluster_time, cluster_address, cluster_access_user, cluster_access_passwd) " +
            "values(#{clusterName}, #{clusterTime}, #{clusterAddress}, #{clusterAccessUser}, #{clusterAccessPasswd})")
    @Options(useGeneratedKeys = true, keyColumn = "cluster_id", keyProperty = "clusterId")
    public void insertClusterMessage(ClusterMessage clusterMessage);

    // select
    @Select("select * from cluster_manager")
    @Results(
            id = "clusterMessage",
            value = {
                    @Result(column = "cluster_name", property = "clusterName", javaType = String.class, jdbcType = JdbcType.VARCHAR),
                    @Result(column = "cluster_time", property = "clusterTime", javaType = Long.class, jdbcType = JdbcType.BIGINT),
                    @Result(column = "cluster_address", property = "clusterAddress", javaType = String.class, jdbcType = JdbcType.VARCHAR),
                    @Result(column = "cluster_access_user", property = "clusterAccessUser", javaType = String.class, jdbcType = JdbcType.VARCHAR),
                    @Result(column = "cluster_access_passwd", property = "clusterAccessPasswd", javaType = String.class, jdbcType = JdbcType.VARCHAR)
            }
    )
    public List<ClusterMessage> getClusterMessage();

    @Select("select * from cluster_manager")
    @MapKey("clusterId")
    public Map<Integer, ClusterMessage> getClusterMessageMapper();


    @Select("select * from cluster_manager where cluster_id=#{clusterId}")
    @ResultMap("clusterMessage")
    public ClusterMessage getClusterMessageById(@Param("clusterId") int clusterId);


    // update
    @Update("update cluster_manager set cluster_name=#{clusterName} where cluster_id=#{clusterId}")
    public void updateClusterMessage(ClusterMessage clusterMessage);

    // delete
    @Delete("delete from cluster_manager where cluster_id=#{clusterId}")
    public void deleteClusterMessage(@Param("clusterId")int clusterId);
}
```

### 调用定义的SQL

```java
// 1、获得 SqlSessionFactory
String resource = "sqlMapConfig.xml";           // 定位核心配置文件
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);  

// 2、获得 SqlSession
SqlSession sqlSession = sqlSessionFactory.openSession();   

// 3、调用在 mapper 文件中配置的 SQL 语句
List<Person> personList = sqlSession.selectList("yeepay.payplus/mapper.UserMapper.findAll");
Person p = sqlSession.selectOne("yeepay.payplus.mapper.UserMapper.get", 2);
sqlSession.insert("yeepay.payplus.mapper.UserMapper.insert", p);
sqlSession.insert("yeepay.payplus.mapper.UserMapper.update", p);
sqlSession.delete("yeepay.payplus.mapper.UserMapper.deleteById", 2);
```

# MyBatis-plus

虽然mybatis可以直接在xml中通过SQL语句操作数据库，很是灵活。但正其操作都要通过SQL语句进行，就必须写大量的xml文件，很是麻烦。mybatis-plus就很好的解决了这个问题。

Mybatis-Plus（简称MP）是一个 Mybatis 的增强工具，在 Mybatis 的基础上只做增强不做改变，为简化开发、提高效率而生。其实就是它已经封装好了一些crud方法，我们不需要再写xml了，直接调用这些方法就行，就类似于JPA



## 架构

![架构](JAVA操作关系型数据库/mybatis-plus-framework.jpg)

![架构细节](JAVA操作关系型数据库/mybatis-plus-framework-detail.jpg)

## 开发步骤

### 环境准备

### 配置

在 `application.yml` 配置文件中添加 H2 数据库的相关配置：

```yaml
# DataSource Config
spring:
  datasource:
    driver-class-name: org.h2.Driver
    schema: classpath:db/schema-h2.sql
    data: classpath:db/data-h2.sql
    url: jdbc:h2:mem:test
    username: root
    password: test
```

在 Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹：

```java
@SpringBootApplication
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(QuickStartApplication.class, args);
    }

}
```

### 创建实体类

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

### 创建Mapper类或Service类

#### Mapper类

创建Mapper类继承BaseMapper类，BaseMapper类是

```java
public interface UserMapper extends BaseMapper<User> {

}
```

> - 通用 CRUD 封装`BaseMapper`接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器
> - 泛型 `T` 为任意实体对象
> - 参数 `Serializable` 为任意类型主键 `Mybatis-Plus` 不推荐使用复合主键约定每一张表都有自己的唯一 `id` 主键
> - 对象 `Wrapper` 为条件构造器

#### Service类

创建Service接口继承IService接口，创建Service类继承ServiceImpl类和Service接口

```java
import com.baomidou.mybatisplus.extension.service.IService;
import com.tp.mybatisplusstudy.vo.User;

public interface IUserService extends IService<User> {
}
```

```java
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.tp.mybatisplusstudy.dao.UserMapper;
import com.tp.mybatisplusstudy.service.IUserService;
import com.tp.mybatisplusstudy.vo.User;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {
}
```


> 说明:
>
> - 通用 Service CRUD 封装`IService`接口，进一步封装 CRUD 采用 `get 查询单行` `remove 删除` `list 查询集合` `page 分页` 前缀命名方式区分 `Mapper` 层避免混淆，
> - 泛型 `T` 为任意实体对象
> - 建议如果存在自定义通用 Service 方法的可能，请创建自己的 `IBaseService` 继承 `Mybatis-Plus` 提供的基类
> - 对象 `Wrapper` 为条件构造器

### 调用函数操纵数据库

| Service CRUD 接口                                            | Mapper CRUD 接口                           |
| ------------------------------------------------------------ | ------------------------------------------ |
| Save<br />SaveOrUpdate<br />Remove<br />Update<br />Get<br />List<br />Page<br />Count<br />Chain | Insert<br />Delete<br />Update<br />Select |

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SampleTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);    // 使用了Mapper接口
        Assert.assertEquals(5, userList.size());
        userList.forEach(System.out::println);
    }

}
```

参考：

https://www.cnblogs.com/dongguangming/p/12742533.html

https://blog.csdn.net/lonelymanontheway/article/details/83339837

https://blog.csdn.net/binqijiang9465/article/details/106993191/