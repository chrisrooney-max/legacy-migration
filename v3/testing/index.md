# Testing & Quality

## Testing Strategy
Spring Framework has one of the most comprehensive test suites in the Java open-source ecosystem. Testing is treated as a first-class concern — the `spring-test` module ships as a production artifact specifically to enable testing of Spring-based applications. The framework's own tests validate every layer from unit-level bean definitions to full integration scenarios using embedded servers.

## Test Types

- **Unit:** Isolated tests for individual classes (e.g., `ResolvableTypeTests`, `AnnotationUtilsTests`). Use Mockito/MockK for collaborators. Found in `src/test/java` of each module.
- **Integration:** Tests that load an `ApplicationContext` with real beans and optionally an embedded database (H2) or embedded server. Annotated with `@SpringJUnitConfig` or `@SpringBootTest`-style context loading. `@Transactional` with rollback on test completion.
- **MockMvc / WebTestClient:** `spring-test` provides `MockMvc` for Servlet MVC integration tests (no running server) and `WebTestClient` for WebFlux tests against a `WebApplicationContext` or running server.
- **Spring MVC Test (server-side):** Tests via `MockDispatcherServlet` — covers handler mapping, argument resolution, response serialisation.
- **End-to-end / cross-module:** `integration-tests/` subproject runs tests that span multiple modules; uses embedded Tomcat/Jetty/Netty.
- **Performance (JMH micro-benchmarks):** Selected modules use the `me.champeau.jmh` Gradle plugin for micro-benchmark suites (e.g., `ResolvableType`, `BeanFactory` creation).

## Coverage

| Area | Coverage Level | Notes |
|------|---------------|------|
| spring-core | High | 1,086 source files; extensive `*Tests.java` files for all utilities |
| spring-beans | High | BeanFactory and DI engine extensively tested; edge cases for circular deps, scopes |
| spring-context | High | 1,248 source files; annotation config, event publishing, lifecycle all tested |
| spring-aop | High | Proxy creation, pointcut matching, advice ordering |
| spring-webmvc | High | `DispatcherServlet`, handler mapping, view resolution, REST controllers all tested with MockMvc |
| spring-webflux | High | `DispatcherHandler`, `RouterFunction`, WebFilter chain, `WebTestClient` integration |
| spring-tx | High | Transaction propagation, rollback, `@Transactional` semantics with embedded DB |
| spring-jdbc | High | `JdbcTemplate` operations against H2 embedded database |
| spring-test | High | The testing module tests itself; TestContext framework, `@MockitoBean`, `MockMvc` |
| spring-orm | Medium | JPA integration; requires Hibernate 7.x on classpath; some tests marked `@Disabled` without container |
| spring-r2dbc | Medium | R2DBC tested against H2 R2DBC driver |
| spring-oxm | Low | Minimal tests; Castor and XStream marshallers have limited coverage |
| AOT / native image | Low-Medium | `RuntimeHints` generation tests exist; full GraalVM compilation not run in standard CI |

Total test files: ~3,900 `*Test.java` / `*Tests.java` files across all modules.

## Critical Test Scenarios
- `DefaultListableBeanFactory` circular dependency detection — covered in `DefaultListableBeanFactoryTests`
- `@Transactional` rollback on `RuntimeException` and `Error` — `TransactionalTestExecutionListenerTests`
- `DispatcherServlet` handler mapping priority ordering — `DispatcherServletTests`
- `@MockitoBean` replacement of a real bean in the `ApplicationContext` — `MockitoBeanTests` in spring-test
- WebFlux `RouterFunction` composition and `WebFilter` ordering — `RouterFunctionTests`
- AOP proxy for `final` classes (CGLIB limitation) and Kotlin `data class` (open plugin requirement)
- Multi-release JAR selection — verified by building with Java 21 and Java 24 toolchains

## Manual Testing
- AOT processing followed by GraalVM native image compilation — not automated in CI; requires GraalVM Community toolchain
- Browser-based WebSocket/STOMP scenarios — functional testing via `spring-messaging` samples
- Servlet container compatibility (Tomcat 11, Jetty 12, Undertow 2) — `integration-tests/` covers embedded containers; external container compatibility validated by Spring team and community

## Gaps
- GraalVM native image test coverage — AOT hints exist but full native compilation is not run as part of standard CI
- Full Kotlin coroutine/flow interop edge cases — some scenarios only tested on JVM without coroutine dispatcher variations
- `spring-oxm` Castor and XStream marshallers — low test density; Castor unmaintained
- Groovy scripted beans (`GroovyBeanDefinitionReader`) — minimal CI coverage; Groovy 5 compatibility uncertain

## Release Validation
1. Full `./gradlew build` on Java 17, 21, and 25 (CI matrix must be green)
2. All `integration-tests` pass with embedded Tomcat, Jetty, Netty
3. Develocity build scan shows no test regressions vs previous release
4. Spring Boot snapshot compatibility run (Spring Boot CI consumes SNAPSHOT and reports failures)
5. Security advisory review (no open critical CVEs in shipped dependencies)
6. Signed JAR verification via GPG key published at `https://spring.io/GPG-KEY-spring.txt`

## Known Issues
- Some JPA tests in spring-orm require `hibernate-core` on classpath; marking `@Disabled` when not present makes it easy to accidentally miss regressions
- `@DirtiesContext` tests slow down CI significantly when heavily used in the test suite; test context caching is critical for reasonable build times
