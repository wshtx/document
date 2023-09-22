# Event bus
>[offcial document](https://quarkus.io/guides/reactive-event-bus)

Qurkuus allows different beans to **interact using asynchronous events**, thus promoting loos-coupling.The message is sent to virtual address.Quarkus offers 3 types of delivery mechanism:

- *point-to-point* - send the message, one consumer receives it. If several consumers listen to the address, a round-robin is applied;
- *publish/subscribe* - publish a message, all the consumers listening to the address are receiving the message;
- *request/reply* - send the message and expect a response. The receiver can respond to the message in an asynchronous-fashion

## Sending/Consuming events

Any bean can use the annotation `ConsumeEvent(value = "", blocking = true)` to regard a method as a event.

***The address is fully qualified name of the bean, The method parameters is the message body, and the method returns something(i.e.`Uni`, `CompletionStage`, `void`) it`s the message response.*** 

The follow example show how to implement a *point-to-point* mechanism.

```java

// Event resolver(message receiver)

//event name: consume()
//virtual address: address/org.acme.vertx.GreetingService(if not set)
//message body: String name
//message response: String
//is_blocking: false
package org.acme.vertx;

import io.quarkus.vertx.ConsumeEvent;

import javax.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class GreetingService {

    @ConsumeEvent(value="greeting", blocking = false)                       
    public String consume(String name) {    
        return name.toUpperCase();
    }
}
```

```java
// Event Sender(message sender)

package org.acme.vertx;

import io.smallrye.mutiny.Uni;
import io.vertx.mutiny.core.eventbus.EventBus;
import io.vertx.mutiny.core.eventbus.Message;

import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/async")
public class EventResource {

    @Inject
    EventBus bus;                                       

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("{name}")
    public Uni<String> greeting(String name) {
        return bus.<String>request("greeting", name)        
                .onItem().transform(Message::body);
    }
}

```

And The EventBus object provides methods to:

1. send a message to a specific address - one single consumer receives the message.
2. publish a message to a specific address - all consumers receive the messages.
3. send a message and expect reply asynchronously
4. send a message and expect reply in a blocking manner

```java
// Case 1
bus.<String>requestAndForget("greeting", name);
// Case 2
bus.publish("greeting", name);
// Case 3
Uni<String> response = bus.<String>request("address", "hello, how are you?")
        .onItem().transform(Message::body);
// Case 4
String response = bus.<String>requestAndAwait("greeting", name).body();
```

