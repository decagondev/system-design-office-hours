# GitHub issues for the PRD

Source: [PRD.md](PRD.md). One issue equals one slice. A slice is the smallest cut that can merge on its own, with its tests. The detail lives in the issue body. Do not split these further into one ticket per function.

The title is the upsert key. Keep the `[S…]` prefix. The first line of every body is `Slice: S…`, which is unique (`Slice: S0` does not match `Slice: S0.1`).

## Categories

Use these labels. `gh issue create` fails if a label is missing, so create them first.

| Label | Kind | Meaning |
|-------|------|---------|
| `wave-0` … `wave-6` | Wave | Build order. Wave 6 is the dashboard stretch |
| `required` | Track | Waves 0–5. The build is done when these are done |
| `stretch` | Track | Wave 6. The required build stays done without it |
| `type-feature` | Type | Product behavior |
| `type-test` | Type | Harness, smoke, or golden files |
| `type-chore` | Type | Skeleton, tooling, lint |
| `area-foundation` | Area | Package, settings, migrate |
| `area-auth` | Area | Users, passwords, JWT, roles |
| `area-links` | Area | Create and update short links |
| `area-redirect` | Area | Public `GET /{code}` |
| `area-attribution` | Area | Queue, worker, ledger, dedup |
| `area-cache` | Area | In-memory link cache |
| `area-spill` | Area | Spill file and `GET /ops/status` |
| `area-tests` | Area | Cross-cutting verification |
| `area-dashboard` | Area | Approximate marketer counts |

## Issue index

| Slice | Title | Labels | After |
|-------|-------|--------|-------|
| S0 | Scaffold the package, settings, and database migrate | `wave-0` `required` `type-chore` `area-foundation` | — |
| S0.1 | Store users, hash passwords, and bootstrap the first admin | `wave-0` `required` `type-feature` `area-auth` | S0 |
| S0.2 | Issue an HS256 JWT and resolve the caller | `wave-0` `required` `type-feature` `area-auth` | S0.1 |
| S0.3 | Admin CRUD for users | `wave-0` `required` `type-feature` `area-auth` | S0.2 |
| S1 | Create a short link and redirect an active code | `wave-1` `required` `type-feature` `area-links` `area-redirect` | S0.2 |
| S2 | Update a link and refuse a missing or inactive code | `wave-1` `required` `type-feature` `area-links` `area-redirect` | S1 |
| S3 | Enqueue a click candidate, then redirect | `wave-2` `required` `type-feature` `area-attribution` `area-redirect` | S2 |
| S4 | Credit the ledger and let finance read it | `wave-2` `required` `type-feature` `area-attribution` | S3 |
| S5 | Dedup credits with a click-time key | `wave-2` `required` `type-feature` `area-attribution` | S4 |
| S6 | Serve hot links from an in-memory cache | `wave-3` `required` `type-feature` `area-cache` | S5 |
| S7 | Invalidate the cache when a link is written | `wave-3` `required` `type-feature` `area-cache` `area-links` | S6 |
| S8 | Reload from the database when the cache entry expires | `wave-3` `required` `type-feature` `area-cache` | S7 |
| S9 | Redirect a prefetch and leave it off the ledger | `wave-4` `required` `type-feature` `area-attribution` | S8 |
| S10 | Collapse retries that share an Idempotency-Key | `wave-4` `required` `type-feature` `area-attribution` | S9 |
| S11 | Spill a failed enqueue and expose ops status | `wave-4` `required` `type-feature` `area-spill` | S10 |
| S12 | Add the golden harness | `wave-5` `required` `type-test` `area-tests` | S11 |
| S13 | Add the click smoke test | `wave-5` `required` `type-test` `area-tests` | S12 |
| S14 | Run lint, format, and tests as one gate | `wave-5` `required` `type-chore` `area-foundation` `area-tests` | S13 |
| S15 | Project approximate click counts | `wave-6` `stretch` `type-feature` `area-dashboard` | S14 |
| S16 | Fan out the candidate and serve the dashboard | `wave-6` `stretch` `type-feature` `area-dashboard` | S15 |
| S17 | Lock the dashboard with golden files and smoke | `wave-6` `stretch` `type-test` `area-dashboard` `area-tests` | S16 |

S0.3 and S1 can proceed in parallel after S0.2. Every later slice assumes the one before it in this table.

## How to upsert with gh

Run this from the repo root, already authenticated (`gh auth status`). It creates labels, then creates an issue whose title is missing, or replaces the body and labels when the title already exists.

