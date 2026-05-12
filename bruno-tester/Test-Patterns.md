# Test Patterns: Standard OpenCollection YAML Templates

Use these templates for each test type. Every file MUST be a `.yml` file.

## 1. Happy Path (POST — Create Resource)

```yaml
info:
  name: Create [Resource] (Happy Path)
  type: http
  seq: 1

http:
  method: POST
  url: "{{baseUrl}}/v1/[resource]"
  auth: inherit
  body:
    type: json
    data: |-
      {
        "[field_1]": "[valid_value]",
        "[field_2]": "[valid_value]"
      }

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 201
    - expression: res.body.success
      operator: eq
      value: true
    - expression: res.body.data.id
      operator: isString
  scripts:
    post-response: |-
      if (res.status === 201) {
        bru.setVar("resource_id", res.body.data.id);
      }

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5
```

## 2. Validation Error (400)

```yaml
info:
  name: Create [Resource] (Validation Error — 400)
  type: http
  seq: 2

http:
  method: POST
  url: "{{baseUrl}}/v1/[resource]"
  auth: inherit
  body:
    type: json
    data: |-
      {
        "[invalid_or_missing_field]": "[bad_value]"
      }

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 400
    - expression: res.body.error.code
      operator: eq
      value: VALIDATION_ERROR
    - expression: res.body.error.details
      operator: isArray

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5
```

## 3. Business Logic Error (422)

```yaml
info:
  name: [Action] — [Error Scenario] (422)
  type: http
  seq: 3

http:
  method: POST
  url: "{{baseUrl}}/v1/[resource]"
  auth: inherit
  body:
    type: json
    data: |-
      {
        "[field]": "{{[trigger_value]}}"
      }

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 422
    - expression: res.body.error.code
      operator: eq
      value: "[BUSINESS_ERROR_CODE]"

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5
```

## 4. Paginated List Pattern

```yaml
info:
  name: List [Resources] (Paginated)
  type: http
  seq: 1

http:
  method: GET
  url: "{{baseUrl}}/v1/[resource]?page=1&per_page=25"
  auth: inherit

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
    - expression: res.body.data
      operator: isArray
    - expression: res.body.meta.page
      operator: isDefined

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5
```
