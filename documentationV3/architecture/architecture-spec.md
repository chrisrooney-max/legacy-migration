# Architecture

## Overview
Spring Framework is a layered, multi-module Java library. It is not a running service — it is distributed as a set of JARs consumed by application runtimes. The architecture is a dependency-directed graph of Gradle subprojects, with `spring-core` and `spring-beans` at the base, `spring-context` building the IoC container on top, and web/data/messaging modules as leaf nodes.

## Context Diagram
```
┌───────────────────────────────────────────────────────────────┐
│  Consumer Application / Spring Boot                           │
├───────────┬───────────┬───────────┬────────────┬─────────────┤
│ spring-   │ spring-   │ spring-   │ spring-    │ spring-     │
│ webmvc    │ webflux   │ jdbc/orm  │ messaging  │ test        │
├───────────┴───────────┴───────────┴────────────┴─────────────┤
│ spring-tx  │ spring-aop  │ spring-expression  │ spring-web   │
├────────────┴─────────────┴───────────────────────────────────┤
│                   spring-context                              │
├───────────────────────────────────────────────────────────────┤
│                   spring-beans                                │
├───────────────────────────────────────────────────────────────┤
│                   spring-core                                 │
└───────────────────────────────────────────────────────────────┘
```

## Components

| Component | Responsibility | Location | Notes |
|---|---|---|---|
| spring-core | Type system utilities, resource abstraction, SpEL foundation, AOT hints, Kotlin utilities | `spring-core/src/main/java/org/springframework/core` | Multi-release JAR targets Java 21 and 24 |
| spring-beans | BeanFactory, BeanDefinition, dependency injection engine | `spring-beans/src/main/java/org/springframework/beans` | Core IoC container primitives |
| spring-context | ApplicationContext, event publishing, annotation config, lifecycle | `spring-context/src/main/java/org/springframework/context` | Builds the full container on top of spring-beans |
| spring-aop | AOP framework: Advisor, Pointcut, proxy creation (JDK + CGLIB) | `spring-aop/src/main/java/org/springframework/aop` | AspectJ weaving via spring-aspects |
| spring-aspects | AspectJ-based aspects including `@Configurable`, `@Transactional` weaving | `spring-aspects/` | Uses AspectJ compile-time/load-time weaving |
| spring-expression | Spring Expression Language (SpEL) parser and evaluator | `spring-expression/` | Used by annotations, security, data |
| spring-web | Common web abstractions: `HttpMethod`, `MediaType`, content negotiation, API versioning, `RestClient`/`RestTemplate` | `spring-web/` | Shared by MVC and WebFlux |
| spring-webmvc | DispatcherServlet, `@Controller`, `@RequestMapping`, view resolution, Servlet-based MVC | `spring-webmvc/` | Blocking / Servlet API |
| spring-webflux | DispatcherHandler, reactive `@Controller`, `RouterFunction`, Reactor Netty integration | `spring-webflux/` | Non-blocking, Project Reactor |
| spring-websocket | WebSocket protocol, STOMP sub-protocol, SockJS fallback | `spring-websocket/` | Bridges both MVC and WebFlux |
| spring-messaging | Message, MessageChannel, MessageHandler pipeline; STOMP, RSocket, JMS adapters | `spring-messaging/` | Foundation for WebSocket STOMP and Spring Integration |
| spring-tx | PlatformTransactionManager, ReactiveTransactionManager, @Transactional, transaction synchronisation | `spring-tx/` | Supports synchronous and reactive transactions |
| spring-jdbc | JdbcTemplate, DataSource utilities, SQL script execution | `spring-jdbc/` | Simplifies raw JDBC |
| spring-orm | JPA/Hibernate integration: EntityManagerFactory, JpaTransactionManager | `spring-orm/` | Thin adapter over JPA/Hibernate APIs |
| spring-r2dbc | Reactive relational DB support via R2DBC SPI | `spring-r2dbc/` | Reactive sibling of spring-jdbc |
| spring-jms | JmsTemplate, @JmsListener, message conversion | `spring-jms/` | Jakarta JMS 3.x |
| spring-oxm | Object/XML mapping (JAXB, Castor, XStream) | `spring-oxm/` | Less commonly used; niche XML binding |
| spring-test | TestContext framework, MockMvc, WebTestClient, JUnit Jupiter extensions, @MockitoBean | `spring-test/` | Largest module by file count (1,370 files) |
| spring-context-support | Mail, scheduling (Quartz), caching, Freemarker, Velocity integration | `spring-context-support/` | Optional add-ons to spring-context |
| spring-context-indexer | Annotation processor generating candidate component index | `spring-context-indexer/` | Optimises component scan startup time |
| spring-instrument | ClassLoader instrumentation agent for load-time weaving | `spring-instrument/` | Standalone Java agent JAR |