```powershell
gh label create "wave-0" --color "1D76DB" --description "Sign in and roles" --force
gh label create "wave-1" --color "0E8A16" --description "Redirect from the database" --force
gh label create "wave-2" --color "5319E7" --description "Hand off and credit once" --force
gh label create "wave-3" --color "FBCA04" --description "Cache and invalidation" --force
gh label create "wave-4" --color "D93F0B" --description "Prefetch, idempotency, spill" --force
gh label create "wave-5" --color "006B75" --description "Harness, smoke, lint" --force
gh label create "wave-6" --color "C5DEF5" --description "Dashboard stretch" --force
gh label create "required" --color "0E8A16" --description "Waves 0-5" --force
gh label create "stretch" --color "BFD4F2" --description "Optional Wave 6" --force
gh label create "type-feature" --color "1D76DB" --description "Product behavior" --force
gh label create "type-test" --color "FBCA04" --description "Harness, smoke, golden files" --force
gh label create "type-chore" --color "EDEDED" --description "Skeleton and tooling" --force
gh label create "area-foundation" --color "EDEDED" --description "Package, settings, migrate" --force
gh label create "area-auth" --color "B60205" --description "Users, JWT, roles" --force
gh label create "area-links" --color "0052CC" --description "Short link writes" --force
gh label create "area-redirect" --color "0052CC" --description "Public redirect" --force
gh label create "area-attribution" --color "5319E7" --description "Queue, worker, ledger" --force
gh label create "area-cache" --color "FBCA04" --description "In-memory link cache" --force
gh label create "area-spill" --color "D93F0B" --description "Spill file and ops status" --force
gh label create "area-tests" --color "006B75" --description "Harness and smoke" --force
gh label create "area-dashboard" --color "C5DEF5" --description "Approximate counts" --force

$repo = (gh repo view --json nameWithOwner --jq ".nameWithOwner")
$raw = Get-Content -Raw "docs/ISSUES.md"
$blocks = [regex]::Matches($raw, "(?s):::issue\r?\n(.*?):::\r?\n(.*?):::end")
$existing = @(gh issue list --state all --limit 200 --json number,title | ConvertFrom-Json)
$utf8 = New-Object System.Text.UTF8Encoding $false

foreach ($block in $blocks) {
  $header = @{}
  foreach ($line in ($block.Groups[1].Value -split "\r?\n")) {
    if ($line -match "^([^:]+):\s*(.*)$") { $header[$Matches[1].Trim()] = $Matches[2].Trim() }
  }
  $title = $header["title"]
  $labels = $header["labels"]
  $bodyFile = New-TemporaryFile
  [System.IO.File]::WriteAllText($bodyFile.FullName, ($block.Groups[2].Value.Trim() + "`n"), $utf8)
  $found = @($existing | Where-Object { $_.title -eq $title })
  if ($found.Count -eq 0) {
    gh issue create --title $title --label $labels --body-file $bodyFile.FullName
    $existing += [pscustomobject]@{ number = 0; title = $title }
  } else {
    $number = $found[0].number
    gh issue edit $number --title $title --body-file $bodyFile.FullName
    $apiArgs = @("--method", "PUT", "repos/$repo/issues/$number/labels")
    foreach ($label in ($labels -split ",")) { $apiArgs += @("-f", "labels[]=$label") }
    gh api @apiArgs | Out-Null
  }
  Remove-Item $bodyFile.FullName
}
```

`--force` on `gh label create` updates the description and color when the label already exists. Re-run the script after editing an issue in this file. It matches on the exact title, so a renamed title creates a second issue. Rename in GitHub in the same change, or close the old one.

Shared rules for every issue, from the PRD:

- Python 3.12. Runtime imports stay in the standard library. Dev tools are `pytest` and `ruff`.
- One process. SQLite for users, links, and the ledger. Cache and queue are in-memory objects behind protocols.
- Constructors receive their dependencies. HTTP parses and calls a module. It does not own the business rule.
- Do not log passwords, hashes, tokens, or `JWT_SECRET`.

## Issues

:::issue
id: S0
title: [S0] Scaffold the package, settings, and database migrate
labels: wave-0,required,type-chore,area-foundation
depends_on:
:::
Slice: S0

## Why

Later slices need a package, a settings object, an empty database, and a lint command. This issue creates that shell. It does not sign anyone in and it does not redirect.

## Scope

In this issue:

- `pyproject.toml` for Python 3.12, `pytest`, and `ruff` (`E`, `F`, `I`, `UP`, `B`, `N`, line length 100). Ruff covers `src` and `tests`.
- Package `src/affiliate_links/` with `main.py` and a settings object.
- Settings read from the environment: `DB_PATH`, `HTTP_HOST`, `HTTP_PORT`, `JWT_SECRET`, `JWT_TTL_SECONDS` (default 3600), `BOOTSTRAP_ADMIN_USERNAME`, `BOOTSTRAP_ADMIN_PASSWORD`.
- Startup fails when `JWT_SECRET` is missing or shorter than 32 characters.
- SQLite migrate creates empty `users` and `links` tables. See the column list in the PRD data section. `billable_clicks` waits for S4.

Later issues: passwords, JWT, HTTP routes, cache, queue, spill.

## Plan

1. Add the project files and an empty test that imports the package.
2. Implement settings with the defaults above. Keep cache TTL, click bucket, and spill path out until the slice that uses them.
3. Add `migrate(db_path)` and call it from `main`.
4. Run `ruff check src tests`, `ruff format --check src tests`, and `pytest`.

## Good and bad

Good: `main` builds a settings object and passes the database path into `migrate`. A test calls `migrate` on a temp file and checks the table names.

Bad: a module-level `sqlite3.connect("data/app.sqlite")` that runs on import. Bad: adding FastAPI, Redis, or a JWT library in this issue. Bad: creating `billable_clicks` “while we are here.”

## Done when

- [ ] `pytest` and both ruff commands pass.
- [ ] Startup fails a clear check when `JWT_SECRET` is absent or shorter than 32 characters.
- [ ] A temp database has `users` and `links` and does not have `billable_clicks`.
:::end

:::issue
id: S0.1
title: [S0.1] Store users, hash passwords, and bootstrap the first admin
labels: wave-0,required,type-feature,area-auth
depends_on: S0
:::
Slice: S0.1

## Why

An admin has to exist before anyone can log in, and the password must never be stored or returned in clear text. This issue is the user store and the rules around the last admin. HTTP comes in S0.2 and S0.3.

## Scope

In this issue:

- `users`: `User` (`id`, `username`, `role`, `status`) and `UserStore` as in the PRD ports.
- Password hash `pbkdf2_sha256` with a random 16-byte salt and 600000 iterations. `authenticate` checks it.
- Username: 3–32 characters, `A–Z` `a–z` `0–9`. Password: 8–72 characters. Role: `admin`, `marketer`, `finance`, `operator`. Status: `active` or `disabled`.
- `accounts`: create, update, delete. Duplicate username raises `Conflict`. Deleting the caller raises `Conflict`. A change that would leave zero active admins raises `Conflict`. A password-only update of the last admin succeeds.
- `authenticate` returns `None` for an unknown username, a wrong password, and a disabled user.
- `main` inserts one active admin from the bootstrap settings only when the user table is empty. If the table is empty and either bootstrap value is missing, startup fails. If the table has rows, the bootstrap values are ignored.

Later issues: JWT, login route, user HTTP.

## Plan

1. Implement `UserStore` against the `users` table from S0. Generate `id` with `uuid4`.
2. Implement hash and check in `users`. Keep JWT out of this module.
3. Implement `accounts` on top of `UserStore`. Put the last-admin rule here, not in the SQL adapter and not in HTTP.
4. Call the bootstrap from `main` after migrate.
5. Unit-test the cases in Done when. Use a temp database.

## Good and bad

Good: `accounts.delete(actor_id, target_id)` asks `count_active_admins()` and raises `Conflict` before it deletes the last active admin.

Bad: `UserStore.delete` silently refusing some deletes and allowing others. The store should delete the row it is given. Bad: comparing passwords with `==` on the raw string. Bad: bootstrap resetting the admin password on every startup.

## Done when

- [ ] A correct password authenticates, and the three failure cases return `None`.
- [ ] A second insert of the same username raises `Conflict`.
- [ ] The only active admin cannot be deleted, demoted, or disabled. A second admin can delete the other admin.
- [ ] An admin cannot delete their own id.
- [ ] Bootstrap creates one admin on an empty table and does not add another on the next startup.
- [ ] `accounts` does not import `tokens`.
:::end

:::issue
id: S0.2
title: [S0.2] Issue an HS256 JWT and resolve the caller
labels: wave-0,required,type-feature,area-auth
depends_on: S0.1
:::
Slice: S0.2

## Why

Callers need a token, and protected routes need one function that turns an `Authorization` header into an active user. The role in the database wins over the role written into the token.

## Scope

In this issue:

- `tokens`: `issue(user, now)` and `read(token, now)`. Compact JWT, HMAC-SHA256, header fixed to `alg=HS256` and `typ=JWT`. Claims: `sub`, `username`, `role`, `iat`, `exp`. Lifetime is `JWT_TTL_SECONDS`.
- `read` rejects any other `alg`, including `none`, a bad signature, and an expired token. Use the clock passed in.
- `access.Access.require(authorization, allowed_roles)` verifies the token, loads `sub`, and returns the user only when that row is `active` and the stored role is in `allowed_roles`. Otherwise `Unauthenticated` or `Forbidden`.
- `POST /login` with `username` and `password`. Success is `200` and `{access_token, token_type: "Bearer", expires_in}`. Failure is `401` with the same body for unknown user, wrong password, and disabled user. Log `login_failed` with the username only.
- The token is not stored. There is no refresh token and no denylist.

Later issues: `/users` CRUD (S0.3), link routes (S1). Do not add a public `/protected` sample route.

## Plan

1. Implement `tokens` with `hmac` and `hashlib` only. Unit-test round trip, tampered signature, expiry, and `alg=none`.
2. Implement `Access` with `Tokens` and `UserStore` injected.
3. Add `POST /login`. Map `None` from `authenticate` to `401`.
4. Unit-test `Access.require`: disabled user, role changed in the database after the token was issued, missing header.

## Good and bad

Good: `read` checks `alg` and the signature, then `Access.require` loads the user and compares the stored role to `allowed_roles`.

Bad: trusting `role` inside the payload after a base64 decode. Bad: a login error body that says “unknown user” or “wrong password.” Bad: logging the token or `JWT_SECRET`.

## Done when

- [ ] Login with a good password returns a token whose `sub` is that user id.
- [ ] The three bad logins return `401` and the same body.
- [ ] An expired token and a non-HS256 token fail `read`.
- [ ] After the user’s role is updated in the store, `Access.require` uses the new role on the next call with the old token.
- [ ] `tokens` does not import `users`. `redirect` does not import `tokens` or `access`.
:::end

:::issue
id: S0.3
title: [S0.3] Admin CRUD for users
labels: wave-0,required,type-feature,area-auth
depends_on: S0.2
:::
Slice: S0.3

## Why

The admin who was bootstrapped needs to create the marketer, finance user, and operator. This issue is those HTTP routes. It does not create short links.

## Scope

In this issue, all behind `Access.require(..., {"admin"})`:

- `POST /users` body `username`, `password`, `role`. `201` and `{id, username, role, status}` with `status=active`. `409` duplicate username. `400` for a bad username, password, or role.
- `GET /users` and `GET /users/{id}`. `404` for an unknown id.
- `PATCH /users/{id}` optional `role`, `password`, `status`. `409` when the result would leave zero active admins.
- `DELETE /users/{id}`. `204`. `409` when the target is the caller or the last active admin. `404` when missing.
- A marketer token receives `403`. No token receives `401`.
- Responses never include `password_hash`.

The handler calls `accounts`. It does not hash passwords itself and it does not write SQL.

## Plan

1. Register `/users` before any `GET /{code}` catch-all, even if the catch-all does not exist yet.
2. Map `Unauthenticated` to `401`, `Forbidden` to `403`, `Conflict` to `409`, `NotFound` to `404`, `Invalid` to `400`.
3. HTTP-test create, list, get, patch, delete, the last-admin cases, and the marketer `403`.

## Good and bad

Good: `POST /users` calls `accounts.create(...)` and returns the `User` fields.

Bad: the handler checking `count_active_admins` itself, or returning the stored hash “for debugging.” Bad: a public `POST /users` so people can sign themselves up.

## Done when

- [ ] An admin can create a user and then list and fetch that user without a password hash in the JSON.
- [ ] Patching a password changes the next `authenticate` result.
- [ ] The last-admin and self-delete cases return `409` and leave the row in place.
- [ ] A marketer token gets `403` on `GET /users`. A missing token gets `401`.

Auth golden files to add once S12 exists: `login_ok`, `login_bad_password`, `login_disabled`, `admin_create_user`, `duplicate_username`, `marketer_cannot_list_users`, `delete_last_admin`. Until then, cover them with ordinary tests.
:::end

:::issue
id: S1
title: [S1] Create a short link and redirect an active code
labels: wave-1,required,type-feature,area-links,area-redirect
depends_on: S0.2
:::
Slice: S1

## Why

A marketer needs to create `abc123`, and a visitor needs to be sent to the destination without a token. This is the first real redirect. It reads the database on every request. Cache, queue, and ledger are later slices.

## Scope

In this issue:

- `LinkStore.insert` and `get`. `POST /links` body `code`, `destination_url`, `affiliate_id`. Caller must be an active `marketer` or `admin`.
- Code: 4–32 characters, `A–Z` `a–z` `0–9`. Destination: `http` or `https` only. Stored row: `status=active`, `version=1`.
- `201` with the link. `409` if the code exists. `400` if the code or URL is invalid. `401` with no token. `403` for finance or operator.
- `GET /{code}` with no token. Active link: `302` and `Location` set to `destination_url`. Missing code: `404` (S2 covers inactive in full; a missing code already 404s here).
- `GET /health` returns `200` with a short plain body and no queue or spill numbers.
- Register `/login`, `/users`, `/links`, and `/health` before `GET /{code}`.

Out of this issue: `PATCH`, cache, `ClickCandidate`, ledger insert. `GET` must not write a click row.

## Plan

1. Add link validation in `admin` (the link module, not the admin role).
2. `POST /links` calls `Access.require` then `admin.create`.
3. `GET /{code}` calls `LinkStore.get` and returns `302` or `404`. It does not call `Access`.
4. Test create, conflict, validation, 401, 403, and a public 302.

## Good and bad

Good: the GET handler is a few lines that call `redirect.resolve(code)` and set `Location`. `redirect` does not import `users`, `tokens`, `access`, or the ledger.

Bad: `GET /abc123` requiring a JWT. Bad: inserting into `billable_clicks` before the 302. Bad: accepting `javascript:` or a bare hostname as the destination.

Example:

```http
POST /links
Authorization: Bearer <marketer>
{"code":"abc123","destination_url":"https://shop.example/a","affiliate_id":"aff-1"}
```

```http
HTTP/1.1 201
{"code":"abc123","destination_url":"https://shop.example/a","affiliate_id":"aff-1","status":"active","version":1}
```

## Done when

- [ ] A marketer or admin can create a link and a second create of the same code returns `409`.
- [ ] `GET /abc123` with no token returns `302` to that URL.
- [ ] Finance receives `403` on `POST /links` and no row is written.
- [ ] `GET /health` with no token returns `200`.
:::end

:::issue
id: S2
title: [S2] Update a link and refuse a missing or inactive code
labels: wave-1,required,type-feature,area-links,area-redirect
depends_on: S1
:::
Slice: S2

## Why

A marketer must change the destination or turn a link off, and the next visitor must see that. An inactive link must not be treated as a successful redirect.

## Scope

In this issue:

- `PATCH /links/{code}` for `marketer` or `admin`. Optional `destination_url`, optional `status` (`active` or `inactive`).
- `LinkStore.update` writes the database and increments `version` by 1.
- Unknown code: `404`. Invalid URL or status: `400`.
- `GET` of a missing or inactive code: `404`. The queue, if it already exists, is unchanged. In this wave the queue may not exist yet, so the test is: no candidate is created and no ledger write happens.
- Any marketer or admin may patch any code. Do not add an owner column.

Still no cache invalidation (S7) and no enqueue (S3).

## Plan

1. Implement update and the version bump in `LinkStore`.
2. `PATCH` calls `Access.require` then the link admin module.
3. Teach `redirect` to return 404 for `inactive` as well as missing.
4. Test both patch fields, version 1 → 2, unknown code, and GET 404 for inactive.

## Good and bad

Good: after `PATCH` sets `status=inactive`, `GET /abc123` is `404` and `version` is one higher.

Bad: leaving `version` unchanged, so a later cache cannot tell two writes apart. Bad: redirecting an inactive link and “just not counting it.” The visitor must not be sent.

## Done when

- [ ] A destination change is stored and `version` increases by 1.
- [ ] An inactive link and a missing code both return `404` on `GET` with no token.
- [ ] `PATCH` of an unknown code returns `404`.
- [ ] A finance token cannot patch.
:::end

:::issue
id: S3
title: [S3] Enqueue a click candidate, then redirect
labels: wave-2,required,type-feature,area-attribution,area-redirect
depends_on: S2
:::
Slice: S3

## Why

The visitor should leave before any credit is written. This issue adds the handoff: build a `ClickCandidate`, publish it, then return `302`. The worker that writes the ledger is S4.

## Scope

In this issue:

- `ClickQueue` protocol and `MemoryClickQueue` (`publish`, `pull`, `ack`, `nack`). Unbounded. Pull moves the item to in-flight. `nack` returns it. Process exit drops the queue.
- `ClickCandidate` fields from the PRD: `event_id`, `code`, `affiliate_id`, `destination_url`, `link_version`, `clicked_at` (UTC, set now), `identity`, `idempotency_key` (empty string when absent), `billable`.
- `identity` is the `visitor_id` cookie when the request sent one, otherwise `ua:` plus the `User-Agent` (which may be empty). Set a `visitor_id` cookie only when the request did not send one.
- Active link: `publish`, then `302`. Inactive or missing: `404` and no publish.
- `billable` may be `True` for every active click in this issue. Prefetch is S9.
- After `302`, the ledger has no row for this click. If the ledger table does not exist yet, the test is that `GET` does not open a ledger connection. S4 adds the table.
- Log `redirect` with `code`, `status`, and `cache_hit`. `cache_hit` may be false until S6.
- Inject the queue into `redirect`. `redirect` still does not import the ledger.

Publish failure and the spill file are S11. In this issue a publish error can surface as a failed request. S11 changes that to spill-and-302.

## Plan

1. Implement the memory queue and unit-test pull, ack, and nack.
2. Build the candidate inside `redirect` and publish it only for an active link.
3. HTTP-test: one GET leaves queue depth 1, and a second object that stands in for the ledger is untouched.
4. Test that inactive and missing codes leave the queue depth unchanged.

## Good and bad

Good: the handler publishes, and only then writes `302`. A test observes queue depth 1 and ledger length 0.

Bad: `GET` calling `Ledger.credit` and then redirecting. Bad: generating `clicked_at` inside the worker later, so a delay changes the time bucket.

## Done when

- [ ] An active `GET` returns `302` and one candidate with that code, affiliate, and `clicked_at`.
- [ ] The ledger is unchanged by that `GET`.
- [ ] Missing and inactive codes return `404` and do not publish.
:::end

:::issue
id: S4
title: [S4] Credit the ledger and let finance read it
labels: wave-2,required,type-feature,area-attribution
depends_on: S3
:::
Slice: S4

## Why

The affiliate is paid from stored clicks, not from the fact that a visitor was redirected. This issue adds the worker and the finance read. Dedup of two different GETs is S5. This issue still inserts one row per candidate, and a second insert of the same `event_id` must be safe once S5 adds the unique key. For now, `credit` inserts the row and a repeated `event_id` is the conflict case you can already return as `Duplicate` if the column is unique.

## Scope

In this issue:

- Migrate `billable_clicks`: `dedup_key` primary key, `event_id` unique, `code`, `affiliate_id`, `clicked_at`. Until S5, `dedup_key` can be the `event_id` so the primary key exists. S5 replaces that with the real hash. Write that down in the migrate comment so S5 does not leave both meanings in one function without noticing.
- `Ledger.credit(candidate) -> Credited | Duplicate`. Duplicate is a return value, not an exception. The worker acks on both.
- `Ledger.list(affiliate_id | None, limit)` ordered by `clicked_at`. Default limit 100, max 1000.
- Worker: pull, skip when `billable` is false (nothing sets false until S9), `credit`, ack. `nack` if `credit` raises.
- `GET /ledger` for `finance` or `admin`. Optional `affiliate_id` and `limit`. Rows: `event_id`, `code`, `affiliate_id`, `clicked_at`. A marketer receives `403`.
- Log `credited` or `duplicate` with `code` and `event_id`.

The worker is the only writer of the ledger. `redirect` does not call `credit`.

## Plan

1. Add the table and `Ledger` with an injected connection.
2. Add the worker function. Tests call it directly. A background thread is optional and not required for the unit test.
3. Add `GET /ledger` through `Access.require`.
4. Test one candidate becomes one row, and a marketer cannot list.

## Good and bad

Good: the test publishes a candidate, asserts the ledger is empty, pumps the worker once, and asserts one row.

Bad: the HTTP handler inserting the ledger row so the 302 waits on the write. Bad: `GET /ledger` with no auth because “it is only a read.”

## Done when

- [ ] Pumping the worker after one active click writes one ledger row for that affiliate.
- [ ] `GET /ledger` as finance returns that row. A marketer receives `403`.
- [ ] A second `credit` of the same `event_id` returns `Duplicate` and the row count stays 1.
- [ ] `attribution` does not import the HTTP package.
:::end

:::issue
id: S5
title: [S5] Dedup credits with a click-time key
labels: wave-2,required,type-feature,area-attribution
depends_on: S4
:::
Slice: S5

## Why

A browser retry must not pay twice. The key is computed from values stored on the candidate at click time, and the ledger primary key makes the second apply a no-op.

## Scope

In this issue:

- `domain` function, no I/O. When `idempotency_key` is non-empty, S10 defines that branch. In this issue the key is `sha256(code + "|" + identity + "|" + bucket)` with `bucket = clicked_at_unix // 600`.
- Add `CLICK_BUCKET_SECONDS` default `600` to settings.
- `Ledger.credit` uses this hash as `dedup_key`. A conflict returns `Duplicate`. The worker acks.
- Same candidate delivered twice: one row.
- Two GETs, same identity, no idempotency key, same bucket: one row.
- Two GETs with `clicked_at` 600 seconds apart: two rows.
- Boundary: a timestamp at 599 seconds and one at 600 seconds are different buckets. Freeze `clicked_at` on the candidate. Do not recompute the bucket from the worker clock.

