# Message Intercepting

There are two different types of interceptors: dispatch interceptors and handler interceptors. Dispatch interceptors are invoked before a message is dispatched to a message handler. At that point, it may not even be known that a handler exists for that message. Handler interceptors are invoked just before the message handler is invoked.

## Command Interceptors

One of the advantages of using a command bus is the ability to undertake action based on all incoming commands. Examples are logging or authentication, which you might want to do regardless of the type of command. This is done using Interceptors.

### Command Dispatch Interceptors

Message dispatch interceptors are invoked when a command is dispatched on a command bus. They have the ability to alter the command message by adding metadata. They can also block the command by throwing an exception. These interceptors are always invoked on the thread that dispatches the command.

Let's create a `MessageDispatchInterceptor` which logs each command message being dispatched on a `CommandBus`.

```java
public class MyCommandDispatchInterceptor implements MessageDispatchInterceptor<CommandMessage<?>> {

    private static final Logger LOGGER = LoggerFactory.getLogger(MyCommandDispatchInterceptor.class);

    @Override
    public BiFunction<Integer, CommandMessage<?>, CommandMessage<?>> handle(List<? extends CommandMessage<?>> messages) {
        return (index, command) -> {
            LOGGER.info("Dispatching a command {}.", command);
            return command;
        };
    }
}
```

We can register this dispatch interceptor with a `CommandBus` by doing the following:

```java
public class CommandBusConfiguration {

    public CommandBus configureCommandBus() {
        CommandBus commandBus = SimpleCommandBus.builder().build();
        commandBus.registerDispatchInterceptor(new MyCommandDispatchInterceptor());
        return commandBus;
    }
}
```

#### Structural validation

There is no point in processing a command if it does not contain all required information in the correct format. In fact, a command that lacks information should be blocked as early as possible, preferably even before a transaction has been started. Therefore, an interceptor should check all incoming commands for the availability of such information. This is called structural validation.

Axon Framework has support for JSR 303 Bean Validation based validation. This allows you to annotate the fields on commands with annotations like `@NotEmpty` and `@Pattern`. You need to include a JSR 303 implementation \(such as Hibernate-Validator\) on your classpath. Then, configure a `BeanValidationInterceptor` on your command bus, and it will automatically find and configure your validator implementation. While it uses sensible defaults, you can fine-tune it to your specific needs.

> **Interceptor Ordering Tip**
>
> You want to spend as few resources on an invalid command as possible. Therefore, this interceptor is generally placed at the very front of the interceptor chain. In some cases, a `LoggingInterceptor` or `AuditingInterceptor` might need to be placed first, with the validating interceptor immediately following it.

The `BeanValidationInterceptor` also implements `MessageHandlerInterceptor`, allowing you to configure it as a handler interceptor as well.

### Command Handler Interceptors

Message handler interceptors can take action both before and after command processing. Interceptors can even block command processing altogether, for example for security reasons.

Interceptors must implement the `MessageHandlerInterceptor` interface. This interface declares one method, `handle`, that takes two parameters: the current `UnitOfWork` and an `InterceptorChain`. The `InterceptorChain` is used to continue the dispatching process. The `UnitOfWork` gives you \(1\) the message being handled and \(2\) provides the possibility to tie in logic prior, during or after \(command\) message handling \(see [Unit Of Work](unit-of-work.md) for more information about the phases\).

Unlike dispatch interceptors, handler interceptors are invoked in the context of the command handler. That means they can attach correlation data based on the message being handled to the unit of work, for example. This correlation data will then be attached to messages being created in the context of that unit of work.

Handler interceptors are also typically used to manage transactions around the handling of a command. To do so, register a `TransactionManagingInterceptor`, which in turn is configured with a `TransactionManager` to start and commit \(or roll back\) the actual transaction.

Let's create a Message Handler Interceptor which will only allow the handling of commands that contain `axonUser` as a value for the `userId` field in the `MetaData`. If the `userId` is not present in the metadata, an exception will be thrown which will prevent the command from being handled. Additionally, if the value of `userId` does not match `axonUser`, we will also not proceed up the chain.

