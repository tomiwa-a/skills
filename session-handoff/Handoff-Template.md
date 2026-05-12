# Handoff Template: The Standard handoff.md Structure

Every session handoff must follow this exact structure. Fill in every section — do not leave any section empty or with a placeholder.

---

```markdown
# Session Handoff — [DATE e.g. 2025-05-12]

**Project:** [Project Name]
**Current Phase:** Phase N — [Phase Name]
**Session Duration:** [Approximate time spent]

---

## ✅ Completed This Session

A precise list of what was done. Map every item to the relevant `plan/tasks/` file.

- [x] Implemented `POST /v1/auth/register` with Argon2 password hashing (`tasks/phase-2-lender-onboarding.md`)
- [x] Added JWT middleware with `lender_id` extraction into request context
- [x] Created migration `0002_create_lender_profiles.sql` and verified up/down

---

## 🚧 In Progress (Not Yet Complete)

Tasks that were started but not finished. Be honest — a half-done task is a blocker.

- [ ] `POST /v1/keys/generate` — Endpoint scaffolded. Key prefix logic done. Key hashing NOT yet implemented. Next: add SHA-256 hash storage.

---

## 🚨 Blockers & Open Questions

Unresolved decisions, failing tests, or pending architectural questions that must be resolved before moving forward.

1. **[Decision Needed]** Should API key expiry be set at key generation time or configurable per-lender? Currently assuming no expiry for MVP but this should be confirmed.
2. **[Failing Test]** `TestJWTMiddleware_InvalidToken` is failing intermittently. Suspected race condition in test setup. Investigate `t.Parallel()` conflict.
3. **[Dependency]** `POST /v1/scores/request` (Phase 3) requires the Mono Sandbox API key. The key has not been provisioned yet.

---

## ⚡ Next Immediate Action

The exact next thing to do. One sentence. Names the file and the function.

> Open `internal/adapters/http/keys_handler.go` and implement the `GenerateKey` handler — specifically the SHA-256 hashing of the raw key before storing it in `api_keys.key_hash`.

---

## 📋 Plan Checklist Status

Current completion status of the active phase.

**Phase 2: Lender Onboarding** — 4/8 tasks complete.
- [x] Lender Registration
- [x] Lender Login & JWT
- [x] Auth Middleware
- [ ] API Key Generation ← IN PROGRESS
- [ ] API Key Auth Middleware
- [ ] API Key Revocation
- [ ] Webhook URL Registration
- [ ] Lender Profile (GET /v1/me)

---

## 🗂️ Files Changed This Session

A list of files that were created or modified, for quick orientation.

- `internal/adapters/http/auth_handler.go` — Created (Register + Login handlers)
- `internal/adapters/db/postgres/identity_repo.go` — Created (DB queries for identities)
- `migrations/0002_create_lender_profiles.up.sql` — Created
- `migrations/0002_create_lender_profiles.down.sql` — Created
- `pkg/auth/jwt.go` — Created (JWT generation + validation)
```

---

## Usage Notes

- **Filename Convention:** Always name it `handoff-YYYY-MM-DD.md` in the project root.
- **Multiple Sessions Same Day:** Append `-v2`, `-v3` to the filename.
- **After Reading a Handoff:** Immediately open the file named in "Next Immediate Action" before doing anything else.