The Idempotency-Key branch is S10. This issue can ignore a non-empty key by still hashing identity and bucket, as long as S10 replaces that path and its tests fail until then. Cleaner: if the key is non-empty, still hash identity and bucket in this issue, and say so in a one-line comment that S10 owns the other formula. Do not pretend both formulas are done.

## Plan

1. Put the hash function in `domain` and unit-test the bucket boundary with fixed timestamps.
2. Change `credit` to store that hash. Migrate existing temp tests so they still pass.
3. Worker test: two deliveries of one candidate, two GETs in one bucket, two GETs in two buckets.

## Good and bad

Good: both candidates carry `clicked_at`. The worker hashes those fields. The second insert hits the primary key and returns `Duplicate`.

Bad: hashing `time.time()` inside the worker, so a slow queue splits one retry into two payments. Bad: an in-memory `set` on the redirect process as the only dedup.

## Done when

- [ ] Bucket edges 599 and 600 produce different hashes for the same code and identity.
- [ ] Two GETs in one bucket credit one row. Two buckets credit two rows.
- [ ] Replaying one candidate credits one row.
:::end

:::issue
id: S6
title: [S6] Serve hot links from an in-memory cache
labels: wave-3,required,type-feature,area-cache
depends_on: S5
:::
Slice: S6

