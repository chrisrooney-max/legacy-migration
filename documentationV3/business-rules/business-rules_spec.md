# Business Rules

## Overview
Spring Framework's "business rules" are the contract rules that govern how the IoC container, AOP proxy system, transaction management, and web dispatch behave. These are codified in the framework itself — they are the rules that all consumer applications must respect. Violations typically result in runtime errors (`NoSuchBeanDefinitionException`, `BeanCurrentlyInCreationException`, transaction rollback surprises) that are notoriously difficult to diagnose.

## Core Rules

| Rule | Description | Location in Code | Notes |
|------|------------|-----------------|------|
| Singleton scope by default | Beans are singletons unless `@Scope` specifies otherwise; singleton beans are shared across the entire `ApplicationContext` | `DefaultListableBeanFactory`, `AbstractBeanFactory` | Breaking this assumption (mutable singletons) causes thread-safety bugs |
| Proxy self-invocation bypass | Calling a `@Transactional` (or any AOP-advised) method from within the same bean bypasses the proxy — the advice does not run | `CglibAopProxy`, `JdkDynamicAopProxy` | Well-known gotcha; use `AopContext.currentProxy()` or restructure into a separate bean |
| `@Transactional` only on public methods | AOP proxy-based `@Transactional` only intercepts public methods; private/protected methods are not wrapped | `AbstractFallbackTransactionAttributeSource` | Annotation on private method is silently ignored |
| Circular constructor injection fails | Constructor-injection circular dependencies throw `BeanCurrentlyInCreationException` at context refresh; setter-injection cycles are resolved (with a warning) | `DefaultSingletonBeanRegistry` | Prefer redesign over setter-cycle workaround |
| `ApplicationContext` refresh is one-shot | Once `refresh()` is called, the context configuration is effectively frozen; hot re-registration of beans is not supported in standard contexts | `AbstractApplicationContext.refresh()` | `GenericApplicationContext` forbids multiple refreshes |
| `@Bean` methods in `@Configuration` classes are proxied | `@Configuration` classes are CGLIB-proxied so that inter-`@Bean` calls return the same singleton instance; `@Component`-annotated config classes are NOT proxied (lite mode) | `ConfigurationClassPostProcessor` | Subtle difference between full `@Configuration` and lite mode |
| `@EventListener` ordering | Event listeners fire synchronously in the calling thread by default; `@Async` listeners run in a task executor but lose the original transaction context | `SimpleApplicationEventMulticaster` | Async listeners must not assume transaction context |
| PropertySource override order | `Environment` property sources are searched in order; later-added sources override earlier ones; `@PropertySource` has lower priority than system/OS env vars | `MutablePropertySources`, `PropertySourcesPropertyResolver` | Source order determines which value wins for the same key |
| `HandlerMapping` priority | `DispatcherServlet` iterates handler mappings in `Ordered` priority; first match wins | `DispatcherServlet.getHandler()` | Custom mappings at `Integer.MIN_VALUE` take priority over `RequestMappingHandlerMapping` |
| Transaction rollback on `RuntimeException` only | `@Transactional` rolls back on `RuntimeException` and `Error` by default; checked exceptions commit unless `rollbackFor` is specified | `DefaultTransactionAttribute` | Checked exceptions committing is a common source of data integrity bugs |

## Workflows

**Bean Lifecycle (Singleton):**
1. `BeanDefinition` registered (scan or explicit)
2. `BeanFactoryPostProcessor` runs (e.g., `PropertySourcesPlaceholderConfigurer` resolves `${...}` in definitions)
3. Bean instantiated (constructor or factory method)
4. Dependencies injected (setter / field injection)
5. `BeanPostProcessor.postProcessBeforeInitialization()` (e.g., `@PostConstruct` via `CommonAnnotationBeanPostProcessor`)
6. `InitializingBean.afterPropertiesSet()` / `@Bean(initMethod)`
7. `BeanPostProcessor.postProcessAfterInitialization()` (AOP proxy wrapping happens here)
8. Bean ready for use
9. On `ApplicationContext.close()`: `@PreDestroy` → `DisposableBean.destroy()` → `@Bean(destroyMethod)`

