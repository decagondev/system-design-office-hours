**Problem statement:** Marketers create short links (`go.ourapp.com/abc123`) that redirect visitors to a destination URL while crediting the correct affiliate for each real click — without making the visitor wait on analytics or payout work.

**Fraud (one line):** fraud/bot scoring plugs in as an async consumer or pre-credit filter on the click stream after redirect handoff, never on the 302 path.

### Understand
Two jobs, two columns on the board: **redirect** (fast, public) and **attribution** (careful, money). A deactivated or updated link must not keep sending people to the wrong place. A browser retry must not pay twice. If payout is down, the visitor should still leave.

---

### Pass 1 — Naive (sync everything)
On `GET /abc123`: DB lookup → if active, `INSERT` click → 302. Reason: proves credit exists in the simplest way.

```mermaid
sequenceDiagram
  participant V as Visitor
  participant R as Redirect API
  participant DB as Database
  V->>R: GET /abc123
  R->>DB: SELECT link by code
  alt inactive or missing
    R-->>V: 404
  else active
    R->>DB: INSERT click
    R-->>V: 302 destination_url
  end
```

**What breaks:** redirect waits on INSERT; retries double-credit; no cache; DB outage kills redirects; reporting/payout either hammer OLTP or don’t exist.

---

### Pass 2 — Cache, handoff, dedup
**Cached:** `link:{code}` → `{destination_url, affiliate_id, status, version}`.

**Keep correct on update/deactivate:** Admin writes DB first (source of truth), then **invalidates** the cache key. Short TTL is a safety net only — TTL alone can still redirect for seconds after kill.

**Hot path:** cache/DB resolve → if active, **enqueue** `ClickCandidate` → **302**. Do not await analytics.

**Dedup key (classroom default):**
`hash(code | cookie_or_device | coarse_time_bucket | optional_client_request_id)`  
Prefer a client idempotency key when the app sends one. Prefetch headers: don’t treat every GET as billable.

**Where dedup sits:** authoritative check is in the **async attribution worker** before writing the billable ledger (unique constraint on `dedup_key`). Redirect only attaches hints/`event_id` and enqueues. At-least-once delivery + idempotent apply ⇒ effectively once per real click for pay.

**Enqueue failure tradeoff:** still 302 + local spill/retry (better UX, risk lost credit) vs fail the redirect (hurts visitors). Recommend spill/retry and say the risk aloud.

```mermaid
flowchart LR
  V[Visitor] -->|GET /abc123| R[Redirect API]
  R --> Cache[(Link cache)]
  Cache -.->|miss| DB[(Links DB)]
  R -->|enqueue ClickCandidate| Q[Click queue]
  R -->|302| V
  Q --> W[Attribution worker]
  W --> Dedup[(Dedup store)]
  W -->|credit once| Clicks[(Billable clicks)]
```

```mermaid
sequenceDiagram
  participant M as Marketer
  participant API as Admin API
  participant DB as Links DB
  participant Cache as Link cache
  M->>API: Update or deactivate
  API->>DB: UPDATE link
  API->>Cache: DELETE link:abc123
```

---

### Pass 3 — Pipeline & scale
Stateless redirect replicas + shared link cache. Durable click log feeds consumers that scale on their own.

```mermaid
flowchart TB
  V[Visitor] -->|GET /abc123| LB[Load balancer / edge]
  LB --> R1[Redirect]
  LB --> R2[Redirect]
  R1 --> Cache[(Link cache)]
  R2 --> Cache
  Cache -.-> DB[(Links DB)]
  R1 -->|302| V
  R1 -->|ClickCandidate| Log[Durable click log]
  Log --> Attr[Attribution consumer]
  Attr --> Dedup[(Dedup / unique keys)]
  Attr --> Ledger[(Billable click ledger)]
  Log --> Dash[Dashboard aggregator]
  Dash --> DashStore[(Realtime counts)]
  Ledger --> Pay[Payout calculator]
  Ledger --> WH[(Warehouse / recon)]
  M[Marketer] -->|update / deactivate| Admin[Admin API]
  Admin --> DB
  Admin -->|invalidate| Cache
```

| Concern | Guarantee | How |
|---------|-----------|-----|
| After enqueue ack | At-least-once | Durable log + retries |
| Double delivery | Expected | Idempotent consumers |
| Double pay | Effectively once | Unique `dedup_key` on ledger |
| Payout down / behind | Redirect unaffected | Backlog in log; lag alerts |
| Live dashboard | Eventually consistent / approx OK | Fast projection from stream |
| Payout numbers | Reconcilable, ledger-backed | Append-only billable ledger; settle from ledger, not dashboard counters |

**Why dashboard ≠ paycheck:** marketers want freshness; finance needs every credited click explainable. They may disagree briefly — finance trusts the ledger.

Conversions later: same pattern (`ConversionCandidate` + order idempotency), never on the redirect path.

---

### Reflect
Pass 1 surfaces latency and double-count. Pass 2 answers redirect cache, invalidation, async handoff, and dedup placement. Pass 3 isolates consumers and splits approximate dashboards from the payout ledger.
