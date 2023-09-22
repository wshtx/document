# Spring核心

[TOC]

## IOC

IOC容器支持Bean对象的完整生命周期。
`BeanFactory`定义了基本的容器功能，`ApplicationContext`基于`BeanFactory`构建，并提供面向应用的服务，例如从属性文件中解析文本信息，以及发布应用事件给感兴趣的事件监听者。因此，`ApplicationContext`比`BeanFactory`更受欢迎。

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221201094112.png)

### 容器创建过程

BeanFactory DafaultListenableBeanFactory

#### BeanFactory和FactoryBean的区别

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221129113742.png)

### bean对象的生命周期

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221129000433.png)

### bean的作用域

- singleton单例模式，默认
对象的创建，依赖注入和初始化三个阶段会在启动IOC容器时完成
- prototype原型模式，
每次获取都是新的对象，并且对象的创建，依赖注入和初始化三个阶段会在获取bean时完成；销毁阶段不由IOC容器负责。

### bean创建流程

流程图

getBean(), doGetBean(), createBean(),doCreateBean(), createBeanInstance(), populateBean()

### BeanPostProcessor后置处理器

### 依赖注入

- 构造器注入

按照配置的参数个数去寻找对应的构造器。如果存在多个参数个数相同的构造器，则可以通过name属性去指定使用具体哪个构造器(指定对应的属性名)

```xml
<bean id="" class="">
    <constructor-arg value="htx"/>
    <constructor-arg value="24" name="age"/>
</bean>
```

- setter方法注入

```xml
<bean id="" class="">
    <property name="name" value="htx"/>
</bean>
```

属性值不同时，赋值方式有不同：

- 当属性类型为基本类型时，直接使用value属性进行字面量赋值；
- 当需要将属性赋值为null时，使用null标签

```XML
<property name="name">
    <null/>
</property>
```

- 当属性值有特殊字符时，使用value标签配合<![CDATA[]]>标签,CD标签中的内容不会解析

```XML
<property name="name">
    <value><![CDATA[<123>]]></value>
</property>
```

- 为类类型(普通类，接口)的属性赋值

```XML
//第一种方式
<property name="name" ref="xxx">
</property>

//级联方式
<bean id="" class="">
    <property name="name" value="htx"/>
    <property name="" value="xxx"/>
    <property name="xxx.name" value=""/>
    <property name="xxx.age" value=""/>
</bean>

//内部bean，只能在当前bean内部使用，不能在外部通过IOC容器获取
<property name="name" value="htx">
    <bean id="" class="">
        <property name="name" value="htx"/>
    </bean>
</property>
```

- 为数组类型的属性赋值

```XML
<property name="name" value="htx">
    <bean id="" class="">
        <property name="name" value="htx">
            <array>
                <!--   字面量类型 -->
                <!-- <value>123</value> -->
                <!-- <value>asd</value> -->

                <!-- bean类型 -->
                <ref bean="" />
                <ref bean="" />
            </array>
        </property>
    </bean>
</property>
```

- 为list类型的属性赋值

```XML
<!-- 第一种方法，list标签 -->
<property name="name" value="htx">
    <bean id="" class="">
        <property name="name" value="htx">
            <list>
                <!--   字面量类型 -->
                <!-- <value>123</value> -->
                <!-- <value>asd</value> -->

                <!-- bean类型 -->
                <ref bean="" />
                <ref bean="" />
            </list>
        </property>
    </bean>
</property>

<!-- 第二种方法util约束 -->
<util.list id="listone">
    <ref bean=""/>
    <ref bean=""/>
    <ref bean=""/>
</util.list>

<property name="name" value="htx">
    <bean id="" class="">
        <property name="name" ref="listone">
        </property>
    </bean>
</property>
```

- 为map类型的属性赋值