```java
public class MyCommandHandlerInterceptor implements MessageHandlerInterceptor<CommandMessage<?>> {

    @Override
    public Object handle(UnitOfWork<? extends CommandMessage<?>> unitOfWork, InterceptorChain interceptorChain) throws Exception {
        CommandMessage<?> command = unitOfWork.getMessage();
        String userId = Optional.ofNullable(command.getMetaData().get("userId"))
                                .map(uId -> (String) uId)
                                .orElseThrow(IllegalCommandException::new);
        if ("axonUser".equals(userId)) {
            return interceptorChain.proceed();
        }
        return null;
    }
}
```

We can register the handler interceptor with a `CommandBus` like so:

```java
public class CommandBusConfiguration {

    public CommandBus configureCommandBus() {
        CommandBus commandBus = SimpleCommandBus.builder().build();
        commandBus.registerHandlerInterceptor(new MyCommandHandlerInterceptor());
        return commandBus;
    }

}
```

#### `@CommandHandlerInterceptor` Annotation

The framework has the ability to add a Handler Interceptor as a `@CommandHandlerInterceptor` annotated method on the Aggregate/Entity. The difference between a method on an Aggregate and a "[regular](message-intercepting.md#command-handler-interceptors)" Command Handler Interceptor, is that with the annotation approach you can make decisions based on the current state of the given Aggregate. Some properties of an annotated Command Handler Interceptor are:

* The annotation can be put on entities within the Aggregate.
* It is possible to intercept a command on Aggregate Root level, whilst the command handler is in a child entity.
* Command execution can be prevented by firing an exception from an annotated command handler interceptor.
* It is possible to define an `InterceptorChain` as a parameter of the command handler interceptor method and use it to control command execution.
* By using the `commandNamePattern` attribute of the `@CommandHandlerInterceptor` annotation we can intercept all commands matching the provided regular expression.
* Events can be applied from an annotated command handler interceptor.

In the example below we can see a `@CommandHandlerInterceptor` annotated method which prevents command execution if a command's `state` field does not match the Aggregate's `state` field:

```java
public class GiftCard {
    //..
    private String state;
    //..
    @CommandHandlerInterceptor
    public void intercept(RedeemCardCommand command, InterceptorChain interceptorChain) {
        if (this.state.equals(command.getState())) {
            interceptorChain.proceed();
        }
    }
}
```

