# Drift Detection: How to Compare Code vs. Plan

When a user asks for a code review or implementation check, use this process to detect drift between the current codebase and the established `plan/` directory.

## Step 1: Read the Plan Files First
Before looking at any code, read these files in order:
1. `plan/technical/entities.md` — What tables and fields should exist?
2. `plan/TRD.md` — What is the mandated tech stack?
3. `plan/tasks/phase-N-*.md` — What tasks are in scope for the current phase?
4. `plan/technical/req-res.md` — What is the expected API contract?

## Step 2: Check the Codebase
After reading the plan, inspect the code for:

### Database Drift
- Are there tables in the migrations that don't exist in `entities.md`?
- Are there fields missing from tables that the plan requires?
- Is the Dual-ID pattern (BIGINT `id` + UUID v7 `public_id`) applied to all tables?
- Do all new tables have `created_at` / `updated_at`? (Except `audit_logs` which has no `updated_at`.)

### Stack Drift
- Is the correct HTTP framework being used? (Cross-check against `plan/TRD.md` — e.g., if the TRD mandates Gin, using Fiber is drift; if it mandates Express, using Fastify is drift.)
- Is the correct logger being used? (Check `plan/TRD.md` for the specified logger.)
- Is the correct ORM or database driver being used? (Check `plan/TRD.md`.)
- Is the correct language runtime/version being used?

### Scope Drift
- Are there endpoints in the router that are not in any phase task file?
- Are there new third-party packages or external dependencies that aren't referenced in the TRD?
- Is the project structure (folders and modules) consistent with the architecture defined in `plan/TRD.md`?

### API Contract Drift
- Do all handler responses use the standard envelope `{"success": true, "data": ...}`?
- Are the correct HTTP status codes being used per `req-res.md`?
- Are paginated endpoints returning the correct `meta` structure?

## Step 3: Classify the Drift

| Drift Type | Severity | Action |
|---|---|---|
| Unauthorized new table | HIGH | Stop. Ask the user. |
| Wrong tech stack choice | HIGH | Stop. Reference TRD. |
| Missing required field | MEDIUM | Flag and ask. |
| Non-standard response envelope | MEDIUM | Flag and ask. |
| Ahead of phase (future feature) | MEDIUM | Flag and ask. |
| Minor naming inconsistency | LOW | Flag as a note. |

## Step 4: Report and Gate
- For HIGH severity: Do NOT proceed with any code writing until the user resolves the conflict.
- For MEDIUM severity: List all issues together and ask the user to confirm resolution before proceeding.
- For LOW severity: Include as a note at the end of your response. Do not block on it.
