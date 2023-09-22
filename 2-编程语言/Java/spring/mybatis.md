# mybatis

- [mybatis](#mybatis)
  - [原生Mybatis的数据库操作过程](#原生mybatis的数据库操作过程)
  - [Mybatis核心XML配置](#mybatis核心xml配置)
    - [Properties](#properties)
    - [Environment设置](#environment设置)
      - [transactionManager](#transactionmanager)
      - [数据源](#数据源)
    - [Mybattis内置设置settings](#mybattis内置设置settings)
    - [类型别名](#类型别名)
    - [引入mapper文件](#引入mapper文件)
    - [使用IDEA文件模板简化配置文件](#使用idea文件模板简化配置文件)
  - [基于xml管理bean](#基于xml管理bean)
    - [sql语句获取参数值](#sql语句获取参数值)
    - [查询](#查询)
      - [查询结果](#查询结果)
      - [自定义映射ResultMap](#自定义映射resultmap)
      - [多对一的映射关系](#多对一的映射关系)
      - [一对多的映射关系](#一对多的映射关系)
      - [模糊查询](#模糊查询)
      - [获取生成的自增主键](#获取生成的自增主键)
  - [动态Sql](#动态sql)
  - [Mybatis缓存](#mybatis缓存)
    - [一级缓存](#一级缓存)
    - [二级缓存](#二级缓存)
    - [整合第三方缓存](#整合第三方缓存)
  - [Mybatis逆向工程](#mybatis逆向工程)
  - [Mybatis分页插件](#mybatis分页插件)

## 原生Mybatis的数据库操作过程

```java
SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build();
SqlSession sqlSession = sqlSessionFactory.openSession();

//sqlSession.insert(namespace.sqlId);
xxxMapper mapper = sqlSession.getMapper(xxxMapper.class);
mapper.insertUser();

sqlSession.commit();
sqlSession.close();
```

## Mybatis核心XML配置

Mybatis配置xml时，标签需要按照一定的顺序。

### Properties

`<Properties/>`引入properties文件，使用${key}访问value。

### Environment设置

#### transactionManager

用来设置事务管理方式，有两种类型：

- JDBC 使用JDBC中原生的事务管理方式
- MANAGED 被管理，例如Spring

#### 数据源

datasource设置数据源，有如下几种数据源类型：

- POOLED 使用数据库连接池
- UNPOOLED 不使用数据库连接池
- JNDI 使用上下文中的数据源

### Mybattis内置设置settings

```xml
<settings>
    //下划线转驼峰
    <setting name="mapUnderscoreToCamelCaseEnables" value="true">
    //开启懒加载
    <setting name="lazyLoadingEnabled" value="true">
    <setting name="aggressiveLazyLoading" value="false">//按需加载
<settings/>
```

### 类型别名

`<typeAliases/>`两种方法：

- `<typeAlias/>`为每个实体类分别设置mybatis中用到的类型别名。默认别名是类名，且不区分大小写。
- `<package/>`为该包下的所有类设置默认别名

### 引入mapper文件

`<mappers>`有两种方法引入mapper映射文件：

- `<mapper/>`引入单个mapper文件
- `<package/>引入该包下的所有mapper文件`，需要保证1.mapper接口所在包和映射文件所在文件夹名相同 2.mapper接口和映射文件的名字必须一致

### 使用IDEA文件模板简化配置文件

Mybatis核心配置文件Mybatis-config.xml模板：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <properties resource="xxx/config.properties">
        <property name="username" value="dev_user"/>
        <property name="password" value="F2Fa3!33TYyg"/>
    </properties>

    <typeAliases>
        <package name="xxx.entity"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
        </dataSource>
        </environment>
    </environments>

    <mappers>
        <!-- <mapper resource="org/mybatis/example/BlogMapper.xml"/> -->
        <package name="xxx.xxx.entity"/>
    </mappers>

</configuration>
```

## 基于xml管理bean

mapper接口和mapper配置文件配合实现增删改查本质上就是通过Sqlsession中的方法调用mapper文件中sql语句(id为namespace.sqlId)实现的。

### sql语句获取参数值

占位符赋值#{}，可以避免sql注入，**结果就是带单引号的字符串**。

拼接${}，会存在sql注入风险，**结果不带单引号，如果结果是字符串，需要手动加单引号''号**。

- 单个字面量参数
- 多个字面量参数
  - 默认方式，Mybatis会将参数放在map中，键名为argN或者paramN，值为参数值。因此#{}和${}可以通过访问map中的键访问相应的参数值。如下：
    - #{param1},#{param2}...,#{paramN}
    - #{arg1},#{arg2}...,#{argN}
  - 手动设置map作为参数，可以自定义键名例#{username}/'${username}'
  - ***注解方式***，使用`@Param("username")` ，此时Mybatis会将参数放在map中，有两种方式同时存储：键名为注解中value属性值，键名为param1,...,paramN。例#{username}/'${username}'或者#{param1}/'${param1}'
- ***单个实体类参数***
  - 相当于map，使用属性名获取参数即可，例#{username}/'${username}'

### 查询

#### 查询结果

属性名和类型匹配时，才能成功映射。针对集合返回值也符合下列规则，只是mapper接口中返回值改为List即可。两者不可同时存在，但也不可同时缺少：

- `ResultType` 将查询结果映射到指定类型
  - 实体类(必须字段名和列明一致)
  - Java常用类型
  Mybatis为Java常用类型设置了类型别名，例`Integer`: `int`，具体参考官方文档。
  - 单条记录映射到Map。
  经常查询出来的结果不映射到实体类，则可以设置`ResultType`为map类型，将查询结果映射为map，列名为键名，记录为值。本质上查询结果映射到实体类和map没有区别，区别是：实体类中的属性是固定的；值为`null`的列不映射到map中;
  - 多条记录映射到`List/Map`。
  如果查询结果有多条就映射到`List`(mapper方法返回值为`List`)；当然，也可以将多条记录映射到map中(mapper方法返回值为`Map`)，并且使用`@MapKey(columnName)`手动指定key，值为记录。
- `ResultMap`: 自定义查询结果的映射(字段名和列名不一致或者有关联关系时)

#### 自定义映射ResultMap

当查询结果的列名和实体类的属性名不一致时，有两种办法：

- sql查询时，为列名设置别名，并于实体类属性名保持一致。

```xml
<select id="" resultType="Emp">
select emp_id empId,emp_name empName from emp
<select/>
```

- 在Mybatis全局配置文件中配置`mapUnderscoreToCamelCaseEnables`属性

```xml
<settings>
    <setting name="mapUnderscoreToCamelCaseEnables" value="true">
<settings/>
```

- 使用ResultMap自定义映射

```xml
<resultMap id="empResultMap" type="Emp">
    //主键
    <id column="emp_id" property="empId"/>
    //普通字段
    <result column="emp_name" property="empName"/>
<resultMap/>
```

#### 多对一的映射关系

```java
class Emp{
  private Long empId;
  private String empName;
  private Dept dept;//多对一
}

class Dept{
  private Long deptId;
  private String deptName;
  private List<Emp> emps;//多对一
}

```

基于`ResultMap`，有如下几种方式：

- 级联方式处理

```xml
<resultMap id="empResultMap" type="Emp">
    <id column="emp_id" property="empId"/>
    <result column="emp_name" property="empName"/>
    //多对一
    <result column="dep_id" property="dept.id"/>
<resultMap/>
```

- association标签，用来处理多对一的映射关系(处理实体类类型的属性)。javaType：设置要处理的属性的类型

```xml
<resultMap id="empResultMap" type="Emp">
    <id column="emp_id" property="empId"/>
    <result column="emp_name" property="empName"/>
    //多对一 
    <association property="dept" javaType="Dept">
      <id column="dept_id" property="deptId"/>
      <result column="dept_name" property="deptName"/>
    <association/>
<resultMap/>
```

- 分步查询

可以通过设置全局属性`lazyLoadingEnabled`来开启懒加载。

```java
//DeptMapper.java

//DeptMapper.xml

```

```xml
<resultMap id="empResultMap" type="Emp">
    <id column="emp_id" property="empId"/>
    <result column="emp_name" property="empName"/>
    //多对一 
    <association property="dept" //分步查询出的结果赋值给该属性，所以分步查询的结果必须与该属性的类型相同 
      fetchType="eager" //在开启全局懒加载时，指定该分步查询是延迟加载还是立即加载
      select="DeptMapper.xxxMethod" //分步查询的sql的唯一标识: mapperClassName.methodName。需要额外定义mapper接口和mapper映射文件，
      column="dep_id"/> //分步查询的条件，使用第一步查询中的某个列作为第二步查询的参数
<resultMap/>
```

#### 一对多的映射关系

- collection标签，处理一对多的映射关系(处理集合类型的属性)。

```xml
<resultMap id="depResultMap" type="Dept">
    <id column="dept_id" property="deptId"/>
    <result column="dept_name" property="deptName"/>
    //一对多
    <collection property="emps" ofType="Emp">//ofType: 集合元素类型
      <id column="emp_id" property="empId"/>
      <result column="emp_name" property="empName"/>
    <collection/>
<resultMap/>
```

- 分步查询

#### 模糊查询

- **使用双引号**`select * from user where username like "%"#{username}"%"`
- 单引号中只能用`${}`，不能用`#{}`。`select * from user where username like '%${username}%'`
- 字符串拼接函数。`select * from user where username like concat('%', #{username}. '%')'`

#### 获取生成的自增主键

```xml
//keyProperty: 将添加数据的自增主键为实体类型的参数的属性赋值
<insert useGeneratedKeys="true" keyProperty="id"/>
```

## 动态Sql

Mybatis动态SQL就是根据特定条件动态拼装SQL语句的功能，存在的意义是为了解决拼接sql语句字符串的痛点问题。
当有多个if条件时，若第一个条件不满足则会多出一个and字符串，可以使用where/trim标签，还可以加上恒成立条件，`select * from t_emp where 1=1 ...`。

- if
- choose(when, otherwise)
- trim(where, set)
- foreach

```xml
<select id="" resultType="Emp">
  select * from t_emp where
  <if test="empName != null and empName != ''">
    empName = #{empName}
  </if>
  <if test="age != null and age != ''">
    and empName = #{empName}
  </if>
</select>
```

- where where标签提供的功能：自动生成where关键字；自动删除前面多余的and，但是无法去除后面多余的and；没有条件成立时，where标签不起作用。

```xml
<select id="" resultType="Emp">
  select * from t_emp
  <where>
    <if test="empName != null and empName != ''">
      empName = #{empName}
    </if>
    <if test="age != null and age != ''">
      and age = #{age}
    </if>
    <if test="gender != null and gender != ''">
      and gender = #{gender}
    </if>
  </where>
</select>
```

- trim(set) 在内容前后添加(prefix,suffix)或删除(prefixOverrides, suffixOverrides)指定内容

```xml
<select id="" resultType="Emp">
  select * from t_emp
  <trim prefix="where", suffixOverrides="and">
    <if test="empName != null and empName != ''">
      empName = #{empName}
    </if>
    <if test="age != null and age != ''">
      and age = #{age}
    </if>
    <if test="gender != null and gender != ''">
      and gender = #{gender}
    </if>
  </trim>
</select>
```

- choose(when, otherwise)，相对于if-elseif

```xml
<select id="" resultType="Emp">
  select * from t_emp where
  <choose>
    <when test="empName != null and empName != ''">
      empName = #{empName}
    </when>
    <when test="age != null and age != ''">
      empName = #{empName}
    </when>
    <otherwise>
    </otherwise>
  </choose>
  
</select>
```

- foreach，批量操作常用

```xml
<insert id="">
  insert into t_ emp values
  <foreach collection="emps" item="emp" separator=",">
    (null, #{emp.empName}, #{emp.age})
  </foreach>
</insert>

<delete id="">
  delete from t_ emp where emp_id in 
    <foreach collection="empIds" item="empId" separator="," open="(" close=")">
      #{empId}
    </foreach>
</delete>

<delete id="">
  delete from t_ emp where  
    <foreach collection="empIds" item="empId" separator="or">
      emp_id = #{empId}
    </foreach>
</delete>
```

- sql标签，封装sql语句片段，然后使用include标签引入

```xml
<sql id="sql1">
xxx
<sql>

<select id="" resultType="">
  select <include refid="sql1"/> from t_emp
</select>
```

## Mybatis缓存

### 一级缓存

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221201000922.png)

### 二级缓存

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221201001449.png)

### 整合第三方缓存

## Mybatis逆向工程

[Mybatis逆向工程](https://www.cnblogs.com/MuYg/p/16924419.html)
[Mybatis-generator gui工具](https://github.com/zouzg/mybatis-generator-gui)

## Mybatis分页插件

[Mybatis分页插件](https://github.com/pagehelper/Mybatis-PageHelper)