```xml
<!-- 第一种方式 -->
<property name="name" value="htx">
    <bean id="" class="">
        <property name="name" value="htx">
            <map>
                <!-- 字面量类型 -->
                <entry key="" value=""/>
                <!-- bean类型 -->
                <entry key-ref="" value-ref=""/>
            </map>
        </property>
    </bean>
</property>


<!-- 第二种方法util约束 -->
<util.map id="listone">
   <!-- 字面量类型 -->
    <entry key="" value=""/>
    <!-- bean类型 -->
    <entry key-ref="" value-ref=""/>
</util.map>
<property name="name" value="htx">
    <bean id="" class="">
        <property name="name" ref="listone">
        </property>
    </bean>
</property>
```

- 引入别的配置文件中值

```XML
<context:property-placeholder location="jdbc.properties"/>

<bean id="">
    <property name="url" value="${jdbc.url}">
</bean>
```

### 基于xml的自动装配bean

基于xml的自动装配会将自动装配bean中的所有类类型的属性，不灵活，一般使用注解。 autowire的类型：

- no|default 不自动装配，使用默认值
- byType 通过类型找到合适的对象注入
  - 没有找到匹配的对象时，使用默认值
  - 找到多个匹配的对象时，会报异常
- byName 通过属性名找到合适的对象注入

```xml  
<bean id="" autowire="no|default|byType|byName|">
    <property name="url" value="${jdbc.url}">
</bean>
```

### 基于注解管理bean

#### 扫描组件

```xml  
<context:component-scan base-package="">
  <!-- 排除某个注解标识的bean -->
  <context:exclude-filter type="annotation" expression="Controller"/>
  <!-- 排除某个类的bean -->
  <context:exclude-filter type="assignable" expression="UserController"/>
</context:component-scan>

<!-- use-default-filters默认为true,扫描包下的所有类 -->
<context:component-scan base-package="" use-default-filters="false">
  <!-- 只扫描某个注解的bean -->
  <context:include-filter type="annotation" expression="Controller"/>
</context:component-scan>
```

#### 注册bean

bean的id默认为类名的驼峰 ，即首字母小写的驼峰格式；也可以自定义id

- @Component 将类标识为普通组件
- @Controller 将类标识为控制层组件
- @Service 将类标识为服务层组件
- @Repository 将类标识为数据访问层组件

#### 自动装配

`@Autowired`能够标识的位置：

- 标识在成员变量上，此时不需要设置成员变量的set方法
- 标识在set方法上
- 标识在为当前成员变量赋值的有参构造器上

#### @Autowired实现原理

- 默认通过byType方式，在IOC容器中通过类型匹配为某个bean属性赋值。若IOC容器中没有任何一个类型匹配的bean时，此时抛出异常NoSuchBeanDefinitionException。此时与通过xml进行自动装配的情况不一致(基于xml自动装配找不到类型匹配的bean会使用默认值)。在`@Autowired`注解中有个属性required，默认值为true，要求必须要完成自动装配；可以设置成false，此时能装配就装配 ，不能装配就是用默认值。
- 若有多个类型匹配的bean，会自动转换为byName的方式实现自动装配。
- 若byType和byName都无法实现自动装配，即IOC容器中存在多个类型匹配的bean，且这些bean的id和要赋值的属性的名称都不一致，此时抛异常：`NoUniqueBeanDefinitionException`。此时，可以在要赋值的属性上使用`NoUniqueBeanDefinitionException`的属性值指定某个bean的id，将这个bean赋值到该属性。

## 三级缓存

- 第三级缓存 `Map<String, ObjectFactory<?>> singletonFactories`
- 第二级缓存 `Map<String, Object> earlySingletonObjects`
存放已创建但未完全初始化的对象
- 第一级缓存 `Map<String, Object> singletonObjects`
存放完整对象

### 二级缓存的必要性

缓存完成所有初始化操作的完整对象，解决创建bean时的循环依赖问题。

### 三级缓存的意义

容器中对象都是单例的，意味着只能根据名称获取唯一一个对象的值，而不能存在名称相同的两个对象。因为外界调用对象的时刻无法确定，所以必须要保证容器中的任何时候都只有一个对象供外界访问，所以在第三级缓存中主要任务就是用**代理对象替换非代理对象**，确定唯一返回的对象。

三级缓存是为了解决在aop代理过程中产生的循环依赖问题，如果没有aop，二级缓存即可解决循环依赖问题。

