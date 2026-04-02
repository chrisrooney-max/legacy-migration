# System Overview

## Purpose
- Spring Framework is a comprehensive application framework and inversion-of-control (IoC) container for the Java/JVM ecosystem.
- It solves the enterprise application development problem by providing declarative infrastructure services (dependency injection, transaction management, aspect-oriented programming, web MVC, reactive web, data access) so application code can focus on business logic.

## Key Capabilities
- **IoC / Dependency Injection** — `BeanFactory` and `ApplicationContext` manage object lifecycles and wiring via annotation (`@Component`, `@Bean`, `@Autowired`) and XML/programmatic configuration.
- **Aspect-Oriented Programming (AOP)** — proxy-based and AspectJ-weaving AOP for cross-cutting concerns (security, transactions, logging).
- **Web frameworks** — Spring MVC (Servlet/blocking) and Spring WebFlux (reactive, Project Reactor) for building REST and browser-facing APIs.
- **Data access** — `JdbcTemplate`, transaction abstraction (`@Transactional`), ORM integration (Hibernate/JPA via spring-orm), reactive R2DBC support.
- **Messaging** — STOMP/WebSocket, JMS, RSocket, and generic `MessageChannel`-based messaging pipeline.
- **Testing support** — deep integration with JUnit Jupiter: `@SpringBootTest`-style context loading, `MockMvc`, `WebTestClient`, transactional rollback, and mocked bean support.
- **AOT / GraalVM Native** — ahead-of-time compilation hints and `RuntimeHints` infrastructure for native image compatibility.
- **Expression Language (SpEL)** — powerful in-process expression evaluation used across annotations and configuration.

## Primary Users
- **Application developers** — build Spring-based applications (Spring Boot being the primary entry point) using the abstractions provided by this framework.
- **Framework authors** — build higher-level Spring projects (Spring Boot, Spring Security, Spring Data, Spring Cloud) on top of this core.
- **Library integrators** — third-party libraries integrate with Spring's `BeanFactory`, `ApplicationContext`, and web abstractions.

## Business Criticality
- [x] Mission critical — Spring Framework underpins the majority of Java enterprise applications worldwide; it is the dependency of Spring Boot, which has hundreds of millions of downloads per month.

## Current State
- **Stability:** Active development; version 7.1.0-SNAPSHOT targets Jakarta EE 11 APIs and Java 17+ baseline.
- **Known issues:** See [GitHub Issues](https://github.com/spring-projects/spring-framework/issues); ongoing work on native image improvements, Kotlin coroutine integration, and virtual-thread support.
- **General health:** Excellent — comprehensive test suite (3,900+ test files), daily CI matrix builds, Develocity-backed build caching, active Spring team maintainership.

## High-Level Risks
- **API surface size** — 9,224 Java + 389 Kotlin source files across 22 modules; changes to core abstractions (`BeanFactory`, `ApplicationContext`, `DispatcherServlet`) have enormous downstream blast radius.
- **Backwards-compatibility contract** — every minor release must preserve binary and behavioural compatibility for thousands of downstream Spring projects and end-user applications.

## Ownership
- **Team:** VMware/Broadcom Spring team (spring-projects GitHub org)
- **Tech Lead:** Juergen Hoeller (core), Rossen Stoyanchev (web), Sam Brannen (test) — per CONTRIBUTING.md and git history
- **Product Owner:** Spring team / Broadcom

## Related Systems
- **Spring Boot** — opinionated auto-configuration layer built on top of this framework
- **Spring Security** — security framework that plugs into Spring MVC/WebFlux filter chains
- **Spring Data** — repository abstraction built on spring-tx and spring-jdbc
- **Spring Cloud** — distributed-system primitives extending Spring context and messaging
- **GraalVM Native Image** — compilation target enabled by the AOT processing infrastructure in spring-core and spring-context
