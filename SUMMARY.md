# Table of contents

* [Introduction](README.md)
* [Architecture Overview](architecture-overview/README.md)
  * [DDD & CQRS Concepts](architecture-overview/ddd-cqrs-concepts.md)
  * [Event Sourcing](architecture-overview/event-sourcing.md)
  * [Event-Driven Microservices](architecture-overview/event-driven-microservices.md)
* [Axon Server](axon-server-introduction.md)
* [Release Notes](release-notes/README.md)
  * [Axon Framework](release-notes/rn-axon-framework/README.md)
    * [Major Releases](release-notes/rn-axon-framework/rn-af-major-releases.md)
    * [Minor Releases](release-notes/rn-axon-framework/rn-af-minor-releases.md)
  * [Axon Server](release-notes/rn-axon-server/README.md)
    * [Major Releases](release-notes/rn-axon-server/rn-as-major-releases.md)
    * [Minor Releases](release-notes/rn-axon-server/rn-as-minor-releases.md)
  * [Axon Framework Extensions](release-notes/axon-framework-extensions.md)

## Getting Started

* [Quick Start](getting-started/quick-start.md)

## Axon Framework

* [Introduction](axon-framework/introduction.md)
* [Messaging Concepts](axon-framework/messaging-concepts/README.md)
  * [Anatomy of a Message](axon-framework/messaging-concepts/anatomy-message.md)
  * [Message Correlation](axon-framework/messaging-concepts/message-correlation.md)
  * [Message Intercepting](axon-framework/messaging-concepts/message-intercepting.md)
  * [Supported Parameters for Annotated Handlers](axon-framework/messaging-concepts/supported-parameters-annotated-handlers.md)
  * [Exception Handling](axon-framework/messaging-concepts/exception-handling.md)
  * [Unit of Work](axon-framework/messaging-concepts/unit-of-work.md)
* [Commands](axon-framework/axon-framework-commands/README.md)
  * [Modeling](axon-framework/axon-framework-commands/modeling/README.md)
    * [Aggregate](axon-framework/axon-framework-commands/modeling/aggregate.md)
    * [Multi-Entity Aggregates](axon-framework/axon-framework-commands/modeling/multi-entity-aggregates.md)
    * [State Stored Aggregates](axon-framework/axon-framework-commands/modeling/state-stored-aggregates.md)
    * [Aggregate Creation from another Aggregate](axon-framework/axon-framework-commands/modeling/aggregate-creation-from-another-aggregate.md)
    * [Aggregate Polymorphism](axon-framework/axon-framework-commands/modeling/aggregate-polymorphism.md)
    * [Conflict Resolution](axon-framework/axon-framework-commands/modeling/conflict-resolution.md)
  * [Command Dispatchers](axon-framework/axon-framework-commands/command-dispatchers.md)
  * [Command Handlers](axon-framework/axon-framework-commands/command-handlers.md)
  * [Implementations](axon-framework/axon-framework-commands/implementations.md)
  * [Configuration](axon-framework/axon-framework-commands/configuration.md)
* [Events](axon-framework/events/README.md)
  * [Event Dispatchers](axon-framework/events/event-dispatchers.md)
  * [Event Handlers](axon-framework/events/event-handlers.md)
  * [Event Processors](axon-framework/events/event-processors.md)
  * [Event Bus & Event Store](axon-framework/events/event-bus-and-event-store.md)
  * [Event Versioning](axon-framework/events/event-versioning.md)
  * [Event Serialization](axon-framework/events/event-serialization.md)
* [Queries](axon-framework/queries/README.md)
  * [Query Processing](axon-framework/queries/query-processing.md)
  * [Query Dispatchers](axon-framework/queries/query-dispatchers.md)
  * [Query Handlers](axon-framework/queries/query-handlers.md)
  * [Implementations](axon-framework/queries/implementations.md)
  * [Configuration](axon-framework/queries/configuration.md)
* [Sagas](axon-framework/sagas/README.md)
  * [Implementation](axon-framework/sagas/implementation.md)
  * [Associations](axon-framework/sagas/associations.md)
  * [Infrastructure](axon-framework/sagas/infrastructure.md)