## Why

A popular code should not read SQLite on every click. The cache is process-local and loses its entries on exit. The database stays the source of truth.

## Scope

In this issue:

- `LinkCache` protocol and `MemoryLinkCache`: `get`, `put`, `invalidate`. `get` returns `None` on a miss or when the entry is past TTL. Check TTL on read. No background sweeper.
- `CACHE_TTL_SECONDS` default 60.
- Cache key `link:{code}`. Value is the link fields including `version`.
- On a miss, load from `LinkStore`, then `put` when a row was found. Do not `put` a missing code.
- Cache both active and inactive rows that were found.
- Redirect uses `LinkCache` then `LinkStore`. Inject both. Add `cache_hit` to the existing redirect log.
- A counting wrapper around `LinkStore` is allowed in tests only.

Invalidation on write is S7. This issue can leave a stale entry until TTL. Do not call `invalidate` from `PATCH` yet, or S7 has nothing to add. Tests that need a second read to hit the cache should avoid an update between the two reads.

## Plan

1. Implement `MemoryLinkCache` with an injected clock. Unit-test put, get, expiry, and invalidate.
2. Wire it into the read path in `main`.
3. HTTP-test two GETs of an active code and assert the store read count stays 1.

## Good and bad

Good: the first GET reads SQLite and fills the cache. The second GET before TTL does not call `LinkStore.get`.

