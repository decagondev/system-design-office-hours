# Sample Interview Questions

**System:** Affiliate Link Redirect & Attribution Service  
**Based on:** the three designs from the UPER walkthrough (naive → second pass → scaling)

---

## Solution 1 — Naive (sync lookup + INSERT click + 302)

### Q1. Walk me through what happens on a single click in your design.

**Answer:**  
The visitor hits `GET /abc123`. The redirect service looks up the link row (`destination_url`, `affiliate_id`, `status`). If the link is missing or inactive, it returns 404. If active, it inserts a click row (`click_id`, `code`, `affiliate_id`, `clicked_at`, request metadata), then responds with `302` and `Location` set to the destination URL.

---

### Q2. Why check `status` before inserting the click and redirecting?

**Answer:**  
Deactivated links should not send traffic or create billable attribution. Checking status first keeps “do not redirect” and “do not credit” aligned in this naive design. If we redirected without checking, we could send users to a killed offer and still write click rows.

---

### Q3. What’s the latency problem with inserting the click before the 302?

**Answer:**  
The visitor’s redirect waits on a database write. Under load, INSERT latency and lock/contention directly add to time-to-redirect. If the DB is slow or down, redirects fail even though the destination is known. In production affiliate systems you usually want redirect success decoupled from persistence of the click.

---

### Q4. How does this design behave if the browser retries the same GET twice?

**Answer:**  
Badly for money. Two requests become two INSERTs and two credits for one user action (retry, double-tap, or some prefetches). There is no dedup key and no idempotency, so Solution 1 over-attributes under duplicate delivery.

---

### Q5. A marketer updates `destination_url` or sets `status=inactive`. Is that safe in Solution 1?

**Answer:**  
For correctness against the DB, yes — the next request reads the updated row, so there is no cache-staleness issue yet. The risks here are different: every request still hits the DB, and there is still no protection against duplicate credits. Solution 1 is “correct but naive,” not “correct under load and retries.”

---

### Q6. What happens if the database is unavailable?

**Answer:**  
Both redirect and attribution fail together, because the hot path depends on SELECT (and INSERT). That couples availability of the public redirect to OLTP health — usually unacceptable for a link that marketers put in ads.

---

### Q7. Why might an interviewer still like hearing Solution 1 first?

**Answer:**  
It shows you can state the core loop clearly: resolve link → optionally credit → redirect. It also creates a clean runway to discuss what breaks (latency, duplicates, availability) before you add cache, queues, and ledgers.

---

## Solution 2 — Second pass (cache + invalidate, enqueue then 302, async dedup)

### Q1. What’s cached for `abc123`, and why those fields?

**Answer:**  
Cache `link:{code}` → `{ destination_url, affiliate_id, status, version }` (or equivalent). You need the destination to redirect, status to refuse inactive links, affiliate_id so the handoff event can credit the right partner without another DB hop, and version (or updated_at) to reason about freshness when debugging stale entries.

---

### Q2. Marketer deactivates a link. How do you keep the cache correct?

**Answer:**  
Admin/API updates the database first (source of truth), then invalidates `link:{code}` (delete the key) or writes the new value. Next redirect misses cache and reloads from DB, sees `inactive`, returns 404. A short TTL alone is not enough — after deactivate you can still redirect for the TTL window. Invalidate-on-write is the primary control; TTL is a safety net.

---

### Q3. Why enqueue a `ClickCandidate` and then 302 instead of writing the click in-request?

**Answer:**  
So redirect latency does not wait on attribution storage, downstream analytics, or payout. The hot path only needs: resolve active destination, durably hand off a candidate event (or spill/retry if enqueue fails under your chosen policy), then 302. Heavy work moves to async consumers.

---

### Q4. What is your dedup key, and where does the dedup check run relative to the redirect?

**Answer:**  
Example key: `hash(code | cookie_or_device_id | coarse_time_bucket | optional_client_request_id)`. Prefer an explicit client idempotency / request id when the app can send one.  
**Authoritative dedup runs in the async attribution worker** before writing a billable click (e.g. unique constraint on `dedup_key`). The redirect path should not be the source of truth for money; it may attach hints and an `event_id`, but “credit once” is enforced after handoff. That way retries can duplicate HTTP and even duplicate queue messages without double-paying.

---

### Q5. You said “exactly once” credit. Do you literally get exactly-once delivery?

**Answer:**  
Usually no — the pipeline is **at-least-once**. We get **effectively once** credit by combining at-least-once delivery with an **idempotent apply** (unique `dedup_key` / upsert-ignore). Interviewers want that distinction: delivery semantics vs business idempotency.

---

### Q6. If enqueue to the click queue fails, do you still 302?

**Answer:**  
Product tradeoff.  
- **Still 302 + local spill/retry:** better user/marketer redirect UX; risk of lost attribution if spill fails.  
- **Fail the request:** preserves “no redirect without handoff attempt,” but hurts visitors and campaign UX.  
A strong answer picks one, names the failure mode, and mentions metrics/alerts on spill depth and enqueue errors.

