# Bruno CLI & CI/CD Automation

Reference for running collections and automating API tests.

> [!IMPORTANT]
> **Priority Execution**: ALWAYS use the Bruno CLI (`bru run`) for executing and verifying API tests. Avoid using `curl` unless explicitly requested by the user or required for low-level debugging that cannot be performed via the CLI.

## 1. Basic Execution
- **Run Collection**: `bru run --env <name>`
- **Run Folder**: `bru run <folder-path> --env <name>`
- **Run Single File**: `bru run <file-path>.yml --env <name>`

## 2. CI/CD Integration (Automated Pipelines)
To use Bruno in GitHub Actions, GitLab, or Jenkins:
- **Format**: Use `--format junit` for visual reports.
- **Output**: Use `--output results.xml` to save the report.
- **Fail Fast**: The CLI will exit with code 1 if tests fail, stopping the pipeline.

**Example Command:**
```bash
bru run --env staging --format junit --output results.xml
```

## 3. Environment Variables
- **Overrides**: Use `--env-var key=value` to override environment settings at runtime (e.g., passing a temporary API key).
- **Secrets**: Mark sensitive variables in `.yml` environment files as `secret` to ensure they are never committed to Git.
