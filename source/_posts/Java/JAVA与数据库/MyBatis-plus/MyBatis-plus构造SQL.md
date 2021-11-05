---
title: MyBatis-plus构造SQL
date: 2020-08-31 14:39:15
tags:
  - ORM框架
categories:
  - Java
  - Java与数据库
  - MyBatis-plus
---

# CRUD接口

## Mapper CRUD 接口


| 增               | 删                                                           | 改                          | 查                                                           |
| ---------------- | ------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| `insert`         | `delete` <br />`deleteBatchIds`<br />`deleteById` <br />`deleteByMap` | `update `<br />`updateById` | `selectById`<br />`selectOne`<br />`selectBatchIds`<br />`selectList`<br />`selectByMap`<br />`selectMaps`<br />`selectObjs`<br />`selectPage`<br />`selectMapsPage`<br />`selectCount` |
| 返回插入数据条数 | 返回删除数据条数                                             | 返回修改数据条数            | 返回查询数据                                                 |

> 通用 CRUD 封装`BaseMapper`接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器

### 增

```java
// 插入一条记录
int insert(T entity);
```

insert方法默认底层会增加uuid随机生成的id字段

所以使用此方法 必须在数据库id字段设置成bigint类型 （不然放不下这么长的数字）。在pojo中id属性不能为int和Integer 了 ， 可以用Long

id不仅是可以自动生成UUID 还可以自增 或者其他

### 删

```java
// 全部返回删除数据条数

// 根据 entity 条件，删除记录
int delete(Wrapper<T> wrapper);

// 删除（根据ID 批量删除）
int deleteBatchIds(Collection<? extends Serializable> idList);

// 根据 ID 删除
int deleteById(Serializable id);

// 根据 columnMap 条件，删除记录
int deleteByMap(Map<String, Object> columnMap);
```

### 改

```java
// 根据 whereEntity 条件，更新记录，没有set的属性将不会被修改
int update(T entity, Wrapper<T> updateWrapper);

// 根据 ID 修改，如果没有id就不会修改，没有set的属性将不会被修改
int updateById(T entity);
```

### 查

```java
// 所有不存在的查询会返回空对象，不是NULL

// 根据 ID 查询
T selectById(Serializable id);

// 根据 entity 条件，查询一条记录，如果是多个会抛出异常，随机取一条加上限制条件 wrapper.last("LIMIT 1")
T selectOne(Wrapper<T> queryWrapper); 
// 多于一条记录返回会报错`TooManyResultsException`,`selectOne`内部调用的是selectList

// 查询（根据ID 批量查询）
List<T> selectBatchIds(Collection idList); 

// 根据Wrapper条件，查询全部记录 
List<T> selectList(Wrapper<T> queryWrapper);

// 查询（根据 columnMap 条件）
List<T> selectByMap(Map<String, Object> columnMap);

// 根据 Wrapper 条件，查询全部记录，返回的是有值的字段组成的HashMap数组
List<Map<String, Object>> selectMaps(Wrapper<T> queryWrapper);

// 根据 Wrapper 条件，查询全部记录，只返回第一个字段的值(一般是ID)
List<Object> selectObjs(Wrapper<T> queryWrapper);

// 根据 entity 条件，查询全部记录（并翻页）
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据 Wrapper 条件，查询全部记录（并翻页），返回的是有值的字段组成的HashMap数组
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, Wrapper<T> queryWrapper);

// 根据 Wrapper 条件，查询总记录数
Integer selectCount( Wrapper<T> queryWrapper);
```

## Service CRUD 接口

| Save                    | SaveOrUpdate                            | Remove                                                       | Update                                            | Get                                                 | List                                                         | Page                   | Count   |
| ----------------------- | --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ | ---------------------- | ------- |
| 新增                    | 新增或修改                              | 删除                                                         | 修改                                              | 获取一条                                            | 获取全部                                                     | 分页获取               | 计数    |
| `save`<br />`saveBatch` | `saveOrUpdate`<br />`saveOrUpdateBatch` | `remove`<br />`removeById`<br />`removeByMap`<br />`removeByIds` | `update`<br />`updateById`<br />`updateBatchById` | `getById`<br />`getOne`<br />`getMap`<br />`getObj` | `list`<br />`listByIds`<br />`listByMap`<br />`listMaps`<br />`listObjs` | `page`<br />`pageMaps` | `count` |

