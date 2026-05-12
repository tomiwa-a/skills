# Entity Patterns: Standard Database Schema Design

Apply these patterns consistently to every project's `plan/technical/entities.md`.

## Pattern 1: The Dual-ID Strategy (Non-Negotiable)

Every table must have two ID fields:
- `id` (BIGINT, auto-increment): The internal PK used for all foreign keys and JOIN operations. Keeps B-Tree indexing fast.
- `public_id` (UUID v7): Time-ordered, non-guessable. Used in all API responses and URL parameters. Never used in JOINs.

**Why:** Exposing sequential integer IDs in URLs lets attackers enumerate resources. UUID v4 is random and unindexable. UUID v7 is time-ordered (fast to index) and non-guessable (secure).

## Pattern 2: Auth / Profile Separation (The Identity Pattern)

For any system where multiple user types exist, separate authentication from business metadata:

```
identities          ← Auth table. One row per login. No business data.
  ↓ (1:1)
admin_profiles      ← Business metadata for admins.
lender_profiles     ← Business metadata for lenders.
```

**Why:** Auth logic (password hashing, session invalidation) is isolated from business logic. A bug in the lender profile update cannot corrupt the auth table.

## Pattern 3: The Audit Layer

Every production system must have an `audit_logs` table with these characteristics:
- `payload` (BYTEA): Stores `old_values` and `new_values` as gzip-compressed, base64-encoded JSON.
- `created_at` only: No `updated_at`. This table is append-only.
- No `public_id`: Audit logs are never exposed via API.
- `id` must be sequential: Gaps in the sequence indicate deleted rows.

**Enforcement (3 layers):**
1. PostgreSQL GRANT: Only INSERT privilege granted to the app role.
2. Trigger: BEFORE UPDATE OR DELETE raises an exception.
3. RLS Policy: Only SELECT and INSERT permitted for the app role.

## Pattern 4: Enum Over Boolean

Prefer enums over boolean flags where a field might gain states in the future.

| Bad | Good |
|---|---|
| `is_verified: Boolean` | `kyc_status: Enum(PENDING, VERIFIED, REJECTED)` |
| `is_active: Boolean` | `account_status: Enum(OPEN, CLOSED, SUSPENDED)` |

**Exception:** `is_revoked`, `is_enabled`, `is_restructured` are acceptable as booleans when the states are truly binary and permanent.

## Pattern 5: Nullable vs. Required

- Fields that are required at creation time: NOT NULL.
- Fields that are set after creation: Nullable (e.g., `closed_date`, `resolved_at`, `revoked_at`).
- Fields used for soft relations: Nullable FK (e.g., `report_id` in `bureau_enquiries` — the report may not exist yet).

## Pattern 6: Denormalization for Performance

In high-read tables, it is acceptable to store a denormalized foreign key value to avoid expensive JOINs:

```
repayment_history.user_id ← Denormalized from credit_agreements.user_id
```

This allows `WHERE user_id = ?` queries on `repayment_history` without joining through `credit_agreements`.

Document every denormalized field with a comment explaining why.

## Pattern 7: Layer Organization

Organize tables by function, not by name:

1. **Authentication Layer**: `identities`
2. **Profile Layer**: `admin_profiles`, `lender_profiles`
3. **Core Domain Layer**: The main business entities
4. **Operations Layer**: State-tracking tables (e.g., `repayment_history`, `bureau_enquiries`)
5. **Consumer Rights Layer**: `consumer_consents`, `disputes`, `dispute_responses`
6. **Audit & Compliance Layer**: `audit_logs`
