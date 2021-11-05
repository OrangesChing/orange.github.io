---
title: MyBatis-plus踩坑记录
date: 2021-01-04 18:02:45
tags:
  - ORM框架
  - 开发经验
categories:
  - Java
  - Java与数据库
  - MyBatis-plus

---

# 映射为NULL的问题

1. 数据库为下划线命名例如：`enable_type`，当`MyBatis-plus`设置了`map-underscore-to-camel-case: true`时，JAVA对象应命名为`enableType`，否则会映射不到
2. 当`MyBatis-plus`设置了`type-enums-package: `时，会自动映射到枚举类，当枚举类的值为`Integer`，但数据库该字段的类型为`Tinyint(1)`时，会映射失败。因为java在获取`Tinyint(1)`类型的表对象时会转化为`Boolean`。可将该字段类型改为`Tinyint(4)`，或将枚举类型的值改为`Boolean`