> - 通用 Service CRUD 封装`IService`接口，进一步封装 CRUD 采用 `get 查询单行` `remove 删除` `list 查询集合` `page 分页` 前缀命名方式区分 `Mapper` 层避免混淆，
> - 建议如果存在自定义通用 Service 方法的可能，请创建自己的 `IBaseService` 继承 `Mybatis-Plus` 提供的基类

### Save

```java
// entity没有set的属性不会被修改，entitiy即使set了id属性也不会使用这个值，所以如果数据库只有id不可重复的限制时，可重复save同一entity对象，这与saveOrUpdate不一样
// 次类方法插入完成会set entity中的id属性为数据表中的值

// 插入一条记录（选择字段，策略插入）
boolean save(T entity);
// 插入（批量） 调用saveBatch(entityList, 1000)
boolean saveBatch(Collection<T> entityList);
// 插入（批量）
boolean saveBatch(Collection<T> entityList, int batchSize);
```

### SaveOrUpdate

```java
// 次类方法插入完成会set entity中的id属性为数据表中的值
// saveOrUpdate依据id执行插入和修改操作，所以不能重复插入同一entity对象（除了第一次插入其他操作都会认为时修改），当id不存在时也会执行插入操作
// 谨慎用该类方法
// 没有任何数据修改也会返回true

// TableId 注解存在更新记录，否插入一条记录
boolean saveOrUpdate(T entity);
// 根据updateWrapper尝试更新，否继续执行saveOrUpdate(T)方法
boolean saveOrUpdate(T entity, Wrapper<T> updateWrapper);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
```

### Remove

```java
// 没有删除记录时返回false，只要删除了一条记录就返回true

// 根据 entity 条件，删除记录
boolean remove(Wrapper<T> queryWrapper);
// 根据 ID 删除
boolean removeById(Serializable id);
// 根据 columnMap 条件，删除记录
boolean removeByMap(Map<String, Object> columnMap);
// 删除（根据ID 批量删除）,只要删除了list中的一条记录就会返回true
boolean removeByIds(Collection<? extends Serializable> idList);
```

### Update

```java
// 没有更新记录则返回false，更新成功返回true

// 根据 UpdateWrapper 条件，更新记录 需要设置sqlset，没有设置setsql将报错BadSqlGrammarException
boolean update(Wrapper<T> updateWrapper);
// 根据 whereEntity 条件，更新记录
boolean update(T entity, Wrapper<T> updateWrapper);
// 根据 ID 选择修改 没有设置id将不更新
boolean updateById(T entity);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList, int batchSize);
```

### Get

```java
// 根据 ID 查询
T getById(Serializable id);
// 根据 Wrapper，查询一条记录。结果集，如果是多个会抛出异常，随机取一条加上限制条件 wrapper.last("LIMIT 1")
T getOne(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录， throwEx为有多个 result 是否抛出异常
T getOne(Wrapper<T> queryWrapper, boolean throwEx);
// 根据 Wrapper，查询一条记录，返回有值属性组成的map
Map<String, Object> getMap(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录的第一个字段（一般是id），mapper为映射函数
<V> V getObj(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```

### List

```java
// 查询所有
List<T> list();
// 查询列表
List<T> list(Wrapper<T> queryWrapper);
// 查询（根据ID 批量查询）
Collection<T> listByIds(Collection<? extends Serializable> idList);

// 以下返回有值属性组成的map
// 查询（根据 columnMap 条件）
Collection<T> listByMap(Map<String, Object> columnMap);
// 查询所有列表
List<Map<String, Object>> listMaps();
// 查询列表
List<Map<String, Object>> listMaps(Wrapper<T> queryWrapper);

// 以下返回记录的第一个字段（一般是ID）
// 查询全部记录
List<Object> listObjs();
// 查询全部记录
<V> List<V> listObjs(Function<? super Object, V> mapper);
// 根据 Wrapper 条件，查询全部记录
List<Object> listObjs(Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录
<V> List<V> listObjs(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```

### Page

```java
// 无条件分页查询
IPage<T> page(IPage<T> page);
// 条件分页查询
IPage<T> page(IPage<T> page, Wrapper<T> queryWrapper);

// 以下page中的record格式是Map<String, Object>，由有值字段组成
// 无条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page);
// 条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page, Wrapper<T> queryWrapper);
```

