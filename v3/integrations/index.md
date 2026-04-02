# Integrations

## Overview
Spring Framework is itself an integration platform — it provides the abstractions that consumer applications use to integrate with external systems. The framework ships adapters for a broad set of protocols, message brokers, databases, and runtime platforms.

## Integration Catalogue

| System | Direction | Method | Purpose | Criticality |
|--------|----------|--------|---------|------------|
| JDBC databases (any RDBMS) | Outbound | `JdbcTemplate`, `DataSourceUtils` | Relational data access | High |
| JPA / Hibernate 7.x | Outbound | `LocalContainerEntityManagerFactoryBean`, `JpaTransactionManager` | ORM data access | High |
| R2DBC stores (H2, PostgreSQL, etc.) | Outbound | `R2dbcEntityTemplate`, `DatabaseClient` | Reactive relational data access | High |
| Jakarta JMS 3.x brokers (ActiveMQ, etc.) | Bidirectional | `JmsTemplate`, `@JmsListener`, `DefaultMessageListenerContainer` | Asynchronous message passing | Medium |
| WebSocket / STOMP | Bidirectional | `WebSocketHandler`, `@MessageMapping`, `SimpMessagingTemplate` | Real-time browser communication | Medium |
| RSocket | Bidirectional | `RSocketRequester`, `@MessageMapping` | Reactive inter-service messaging | Medium |
| HTTP (REST/GraphQL clients) | Outbound | `RestClient`, `RestTemplate`, `WebClient` | Calling external HTTP services | High |
| Servlet containers (Tomcat, Jetty, Undertow) | Inbound | `DispatcherServlet` (Jakarta Servlet 6) | HTTP request handling, MVC | High |
| Reactor Netty | Inbound | `ReactorHttpHandlerAdapter` | HTTP request handling, WebFlux | High |
| Undertow (non-Servlet) | Inbound | `UndertowHttpHandlerAdapter` | HTTP request handling, WebFlux | Medium |
| AspectJ weaver | Build/load-time | `spring-aspects` + `spring-instrument` agent | Load-time weaving for `@Configurable` | Low |
| GraalVM Native Image | Build-time | `RuntimeHints`, AOT processors | Native executable compilation | Growing |
| Micrometer | Outbound | `ObservationRegistry` instrumentation hooks | Metrics and distributed tracing | Medium |
| Quartz Scheduler | Bidirectional | `SchedulerFactoryBean` (spring-context-support) | Job scheduling | Low |
| JavaMail (Jakarta Mail) | Outbound | `JavaMailSenderImpl` (spring-context-support) | Email sending | Low |
| Groovy | Embedded | `GroovyBeanDefinitionReader`, script beans | Dynamic bean scripting | Low |

## APIs
- **RestClient / RestTemplate** — synchronous HTTP client built on `ClientHttpRequestFactory` adapters (JDK `HttpClient`, Apache HttpComponents 5, OkHttp 3)
- **WebClient** — reactive HTTP client backed by `ReactorClientHttpConnector` (Reactor Netty); part of spring-webflux
- **Auth:** The framework itself does not enforce authentication on outgoing requests — consumers configure interceptors/filters (Spring Security integrates here)
- **Contract:** No OpenAPI-level contract owned by this framework; consumers define their own APIs

## Events / Messaging
- **In-process events:** `ApplicationEventPublisher` → `@EventListener` methods; supports synchronous and `@Async` delivery; `ApplicationEvent` hierarchy or any arbitrary object as event payload
- **WebSocket STOMP:** `StompBrokerRelayMessageHandler` relays to external broker (RabbitMQ, ActiveMQ); `SimpleBrokerMessageHandler` for in-memory broker
- **JMS:** `DefaultMessageListenerContainer` polls JMS queue/topic; `@JmsListener` annotation-driven
- **RSocket:** `RSocketRequester` wraps RSocket Java SDK; `@MessageMapping` on controllers handles request/response, fire-and-forget, streaming

## Error Handling
- **Retry strategy:** Not built into the framework core; `RetryTemplate` is in Spring Retry (separate project); Reactor retry operators available for WebFlux
- **Failure behavior:** `@Transactional` rolls back on `RuntimeException` by default; `JmsTemplate` throws `JmsException` hierarchy; `RestClient` throws `RestClientException` hierarchy; WebFlux errors propagate as `Mono.error()`

## Dependencies
- **Upstream (frameworks depend on):** Jakarta EE APIs, Project Reactor, Reactor Netty, AspectJ, Kotlin stdlib + coroutines, Jackson, Micrometer, JUnit 6
- **Downstream (consume this framework):** Spring Boot, Spring Security, Spring Data, Spring Cloud, Spring Batch, virtually every Spring-ecosystem project

## Failure Modes
- If `DataSource` is unavailable at startup: `ApplicationContext` refresh fails; application does not start
- If reactive `ConnectionFactory` (R2DBC) is unavailable: Operators on `DatabaseClient` emit errors on subscription
- If JMS broker is unavailable: `DefaultMessageListenerContainer` retries with exponential backoff; `JmsTemplate` throws `CannotGetJmsConnectionException`
- If Reactor Netty fails to bind port: WebFlux `ReactiveWebServerApplicationContext` startup fails with `BindException`

## Known Issues
- `spring-oxm` XStream integration depends on a library with active CVEs; consumers must pin or exclude XStream
- `WebClient` (in spring-webflux) requires Reactor Netty on the classpath for HTTP calls; swapping to a different connector requires explicit `ClientHttpConnector` configuration
