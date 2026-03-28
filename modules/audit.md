# Module: audit

> **Package:** `io.sdkman.broker.audit` | **Files:** `AuditRepo.java`, `AuditEntry.java` | **Language:** Java

Records a download event to the MongoDB `audit` collection each time a download request is handled. Called by all three download handlers (`CandidateDownloadHandler`, `BinaryDownloadHandler`, `NativeBinaryDownloadHandler`).

---

## Responsibilities

- Provide a structured model (`AuditEntry`) for a download event
- Write audit records to the MongoDB `audit` collection asynchronously

---

## Public Interface

| Class | Method | Signature | Description |
|---|---|---|---|
| `AuditRepo` | `insertAudit` | `void insertAudit(AuditEntry auditEntry)` | Asynchronously writes the audit entry to MongoDB. No return value; failures are silent. |
| `AuditEntry` | `of` (7-arg) | `static AuditEntry of(String command, String candidate, String version, String host, String agent, String platform, String dist)` | Factory method for candidate download audit entries |
| `AuditEntry` | `of` (4-arg) | `static AuditEntry of(RequestDetails details, String candidate, String platform, String distribution)` | Factory method for bash/native CLI download audit entries, extracted from `RequestDetails` |

---

## Dependencies

**Internal:**
| Dependency | Purpose |
|---|---|
| `io.sdkman.broker.db.MongoProvider` | Provides MongoDB database handle |
| `io.sdkman.broker.binary.RequestDetails` | Used in the 4-argument `AuditEntry.of` factory |

**External:**
| Library | Version | Purpose |
|---|---|---|
| `mongo-java-driver` | 3.2.2 | `MongoCollection`, `BasicDBObject`, `ObjectId` |
| Ratpack exec | 1.9.0 | `Blocking.exec` for non-blocking DB write |
| SLF4J | 2.0.17 | Debug logging of audit entries |

---

## Inputs and Outputs

**Input:** `AuditEntry` containing:

| Field | Type | Description |
|---|---|---|
| `command` | String | Action performed (e.g. `install`) |
| `candidate` | String | SDK or CLI name (e.g. `java`, `sdkman`) |
| `version` | String | Version string |
| `host` | String | Client IP from `X-Real-IP` header |
| `agent` | String | Client user-agent string |
| `platform` | String | Canonical platform ID (e.g. `DarwinX64`) |
| `dist` | String | Distribution name (e.g. `MAC_OSX`, `UNIVERSAL`) |
| `timestamp` | Long | `System.currentTimeMillis()` at construction time |

**Output:** None. Record is written to the `audit` MongoDB collection.

**Side effects:**
- Inserts one document into `audit` on each call
- Logs at DEBUG level on success

---

## Known Issues / Technical Debt

- `AuditRepo.insertAudit` uses `Blocking.exec` with no error handler. If MongoDB is unavailable or the write fails, the exception is swallowed. There is no logging at error level, no retry, and no dead-letter mechanism.
- `AuditEntry.timestamp` is set at object construction time using `System.currentTimeMillis()`. If there is any delay between construction and the actual DB write (e.g. Ratpack thread scheduling), the recorded time may be slightly earlier than the actual write.
- The 4-argument `AuditEntry.of` factory throws `IllegalArgumentException` if `details` is null, but the 7-argument version has no null checks. Inconsistent defensive programming.