Bad: caching `None` for an unknown code, so a link created a second later stays 404 until TTL. Bad: a process-global dict created inside the redirect function on first use.

## Done when

- [ ] Two clicks on a warm link do one store read.
- [ ] An expired entry is a miss. The next get loads the store again.
- [ ] A missing code is not stored in the cache.
:::end

:::issue
id: S7
title: [S7] Invalidate the cache when a link is written
labels: wave-3,required,type-feature,area-cache,area-links
depends_on: S6
:::
Slice: S7

## Why

After a marketer edits or deactivates a link, the next visitor must see the database, not the previous cache entry. TTL is only the backup for a missed delete.

## Scope

In this issue:

- Link update sequence: commit the SQLite update, then `cache.invalidate(code)`.
- If invalidate raises, log it and still return `200`. The database is already correct.
- Create (`POST`) does not need to warm the cache.
- Tests: warm the cache with a GET, patch the destination, GET again, and expect the new `Location`. Warm, deactivate, GET, expect `404`.
- The cache key is gone before that following GET. A test can spy on `invalidate`.

## Plan

1. Pass `LinkCache` into the link admin module.
2. Call `invalidate` only after a successful commit.
3. HTTP-test destination change and deactivate from a warm cache.

## Good and bad

Good: `update` commits, then deletes `link:abc123`. The next GET misses, reads SQLite, and caches the new row.

