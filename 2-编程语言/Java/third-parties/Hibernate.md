# hibernate JPQL/HQL

- [hibernate JPQL/HQL](#hibernate-jpqlhql)
  - [Hibernate架构](#hibernate架构)
  - [领域模型/持久化类](#领域模型持久化类)
    - [实体状态刷新flush策略](#实体状态刷新flush策略)
  - [HQL](#hql)
    - [Hibernate创建Query](#hibernate创建query)
    - [HQL绑定参数](#hql绑定参数)
    - [HQL查询结果](#hql查询结果)
      - [HQL滚动结果集](#hql滚动结果集)
    - [HQL提供函数](#hql提供函数)
    - [HQL额外支持的构造器表达式特性](#hql额外支持的构造器表达式特性)
    - [动态Fetch策略](#动态fetch策略)
  - [Hibernate二级缓存(session级别)](#hibernate二级缓存session级别)
  - [缓存配置属性](#缓存配置属性)
    - [实体缓存](#实体缓存)
    - [集合缓存](#集合缓存)
    - [查询缓存](#查询缓存)
    - [Hibernate实体查询计划缓存](#hibernate实体查询计划缓存)

>[Hibernate官方文档](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)

## Hibernate架构

![Hibernate架构](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221116160742.png)

- `SessionFactory`
A thread-safe (and immutable) representation of the mapping of the application domain model to a database. *A SessionFactory is very expensive to create, so, for any given database, the application should have only one associated SessionFactory*. The SessionFactory maintains services that Hibernate uses across all Session(s) such as second level caches, connection pools, transaction system integrations, etc.
- `Session`
A single-threaded, short-lived object conceptually modeling a "Unit of Work". Hibernate `Session` wraps a JDBC `java.sql.Connection` and acts as a factory for `org.hibernate.Transaction` instances. ***It maintains a generally "repeatable read" persistence context (first level cache)*** of the application domain model.
- `Transaction`
A single-threaded, short-lived object used by the application to demarcate individual physical transaction boundaries. *EntityTransaction is the JPA equivalent and both act as an abstraction API to isolate the application from the underlying transaction system in use (JDBC or JTA)*.

![JPA_Hibernate](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/JPA_Hibernate.svg)

## 领域模型/持久化类

Hibernate不严格要求持久化类完全遵守POJO规范，当然遵守POJO规范，性能会更好。
详见[](../javaee/JPA.md#领域模型持久化类)

### 实体状态刷新flush策略

JPA仅提供AUTO, COMMIT两种策略：

- AUTO
This is the default mode, and it flushes the Session only if necessary.
- COMMIT
The Session tries to delay the flush until the current Transaction is committed, although it might flush prematurely too
- ALWAYS
Flushes the Session before every query. 即使是native sql.
- MANUAL
The Session flushing is delegated to the application, which must call Session.flush() explicitly in order to apply the persistence context changes.

## HQL

***The Hibernate `Session` interface extends the JPA `EntityManager` interface.***For this reason, the query API was also merged, and now the Hibernate `org.hibernate.query.Query` interface extends the JPA `javax.persistence.Query`.

### Hibernate创建Query

In Hibernate, the HQL query is represented as `org.hibernate.query.Query` which is obtained from a `Session`. If the HQL is a named query, `Session#getNamedQuery` would be used; otherwise `0` is needed.

```java
org.hibernate.query.Query query = session.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name"
);

org.hibernate.query.Query query = session.getNamedQuery( "get_person_by_name" );
```

然后`Query`接口可以控制查询语句的执行，以及一些特性：

```java
org.hibernate.query.Query query = session.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name" )
// timeout - in seconds
.setTimeout( 2 )
// write to L2 caches, but do not read from them
.setCacheMode( CacheMode.REFRESH )
// assuming query cache was enabled for the SessionFactory
.setCacheable( true )
// add a comment to the generated SQL if enabled via the hibernate.use_sql_comments configuration property
.setComment( "+ INDEX(p idx_person_name)" );
```

### HQL绑定参数

不支持占位参数。

命名参数:

```java
org.hibernate.query.Query query = session.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name" )
.setParameter( "name", "J%", StringType.INSTANCE );//指定参数的数据类型

org.hibernate.query.Query query = session.createQuery(
 "select p " +
 "from Person p " +
 "where p.name like :name" )
.setParameter( "name", "J%" );//简写，Hibernate会推断出参数类型
```

### HQL查询结果

- `Query#list` - executes the select query and returns back the list of results.
- `Query#uniqueResult` - executes the select query and returns the single result. If there were more than one result an exception is thrown.
- `Query#iterate()`
- `Query#stream()` - 查询结果流,需要关闭资源

```java
try( Stream<Person> persons = session.createQuery(
 "select p " +
 "from Person p " +
 "where p.name like :name" )
.setParameter( "name", "J%" )
.stream() ) {

 Map<Phone, List<Call>> callRegistry = persons
   .flatMap( person -> person.getPhones().stream() )
   .flatMap( phone -> phone.getCalls().stream() )
   .collect( Collectors.groupingBy( Call::getPhone ) );

 process(callRegistry);
}
```

Since Hibernate 5.4, the `Stream` is also closed when calling a terminal operation.

```java
List<Person> persons = entityManager.createQuery(
  "select p " +
  "from Person p " +
  "where p.name like :name", Person.class )
.setParameter( "name", "J%" )
.getResultStream()
.skip( 5 )
.limit( 5 )
.collect( Collectors.toList() );
```

- `Query#scroll`- 滚动结果集

#### HQL滚动结果集

Hibernate `Query#scroll` works in tandem with the JDBC notion of a scrollable `ResultSet`. `Query#scroll` returns a `org.hibernate.ScrollableResults` which wraps the underlying JDBC (scrollable) `ResultSet` and provides access to the results. *Unlike a typical forward-only `ResultSet`, the `ScrollableResults` allows you to navigate the ResultSet in any direction*.
`ScrollableResults`会打开JDBC `ResultSet`,因此还要对其进行资源关闭，或者使用`try-with-resources`. 虽然Hibernate会在事务结束(提交或者回滚)后。自动关闭`ScrollableResults`底层的资源，但是手动释放资源还是个推荐做法。

```java
try ( ScrollableResults scrollableResults = session.createQuery(
  "select p " +
  "from Person p " +
  "where p.name like :name" )
  .setParameter( "name", "J%" )
  .scroll()
) {
 while(scrollableResults.next()) {
  Person person = (Person) scrollableResults.get()[0];
  process(person);
 }
}
```

### HQL提供函数

- BIT_LENGTH() 二进制数据的长度
- CAST() 类型转换，要使用[hibernate mapping type](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#basic-provided)

```java
List<String> durations = entityManager.createQuery(
 "select cast( c.duration as string ) " +
 "from Call c ", String.class )
.getResultList();
```

- EXTRACT() 在时间类型的值上提取某部分

```java
List<Integer> years = entityManager.createQuery(
 "select extract( YEAR from c.timestamp ) " +
 "from Call c ", Integer.class )
.getResultList();
```

- YEAR()/MONTH()/DAY()/HOUR()/MINUTE()/SECOND()/ EXTRACT()提取年份/...的缩写
- STR() CAST()为字符串类型的简写

```java
List<String> timestamps = entityManager.createQuery(
 "select str( c.timestamp ) " +
 "from Call c ", String.class )
.getResultList();
List<String> timestamps = entityManager.createQuery(
 "select str( cast(duration as float) / 60, 4, 2 ) " +
 "from Call c ", String.class )
.getResultList();
```

### HQL额外支持的构造器表达式特性

- The query can specify to return a List rather than an Object[] for scalar results:

```java
List<List> phoneCallDurations = entityManager.createQuery(
 "select new list(" +
 " p.number, " +
 " c.duration " +
 ")  " +
 "from Call c " +
 "join c.phone p ", List.class )
.getResultList();
```

- HQL also supports wrapping the scalar results in a Map.

The keys of the map are defined by the aliases given to the select expressions. If the user doesn’t assign aliases, the key will be the index of each particular result set column (e.g. 0, 1, 2, etc).

```java
List<Map> phoneCallTotalDurations = entityManager.createQuery(
 "select new map(" +
 " p.number as phoneNumber , " +
 " sum(c.duration) as totalDuration, " +
 " avg(c.duration) as averageDuration " +
 ")  " +
 "from Call c " +
 "join c.phone p " +
 "group by p.number ", Map.class )
.getResultList();
```

### 动态Fetch策略

除了在注解中指定`FetchType.LAZY` or `FetchType.EAGER`, 也可以使用`@Fetch`和`FetchMode`指定fetch策略. `FetchMode`有以下模式：

- SELECT = Fetchtype.EAGER
The association is going to be fetched lazily using a secondary select for each individual entity, collection, or join load. It’s equivalent to JPA · fetching strategy.
- JOIN = Fetch.LAZY，对于每一个关联关系，都需要一个额外的select查询。
Use an outer join to load the related entities, collections or joins when using direct fetching. It’s equivalent to JPA `FetchType.EAGER` fetching strategy.
- SUBSELECT 在Fetch.LAZY基础上，只是用一个额外的select查询将与实体有关的所有关系集合查询出来。
Available for collections only. When accessing a non-initialized collection, this fetch mode will trigger loading all elements of all collections of the same role for all owners associated with the persistence context using a single secondary select.

## Hibernate二级缓存(session级别)

`@Cache`的属性：

- usage Defines the CacheConcurrencyStrategy
- region Defines a cache region where entries will be stored
- include

## 缓存配置属性

- hibernate.cache.use_second_level_cache
- hibernate.cache.use_query_cache
- hibernate.cache.query_cache_factory
- hibernate.cache.use_minimal_puts
- hibernate.cache.region_prefix
- hibernate.cache.default_cache_concurrency_strategy
- hibernate.cache.use_structured_entries
- hibernate.cache.auto_evict_collection_cache
- hibernate.cache.use_reference_entries
- hibernate.cache.keys_factory

### 实体缓存

```java
@Entity(name = "Phone")
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
public static class Phone {

 @Id
 @GeneratedValue
 private Long id;

 private String mobile;

 @ManyToOne
 private Person person;

 @Version
 private int version;

 //Getters and setters are omitted for brevity

}
```

### 集合缓存

需要在集合属性上使用`@Cache`指定缓存策略`CacheConcurrencyStrategy`, 缓存策略有如下几种:

- NONE
Indicates no concurrency strategy should be applied.
- NONSTRICT_READ_WRITE
Indicates that the non-strict read-write strategy should be applied.
- READ_ONLY
Indicates that read-only strategy should be applied.
- READ_WRITE
Indicates that the read-write strategy should be applied.
- TRANSACTIONAL
Indicates that the transaction strategy should be applied.

```java
@OneToMany(mappedBy = "person", cascade = CascadeType.ALL)
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
private List<Phone> phones = new ArrayList<>(  );
```

### 查询缓存

缓存查询结果会在应用程序的正常事务处理方面带来一些开销。例如，如果你缓存了针对Person的查询结果，Hibernate将需要跟踪这些结果何时失效，因为已经对任何Person实体进行了更改。
再加上大多数应用程序根本无法从缓存查询结果中获得好处，因此Hibernate默认禁用了查询结果的缓存。

```java
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where p.name = :name", Person.class)
.setParameter( "name", "John Doe")
.setHint( "org.hibernate.cacheable", "true")
.getResultList();
```

### Hibernate实体查询计划缓存

>[Query plan cache statistics](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#statistics-query-plan-cache)

Any entity query, be it JPQL or Criteria API, has to be parsed into an AST (Abstract Syntax Tree) so that Hibernate can generate the proper SQL statement. The entity query compilation takes time, and for this reason, Hibernate offers a query plan cache. When executing an entity query, Hibernate first checks the plan cache, and only if there’s no plan available, a new one will be computed right away.

配置：

- `hibernate.query.plan_cache_max_size`
  This setting gives the maximum number of entries of the plan cache. The default value is 2048.
- `hibernate.query.plan_parameter_metadata_max_size`
  The setting gives the maximum number of `ParameterMetadataImpl` instances maintained by the query plan cache. The `ParameterMetadataImpl` object encapsulates metadata about parameters encountered within a query. The default value is 128.
