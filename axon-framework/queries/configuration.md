# Configuration

This page describes the process when it comes to configuring a Query Handlers. Note, that a Query Handler is a \(singleton\) object containing `@QueryHandler` annotated functions.

## Registering a Query Handler

When you register a Query Handler, that means you are registering a class containing annotated query handlers. Upon receiving such a class during configuration, Axon will scan its contents for all `@QueryHandler` annotated methods. In the registration process the following information defines a given query handling function:

1. The first parameter of the method is the _query payload_.
2. The methods response type is the query's _response type_.
3. The value of the `queryName` field in the annotation as the query's _name_ \(this is optional and in its absence will default to the query payload\).    

Note that it is possible to register multiple query handlers for the same query payload, response type and name. Furthermore, when dispatching a query the client can indicate whether he/she wants the result from a [single handler ](query-dispatchers.md#point-to-point-queries)or the result from [all handlers](query-dispatchers.md#scatter-gather-queries) corresponding to the query payload, name and response type combination.

The following snippets point out how a Query Handler can be registered:

{% tabs %}
{% tab title="Axon Configuration API" %}
Given the existence of the following query handler:

```java
public class CardSummaryProjection {

    @QueryHandler
    public CardSummary handle(FetchCardSummaryQuery query) {
        CardSummary cardSummary;
        // Retrieve CardSummary instance, for example from a repository. 
        return cardSummary;
    }

}
```

The following is needed to register a `CardSummaryProjection` as being a Query Handler:

```java
Configurer axonConfigurer = DefaultConfigurer.defaultConfiguration()
    .registerQueryHandler(conf -> new CardSummaryProjection());
```

Or, a more general approach to registering _all_ types of message handlers in a component can be used:

```java
Configurer axonConfigurer = DefaultConfigurer.defaultConfiguration()
    .registerMessageHandler(conf -> new CardSummaryProjection());
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
When using Spring Boot, simply specifying the query handler as a bean is sufficient:

```java
@Component
public class CardSummaryProjection {

    @QueryHandler
    public CardSummary handle(FetchCardSummaryQuery query) {
        CardSummary cardSummary;
        // Retrieve CardSummary instance, for example from a repository. 
        return cardSummary;
    }

}
```
{% endtab %}
{% endtabs %}

> **Identical Query Handling methods in a single Query Handler**
>
> A query handler class can currently contain several identical query handling methods. The outcome of which method will actually be called is however unspecified.
>
> Note that this should be regarded as a _very_ uncommon scenario, as typically identical query handling methods would be spread over several query handlers.