> 这个分页其实并不是物理分页，而是内存分页。
> 也就是说，查询的时候并没有limit语句。等配置了分页插件后才可以实现真正的分页。

### Count

```java
// 查询总记录数
int count();
// 根据 Wrapper 条件，查询总记录数
int count(Wrapper<T> queryWrapper);
```

### Chain

query

```java
// 链式查询 普通
QueryChainWrapper<T> query();
// 链式查询 lambda 式。注意：不支持 Kotlin
LambdaQueryChainWrapper<T> lambdaQuery(); 

// 示例：
query().eq("column", value).one();   // 如果eq出来有多条记录会报错
lambdaQuery().eq(Entity::getId, value).list();
```

update

```java
// 链式更改 普通
UpdateChainWrapper<T> update();
// 链式更改 lambda 式。注意：不支持 Kotlin 
LambdaUpdateChainWrapper<T> lambdaUpdate();

// 示例：
update().eq("column", value).remove();
lambdaUpdate().eq(Entity::getId, value).update(entity);
```

# 条件构造器

**条件构造器是用于生成sql的where条件的**

各种不同构造器的继承关系如下：

![img](MyBatis-plus构造SQL/mybatis-wrapper.png)

## AbstractWrapper

> QueryWrapper(LambdaQueryWrapper) 和 UpdateWrapper(LambdaUpdateWrapper) 的父类
> 用于生成 sql 的 where 条件, entity 属性也用于生成 sql 的 where 条件

- 以下出现的第一个入参`boolean condition`表示该条件是否加入最后生成的sql中
- 以下代码块内的多个方法均为从上往下补全个别`boolean`类型的入参,默认为`true`
- 以下出现的泛型`Param`均为`Wrapper`的子类实例(均具有`AbstractWrapper`的所有方法)
- 以下方法在入参中出现的`R`在普通`wrapper`中是`String`,在`LambdaWrapper`中是函数
- 以下方法入参中的`R column`均表示数据库字段,当`R`具体类型为`String`时则为数据库字段名(字段名是数据库关键字)而不是实体类数据字段名
- 使用中如果入参的`Map`或者`List`为空，则不会加入最后生成的sql中
- `val`不区分类型，最后全部转为`string`

### 比较符

| 操作  | sql                  | 语句                                                         |
| ----- | -------------------- | ------------------------------------------------------------ |
| allEq | 全部eq(或个别isNull) | `allEq(Map<R, V> params)` <br />`allEq(Map<R, V> params, boolean null2IsNull)` <br />`allEq(boolean condition, Map<R, V> params, boolean null2IsNull)` |
| eq    | 等于 =               | `eq(R column, Object val)`<br />`eq(boolean condition, R column, Object val)` |
| ne    | 不等于 <>            | `ne(R column, Object val)`<br />`ne(boolean condition, R column, Object val)` |
| gt    | 大于 >               | `gt(R column, Object val)`<br />`gt(boolean condition, R column, Object val)` |
| ge    | 大于等于 \>=         | `ge(R column, Object val)`<br />`ge(boolean condition, R column, Object val)` |
| lt    | 小于 <               | `lt(R column, Object val)`<br />`lt(boolean condition, R column, Object val)` |
| le    | 小于等于 \>=         | `le(R column, Object val)`<br />`le(boolean condition, R column, Object val)` |



### OR / AND

| 操作      | sql         | 语句                                                         |
| --------- | ----------- | ------------------------------------------------------------ |
| or          | 拼接 OR | `or()`<br />`or(boolean condition)`<br />`or(Consumer<Param> consumer)`<br />`or(boolean condition, Consumer<Param> consumer)`<br />例：`or(i -> i.eq("name", "李白").ne("status", "活着"))`---><br />`or (name = '李白' and status <> '活着')` |
| and         | AND 嵌套 | `and(Consumer<Param> consumer)`<br />`and(boolean condition, Consumer<Param> consumer)`<br />例：`and(i -> i.eq("name", "李白").ne("status", "活着"))`---><br />`and (name = '李白' and status <> '活着')` |
| nested      | 正常嵌套 不带 AND 或者 OR | `nested(Consumer<Param> consumer)`<br />`nested(boolean condition, Consumer<Param> consumer)`<br />例: `nested(i -> i.eq("name", "李白").ne("status", "活着"))`---><br />`(name = '李白' and status <> '活着')` |


### BETWEEN

