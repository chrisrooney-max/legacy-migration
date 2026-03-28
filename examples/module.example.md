# Module: OrderValidator

> **Package:** `com.acme.orderbridge.validation` | **File:** `OrderValidator.java` | **Lines:** 1,412

Applies business rule validation to an ingested `Order` before it is passed to enrichment. Returns a `ValidationResult` containing pass/fail status and a list of rule violations.

---

## Responsibilities

- Execute all 62 validation rules against a given `Order`
- Accumulate violations without short-circuiting (all rules always run)
- Write failed orders to the `ORDER_ERROR` database table
- Push validation failure events to the `ORDER.DLQ` MQ queue

---

## Public Interface

| Method | Signature | Description |
|---|---|---|
| `validate` | `ValidationResult validate(Order order)` | Entry point. Runs all rules and returns the result. |
| `isValid` | `boolean isValid(Order order)` | Convenience wrapper; returns true if `validate` produces no violations. |

No other public methods. All 62 rule methods are `private`.

---

## Dependencies

**Internal:**
| Dependency | Purpose |
|---|---|
| `com.acme.orderbridge.model.Order` | Domain object being validated |
| `com.acme.orderbridge.model.ValidationResult` | Return type carrying pass/fail and violations |
| `com.acme.orderbridge.util.MQPublisher` | Pushes failure events to dead-letter queue |
| `com.acme.orderbridge.util.OrderErrorDao` | Writes failure records to `ORDER_ERROR` table |

**External:**
| Library | Version | Purpose |
|---|---|---|
| `commons-lang` | 2.6 | String utilities in rule checks |
| Spring `JdbcTemplate` | 4.3 | Used in `OrderErrorDao` |

---

## Inputs and Outputs

**Input:** `Order` — a fully parsed, un-enriched domain object. Fields like `customerId` and `productCode` are present; derived fields like `customerName` are not yet populated.

**Output:** `ValidationResult`

```java
public class ValidationResult {
    boolean passed;           // true if zero violations
    List<String> violations;  // human-readable rule failure messages
}
```

**Side effects:**
- On failure: inserts a row into `ORDER_ERROR` (Oracle)
- On failure: publishes a message to `ORDER.DLQ` (IBM MQ)

---

## Known Issues / Technical Debt

> ⚠️ **Unclear:** Rules 47–51 reference a `partnerTier` field on `Order` that is never populated by the ingest layer. These rules always pass. It is not known whether this is intentional or a defect.

- All 62 rules are `private` methods in a single class. There is no rule registry, no way to enable/disable rules per partner, and no metadata (rule ID, description, owner).
- Rule failure messages are hardcoded English strings — no externalisation or i18n support.
- `OrderErrorDao` opens a new JDBC connection per call rather than using the connection pool.
- No unit tests. Tested only through end-to-end integration tests in `OrderProcessingIT.java`.
