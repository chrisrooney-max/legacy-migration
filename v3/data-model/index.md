# Data Model

## Overview
Spring Framework is not a data storage system — it is an application framework library. Its "data model" is the in-memory object model of the IoC container: `BeanDefinition` objects describing beans, `ApplicationContext` as the runtime registry, and the type metadata structures (`ResolvableType`, `MethodParameter`) used throughout. For data access, it provides abstractions over external databases rather than owning a schema.

## Core Entities

| Entity | Description | Owner | Notes |
|--------|------------|------|------|
| `BeanDefinition` | Metadata describing a Spring-managed bean: class, scope, constructor args, property values, init/destroy methods | spring-beans | `RootBeanDefinition`, `GenericBeanDefinition` are primary impls |
| `ApplicationContext` | Runtime registry of all bean instances and definitions; scoped to the application lifecycle | spring-context | `AnnotationConfigApplicationContext` is most common |
| `BeanFactory` | Lowest-level container; lazily creates beans on demand | spring-beans | `DefaultListableBeanFactory` is the primary implementation |
| `ResolvableType` | Rich type descriptor wrapping `java.lang.reflect.Type`; resolves generics | spring-core | Immutable value object; used pervasively for type matching |
| `MethodParameter` | Descriptor for a method or constructor parameter including annotations and generic type | spring-core | Used by `HandlerMethodArgumentResolver`, injection infrastructure |
| `Message<T>` | Immutable container of payload + `MessageHeaders` in the messaging pipeline | spring-messaging | Shared by STOMP, JMS, RSocket adapters |
| `TransactionDefinition` | Specifies isolation, propagation, timeout, and read-only flag for a transaction | spring-tx | Implemented by `@Transactional` attribute metadata |
| `RuntimeHints` | AOT-time record of reflection, resource, serialisation, and proxy hints for native image | spring-core | New in 6.x; written at build time, read by GraalVM |
| `ModelAndView` | MVC view name + model map returned by `@Controller` handler methods | spring-webmvc | Mutable; often replaced by `@ResponseBody` in modern apps |
| `ServerWebExchange` | Reactive equivalent of `HttpServletRequest`/`Response`; immutable request + mutable response | spring-webflux | Central to WebFlux handler chain |

## Relationships
- `ApplicationContext` contains many `BeanDefinition`s (1:N)
- `BeanDefinition` references other `BeanDefinition`s via dependency edges (used for dependency resolution and circular-dependency detection)
- `Message<T>` carries arbitrary payload typed by `T`; `MessageHeaders` is a read-only `Map<String, Object>`
- `TransactionStatus` is produced by `PlatformTransactionManager.getTransaction(TransactionDefinition)` — 1:1 per active transaction per thread
- `ResolvableType` wraps Java reflection types; forms trees for nested generic types

## Data Storage
- **Database(s):** None owned by the framework. spring-jdbc/spring-orm/spring-r2dbc connect to user-configured `DataSource`/`ConnectionFactory`.
- **Type (SQL/NoSQL/etc):** Framework is data-store agnostic; supports any JDBC-compatible RDBMS, JPA provider (Hibernate), and R2DBC-compatible store.

## Key Tables / Collections

| Name | Purpose | Key Fields | Risk |
|------|--------|-----------|------|
| `DefaultListableBeanFactory.beanDefinitionMap` | In-memory map of bean name → `BeanDefinition` | `beanDefinitionNames`, `beanDefinitionMap` (ConcurrentHashMap) | Thread-safety managed by framework; do not mutate after refresh |
| `DefaultListableBeanFactory.singletonObjects` | Singleton bean cache | bean name → object | Long-lived; GC roots for entire app graph |
| `DefaultContextCache` (spring-test) | Caches `ApplicationContext` across test classes by config key | `contextCache` (LinkedHashMap with LRU eviction) | Memory pressure in large test suites |
| `MessageHeaderAccessor.headers` | Mutable headers during message construction | header name → value | Frozen to `MessageHeaders` on send |

## Data Lifecycle
- **Creation:** `BeanDefinition`s created at context startup (refresh); singleton beans instantiated eagerly (or lazily on first access).
- **Updates:** Bean definitions are immutable post-refresh. Singleton instances are mutable (by the application).
- **Deletion:** Beans destroyed when `ApplicationContext.close()` is called, triggering `@PreDestroy` and `DisposableBean.destroy()`.

## Data Quality Issues
- Circular bean dependency detection relies on a creation-in-progress set; constructor injection cycles are hard failures; setter injection cycles are resolved but discouraged
- `ResolvableType` resolution can fail silently (returns `NONE`) for erased generics at runtime

## Reporting Dependencies
- N/A — the framework does not produce business reports; no ETL or analytics dependencies

## Migration Risks
- `BeanDefinition` metadata from XML config must be migrated to annotation config during any Spring 5 → 6/7 upgrade
- The `javax.*` → `jakarta.*` namespace migration (Spring 6.0) is a hard breaking change for any consumer still on Servlet 5 / pre-Jakarta dependencies
- AOT processing output (generated source in `spring-aot/` during build) is environment-specific and must be regenerated for each target