| 操作       | sql         | 语句                                                         |
| ---------- | ----------- | ------------------------------------------------------------ |
| between    | BETWEEN     | `between(R column, Object val1, Object val2)`<br />`between(boolean condition, R column, Object val1, Object val2)` |
| notBetween | NOT BETWEEN | `notBetween(R column, Object val1, Object val2)`<br />`notBetween(boolean condition, R column, Object val1, Object val2)` |

### LIKE

| 操作      | sql             | 语句                                                         |
| --------- | --------------- | ------------------------------------------------------------ |
| like      | LIKE '%值%'     | `like(R column, Object val)`<br />`like(boolean condition, R column, Object val)` |
| notLike   | NOT LIKE '%值%' | `notLike(R column, Object val)`<br />`notLike(boolean condition, R column, Object val)` |
| likeLeft  | LIKE '%值'      | `likeLeft(R column, Object val)`<br />`likeLeft(boolean condition, R column, Object val)` |
| likeRight | LIKE '值%'      | `likeRight(R column, Object val)`<br />`likeRight(boolean condition, R column, Object val)` |

### NULL

| 操作      | sql         | 语句                                                         |
| --------- | ----------- | ------------------------------------------------------------ |
| isNull    | IS NULL     | `isNull(R column)`<br />`isNull(boolean condition, R column)` |
| isNotNull | IS NOT NULL | `isNotNull(R column)`<br />`isNotNull(boolean condition, R column)` |

### IN

| 操作        | sql                                                          | 语句                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| in          | `IN (value.get(0), value.get(1), ...)`<br />`IN (v0, v1, ...)` | `in(R column, Collection<?> value) `<br />`in(boolean condition, R column, Collection<?> value)`<br />`in(R column, Object... values)`<br />`in(boolean condition, R column, Object... values)` |
| notIn       | `NOT IN (value.get(0), value.get(1), ...)`<br />`NOT IN (v0, v1, ...)` | `notIn(R column, Collection<?> value)`<br />`notIn(boolean condition, R column, Collection<?> value)`<br />`notIn(R column, Object... values)`<br />`notIn(boolean condition, R column, Object... values)` |
| inSql       | IN ( sql语句 )<br />可嵌套select语句                         | inSql(R column, String inValue) inSql(boolean condition, R column, String inValue) |
| notInSql    | NOT IN ( sql语句 )<br />可嵌套select语句                     | notInSql(R column, String inValue) notInSql(boolean condition, R column, String inValue) |

### GROUPBY

| 操作      | sql         | 语句                                                         |
| --------- | ----------- | ------------------------------------------------------------ |
| groupBy     | GROUP BY 字段, ... | `groupBy(R... columns)`<br />`groupBy(boolean condition, R... columns)` |

### ORDERBY

| 操作      | sql         | 语句                                                         |
| --------- | ----------- | ------------------------------------------------------------ |
| orderByAsc  | ORDER BY 字段, ... ASC | `orderByAsc(R... columns)`<br />`orderByAsc(boolean condition, R... columns)` |
| orderByDesc | ORDER BY 字段, ... DESC | `orderByDesc(R... columns)`<br />`orderByDesc(boolean condition, R... columns)` |
| orderBy     | ORDER BY 字段, ... | `orderBy(boolean condition, boolean isAsc, R... columns)` |

### HAVING

| 操作      | sql         | 语句                                                         |
| --------- | ----------- | ------------------------------------------------------------ |
| having      | HAVING ( sql语句 ) | `having(String sqlHaving, Object... params)`<br />`having(boolean condition, String sqlHaving, Object... params)` |


### EXISTS

| 操作      | sql         | 语句                                                         |
| --------- | ----------- | ------------------------------------------------------------ |
| exists      | EXISTS ( sql语句 ) | `exists(String existsSql)`<br />`exists(boolean condition, String existsSql)` |
| notExists   | NOT EXISTS ( sql语句 ) | `notExists(String notExistsSql)`<br />`notExists(boolean condition, String notExistsSql)` |



### func

func 方法(主要方便在出现if...else下调用不同方法能不断链)

```java
func(Consumer<Children> consumer)
func(boolean condition, Consumer<Children> consumer)
    
// 例
func(i -> if(true) {i.eq("id", 1)} else {i.ne("id", 1)}) 
```

### apply 

拼接 sql

