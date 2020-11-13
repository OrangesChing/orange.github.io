---
title: MyBatis-plus环境配置
date: 2020-09-01 18:22:43
tags:
  - MyBatis-plus
categories:
  - JavaWeb
  - JAVA与数据库
  - MyBatis-plus
---

# Spring Boot + MyBatis-plus

## 新建MAVEN工程

在https://start.spring.io/中初始化一个Spring Boot工程，或自己新建MAVEN

## 在pom.xml文件中添加依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<!--父级依赖-->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.orange</groupId>
	<artifactId>orange</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>orange</name>
	<description>orange learn java</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- MySql -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>

		<!--小工具可以免写get、set、tostring等方法-->
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		
		<!-- mybatis-plus的依赖包，有这个包不要引mybatis的包以免混乱 -->
		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-boot-starter</artifactId>
			<version>3.4.0</version>
		</dependency>
		<!-- 默认模板引擎和代码生成器依赖 当要使用mybatis-plus的自动代码生成器的时候要这两个依赖 -->
		<dependency>
			<groupId>org.apache.velocity</groupId>
			<artifactId>velocity-engine-core</artifactId>
			<version>2.0</version>
		</dependency>
		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-generator</artifactId>
			<version>3.4.0</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<version>2.2.6.RELEASE</version>
			</plugin>
		</plugins>
	</build>

</project>
```

## 在application.yml中的配置

注意这是application.yml的配置格式，application.properity文件配置不是这个格式

```yml
spring:
  application:
    name: demo
  profiles:
    active: dev
  # 配置数据源
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/orange_learning?useUnicode=true&characterEncoding=UTF8&useSSL=false # MySQL在高版本需要指明是否进行SSL连接 解决则加上 &useSSL=false
    name: orange_learning
    username: root
    password: .a123456

# mybatis-plus相关配置
mybatis-plus:
  # xml扫描，多个目录用逗号或者分号分隔（告诉 Mapper 所对应的 XML 文件位置）
  mapper-locations: classpath:**/*Mapper.xml
  # 以下配置均有默认值,可以不设置
  global-config:
    #主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
    id-type: 0
    #字段策略 0:"忽略判断",1:"非 NULL 判断"),2:"非空判断"
    field-strategy: 2
    #驼峰下划线转换
    db-column-underline: true
    #刷新mapper 调试神器
    refresh-mapper: false
  configuration:
    # 是否开启自动驼峰命名规则映射:从数据库列名到Java属性驼峰命名的类似映射
    map-underscore-to-camel-case: true
    cache-enabled: false
    jdbc-type-for-null: 'null'
```

## 使用自动代码生成器生成mapper、dao、service、domain（也可以自己建）

与启动类同级的地方创建自动代码生成器类AutoGenerater，自动代码生成器会根据MySQL中的数据表自动创建相应的mapper、dao、service、domain

```java
public class AutoGenerater {

    public static void main(String[] args) {
        AutoGenerator mpg = new AutoGenerator();
        // =============================全局配置===============================
        mpg.setGlobalConfig(new GlobalConfig()
                // 覆盖输出到xxx目录
                .setFileOverride(true).setOutputDir("D:/TestProject/orange/src/main/java/")
                // 主键生成策略 生成BaseResultMap
                .setIdType(IdType.AUTO).setBaseResultMap(true)
                // 指定作者
                .setAuthor("orange")
                // 设置Controller、Service、ServiceImpl、Dao、Mapper文件名称，%s是依据表名转换来的
                .setControllerName("%sController")
                .setServiceName("MP%sService")
                .setServiceImplName("%sServiceImpl")
                .setMapperName("%sDao")
                .setXmlName("%sMapper"));
        // ================================数据源配置============================
        mpg.setDataSource(new DataSourceConfig()
                // 用户名、密码、驱动、url
                .setUsername("root")
                .setPassword(".a123456")
                .setDbType(DbType.MYSQL)
                .setDriverName("com.mysql.cj.jdbc.Driver")
                .setUrl("jdbc:mysql://localhost:3306/orange_learning?useUnicode=true&characterEncoding=utf-8&useSSL=false"));
        // ==================包名配置：父包.模块.controller==========================
        mpg.setPackageInfo(new PackageConfig()
                // 父包名 模块名
                .setParent("com.orange")
                .setModuleName("learn")
                // 分层包名
                .setController("controller")
                .setService("service")
                .setServiceImpl("service.impl")
                .setEntity("domain")
                .setMapper("mapper"));
        // =====================================策略配置==================================
        mpg.setStrategy(new StrategyConfig()
                // 命名策略：实体的类名和属性名按下划线转驼峰 user_info -> userInfo
                .setNaming(NamingStrategy.underline_to_camel)
                // controller生成@RestCcontroller
                .setRestControllerStyle(true));
        // 执行生成
        mpg.execute();
    }
}
```

> 创建完需注意Service类是否有@Service注释
>
> Mapper类在启动类中的MapperScan中已经配置了，所以不需要任何Bean相关的注释

## 最终项目结构

![springboot目录结构](MyBatis-plus环境配置/springboot目录结构.png)


## 在启动类添加MapperScan

```java
@SpringBootApplication
@MapperScan("com.orange.learn.mapper")   // 添加Mapper扫描包
public class OrangeApplication {

   public static void main(String[] args) {
      SpringApplication.run(OrangeApplication.class, args);
   }

}
```

## 添加测试文件测试环境是否正常

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MapperCRUDTest {

    @Autowired
    StudentDao studentDao;

    /**
     * 新增数据
     */
    @Test
    public void testAdd() throws Exception{
        List<Student> search = studentDao.selectByMap(new HashMap<String, Object>(){{put("name","橙子");}});
        if(search.size()==0){
            Student stu = new Student();
            stu.setName("橙子");
            stu.setAge(12);
            stu.setTel("123456");
            int result = studentDao.insert(stu);
            System.out.println(result);
        } else{
            System.out.println("已存在该数据");
        }
    }
}
```