# Async Testing Patterns (OpenCollection YAML)

Use these patterns for 202-accepted background processing flows.

## Async Trigger

```yaml
info:
  name: [Action] — Async Trigger (202 Accepted)
  type: http
  seq: 1

http:
  method: POST
  url: "{{baseUrl}}/v1/[resource]/[action]"
  auth: inherit
  body:
    type: json
    data: |-
      {
        "[field]": "{{[value]}}"
      }

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 202
    - expression: res.body.data.status
      operator: isString
    - expression: res.body.data.request_id
      operator: isString
  scripts:
    post-response: |-
      if (res.status === 202) {
        bru.setVar("async_request_id", res.body.data.request_id);
      }

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5
```

## Idempotency Check

```yaml
info:
  name: [Action] — Idempotency Check
  type: http
  seq: 2

http:
  method: POST
  url: "{{baseUrl}}/v1/[resource]/[action]"
  headers:
    - name: X-Idempotency-Key
      value: "{{prev_key}}"
  auth: inherit
  body:
    type: json
    data: |-
      {
        "[field]": "{{[value]}}"
      }

runtime:
  assertions:
    - expression: res.status
      operator: in
      value: "200,202"
    - expression: res.body.data.request_id
      operator: eq
      value: "{{async_request_id}}"

settings:
  encodeUrl: true
  timeout: 0
  followRedirects: true
  maxRedirects: 5
```
