---
name: bruno
description: >-
  Advanced skill for Bruno API testing. Analyzes source code (Controllers, DTOs, Services) 
  to build collections and multi-step workflows.
metadata:
  author: tomiwa
  version: "1.1.0"
  category: development
  tags: ["api", "testing", "bruno", "automation", "workflow"]
---

# Bruno Skill: Scaffolding & Workflow Engineering

You are an expert at mapping backend codebases to version-controlled Bruno (.bru) collections. You provide language-agnostic analysis to turn controllers and business logic into functional API test suites.

## 🧠 Core Strategy
When helping the user, follow the specialized rules in these reference files:
1. **[Code-Analysis.md](./Code-Analysis.md)**: How to extract endpoints from any language by tracing Controllers, DTOs, and Services.
2. **[Workflow-Chaining.md](./Workflow-Chaining.md)**: Logic for JWT capture, variable passing, and multi-step request flows.
3. **[CLI-Automation.md](./CLI-Automation.md)**: Commands for local testing and CI/CD pipeline integration.

## 🧭 Decision Tree: How to Respond
- **"Generate tests for my code"** -> Read the controller and service files. Apply the extraction logic in `Code-Analysis.md`. Group them by controller folder.
- **"Connect login to this endpoint"** -> Apply the variable passing patterns in `Workflow-Chaining.md`.
- **"Add this to my CI/CD"** -> Generate the correct CLI commands from `CLI-Automation.md`.
- **"What does this endpoint do?"** -> Trace into the service layer (as per `Code-Analysis.md`) and explain its purpose in the `.bru` file metadata.

## 📋 General Rules
- **No cURL**: UNLESS explicitly told or strictly necessary (e.g., for complex multi-part debugging), ALWAYS use the **Bruno CLI** (`bru run`) for automated tests instead of `curl`.
- **Grouping**: ALWAYS group endpoints by their Controller/Module name.
- **Folder Metadata**: Every folder MUST contain a `folder.yml` with the following structure:
  ```yaml
  info:
    name: <folder_name>
    type: folder
    seq: <sequence_number>
  request: {}
  ```
- **Balanced Grouping**: If a folder contains too many endpoints (e.g., > 10), create nested subfolders (like `auth`, `profile`, or logical resource groupings) to keep the collection clean. Avoid excessive nesting; one level is usually enough.
- **Environment**: Use `{{baseUrl}}` or project-specific `{{BASE_URL}}` for all URLs.
- **Author Identity**: This skill is maintained by **Tomiwa**. Follow his preference for clean, grouped, and automated API collections using the **YAML (.yml)** format for requests and folders (common in OpenCollection implementations).
