---
title: JUNIT测试框架
date: 2020-08-12 22:18:48
categories:
  - 测试
  - 测试技术
  - 单元测试
---

# JUNIT的使用

## 依赖添加

```xml
<dependency>         
	<groupId>junit</groupId>         
	<artifactId>junit</artifactId>         
	<version>4.10</version>         
	<scope>test</scope>     
</dependency>
```

## @Test

@Test(expected = Exception.class) 预期会抛出Exception.class异常

@Test(timeout=100)预期方法执行不会超过100毫秒，控制死循环



## @Runwith 

@RunWith(Parameterized.class) 使用参数化测试方式运行

@RunWith(SpringRunner.class)   使用Spring测试方式运行



## @Parameters

@Parameterized.Parameters 申明当前方法为测试数据来源



## 完整代码

### 被测试类

```java
public class HelloJunit {
    private final int MAX_FACT = 50;
    public long fact(long n) {
        if (n<0){
            throw new RuntimeException("复数不能阶乘");
        }
        if (n>MAX_FACT) {
            throw new RuntimeException("暂不支持50以上的阶乘");
        }
        int r = 1;
        for (int i = 1; i <= n; i++) {
            r = r * i;
        }
        return r;
    }
}
```

### 默认测试方式

```java
import org.junit.Before;
import org.junit.Test;
import static org.junit.Assert.*;
 
public class HelloJunitTest {
    private HelloJunit j;

    // 实例化被测试类
    @Before
    public void setUp() throws Exception {
        j = new HelloJunit();
    }

    // 正常功能测试
    @Test
    public void fact() {
        long n = 1L;
        long result = j.fact(n);
        assertEquals(expResult, result);
    }

    // 异常的测试
    @Test(expected = RuntimeException.class)
    public void factExceptionTest() {
            long n = -1L;
            long result = j.fact(n);
    }
}
```


### 参数化测试方式

```java
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;

import java.util.Arrays;
import java.util.Collection;
import java.util.Collections;

import static org.junit.Assert.*;

@RunWith(Parameterized.class)   
public class HelloJunitTest {
    private HelloJunit j;
    private long n;
    private long expResult;

    // 使用参数化测试方式时，必须声明构造方法
    public HelloJunitTest(long n, long expResult) {
        this.n = n;
        this.expResult = expResult;
    }

    @Before
    public void setUp() throws Exception {
        j = new HelloJunit();
    }

    // 创建测试参数，创建的测试参数将通过类属性n和expResult传入测试方法
    @Parameterized.Parameters   
    public static Collection data(){
        return Arrays.asList(new Object[][]{{-1L,1L},{-3L,6L},{-5L,120L}});
    }

    // 正常功能测试
    @Test
    public void fact() {
        long result = j.fact(n);
        assertEquals(expResult, result);
    }

    // 异常的测试
    @Test(expected = RuntimeException.class)
    public void factExceptionTest() {
        long result = j.fact(n);
    }
}
```

### 在spring框架下的测试方式

导入相关依赖

```xml
<dependency>         
    <groupId>org.springframework</groupId>         
    <artifactId>spring-test</artifactId>         
    <version>3.2.9.RELEASE</version>         
    <scope>test</scope>     
</dependency>
```

测试类

```java
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;

import java.util.Arrays;
import java.util.Collection;
import java.util.Collections;

import static org.junit.Assert.*;

// 使用spring框架的Test，会自动搜索相关依赖并实例。支持test环境下的配置文件，放在test中的resources中自动使用
@RunWith(SpringRunner.class)   
@SpringBootTest
public class HelloJunitTest {

    @Autowired  // 使用注入方式实例化
    private HelloJunit j;
    private long n;
    private long expResult;

    public HelloJunitTest(long n, long expResult) {
        this.n = n;
        this.expResult = expResult;
    }

    @Parameterized.Parameters
    public static Collection data(){
        return Arrays.asList(new Object[][]{{-1L,1L},{-3L,6L},{-5L,120L}});
    }

    // 正常功能测试
    @Test
    public void fact() {
        long result = j.fact(n);
        assertEquals(expResult, result);
    }

    // 异常的测试
    @Test(expected = RuntimeException.class)
    public void factExceptionTest() {
        long result = j.fact(n);
    }
}
```


# 常用注解

| 注释名       | 放置位置               | 功能                                                         |
| ------------ | ---------------------- | ------------------------------------------------------------ |
| @Test        | 方法                   | 标记测试方法，在这里可以测试期望异常和超时时间               |
| @Ignore      | 方法                   | 标记忽略的测试方法                                           |
| @BeforeClass | public static void方法 | 运行测试类中的所有用例前都会运行该方法，可在该方法中做一些前置操作，如创建数据库连接，初始化数据等 |
| @AfterClass  | public static void方法 | 和@BeforeClass注解对应，在运行完测试类中的所有测试用例后执行，可在该方法中做一些后续工作，如回复数据库，断开数据库连接等 |
| @Before      | public void方法        | 会在每个用例运行之前都运行一次该方法。主要用于一些独立与用例之间的准备工作 |
| @After       | public void方法        | 与@Before注解对应，会在每个用例运行之后运行一次              |
| @Runwith     | 类                     | 用来确定这个类如何运行，也可以不标注，会使用默认运行器       |
| @Parameters  | 类                     | 用于参数化测试                                               |

一个JUNIT的单元测试用例执行顺序

@BeforeClass -> @Before -> @Test -> @After -> @AfterClass

每个测试方法的调用顺序

@Before -> @Test -> @After

# 常用断言

| 断言                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| void assertEquals\(\[String message\], expected value, actual value\) | 断言两个值相等。值可能是类型有 int, short, long, byte, char or java\.lang\.Object\. 第一个参数是一个可选的字符串消息 |
| void assertTrue\(\[String message\], boolean condition\)     | 断言一个条件为真                                             |
| void assertFalse\(\[String message\],boolean condition\)     | 断言一个条件为假                                             |
| void assertNotNull\(\[String message\], java\.lang\.Object object\) | 断言一个对象不为空\(null\)                                   |
| void assertNull\(\[String message\], java\.lang\.Object object\) | 断言一个对象为空\(null\)                                     |
| void assertSame\(\[String message\], java\.lang\.Object expected, java\.lang\.Object actual\) | 断言，两个对象引用相同的对象，类似==判断                     |
| void assertNotSame\(\[String message\], java\.lang\.Object unexpected, java\.lang\.Object actual\) | 断言，两个对象不是引用同一个对象，类似!=判断                 |
| void assertArrayEquals\(\[String message\], expectedArray, resultArray\) | 断言预期数组和结果数组相等。数组的类型可能是 int, long, short, char, byte or java\.lang\.Object\. |
| void assertThat(actual, matcher)                             | 查看实际值是否满足指定条件                                   |