```java
apply(String applySql, Object... params)
apply(boolean condition, String applySql, Object... params)
// 例: 
apply("id = 1")  // id = 1
apply("date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")// date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`
apply("date_format(dateColumn,'%Y-%m-%d') = {0}", "2008-08-08")// date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`
```

> 注意:
>
> 该方法可用于数据库函数动态入参的params对应前面applySql内部的{index}部分.
> 这样是不会有sql注入风险的,反之会有!

### last

无视优化规则直接拼接到 sql 的最后

```java
last(String lastSql)
last(boolean condition, String lastSql)
    
// 例子
last("limit 1")
```

> 注意: 
>
> 只能调用一次，多次调用以最后一次为准，**有sql注入的风险**

## QueryWrapper

在`QueryWrapper`中通过`new QueryWrapper().lambda()`获取`LambdaQueryWrapper`

###  SELECT

设置查询字段，即select * from中*的地方写的

```java
select(String... sqlSelect)
select(Predicate<TableFieldInfo> predicate)
select(Class<T> entityClass, Predicate<TableFieldInfo> predicate)
    
// 例: 
select("id", "name", "age")
select(i -> i.getProperty().startsWith("test"))
```

注意：带`select`的查询返回的数据依旧是整个实体类只不过其他字段都是`null`，例如：

```java
QueryWrapper classesWrapper = new QueryWrapper<Classes>()
                    .select("id")
                    .eq("grade","高三")
# 用于接收的List<Long>就错了，其实接收到的是List<Classes>
List<Long> classesList = classesService.list(classesWrapper);
# 报错
List<Student> students = studentService.list(new QuerryWrapper<Student>().in("class_id",classesList))
            
```



## UpdateWrapper

在`UpdateWrapper`中通过`new UpdateWrapper().lambda()`获取`LambdaUpdateWrapper`

### SET

SQL SET 字段

```java
set(String column, Object val)
set(boolean condition, String column, Object val)
setSql(String sql)
// 例: 
set("name", "张三")
set("name", "")   // 数据库字段值变为空字符串
set("name", null) // 数据库字段值变为空
setSql("name = '张三'")
```

## LambdaWrapper

使用lambda表达式的语法来操作

### LambdaQueryWrapper

在`QueryWrapper`中获取`LambdaQueryWrapper`

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.lambda()
       .between(User::getAge,30,60)
       .orderByDesc(User::getId);
List<User> list = userService.list(queryWrapper);
```

### LambdaUpdateWrapper

在`UpdateWrapper`中获取`LambdaUpdateWrapper`

```java
UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
updateWrapper.lambda()
	.le(User::getAge, 30)
	.setSql("email = 'abc@163.com'");
userService.update(updateWrapper);
```

> 注意，使用`save()`接口和`update()`更新时若`wrapper`为`UpdateWrapper`，则为`null`的字段讲被跳过而不会更新为`null`，所以如果需要将某一字段置为null时，应使用`LambdaUpdateWrapper`

# 自定义SQL

## 方案一：原始的自定义SQL方法

注意要在application.yml中配置Mapper接口对应的xml文件所在位置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.test.mapper.WxUserExtMapper">
<!--注意上面的namespace一定要修改-->

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="com.test.entity.WxUser">
        <id column="id" property="id" />
        <result column="openid" property="openid" />
        <result column="nickname" property="nickname" />
        <result column="sex" property="sex" />
    </resultMap>

    <!--以下是新增的方法-->
    <select id="selectMethod" resultType="com.test.entity.WxUser">
        select * from wx_user;
    </select>

    <select id="oneUser" parameterType="java.lang.Integer" resultType="com.test.entity.WxUser">
        select * from wx_user WHERE id = #{id};
    </select>
</mapper>
```

mapper文件

```java
@Mapper
public interface WxUserExtMapper {
   
    List<WxUser> selectMethod();

    WxUser oneUser(@Param("id") Integer id);
}
```

##  方案二：自定义接口方法使用Wrapper条件构造器

如果我们想在自定义的方法中，使用Wrapper条件构造器。可以参考下面的方式实现。这种方式虽然简单，但仍然只适用于单表（可以是多表关联查询，但查询条件也是基于单表的）。

### 使用注解方式 + Wrapper 

`${ew.customSqlSegment}`是一个查询条件占位符，代表Wapper查询条件，用于生成where

```java
@Select("select * from `user` ${ew.customSqlSegment}")
List<User> selectAll(@Param(Constants.WRAPPER) Wrapper wrapper);
```

### 使用xml 配置方式 + Wrapper

`${ew.customSqlSegment}`是一个查询条件占位符，代表Wapper查询条件，用于生成where

```java
List<User> selectAll(@Param(Constants.WRAPPER) Wrapper wrapper);