---

### Q7. How do you avoid counting prefetch as a billable click?

**Answer:**  
Inspect headers such as `Sec-Purpose` / `Purpose` (and product-specific signals). Prefetch can generate GETs that are not intentional clicks. Policy often: still redirect if the link is active, but mark the candidate non-billable or drop it before ledger credit. Dedup alone does not solve prefetch.

---

### Q8. Could you dedupe only in memory on the redirect box?

**Answer:**  
Not safely for payout. Multiple redirect replicas won’t share that memory; restarts lose state; at-least-once queue delivery still needs a shared idempotent store. In-memory seen-sets can be a best-effort optimization, not the ledger’s uniqueness mechanism.

---

## Solution 3 — Scaling (replicas, durable log, dashboard vs payout)

### Q1. How does the redirect tier scale?

**Answer:**  
Stateless redirect replicas behind a load balancer (or edge). Shared link cache (e.g. Redis) for hot codes; DB remains source of truth on miss and for writes. Redirect nodes enqueue to a durable log/queue and return 302. You scale redirect CPU/connections independently from attribution workers.

---

### Q2. What guarantees does your click pipeline make about loss and duplicates?

**Answer:**  
Once the producer gets an ack from the durable log/queue, aim for **at-least-once** retention/delivery within retention policy. Duplicates are expected (producer retries, consumer retries). **Loss before ack** is still possible if you 302 after a failed enqueue unless you have local spill. Consumers must be idempotent so duplicates don’t double-pay.

---

### Q3. The payout calculator is down or hours behind. Does the redirect path notice?

**Answer:**  
No — and it shouldn’t. Redirect only depends on link resolution + enqueue. The log buffers. Ops see consumer lag / backlog alerts. When payout recovers, it drains the ledger feed. Coupling redirect health to payout health would recreate Solution 1’s availability failure at larger scale.

---

### Q4. Why might live dashboard click counts disagree with numbers used to pay affiliates?

**Answer:**  
Different consistency goals.  
- **Dashboard:** optimize for freshness; eventually consistent rollups/approximations are OK for marketers watching campaigns.  
- **Payout:** optimize for explainability and no double pay; read an append-only **billable ledger** written only after idempotent credit.  
They can diverge briefly (or under failure). Finance trusts the ledger and reconciliation, not the realtime counter.

---

### Q5. How do you enforce “don’t pay twice” at scale?

**Answer:**  
Attribution consumer applies `ClickCandidate` with a unique constraint (or idempotent upsert) on `dedup_key` (and retains `event_id` for tracing). Payout settlement jobs read the ledger of successfully credited clicks, not the dashboard aggregate table. Re-running a consumer or payout job becomes safe.

---

### Q6. Where would fraud / bot detection plug in without blocking redirects?

**Answer:**  
As an async consumer or filter on the click stream **after** handoff and **before or beside** payout eligibility — never on the 302 critical path. Redirect stays fast; suspicious events can be marked non-billable in the ledger path.

---

### Q7. When do you shard, and on what key?

**Answer:**  
Shard when a single primary/cache/queue partition can’t hold data or write/read volume. Common keys: `hash(code)` for link records, and partition the click log by `code` or `affiliate_id` if you want per-link ordering. Don’t shard on day one of the interview answer unless the prompt forces huge scale — show the trigger (size, QPS, hotspot).

---

### Q8. How would conversion events fit without changing the redirect hot path?

**Answer:**  
Same pattern as clicks: a `ConversionCandidate` enters the durable pipeline later (post-purchase webhook, pixel, server event) with its own idempotency key (e.g. `order_id`), joined to the originating click/affiliate. Redirect remains resolve + enqueue click candidate + 302 only.

---

### Q9. What’s a good end-to-end summary an interviewer wants in 30 seconds?

**Answer:**  
“Redirect is a cached, status-aware lookup with invalidate-on-write, then at-least-once handoff of a click candidate and an immediate 302. Attribution is async and idempotent on a dedup key so retries don’t double-pay. Dashboards read a fast projection; payout reads a billable ledger. Downstream lag never blocks redirects. Fraud is an async filter on the stream.”

---

## Quick comparison (interviewer follow-up)

| Topic | Solution 1 | Solution 2 | Solution 3 |
|-------|------------|------------|------------|
| Redirect latency | Waits on INSERT | Enqueue then 302 | Same + horizontal replicas |
| Duplicate GETs | Double credit | Async unique dedup | Same + durable at-least-once |
| Link update/deactivate | DB read (OK) | DB + cache invalidate | Same at scale |
| Payout consumer down | N/A / coupled | Backlog after handoff | Redirect isolated; lag alerts |
| Dashboard vs pay | Often one table | Emerging split | Explicit projection vs ledger |

---