## Interaction Patterns
- **Synchronous request/response** — Spring MVC: `DispatcherServlet` → `HandlerMapping` → `HandlerAdapter` → `@Controller` → `ViewResolver`/`MessageConverter`
- **Reactive non-blocking** — Spring WebFlux: `DispatcherHandler` → `HandlerMapping` → `HandlerAdapter` → `@Controller`/`RouterFunction` returning `Mono`/`Flux`
- **Event-driven (in-process)** — `ApplicationEventPublisher` / `@EventListener` for synchronous and async Spring events
- **Message-driven** — `@JmsListener`, STOMP/WebSocket `@MessageMapping`, RSocket via `spring-messaging` pipeline

## Data Flow
Spring Framework itself is stateless infrastructure. At runtime in a consuming application:
1. Container startup: `ApplicationContext` reads `BeanDefinition`s (from annotations or XML), resolves dependencies, instantiates and wires beans.
2. Request path: incoming HTTP hits `DispatcherServlet` (MVC) or `DispatcherHandler` (WebFlux), which delegates through `HandlerMapping` → `HandlerAdapter` → controller method → response serialisation.
3. Transaction boundary: `@Transactional` advice (AOP proxy) begins/commits/rolls back a `PlatformTransactionManager` around the annotated method.

## External Dependencies
- **Project Reactor** (reactor-bom 2025.0.4) — Mono/Flux reactive types used throughout WebFlux and R2DBC
- **Reactor Netty** — embedded server for WebFlux (via Netty BOM 4.2.12)
- **Jakarta EE APIs** — jakarta.servlet, jakarta.transaction, jakarta.annotation, jakarta.persistence, jakarta.jms, jakarta.activation (Jakarta EE 11 level)
- **AspectJ** — compile-time and load-time weaving runtime
- **Kotlin 2.3.20 + Coroutines 1.10.2** — first-class Kotlin support throughout
- **Jackson BOM 2.21.2** — default JSON serialisation for MVC/WebFlux
- **Micrometer 1.16.4** — metrics instrumentation hooks in web and context modules
- **JUnit 6.0.3** — testing framework (JUnit Jupiter)
- **Mockito 5.22.0 + MockK 1.14.5** — mocking in tests

## Deployment Architecture
- **Environments:** Not deployed as a service; released as JARs to Maven Central and repo.spring.io/snapshot
- **Hosting:** Artefacts on Maven Central (signed JARs) and Artifactory (snapshots/milestones)
- **CI:** GitHub Actions on ubuntu-latest; daily matrix builds for Java 17, 21, and 25; snapshot deploy on every push to `main`

## Key Constraints
- **Java 17 baseline** — modules require Java 17+ to compile and run; some modules ship multi-release JARs with Java 21/24 optimisations (e.g., Virtual Threads in spring-core)
- **Jakarta EE 11** — v7.x targets Jakarta namespace (javax.* has been removed); not compatible with pre-Jakarta consumers

## Known Issues
- GraalVM native image coverage is incomplete for certain dynamic features (SpEL at runtime, reflective bean registration)
- Some `spring-oxm` marshallers (Castor, XStream) depend on libraries with known CVEs; consumers must manage these carefully
