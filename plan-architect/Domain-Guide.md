# Domain Guide: How to Build Domain Knowledge Files

The `plan/domain/` folder is what separates a "software project" from a "domain-first software project." It captures the rules, vocabulary, and logic of the industry being automated. Any engineer or AI agent reading these files should be able to understand the business without any prior knowledge.

## The Required Domain Files

Every project must have at minimum:

### 1. `dictionary.md` — The A-Z Glossary
A complete vocabulary list of every domain term, written as a reference that any stakeholder can read.

**Format:**
```markdown
**[Term]**
[2-3 sentence definition. Explain what it is, why it matters, and how Shard uses it.]
```

**Rules:**
- Every acronym must be spelled out on first use (e.g., BVN (Bank Verification Number)).
- Every table name or system term that non-technical stakeholders might encounter must be explained in plain language.
- Organize alphabetically by letter heading.

### 2. `rules.md` — Business & Regulatory Rules
Operational constraints that shape the system's behavior. Written as enforceable rules, not suggestions.

**Must Include:**
- Who is allowed to do what (permissions matrix).
- Time-based rules (e.g., "Disputes must be resolved within 30 days").
- Regulatory requirements (e.g., CBN mandates, data retention periods).
- Data accuracy obligations (e.g., who is liable for incorrect data).

### 3. Domain-Specific Logic Files

Create these additional files when the domain requires them:

| File | Use When |
|---|---|
| `scoring-model.md` | The system calculates a score or risk rating |
| `score-normalization.md` | Multiple scoring inputs must be blended or weighted |
| `collections.md` | The domain involves debt, payments, or lifecycle state machines |
| `interest-rates.md` | The domain involves lending, finance, or pricing |
| `state-machines.md` | Any entity has a defined lifecycle with specific transitions |
| `integrations.md` | The system relies on 3rd-party APIs with specific behaviors |

## How to Write a Scoring Model File

A `scoring-model.md` must answer these questions:
1. What are the inputs to the score?
2. What does each input measure in plain English?
3. What is the weight or relative importance of each input?
4. What is the output format (range, bands, labels)?
5. What are the edge cases (e.g., cold start, thin file, insufficient data)?
6. What signals can override the computed score regardless of the calculation?

## How to Write a State Machine File

When an entity transitions through states, document:
1. Every valid state (with a clear enum name).
2. Every valid transition (from state A to state B).
3. What triggers the transition (user action, time-based job, external event).
4. What is NOT a valid transition (i.e., states that cannot go backwards).

**Example Format:**
```
OPEN → UNDER_REVIEW (Triggered by: Shard admin picks up the dispute)
UNDER_REVIEW → RESOLVED (Triggered by: Subscriber submits a dispute_response)
UNDER_REVIEW → REJECTED (Triggered by: Admin closes with no action after deadline)
OPEN → RESOLVED: INVALID (A dispute cannot be resolved without review)
```

## Quality Standards for Domain Files

- **No technical jargon in dictionaries**: A `dictionary.md` entry must not reference table names, SQL types, or API endpoints.
- **Regulatory references must be dated**: When citing a regulation (e.g., CBN Credit Reporting Act, 2017), include the year.
- **Scoring weights must be justified**: Do not list weights without explaining the rationale (e.g., "Repayment history is weighted at 35% because it is the strongest single predictor of future default").
