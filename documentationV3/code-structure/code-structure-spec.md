# Code Structure

## Repository Overview
- **Repo name:** spring-projects/spring-framework
- **Purpose:** Multi-module Gradle monorepo containing the entire Spring Framework source, tests, and documentation

## Directory Structure
```
spring-framework/
├── spring-core/              # Type system, resource IO, AOT hints, Kotlin utils
├── spring-beans/             # BeanFactory, BeanDefinition, DI engine
├── spring-context/           # ApplicationContext, annotation config, events
├── spring-context-support/   # Mail, Quartz scheduling, caching integrations
├── spring-context-indexer/   # Annotation processor for component index
├── spring-aop/               # AOP framework, proxy creation
├── spring-aspects/           # AspectJ-based aspects
├── spring-expression/        # Spring Expression Language (SpEL)
├── spring-web/               # Shared web abstractions, RestClient
├── spring-webmvc/            # Servlet-based MVC (DispatcherServlet)
├── spring-webflux/           # Reactive web (DispatcherHandler)
├── spring-websocket/         # WebSocket + STOMP
├── spring-messaging/         # Message pipeline, RSocket, JMS adapters
├── spring-tx/                # Transaction abstraction
├── spring-jdbc/              # JdbcTemplate, DataSource utils
├── spring-orm/               # JPA/Hibernate integration
├── spring-r2dbc/             # Reactive relational DB (R2DBC)
├── spring-jms/               # JMS template and listener
├── spring-oxm/               # Object/XML mapping
├── spring-test/              # TestContext framework, MockMvc
├── spring-instrument/        # ClassLoader agent for LTW
├── spring-core-test/         # Test utilities for core module
├── framework-api/            # Aggregated public API JAR
├── framework-bom/            # Bill of Materials POM
├── framework-docs/           # Asciidoc reference documentation
├── framework-platform/       # Dependency management platform
├── integration-tests/        # Cross-module integration tests
├── buildSrc/                 # Gradle build logic (conventions plugins)
└── .github/                  # CI workflows, issue templates
```

## Key Modules

| Module | Responsibility | Entry Points | Notes |
|--------|--------------|-------------|------|
| spring-core | Type introspection, `ResolvableType`, `Resource` abstraction, `ReactiveAdapterRegistry`, AOT `RuntimeHints` | `SpringVersion`, `ReactiveAdapterRegistry` | Multi-release JAR; 1,086 source files |
| spring-beans | IoC engine — `BeanFactory`, `BeanDefinition`, `AutowiredAnnotationBeanPostProcessor` | `BeanFactory`, `BeanDefinitionRegistry` | Foundation for everything above |
| spring-context | `AnnotationConfigApplicationContext`, `ClassPathBeanDefinitionScanner`, `@ComponentScan`, `@EventListener` | `ApplicationContext`, `ConfigurableApplicationContext` | 1,248 source files |
| spring-aop | `ProxyFactory`, `AopProxyUtils`, `AspectJExpressionPointcut` | `ProxyFactory`, `Advisor`, `Pointcut` | CGLIB + JDK dynamic proxies |
| spring-web | `RestClient`, `RestTemplate`, `WebClient` (builder), content negotiation, `HttpMethod`, `ApiVersionStrategy` | `RestClient.Builder`, `RestTemplate` | API versioning feature new in 7.x |
| spring-webmvc | `DispatcherServlet`, `@Controller`, `@RequestMapping`, view resolution, `MockMvc` builder | `DispatcherServlet` | Servlet 6 (Jakarta EE 11) |
| spring-webflux | `DispatcherHandler`, `RouterFunction`, `WebFilter`, `@RestController` on reactive stack | `DispatcherHandler`, `RouterFunction` | Reactor Netty or Servlet async |
| spring-tx | `@Transactional`, `PlatformTransactionManager`, `TransactionTemplate`, reactive tx | `PlatformTransactionManager` | Supports virtual threads |
| spring-jdbc | `JdbcTemplate`, `NamedParameterJdbcTemplate`, `DataSourceUtils` | `JdbcTemplate` | Thin JDBC wrapper |
| spring-test | `TestContextManager`, `@SpringJUnitConfig`, `MockMvc`, `WebTestClient`, `@MockitoBean` | `@SpringJUnitConfig`, `MockMvc` | 1,370 source files; largest module |

## Entry Points
- **API:** No HTTP server — library only. Entry points are `ApplicationContext` implementations and `DispatcherServlet`/`DispatcherHandler` for web stacks.
- **Jobs:** No built-in schedulers are run by the framework itself; `spring-context` provides `@Scheduled` support for consumer apps.
- **CLI:** No CLI. Build via `./gradlew build`.

## Configuration
- **Where config lives:** `gradle.properties` (versions), `buildSrc/` (Gradle convention plugins), `framework-platform/framework-platform.gradle` (BOM/dependency management)
- **Environment variables:** `DEVELOCITY_ACCESS_KEY` (build scans, CI only), standard Java toolchain env vars (`JAVA_HOME`)

## Coding Patterns
- **Interface-first design** — every major abstraction has an interface (`BeanFactory`, `ApplicationContext`, `HandlerMapping`), a configurable sub-interface, and one or more concrete implementations
- **Template Method pattern** — widely used (`AbstractApplicationContext`, `FrameworkServlet`, `JdbcTemplate`) to define skeleton algorithms with extension hooks
- **Strategy pattern** — pluggable strategies everywhere: `HandlerMapping`, `ViewResolver`, `ContentNegotiationStrategy`, `TransactionManager`
- **Decorator/Composite** — `CompositeContentNegotiationStrategy`, `CompositeMessageConverter`, layered `HandlerMapping` chains
- **Nullability annotations** — `@Nullable`/`@NonNull` (via `io.spring.nullability` plugin) on all public APIs from Spring 6.2+
- **Kotlin coroutine suspension** — `CoroutinesUtils`, `ReactiveAdapterRegistry` bridging `suspend fun` to `Mono`/`Flux`

## Dependencies
- **Internal:** Strict directed acyclic graph; leaf modules (spring-webmvc, spring-jdbc) depend on foundation modules (spring-core, spring-beans, spring-context); no circular dependencies
- **External:** Managed centrally in `framework-platform/framework-platform.gradle`; key BOMs: Reactor, Netty, Jakarta EE, Kotlin, Jackson, JUnit

## Areas of Concern
- **spring-beans and spring-context** — extremely high coupling surface; almost all modules depend on them; changes propagate widely
- **spring-core AOT module** — `RuntimeHints`, `AotProcessor` infrastructure is relatively new (added in 6.x) and still evolving; GraalVM coverage incomplete
- **spring-webmvc DispatcherServlet** — central single point of entry for MVC; contains decades of accumulated configuration surface; `DispatcherServlet` class is ~1,200 lines
- **spring-oxm** — depends on XStream (known CVEs) and JAXB; considered low-priority maintenance burden