Note that the `@CommandHandlerInterceptor` is essentially a more specific implementation of the `@MessageHandlerInterceptor` described [here](message-intercepting.md#messagehandlerinterceptor).

## Event Interceptors

Similar to command messages, event messages can also be intercepted prior to publishing and handling to perform additional actions on all events. This boils down to the same two types of interceptors for messages: the dispatch interceptor and the handler interceptor.

### Event Dispatch Interceptors

Any message dispatch interceptors registered to an event bus will be invoked when an event is published. They have the ability to alter the event message by adding metadata. They can also provide you with overall logging capabilities for when an event is published. These interceptors are always invoked on the thread that published the event.

Let's create an event message dispatch interceptor which logs each event message being published on an `EventBus`.

```java
public class EventLoggingDispatchInterceptor
                implements MessageDispatchInterceptor<EventMessage<?>> {

    private static final Logger logger =
                LoggerFactory.getLogger(EventLoggingDispatchInterceptor.class);

    @Override
    public BiFunction<Integer, EventMessage<?>, EventMessage<?>> handle(
                List<? extends EventMessage<?>> messages) {
        return (index, event) -> {
            logger.info("Publishing event: [{}].", event);
            return event;
        };
    }
}
```

We can then register this dispatch interceptor with an `EventBus` by doing the following:

```java
public class EventBusConfiguration {

    public EventBus configureEventBus(EventStorageEngine eventStorageEngine) {
        // note that an EventStore is a more specific implementation of an EventBus
        EventBus eventBus = EmbeddedEventStore.builder()
                                              .storageEngine(eventStorageEngine)
                                              .build();
        eventBus.registerDispatchInterceptor(new EventLoggingDispatchInterceptor());
        return eventBus;
    }
}
```

### Event Handler Interceptors

Message handler interceptors can take action both before and after event processing. Interceptors can even block event processing altogether, for example for security reasons.

Interceptors must implement the `MessageHandlerInterceptor` interface. This interface declares one method, `handle()`, that takes two parameters: the current `UnitOfWork` and an `InterceptorChain`. The `InterceptorChain` is used to continue the dispatching process. The `UnitOfWork` gives you \(1\) the message being handled and \(2\) provides the possibility to tie in logic prior, during or after \(event\) message handling \(see [Unit Of Work](unit-of-work.md) for more information about the phases\).

Unlike dispatch interceptors, handler interceptors are invoked in the context of the event handler. That means they can attach correlation data based on the message being handled to the unit of work, for example. This correlation data will then be attached to event messages being created in the context of that unit of work.

Let's create a message handler interceptor which will only allow the handling of events that contain `axonUser` as the value for the `userId` field in the `MetaData`. If the `userId` is not present in the metadata, an exception will be thrown which will prevent the Event from being handled. And if the value of `userId` does not match `axonUser`, we will also not proceed up the chain. Authenticating the event message like shown in this example is a regular use case of the `MessageHandlerInterceptor`.

```java
public class MyEventHandlerInterceptor
        implements MessageHandlerInterceptor<EventMessage<?>> {

    @Override
    public Object handle(UnitOfWork<? extends EventMessage<?>> unitOfWork,
                         InterceptorChain interceptorChain) throws Exception {
        EventMessage<?> event = unitOfWork.getMessage();
        String userId = Optional.ofNullable(event.getMetaData().get("userId"))
                                .map(uId -> (String) uId)
                                .orElseThrow(IllegalEventException::new);
        if ("axonUser".equals(userId)) {
            return interceptorChain.proceed();
        }
        return null;
    }
}
```

We can register the handler interceptor with an `EventProcessor` like so:

```java
public class EventProcessorConfiguration {

    public void configureEventProcessing(Configurer configurer) {
        configurer.eventProcessing()
                  .registerTrackingEventProcessor("my-tracking-processor")
                  .registerHandlerInterceptor("my-tracking-processor",
                                              configuration -> new MyEventHandlerInterceptor());
    }
}
```

> **Interceptor Registration**
>
> Different from the `CommandBus` and `QueryBus`, which both can have handler interceptors and dispatch interceptors, the `EventBus` can only register dispatch interceptors. This is because the sole purpose of the `EventBus` is event publishing/dispatching, thus they are where event dispatch interceptors are registered. `EventProcessor`s are in charge of handling event messages, thus they are where event handler interceptors are registered.

## Query Interceptors

One of the advantages of using a query bus is the ability to undertake action based on all incoming queries. Examples are logging or authentication, which you might want to do regardless of the type of query. This is done using interceptors.

### Query Dispatch Interceptors

Message dispatch interceptors are invoked when a query is dispatched on a query bus or when a subscription update to a query message is dispatched on a query update emitter. They have the ability to alter the message by adding metadata. They can also block the handler execution by throwing an exception. These interceptors are always invoked on the thread that dispatches the message.

#### Structural validation

There is no point in processing a query if it does not contain all required information in the correct format. In fact, a query that lacks information should be blocked as early as possible. Therefore, an interceptor should check all incoming queries for the availability of such information. This is called structural validation.

Axon Framework has support for JSR 303 Bean Validation based validation. This allows you to annotate the fields on queries with annotations like `@NotEmpty` and `@Pattern`. You need to include a JSR 303 implementation \(such as Hibernate-Validator\) on your classpath. Then, configure a `BeanValidationInterceptor` on your query bus, and it will automatically find and configure your validator implementation. While it uses sensible defaults, you can fine-tune it to your specific needs.

> **Interceptor Ordering Tip**
>
> You want to spend as few resources on an invalid queries as possible. Therefore, this interceptor is generally placed at the very front of the interceptor chain. In some cases, a logging or auditing interceptor might need to be placed first, with the validating interceptor immediately following it.

The `BeanValidationInterceptor` also implements `MessageHandlerInterceptor`, allowing you to configure it as a handler interceptor as well.

### Query Handler Interceptors

Message handler interceptors can take action both before and after query processing. Interceptors can even block query processing altogether, for example for security reasons.

Interceptors must implement the `MessageHandlerInterceptor` interface. This interface declares one method, `handle`, that takes two parameters: the current `UnitOfWork` and an `InterceptorChain`. The `InterceptorChain` is used to continue the dispatching process. The `UnitOfWork` gives you \(1\) the message being handled and \(2\) provides the possibility to tie in logic prior, during or after \(query\) message handling \(see[ Unit Of Work ](unit-of-work.md)for more information about the phases\).

Unlike dispatch interceptors, handler interceptors are invoked in the context of the query handler. That means they can attach correlation data based on the message being handled to the unit of work, for example. This correlation data will then be attached to messages being created in the context of that unit of work.

## @MessageHandlerInterceptor

Alongside defining overall `MessageHandlerInterceptor` instances on the component handling a message \(e.g. a command, query or event\), it is also possible to define a handler interceptor for a specific component containing the handlers. This can be achieved by adding a method handling the message, combined with the `@MessageHandlerInterceptor` annotation. Adding such a method allows you more fine-grained control over which message handling components should react and how these should react.

Several handles are given to you when it comes to adding the `@MessageHandlerInterceptor`, like:

1. `MessageHandlerInterceptor` instances work with the `InterceptorChain` to decide when to proceed with other interceptors in the chain. The `InterceptorChain` is an _optional_ parameter which can be added to the intercepting method to provide you with the same control. In absence of this parameter, the framework will call `InterceptorChain#proceed` once the method is exited.
2. You can define the type of `Message` the interceptor should deal with. By default, it reacts to any `Message` implementation. If an `EventMessage` specific interceptor is desired, the `messageType` parameter on the annotation should be set to `EventMessage.class`.
3. For even more fine-grained control of which messages should react to the interceptor, the `payloadType` contained in the `Message` to handle can be specified.    

The following snippets shows some possible approaches of using the `@MessageHandlerInterceptor` annotation:

{% tabs %}
{% tab title="Simple @MessageHandlerInterceptor method" %}
```java
public class CardSummaryProjection {
    /*
     * Some @EventHandler and @QueryHandler annotated methods
     */
    @MessageHandlerInterceptor
    public void intercept(Message<?> message) {
        // Add your intercepting logic here based on the
    }
}
```
{% endtab %}

{% tab title="@MessageHandlerInterceptor method defining the Message type" %}
```java
public class CardSummaryProjection {
    /*
     * Some @EventHandler and @QueryHandler annotated methods
     */
    @MessageHandlerInterceptor(messageType = EventMessage.class)
    public void intercept(EventMessage<?> eventMessage) {
        // Add your intercepting logic here based on the
    }
}
```



```java
public class CardSummaryProjection {
    /*
     * Some @EventHandler and @QueryHandler annotated methods
     */
    @MessageHandlerInterceptor(
        messageType = EventMessage.class,
        payloadType = CardRedeemedEvent.class
    )
    public void intercept(CardRedeemedEvent event) {
        // Add your intercepting logic here based on the
    }
}
```
{% endtab %}

{% tab title="@MessageHandlerInterceptor method defining an InterceptorChain parameter" %}
```java
public class CardSummaryProjection {
    /*
     * Some @EventHandler and @QueryHandler annotated methods
     */
    @MessageHandlerInterceptor(messageType = QueryMessage.class)
    public void intercept(QueryMessage<?, ?> queryMessage,
                          InterceptorChain interceptorChain) throws Exception {
        // Add your intercepting logic before
        interceptorChain.proceed();
        // or after the InterceptorChain#proceed invocation
    }
}
```
{% endtab %}
{% endtabs %}

Next to the message, payload and `InterceptorChain`, a `@MessageHandlerInterceptor` annotated method can resolve other parameters as well. Which parameters the framework can resolve on such a function, is based on the type of `Message` being handled by the interceptor. For more specifics on which parameters are resolvable for the `Message` being handled, take a look at [this](supported-parameters-annotated-handlers.md) page.

### @ExceptionHandler

The `@MessageHandlerInterceptor` allows for a more specific version of an intercepting function too. Namely, an `@ExceptionHandler` annotated method. A function annotated with `@ExceptionHandler` will be regarded as a handler interceptor which will _only_ be invoked for exceptional results. Using annotated functions to this end for example allow you to throw a more domain specific exception as a result of a thrown database/service exception. You can introduce an `@ExceptionHandler` which reacts to all exceptions, or specify which exceptions annotated handler reacts on as shown here:

```java
public class CardSummaryProjection {

    /*
     * Some @EventHandler and @QueryHandler annotated methods
     */
    @ExceptionHandler
    public void handle(Exception exception) {
        // How you prefer to react to this generic exception,
        //  for example by throwing a domain specific exception.
    }

    @ExceptionHandler(resultType = IllegalArgumentException.class)
    public void handle(IllegalArgumentException exception) {
        // How you prefer to react to the IllegalArgumentException,
        //  for example by throwing a domain specific exception.
    }
}
```

