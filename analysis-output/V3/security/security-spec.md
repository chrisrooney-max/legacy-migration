# Security

## Overview
Spring Framework is an application framework library — it does not directly enforce authentication or authorisation at runtime (that is Spring Security's domain). However, it provides infrastructure that has significant security implications: SpEL expression evaluation, HTTP client configuration, XML parsing, AOP proxy-based interceptors, and the trust model around `BeanFactory` configuration. The project has a published security policy and uses GitHub security advisories for CVE disclosure.

## Authentication
- **Method:** Spring Framework itself does not implement authentication. It provides integration points: `OncePerRequestFilter`, servlet filter chains in `spring-webmvc`, `WebFilter` in spring-webflux — all consumed by Spring Security.
- **Flow:** An authentication filter (provided by Spring Security or custom code) runs before `DispatcherServlet` / `DispatcherHandler`. Spring Framework provides the HTTP infrastructure; authentication logic is outside its scope.

## Authorization
- **Roles:** Not enforced by the framework; `@PreAuthorize` / `@PostAuthorize` are annotations processed by Spring Security's AOP advisor, not by this framework directly.
- **Permissions:** Spring Security's method security uses Spring AOP proxies created by this framework — the proxy infrastructure is a Spring Framework responsibility, the access-decision logic is Spring Security's.

## Sensitive Data
- **Types:** No PII or credentials are processed by the framework itself at runtime. Configuration properties may contain credentials (`DataSource` passwords, JMS credentials) — the framework provides `EnvironmentPostProcessor` / `PropertySourcesPropertyResolver` for reading them without hard-coding.
- **Storage:** `Environment` / `PropertySources` hold config values in memory; no persistence by the framework.

## Data Protection
- **Encryption:** The framework does not encrypt data at rest or in transit. `spring-web` `RestClient`/`RestTemplate`/`WebClient` rely on the underlying HTTP client's TLS configuration (JDK `HttpClient`, Apache HttpComponents, Reactor Netty — all support TLS/mTLS configuration).
- **Masking:** No built-in credential masking in logging; consumers must configure Log4j/SLF4J pattern layouts to mask sensitive values.

## Audit Logging
- **What is logged:** The framework logs bean creation, context refresh events, handler mapping resolution (at DEBUG/TRACE), and exceptions. No security-specific audit log is produced by the framework itself.
- **Log level guidance:** Do not run at TRACE in production — trace logs can expose bean property values and request parameters.

## Vulnerabilities
- **Known issues:**
  - `spring-oxm` ships XStream and Castor bindings; XStream has historical RCE CVEs (e.g., CVE-2021-21344 family). The framework does not use XStream internally but exposes it as an optional marshaller. Consumers must manage XStream version or exclude it.
  - SpEL expressions evaluated against untrusted user input are a documented injection risk. The framework does not sandbox SpEL evaluation. Applications must validate or sanitise inputs before passing to `ExpressionParser`.
  - Multi-part file upload via `CommonsMultipartResolver` (deprecated, uses Commons FileUpload) has a history of DoS vulnerabilities; consumers should use Servlet 3 `StandardServletMultipartResolver` instead.

## Dependencies Risk
- **XStream** (via spring-oxm) — active CVE history; exclude if not using XML marshalling
- **Commons FileUpload** (legacy `CommonsMultipartResolver`) — DoS-vulnerable versions; replace with standard multipart handling
- **groovy-bom 5.0.4** (optional script beans) — Groovy RCE CVEs have occurred historically; avoid script bean features unless explicitly needed and sandboxed
- **Aalto-XML, Woodstox** (XML parsing) — XML entity expansion / XXE risks; framework configures `StaxUtils` to disable external entity processing by default

## Compliance
- **Security policy:** Published at `SECURITY.md` — vulnerabilities must be reported via GitHub Draft Security Advisory to keep disclosures private until patched.
- **JAR signing:** All GA release JARs are GPG-signed; key at `https://spring.io/GPG-KEY-spring.txt`. Consumers can verify integrity of published artefacts.
- **GDPR / HIPAA / etc.:** Not directly applicable to a framework library; compliance is the responsibility of the consuming application. The framework does not collect, store, or transmit user data.
- **CVE coordination:** Spring team coordinates with the security community via GitHub security advisories; patch releases are published for supported branches (see `https://spring.io/security-policy` for supported versions).

## Recommendations
- Audit all SpEL expressions that accept user-controlled input — use `SimpleEvaluationContext` instead of `StandardEvaluationContext` to restrict the evaluation surface
- Exclude or update `com.thoughtworks.xstream:xstream` if not actively used by the application
- Replace any `CommonsMultipartResolver` usage with `StandardServletMultipartResolver`
- Configure external entity processing restrictions on any custom `DocumentBuilderFactory` / `XMLInputFactory` instances (the framework does this for its own parsers via `StaxUtils`)
- Pin all dependency versions explicitly via `framework-platform` BOM to avoid transitive CVE exposure from version ranges
