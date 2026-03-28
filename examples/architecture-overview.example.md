# Architecture Overview — OrderBridge

> **System:** OrderBridge v2.3 | **Language:** Java 8 | **Last active development:** 2019

OrderBridge is a monolithic order management system that ingests purchase orders from retail partners, validates and enriches them, then routes them to downstream fulfilment and finance systems.

---

## Tech Stack

| Layer | Technology | Version | Notes |
|---|---|---|---|
| Language | Java | 8 | No lambdas used in core modules |
| Framework | Spring MVC | 4.3 | XML config, no annotations |
| ORM | Hibernate | 4.2 | Legacy dialect, no JPA |
| Database | Oracle | 11g | Stored procedures used heavily |
| Messaging | IBM MQ | 7.5 | Point-to-point queues |
| Build | Ant | 1.9 | No Maven/Gradle |
| Server | JBoss AS | 6.1 | EAR deployment |

---

## Top-Level Structure

```
orderbridge/
├── src/
│   ├── main/
│   │   ├── com/acme/orderbridge/
│   │   │   ├── ingest/          # Partner order ingestion
│   │   │   ├── validation/      # Business rule validation
│   │   │   ├── enrichment/      # Data lookup and enrichment
│   │   │   ├── routing/         # Downstream system routing
│   │   │   ├── fulfilment/      # Fulfilment system adapter
│   │   │   ├── finance/         # Finance system adapter
│   │   │   ├── model/           # Domain objects
│   │   │   └── util/            # Shared utilities
│   │   └── resources/
│   │       ├── spring/          # Spring XML configs
│   │       └── sql/             # Named queries and DDL
├── test/
├── config/
│   ├── jboss/                   # Server deployment descriptors
│   └── mq/                      # MQ queue definitions
└── build.xml
```

---

## Component Diagram

```
  Partner Systems
       │
       ▼  (HTTP / SFTP)
  ┌──────────┐
  │  Ingest  │  ← Parses XML/CSV order feeds
  └────┬─────┘
       │
       ▼
  ┌────────────┐
  │ Validation │  ← Applies ~60 business rules
  └────┬───────┘
       │ invalid → DeadLetterQueue (MQ)
       ▼
  ┌────────────┐
  │ Enrichment │  ← Looks up product, customer, pricing data (Oracle)
  └────┬───────┘
       │
       ▼
  ┌─────────┐
  │ Routing │  ← Determines fulfilment path
  └────┬────┘
       │
  ┌────┴────────────────┐
  ▼                     ▼
┌────────────┐   ┌─────────┐
│ Fulfilment │   │ Finance │
│  Adapter   │   │ Adapter │
└────────────┘   └─────────┘
  (MQ → WMS)      (MQ → SAP)
```

---

## Data Flow

1. **Ingest** — A scheduled poller picks up XML or CSV files from SFTP, or receives HTTP POST from partner API. Parses into an internal `Order` domain object.
2. **Validation** — Rules engine applies 62 hardcoded business rules. Failures are written to `ORDER_ERROR` table and pushed to dead-letter queue.
3. **Enrichment** — Looks up product catalogue, customer account, and current pricing from Oracle. Mutates the `Order` object in place.
4. **Routing** — Inspects order attributes (partner code, fulfilment region, order type) to determine downstream path. Single order can fan out to multiple targets.
5. **Dispatch** — Adapters serialise the enriched order to the required format and push to the appropriate MQ queue.

---

## Key Design Decisions

| Decision | Rationale |
|---|---|
| Monolithic EAR deployment | Simplified ops at the time; no container orchestration available |
| Oracle stored procedures for validation | Historical: validation logic was owned by a DBA team |
| Mutable `Order` object through pipeline | Convenience; no immutability requirement existed |
| XML Spring config over annotations | Project predates annotation-driven Spring adoption |

---

## Technical Debt / Risk Areas

> ⚠️ **Unclear:** Routing rules in `RoutingEngine.java` contain commented-out branches with no explanation. Unclear if they represent dead code or deferred logic.

- **No unit tests** on `enrichment` or `routing` packages. Integration tests only, require live Oracle connection.
- **62 validation rules** are hardcoded in a single 1,400-line class (`OrderValidator.java`). No rule registry or configuration.
- **IBM MQ 7.5** is end-of-life. No retry or backoff logic on queue writes.
- **Oracle 11g** stored procedures contain business logic that is not reflected in Java code or documented anywhere.
- **Ant build** has no dependency management; JAR files are checked into `lib/` with no version tracking.
