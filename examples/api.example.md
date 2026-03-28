# API Documentation — OrderBridge

> **Base URL:** `https://orderbridge.internal/api` | **Protocol:** HTTP/1.1 | **Format:** XML (request and response)

---

## Authentication

All endpoints require HTTP Basic Auth. Credentials are per-partner and provisioned manually by the ops team. No token rotation mechanism exists.

```
Authorization: Basic <base64(partner_id:secret)>
```

> ⚠️ **Unclear:** The `/admin/*` endpoints do not appear to enforce authentication in the servlet filter chain. Behaviour in production is unknown.

---

## Endpoint Index

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/orders` | Submit a new order |
| `GET` | `/api/orders/{orderId}` | Get order status by ID |
| `GET` | `/api/orders` | List orders for the authenticated partner |
| `POST` | `/api/orders/{orderId}/cancel` | Request cancellation of an order |

---

## Endpoints

### POST /api/orders

Submit a new order for processing.

**Request body** (XML):

```xml
<order>
  <partnerOrderId>PO-12345</partnerOrderId>
  <partnerId>ACME_UK</partnerId>
  <orderDate>2019-04-01</orderDate>
  <lines>
    <line>
      <productCode>SKU-9981</productCode>
      <quantity>2</quantity>
      <unitPrice>49.99</unitPrice>
    </line>
  </lines>
  <shippingAddress>
    <line1>123 High Street</line1>
    <city>London</city>
    <postcode>EC1A 1BB</postcode>
    <country>GB</country>
  </shippingAddress>
</order>
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `partnerOrderId` | string | yes | Partner's own reference; must be unique per partner |
| `partnerId` | string | yes | Must match the authenticated partner |
| `orderDate` | date (YYYY-MM-DD) | yes | |
| `lines` | list | yes | Min 1 line |
| `lines.line.productCode` | string | yes | Must exist in product catalogue |
| `lines.line.quantity` | integer | yes | Must be ≥ 1 |
| `lines.line.unitPrice` | decimal | yes | Must match catalogue price within 1% tolerance |
| `shippingAddress` | object | yes | |
| `shippingAddress.country` | string (ISO 3166-1 alpha-2) | yes | |

**Response — 202 Accepted:**

```xml
<orderAck>
  <internalOrderId>OB-789456</internalOrderId>
  <partnerOrderId>PO-12345</partnerOrderId>
  <status>RECEIVED</status>
</orderAck>
```

**Error responses:**

| Status | Code | Meaning |
|---|---|---|
| `400` | `VALIDATION_FAILED` | One or more validation rules failed. See `<violations>` in body. |
| `400` | `DUPLICATE_ORDER` | `partnerOrderId` already exists for this partner. |
| `401` | `UNAUTHORISED` | Missing or invalid credentials. |
| `500` | `INTERNAL_ERROR` | Unhandled exception. Check server logs. |

---

### GET /api/orders/{orderId}

Retrieve the current status of an order by internal ID.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `orderId` | string | Internal order ID (`OB-*`) returned at submission |

**Response — 200 OK:**

```xml
<order>
  <internalOrderId>OB-789456</internalOrderId>
  <partnerOrderId>PO-12345</partnerOrderId>
  <status>DISPATCHED</status>
  <lastUpdated>2019-04-02T14:23:00Z</lastUpdated>
</order>
```

**Status values:**

| Value | Meaning |
|---|---|
| `RECEIVED` | Ingested, awaiting processing |
| `PROCESSING` | Currently in validation/enrichment pipeline |
| `DISPATCHED` | Sent to fulfilment system |
| `CANCELLED` | Cancellation confirmed |
| `FAILED` | Validation or routing failure — see dead-letter queue |

**Error responses:**

| Status | Code | Meaning |
|---|---|---|
| `404` | `NOT_FOUND` | No order with this ID exists for the authenticated partner. |

---

### GET /api/orders

List orders submitted by the authenticated partner.

**Query parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `status` | string | no | — | Filter by status value (see above) |
| `from` | date | no | 30 days ago | Start of date range (orderDate) |
| `to` | date | no | today | End of date range (orderDate) |
| `page` | integer | no | 1 | Page number |
| `pageSize` | integer | no | 50 | Results per page. Max: 200 |

> ⚠️ **Unclear:** Pagination appears to be offset-based. Behaviour when records are inserted during pagination is not handled.

**Response — 200 OK:**

```xml
<orders total="142" page="1" pageSize="50">
  <order>...</order>
  <order>...</order>
</orders>
```

---

### POST /api/orders/{orderId}/cancel

Request cancellation of an order.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `orderId` | string | Internal order ID |

**Request body:** empty

**Response — 200 OK:**

```xml
<cancelAck>
  <internalOrderId>OB-789456</internalOrderId>
  <status>CANCELLED</status>
</cancelAck>
```

**Error responses:**

| Status | Code | Meaning |
|---|---|---|
| `409` | `CANNOT_CANCEL` | Order is already dispatched or failed — cancellation not possible. |
| `404` | `NOT_FOUND` | Order does not exist for this partner. |
