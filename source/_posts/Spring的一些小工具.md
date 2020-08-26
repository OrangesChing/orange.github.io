---
title: Spring的一些小工具
date: 2020-08-24 17:05:23
tags:
  - JAVA工具
categories:
  - Spring
  - Spring实践
---

# Liquibase

Liquibase 是一个用于跟踪，管理和应用数据库变化的开源的数据库重构工具。它将所有数据库的变化都保存在XML文件中，包括对数据库的改变，以及数据库的版本信息，方便数据的升级和回滚等操作。

## Liquibase是如何工作的

Liquibase的核心是依靠一种简单的机制来跟踪、版本和部署更改：
- Liquibase使用更改日志（是更改的分类）按特定顺序显式列出数据库更改。更改日志中的每个更改都是一个`change set`。更改日志可以任意嵌套，以帮助组织和管理数据库迁移。

    注意： 最佳做法是确保每个`change set`都尽可能原子性更改，以避免失败的结果使数据库中剩下的未处理的语句处于unknown 状态;不过，可以将大型 `SQL` 脚本视为单个更改集。

- Liquibase 使用跟踪表（具体称为`DATABASECHANGELOG` ），该表位于每个数据库上，并跟踪已部署更改日志中的`change set`
    注意：如果 Liquibase所在的数据库没有跟踪表，Liquibase将创建一个跟踪表。
    注意：为了协助处理您未从空白数据库开始的项目，Liquibase具有生成一条更改日志以表示数据库模式当前状态的功能。
    
    > 查看数据库可以除了自己创建的表外，还有另外两个表：`databasechangeloglock`和`databasechangeloglock`。`databasechangelog`表包含针对数据库运行的所有更改的列表。`databasechangeloglock`表用于确保两台计算机不会同时尝试修改数据库。


使用分类和跟踪表，Liquibase能够：

- 跟踪和以版本控制数据库更改 – 用户确切知道已部署到数据库的更改以及尚未部署的更改。

- 部署更改 — 具体来说，通过将分类(`ledger`)中的内容与跟踪表中的内容进行比较，  Liquibase只能将以前尚未部署到数据库的更改部署到数据库中。

  注意：Liquibase具有上下文、标签和先决条件等高级功能，可精确控制`changeSet`的部署时间以及位置。


## 使用

使用 `Liquibase`时，可以使用 [Liquibase 函数](https://www.liquibase.org/quickstart.html#simpleSQL)或 [SQL](https://www.liquibase.org/quickstart.html#lbmodel) 定义更改。重要的是，这些模式不是互斥的，可以结合使用

[手册地址](https://docs.liquibase.com/commands/home.html)

### Change Log

这是一次性步骤，用于配置更改日志以指向将包含 `SQL` 脚本的 `sql` 文件夹。在解压的`*.zip` 或`*.tar.gz`的 `Liquibase` 的目录中创建并保存文件名为 `myChangeLog.xml` 的文件 。`myChangeLog.xml` 的内容应如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

  <includeAll path="/sql"/>
</databaseChangeLog>
```

### ChangeSet

每个`changeSet`都由`id`属性和`author`属性进行唯一标识。这两个标记以及更改日志文件的名称和包唯一地标识了更改。如果只需要指定一个`id`，则很容易意外重用它们，尤其是在处理多个开发人员和代码分支时。包括`author`属性可最大程度地减少重复的可能性。

将每个`changeSet`视为要应用于数据库的原子更改。通常最好在`changeSet`中只包含一个更改，但如果插入多个行时，这些行一起有作为单个事务添加的意义，则允许进行更多更改。`Liquibase` 将尝试将每个`changeSet`运行为单个事务，但对于某些命令，许多数据库将静默地提交和回滚事务（创建表、删除表等）。

```xml
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <changeSet id="1" author="bob">
        <createTable tableName="department">
            <column name="id" type="int">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="name" type="varchar(50)">
                <constraints nullable="false"/>
            </column>
            <column name="active" type="boolean" defaultValueBoolean="true"/>
        </createTable>
    </changeSet>

</databaseChangeLog>
```



# Lombok

