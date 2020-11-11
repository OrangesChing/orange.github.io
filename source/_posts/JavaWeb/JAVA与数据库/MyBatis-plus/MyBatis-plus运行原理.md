---
title: MyBatis-plus运行原理
date: 2020-09-01 19:46:33
tags:
  - MyBatis-plus
categories:
  - JavaWeb
  - JAVA与数据库
  - MyBatis-plus
---



转载自https://blog.csdn.net/lizhiqiang1217/article/details/89738382

1、 employeeMapper 的本质 org.apache.ibatis.binding.MapperProxy
2、 MapperProxy 中 sqlSession –>SqlSessionFactory
![在这里插入图片描述](.\MyBatis-plus运行原理\sqlSessionFactory.png)
3、SqlSessionFacotry 中 → Configuration→ MappedStatements 。每一个 mappedStatement 都表示 Mapper 接口中的一个方法与 Mapper 映射文件中的一个 SQL。 MyBatisPlus在启动就会挨个分析 xxxMapper 中的方法，并且将对应的 SQL 语句处理好，保存到 configuration 对象中的mappedStatements 中
![在这里插入图片描述](.\MyBatis-plus运行原理\Configuration.png)
![在这里插入图片描述](.\MyBatis-plus运行原理\Configuration2.png)

mappedStatements 中
![在这里插入图片描述](.\MyBatis-plus运行原理\mappedStatements.png)

4、本质:
![在这里插入图片描述](.\MyBatis-plus运行原理\本质.png)

- Configuration： MyBatis 或者 MyBatisPlus全局配置对象。
- MappedStatement：一个 MappedStatement 对象对应 Mapper 配置文件中的一个。 select/update/insert/delete 节点，主要描述的是一条 SQL 语句。
- SqlMethod : 枚举对象 ，MyBatisPlus支持的 SQL 方法。
- TableInfo：数据库表反射信息 ，可以获取到数据库表相关的信息。
- SqlSource: SQL 语句处理对象。
- MapperBuilderAssistant： 用于缓存、SQL 参数、查询方剂结果集处理等。通过 MapperBuilderAssistant 将每一个 mappedStatement 添加到configuration 中的 mappedstatements 中。

