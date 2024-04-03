---
title: Spring DI源码阅读
date: 2022-11-15 08:59:43
tags:
  - 源码
categories:
  - spring框架
  - Spring源码分析
---

# 转换BeanName

若提供的是别名，则转化为实际的beanName

# 尝试从单例缓存中获取Bean



# Bean的实例化



## 若从缓存中获取到Bean

直接调用getObjectForBeanInstance(sharedInstance, name, beanName, null)实例化

## 没有从缓存中获取到Bean

### 原型模式依赖检查

### 检测parentBeanFactory

若Bean定义不在当前的Factory时，使用父BeanFactory获取Bean

### 将IOC容器中的GernericBeanDefinition转换为RootBeanDefinition

如果指定 的BeanName是子Bean的话同时会合并父类的相关属性

### 寻找依赖



### 对不同的scope进行bean实例化



# 类型转换



```
update tp_archive_node set job_process_key = "autoProcess2" where job_process_key = "autoProcess";
```