**Request Dispatch (MVC):**
1. `DispatcherServlet.service()` receives `HttpServletRequest`
2. `HandlerMapping.getHandler()` returns `HandlerExecutionChain` (handler + interceptors)
3. `HandlerInterceptor.preHandle()` for each interceptor
4. `HandlerAdapter.handle()` invokes the controller method
5. `HandlerInterceptor.postHandle()`
6. `ViewResolver.resolveViewName()` (or `HttpMessageConverter` for `@ResponseBody`)
7. `HandlerInterceptor.afterCompletion()`
8. Exception path: `HandlerExceptionResolver` chain processes any thrown exception

**Transaction Boundary:**
1. Call to `@Transactional` method reaches AOP proxy
2. `TransactionInterceptor` calls `PlatformTransactionManager.getTransaction()`
3. Method executes in transactional context (DataSource connection bound to thread)
4. On normal return: `commit()`
5. On `RuntimeException`/`Error`: `rollback()`
6. On checked exception: `commit()` (unless `rollbackFor` specified)

## Validations
- Bean name uniqueness enforced by `DefaultListableBeanFactory` — duplicate registration throws `BeanDefinitionOverrideException` (override allowed only if explicitly enabled via `setAllowBeanDefinitionOverriding(true)`)
- `@Autowired` with `required=true` (default) fails at context refresh if no matching bean found — eager validation catches wiring errors at startup
- `@Value("${property.key}")` with no default throws `IllegalArgumentException` at injection time if the property is missing

## Edge Cases
- Prototype-scoped bean injected into singleton: the singleton receives a single prototype instance at creation time; subsequent calls get the same stale instance unless `ObjectProvider` / `@Lookup` is used
- `@Transactional` on `@Async` methods: the async task runs in a new thread; the transaction does NOT propagate from the caller (new transaction is started unless method is also `@Transactional`)
- Kotlin `open` requirement: Spring AOP proxies require the target class and methods to be `open`; Kotlin's `kotlin-spring` compiler plugin applies `open` automatically to `@Component` classes
- Abstract `@Configuration` classes: supported; methods can be `@Bean`-annotated; useful for shared config hierarchies

## Exceptions
- `BeanCurrentlyInCreationException` — circular constructor dependency detected; application must redesign or use `@Lazy`
- `NoSuchBeanDefinitionException` — referenced bean not found; check component scan configuration
- `BeanDefinitionOverrideException` — duplicate bean name detected; enable override explicitly or rename
- `NoUniqueBeanDefinitionException` — multiple candidates for `@Autowired` injection; use `@Qualifier` or `@Primary`
- `LazyInitializationException` (Hibernate, via spring-orm) — accessing a lazy-loaded association outside a transaction; verify `@Transactional` scope

## Time-Based Logic
- **Schedules:** `@Scheduled` in spring-context-support supports cron expressions, fixed-rate, and fixed-delay; cron uses `CronExpression` parser (Java 8 `java.time` based in Spring 5.3+)
- **Timezones:** `@Scheduled(cron="...", zone="UTC")` supports explicit timezone; defaults to server default timezone — this is a common source of environment-specific bugs in international deployments

## Inconsistencies
- `@Transactional` on a `@Repository` bean: exception translation (`PersistenceExceptionTranslationPostProcessor`) and transaction management are two separate AOP advisors — both wrap the proxy, and ordering matters; Spring Boot auto-configures the correct order
- `@EnableWebMvc` and Spring Boot auto-configuration conflict: importing `@EnableWebMvc` in a Spring Boot app disables MVC auto-configuration entirely — documented but frequently misunderstood

## Risks
- SpEL expression complexity — deeply nested or dynamically constructed SpEL expressions are difficult to test exhaustively; runtime evaluation errors surface as `SpelEvaluationException` which can be cryptic
- Prototype beans in singleton scope — misuse of prototype scope (injecting directly into a singleton) is a well-documented but still common mistake that silently produces stale instances
- `@Transactional` self-invocation — the proxy bypass rule is not enforced at compile time; static analysis tools (SpotBugs Spring plugin) can help but are not mandatory
