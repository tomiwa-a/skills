# Contract Analysis: Extracting Test Cases from Plan Files

This guide explains how to read `plan/technical/req-res.md` and `plan/technical/entities.md` to generate a comprehensive set of Bruno test cases for any project.

## Step 1: Read the Standard Envelope
From `plan/technical/req-res.md`, extract the base response structure. Every test must assert this shape:

```javascript
tests {
  test("Response has standard envelope", () => {
    expect(res.body).to.have.property("success");
    expect(res.body).to.have.property("data");
    expect(res.body).to.have.property("error");
    expect(res.body).to.have.property("request_id");
    expect(res.body).to.have.property("timestamp");
  });
}
```

## Step 2: Map Endpoints to Test Suites
For each endpoint listed in the plan, generate these test cases:

| Test Type | Trigger | Expected Status | Key Assertions |
|---|---|---|---|
| Happy Path | Valid request body + valid auth | 200/201/202 | `success: true`, `data` is populated |
| Missing Required Field | Omit one required field from body | 400 | `error.code = "VALIDATION_ERROR"`, `error.details` is array |
| Multiple Validation Errors | Omit multiple required fields | 400 | `error.details.length > 1` |
| Wrong Type | Pass wrong data type for a field | 400 | `error.details` contains the field name |
| Missing Auth | No Authorization header | 401 | `error.code = "UNAUTHORIZED"` |
| Invalid Token | Malformed or expired token | 401 | `error.code = "UNAUTHORIZED"` |
| Wrong Role / Ownership | Valid token, accessing another user's resource | 403/404 | The specific error code from req-res.md |
| Duplicate Resource | Submit same idempotency key twice | 409 | `error.code = "CONFLICT"` |
| Business Logic Error | Valid format, but violates a business rule | 422 | The specific error code from `plan/technical/req-res.md` |
| Not Found | Query for non-existent ID | 404 | `error.code = "NOT_FOUND"` |

## Step 3: Extract Entity-Based Test Data
From `plan/technical/entities.md`, extract:
- **Required fields**: Every field without "(Nullable)" in the schema is required. Omitting it must trigger a 400 validation error.
- **Enum values**: Test that submitting an invalid enum value triggers a 400 error.
- **Unique constraints**: Test that submitting a duplicate value for a unique field triggers a 409.
- **FK constraints**: Test that referencing a non-existent parent ID triggers an appropriate error.

## Step 4: Extract Error Codes
From the error codes table in `plan/technical/req-res.md`, generate one negative test per error code. Every defined error code must have a test that can trigger it.

## Step 5: Folder Structure for the Collection

Organize the Bruno collection to mirror the project's API resource structure:

```
collection/
├── environments/
│   ├── local.bru
│   └── staging.bru
├── [resource-1]/
│   ├── folder.yml
│   ├── [happy-path].bru
│   ├── [validation-error].bru
│   └── [auth-error].bru
├── [resource-2]/
│   ├── folder.yml
│   └── ...
└── [async-resource]/
    ├── folder.yml
    └── [202-accepted].bru
```

The folder names must match the API resource groups defined in the project's `plan/tasks/` files.
