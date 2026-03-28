# Getting Started — OrderBridge

> Estimated setup time: ~2 hours (Oracle dependency makes this slow)

---

## Prerequisites

| Tool | Version | Notes |
|---|---|---|
| JDK | 1.8 | Must be Oracle JDK 8, not OpenJDK — Hibernate dialect incompatibility |
| Ant | 1.9+ | |
| JBoss AS | 6.1.0.Final | Download from [JBoss archive](https://jbossas.jboss.org/downloads) |
| Oracle DB | 11g | Dev instance available — see below |
| IBM MQ client | 7.5 | Client JAR is in `lib/` — no install needed |

> ⚠️ **Unclear:** The project has never been built or run on macOS Apple Silicon. JBoss AS 6.1 predates ARM64 JVM support. Use an x86 machine or VM.

---

## Setup

**1. Clone the repository**

```bash
git clone git@github.com:acme/orderbridge.git
cd orderbridge
```

**2. Configure database connection**

Copy the template and fill in the dev Oracle credentials (ask the ops team):

```bash
cp config/db.properties.template config/db.properties
```

Edit `config/db.properties`:

```properties
db.url=jdbc:oracle:thin:@dev-oracle.internal:1521:OBDEV
db.username=<your-username>
db.password=<your-password>
```

> These credentials are individual. Do not share or commit them.

**3. Configure JBoss datasource**

Copy the datasource descriptor into your JBoss installation:

```bash
cp config/jboss/orderbridge-ds.xml $JBOSS_HOME/server/default/deploy/
```

**4. Configure MQ**

Edit `config/mq/mq.properties` with the dev MQ broker details (also from ops):

```properties
mq.host=dev-mq.internal
mq.port=1414
mq.channel=DEV.SVRCONN
mq.queueManager=QMDEV
```

**5. Build**

```bash
ant clean build
```

The EAR is output to `dist/orderbridge.ear`.

**6. Deploy to JBoss**

```bash
cp dist/orderbridge.ear $JBOSS_HOME/server/default/deploy/
$JBOSS_HOME/bin/run.sh
```

JBoss is ready when you see:
```
INFO  [org.jboss.as] JBoss AS 6.1.0.Final started in Xs
```

---

## Running the Application

OrderBridge has no web UI. It runs as a background service. Verify it is running:

```bash
curl -u test_partner:secret -X GET http://localhost:8080/api/orders
```

Expected response: `200 OK` with an empty `<orders>` element.

Logs are written to `$JBOSS_HOME/server/default/log/server.log`.

---

## Running Tests

> ⚠️ All tests require a live Oracle connection. There are no unit tests that run without the database.

```bash
ant test
```

Tests connect to the dev Oracle instance using the same `config/db.properties` you set up above. Expect the full suite to take ~8 minutes.

To run a single test class:

```bash
ant test -Dtest.class=com.acme.orderbridge.validation.OrderValidatorIT
```

---

## Key Concepts

**1. The `Order` domain object is mutable and shared across the pipeline.**
Every stage (ingest → validation → enrichment → routing) reads from and writes to the same `Order` instance. There is no immutability. If you're debugging a field value, check which stage last wrote to it.

**2. Validation rules are all-or-nothing per order, but individual rules never short-circuit.**
All 62 rules always run, even after the first failure. An order with 10 violations will produce all 10 in the `ValidationResult`.

**3. Business logic lives in Oracle stored procedures, not just Java.**
The enrichment layer calls several Oracle stored procs (`PKG_PRICING.get_price`, `PKG_CUSTOMER.enrich_order`). Some logic exists only there. If you're tracing a data issue, check the DB as well as the Java.

**4. MQ is fire-and-forget.**
There is no acknowledgement or retry logic on MQ writes. If the broker is unavailable, the exception is logged and the operation silently fails. Do not assume a dispatched order has definitely reached the downstream system.

**5. The Ant build has no dependency management.**
All JARs are in `lib/` with no version file. Do not upgrade any library without checking all transitive dependencies manually.

---

## Common Gotchas

| Gotcha | Detail |
|---|---|
| Must use Oracle JDK 8 | OpenJDK causes a Hibernate `OracleDialect` ClassCastException at startup. |
| JBoss classpath conflicts | If you see `ClassNotFoundException` on startup, check for duplicate JARs between `lib/` and JBoss's own modules. |
| `ORDER_ERROR` table fills fast in dev | The dev Oracle instance has a shared `ORDER_ERROR` table. Run `TRUNCATE TABLE ORDER_ERROR` if integration tests start failing with constraint violations. |
| MQ queues persist across restarts | Dev MQ queues are not purged on restart. Stale messages from a previous test run can cause unexpected behaviour. Purge queues manually via the MQ Explorer tool before running tests. |
| `config/db.properties` is gitignored — don't commit it | It contains plain-text credentials. The `.gitignore` covers it but double-check before pushing. |