### Spring如何解决循环依赖问题

循环依赖问题产生的原因如下图，关键是打断依赖图中的循环操作：
![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221129112825.png)

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221129113356.png)
![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221129113330.png)

### 三级缓存的放置时间和删除时间

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221129113547.png)

## AOP

### Aop要素

- 横切关注点
- 通知
  - 前置通知
  - 返回通知
  - 异常通知
  - 终止通知
  - 环绕通知
- **切面** 封装通知方法的类
- 目标 被代理的目标对象
- 代理 向目标应用通知之后创建的代理对象
- 连接点 逻辑概念，抽取横切关注点的位置，逻辑上是客观存在的。
- 切入点  代码层面概念，定义连接点的表达式

### Aop使用

- 在spring配置文件中开启基于注解的Aop

  - `<aop:aspectJ-autoproxy/>`

- 定义切面类

  - 使用`@Aspect`注解标识为切面，`@Component`标识为bean

  - 定义通知方法同时定义切入点，可以加入`JoinPoint`类型参数来获取连接点信息

    - 前置通知 `@Before`
    - 返回通知 `@AfterReturning`
    - 异常通知 `@Throwing`
    - 后置通知 `@After` 在目标对象方法的finally中执行
    - 环绕通知 `@Around` 上面四种通知的集合，**环绕通知方法的返回值必须与目标方法的返回值保持一致**

  - 通知方法的调用顺序

    ![image-20221201160756019](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/image-20221201160756019.png)

  - 切入点表达式语法

    具体到某个方法：`@Before("execution(public int com.xxx.Test.testMethod(int, int))")`

    扫描某个包的所有类的所有方法，不限制访问控制符，返回值，类，方法：`@Before("execution(* com.xxx.*.*(..))")`

  - 复用切入点表达式

    ```java
    @Pointcut("execution(* com.xxx.*.*(..)")
    public void pointCut(){}
    
    @Before("pointCut")
    public void before(){
        ...
    }
    ```

### 切面的优先级

通过`@Order`定义切面优先级，值越小，切面优先级越高。

## Spring事务

### 编程式事务

```java
Connection conn = null;

try{
    //关闭事务的自动提交，事务自动提交模式下每个sql语句都是一个事务
    conn.setAutoCommit(false);
    
    //业务逻辑
    
    //conn.commit();
}catch(Exception e){
    conn.rollback();
}finally(){
    conn.close();
}
```

### 声明式事务

声明式事务：通过配置让框架实现事务。

```xml
<!--配置事务管理器，定义了事务操作的切面-->
<bean id="txmanager" class="DatasourceTransactionManager">
    <property name="" ref=""></property>
</bean>
<!--开启事务的注解驱动，将使用@Transactional注解所标识的方法或类中的所有方法使用事务进行管理-->
<tx:annotation-driven transaction-manager="txmanager"/>
```

#### 声明式事务的属性

- 只读

   `@Transactional(readOnly=true)` 只有查询操作才能使用，增删改操作使用会有异常

- 超时

   `@Transactional(timeout=3)` 单位为秒；超时后会抛出异常并且回滚

- 回滚策略

   `@Transactional(noRollbackFor=ArithmeticException.class)` 默认回滚策略就是运行时遇到任何异常都回滚。`noRollbackFor`属性用来设置不引起回滚的异常。

- 隔离级别

   `@Transactional(isolation=Isolation.DEFAULT)` 。`Isolation.DEFAULT`属性值使用数据库的默认隔离级别

- 事务的传播性

​ 当发生异常时，开启了事务传播性的方法，会在调用者回滚所有操作；而不开启事务传播的方法，只是回滚当前事务内(当前方法内)的操作。

```java
//使用本身的事务
@Transactional(propagation=Propagation.REQUIRED_NEW)
//使用调用者的事务，此时事务由调用者传递到被调用方法
@Transactional(propagation=Propagation.REQUIRED)
```

#### 基于xml的声明式事务

![image-20221201170755912](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/image-20221201170755912.png)

### Spring事务实现

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221129115039.png)

### Spring事务传播

## Spring中的设计模式

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221129114106.png)
