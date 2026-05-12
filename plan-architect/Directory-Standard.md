# Directory Standard: The Canonical plan/ Structure

Every project scaffolded by the Plan Architect skill must follow this structure exactly.

## The Full Directory Tree

```
plan/
├── BRD.md                        # Business Requirements Document (Non-technical)
├── MVP.md                        # MoSCoW prioritization, phase list, success definition
├── PHASES.md                     # High-level phase overview and goals
├── TRD.md                        # Technical Requirements Document (Stack, Architecture)
├── VIABILITY.md                  # Market viability analysis (optional but recommended)
│
├── technical/
│   ├── entities.md               # Full database schema with Dual-ID pattern
│   ├── req-res.md                # HTTP envelope, error codes, pagination
│   └── CAP.md                    # Consistency vs. Availability decisions
│
├── domain/                       # Domain-specific knowledge base
│   ├── dictionary.md             # Authoritative A-Z glossary
│   ├── rules.md                  # Business and regulatory rules
│   └── [domain-specific].md     # e.g., scoring-model.md, collections.md
│
├── tasks/
│   ├── phase-1-foundation.md     # Detailed tasks + Definition of Done
│   ├── phase-2-*.md
│   └── phase-N-*.md
│
└── user_stories/
    ├── US-01_*.md
    └── US-NN_*.md
```

## Phase Task File Template

Every `tasks/phase-N-*.md` file must follow this template:

```markdown
# Phase N: [Phase Name]

**Goal:** [One sentence describing the outcome of this phase.]

---

## 1. [Task Group Name]

- [Specific, actionable task. Includes the exact endpoint or file name.]
- [Another specific task.]

## 2. [Next Task Group]

- [Task with enough detail to implement without asking questions.]

---

## Definition of Done

- [ ] [A testable, binary condition. Either it passes or it doesn't.]
- [ ] [Another specific DoD condition.]
```

## Entity File Template

Every table in `technical/entities.md` must follow this format:

```markdown
### `table_name`
[One sentence explaining what this table represents.]

- `id` (BIGINT, PK) — Auto-increment. Never exposed externally.
- `public_id` (UUID v7, Unique) — Used in all API responses and URL parameters.
- [Other fields with type, constraints, and a brief description.]
- `created_at` / `updated_at`
```

## Non-Negotiable Rules

1. **No table without Dual-ID**: Every table must have `id` (BIGINT) AND `public_id` (UUID v7). No exceptions unless explicitly approved.
2. **No table without timestamps**: Every table must have `created_at`. Most must also have `updated_at`. Append-only tables (like `audit_logs`) have only `created_at`.
3. **Non-technical language in BRD/MVP**: These files are for stakeholders, not engineers. No table names, no SQL, no code.
4. **Technical precision in TRD/tasks**: These files are for engineers. Vague descriptions like "implement auth" are not acceptable.
