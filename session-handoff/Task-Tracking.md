# Task Tracking: How to Update plan/tasks/ Checklists

Use this guide to identify completed work and accurately update the task files at the end of a session. This process is language-agnostic and works for any tech stack.

## Step 1: Identify What Was Done

### Via Git (Preferred — Works for Any Language)
```bash
# List files changed in the last commit or since last session
git diff --name-only HEAD~1

# Full diff to see exactly what was added or changed
git diff HEAD~1

# List all commits made today
git log --since="today 00:00" --oneline
```

### Via File Modification Time (Fallback)
```bash
# Find all source files modified in the last 8 hours
# Adjust the extension or path pattern for the project's language
find . -newer plan/tasks/phase-1-foundation.md -type f \
  ! -path "./.git/*" \
  ! -path "./node_modules/*" \
  ! -path "./.next/*"
```

## Step 2: Map Changes to Tasks

For every changed file, trace it back to its corresponding task. Use the project's `plan/TRD.md` and folder structure to understand which layer a file belongs to. General mapping principles:

| Changed File Layer | Likely Task Location |
|---|---|
| HTTP handlers / controllers / routes | The phase that introduces that endpoint |
| Database migrations / schema files | Phase 1 Foundation (initial) or the phase that added the entity |
| Auth utilities / middleware | The phase covering authentication |
| Background workers / queue consumers | The phase covering async processing |
| Domain / business logic | The phase covering the core feature |
| Tests | The same phase as the feature being tested |
| CI/CD config | Phase 1 Foundation |

> Read the project's `plan/TRD.md` to understand the folder structure. The same principle applies whether the project uses Go, TypeScript, Python, Ruby, or any other language.

## Step 3: Update the Checkboxes

Find the matching `- [ ]` in the task file and change it to `- [x]`.

**Before:**
```markdown
- [ ] Implement the user registration endpoint accepting: name, email, password
```

**After:**
```markdown
- [x] Implement the user registration endpoint accepting: name, email, password
```

## Step 4: Handle Partial Completion

If a task was started but not finished, add a note inline. Do NOT mark it as `[x]`.

```markdown
- [ ] API Key Generation ← Prefix generation done. Key hashing NOT yet implemented.
```

## Step 5: Verify the Definition of Done

Before marking any task as `[x]`, check every item in the "Definition of Done" checklist at the bottom of the task file. If even one DoD item is unchecked, the task is NOT complete.
