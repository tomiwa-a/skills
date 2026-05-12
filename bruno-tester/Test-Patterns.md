# Test Patterns: Standard .bru File Templates

Use these templates for each test type. Replace bracketed values with project-specific content from the project's `plan/technical/req-res.md` and `plan/technical/entities.md`.

## 1. Happy Path (POST — Create Resource)

```bru
meta {
  name: Create [Resource] (Happy Path)
  type: http
  seq: 1
}

post {
  url: {{base_url}}/v1/[resource]
  body: json
  auth: none
}

body:json {
  {
    "[field_1]": "[valid_value]",
    "[field_2]": "[valid_value]"
  }
}

tests {
  test("Status is 201 Created", () => {
    expect(res.status).to.equal(201);
  });
  test("Success is true", () => {
    expect(res.body.success).to.be.true;
  });
  test("Data contains resource public_id", () => {
    expect(res.body.data).to.have.property("id");
  });
  test("Error is null", () => {
    expect(res.body.error).to.be.null;
  });
}

script:post-response {
  if (res.status === 201) {
    bru.setVar("resource_id", res.body.data.id);
  }
}
```

## 2. Validation Error (Multiple Fields Missing)

```bru
meta {
  name: Create [Resource] (Missing Fields — 400)
  type: http
  seq: 2
}

post {
  url: {{base_url}}/v1/[resource]
  body: json
  auth: none
}

body:json {
  {
    "[invalid_or_missing_field]": "[bad_value]"
  }
}

tests {
  test("Status is 400 Bad Request", () => {
    expect(res.status).to.equal(400);
  });
  test("Success is false", () => {
    expect(res.body.success).to.be.false;
  });
  test("Error code is VALIDATION_ERROR", () => {
    expect(res.body.error.code).to.equal("VALIDATION_ERROR");
  });
  test("Details array has multiple errors", () => {
    expect(res.body.error.details).to.be.an("array");
    expect(res.body.error.details.length).to.be.greaterThan(1);
  });
}
```

## 3. Auth Failure (Missing or Invalid Token)

```bru
meta {
  name: Get [Resource] (No Auth — 401)
  type: http
  seq: 1
}

get {
  url: {{base_url}}/v1/[resource]
  auth: none
}

tests {
  test("Status is 401 Unauthorized", () => {
    expect(res.status).to.equal(401);
  });
  test("Error code is UNAUTHORIZED", () => {
    expect(res.body.error.code).to.equal("UNAUTHORIZED");
  });
}
```

## 4. Business Logic Error (422)

Replace `[BUSINESS_ERROR_CODE]` and `[trigger_scenario]` with the specific error code from `plan/technical/req-res.md`.

```bru
meta {
  name: [Action] — [Error Scenario] (422)
  type: http
  seq: 3
}

post {
  url: {{base_url}}/v1/[resource]
  body: json
  auth: bearer
}

auth:bearer {
  token: {{auth_token}}
}

body:json {
  {
    "[field]": "{{[trigger_value]}}"
  }
}

tests {
  test("Status is 422 Unprocessable Entity", () => {
    expect(res.status).to.equal(422);
  });
  test("Error code matches expected business rule", () => {
    expect(res.body.error.code).to.equal("[BUSINESS_ERROR_CODE]");
  });
  test("Details array is empty for business errors", () => {
    expect(res.body.error.details).to.be.an("array").that.is.empty;
  });
}
```

## 5. Token Capture Pattern (Login → Use Token)

```bru
# In your login/auth request — post-response script:
script:post-response {
  if (res.status === 200) {
    bru.setVar("auth_token", res.body.data.token);
    bru.setVar("user_id", res.body.data.id);
  }
}

# In subsequent protected requests:
auth:bearer {
  token: {{auth_token}}
}
```

## 6. Async 202 Accepted Pattern

Use this when the endpoint queues a job rather than returning a result immediately. Refer to `plan/technical/req-res.md` for the specific async flow.

```bru
meta {
  name: [Action] — Async Trigger (202 Accepted)
  type: http
  seq: 1
}

post {
  url: {{base_url}}/v1/[resource]/[action]
  body: json
  auth: bearer
}

auth:bearer {
  token: {{auth_token}}
}

body:json {
  {
    "[field]": "{{[value]}}"
  }
}

tests {
  test("Status is 202 Accepted for async operation", () => {
    expect(res.status).to.equal(202);
  });
  test("Data contains status QUEUED or equivalent", () => {
    expect(res.body.data.status).to.be.a("string");
  });
  test("Data contains a request_id for tracking", () => {
    expect(res.body.data.request_id).to.be.a("string");
  });
}

script:post-response {
  if (res.status === 202) {
    bru.setVar("async_request_id", res.body.data.request_id);
  }
}
```

## 7. Paginated List Pattern

```bru
meta {
  name: List [Resources] (Paginated)
  type: http
  seq: 1
}

get {
  url: {{base_url}}/v1/[resource]?page=1&per_page=25
  auth: bearer
}

auth:bearer {
  token: {{auth_token}}
}

tests {
  test("Status is 200 OK", () => {
    expect(res.status).to.equal(200);
  });
  test("Data is an array", () => {
    expect(res.body.data).to.be.an("array");
  });
  test("Meta contains pagination fields", () => {
    expect(res.body.meta).to.have.property("page");
    expect(res.body.meta).to.have.property("per_page");
    expect(res.body.meta).to.have.property("total_items");
    expect(res.body.meta).to.have.property("has_next");
  });
}
```
