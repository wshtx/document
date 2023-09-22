# Java Persistence(JPA)

- [Java Persistence(JPA)](#java-persistencejpa)
  - [相关概念](#相关概念)
    - [原生操作JPA API的过程](#原生操作jpa-api的过程)
    - [Entity manager](#entity-manager)
      - [Persistence Context Lifetime](#persistence-context-lifetime)
      - [Container-Managed Entity Manager](#container-managed-entity-manager)
      - [Application-Managed Entity Manager](#application-managed-entity-manager)
      - [Entity Listener and callback methods](#entity-listener-and-callback-methods)
    - [实体类操作](#实体类操作)
      - [Entity Instance`s lifestyle](#entity-instances-lifestyle)
      - [operations with entity instance`s state](#operations-with-entity-instances-state)
      - [实体状态flush](#实体状态flush)
  - [领域模型/持久化类](#领域模型持久化类)
    - [映射类型](#映射类型)
      - [值类型](#值类型)
      - [实体类型](#实体类型)
    - [标识符](#标识符)
      - [简单标识符](#简单标识符)
      - [复合标识符](#复合标识符)
      - [标识符生成策略](#标识符生成策略)
    - [继承](#继承)
    - [实体注解](#实体注解)
    - [primary key class](#primary-key-class)
    - [Collections as Entity Fields](#collections-as-entity-fields)
    - [关联关系](#关联关系)
      - [关联关系注解](#关联关系注解)
      - [单向/双向@OneToOne](#单向双向onetoone)
      - [单向@OneToMany](#单向onetomany)
      - [单向@ManyToOne](#单向manytoone)
      - [双向@OneToMany/@ManyToOne](#双向onetomanymanytoone)
      - [单向@ManyToMany](#单向manytomany)
      - [双向@ManyToMany](#双向manytomany)
      - [懒加载](#懒加载)
  - [JPQL](#jpql)
    - [JPQL创建Query](#jpql创建query)
    - [JPQL绑定参数](#jpql绑定参数)
    - [JPA查询结果](#jpa查询结果)
    - [语句](#语句)
    - [显式Join](#显式join)
      - [fetch join](#fetch-join)
      - [隐式join((path expressions))](#隐式joinpath-expressions)
    - [Distinct](#distinct)
    - [字面量](#字面量)
    - [算术运算](#算术运算)
    - [函数](#函数)
      - [聚合函数](#聚合函数)
      - [JPQL标准函数](#jpql标准函数)
    - [集合属性相关函数](#集合属性相关函数)
      - [实体类型](#实体类型-1)
    - [CASE表达式](#case表达式)
      - [简单形式](#简单形式)
      - [搜索形式](#搜索形式)
      - [CASE表达式中有算术运算](#case表达式中有算术运算)
        - [NULLIF表达式](#nullif表达式)
        - [coalesce函数](#coalesce函数)
    - [动态初始化/构造器表达式](#动态初始化构造器表达式)
      - [JPQL的构造器表达式](#jpql的构造器表达式)
    - [谓语](#谓语)
    - [Group by](#group-by)
    - [Order by](#order-by)
    - [Read-only 实体](#read-only-实体)
  - [事务](#事务)
  - [Fetch](#fetch)
    - [Direct fetching vs. entity queries](#direct-fetching-vs-entity-queries)
    - [Fetch策略](#fetch策略)
    - [动态fetch策略](#动态fetch策略)
    - [JPA entity graph策略](#jpa-entity-graph策略)
  - [批处理](#批处理)
    - [EntityManager级别的批处理](#entitymanager级别的批处理)
      - [批插入](#批插入)
      - [滚动结果集](#滚动结果集)

## 相关概念

### 原生操作JPA API的过程

1. 加载配置文件创建实体管理器工厂
Persisitence.createEntityMnagerFactory 作用：创建实体管理器工厂
2. 根据实体管理器工厂，创建实体管理器
EntityManagerFactory.createEntityManager 作用：获取EntityManager对象
特点：内部维护的很多的内容：内部维护了数据库信息，维护了缓存信息 维护了所有的实体管理器对象，再创建EntityManagerFactory的过程中会根据配置创建数据库表
EntityManagerFactory的创建过程比较浪费资源；线程安全的对象，多个线程访问同一个EntityManagerFactory不会有线程安全问题
3. 创建事务对象，开启事务
实体类管理器EntityManager

- beginTransaction() : 创建事务对象
- presist() ： 保存
- merge()  ： 更新
- remove() ： 删除
- find/getRefrence() ： 根据id查询，getReference()是懒加载

事务对象Transaction

- begin()：开启事务
- commit()：提交事务
- rollback()：回滚

4. 增删改查操作
5. 提交事务
6. 释放资源

### Entity manager

Entities are managed by the entity manager, which is represented by `javax.persistence.EntityManager` instances. *Each EntityManager instance is associated with a persistence context: a set of managed entity instances that exist in a particular data store*. A persistence context defines the scope under which particular entity instances are created, persisted, and removed. *The EntityManager interface defines the methods that are used to interact with the persistence context*.
The set of entities that can be managed by a given EntityManager instance is defined by a persistence unit.*A persistence unit defines a set of all entity classes that are managed by EntityManager instances in an application.* This set of entity classes represents the data contained within a single data store.

#### Persistence Context Lifetime

The lifetime of a container-managed persistence context can either be scoped to a transaction (transaction-scoped persistence context), or have a lifetime scope that extends beyond that of a single transaction(extended persistence context). The enum `PersistenceContextType` is used to define the
persistence context lifetime scope for container-managed entity managers:

- `TRANSACTION`
  - the lifetime of the persistence context of a container-managed entity manager corresponds to the scope of a transaction
- `EXTENDED`
  - When an extended persistence context is used, the extended persistence context exists from the time the EntityManager instance is created until it is closed. This persistence context might span multiple transactions and non-transactional invocations of the EntityManager.

相关重点：

- When an EntityManager with an extended persistence context is used, the persist, remove, merge, and refresh operations can be called regardless of whether a transaction is active.
- The managed entities of a transaction-scoped persistence context become detached when the transaction commits; the managed entities of an extended persistence context remain managed.
- For both transaction-scoped and extended persistence contexts, transaction rollback causes all pre-existing managed instances and removed instances to become detached. The instances’ state will be the state of the instances at the point at which the transaction was rolled back. Transaction rollback typically
causes the persistence context to be in an inconsistent state at the point of rollback. In particular, the state of version attributes and generated state (e.g., generated primary keys) may be inconsistent. Instances that were formerly managed by the persistence context (including new instances that were
made persistent in that transaction) may therefore not be reusable in the same manner as other detached objects—for example, they may fail when passed to the merge operation.

#### Container-Managed Entity Manager

With a container-managed entity manager, ***anEntityManager instance’s persistence context is automatically propagated by the container to all application components that use the EntityManager instance within a single Java Transaction API (JTA) transaction***. 相比于application-managed entitymanager, container-managed entity manager的许多工作(persistence context生命周期管理，persistence context在JTA事务中传递等)都是容器自动管理。

```java
@PersistenceContext
EntityManager em;
```

#### Application-Managed Entity Manager

*Application-managed entity managers are used when applications need to access a persistence context that is not propagated with the JTA transaction across EntityManager instances in a particular persistence unit*. In this case, each EntityManager creates a new, isolated persistence context. The EntityManager and its associated persistence context are created and destroyed explicitly by the application. They are also used when directly injecting EntityManager instances can’t be done because *EntityManager instances are not thread-safe. EntityManagerFactory instances are thread-safe*.

```java
//inject a EntityManagerFactory instance
@PersistenceUnit
EntityManagerFactory emf;
//create a EntityManager with EntityManagerFactory
EntityManager em = emf.createEntityManager();
```

Application-managed entity managers don’t automatically propagate the JTA transaction context. Such applications need to manually gain access to the JTA transaction manager and add transaction demarcation information when performing entity operations.The
`javax.transaction.UserTransaction` interface defines methods to begin, commit, and roll back transactions.

```java
@Resource
UserTransaction utx;
```

简单的实例展示Application-managed entitymanager和事务的使用：

```java
@PersistenceContext
EntityManagerFactory emf;
//create a EntityManager with EntityManagerFactory
EntityManager em;

@Resource
UserTransaction utx;

public void dosomething(){
  em = emf.createEntityManager();
  try{
    utx.begin();
    em.persist(SomeEntity);
    em.remove(SomeEntity1);
    utx.commit();
  }catch (Exception e){
    utx.rollback();
  }
}
```

#### Entity Listener and callback methods

//TODO Entity Listener

### 实体类操作

#### Entity Instance`s lifestyle

Entity instances are in one of four states: new, managed, detached, or
removed. 实体实例状态取决于是否有`persistent identity`和是否关联`persistence context`。

- New entity instances have no persistent identity and are not yet associated with a persistence context.
- Managed entity instances have a persistent identity and are associated with a persistence context.
- Detached entity instances have a persistent identity and are not currently associated with a persistence context.
- Removed entity instances have a persistent identity, are associated with a persistent context,
and are scheduled for removal from the data store.

#### operations with entity instance`s state

The `persist()`, `merge()`, `remove()`, and `refresh()` methods must be invoked within a transaction context when an entity manager with a transaction-scoped persistence context is used.
Methods that specify a lock mode other than `LockModeType.NONE` must be invoked within a transaction context.
If an entity manager with transaction-scoped persistence context is in use, the resulting entities will be detached; if an entity manager with an extended persistence context is used, they will be managed.

- contains
  - returns true
    - If the entity has been retrieved from the database or has been returned by getReference, and has not been removed or detached.
    - If the entity instance is new, and the persist method has been called on the entity or the persist operation has been cascaded to it.
  - returns false
    - If the instance is detached.
    - If the remove method has been called on the entity, or the remove operation has been cascaded to it.
    - If the instance is new, and the persist method has not been called on the entity or the persist
operation has not been cascaded to it.
- persist()
  - `New` entity instances become managed and persistent either by invoking the persist method or by a cascading persist operation invoked from related entities that have the cascade=PERSIST or cascade=ALL elements set in the relationship annotation.
  - If the entity is already `managed`, the persist operation is ignored, although the persist operation will cascade to related entities that have the cascade element set to PERSIST or ALL in the relationship annotation.
  - If persist is called on a `removed` entity instance, the entity becomes managed.
  - If the entity is `detached`, either persist will throw an `IllegalArgumentException`, or the transaction commit will fail.
- remove()
  - `Managed` entity instances are removed by invoking the remove method or by a cascading remove operation invoked from related entities that have the cascade=REMOVE or cascade=ALL elements set in the relationship annotation.
  - If the remove method is invoked on a `new` entity, the remove operation is ignored, although remove will cascade to related entities that have the cascade element set to REMOVE or ALL in the relationship annotation.
  - If remove is invoked on a `detached` entity, either remove will throw an IllegalArgumentException, or the transaction commit will fail.
  - If invoked on an already `removed` entity, remove will be ignored.
- merge()
  - If X is a detached entity, the state of X is copied onto a pre-existing managed entity instance X' of the same identity or a new managed copy X' of X is created.
  - If X is a new entity instance, a new managed entity instance X' is created and the state of X is copied into the new managed entity instance X'.
  - If X is a removed entity instance, an IllegalArgumentException will be thrown by the merge operation (or the transaction commit will fail).
  - If X is a managed entity, it is ignored by the merge operation, however, the merge operation is cascaded to entities referenced by relationships from X if these relationships have been annotated with the cascade element value cascade=MERGE or cascade=ALL annotation.
  - For all entities Y referenced by relationships from X having the cascade element value cascade=MERGE or cascade=ALL, Y is merged recursively as Y'. For all such Y referenced by X, X' is set to reference Y'. (Note that if X is managed then X is the same object as X'.)
  - If X is an entity merged to X', with a reference to another entity Y, where cascade=MERGE or cascade=ALL is not specified, then navigation of the same association from X' yields a reference to a managed object Y' with the same persistent identity as Y.
- refresh()
  - If X is a managed entity, the state of X is refreshed from the database, overwriting changes made to the entity, if any. The refresh operation is cascaded to entities referenced by X if the relationship from X to these other entities is annotated with the `cascade=REFRESH` or `cascade=ALL` annotation element value.
  - If X is a new, detached, or removed entity, the `IllegalArgumentException` is thrown.
- flush()
  - The state of persistent entities is synchronized to the database when the transaction with which the entity is associated commits. To *force synchronization* of the managed entity to the data store, invoke the flush method of the EntityManager instance.
  - If X is a managed entity, it is synchronized to the database.
    - For all entities Y referenced by a relationship from X, if the relationship to Y has been annotated with the cascade element value `cascade=PERSIST` or `cascade = ALL`, the persist operation is applied to Y.
    - For any entity Y referenced by a relationship from X, where the relationship to Y has not been annotated with the cascade element value `cascade=PERSIST` or `cascade = ALL`:
      - If Y is new or removed, an IllegalStateException will be throw by the flush operation (and the transaction marked for rollback) or the transaction commit will fail.
      - If Y is detached, the semantics depend upon the ownership of the relationship. If X owns the relationship, any changes to the relationship are synchronized with the database; otherwise, if Y owns the relationships, the behavior is undefined.
  - If X is a removed entity, it is removed from the database. No cascade options are relevant.
- detach()
  - If X is a managed entity, the detach operation causes it to become detached. The detach operation is cascaded to entities referenced by X if the relationships from X to these other entities is annotated with the cascade=DETACH or cascade=ALL annotation element value. Entities
which previously referenced X will continue to reference X.
  - If X is a new or detached entity, it is ignored by the detach operation.
  - If X is a removed entity, the detach operation is cascaded to entities referenced by X if the relationships
from X to these other entities is annotated with the cascade=DETACH or cascade= ALL annotation element value. Entities which previously referenced X will continue to reference X. Portable applications should not pass removed entities that have been detached from the persistence context to further EntityManager operations.

#### 实体状态flush

>[Hibernate Flush strategies](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#flushing)

JPA提供的刷新策略：

- AUTO
This is the default mode, and it flushes the Session only if necessary.可能有以下三种情形：
  - prior to committing a Transaction
  - prior to executing a JPQL/HQL query that overlaps with the queued entity actions.
  - before executing any native SQL query that has no registered synchronization.
  When executing a native SQL query, a flush is always triggered when using the EntityManager API
- COMMIT
延迟到事务被提交过程中才刷新。或者在执行native sql语句之前刷新

刷新操作会会映射到具体的SQL语句，

- INSERT
INSERT语句被`EntityInsertAction`或者`EntityIdentityInsertAction`生成。这些操作是由显式persist()触发的，也可以是通过从父实体级联到子实体的`PersistEvent`。
- DELETE
DELETE语句被`EntityDeleteAction`或者`OrphanRemovalAction`生成。
- UPDATE
如果managed状态的实体被标记为已被修改，UPDATE语句将由`EntityUpdateAction`在刷新期间生成。dirty-checking机制负责确定一个被管理的实体在首次加载后是否被修改。

执行SQL语句的顺序是由ActionQueue给出的，而不是由之前定义的实体状态操作的顺序决定的:

- OrphanRemovalAction(孤儿删除)
- EntityInsertAction or EntityIdentityInsertAction(插入)
- EntityUpdateAction(更新)
- QueuedOperationCollectionAction
- CollectionRemoveAction
- CollectionUpdateAction
- CollectionRecreateAction
- EntityDeleteAction(删除)

## 领域模型/持久化类

### 映射类型

#### 值类型

值类型就是没有声明周期，构成实体类型的类型。一般分为：

- [基本数据类型](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#basic) 就是将数据库表的列映射为非聚合的Java类型
- 嵌套类型

>实体类中同时存在多个同一组件，会造成命名冲突，[解决办法1](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#embeddable-override)，[解决办法2](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#embeddable-multiple-namingstrategy)  
>使用接口来表示嵌套类型时，[做法](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#embeddable-Target)

Historically Hibernate called these components. JPA calls them embeddables. Either way, the concept is the same: a composition of values. 使用`@Embeddable`定义嵌套类和`@Embedded`在实体类中标明嵌套类. 主要是用来组合多个基本类型映射并在多个实体类型中复用。

```java
@Entity(name = "Book")
public static class Book {

 @Id
 @GeneratedValue
 private Long id;

 private String title;

 private String author;

 private Publisher publisher;

 //Getters and setters are omitted for brevity
}

@Embeddable
public static class Publisher {

 @Column(name = "publisher_name")
 private String name;

 @Column(name = "publisher_country")
 private String country;

 //Getters and setters, equals and hashCode methods omitted for brevity

}
```

也可以使用`@Parent`在子类中标明父类，作为对父类的引用。

```java
@Embeddable
public static class GPS {

 private double latitude;

 private double longitude;

 @Parent
 private City city;

 //Getters and setters omitted for brevity

}

@Entity(name = "City")
public static class City {

 @Id
 @GeneratedValue
 private Long id;

 private String name;

 @Embedded
 @Target( GPS.class )
 private GPS coordinates;

 //Getters and setters omitted for brevity

}

doInJPA( this::entityManagerFactory, entityManager -> {

 City cluj = entityManager.find( City.class, 1L );

 assertSame( cluj, cluj.getCoordinates().getCity() );
} );
```

```SQL
create table Book (
    id bigint not null,
    author varchar(255),
    publisher_country varchar(255),
    publisher_name varchar(255),
    title varchar(255),
    primary key (id)
)
```

- 集合类型

#### 实体类型

>[Hibernate对实体类的要求](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#entity-pojo)
>[有关scheme/catelog的设置，mysql不支持](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#mapping-entity-table-catalog)
>[将实体映射到SQL语句(视图)](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#entity-sql-query-mapping)
实体类型就是领域模型，与数据库的表相对应，使用唯一标识符，有自己独立的生命周期。Entity Class的要求：

- 实体类必须要有`public/protected`的无参构造器
- If an entity instance is passed by value as a detached object, such as through a session bean’s remote business interface, the class must implement the `Serializable` interface.
- 实体类属性必须声明成`private/protected/package-private`，并且提供访问方法
- All fields not annotated `javax.persistence.Transient` or not marked as Java transient will be persisted to the data store.
- If the property is a Boolean, you may use isProperty instead of getProperty.
- 非实体类超类中的属性和注解在实体类子类中都被忽略

### 标识符

标识符不需要映射到作为表主键的物理定义的列。他们只需要映射到能唯一识别每一行的列。每个实体都必须定义一个标识符。对于实体的继承层次，标识符必须只定义在作为根的实体上。
标识符不需要映射到作为表主键的物理定义的列。他们只需要映射到能唯一识别每一行的列。每个实体都必须定义一个标识符。对于实体的继承层次，标识符必须只定义在作为根的实体上。

#### 简单标识符

JPA中定义的标识符类型：

- any Java primitive type
- any primitive wrapper type
- java.lang.String
- java.util.Date (TemporalType#DATE)
- java.sql.Date
- java.math.BigDecimal
- java.math.BigInteger

#### 复合标识符

- [使用`@EmbeddedId`来定义实体类标识符](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#identifiers-composite-aggregated)
- [使用`@IdClass`定义实体类标识符](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#identifiers-composite-nonaggregated)
- [有关联关系的复合标识符](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#identifiers-composite-associations)
JPA中定义的复合类型标识符的规范：

- The composite identifier must be represented by a "primary key class". The primary key class may be defined using the javax.persistence.EmbeddedId annotation (see Composite identifiers with @EmbeddedId), or defined using the javax.persistence.IdClass annotation (see Composite identifiers with @IdClass).
- The primary key class must be public and must have a public no-arg constructor.
- The primary key class must be serializable.
- The primary key class must define equals and hashCode methods, consistent with equality for the underlying database types to which the primary key is mapped.

#### 标识符生成策略

JPA portably defines identifier value generation just for integer types.

使用`@GeneratedValue`和`@GenerationType`来指定标识符生成策略：

- GeneratedType.AUTO(Default)
取决于JPA规范的实现者，由它来选择一个合适的策略
- GeneratedType.TABLE
JPA提供的机制，通过一张表帮助完成主键自增。
- GeneratedType.IDENTIFY
自增，底层数据库必须支持自动增长方式(mysql)。Indicates that database IDENTITY columns will be used for primary key value generation.
- GeneratedType.SEQUENCE
序列，底层数据库必须支持序列(oracle)。Indicates that database sequence should be used for obtaining primary key values.

```java
@Entity(name = "Product")
public static class Product {

 @Id
 @GeneratedValue(
  strategy = GenerationType.SEQUENCE,
  generator = "sequence-generator"
 )
 @SequenceGenerator(
  name = "sequence-generator",
  sequenceName = "product_sequence",
  allocationSize = 5
 )
 private Long id;

 @Column(name = "product_name")
 private String name;

 //Getters and setters are omitted for brevity

}
```

### 继承

Inheritance Mapping Strategies:

- a single table per class hierarchy
  - 每个继承层次都对应着一张数据库表
- a joined subclass strategy
  - 超类对应着一张表，子类对应着一张表，但是子类表中不包含从超类中继承的关系和属性，而是通过外键和超类表关联
- a table per concrete entity class
  - 每个实体类都对应着一张表(包括超类和子类), 且子类中包含从超类中继承的所有关系和属性

### 实体注解

- `@MappedSuperclass` 标明实体类的超类，超类只用于封装公用属性，不用作查询，关联关系等操作
- `@AttributeOverride` 覆盖从超类中继承的属性
- `@Entity`
  - 标识该类是数据库实体类
  - The @Entity annotation defines just the name attribute which is used to give a specific entity name for use in JPQL queries.
  - if the name attribute of the @Entity annotation is missing, the unqualified name of the entity class itself will be used as the entity name.
- `@Table` 指定实体类对应的表名
- `@Id` 用来指定主键
- `@Column` 指定实体类属性对应的表字段名

### primary key class

### Collections as Entity Fields

如果实体类有集合作为属性时, 使用`javax.persistence.ElementCollection`，两个属性：

- targetClass 集合的元素类型
- fetch 指定集合被检索的方式：lazily or eagerly, 使用`javax.persistence.FetchType` constants of either LAZY or EAGER, respectively. By default, the collection will be fetched lazily.

### 关联关系

#### 关联关系注解

***关联关系有三个属性：多/一方、外键表方、主(owner side)/从表(inverse side)。使用关联关系时，会在owner side生成一个指向inverse side的外键字段或者使用中间表来维护关系***。

- `@ManyToOne` 多对一，数据表中关联关系(外键)一般    放在多的一方维护
  - `mappedBy`
  一般用在"inverse side"，值等于另一个实体类的实体属性名，用于双向关联关系(有两个外键约束)时，放弃这一方的外键约束，保留另一方的外键约束。这是因为级联操作时会由于两个外键相互引用，导致删除失败。
  - `cascade` 级联类型
    - 所有`CascadeType.All`
    - 插入`CascadeType.PERSIST`
    - 更新`CascadeType.MERGE`
    - 刷新 `CascadeType.REFRESH`
  - `fetch`
    - 立刻加载`FetchType.EAGER`
    - 懒加载`FetchType.LAZY`
  - `orphanRemoval`
- `@OneToMany` 一对多
- `@ManyToMany` 多对多
- `@OneToOne` 一对一
- `@JoinColumn` 在owner side(有外键的一方)，设置关联关系
  - `name` "owner side"对应的表中的外键字段名字，默认是"表名_id"
  - `referencedColumnName` 外键映射到另外一张表的表字段名
  - `foreignKey`
- `@JoinTables` 在owner side(有外键的一方)使用, 设置关联关系
  - `name` 中间表名
  - `joinColumns` 设置当前实体类对应的外键名
  - `inverseJoinColumns` 设置关联表的外键名

*A bidirectional relationship has both an owning side and an inverse side. A unidirectional relationship has only an owning side*. The owning side of a relationship determines how the Persistence runtime makes updates to the relationship in the database. Bidirectional relationships must follow these rules.

- ***The inverse side of a bidirectional relationship must refer to its owning side by using the mappedBy element*** of the `@OneToOne`, `@OneToMany`, or `@ManyToMany` annotation. The mappedBy element designates the property or field in the entity that is the owner of the relationship.
- The many side of many-to-one bidirectional relationships must not define the mappedBy element. ***The many side is always the owning side of the relationship in many-to-one/one-to-many relationships***.
- For one-to-one bidirectional relationships, the owning side corresponds to the side that contains the corresponding foreign key.
- For many-to-many bidirectional relationships, either side may be the owning side.

#### 单向/双向@OneToOne

#### 单向@OneToMany

单向OneToMany关系，有两种实现方式：
>A bidirectional `@OneToMany` association is much more efficient because the child entity controls the association. Every element removal only requires a single update (in which the foreign key column is set to NULL.

- @OneToMany 生成中间表维护关联关系

```java
@Entity(name = "Person")
public static class Person {

 @Id
 @GeneratedValue
 private Long id;

 @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
 private List<Phone> phones = new ArrayList<>();

 //Getters and setters are omitted for brevity

}

@Entity(name = "Phone")
public static class Phone {

 @Id
 @GeneratedValue
 private Long id;

 @Column(name = "`number`")
 private String number;

 //Getters and setters are omitted for brevity

}
```

```sql
CREATE TABLE Person (
    id BIGINT NOT NULL ,
    PRIMARY KEY ( id )
)

CREATE TABLE Person_Phone (
    Person_id BIGINT NOT NULL ,
    phones_id BIGINT NOT NULL
)

CREATE TABLE Phone (
    id BIGINT NOT NULL ,
    number VARCHAR(255) ,
    PRIMARY KEY ( id )
)

ALTER TABLE Person_Phone
ADD CONSTRAINT UK_9uhc5itwc9h5gcng944pcaslf
UNIQUE (phones_id)

ALTER TABLE Person_Phone
ADD CONSTRAINT FKr38us2n8g5p9rj0b494sd3391
FOREIGN KEY (phones_id) REFERENCES Phone

ALTER TABLE Person_Phone
ADD CONSTRAINT FK2ex4e4p7w1cj310kg2woisjl2
FOREIGN KEY (Person_id) REFERENCES Person
```

- @OneToMany + @JoinColumn 就在多方生成外键字段，建立外键约束指向一方

```java
@Data
@Entity
@Table(name = "tb_people")
public class People {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "people_id")
    private Long peopleId;
    private String name;

    /**
     *  @OneToMany
     *    cascade = CascadeType.ALL：级联保存、更新、删除、刷新
     *    fetch = FetchType.LAZY   ：延迟加载
     *  @JoinColumn
     *    name 指定外键列，这里注意指定的是people_id,实际上是为了外键表定义的字段。该字段在PeoplePhone类必须定义
     */
    @OneToMany
    @JoinColumn(name="people_id")
    private List<PeoplePhone> peoplePhones = new ArrayList<>();
}

@Data
@Entity
@Table(name = "tb_people_phone")
public class PeoplePhone {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "phone_id")
    private Long phoneId;
    private String type;
    private String phone;
}
```

```sql
 create table tb_people (
       people_id bigint not null auto_increment,
        name varchar(255),
        primary key (people_id)
    ) engine=InnoDB
Hibernate: 
    
create table tb_people_phone (
    phone_id bigint not null auto_increment,
    phone varchar(255),
    type varchar(255),
    people_id bigint,
    primary key (phone_id)
) engine=InnoDB

alter table tb_people_phone 
    add constraint FKnied6axrmqsyl5olnjywa7set 
    foreign key (people_id) 
    references t_people (people_id)
```

#### 单向@ManyToOne

在onwer side(多方、外键表方、从表)中使用注解 @ManyToOne 和 @JoinColumn 标注类型为对方的属性.

#### 双向@OneToMany/@ManyToOne

1的一端需要使用注解@OneToMany标注类型为对方的集合属性，同时指定mappedBy属性表示1的一端不控制关系，N的一端则需要使用注解@ManyToOne 和 @JoinColumn 标注类型为对方的属性。

#### 单向@ManyToMany

When using a unidirectional `@ManyToMany` association,  using a link table between the two joining entities.

需要在owner side中使用注解@ManyToMany 和 @JoinTable设置中间表信息.

```java
@Data
@Entity
@Table(name = "tb_role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "role_id")
    private Long roleId;
    @Column(name = "role_name")
    private String roleName;

    /**
     * @JoinTable:
     *   name：中间表名称
     *   joinColumns          中间表对应本类的信息
     *     @JoinColumn；
     *       name：中间表中owner side对应的字段（中间表的数据库字段）
     *       referencedColumnName：中间表该字段指向的字段（owner side的主键字段）
     *   inverseJoinColumns    中间表对应对方类的信息
     *     @JoinColumn：
     *       name：中间表中inverse side对应的字段（中间表的数据字段）
     *       referencedColumnName：中间表该字段指向的字段（inverse side的主键字段）
     */
    @ManyToMany
    @JoinTable(name="tb_role_permission", // 中间表明
            joinColumns=@JoinColumn(
                    name="role_id", // 本类的外键
                    referencedColumnName = "role_id"), // 本类与外键(表)对应的主键
            inverseJoinColumns=@JoinColumn(
                    name="permission_id", // 对方类的外键
                    referencedColumnName = "permission_id")) // 对方类与外键(表)对应的主键
    private Set<Permission> permissions = new HashSet<>();
}
```

#### 双向@ManyToMany

owner side设置中间表:

```java
@Data
@Entity
@Table(name = "tb_role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "role_id")
    private Long roleId;
    @Column(name = "role_name")
    private String roleName;

    /**
     * @JoinTable:
     *   name：中间表名称
     *   joinColumns          中间表对应本类的信息
     *     @JoinColumn；
     *       name：中间表中owner side对应的字段（中间表的数据库字段）
     *       referencedColumnName：中间表该字段指向的字段（owner side的主键字段）
     *   inverseJoinColumns    中间表对应对方类的信息
     *     @JoinColumn：
     *       name：中间表中inverse side对应的字段（中间表的数据字段）
     *       referencedColumnName：中间表该字段指向的字段（inverse side的主键字段）
     */
    @ManyToMany
    @JoinTable(name="tb_role_permission", // 中间表明
            joinColumns=@JoinColumn(
                    name="role_id", // 本类的外键
                    referencedColumnName = "role_id"), // 本类与外键(表)对应的主键
            inverseJoinColumns=@JoinColumn(
                    name="permission_id", // 对方类的外键
                    referencedColumnName = "permission_id")) // 对方类与外键(表)对应的主键
    private Set<Permission> permissions = new HashSet<>();
}
```

inverse side设置`mappedby`属性:

```java
@Data
@Entity
@Table(name = "t_permission")
public class Permission {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "permission_id")
    private Long permissionId;
    @Column(name = "permission_name")
    private String permissionName;

    /**
     * @ManyToMany
     *   mappedBy:对方类应用该类的属性名，指明这端不控制关系
     *   cascade | fetch | targetEntity 为可选属性
     */
    @ManyToMany(mappedBy = "permissions")
    private Set<Role> role = new HashSet<>();
}
```

#### 懒加载

## JPQL

JPQL是HQL的一个子集，JPQL的查询语句必定是HQL语句，反之则不然。

### JPQL创建Query

In JPA, the query is represented by `javax.persistence.Query` or `javax.persistence.TypedQuery` as obtained from the `EntityManager`. The create an inline `Query` or `TypedQuery`, you need to use the `EntityManager#createQuery` method. For named queries, the `EntityManager#createNamedQuery` method is needed.

创建JPQL的`Query`和`TypedQuery`实例如下：

```java
Query query = entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name"
);

TypedQuery<Person> typedQuery = entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name", Person.class
);
```

创建JPQL的命名查询实例如下：

```java
@NamedQuery(
    name = "get_person_by_name",
    query = "select p from Person p where name = :name"
)

Query query = entityManager.createNamedQuery( "get_person_by_name" );
TypedQuery<Person> typedQuery = entityManager.createNamedQuery("get_person_by_name", Person.class);
```

The `Query` interface can then be used to control the execution of the query. For example, we may want to specify an execution timeout or control caching.
>[Query接口信息](https://javaee.github.io/javaee-spec/javadocs/javax/persistence/Query.html)

```java
Query query = entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name" )
// timeout - in milliseconds
.setHint( "javax.persistence.query.timeout", 2000 )
// flush only at commit time
.setFlushMode( FlushModeType.COMMIT );
```

### JPQL绑定参数

命名参数:

```java
Query query = entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name" )
.setParameter( "name", "J%" );

// For generic temporal field types (e.g. `java.util.Date`, `java.util.Calendar`)
// we also need to provide the associated `TemporalType`
Query query = entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.createdOn > :timestamp" )
.setParameter( "timestamp", timestamp, TemporalType.DATE );
```

占位参数

```java
Query query = entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like ?1" )
.setParameter( 1, "J%" );
```

### JPA查询结果

JPQL提供了三种查询结果的方法：

- `Query#getResultList()` - executes the select query and returns back the list of results.

```java
  List<Person> persons = entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name" )
.setParameter( "name", "J%" )
.getResultList();
```

- `Query#getResultStream()` - executes the select query and returns back a `Stream` over the results.

```java
try(Stream<Person> personStream = entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name", Person.class )
.setParameter( "name", "J%" )
.getResultStream()) {
    List<Person> persons = personStream
        .skip( 5 )
        .limit( 5 )
        .collect( Collectors.toList() );
}
```

- `Query#getSingleResult()` - executes the select query and returns `a single result`1. If there were more than one result an exception is thrown.  

```java
Person person = (Person) entityManager.createQuery(
    "select p " +
    "from Person p " +
    "where p.name like :name" )
.setParameter( "name", "J%" )
.getSingleResult();
```

### 语句

语句作为查询API的参数。

>Caution should be used when executing bulk update or delete operations because they may result in inconsistencies between the database and the entities in the active persistence context. In general, *bulk update and delete operations should only be performed within a transaction in a new persistence context or before fetching or accessing entities whose state might be affected by such operations*

- Select语句

JPQL中的SELECT语句与HQL中相同，但是JPQL需要select从句，HQL不需要. 但是显式包括select从句是推荐做法。HQL中SELECT语句的BNF：

```java
select_statement :: =
    [select_clause]
    from_clause
    [where_clause]
    [groupby_clause]
    [having_clause]
    [orderby_clause]
```

- Update语句

>***UPDATE statements, by default, do not affect the version or the timestamp attribute values for the affected entities***. However just in Hibernate, you can force Hibernate to set the version or timestamp attribute values through the use of a versioned update. This is achieved by adding the VERSIONED keyword after the UPDATE keyword.  
>An UPDATE statement is executed using the `executeUpdate()` of either `org.hibernate.query.Query` or `javax.persistence.Query`. The int value returned by the `executeUpdate()` method indicates the number of entities affected by the operation. The int value returned by the executeUpdate() method indicates the number of entities affected by the operation. This may or may not correlate to the number of rows affected in the database. An HQL bulk operation might result in multiple actual SQL statements being executed (for joined-subclass, for example). The returned number indicates the number of actual entities affected by the statement. Using a JOINED inheritance hierarchy, a delete against one of the subclasses may actually result in deletes against not just the table to which that subclass is mapped, but also the "root" table and tables "in between".

JPQL和HQL的Update语句的BNF相同：

```java
update_statement ::=
    update_clause [where_clause]

update_clause ::=
    UPDATE entity_name [[AS] identification_variable]
    SET update_item {, update_item}*

update_item ::=
    [identification_variable.]{state_field | single_valued_object_field} = new_value

new_value ::=
    scalar_expression | simple_entity_expression | NULL
```

- Delete语句
A DELETE statement is also executed using the executeUpdate() method of either org.hibernate.query.Query or javax.persistence.Query. HQL和JPQL中Delete语句的BNF相同：

```java
delete_statement ::=
    delete_clause [where_clause]

delete_clause ::=
    DELETE FROM entity_name [[AS] identification_variable]
```

- Insert语句

>only attributes directly defined on the named entity can be used in the attribute_list. Superclass properties are not allowed and subclass properties do not make sense. In other words, INSERT statements are inherently non-polymorphic.  
>

JPQL不支持HQL风格的Insert语句,HQL中Insert语句的BNF如下：

```java
insert_statement ::=
    insert_clause select_statement

insert_clause ::=
    INSERT INTO entity_name (attribute_list)

attribute_list ::=
    state_field[, state_field ]*
```

```java
int insertedEntities = session.createQuery(
 "insert into Partner (id, name) " +
 "select p.id, p.name " +
 "from Person p ")
.executeUpdate();
```

### 显式Join

有innerleft outer两种join方式。

```java
List<Person> persons = entityManager.createQuery(
 "select distinct pr " +
 "from Person pr " +
 "join pr.phones ph " + //"inner join pr.phones ph " +
 "where ph.type = :phoneType", Person.class )
.setParameter( "phoneType", PhoneType.MOBILE )
.getResultList();

List<Person> persons = entityManager.createQuery(
 "select distinct pr " +
 "from Person pr " +
 "left join pr.phones ph " + //"left outer join pr.phones ph " +
 "where ph is null " +
 "   or ph.type = :phoneType", Person.class )
.setParameter( "phoneType", PhoneType.LAND_LINE )
.getResultList();
```

HQL定义了with从句来限定join的条件，而JPQL提供的是on从句。

```java
//HQL
List<Object[]> personsAndPhones = session.createQuery(
 "select pr.name, ph.number " +
 "from Person pr " +
 "left join pr.phones ph with ph.type = :phoneType " )
.setParameter( "phoneType", PhoneType.LAND_LINE )
.list();

//JPQL
List<Object[]> personsAndPhones = entityManager.createQuery(
 "select pr.name, ph.number " +
 "from Person pr " +
 "left join pr.phones ph on ph.type = :phoneType " )
.setParameter( "phoneType", PhoneType.LAND_LINE )
.getResultList();
```

#### fetch join

>Fetch joins are not valid in sub-queries. Care should be taken when fetch joining a collection-valued association which is in any way further restricted (the fetched collection will be restricted too). For this reason, it is usually considered best practice not to assign an identification variable to fetched joins except for the purpose of specifying nested fetch joins. Fetch joins should not be used in paged queries (e.g. setFirstResult() or setMaxResults()), nor should they be used with the scroll() or iterate() features.

fetch join特性是为了覆盖joined association的懒加载特性。

```java
List<Person> persons = entityManager.createQuery(
 "select distinct pr " +
 "from Person pr " +
 "left join fetch pr.phones ", Person.class )
.getResultList();
```

#### 隐式join((path expressions))

>Implicit joins are always treated as inner joins.  
>If the attribute represents an entity association (non-collection) or a component/embedded, that reference can be further navigated. Basic values and collection-valued associations cannot be further navigated.

```java
List<Phone> phones = entityManager.createQuery(
 "select ph " +
 "from Phone ph " +
 "where ph.person.address = :address ", Phone.class )
.setParameter( "address", address )
.getResultList();

// same as
List<Phone> phones = entityManager.createQuery(
 "select ph " +
 "from Phone ph " +
 "join ph.person pr " +
 "where pr.address = :address ", Phone.class )
.setParameter( "address", address)
.getResultList();
```

### Distinct

对于HQL和JPQL，DISTINCT有两种含义：

- It can be passed to the database so that duplicates are removed from a result set
  
```java
List<String> lastNames = entityManager.createQuery(
 "select distinct p.lastName " +
 "from Person p", String.class)
.getResultList();
```

- It can be used to filter out the same parent entity references when join fetching a child collection

```java
//会返回六个结果，因为the SQL-level result-set size is given by the number of joined Book records. In this case, the DISTINCT SQL keyword is undesirable since it does a redundant result set sorting.
List<Person> authors = entityManager.createQuery(
 "select distinct p " +
 "from Person p " +
 "left join fetch p.books", Person.class)
.getResultList();

//使用`HINT_PASS_DISTINCT_THROUGH`设置，Hibernate can still remove the duplicated parent-side entities from the query result.
List<Person> authors = entityManager.createQuery(
 "select distinct p " +
 "from Person p " +
 "left join fetch p.books", Person.class)
.setHint( QueryHints.HINT_PASS_DISTINCT_THROUGH, false )
.getResultList();
```

### 字面量

```java
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where p.name like 'Joe'", Person.class)
.getResultList();

// Escaping quotes
//String literals are enclosed in single quotes. To escape a single quote within a string literal, use double single quotes.
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where p.name like 'Joe''s'", Person.class)
.getResultList();

// simple integer literal
Person person = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where p.id = 1", Person.class)
.getSingleResult();

// simple integer literal, typed as a long
Person person = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where p.id = 1L", Person.class)
.getSingleResult();

// decimal notation
List<Call> calls = entityManager.createQuery(
 "select c " +
 "from Call c " +
 "where c.duration > 100.5", Call.class )
.getResultList();

// decimal notation, typed as a float
List<Call> calls = entityManager.createQuery(
 "select c " +
 "from Call c " +
 "where c.duration > 100.5F", Call.class )
.getResultList();

// scientific notation
List<Call> calls = entityManager.createQuery(
 "select c " +
 "from Call c " +
 "where c.duration > 1e+2", Call.class )
.getResultList();

// scientific notation, typed as a float
List<Call> calls = entityManager.createQuery(
 "select c " +
 "from Call c " +
 "where c.duration > 1e+2F", Call.class )
.getResultList();
```

### 算术运算

算术运算可以用在select从句和where从句中：

```java
// select clause date/time arithmetic operations
Long duration = entityManager.createQuery(
 "select sum(ch.duration) * :multiplier " +
 "from Person pr " +
 "join pr.phones ph " +
 "join ph.callHistory ch " +
 "where ph.id = 1L ", Long.class )
.setParameter( "multiplier", 1000L )
.getSingleResult();

// select clause date/time arithmetic operations
Integer years = entityManager.createQuery(
 "select year( current_date() ) - year( p.createdOn ) " +
 "from Person p " +
 "where p.id = 1L", Integer.class )
.getSingleResult();

// where clause arithmetic operations
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where year( current_date() ) - year( p.createdOn ) > 1", Person.class )
.getResultList();
```

The following rules apply to the result of arithmetic operations:

- If either of the operands is Double/double, the result is a Double
- else, if either of the operands is Float/float, the result is a Float
- else, if either operand is BigDecimal, the result is BigDecimal
- else, if either operand is BigInteger, the result is BigInteger (except for division, in which case the result type is not further defined)
- else, if either operand is Long/long, the result is Long (except for division, in which case the result type is not further defined)
- else, (the assumption being that both operands are of integral type) the result is Integer (except for division, in which case the result type is not further defined)

### 函数

#### 聚合函数

```java
Object[] callStatistics = entityManager.createQuery(
 "select " +
 " count(c), " +
 " sum(c.duration), " +
 " min(c.duration), " +
 " max(c.duration), " +
 " avg(c.duration)  " +
 "from Call c ", Object[].class )
.getSingleResult();

Long phoneCount = entityManager.createQuery(
 "select count( distinct c.phone ) " +
 "from Call c ", Long.class )
.getSingleResult();

List<Object[]> callCount = entityManager.createQuery(
 "select p.number, count(c) " +
 "from Call c " +
 "join c.phone p " +
 "group by p.number", Object[].class )
.getResultList();
```

#### JPQL标准函数

- CONCAT() 字符串连接函数，把两个或以上的字符串连接起来
  
```java
  List<String> callHistory = entityManager.createQuery(
 "select concat( p.number, ' : ' , cast(c.duration as string) ) " +
 "from Call c " +
 "join c.phone p", String.class )
.getResultList();
```

- SUBSTRING() 字符串截取函数，第2个参数是起始位置，第s个参数是截取长度

```java
List<String> prefixes = entityManager.createQuery(
 "select substring( p.number, 1, 2 ) " +
 "from Call c " +
 "join c.phone p", String.class )
.getResultList();
```

- UPPER() 大写化给定字符串
- LOWER() 小写化给定字符串
- TRIM() 去除空格
- LENGTH() 字符串长度
- LOCATE() 定位子串的位置，第3个参数是开始查找位置
  
```java
  List<Integer> sizes = entityManager.createQuery(
 "select locate( 'John', p.name ) " +
 "from Person p ", Integer.class )
.getResultList();
```

- ABS() 绝对值
- MOD() 取模
- SQRT() 开平方
- CURRENT_DATE() 返回数据库的当前日期
- CURRENT_TIME() 返回数据库的当前时间
- CURRENT_TIMESTAMP() 返回数据库的当前timestamp

### 集合属性相关函数

```java

//maxelement 
//Available for use on collections of basic type. Refers to the maximum value as determined by applying the max SQL aggregation.
List<Phone> phones = entityManager.createQuery(
 "select p " +
 "from Phone p " +
 "where maxelement( p.calls ) = :call", Phone.class )
.setParameter( "call", call )
.getResultList();

//minelement
//Available for use on indexed collections. Refers to the maximum index (key/position) as determined by applying the max SQL aggregation.
List<Phone> phones = entityManager.createQuery(
 "select p " +
 "from Phone p " +
 "where minelement( p.calls ) = :call", Phone.class )
.setParameter( "call", call )
.getResultList();

//maxindex
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where maxindex( p.phones ) = 0", Person.class )
.getResultList();

// the above query can be re-written with member of
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where :phone member of p.phones", Person.class )
.setParameter( "phone", phone )
.getResultList();

//elements
//Used to refer to the elements of a collection as a whole. Only allowed in the where clause. Often used in conjunction with ALL, ANY or SOME restrictions.
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where :phone = some elements ( p.phones )", Person.class )
.setParameter( "phone", phone )
.getResultList();

List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where exists elements ( p.phones )", Person.class )
.getResultList();

List<Phone> phones = entityManager.createQuery(
 "select p " +
 "from Phone p " +
 "where current_date() > key( p.callHistory )", Phone.class )
.getResultList();

List<Phone> phones = entityManager.createQuery(
 "select p " +
 "from Phone p " +
 "where current_date() > all elements( p.repairTimestamps )", Phone.class )
.getResultList();

//indices
//
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where 1 in indices( p.phones )", Person.class )
.getResultList();

//size
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where size( p.phones ) = 2", Person.class )
.getResultList();

```

Elements of indexed collections (arrays, lists, and maps) can be referred to by index operator.

```java
// indexed lists
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where p.phones[ 0 ].type = 'LAND_LINE'", Person.class )
.getResultList();

// maps
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where p.addresses[ 'HOME' ] = :address", Person.class )
.setParameter( "address", address)
.getResultList();

//max index in list
List<Person> persons = entityManager.createQuery(
 "select pr " +
 "from Person pr " +
 "where pr.phones[ maxindex(pr.phones) ].type = 'LAND_LINE'", Person.class )
.getResultList();
```

#### 实体类型

实体类型作为表达式，在实体继承层次中有用。

```java
List<Payment> payments = entityManager.createQuery(
 "select p " +
 "from Payment p " +
 "where type(p) = CreditCardPayment", Payment.class )
.getResultList();
List<Payment> payments = entityManager.createQuery(
 "select p " +
 "from Payment p " +
 "where type(p) = :type", Payment.class )
.setParameter( "type", WireTransferPayment.class)
.getResultList();
```

### CASE表达式

#### 简单形式

CASE表达式格式: `CASE {operand} WHEN {test_value} THEN {match_result} ELSE {miss_result} END`.

```java
List<String> nickNames = entityManager.createQuery(
 "select " +
 " case p.nickName " +
 " when 'NA' " +
 " then '<no nick name>' " +
 " else p.nickName " +
 " end " +
 "from Person p", String.class )
.getResultList();

// same as above
List<String> nickNames = entityManager.createQuery(
 "select coalesce(p.nickName, '<no nick name>') " +
 "from Person p", String.class )
.getResultList();
```

#### 搜索形式

CASE表达式格式：`CASE [ WHEN {test_conditional} THEN {match_result} ]* ELSE {miss_result} END`

```java
List<String> nickNames = entityManager.createQuery(
 "select " +
 " case " +
 " when p.nickName is null " +
 " then " +
 "  case " +
 "  when p.name is null " +
 "  then '<no nick name>' " +
 "  else p.name " +
 "  end" +
 " else p.nickName " +
 " end " +
 "from Person p", String.class )
.getResultList();

// coalesce can handle this more succinctly
List<String> nickNames = entityManager.createQuery(
 "select coalesce( p.nickName, p.name, '<no nick name>' ) " +
 "from Person p", String.class )
.getResultList();
```

#### CASE表达式中有算术运算

Without wrapping the arithmetic expression in ( and ), the entity query parser will not be able to parse the arithmetic operators.

```java
List<Long> values = entityManager.createQuery(
 "select " +
 " case when p.nickName is null " +
 "   then (p.id * 1000) " +
 "   else p.id " +
 " end " +
 "from Person p " +
 "order by p.id", Long.class)
.getResultList();

assertEquals(3, values.size());
assertEquals( 1L, (long) values.get( 0 ) );
assertEquals( 2000, (long) values.get( 1 ) );
assertEquals( 3000, (long) values.get( 2 ) );
```

##### NULLIF表达式

NULLIF表达式是表达式相等返回NUll的CASE表达式的简写：

```java
List<String> nickNames = entityManager.createQuery(
 "select nullif( p.nickName, p.name ) " +
 "from Person p", String.class )
.getResultList();

// equivalent CASE expression
List<String> nickNames = entityManager.createQuery(
 "select " +
 " case" +
 " when p.nickName = p.name" +
 " then null" +
 " else p.nickName" +
 " end " +
 "from Person p", String.class )
.getResultList();
```

##### coalesce函数

coalesce()是返回第一个非空操作数的CASE语句的简写, 例子见[coalesce用法](#搜索形式)

### 动态初始化/构造器表达式

JPQL/HQL都支持该特性。
>The projection class must be fully qualified in the entity query, and it must define a matching constructor. The class here need not be mapped. It can be a DTO class. If it does represent an entity, the resulting instances are returned in the NEW state (not managed!).

#### JPQL的构造器表达式

```java
public class CallStatistics {

    private final long count;
    private final long total;
    private final int min;
    private final int max;
    private final double avg;

    public CallStatistics(long count, long total, int min, int max, double avg) {
        this.count = count;
        this.total = total;
        this.min = min;
        this.max = max;
        this.avg = avg;
    }

    //Getters and setters omitted for brevity
}

CallStatistics callStatistics = entityManager.createQuery(
 "select new org.hibernate.userguide.hql.CallStatistics(" +
 " count(c), " +
 " sum(c.duration), " +
 " min(c.duration), " +
 " max(c.duration), " +
 " avg(c.duration)" +
 ")  " +
 "from Call c ", CallStatistics.class )
.getSingleResult();
```

### 谓语

where，having从句由谓语组成。

- In
- Exists
- Empty
- Empty collection
- Member-of collection

```java
List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where 'Home address' member of p.addresses", Person.class )
.getResultList();

List<Person> persons = entityManager.createQuery(
 "select p " +
 "from Person p " +
 "where 'Home address' not member of p.addresses", Person.class )
.getResultList();
```

### Group by

```java
Long totalDuration = entityManager.createQuery(
 "select sum( c.duration ) " +
 "from Call c ", Long.class )
.getSingleResult();

List<Object[]> personTotalCallDurations = entityManager.createQuery(
 "select p.name, sum( c.duration ) " +
 "from Call c " +
 "join c.phone ph " +
 "join ph.person p " +
 "group by p.name", Object[].class )
.getResultList();

//It's even possible to group by entities!
List<Object[]> personTotalCallDurations = entityManager.createQuery(
 "select p, sum( c.duration ) " +
 "from Call c " +
 "join c.phone ph " +
 "join ph.person p " +
 "group by p", Object[].class )
.getResultList();
```

The HAVING clause follows the same rules as the WHERE clause and is also made up of predicates. HAVING is applied after the groupings and aggregations have been done, while the WHERE clause is applied before.

```java
List<Object[]> personTotalCallDurations = entityManager.createQuery(
 "select p.name, sum( c.duration ) " +
 "from Call c " +
 "join c.phone ph " +
 "join ph.person p " +
 "group by p.name " +
 "having sum( c.duration ) > 1000", Object[].class )
.getResultList();
```

### Order by

>Additionally, ***JPQL says that all values referenced in the ORDER BY clause must be named in the SELECT clause***. HQL does not mandate that restriction, but applications desiring database portability should be aware that not all databases support referencing values in the ORDER BY clause that are not referenced in the select clause. Null values can be placed in front or at the end of the sorted set using NULLS FIRST or NULLS LAST clause respectively.

The types of expressions considered valid as part of the ORDER BY clause include:

- state fields
- component/embeddable attributes
- scalar expressions such as arithmetic operations, functions, etc.
- identification variable declared in the select clause for any of the previous expression types

### Read-only 实体

As explained in [entity immutability](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#entity-immutability) section, fetching entities in read-only mode is much more efficient than fetching read-write entities. Even if the entities are mutable, you can still fetch them in read-only mode, and benefit from reducing the memory footprint and speeding up the flushing process.

Read-only entities are skipped by the dirty checking mechanism as illustrated by the following example:

```java
//第一种方法
List<Call> calls = entityManager.createQuery(
 "select c " +
 "from Call c " +
 "join c.phone p " +
 "where p.number = :phoneNumber ", Call.class )
.setParameter( "phoneNumber", "123-456-7890" )
.setHint( "org.hibernate.readOnly", true )
.getResultList();

calls.forEach( c -> c.setDuration( 0 ) );
```

```SQL
SELECT c.id AS id1_5_ ,
       c.duration AS duration2_5_ ,
       c.phone_id AS phone_id4_5_ ,
       c.call_timestamp AS call_tim3_5_
FROM   phone_call c
INNER JOIN phone p ON c.phone_id = p.id
WHERE   p.phone_number = '123-456-7890'
```

As you can see, there is no SQL UPDATE being executed.
You can also pass the read-only hint to named queries using the JPA `@QueryHint` annotation. The Hibernate native API offers a `Query#setReadOnly` method, as an alternative to using a JPA query hint:

```java
//第二种方法
@NamedQuery(
    name = "get_read_only_person_by_name",
    query = "select p from Person p where name = :name",
    hints = {
        @QueryHint(
            name = "org.hibernate.readOnly",
            value = "true"
        )
    }
)

//第三种方法
List<Call> calls = entityManager.createQuery(
 "select c " +
 "from Call c " +
 "join c.phone p " +
 "where p.number = :phoneNumber ", Call.class )
.setParameter( "phoneNumber", "123-456-7890" )
.unwrap( org.hibernate.query.Query.class )
.setReadOnly( true )
.getResultList();
```

## 事务

在持久性和对象/关系映射方面，事务这个术语有许多不同但相关的含义。在大多数情况下，这些定义是一致的，但情况并非总是如此。

- 它可能指的是与数据库的物理事务。
- 它可能指的是与持久化上下文有关的事务的逻辑概念。
- 它可能是指工作单元的应用级别概念，正如原型模式所定义的那样。

在Java的世界里，有两种定义明确的机制来处理JDBC中的事务：JDBC本身和JTA。Hibernate使用JDBC API进行持久化。Hibernate支持这两种机制，以便与事务集成，并允许应用程序管理物理事务。

## Fetch

### Direct fetching vs. entity queries

Domain model:

```java
@Entity(name = "Department")
public static class Department {

 @Id
 private Long id;

 //Getters and setters omitted for brevity
}

@Entity(name = "Employee")
public static class Employee {

 @Id
 private Long id;

 @NaturalId
 private String username;

 @ManyToOne(fetch = FetchType.EAGER)
 private Department department;

 //Getters and setters omitted for brevity
}
```

Direct fetching: 在生成sql语句时添加`left outer join`确保关联关系被fetch eagerly.

```java
Employee employee = entityManager.find( Employee.class, 1L );
```

```sql
select
    e.id as id1_1_0_,
    e.department_id as departme3_1_0_,
    e.username as username2_1_0_,
    d.id as id1_0_1_
from
    Employee e
left outer join
    Department d
        on e.department_id=d.id
where
    e.id = 1
```

entity queries: 由于在JPQL中未显式指定`JOIN FETCH`, Hibernate使用第二次select来代替。这是因为实体查询的获取策略不能被重写，所以Hibernate需要一个第二次select来确保在向用户返回结果之前保证EAGER关联。

```java
Employee employee = entityManager.createQuery(
  "select e " +
  "from Employee e " +
  "where e.id = :id", Employee.class)
.setParameter( "id", 1L )
.getSingleResult();
```

```sql
select
    e.id as id1_1_,
    e.department_id as departme3_1_,
    e.username as username2_1_
from
    Employee e
where
    e.id = 1

select
    d.id as id1_0_0_
from
    Department d
where
    d.id = 1
```

### Fetch策略

JPA定义@OneToOne和@ManyToOne关系默认情况下是`fetch eagerly`, 集合属性都是`fetch lazily`. Hibernate的建议是静态地标记所有的关联为LAZY，并使用动态获取策略来获取EAGER。

### 动态fetch策略

```java
Employee employee = entityManager.createQuery(
 "select e " +
 "from Employee e " +
 "left join fetch e.projects " +
 "where " +
 " e.username = :username and " +
 " e.password = :password",
 Employee.class)
.setParameter( "username", username)
.setParameter( "password", password)
.getSingleResult();
```

### JPA entity graph策略

//TODO entity graph

使用entity graph更好的控制fetch策略

- fetch graph
In this case, all attributes specified in the entity graph will be treated as `FetchType.EAGER`, and all attributes not specified will ALWAYS be treated as `FetchType.LAZY`.

```java
@Entity(name = "Employee")
@NamedEntityGraph(name = "employee.projects",
 attributeNodes = @NamedAttributeNode("projects")
)


Employee employee = entityManager.find(
 Employee.class,
 userId,
 Collections.singletonMap(
  "javax.persistence.fetchgraph",
  entityManager.getEntityGraph( "employee.projects" )
 )
);
```

An EntityGraph is the root of a "load plan" and must correspond to an EntityType.

- load graph
In this case, all attributes specified in the entity graph will be treated as `FetchType.EAGER`, but attributes not specified use their static mapping specification.

## 批处理

### EntityManager级别的批处理

周期性的刷新和清理一级缓存：

#### 批插入

```java
EntityManager entityManager = null;
EntityTransaction txn = null;
try {
 entityManager = entityManagerFactory().createEntityManager();

 txn = entityManager.getTransaction();
 txn.begin();

 int batchSize = 25;

 for ( int i = 0; i < entityCount; i++ ) {
  if ( i > 0 && i % batchSize == 0 ) {
   //flush a batch of inserts and release memory
   entityManager.flush();
   entityManager.clear();
  }

  Person Person = new Person( String.format( "Person %d", i ) );
  entityManager.persist( Person );
 }

 txn.commit();
} catch (RuntimeException e) {
 if ( txn != null && txn.isActive()) txn.rollback();
  throw e;
} finally {
 if (entityManager != null) {
  entityManager.close();
 }
}
```

#### 滚动结果集

```java
EntityManager entityManager = null;
EntityTransaction txn = null;
ScrollableResults scrollableResults = null;
try {
 entityManager = entityManagerFactory().createEntityManager();

 txn = entityManager.getTransaction();
 txn.begin();

 int batchSize = 25;

 Session session = entityManager.unwrap( Session.class );

 scrollableResults = session
  .createQuery( "select p from Person p" )
  .setCacheMode( CacheMode.IGNORE )
  .scroll( ScrollMode.FORWARD_ONLY );

 int count = 0;
 while ( scrollableResults.next() ) {
  Person Person = (Person) scrollableResults.get( 0 );
  processPerson(Person);
  if ( ++count % batchSize == 0 ) {
   //flush a batch of updates and release memory:
   entityManager.flush();
   entityManager.clear();
  }
 }

 txn.commit();
} catch (RuntimeException e) {
 if ( txn != null && txn.isActive()) txn.rollback();
  throw e;
} finally {
 if (scrollableResults != null) {
  scrollableResults.close();
 }
 if (entityManager != null) {
  entityManager.close();
 }
}
```
