# Tuning Command Processing

This page provides specifics around tuning the command processing within an Axon application.‌

## Duplicate Command Handler Registration <a id="duplicate-command-handler-registration"></a>

As described in the [Messaging Concepts](../../axon-framework/messaging-concepts/) page, a command is always routed to a single destination. This means that during the [registration of a command handler](../../axon-framework/axon-framework-commands/configuration.md#registering-a-command-handler) within a given JVM, a second registration of an identical command handler method should be dealt with in a desirable manor.‌

How an Axon application reacts to such a duplicate registration is defined by the `DuplicateCommandHandlerResolver`. This resolver is a functional interface ingesting a command name and a registered candidate command handler method; a single command handler method is the return value. By default the `LoggingDuplicateCommandHandlerResolver` is used, which will log a warning and returns the candidate handler.‌

To configure the used `DuplicateCommandHandlerResolver` it is suggested to use the `DuplicateCommandHandlerResolution`, as this class gives a handle to all provided implementations. To configure the duplicate resolver to, for example, throw a `DuplicateCommandHandlerSubscriptionException` as a warning, the following approach can be taken:

{% tabs %}
{% tab title="Axon Configuration API" %}
Somewhere in a configuration class:

```java
DefaultConfigurer.defaultConfiguration().registerComponent(
    DuplicateCommandHandlerResolution.class,
    config -> DuplicateCommandHandlerResolution.rejectDuplicates()
);
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
Somewhere in a configuration class:

```java
@Bean
public DuplicateCommandHandlerResolver duplicateCommandHandlerResolver() {
    return DuplicateCommandHandlerResolution.rejectDuplicates();
}
```
{% endtab %}
{% endtabs %}