<select id="selectAll" resultType="com.zimug.example.model.User">
  select * from `user` ${ew.customSqlSegment}
</select>
```

### 通过Wapper传递查询参数

上面两种方式任意选择一种，参数都是Wrapper

```java
@Test
public void testCustomSQL2() {
  LambdaQueryWrapper<User> query = new LambdaQueryWrapper<>();
  query.eq(User::getName, "字母");

  List<User> list = userMapper.selectAll(query);
  list.forEach(System.out::println);
}

// 最终得到的SQL
// SELECT id,name,age,email 
// FROM user 
// WHERE name = '字母'
```

# ActiveRecord

Active Record(活动记录)，是一种领域模型模式，特点是一个模型类对应关系型数据库中的一个表，而模型类的一个实例对应表中的一行记录。

MyBatis-plus仅需让实体类继承` Model `类且实现主键指定方法即可

## 启动AR的准备工作

domain

```java
@Data
public class User extends Model<User> {    // 实体类继承Model类，重写pkVal方法
    private Integer id;
    private String name;
    private Integer age;
    private Integer gender;
    //重写这个方法，return当前类的主键
    @Override
    protected Serializable pkVal() {
        return id;
    }
}
```

mapper

```java
public interface UserDao extends BaseMapper<User> {
}
```

**注：**虽然AR模式不会直接调用该接口，但是一定要定义，否则使用AR时会报空指针异常。

## 使用AR

使用时并不需要注入mapper接口。AR操作是通过对象本身间接调用Mapper的相关方法，AR的大部分接口也与Mapper同名，比如要insert一个user，那就用这个user调用insert方法即可

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:spring/spring-dao.xml"})
public class TestAR {
    
    // 插入操作
    @Test
    public void testArInsert(){
        User user = new User();
        user.setName("林青霞");
        user.setAge(22);
        user.setGender(1);
        boolean result = user.insert();
        System.out.println(result);
    }
    
    // 更新操作
    @Test
    public void testArUpdate(){
        User user = new User();
        user.setId(1);
        user.setName("刘亦菲");
        boolean result = user.updateById();
        System.out.println(result);
    }
    
    // 查询操作
    @Test
    public void testArSelect(){
        User user = new User();
        //1、根据id查询
        //user = user.selectById(1);
        //或者这样用
        //user.setId(1);
        //user = user.selectById();

        //2、查询所有
        //List<User> users = user.selectAll();

        //3、根据条件查询
        //List<User> users = user.selectList(new EntityWrapper<User>().like("name","刘"));

        //4、查询符合条件的总数
        int result = user.selectCount(new EntityWrapper<User>().eq("gender",1));
        System.out.println(result);
    }
    
    // 删除操作，删除数据库中不存在的数据，结果也是true
    @Test
    public void testArDelete(){
        User user = new User();
        //删除数据库中不存在的数据也是返回true
        //1、根据id删除数据
        //boolean result = user.deleteById(1);
        //或者这样写
        //user.setId(1);
        //boolean result = user.deleteById();

        //2、根据条件删除
        boolean result = user.delete(new EntityWrapper<User>().like("name","玲"));
        System.out.println(result);
    }
    
    // 分页操作
    @Test
    public void testArPage(){
       User user = new User();
       Page<User> page =
               user.selectPage(new Page<>(1,4),
               new EntityWrapper<User>().like("name","刘"));
       List<User> users = page.getRecords();
       System.out.println(users);
    }
}
```

> **分页操作的补充：**
>
> 这个分页方法和BaseMapper提供的分页一样都是内存分页，并非物理分页，因为sql语句中没用limit，和BaseMapper的selectPage方法一样，配置了分页插件后就可以实现真正的物理分页。
>
> AR的分页方法与BaseMapper提供的分页方法不同的是，BaseMapper的selectPage方法返回值是查询到的记录的list集合，而AR的selectPage方法返回的是page对象，该page对象封装了查询到的信息，可以通过getRecords方法获取信息。



参考：

https://www.jianshu.com/p/a4d5d310daf8

https://mybatis.plus/