Bad: updating the cache in place and skipping the database write. Bad: waiting 60 seconds for TTL as the only way a deactivate takes effect. Bad: deleting the cache key and rolling back the SQL when the delete fails.

## Done when

- [ ] A warm link that is deactivated returns `404` on the next GET.
- [ ] A warm link whose destination changes redirects to the new URL on the next GET.
- [ ] `version` still increments, from S2.
- [ ] A failed invalidate after a committed update still returns `200`.
:::end

:::issue
id: S8
title: [S8] Reload from the database when the cache entry expires
labels: wave-3,required,type-feature,area-cache
depends_on: S7
:::
Slice: S8

## Why

Invalidate-on-write is the normal path. TTL is the safety net for a write that did not go through the link admin module, or for an invalidate that failed. This issue proves that net.

## Scope

In this issue:

- A test may set the store row inactive without calling the link admin module, advance a fake clock by 61 seconds, and GET.
- The response is `404`, loaded from the database.
- Production code does not grow a second writer. The direct store write exists only in the test.

## Plan

1. Thread the same fake clock into `MemoryLinkCache` in the HTTP test.
2. Warm the cache, bypass `invalidate`, expire the entry, GET.
3. Keep the unit test from S6. This issue adds the HTTP-level safety net.

## Good and bad

Good: after the clock passes TTL, `get` returns `None`, the redirect loads SQLite, and the visitor follows the new status.

Bad: a background thread that scans all keys every second. Bad: treating a failed test of this path as optional because S7 already deletes the key.

## Done when

- [ ] With a warm cache, a direct store deactivate, and a clock past TTL, `GET` returns `404`.
- [ ] Before the clock passes TTL, that same direct store change is still hidden by the cache. That shows TTL is the boundary.
:::end

:::issue
id: S9
title: [S9] Redirect a prefetch and leave it off the ledger
labels: wave-4,required,type-feature,area-attribution
depends_on: S8
:::
Slice: S9

## Why

Browsers prefetch links. The visitor should still arrive if the link is active, and finance should not pay for that request.

## Scope

In this issue:

- `billable` module. First rule: `Purpose` or `Sec-Purpose` contains `prefetch`, case-insensitive.
- `redirect` asks the rule and stores the boolean on the candidate. It still publishes and still returns `302` for an active link.
- The worker already skips `billable=false`. It acks that candidate and does not call `credit`.
- A new rule later is another object in a list. Do not add a fraud implementation.

## Plan

1. Implement the header check with unit cases for `Purpose`, `Sec-Purpose`, and mixed case.
2. Pass a list of one rule into `redirect`.
3. HTTP plus worker: prefetch GET is `302`, and the ledger stays empty.

## Good and bad

Good: the candidate is `billable=false`, the worker acks it, and the ledger length stays 0.

Bad: returning `404` for a prefetch. The offer is still valid. Bad: writing the ledger row and then deleting it. Bad: a comment that says “drop prefetch” inside the 302 handler with no rule object.

## Done when

- [ ] `Purpose: prefetch` on an active link returns `302` and, after the worker, the ledger is unchanged.
- [ ] A normal GET of that link still credits one row.
- [ ] `redirect` does not import `Ledger`.
:::end

:::issue
id: S10
title: [S10] Collapse retries that share an Idempotency-Key
labels: wave-4,required,type-feature,area-attribution
depends_on: S9
:::
Slice: S10

## Why

When the caller sends `Idempotency-Key`, that key is the identity of the click. Two retries with the same key are one credit even if the cookie or the time bucket differs.

## Scope

In this issue, replace the non-empty branch left open in S5:

- Non-empty `idempotency_key`: `dedup_key = sha256(code + "|" + idempotency_key)`.
- Empty key: keep the S5 formula (`code`, `identity`, bucket).
- The header is stored on the candidate at click time.
- Tests: two GETs, same key, different cookies, clock advanced 600 seconds, one ledger row. Two different keys, two rows.

## Plan

1. Change the `domain` function and add unit tests that ignore cookie and bucket when the key is present.
2. Confirm the worker still uses `domain` and does not hash a second way.
3. Two-GET worker test for both cases.

## Good and bad

Good: both candidates store `idempotency_key="req-7"`. Both hashes match. The second `credit` returns `Duplicate`.

Bad: folding the key into the bucket hash, so the same key in the next bucket pays again. Bad: requiring the header on every browser click. The header stays optional.

## Done when

- [ ] The same key across cookies and buckets is one ledger row.
- [ ] Two keys are two rows.
- [ ] A click with no key still follows the S5 bucket rule.
:::end

:::issue
id: S11
title: [S11] Spill a failed enqueue and expose ops status
labels: wave-4,required,type-feature,area-spill
depends_on: S10
:::
Slice: S11

## Why

If the queue rejects the candidate, the visitor should still be redirected, and the click should be retried. The operator needs a way to see the backlog.

## Scope

In this issue:

