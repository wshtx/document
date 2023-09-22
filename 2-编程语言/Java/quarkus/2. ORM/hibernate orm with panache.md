# Hibernat ORM with Panache

- [Hibernat ORM with Panache](#hibernat-orm-with-panache)
  - [Two Hibernate patterns](#two-hibernate-patterns)
    - [Panache Active Record way](#panache-active-record-way)
    - [Repository pattern](#repository-pattern)
  - [Transaction](#transaction)
    - [Contexts Propagation](#contexts-propagation)
      - [Simple usages](#simple-usages)
      - [Overriding which contexts are propagated](#overriding-which-contexts-are-propagated)
    - [声明式启用事务](#声明式启用事务)
      - [基本注解](#基本注解)
      - [手动控制回滚操作](#手动控制回滚操作)
    - [编程式启用事务](#编程式启用事务)
    - [flush()立即提交修改](#flush立即提交修改)
  - [Lock management](#lock-management)
  - [Advanced Query](#advanced-query)
    - [Paging](#paging)
    - [Ranging](#ranging)
    - [Sorting](#sorting)
    - [Query Parameters](#query-parameters)
    - [Query Projection](#query-projection)
  - [Other Features](#other-features)
    - [Custom IDs](#custom-ids)
    - [Mock Testing](#mock-testing)
      - [active record pattern](#active-record-pattern)
      - [the repository pattern](#the-repository-pattern)
    - [Multi persistence units](#multi-persistence-units)
    - [Caching](#caching)
      - [Caching of entities](#caching-of-entities)
      - [Caching of collections and relations](#caching-of-collections-and-relations)
      - [Caching of queries](#caching-of-queries)
      - [Tuning of Cache Regions](#tuning-of-cache-regions)
    - [Hibernate Validator/Bean Validation](#hibernate-validatorbean-validation)
      - [REST end point validation](#rest-end-point-validation)
      - [Service method validation](#service-method-validation)

>[Java Persistence API, JPA](https://docs.oracle.com/javaee/6/tutorial/doc/javaeetutorial6.pdf?p=579)  
>[Hibernate user guide](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#hql)

note: 经典Hibernate框架还是使用xml文件做配置，Hibernat ORM with Panache 提供了*repository pattern*和*active record* pattern两种模式简化持久层的编码工作。
Hibernate Reative的使用和Hibernat ORM with Panache一样，只是返回的是`Uni/Multi`，以及底层实现不同.

## Two Hibernate patterns

### Panache Active Record way

***Use the Entity class to do all the ORM operations.***

1. The entity class(`@Entity` decorated) should extend the `PanacheEntity`, and the fields of the entity should be `public`.
2. Adding a custom query-method of entity as a static method is the Panache Active Recode way.
3. Simply Use the instance or class of the entity to invoke all the operations defining on the `PanacheEntity` class.

### Repository pattern

***Use the Repository class to operate the database entity.***

1. The entity class(`@Entity`) don\`t extend any base class, and you need to bother defining `getter/setter` for your entity, with the `private` fields. But you can make them extend `PanacheEntityBase` and Quarkus will generate them for you.

    Even extend the `PanacheEntity` and take advantage of the default ID it provided.  
2. Defining your repository class making them implements `PanacheRepository`. And you can use all the operations defining on `PanacheEntityBase` or your custom method in the repository class.
3. The repository class need to inject(`@Inject`).

## Transaction

>[Using transcation In Hibernate](https://quarkus.io/guides/transaction)
>[Contexts Propagation In Quarkus](https://quarkus.io/guides/context-propagation)

### Contexts Propagation

#### Simple usages

所需依赖: `implementation("io.quarkus:quarkus-smallrye-context-propagation")`
Mutiny使用实例，该例子中事务会在上下文中被自动传递

```java
    // Get the prices stream
    @Inject
    @Channel("prices") Publisher<Double> prices;

    @Transactional
    @GET
    @Path("/prices")
    @RestStreamElementType(MediaType.TEXT_PLAIN)
    public Publisher<Double> prices() {
        // get the next three prices from the price stream
        return Multi.createFrom().publisher(prices)
                .select().first(3)
                // The items are received from the event loop, so cannot use Hibernate ORM (classic)
                // Switch to a worker thread, the transaction will be propagated
                .emitOn(Infrastructure.getDefaultExecutor())
                .map(price -> {
                    // store each price before we send them
                    Price priceEntity = new Price();
                    priceEntity.value = price;
                    // here we are all in the same transaction
                    // thanks to context propagation
                    priceEntity.persist();
                    return price;
                    // the transaction is committed once the stream completes
                });
    }
```

`CompletionStage`使用实例：

```java
    @Inject ThreadContext threadContext;
    @Inject ManagedExecutor managedExecutor;
    @Inject Vertx vertx;

    @Transactional
    @GET
    @Path("/people")
    public CompletionStage<List<Person>> people() throws SystemException {
        // Create a REST client to the Star Wars API
        WebClient client = WebClient.create(vertx,
                         new WebClientOptions()
                          .setDefaultHost("swapi.dev")
                          .setDefaultPort(443)
                          .setSsl(true));
        // get the list of Star Wars people, with context capture
        return threadContext.withContextCapture(client.get("/api/people/").send())
                .thenApplyAsync(response -> {
                    JsonObject json = response.bodyAsJsonObject();
                    List<Person> persons = new ArrayList<>(json.getInteger("count"));
                    // Store them in the DB
                    // Note that we're still in the same transaction as the outer method
                    for (Object element : json.getJsonArray("results")) {
                        Person person = new Person();
                        person.name = ((JsonObject) element).getString("name");
                        person.persist();
                        persons.add(person);
                    }
                    return persons;
                }, managedExecutor);
    }
```

#### Overriding which contexts are propagated

>[Overriding which contexts are propagated](https://quarkus.io/guides/context-propagation#overriding-which-contexts-are-propagated)

Overriding the propagated contexts using annotations:

```java
    // Get the prices stream
    @Inject
    @Channel("prices") Publisher<Double> prices;

    @GET
    @Path("/prices")
    @RestStreamElementType(MediaType.TEXT_PLAIN)
    // Get rid of all context propagation, since we don't need it here
    @CurrentThreadContext(unchanged = ThreadContext.ALL_REMAINING)
    public Publisher<Double> prices() {
        // get the next three prices from the price stream
        return Multi.createFrom().publisher(prices)
                .select().first(3);
    }
```

Overriding the propagated contexts using CDI injection

```java
    // Get the prices stream
    @Inject
    @Channel("prices") Publisher<Double> prices;
    // Get a ThreadContext that doesn't propagate context
    @Inject
    @ThreadContextConfig(unchanged = ThreadContext.ALL_REMAINING)
    SmallRyeThreadContext threadContext;

    @GET
    @Path("/prices")
    @RestStreamElementType(MediaType.TEXT_PLAIN)
    public Publisher<Double> prices() {
        // Get rid of all context propagation, since we don't need it here
        try(CleanAutoCloseable ac = SmallRyeThreadContext.withThreadContext(threadContext)){
            // get the next three prices from the price stream
            return Multi.createFrom().publisher(prices)
                    .select().first(3);
        }
    }
```

Inject a configured instance of `ManagedExecutor` using the `@ManagedExecutorConfig` annotation:

```java
    // Custom ManagedExecutor with different async limit, queue and no propagation
    @Inject
    @ManagedExecutorConfig(maxAsync = 2, maxQueued = 3, cleared = ThreadContext.ALL_REMAINING)
    ManagedExecutor configuredCustomExecutor;
```

Sharing configured CDI instances of ManagedExecutor and ThreadContext if you need to inject the same `ManagedExecutor` or `ThreadContext` into several places and share its capacity:

```java
    // Custom configured ManagedExecutor with name
    @Inject
    @ManagedExecutorConfig(maxAsync = 2, maxQueued = 3, cleared = ThreadContext.ALL_REMAINING)
    @NamedInstance("myExecutor")
    ManagedExecutor sharedConfiguredExecutor;

    // Since this executor has the same name, it will be the same instance as above
    @Inject
    @NamedInstance("myExecutor")
    ManagedExecutor sameExecutor;

    // Custom ThreadContext with a name
    @Inject
    @ThreadContextConfig(unchanged = ThreadContext.ALL_REMAINING)
    @NamedInstance("myContext")
    ThreadContext sharedConfiguredThreadContext;

    // Given equal value of @NamedInstance, this ThreadContext will be the same as the above one
    @Inject
    @NamedInstance("myContext")
    ThreadContext sameContext;
```

### 声明式启用事务

#### 基本注解

- `@Transactional/@ReactiveTransactinoal` 启用事务
- `@TransactionConfiguration` 事务配置

#### 手动控制回滚操作

```java
@ApplicationScoped
public class SantaClausService {

    @Inject TransactionManager tm; 
    @Inject ChildDAO childDAO;
    @Inject SantaClausDAO santaDAO;

    @Transactional
    public void getAGiftFromSanta(Child child, String giftDescription) {
        // some transaction work
        Gift gift = childDAO.addToGiftList(child, giftDescription);
        if (gift == null) {
            tm.setRollbackOnly(); 
        }
        else {
            santaDAO.addToSantaTodoList(gift);
        }
    }
}
```

### 编程式启用事务

```java
import io.quarkus.narayana.jta.QuarkusTransaction;
import io.quarkus.narayana.jta.RunOptions;

public class TransactionExample {

    public void beginExample() {
        QuarkusTransaction.begin();
        //do work
        QuarkusTransaction.commit();

        QuarkusTransaction.begin(QuarkusTransaction.beginOptions()
                .timeout(10));
        //do work
        QuarkusTransaction.rollback();
    }

    public void lambdaExample() {
        QuarkusTransaction.run(() -> {
            //do work
        });


        int result = QuarkusTransaction.call(QuarkusTransaction.runOptions()
                .timeout(10)
                .exceptionHandler((throwable) -> {
                    if (throwable instanceof SomeException) {
                        return RunOptions.ExceptionResult.COMMIT;
                    }
                    return RunOptions.ExceptionResult.ROLLBACK;
                })
                .semantic(RunOptions.Semantic.SUSPEND_EXISTING), () -> {
            //do work
            return 0;
        });
    }
}
```

### flush()立即提交修改

- JPA batches changes you make to your entities and sends changes (it is called flush) at the end of the transaction or before a query. This is usually a good thing as it is more efficient. But if you want to check optimistic locking failures, do object validation right away or generally want to get immediate feedback, you can force the flush operation by calling `entity.flush()` or even use `entity.persistAndFlush()` to make it a single method call. This will allow you to catch any `PersistenceException` that could occur when JPA send those changes to the database. Remember, *this is less efficient so don’t abuse it*. And your transaction still has to be committed.

## Lock management

Panache provides direct support for database locking with your entity/repository, using `findById(Object, LockModeType)` or `find().withLock(LockModeType)`.

```java
public class PersonEndpoint {

    @GET
    @Transactional
    public Person findByIdForUpdate(Long id){
        Person p = Person.findById(id, LockModeType.PESSIMISTIC_WRITE);
        //do something useful, the lock will be released when the transaction ends.
        return person;
    }

    @GET
    @Transactional
    public Person findByNameForUpdate(String name){
        Person p = Person.find("name", name).withLock(LockModeType.PESSIMISTIC_WRITE).findOne();
        //do something useful, the lock will be released when the transaction ends.
        return person;
    }

}
```

***The method that invokes the lock query must be annotated with the @Transactional annotation.***

## Advanced Query

### Paging

You should only use list and stream methods if your table contains small enough data sets. For larger data sets you can use the find method equivalents, which return a `PanacheQuery` on which you can do paging:

```java
// create a query for all living persons
PanacheQuery<Person> livingPersons = Person.find("status", Status.Alive);

// make it use pages of 25 entries at a time
livingPersons.page(Page.ofSize(25));

// get the first page
List<Person> firstPage = livingPersons.list();

// get the second page
List<Person> secondPage = livingPersons.nextPage().list();

// get page 7
List<Person> page7 = livingPersons.page(Page.of(7, 25)).list();

// get the number of pages
int numberOfPages = livingPersons.pageCount();

// get the total number of entities returned by this query without paging
long count = livingPersons.count();

// and you can chain methods of course
return Person.find("status", Status.Alive)
    .page(Page.ofSize(25))
    .nextPage()
    .stream()
```

The PanacheQuery type has many other methods to deal with paging and returning streams.

### Ranging

```java
// create a query for all living persons
PanacheQuery<Person> livingPersons = Person.find("status", Status.Alive);

// make it use a range: start at index 0 until index 24 (inclusive).
livingPersons.range(0, 24);

// get the range
List<Person> firstRange = livingPersons.list();

// to get the next range, you need to call range again
List<Person> secondRange = livingPersons.range(25, 49).list();
```

### Sorting

All methods accepting a query string also accept the following simplified query form. But these methods also accept an optional Sort parameter, which allows you to abstract your sorting:

```java
List<Person> persons = Person.list("order by name,birth");

List<Person> persons = Person.list(Sort.by("name").and("birth"));

// and with more restrictions
List<Person> persons = Person.list("status", Sort.by("name").and("birth"), Status.Alive);

// and list first the entries with null values in the field "birth"
List<Person> persons = Person.list(Sort.by("birth", Sort.NullPrecedence.NULLS_FIRST));
```

### Query Parameters

Every query operation accepts passing parameters by index (Object…​), or by name (`Map<String,Object>` or `Parameters`).

1.You can pass query parameters by index (1-based) as shown below:

```java
Person.find("name = ?1 and status = ?2", "stef", Status.Alive);
```

2.Or by name using a Map:

```java
Map<String, Object> params = new HashMap<>();
params.put("name", "stef");
params.put("status", Status.Alive);
Person.find("name = :name and status = :status", params);
```

3.Or using the convenience class Parameters either as is or to build a Map:

```java
// generate a Map
Person.find("name = :name and status = :status",
         Parameters.with("name", "stef").and("status", Status.Alive).map());

// use it as-is
Person.find("name = :name and status = :status",
         Parameters.with("name", "stef").and("status", Status.Alive));
```

### Query Projection

数据库查询返回的对象一般是Entity对象，如果不想暴露Entity对象，则创建DTO对象，将Entity对象映射到DTO对象(called *dynamic instantiation* or *constructor expression*)。

```java
import io.quarkus.runtime.annotations.RegisterForReflection;

//DTO obejct
@RegisterForReflection 
public class PersonName {
    public final String name; 

    public PersonName(String name){ 
        this.name = name;
    }
}

// only 'name' will be loaded from the database
PanacheQuery<PersonName> query = Person.find("status", Status.Alive).project(PersonName.class);
```

## Other Features

### Custom IDs

### Mock Testing

If you are using Panache Active pattern, you can\`t using Mockito directly, but you can use `quarkus-panache-mock` module.
But the repository pattern instead, which can use `quarkus-junit5-mockito` directly.

#### active record pattern

Mock Entity static method:

```java
@QuarkusTest
public class PanacheFunctionalityTest {

    @Test
    public void testPanacheMocking() {
        PanacheMock.mock(Person.class);

        // Mocked classes always return a default value
        Assertions.assertEquals(0, Person.count());

        // Now let's specify the return value
        Mockito.when(Person.count()).thenReturn(23L);
        Assertions.assertEquals(23, Person.count());

        // Now let's change the return value
        Mockito.when(Person.count()).thenReturn(42L);
        Assertions.assertEquals(42, Person.count());

        // Now let's call the original method
        Mockito.when(Person.count()).thenCallRealMethod();
        Assertions.assertEquals(0, Person.count());

        // Check that we called it 4 times
        PanacheMock.verify(Person.class, Mockito.times(4)).count();

        // Mock only with specific parameters
        Person p = new Person();
        Mockito.when(Person.findById(12L)).thenReturn(p);
        Assertions.assertSame(p, Person.findById(12L));
        Assertions.assertNull(Person.findById(42L));

        // Mock throwing
        Mockito.when(Person.findById(12L)).thenThrow(new WebApplicationException());
        Assertions.assertThrows(WebApplicationException.class, () -> Person.findById(12L));

        // We can even mock your custom methods
        Mockito.when(Person.findOrdered()).thenReturn(Collections.emptyList());
        Assertions.assertTrue(Person.findOrdered().isEmpty());

        // Mocking a void method
        Person.voidMethod();

        // Make it throw
        PanacheMock.doThrow(new RuntimeException("Stef2")).when(Person.class).voidMethod();
        try {
            Person.voidMethod();
            Assertions.fail();
        } catch (RuntimeException x) {
            Assertions.assertEquals("Stef2", x.getMessage());
        }

        // Back to doNothing
        PanacheMock.doNothing().when(Person.class).voidMethod();
        Person.voidMethod();

        // Make it call the real method
        PanacheMock.doCallRealMethod().when(Person.class).voidMethod();
        try {
            Person.voidMethod();
            Assertions.fail();
        } catch (RuntimeException x) {
            Assertions.assertEquals("void", x.getMessage());
        }

        PanacheMock.verify(Person.class).findOrdered();
        PanacheMock.verify(Person.class, Mockito.atLeast(4)).voidMethod();
        PanacheMock.verify(Person.class, Mockito.atLeastOnce()).findById(Mockito.any());
        PanacheMock.verifyNoMoreInteractions(Person.class);
    }
}
```

//ToDO
Mocking EntityManager, Session and entity instance methods:

#### the repository pattern

```java
@QuarkusTest
public class PanacheFunctionalityTest {
    @InjectMock
    PersonRepository personRepository;

    @Test
    public void testPanacheRepositoryMocking() throws Throwable {
        // Mocked classes always return a default value
        Assertions.assertEquals(0, personRepository.count());

        // Now let's specify the return value
        Mockito.when(personRepository.count()).thenReturn(23L);
        Assertions.assertEquals(23, personRepository.count());

        // Now let's change the return value
        Mockito.when(personRepository.count()).thenReturn(42L);
        Assertions.assertEquals(42, personRepository.count());

        // Now let's call the original method
        Mockito.when(personRepository.count()).thenCallRealMethod();
        Assertions.assertEquals(0, personRepository.count());

        // Check that we called it 4 times
        Mockito.verify(personRepository, Mockito.times(4)).count();

        // Mock only with specific parameters
        Person p = new Person();
        Mockito.when(personRepository.findById(12L)).thenReturn(p);
        Assertions.assertSame(p, personRepository.findById(12L));
        Assertions.assertNull(personRepository.findById(42L));

        // Mock throwing
        Mockito.when(personRepository.findById(12L)).thenThrow(new WebApplicationException());
        Assertions.assertThrows(WebApplicationException.class, () -> personRepository.findById(12L));

        Mockito.when(personRepository.findOrdered()).thenReturn(Collections.emptyList());
        Assertions.assertTrue(personRepository.findOrdered().isEmpty());

        // We can even mock your custom methods
        Mockito.verify(personRepository).findOrdered();
        Mockito.verify(personRepository, Mockito.atLeastOnce()).findById(Mockito.any());
        Mockito.verifyNoMoreInteractions(personRepository);
    }
}
```

### Multi persistence units

Hibernate Reactive currently does not support multiple persistence units.

[Multi persistence units](https://quarkus.io/guides/hibernate-orm-panache#multiple-persistence-units)

### Caching

#### Caching of entities

To enable second-level cache, mark the entities that you want cached with `@javax.persistence.Cacheable`:

```java
@Entity
@Cacheable
public class Country {
    int dialInCode;
    // ...
}
```

When an entity is annotated with @Cacheable, all its field values are cached except for collections and relations to other entities.

#### Caching of collections and relations

Collections and relations need to be individually annotated to be cached; in this case the Hibernate specific `@org.hibernate.annotations.Cache` should be used, which requires also to specify the `CacheConcurrencyStrategy`

```java
package org.acme;

@Entity
@Cacheable
public class Country {
    // ...

    @OneToMany
    @Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
    List<City> cities;

    // ...
}
```

#### Caching of queries

To cache a query, mark it as cacheable on the Query instance:

```java
Query query = ...
query.setHint("org.hibernate.cacheable", Boolean.TRUE);
```

If you have a NamedQuery then you can enable caching directly on its definition, which will usually be on an entity:

```java
@Entity
@NamedQuery(name = "Fruits.findAll",
      query = "SELECT f FROM Fruit f ORDER BY f.name",
      hints = @QueryHint(name = "org.hibernate.cacheable", value = "true") )
public class Fruit {
    ···
```

#### Tuning of Cache Regions

>[tuning-of-cache-regions](https://quarkus.io/guides/hibernate-orm#tuning-of-cache-regions)

### Hibernate Validator/Bean Validation

1.Create Constraints
  
Constraints are added on fields, and when an object is validated, the values are checked. The getter and setter methods are also used for JSON mapping.

```java
  package org.acme.validation;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Min;

public class Book {

    @NotBlank(message="Title may not be blank")
    public String title;

    @NotBlank(message="Author may not be blank")
    public String author;

    @Min(message="Author has been very lazy", value=1)
    public double pages;
}
```

2.JSON mapping and validation

Create the following REST resource as `org.acme.validation.BookResource`:

```java
package org.acme.validation;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/books")
public class BookResource {

    @Inject
    Validator validator; 

    @Path("/manual-validation")
    @POST
    public Result tryMeManualValidation(Book book) {
        Set<ConstraintViolation<Book>> violations = validator.validate(book);
        if (violations.isEmpty()) {
            return new Result("Book is valid! It was validated by manual validation.");
        } else {
            return new Result(violations);
        }
    }
}
```

#### REST end point validation

As you can see, we don’t have to manually validate the provided Book anymore as it is automatically validated.

```java
@Path("/end-point-method-validation")
@POST
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public Result tryMeEndPointMethodValidation(@Valid Book book) {
    return new Result("Book is valid! It was validated by end point method validation.");
}
```

#### Service method validation

It might not always be handy to have the validation rules declared at the end point level as it could duplicate some business validation.

The best option is then to annotate a method of your business service with your constraints (or in our particular case with `@Valid`):

```java
package org.acme.validation;

import javax.enterprise.context.ApplicationScoped;
import javax.validation.Valid;

@ApplicationScoped
public class BookService {

    public void validateBook(@Valid Book book) {
        // your business logic here
    }
}
```

Calling the service in your rest end point triggers the Book validation automatically:

```java
@Inject BookService bookService;

@Path("/service-method-validation")
@POST
public Result tryMeServiceMethodValidation(Book book) {
    try {
        bookService.validateBook(book);
        return new Result("Book is valid! It was validated by service method validation.");
    } catch (ConstraintViolationException e) {
        return new Result(e.getConstraintViolations());
    }
}
```

Note that, if you want to push the validation errors to the frontend, you have to catch the exception and push the information yourselves as they will not be automatically pushed to the JSON output.
