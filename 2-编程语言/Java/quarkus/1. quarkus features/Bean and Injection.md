# Contexts and Dependency Injection(CDI)

- [Contexts and Dependency Injection(CDI)](#contexts-and-dependency-injectioncdi)
  - [Concepts](#concepts)
    - [IOC and CDI](#ioc-and-cdi)
    - [What is a bean ?](#what-is-a-bean-)
    - [What does "container-managed" mean ?](#what-does-container-managed-mean-)
    - [What is it good for ?](#what-is-it-good-for-)
  - [Bean](#bean)
    - [Stereotypes](#stereotypes)
    - [Bean scope](#bean-scope)
      - [Default bean scope](#default-bean-scope)
    - [Default bean discovery mode](#default-bean-discovery-mode)
    - [Bean types](#bean-types)
      - [Restrict the bean types of a bean](#restrict-the-bean-types-of-a-bean)
    - [Kinds of Beans](#kinds-of-beans)
      - [managed bean](#managed-bean)
      - [Producer method](#producer-method)
        - [Bean types of a producer method](#bean-types-of-a-producer-method)
      - [Producer field](#producer-field)
      - [Disposer method](#disposer-method)
    - [Bean instantiation](#bean-instantiation)
      - [Injected Fields](#injected-fields)
      - [Bean Constructor](#bean-constructor)
      - [Initializer method](#initializer-method)
      - [Eager instantiation of Beans](#eager-instantiation-of-beans)
        - [Lazy of Default](#lazy-of-default)
        - [Startup Event](#startup-event)
    - [Inheritance](#inheritance)
  - [Qualifier](#qualifier)
    - [Define new qualifier](#define-new-qualifier)
    - [Bean Qualifier](#bean-qualifier)
      - [Default Bean qualifiers](#default-bean-qualifiers)
      - [Declare the qualifiers of a bean](#declare-the-qualifiers-of-a-bean)
    - [Qualifiers of Injection point](#qualifiers-of-injection-point)
  - [Dependency Injection and lookup](#dependency-injection-and-lookup)
    - [Injection point](#injection-point)
    - [Type solution](#type-solution)
      - [Assignability of raw and parameterized types](#assignability-of-raw-and-parameterized-types)
        - [parameterized bean type to raw required type](#parameterized-bean-type-to-raw-required-type)
        - [parameterized bean type to parameterized required type](#parameterized-bean-type-to-parameterized-required-type)
      - [What happens if multiple beans declare the same type ?](#what-happens-if-multiple-beans-declare-the-same-type-)
    - [Client proxies](#client-proxies)
    - [Depedency injection](#depedency-injection)
      - [Destruction of dependent objects](#destruction-of-dependent-objects)
    - [Programmatic lookup](#programmatic-lookup)
      - [Using AnnotationLiteral and TypeLiteral](#using-annotationliteral-and-typeliteral)
  - [Scope and contexts](#scope-and-contexts)
  - [Lifestyle callback](#lifestyle-callback)

## Concepts

### IOC and CDI

You’ve probably heard of the inversion of control (IoC) programming principle. Dependency injection is one of the implementation techniques of IoC.

### What is a bean ?

A bean is a container-managed obejct that supports a set of services, such as injection of dependencies, lifestyle callbacks and interceptors.

A bean comprises the following attributes:

- A (nonempty) set of bean types
- A (nonempty) set of qualifiers
- A scope
- Optionally, a bean name
- A set of interceptor bindings
- A bean implementation

### What does "container-managed" mean ?

Simply put, you don’t control the lifecycle of the object instance directly. Instead, **you can affect the lifecycle through declarative means, such as annotations, configuration**, etc. **The container is the environment where your application runs.** It creates and destroys the instances of beans, associates the instances with a designated context, and injects them into other beans.

### What is it good for ?

An application developer can focus on the business logic rather than finding out "where and how" to obtain a fully initialized component with all of its dependencies.

## Bean

### Stereotypes

### Bean scope

>[Bean scopes](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#defining_new_scope_type)  
>[Which one should I choose for my Quarkus application between @ApplicationScoped and @Singleton ?](https://quarkus.io/guides/cdi#applicationscoped-and-singleton-look-very-similar-which-one-should-i-choose-for-my-quarkus-application)
>[client proxies](https://jakarta.ee/specifications/cdi/2.0/cdi-spec-2.0.html#client_proxies)

#### Default bean scope

The default scope for a bean which does not explicitly declare a scope depends upon its declared stereotypes:

- If the bean does not declare any stereotype with a declared default scope, the default scope for the bean is `@Dependent`.
- If all stereotypes declared by the bean that have some declared default scope have the same default scope, then that scope is the default scope for the bean.
- If there are two different stereotypes present on the bean, directly, indirectly, or transitively, that declare different default scopes, then there is no default scope and the bean must explicitly declare a scope. If it does not explicitly declare a scope, the container automatically detects the problem and treats it as a definition error.

If a bean explicitly declares a scope, any default scopes declared by stereotypes are ignored.

### Default bean discovery mode

The default bean discovery mode for a bean archive is annotated. If the bean discovery mode is annotated then:

- bean classes that don’t have bean defining annotation (as defined in Bean defining annotations) are not discovered, and
- producer methods (as defined in Producer methods) whose bean class does not have a bean defining annotation are not discovered, and
- producer fields (as defined in Producer fields) whose bean class does not have a bean defining annotation are not discovered, and
- disposer methods (as defined in Disposer methods) whose bean class does not have a bean defining annotation are not discovered, and
- observer methods (as defined in Declaring an observer method) whose bean class does not have a bean defining annotation are not discovered.

### Bean types

>[Legal bean types](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#legal_bean_types)
>[Bean types of managed bean](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#managed_bean_types)
>[Bean types of a producer method](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#producer_method_types)
>[Bean types of a producer field](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#producer_field_types)

All beans have the bean type `java.lang.Object`.

```java
public class BookShop
        extends Business
        implements Shop<Book> {
    ...
}
```

The bean types are `BookShop`, `Business`, `Shop<Book>` and `Object`.

#### Restrict the bean types of a bean

The bean types of a bean may be restricted by annotating the bean class or producer method or field with the annotation `@jakarta.enterprise.inject.Typed`.

```java
@Typed(Shop.class)
public class BookShop
        extends Business
        implements Shop<Book> {
    ...
}
```

When a `@Typed` annotation is explicitly specified, only the types whose classes are explicitly listed using the value member, together with `java.lang.Object`, are bean types of the bean. And **the value member specifies a class which does not correspond to a type in the unrestricted set of bean types of a bean**.

### Kinds of Beans

1. managed class beans
2. [Producer methods](https://docs.jboss.org/weld/reference/latest/en-US/html/producermethods.html)
3. Producer fields
4. Synthetic beans

#### managed bean

>[Managed beans](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#what_classes_are_beans)

If a managed bean has a non-static public field or the managed bean class is a generic type, it must have scope `@Dependent`.

A managed bean with a constructor that takes no parameters does not require any special annotations.If the managed bean does not have a constructor that takes no parameters, it must have a constructor annotated `@Inject`. No additional special annotations are required.

#### Producer method

A producer method acts as a source of objects to be injected with the `@jakarta.enterprise.inject.Produces` annotation, where:

- the objects to be injected are not required to be instances of beans, or
- the concrete type of the objects to be injected may vary at runtime, or
- the objects require some custom initialization that is not performed by the bean constructor.

And, there are some restriction using the producer method, as below:

>[some restrictions of a producer method](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#declaring_producer_method)  
>[some restrictions of a producer method](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#producer_method)

A producer method may have any number of parameters. All producer method parameters are injection points.

##### Bean types of a producer method

The bean types of a producer method depend upon the method return type(没有限制，按照常理判断即可):

- If the return type is an interface, the unrestricted set of bean types contains the return type, all interfaces it extends directly or indirectly and `java.lang.Object`.
- If a return type is primitive or is a Java array type, the unrestricted set of bean types contains exactly two types: the method return type and `java.lang.Object`.
- If the return type is a class, the unrestricted set of bean types contains the return type, every superclass and all interfaces it implements directly or indirectly.

#### Producer field

A producer field is a slightly simpler alternative to a producer method with the `@jakarta.enterprise.inject.Produces` annotation.

All restrictions about producer field usage are same as using producer method.

#### Disposer method

A disposer method allows the application to perform customized cleanup of an object returned by a producer method or producer field. A bean may declare multiple disposer methods.

Each disposer method must have exactly one disposed parameter, of the same type as the corresponding producer method return type or producer field type.

A disposer method may be declared by annotating a parameter `@jakarta.enterprise.inject.Disposes`.

[some restrictions about disposer method](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#declaring_disposer_method)

### Bean instantiation

#### Injected Fields

An injected field may be declared by annotating the field `@jakarta.inject.Inject`.

If an injected field is annotated `@Produces`, the container automatically detects the problem and treats it as a definition error.

#### Bean Constructor

The bean constructor may be identified by annotating the constructor `@Inject`. A bean constructor may have any number of parameters. All parameters of a bean constructor are injection points.

```java
@SessionScoped
public class ShoppingCart implements Serializable {

   private User customer;

   @Inject
   public ShoppingCart(User customer) {
       this.customer = customer;
   }

   public ShoppingCart(ShoppingCart original) {
       this.customer = original.customer;
   }

   ShoppingCart() {}

   ...

}
```

There are some restrictions on the usage of bean constructor:

- If a bean class has more than one constructor annotated `@Inject`, the container automatically detects the problem and treats it as a definition error.
- If a bean class does not explicitly declare a constructor using `@Inject`, the constructor that accepts no parameters is the bean constructor.
- If a bean constructor has a parameter annotated `@Disposes`, `@Observes`, or `@ObservesAsync`, the container automatically detects the problem and treats it as a definition error.

#### Initializer method

A bean class may declare multiple (or zero) initializer methods by annotating the method `@jakarta.inject.Inject`. Method interceptors are never called when the container calls an initializer method.

```java
@ConversationScoped
public class Order {

   private Product product;
   private User customer;

   @Inject
   void setProduct(@Selected Product product) {
       this.product = product;
   }

   @Inject
   public void setCustomer(User customer) {
       this.customer = customer;
   }

}
```

#### Eager instantiation of Beans

##### Lazy of Default

By default, CDI beans are created lazily, when needed. What exactly "needed" means depends on the scope of a bean.

- A normal scoped bean (@ApplicationScoped, @RequestScoped, etc.) is needed when a method is invoked upon an injected instance (contextual reference per the specification).
In other words, injecting a normal scoped bean will not suffice because a client proxy is injected instead of a contextual instance of the bean.

- A bean with a pseudo-scope (@Dependent and @Singleton ) is created when injected.

```java
@Singleton // => pseudo-scope
class AmazingService {
  String ping() {
    return "amazing";
  }
}

@ApplicationScoped // => normal scope
class CoolService {
  String ping() {
    return "cool";
  }
}

@Path("/ping")
public class PingResource {

  //Injection triggers the instantiation of AmazingService.
  @Inject
  AmazingService s1; 

  //Injection itself does not result in the instantiation of CoolService. A client proxy is injected.
  @Inject 
  CoolService s2; 

  @GET
  public String ping() {
    //The first invocation upon the injected proxy triggers the instantiation of CoolService.
    return s1.ping() + s2.ping(); 
  }
}
```

##### Startup Event

Declare an observer of the StartupEvent - the scope of the bean does not matter in this case:

```java
@ApplicationScoped
class CoolService {
  void startup(@Observes StartupEvent event) {//A CoolService is created during startup to service the observer method invocation.

  }
}
```

### Inheritance

>[Inheritance](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#inheritance)

Inheritance of type-level metadata by beans from their superclasses is controlled via use of the Java `@Inherited` meta-annotation. Type-level metadata is never inherited from interfaces implemented by a bean.

## Qualifier

### Define new qualifier

A qualifier type is a Java annotation defined as `@Retention`(RUNTIME) and annotated with the `@javax.inject.Qualifier` meta-annotation:

```java
@Qualifier
@Retention(RUNTIME)
@Target({METHOD, FIELD, PARAMETER, TYPE})
public @interface Superior {}
```

A qualifier type may define annotation members.

```java
@Qualifier
@Retention(RUNTIME)
@Target({METHOD, FIELD, PARAMETER, TYPE})
public @interface PayBy {
    PaymentMethod value();
}
```

### Bean Qualifier

>[Qualifiers overview](https://jakarta.ee/specifications/cdi/2.0/cdi-spec-2.0.html#qualifiers)  
>[How to matching a bean](#type-solution)

**Qualifiers are annotations that help the container to distinguish beans that implement the same type**.

#### Default Bean qualifiers

- Every bean has the built-in qualifier `@Any`, even if it does not explicitly declare this qualifier.
- If a bean does not explicitly declare a qualifier other than `@Named` or `@Any`, the bean has exactly one additional qualifier, of type `@Default`. This is called the default qualifier.

**Rules above show whether the qualifier `@Default` exist or not depends on the presence or absence of other qualifiers**.

```java
//condition 1 ,Qualifiers: @Any, @Named, @Default
@Named("ord")
public class Order { ... }

//condition 2 ,Qualifiers: @Any @Another
 @Another
public class Order { ... }

```

#### Declare the qualifiers of a bean

The qualifiers of a bean are declared by annotating the bean class or producer method or field with the qualifier types.

```java
@Another
public class Shop {

   @Produces @All
   public List<Product> getAllProducts() { ... }

   @Produces @WishList
   public List<Product> getWishList() { ... }

}
```

### Qualifiers of Injection point

If a injection point does not explicitly declare a qualifier, the injection point has exactly one additional qualifier, of type `@Default`

## Dependency Injection and lookup

>[Type solution](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#typesafe_resolution)

### Injection point

The container injects references to contextual instances to the following kinds of injection point:

- Any injected field of a bean class
- Any parameter of a bean constructor, bean initializer method, producer method or disposer method
- Any parameter of an observer method, except for the event parameter

### Type solution

The process of matching a bean to an injection point is called typesafe resolution. The container considers bean type and qualifiers when resolving a bean to be injected to an injection point. The type and qualifiers of the injection point are called the *required type* and *required qualifiers*.
A bean is assignable to a given injection point if:

- **The bean has a bean type that matches the required type**. For this purpose, primitive types are considered to match their corresponding wrapper types in java.lang and array types are considered to match only if their element types are identical. Parameterized and raw types are considered to match if they are identical or if the bean type is assignable to the required type, as defined in [Assignability of raw and parameterized types](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#assignable_parameters).
- **The bean has all the required qualifiers**. If no required qualifiers were explicitly specified, the container assumes the required qualifier @Default.

#### Assignability of raw and parameterized types

##### parameterized bean type to raw required type

A parameterized bean type is considered assignable to a raw required type **if the raw types are identical and all type parameters of the bean type are either unbounded type variables or `java.lang.Object`.**

##### parameterized bean type to parameterized required type

A parameterized bean type is considered assignable to a parameterized required type if they have identical raw type and for each parameter:

#### What happens if multiple beans declare the same type ?

There is a simple rule: exactly one bean must be assignable to an injection point, otherwise the build fails. If none is assignable the build fails with `UnsatisfiedResolutionException`. If multiple are assignable the build fails with `AmbiguousResolutionException`.

### Client proxies

>[Client proxies](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#client_proxies)

### Depedency injection

>[Depedency injection](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#injection)

When the container creates a new instance of a managed bean, the container must sequentially:

1. all injected fields declared by X or by superclasses
2. Initializer methods declared by a class X or by superclass
3. Any @PostConstruct callback

#### Destruction of dependent objects

[Destruction of dependent objects](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#dependent_objects_destruction)

### Programmatic lookup

>[The Instance interface](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#dynamic_lookup)  
>[Using AnnotationLiteral and TypeLiteral](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#annotationliteral_typeliteral)  
>[Built-in annotation literals](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#built_in_annotation_literals)

In certain situations, injection is not the most convenient way to obtain a contextual reference. For example, it may not be used when:

- the bean type or qualifiers vary dynamically at runtime, or
- depending upon the deployment, there may be no bean which satisfies the type and qualifiers, or
- we would like to iterate over all beans of a certain type.

Your can use programmatic lookup via **javax.enterprise.inject.Instance** to resolve ambiguities at runtime and even iterate over all beans implementing a given type:

```java
public class Translator {

    @Inject
    Instance<Dictionary> dictionaries; 

    String translate(String sentence) {
      for (Dictionary dict : dictionaries) { 
         // ...
      }
    }
}
```

#### Using AnnotationLiteral and TypeLiteral

jakarta.enterprise.util.AnnotationLiteral makes it easier to specify qualifiers when calling select():

```java
public PaymentProcessor getSynchronousPaymentProcessor(PaymentMethod paymentMethod) {

    class SynchronousQualifier extends AnnotationLiteral<Synchronous>
            implements Synchronous {}

    class PayByQualifier extends AnnotationLiteral<PayBy>
            implements PayBy {
        public PaymentMethod value() { return paymentMethod; }
    }

    return anyPaymentProcessor.select(new SynchronousQualifier(), new PayByQualifier()).get();
}
```

jakarta.enterprise.util.TypeLiteral makes it easier to specify a parameterized type with actual type parameters when calling select():

```java
public PaymentProcessor<Cheque> getChequePaymentProcessor() {
    PaymentProcessor<Cheque> pp = anyPaymentProcessor
        .select( new TypeLiteral<PaymentProcessor<Cheque>>() {} ).get();
}
```

## Scope and contexts

>[Scope and contexts](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#contexts)

Associated with every scope type is a context object. The context object determines the lifecycle and visibility of instances of all beans with that scope. In particular, the context object defines:

- When a new instance of any bean with that scope is created
- When an existing instance of any bean with that scope is destroyed
- Which injected references refer to any instance of a bean with that scope

The context implementation collaborates with the container via the `Context` and `Contextual` interfaces to create and destroy contextual instances.

## Lifestyle callback

A bean class may declare lifecycle @PostConstruct and @PreDestroy callbacks:

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@ApplicationScoped
public class Translator {

    //This callback is invoked before the bean instance is put into service. It is safe to perform some initialization here.
    @PostConstruct 
    void init() {
       // ...
    }
    
    //This callback is invoked before the bean instance is destroyed. It is safe to perform some cleanup tasks here.
    @PreDestroy 
    void destroy() {
      // ...
    }
}
```

It’s a good practice to keep the logic in the callbacks "without side effects", i.e. you should avoid calling other beans inside the callbacks.