- `SPILL_PATH` default `data/clicks.jsonl`. Append one JSON object per failed publish. Flush the write.
- On publish error: log `enqueue_failed` with `code` and `event_id`, append the candidate, log `spill_depth`, then return `302`.
- If the spill write also fails, still return `302` and log the error. That click can be lost. Say that in the log line.
- Drainer reads the file, `publish`es each line, and removes the lines that published. It calls the same publisher `redirect` uses.
- `GET /ops/status` for `operator` or `admin`: `{spill_depth, queue_depth}`. `queue_depth` is the attribution queue, including in-flight items or only waiting items — pick waiting items (not yet pulled) and document it in the handler docstring. A marketer receives `403`.
- `GET /health` stays a plain `200` with no depths.

`MemoryClickQueue.publish` in production does not fail on its own. Tests inject a publisher that raises. Do not add a real outage.

## Plan

1. Implement spill append and drain with a temp file.
2. Change `redirect` so a publish exception takes the spill path and still returns `302`.
3. Add the ops route through `Access.require`.
4. Test: failing publisher, `302`, spill length 1, drain, worker, ledger length 1, spill length 0.

## Good and bad

Good: the visitor receives `302` while the candidate sits on disk, and a later drain credits it once.

Bad: returning `500` so the click is never lost and the visitor never leaves. Bad: writing the spill row into `billable_clicks` directly and skipping the worker. Bad: putting `spill_depth` on `GET /health`.

## Done when

- [ ] A raising publisher still yields `302`, one spill line, and an `enqueue_failed` log.
- [ ] Drain then the worker produces one ledger row and an empty spill file.
- [ ] An operator can read `spill_depth` and `queue_depth`. A marketer receives `403`.
- [ ] A spill write that also fails still returns `302`.
:::end

:::issue
id: S12
title: [S12] Add the golden harness
labels: wave-5,required,type-test,area-tests
depends_on: S11
:::
Slice: S12

## Why

The product rules need one replay that fails when a status code, `Location`, billable flag, or ledger count drifts. Earlier slices keep their unit tests. This issue adds the runner and the checked-in scenarios.

## Scope

In this issue:

- `tests/harness/runner.py` wires the real redirect, link admin, access, accounts, and worker with temp SQLite, `MemoryLinkCache`, `MemoryClickQueue`, a temp spill file, a fake clock, `JWT_SECRET`, and one active user per role (`admin`, `marketer`, `finance`, `operator`, password `password-ok`).
- A counting `LinkStore` wrapper lives only in the harness.
- Core steps: `create_link`, `patch_link`, `click`, `pump_worker`, `drain_spill`, `advance_clock`, `fail_publish`, `credit_again`.
- Auth steps: `login`, `create_user`, `patch_user`, `delete_user`, `list_users`, `get_ledger`, `get_ops`.
- `create_link` and `patch_link` send the seeded marketer token. `click` sends no token.
- Compare the fields the scenario’s `expect` names. Ignore `event_id` and the raw hash. On failure, print the scenario name and the field diff.
- Add the files listed in PRD §10.5 (`tests/golden/`) and §10.6 (`tests/golden/auth/`). Do not add dashboard files.

## Plan

1. Build the runner and one scenario (`active_redirect`) end to end.
2. Add the rest of the core files, then the auth files.
3. Confirm a deliberate wrong `Location` fails the run with a readable diff.
4. Leave the older unit tests in place.

## Good and bad

Good: `tests/golden/active_redirect.json` creates `abc123`, clicks, pumps the worker, and expects `302`, `https://shop.example/a`, ledger length 1, spill 0.

Bad: asserting the raw `dedup_key` hex in the golden file. Bad: a harness that reimplements redirect in the test instead of calling the real modules. Bad: requiring every core file to assert `seen` before the dashboard exists.

## Done when

- [ ] `pytest` runs every file in `tests/golden/` and `tests/golden/auth/`.
- [ ] A tampered expectation fails with the scenario name and the field that differed.
- [ ] Core click scenarios send no token and still pass.
:::end

:::issue
id: S13
title: [S13] Add the click smoke test
labels: wave-5,required,type-test,area-tests
depends_on: S12
:::
Slice: S13

## Why

The golden files call modules through the harness. This issue boots the real process over HTTP once and checks the path a marketer and a visitor actually use.

## Scope

In this issue, `tests/smoke/test_click_once.py`:

- Ephemeral port, temp database, temp spill file, in-memory cache and queue, `JWT_SECRET`, bootstrap admin.
- `POST /login` as the bootstrap admin, `POST /users` for a marketer, `POST /login` as that marketer.
- `POST /links` for `abc123`.
- `GET /abc123` with no token expects `302` and `Location`.
- Ledger length is 0 until the worker is pumped, then 1. If the server pumps on a thread, wait until the count is 1 or the timeout fails. Do not `sleep` a fixed long time.
- `GET /health` with no token returns `200`.

## Plan

1. Start the app from `main` with the temp settings, or call the same wiring `main` uses and bind an ephemeral port.
2. Run the six steps with the standard library HTTP client.
3. Shut the server down at the end of the test.

## Good and bad

Good: the test fails if nobody credits the click, because it looks at the ledger after a pump or a short wait loop with a deadline.

Bad: a smoke test that only checks `302` and never the ledger. Bad: `time.sleep(5)` as the proof the worker ran.

## Done when

- [ ] The smoke test passes against the wired app.
- [ ] The public GET carries no `Authorization` header.
- [ ] The ledger has one row for the affiliate on `abc123`.
:::end

:::issue
id: S14
title: [S14] Run lint, format, and tests as one gate
labels: wave-5,required,type-chore,area-foundation,area-tests
depends_on: S13
:::
Slice: S14

## Why

S0 introduced ruff and pytest. This issue makes the finished tree pass one documented gate, including tests and the modules added since S0.

## Scope

In this issue:

- `pyproject.toml` holds the ruff select list, line length 100, and `src` plus `tests` as the paths.
- These three commands pass on the repo:

```text
ruff check src tests
ruff format --check src tests
pytest
```

- Public functions added so far have type hints. Ports do not use a bare `Any`.
- No new runtime dependency.

Do not add a CI vendor. Do not weaken a rule to make a file pass. Fix the file.

## Plan

1. Run the three commands and fix what they report.
2. Confirm a deliberate unused import fails `ruff check`.
3. Point at the three commands from the module docstring of `main`.

## Good and bad

Good: the same ruff config lints `tests/golden` helpers and `src`.

