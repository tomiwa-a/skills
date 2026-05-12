# Plan Structure: The Canonical plan/ Directory

This file defines the authoritative structure of a `plan/` directory. Any project that follows the Tomiwa engineering standard must conform to this layout.

## The Full Directory Tree

```
plan/
├── BRD.md                        # Business Requirements Document
├── MVP.md                        # Scope, MoSCoW prioritization, Success Definition
├── PHASES.md                     # High-level phase roadmap with goals
├── TRD.md                        # Technical Requirements Document (Stack, Architecture)
├── VIABILITY.md                  # Market viability analysis (optional but recommended)
│
├── technical/
│   ├── entities.md               # Full database schema with Dual-ID pattern
│   ├── req-res.md                # HTTP request/response envelope, error codes, pagination
│   └── CAP.md                    # Consistency vs Availability trade-off decisions
│
├── domain/                       # Domain knowledge (industry-specific)
│   ├── dictionary.md             # Authoritative A-Z glossary of domain terms
│   ├── rules.md                  # Business/regulatory rules
│   ├── scoring-model.md          # If applicable: scoring logic and weights
│   └── [other-domain-files].md
│
├── tasks/
│   ├── phase-1-foundation.md     # Phase 1 detailed tasks + Definition of Done
│   ├── phase-2-*.md              # Each phase gets its own file
│   └── phase-N-*.md
│
└── user_stories/
    ├── US-01_*.md                # User stories in MoSCoW priority order
    └── US-NN_*.md
```

## What Each File Must Contain

### `BRD.md`
- The problem being solved (non-technical)
- The target user / market
- The competitive landscape
- The business model

### `MVP.md`
- MoSCoW prioritization (Must / Should / Could / Won't)
- The 7-phase milestone breakdown
- A concrete, measurable Success Definition

### `TRD.md`
- System architecture (e.g., Hexagonal, Event-Driven)
- Folder structure with explanation
- Technology stack table (Component | Technology | Reasoning)
- Non-functional requirements (SLAs, security, performance)

### `technical/entities.md`
- Every database table, organized by layer (Auth, Profile, Domain, Operations, Rights, Audit)
- Must apply the Dual-ID pattern: `id` (BIGINT PK) + `public_id` (UUID v7)
- Must include `created_at` / `updated_at` on all tables
- Must include a Summary of Relationships section

### `technical/req-res.md`
- Standard response envelope
- Paginated response structure
- Error model (single error + details array for validation)
- Standard HTTP status codes in use
- All error code constants (e.g., `CONSUMER_BUREAU_FROZEN`)

### `tasks/phase-N-*.md`
- Numbered task list with enough detail to implement without ambiguity
- A "Definition of Done" checklist at the bottom
- No vague tasks like "build the auth module." Specific tasks like "Implement `POST /v1/auth/register` with Argon2 password hashing."
