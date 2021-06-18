---
title: Mock技术概述
date: 2020-08-12 22:18:48
tags:
  - 概述
categories:
  - 测试
  - 测试技术
  - Mock技术
---

# Mock技术概述

- mockito官网：http://site.mockito.org/#how
- mockito中文文档：http://blog.csdn.net/bboyfeiyu/article/details/52127551#2

在测试过程中，对于某些不容易构造或者不容易获取的对象，可使用mock来虚拟一个对象来创建以便测试的测试方法。

 

常见mock框架

EasyMock、Mockito、PowerMock、Jmockit

 

常见的mock场景：

对象 模拟一些再应用中不容易构造或比较复杂的对象

接口 调用别的接口中的方法必须mock，达到隔离的效果

静态方法 工具类中的静态方法建议mock，单独测试

Dao层方法 Dao层操作数据库的方法必须Mock，隔离数据库

私有方法 建议通过共有方法直接覆盖私有方法的方法来测试，也可以将司有关方法mock，之后用反射的方法单独测试





 