Bad: `# noqa` on a whole file. Bad: pinning a formatter other than ruff. Bad: adding a GitHub Actions workflow in this issue.

## Done when

- [ ] All three commands exit 0.
- [ ] The ruff config matches the PRD engineering bar.
- [ ] `main`’s docstring names the three commands.
:::end

:::issue
id: S15
title: [S15] Project approximate click counts
labels: wave-6,stretch,type-feature,area-dashboard
depends_on: S14
:::
Slice: S15

## Why

Marketers want a fresh count. That count is a second consumer of `ClickCandidate`, not a second ledger. This issue builds the projector and the in-memory map. HTTP and fan-out are S16.

## Scope

In this issue:

- `DashboardCounts.add(code, affiliate_id) -> seen` and `get(code) -> {affiliate_id, seen} | None`.
- `MemoryDashboardCounts` is a dict. Unknown code returns `None`. Two adds yield `seen` 2.
- `DashboardProjector` pulls a `ClickQueue`, calls `add`, and acks. A second delivery of the same candidate adds again.
- The projector counts prefetch and duplicates. It does not read `billable` and it does not import `Ledger`.
- Unit tests use a fake queue and a fake counts object. No HTTP route in this issue.

## Plan

1. Implement the dict store and the projector in `dashboard`.
2. Test add, missing get, ack after add, and a nack then redelivery that adds twice.
3. Leave `main` unwired until S16.

## Good and bad

Good: two deliveries of one candidate produce `seen == 2`. The ledger is not imported.

Bad: skipping the add when `dedup_key` was already seen. That would make the dashboard a second ledger. Bad: incrementing the count inside `Ledger.credit`.

## Done when

- [ ] `add` then `get` returns the affiliate and the new `seen`.
- [ ] A redelivery increases `seen` again.
- [ ] `dashboard` does not import `Ledger` or the HTTP package.
:::end

:::issue
id: S16
title: [S16] Fan out the candidate and serve the dashboard
labels: wave-6,stretch,type-feature,area-dashboard
depends_on: S15
:::
Slice: S16

## Why

Redirect should still call `publish` once. A fan-out in `main` copies the candidate to the attribution queue and the dashboard queue. The marketer reads `seen` over HTTP. Finance still pays from the ledger.

## Scope

In this issue:

- `FanoutPublisher.publish`: attribution queue first, then dashboard queue. `main` is the only place that constructs it. `redirect` keeps calling one publisher.
- If the attribution publish raises, the dashboard publish is not called, and the error propagates so the S11 spill path still applies.
- If the dashboard publish raises, log `dashboard_enqueue_failed` with `code` and `event_id`, and return normally. The attribution candidate stays queued.
- The spill drainer calls this same publisher, so a later successful publish is offered to both queues.
- `GET /dashboard/links/{code}` for `marketer` or `admin`, via `Access.require`. `200` body: `code`, `affiliate_id`, `seen`, `approximate: true`. Missing link: `404`. Existing link with no candidates: `seen` 0. No token: `401`.
- The handler uses `LinkStore.get` and `DashboardCounts.get`. It does not call `Ledger.credit`.
- Deactivating a link does not change `seen`.
- Register `/dashboard` before `GET /{code}`.

## Plan

1. Implement `FanoutPublisher` and unit-test both failure orders.
2. Wire it and the projector in `main`.
3. Add the read route and the HTTP cases: `seen` 0, `404`, `401`, deactivate leaves `seen`.

## Good and bad

Good: one `publish` from redirect. Attribution success plus dashboard failure still returns `302` from the existing redirect path, and the ledger can still reach 1.

Bad: `redirect` importing `dashboard` and incrementing a dict before the 302. Bad: returning the ledger count as `seen`. Bad: omitting `approximate: true`.

Example body:

```json
{"code": "abc123", "affiliate_id": "aff-1", "seen": 2, "approximate": true}
```

## Done when

- [ ] A quiet link returns `seen` 0 and `approximate` true for a marketer.
- [ ] An unknown code returns `404`. A missing token returns `401`.
- [ ] Dashboard publish failure does not block attribution publish.
- [ ] Attribution publish failure does not call the dashboard queue.
- [ ] A deactivate leaves `seen` unchanged.
:::end

:::issue
id: S17
title: [S17] Lock the dashboard with golden files and smoke
labels: wave-6,stretch,type-test,area-dashboard,area-tests
depends_on: S16
:::
Slice: S17

## Why

The stretch is done when a replay can show `seen` moving independently of the ledger, including a failed dashboard publish that still credits.

## Scope

In this issue:

- Extend the S12 runner with `pump_dashboard` and `fail_dashboard_publish`. Core files do not set `seen`, and the runner does not require it.
- Dashboard reads use the seeded marketer token. `click` still sends no token.
- Add `tests/golden/dashboard/` with the eight files from PRD §13.6: `seen_before_credit`, `prefetch_seen`, `duplicate_diverges`, `quiet_link`, `missing_code`, `restart_clears_counts`, `projector_redelivery`, `dashboard_publish_fails`.
- Add `tests/smoke/test_dashboard_seen.py`: login as marketer, create `abc123`, dashboard `seen` 0, public `302`, pump dashboard so `seen` is 1 while the ledger is still 0, pump attribution so the ledger is 1 and `seen` stays 1.

Do not add charts, date buckets, or an affiliate-wide total.

## Plan

1. Add the two runner steps and `seen` on `expect`.
2. Add the eight JSON files.
3. Add the smoke test using the same boot path as S13, with the fan-out wired.
4. Re-run `tests/golden/` and confirm the core files still pass.

## Good and bad

Good: `duplicate_diverges.json` expects `seen` 2 and ledger length 1. `seen_before_credit.json` expects `seen` 1 and ledger length 0.

Bad: changing `active_redirect.json` to require `seen`. Bad: a dashboard golden that expects `seen` to equal the ledger length in the duplicate case.

## Done when

- [ ] The eight dashboard scenarios pass.
- [ ] The core golden files pass without a `seen` field.
- [ ] The dashboard smoke test shows `seen` 1 before the ledger row exists, then one ledger row with `seen` still 1.
- [ ] `ruff check`, `ruff format --check`, and `pytest` pass.
:::end
