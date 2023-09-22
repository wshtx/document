# java特性

## java8特性

>[github_java8-tutorial](https://github.com/winterbe/java8-tutorial)

### Lambda表达式

### Stream API

>[Java 8 Stream Tutorial](https://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/)  
>[Stream Javadoc](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)  
>[Stream 操作](https://zhuanlan.zhihu.com/p/299064490?utm_source=wechat_timeline)

Stream的特性：

- stream不存储数据，而是按照特定的规则对数据进行计算，一般会输出结果。
- stream不会改变数据源，通常情况下会产生一个新的集合或一个值。
- stream具有延迟执行特性，只有调用终端操作时，中间操作才会执行。

Stream operations are either intermediate or terminal. Intermediate operations return a stream so we can chain multiple intermediate operations without using semicolons. Terminal operations are either void or return a non-stream result.

Terminal operations:

- foreach
- reduce
- collect
  - collect主要依赖java.util.stream.Collectors类内置的静态方法
  - IDEA中可以设置动态模板

```java
// Abbreviation: .toList
.collect(Collectors.toList())

// Abbreviation: .toSet
.collect(Collectors.toSet())

// Abbreviation: .join
.collect(Collectors.joining("$END$"))

// Abbreviation: .groupBy
.collect(Collectors.groupingBy(e -> $END$))
```

- count
- findAny/findFirst
- max
- min
- toArray

#### 并行流/串行流

### Optional容器类

`Optional`是为了清晰地表达返回值中没有结果的可能性，不要滥用`Optional`。编码时遵循以下规范：Optional应只用于返回值，不应用于属性和参数

### 时间和日期API

### 重复式注解

声明重复式注解：

```java
@interface Hints {
    Hint[] value();
}

@Repeatable(Hints.class)
@interface Hint {
    String value();
}

//using the annotation(old shcool)
@Hints({@Hint("hint1"), @Hint("hint2")})
class Person {}

//using the annotation(new shcool)
@Hint("hint1")
@Hint("hint2")
class Person {}
```

反射获取重复式注解信息：
>Although we never declared the @Hints annotation on the Person class, it's still readable via getAnnotation(Hints.class). However, the more convenient method is getAnnotationsByType which grants direct access to all annotated @Hint annotations.

```java
Hint hint = Person.class.getAnnotation(Hint.class);
System.out.println(hint);                   // null

Hints hints1 = Person.class.getAnnotation(Hints.class);
System.out.println(hints1.value().length);  // 2

Hint[] hints2 = Person.class.getAnnotationsByType(Hint.class);
System.out.println(hints2.length);   // 2
```
