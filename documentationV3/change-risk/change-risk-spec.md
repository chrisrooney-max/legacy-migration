# Change Risk & Technical Debt

## Overview
Spring Framework carries a unique risk profile: it is consumed as a library by tens of thousands of downstream projects and applications. Changes to core abstractions or behaviour ripple through the entire Spring ecosystem. The framework team maintains strict binary and behavioural compatibility within a minor version. The codebase is large (9,600+ source files) but well-structured, with test coverage acting as the primary safety net.

## High-Risk Areas

| Area | Description | Reason | Impact |
|------|------------|--------|--------|
| `BeanFactory` / `DefaultListableBeanFactory` | Core DI engine; instantiation, wiring, circular-dependency detection | Virtually every module depends on it; any behavioural change affects all consumer apps | Critical — ecosystem-wide blast radius |
| `DispatcherServlet` | Single entry point for all Spring MVC HTTP traffic | ~1,200 lines; handles handler mapping, exception resolution, view rendering, async dispatch | High — any bug affects all MVC applications |
| `AbstractApplicationContext.refresh()` | Context startup sequence; bean lifecycle, post-processors, event multicasting | Extremely broad surface; hook points used by Spring Boot auto-configuration, Spring Security, Spring Data | High |
| AOP proxy creation (`ProxyFactory`, CGLIB) | Creates runtime proxies for `@Transactional`, `@Async`, security interceptors | CGLIB proxies are sensitive to final classes, private methods, constructors; Kotlin data classes add edge cases | High — silent proxy bypass can cause data corruption |
| Kotlin coroutine bridge (`CoroutinesUtils`) | Suspending function → `Mono`/`Flux` bridging | Kotlin coroutine API evolves quickly; bridge logic is intricate and failure-prone | Medium |
| Spring AOT / `RuntimeHints` | Native image hint collection at build time | Still maturing (introduced 6.0); incomplete coverage causes runtime failures on GraalVM | Medium — growing criticality as native adoption increases |
| `spring-oxm` XStream integration | XStream marshaller | XStream has active CVEs; the integration is low-maintenance but present in the codebase | Medium — security surface |

## Known Fragile Components
- `ClassPathScanningCandidateComponentProvider` — classpath scanning is inherently environment-dependent; fails or finds unexpected results in custom classloader hierarchies (OSGi, servlet containers with isolated classloaders)
- `FrameworkServlet` / `HttpServletBean` — base servlet hierarchy accumulates many configuration properties across releases; ordering of `init()` callbacks can be surprising
- `DataSourceUtils` / `TransactionSynchronizationManager` — thread-local transaction binding; fragile under virtual threads if context is lost across thread hops
- `groovy` bean scripting in spring-beans — Groovy 5.x dependency; minimal test coverage; rarely used in production

## Coupling Issues
- `spring-context` depends on `spring-core`, `spring-beans`, `spring-aop` — any change to `spring-beans` interfaces propagates up to `spring-context` and all feature modules
- `spring-webmvc` depends on `spring-web`, `spring-context`, `spring-beans`, `spring-aop` — it is the most coupled feature module
- `@Transactional` is implemented as an AOP proxy — coupling between spring-tx and spring-aop is intentional but means transaction behaviour is proxy-visibility-dependent

## Technical Debt

| Item | Description | Priority |
|------|------------|---------|
| XML configuration support | `XmlBeanDefinitionReader`, Groovy DSL bean scripting — still supported but no longer idiomatic; large maintenance surface | Low (but can't be removed due to compatibility) |
| `spring-oxm` Castor/XStream marshallers | Castor is unmaintained; XStream has CVEs; both are optional but ship in the module | Medium |
| `RestTemplate` | Synchronous HTTP client; replaced by `RestClient` in 6.1 but still supported; two HTTP client APIs to maintain | Low-Medium |
| `JdbcDaoSupport` / `HibernateDaoSupport` | DAO support classes — legacy pattern; modern usage prefers plain `JdbcTemplate` injection | Low |
| SpEL runtime evaluation security | SpEL expressions from untrusted input are a security risk; framework does not restrict evaluation by default | Medium — documented as consumer responsibility |
| Virtual thread compatibility | `TransactionSynchronizationManager` uses `ThreadLocal`; `InheritableThreadLocal` used in some places — not fully virtual-thread-safe | Medium — Java 21+ concern |

## Obsolete Technology
- **Castor XML** (spring-oxm) — unmaintained; last release 2016
- **XStream** (spring-oxm) — multiple CVEs; avoid unless explicitly needed
- **Groovy DSL bean definitions** — `GroovyBeanDefinitionReader`; used by almost nobody in modern applications
- **Commons FileUpload** — replaced by Servlet 3+ `Part` API and multipart handling in spring-web; legacy path still present

## Safe-to-Change Areas
- `spring-test` — test utilities; changes do not affect production runtime behaviour; only impacts test code
- `spring-context-support` — optional integrations (Quartz, mail, caching); narrow consumers; lower coupling
- `spring-instrument` — standalone agent JAR; no compile-time dependencies from other modules
- `framework-docs` — documentation only; zero runtime impact
- `spring-expression` standalone parsing/evaluation — safe to add new operators/functions without breaking existing expressions

## Recommendations
- Prioritise virtual-thread safety audit across `ThreadLocal` usage in spring-tx and spring-web before production Java 21+ deployments
- Remove or isolate `spring-oxm` XStream support behind an opt-in flag; prevent CVE exposure in default classpath
- Add `@SuppressWarnings("deprecation")` hygiene checks to enforce migration away from `RestTemplate` toward `RestClient` in new code
- Expand AOT test coverage — automated checks that all standard `@SpringBootTest` application contexts can be processed with AOT and compiled to native images
