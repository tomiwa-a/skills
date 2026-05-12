# Attack Vectors: Architectural Weakness Categories

When performing a design review, systematically probe every category below. Do not skip any section even if a category seems "obviously fine."

## Category 1: Data Integrity & Concurrency [HIGHEST RISK]

These are bugs that silently corrupt production data.

**Probe Questions:**
- Is there a race condition on any write path? (e.g., Two requests creating the same user record simultaneously before the unique constraint fires.)
- Are critical multi-step writes wrapped in a database transaction? (e.g., Writing `credit_agreements` AND `audit_logs` must be atomic.)
- Is there an idempotency mechanism for external webhook ingestion? What happens if Mono sends the same webhook twice?
- Can a foreign key constraint be violated? (e.g., A `repayment_history` row referencing a `credit_agreements.id` that doesn't exist yet.)
- Are there any soft-delete patterns that could expose "deleted" records through a JOIN?

## Category 2: CAP Theorem & Caching [HIGH RISK]

These are bugs that serve incorrect data to users.

**Probe Questions:**
- Is there a Cache Stampede risk? If the Redis cache for a credit report expires, and 500 lenders simultaneously query for the same user, do 500 requests all hit the database at once?
- Is there a stale read risk? (e.g., A lender revokes an API key, but a 5-minute cache means the revoked key works for 5 more minutes.)
- Are you caching data that MUST be consistent? (Cross-reference against `plan/technical/CAP.md`.)
- What is the cache invalidation strategy? Is it TTL-only (weak) or event-driven (strong)?

## Category 3: Security [HIGH RISK]

**Probe Questions:**
- Is any PII (BVN, phone number) stored in plaintext anywhere — even in logs?
- Are all SQL queries using parameterized inputs? Is there any string-formatted SQL?
- Is the JWT secret stored in an environment variable, or is it hardcoded anywhere in the codebase?
- What happens if an attacker replays a valid webhook payload with a modified body? Is HMAC verification done on every ingestion?
- Is the `audit_logs` table truly append-only? Could an attacker with DB access UPDATE or DELETE rows?
- Are API keys stored as plaintext or only as hashes?

## Category 4: Scalability & Performance [MEDIUM RISK]

**Probe Questions:**
- Are there database queries running without an index on the search column? (e.g., `WHERE bvn_hash = ?` without an index on `bvn_hash`.)
- Is any endpoint doing N+1 queries? (e.g., Loading 25 reports, then querying the DB once per report to get lender name.)
- Are large payloads (e.g., 12 months of Mono transactions) being loaded fully into memory before processing?
- Is the background worker pool size configurable, or is it fixed?

## Category 5: Operational & Observability [MEDIUM RISK]

**Probe Questions:**
- If the background worker crashes mid-job, is the job re-queued, or is it lost forever?
- Is there a Dead Letter Queue (DLQ) for jobs that fail all retries?
- If the Mono API is down, does the system fail gracefully or does it return a 500?
- Can the team see which jobs are currently in the queue, and how old they are?
- If the system sends a bad webhook to a lender and they complain, can the team replay just that webhook?

## Category 6: Business Logic Edge Cases [MEDIUM RISK]

**Probe Questions:**
- What happens if the same BVN is submitted simultaneously by two different lenders?
- What happens if a Consumer freezes their profile at the exact same moment a score request is being processed?
- What is the behavior when `confidence_level` is `INSUFFICIENT`? Does the API return 422, or does it return a partial score?
- A loan is reported as `PERFORMING` this month. Next month the Subscriber doesn't send an update. Does the loan stay `PERFORMING` forever, or does it decay?
