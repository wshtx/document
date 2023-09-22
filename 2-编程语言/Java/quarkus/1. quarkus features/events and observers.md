# Events and Observer methods

- [Events and Observer methods](#events-and-observer-methods)
  - [Simple Cases about Events and Observers methods](#simple-cases-about-events-and-observers-methods)
  - [Event](#event)
    - [Event Object](#event-object)
    - [Event qualifiers](#event-qualifiers)
    - [Defining event qualifiers](#defining-event-qualifiers)
    - [Applying qualifiers to event](#applying-qualifiers-to-event)
    - [Event producers](#event-producers)
  - [Event Observers](#event-observers)
    - [Observer resolution](#observer-resolution)
  - [Transactional observers](#transactional-observers)

>[more infomation about Event](https://docs.jboss.org/weld/reference/latest/en-US/html/events.html)

Dependency injection enables loose-coupling by allowing the implementation of the injected bean type to vary, either at deployment time or runtime. Events go one step further, allowing beans to interact with no compile time dependency at all. **Event producers raise events that are delivered to event observers by the container.**
This basic schema might sound like the familiar observer/observable pattern, but there are a couple of twists:

- not only are event producers decoupled from observers; observers are completely decoupled from producers,
- observers can specify a combination of "selectors" to narrow the set of event notifications they will receive, and
- observers can be notified immediately, or can specify that delivery of the event should be delayed until the end of the current transaction.

## Simple Cases about Events and Observers methods

```java
class TaskCompleted {
  // ...
}

@ApplicationScoped
class ComplicatedService {

   @Inject
   Event<TaskCompleted> event; //javax.enterprise.event.Event is used to fire events.

   void doSomething() {
      // ...
      event.fire(new TaskCompleted()); //Fire the event synchronously.
   }

}

@ApplicationScoped
class Logger {

   void onTaskCompleted(@Observes TaskCompleted task) {//This method is notified when a TaskCompleted event is fired.

      // ...log the task
   }

}
```

## Event

Beans may produce and consume events. This facility allows beans to interact in a completely decoupled fashion, with no compile-time dependency between the interacting beans.Most importantly, **it allows stateful beans in one architectural tier of the application to synchronize their internal state with state changes that occur in a different tier**.

Events consist of an event object(a java class) and A set of instances of qualifier types - the event qualifiers.

### Event Object

*The event object acts as a payload, to propagate state from producer to consumer. The event qualifiers act as topic selectors, allowing the consumer to narrow the set of events it observes.*

### Event qualifiers

An event may be assigned qualifiers, which allows observers to distinguish it from other events of the same type. **The qualifiers function like topic selectors, allowing an observer to narrow the set of events it observes**.

### Defining event qualifiers

An event qualifier is just a normal qualifier, defined using @Qualifier. Here’s an example:

```java
@Qualifier
@Target({METHOD, FIELD, PARAMETER, TYPE})
@Retention(RUNTIME)
public @interface Role {
   RoleType value();
}
```

The member value is used to narrow the messages delivered to the observer:

```java
public void adminLoggedIn(@Observes @Role(ADMIN) LoggedIn event) { ... }
```

Event qualifier type members may be specified statically by the event producer, via annotations at the event notifier injection point:

```java
@Inject @Role(ADMIN) Event<LoggedIn> loggedInEvent;
```

Alternatively, the value of the event qualifier type member may be determined dynamically by the event producer. We start by writing an abstract subclass of `AnnotationLiteral`:

```java
abstract class RoleBinding
   extends AnnotationLiteral<Role>
   implements Role {}

documentEvent.select(new RoleBinding() {
   public void value() { return user.getRole(); }
}).fire(document);
```

### Applying qualifiers to event

Qualifiers can be applied to an event in one of two ways:

- by annotating the `Event` injection point, or(simpler method)
- by passing qualifiers to the `select()` of `Event`.

```java
@Inject @Updated Event<Document> documentUpdatedEvent;

documentEvent.select(new AnnotationLiteral<Updated>(){}).fire(document);
```

### Event producers

Event producers fire events either synchronously or asynchronously using an instance of the parameterized Event interface. An instance of this interface is obtained by injection:

```java
@Inject @Any Event<Document> documentEvent;

//synchronous event producer
documentEvent.fire(document);

//asynchronous event producer
documentEvent.fireAsync(document);
```

## Event Observers

An observer method is a method of a bean with a parameter annotated `@Observes` or `@ObservesAsync`.

This particular event will only be delivered to asynchronous/synchronous observer method that:

- has an event parameter to which the event object (the Document) is assignable, and
- An observer method is notified if the observer method has no event qualifiers or has a subset of the event qualifiers.

```java
public void onAnyDocumentEvent(@Observes(receive = IF_EXISTS) Document document) 
{... }

public void onAnyDocumentEvent(@ObservesAsync Document document) { ... }
```

We may want to deliver events only to instances of the observer that already exist in the current contexts, A conditional observer is specified by adding `receive = IF_EXISTS` to the `@Observes` annotation.

### Observer resolution

>[Observer resolution](https://jakarta.ee/specifications/cdi/4.0/jakarta-cdi-spec-4.0.html#observer_resolution)

## Transactional observers

Transactional observers receive their event notifications during the before or after completion phase of the transaction in which the event was raised. There are five kinds of transactional observers:

- `during = IN_PROGRESS` observers are called immediately (default)
- `during = AFTER_SUCCESS` observers are called during the after completion phase of the transaction, but only if the transaction completes successfully
- `during = AFTER_FAILURE` observers are called during the after completion phase of the transaction, but only if the transaction fails to complete successfully
- `during = AFTER_COMPLETION` observers are called during the after completion phase of the transaction
- `during = BEFORE_COMPLETION` observers are called during the before completion phase of the transaction

Imagine that we have cached a JPA query result set in the application scope:

```java
import jakarta.ejb.Singleton;
import jakarta.enterprise.inject.Produces;

@ApplicationScoped @Singleton
public class Catalog {

   @PersistenceContext EntityManager em;

   List<Product> products;//needed to refresh when 

   @Produces @Catalog
   List<Product> getCatalog() {
      if (products==null) {
         products = em.createQuery("select p from Product p where p.deleted = false")
            .getResultList();
      }
      return products;
   }

}
```

The bean that creates and deletes `Product`s could raise events, for example:

```java
import jakarta.enterprise.event.Event;

@Stateless
public class ProductManager {
   @PersistenceContext EntityManager em;
   @Inject @Any Event<Product> productEvent;

   public void delete(Product product) {
      em.delete(product);
      productEvent.select(new AnnotationLiteral<Deleted>(){}).fire(product);
   }

   public void persist(Product product) {
      em.persist(product);
      productEvent.select(new AnnotationLiteral<Created>(){}).fire(product);
   }
   ...
}
```

And now Catalog can observe the events after successful completion of the transaction:

```java
import jakarta.ejb.Singleton;

@ApplicationScoped @Singleton
public class Catalog {
   ...
   void addProduct(@Observes(during = AFTER_SUCCESS) @Created Product product) {
      products.add(product);
   }

   void removeProduct(@Observes(during = AFTER_SUCCESS) @Deleted Product product) {
      products.remove(product);
   }
}
```
