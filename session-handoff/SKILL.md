---
name: session-handoff
description: >-
  Context preservation and session closing skill. Captures exact project state at the
  end of a work session. Updates task checklists, writes a handoff.md with the precise
  next action, and prevents context amnesia when resuming work in a new chat or handing
  off to another engineer.
metadata:
  author: tomiwa
  version: "1.0.0"
  category: workflow
  tags: ["handoff", "context", "session", "continuity", "tasks", "documentation"]
---

# Session Handoff Skill: The Context Manager

You are responsible for perfectly capturing the state of the project at the end of a work session. Your output prevents the "blank slate" problem that happens when a new AI session starts with no memory of what was just done.

## 🧠 Core Strategy
When helping the user, follow the specialized rules in these reference files:
1. **[Handoff-Template.md](./Handoff-Template.md)**: The exact structure of a `handoff.md` file and what each section must contain.
2. **[Task-Tracking.md](./Task-Tracking.md)**: How to identify completed tasks and update the `plan/tasks/` checklists accordingly.

## 🧭 Decision Tree: How to Respond
- **"End session" / "We're done for today"** → Trigger the full handoff flow: analyze recent changes, update task files, generate `handoff.md`.
- **"What did we do today?"** → Summarize the session's work from git diffs or changed files. Map it to phase tasks.
- **"Pick up where we left off"** → Read `handoff.md` from the project root. Immediately state: current phase, last completed task, and the exact next action.
- **"Check off these tasks"** → Find the corresponding items in `plan/tasks/*.md` and update the checkboxes.
- **"Write a handoff for the team"** → Generate a `handoff.md` that is detailed enough for a developer who has never seen this project to understand what to do next.

## 📋 General Rules
- **Be Surgical About "Next Action"**: The "Next Immediate Action" section must name the specific file to open and the specific function/endpoint to write. Never say "continue the implementation."
- **Update Checkboxes**: If a task was completed this session, find the `- [ ]` in the task file and update it to `- [x]`.
- **Preserve Blockers**: If there are unresolved questions, bugs, or pending architectural decisions, they must appear in the `handoff.md` Blockers section so they are not forgotten.
- **Session Context in Filename**: Name the handoff file with a date: `handoff-2025-05-12.md` so multiple sessions are traceable.
- **Author Identity**: This skill is maintained by **Tomiwa**. Follow his preference for precise, actionable handoff documentation.
