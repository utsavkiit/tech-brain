# Reliable Write API Workflows

## In brief

A reliable write API preserves business invariants across invalid input, retries, concurrent requests, authorization boundaries, and partial failure. The route itself should remain thin: validate transport data, identify the caller, and delegate to domain and repository components whose contracts make duplicate and failure outcomes explicit.

## Model the lifecycle, not only the request

Keep client input distinct from stored state. A submitted receipt contains purchase facts; the stored receipt additionally has a server-generated ID, owner, processing status, and awarded points. This prevents clients from supplying server-controlled fields and lets the workflow represent intermediate states such as `RECEIVED` before `PROCESSED`.

Validate untrusted JSON at the boundary, then pass the validated value through the application. Represent money as integer minor units. Keep calculation rules independently composable so a new promotion does not require changing one large conditional. These practices build on [[typescript-api-foundations|TypeScript API Foundations]], especially its separation of runtime validation, domain values, and transport results.

## Idempotency is a stored protocol

An idempotency key protects a logical client operation from network retries. Scope the key to the authenticated member and store:

- the key and member ID;
- a deterministic fingerprint of the normalized request;
- processing state;
- the created resource ID;
- the original successful response and status.

Reserve the key atomically before side effects. A completed reservation with the same fingerprint replays the original result; the same key with a different fingerprint returns a conflict. A matching request already in progress must not start a second workflow.

Idempotency is different from receipt deduplication. A new key can still submit the same real-world receipt again, so business deduplication may also need a retailer transaction ID or carefully designed receipt fingerprint. Hashing raw JSON or an image alone is brittle because property order, normalization, image compression, or cropping can change the bytes without changing the logical request.

## Put uniqueness at the storage boundary

Application-level existence checks race under concurrency. The repository and production database must enforce the invariants:

```sql
PRIMARY KEY (receipt_id)
UNIQUE (member_id, receipt_id)
UNIQUE (member_id, idempotency_key)
```

Insert and update should be distinct operations: an insert must reject an existing ID instead of silently overwriting it. Expected conflicts should return explicit domain results such as `DUPLICATE_MEMBER_RECEIPT_AWARD`, ideally including the preserved original record, rather than ambiguous booleans.

An immutable points-award ledger provides auditability and prevents a balance column from being the only record of why a balance changed. A materialized balance can still serve fast reads, provided the ledger entry and balance update share an atomic boundary.

## Partial failure determines the production architecture

A receipt workflow may perform several writes:

```text
reserve idempotency key
→ insert receipt
→ calculate points
→ insert points award
→ mark receipt processed
→ complete idempotency record
```

An in-memory implementation demonstrates the contract but does not make this sequence atomic. A crash can leave a receipt in an intermediate state or an idempotency key permanently `IN_PROGRESS`. A production design needs a transaction where the stores share one database, or a recoverable state machine with an outbox/queue when work crosses process or service boundaries. It also needs retry rules, stale-reservation expiry or recovery, and reconciliation for records whose state disagrees.

## Authentication establishes the ownership boundary

Authenticate before reading idempotency keys or resources, then use the request identity throughout persistence and lookup. Ownership must be checked after loading a receipt and before returning either receipt details or its points. Returning the same not-found response for an absent receipt and another member's receipt reduces resource enumeration.

A fixed token map is acceptable for an interview exercise because it isolates middleware and authorization behavior. It is not a production identity system; JWT verification, token revocation, key rotation, and an external identity provider are separate concerns.

## Test invariants through public behavior

High-value tests cover behaviors that must survive refactoring:

- invalid boundary input produces structured, stable errors;
- point rules compose and honor threshold boundaries;
- an identical idempotent replay returns the same receipt and creates one award;
- a reused key with different input conflicts;
- the same key is independent for different members;
- duplicate receipt IDs and duplicate member/receipt awards preserve the original record;
- one member cannot retrieve another member's receipt or points;
- missing, malformed, and invalid credentials fail consistently.

The most important unresolved test is failure injection between workflow steps. It reveals whether retries recover safely or compound a partial write.

## Interview application

In a time-boxed API exercise, implement a narrow vertical slice first: validation, one working write flow, focused tests, and explicit error contracts. Describe database constraints, transaction boundaries, asynchronous processing, observability, and reconciliation after the core path works. The senior signal is not implementing every production component; it is recognizing which guarantees the simplified implementation does and does not provide.

## Sources

- Personal Fetch-style receipt rewards API mock interview and implementation session, 2026-08-06 through 2026-08-08.