* [Deadlines](axon-framework/deadlines/README.md)
  * [Deadline Managers](axon-framework/deadlines/deadline-managers.md)
  * [Event Schedulers](axon-framework/deadlines/event-schedulers.md)
* [Testing](axon-framework/testing/README.md)
  * [Commands / Events](axon-framework/testing/commands-events.md)
  * [Sagas](axon-framework/testing/sagas-1.md)
* [Tuning](axon-framework/tuning/README.md)
  * [Event Snapshots](axon-framework/tuning/event-snapshots.md)
  * [Event Processing](axon-framework/tuning/event-processing.md)
  * [Command Processing](axon-framework/tuning/command-processing.md)
* [Monitoring and Metrics](axon-framework/monitoring-and-metrics.md)
* [Spring Boot Integration](axon-framework/spring-boot-integration.md)
* [Modules](axon-framework/modules.md)

## Axon Server

* [Introduction](axon-server/introduction.md)
* [Installation](axon-server/installation/README.md)
  * [Local Installation](axon-server/installation/local-installation/README.md)
    * [Axon Server SE](axon-server/installation/local-installation/axon-server-se.md)
    * [Axon Server EE](axon-server/installation/local-installation/axon-server-ee.md)
  * [Docker / K8s](axon-server/installation/docker-k8s/README.md)
    * [Axon Server SE](axon-server/installation/docker-k8s/axon-server-se.md)
    * [Axon Server EE](axon-server/installation/docker-k8s/axon-server-ee.md)
* [Administration](axon-server/administration/README.md)
  * [Configuration](axon-server/administration/admin-configuration/README.md)
    * [System Properties](axon-server/administration/admin-configuration/configuration.md)
    * [Command Line Interface](axon-server/administration/admin-configuration/command-line-interface.md)
    * [REST API](axon-server/administration/admin-configuration/rest-api.md)
  * [Monitoring](axon-server/administration/monitoring/README.md)
    * [Actuator Endpoints](axon-server/administration/monitoring/actuator-endpoints.md)
    * [gRPC Metrics](axon-server/administration/monitoring/grpc-metrics.md)
    * [Heartbeat Monitoring](axon-server/administration/monitoring/heartbeat-monitoring.md)
  * [Clusters](axon-server/administration/clustering.md)
  * [Replication Groups](axon-server/administration/replication-groups.md)
  * [Multi-Context](axon-server/administration/multi-context.md)
  * [Tagging](axon-server/administration/tagging.md)
  * [Backup and Messaging-only Nodes](axon-server/administration/backup-and-messaging-only-nodes.md)
  * [Backups](axon-server/administration/backups.md)
  * [Recovery](axon-server/administration/recovery.md)
* [Security](axon-server/security/README.md)
  * [Access Control](axon-server/security/access-control.md)
  * [SSL](axon-server/security/ssl.md)
* [Performance](axon-server/performance/README.md)
  * [Event Segments](axon-server/performance/tuning-event-processing.md)
  * [Flow Control](axon-server/performance/flow-control.md)
* [Migration](axon-server/migration/README.md)
  * [Standard to Enterprise Edition](axon-server/migration/standard-to-enterprise-edition.md)
  * [Non-Axon Server to Axon Server](axon-server/migration/non-axon-server-to-axon-server.md)

## Extensions

* [Spring AMQP](extensions/spring-amqp.md)
* [JGroups](extensions/jgroups.md)
* [Kafka](extensions/kafka.md)
* [Kotlin](extensions/kotlin.md)
* [Mongo](extensions/mongo.md)
* [Reactor](extensions/reactor/reactor.md)
  * [Reactor Gateways](extensions/reactor/reactive-gateways/reactive-gateways.md)
* [Spring Cloud](extensions/spring-cloud.md)
* [Tracing](extensions/tracing.md)

## Appendices

* [A. RDBMS Tuning](appendices/rdbms-tuning.md)
* [B. Message Handler Tuning](appendices/message-handler-tuning/README.md)
  * [Parameter Resolvers](appendices/message-handler-tuning/parameter-resolvers.md)
  * [Handler Enhancers](appendices/message-handler-tuning/handler-enhancers.md)
* [C. Meta Annotations](appendices/meta-annotations.md)
* [D. Identifier Generation](appendices/identifier-generation.md)
* [E. Axon Server Query Language](appendices/query-reference.md)