# Operations

## Running Locally

```bash
# Clone (shallow for speed)
git clone --depth 1 https://github.com/spring-projects/spring-framework.git
cd spring-framework

# Build all modules and run tests (requires JDK 17+)
./gradlew build

# Build without tests
./gradlew build -x test

# Build a specific module
./gradlew :spring-webmvc:build

# Run a module's tests
./gradlew :spring-context:test
```

IDE import: see `import-into-intellij-idea.md` or `import-into-eclipse.md` in the repo root. IntelliJ is the primary supported IDE; Develocity build scans require `DEVELOCITY_ACCESS_KEY` to publish.

## Build & Deploy

- **Build tool:** Gradle 8.x (wrapper at `./gradlew`); multi-project build with convention plugins in `buildSrc/`
- **Build steps:**
  1. `./gradlew build` — compiles all modules, runs unit + integration tests, generates Javadoc
  2. Gradle build cache (`org.gradle.caching=true` in `gradle.properties`) accelerates incremental builds
  3. Parallel execution enabled (`org.gradle.parallel=true`)
- **Deployment process:**
  - **Snapshots:** Pushed to `repo.spring.io/snapshot` automatically on every merge to `main` via `build-and-deploy-snapshot.yml` workflow using `spring-io/artifactory-deploy-action`
  - **Milestones:** Triggered manually via `release-milestone.yml`; artifacts signed and published to Maven Central staging
  - **GA releases:** Triggered via `release.yml`; full release to Maven Central; JAR signing with Spring GPG key (`https://spring.io/GPG-KEY-spring.txt`)
  - **Docs:** Deployed via `deploy-docs.yml`; Antora-based reference docs published to `docs.spring.io`

## Environments

| Environment | Purpose | Notes |
|------------|--------|------|
| Developer local | Build, test, experiment | JDK 17+ required; JDK 21/25 for full test matrix |
| GitHub Actions (CI) | Automated build & test | ubuntu-latest; matrix: Java 17, 21, 25 |
| repo.spring.io/snapshot | Snapshot artefact hosting | Consumed by Spring Boot SNAPSHOT builds |
| Maven Central | GA and milestone release distribution | Requires staged upload + release via Sonatype |
| docs.spring.io | Published reference documentation | Antora + Asciidoc; updated per release |

## Configuration
- **Env variables (CI):**
  - `DEVELOCITY_ACCESS_KEY` — Gradle Enterprise build scan publishing
  - `GITHUB_TOKEN` — PR/issue labelling (backport-bot, DCO check)
  - `GOOGLE_CHAT_WEBHOOK_URL` — build failure notifications
  - `ARTIFACTORY_USERNAME` / `ARTIFACTORY_PASSWORD` — snapshot artefact deployment
- **Secrets:** No application runtime secrets; all config is build-time. JAR signing key managed by VMware/Broadcom Spring team.

## Monitoring
- **Tools:** Develocity (Gradle Enterprise) at `ge.spring.io` — build scan analytics, test distribution, remote cache
- **Key metrics:**
  - Build success/failure rate per branch (tracked in GitHub Actions)
  - Test pass rate by module
  - Build scan URI recorded to `build/build-scan-uri.txt` after each CI run

## Logging
- **Where logs are stored:** GitHub Actions job logs; build scan linked in PR checks
- **Log format:** Standard Gradle console output; test failures reported via JUnit XML (consumed by GitHub Actions test reporter)
- **Application-level logging:** The framework provides `commons-logging` as an abstraction over Log4j/SLF4J; no log output configuration in the framework itself

## Jobs / Schedulers
- **CI schedule:** Daily at 09:30 UTC (`ci.yml` cron: `30 9 * * *`) — full matrix test run across Java 17, 21, 25
- **Backport bot:** Triggered by GitHub `labeled` events on issues/PRs; auto-cherry-picks commits to maintenance branches (e.g., `6.2.x`)
- **Antora UI update:** Weekly workflow (`update-antora-ui-spring.yml`) to pick up UI component updates

## Common Issues

| Issue | Cause | Resolution |
|------|------|-----------|
| `StackOverflowError` during context refresh | Circular bean dependency via constructor injection | Redesign beans to break cycle; use setter injection as last resort |
| OutOfMemoryError in large test suites | `DefaultContextCache` retaining too many `ApplicationContext` instances | Annotate test classes with `@DirtiesContext` or tune `spring.test.context.cache.maxSize` |
| `NoSuchBeanDefinitionException` | Bean not in scan path or qualifier mismatch | Verify `@ComponentScan` base packages; check `@Qualifier` / `@Primary` |
| Build fails on Java 25 EA | API changes in early-access JDK | Check CI matrix exclusions; may need toolchain version bump in `buildSrc` |
| Develocity scan not published locally | `DEVELOCITY_ACCESS_KEY` not set | Run with `--no-scan` or set the key; scans are optional locally |

## Backup & Recovery
- **Backup process:** Source is authoritative in GitHub (`spring-projects/spring-framework`). Maven Central artefacts are immutable once released. No operational database to back up.
- **Recovery steps:** Re-trigger failed CI workflows via GitHub Actions UI; re-deploy snapshots by pushing to `main`; GA releases require Sonatype Nexus staging re-upload if pipeline fails mid-release.
