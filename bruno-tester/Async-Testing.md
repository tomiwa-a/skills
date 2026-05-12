# Async Testing: Testing the 202 Pattern and Webhooks

Many modern APIs are asynchronous — the client triggers an action and receives the result later via a webhook or a poll. This guide covers how to test async flows using Bruno, regardless of the domain.

## The Generic Async Flow

```
POST /[resource]/[action]
  └── 202 Accepted (job queued)
        └── [background worker/queue processes the job]
              └── POST to registered callback URL (result delivered)
```

## Strategy 1: Test Each Layer Independently

Do NOT try to test the full end-to-end async flow in a single Bruno test. The timing is unpredictable. Test each layer in isolation:

### Layer 1: The Trigger Endpoint
Test that the triggering endpoint correctly:
- Returns `202` when the job is successfully queued.
- Returns `200` immediately if the result is already cached/computed (cache hit).
- Returns the appropriate `422` business logic errors before queuing (e.g., if a precondition is not met).
- Returns `400` for invalid request payloads.

### Layer 2: The Inbound Webhook Receiver
If your system receives webhooks from a third party (e.g., a payment provider, an OAuth callback), test your receiver endpoint directly using Bruno:

```bru
meta {
  name: Simulate [Provider] Webhook (Valid Signature)
  type: http
  seq: 1
}

post {
  url: {{base_url}}/v1/webhooks/[provider]
  body: json
  auth: none
}

headers {
  [provider-signature-header]: {{test_hmac_signature}}
}

body:json {
  {
    "event": "[provider.event.name]",
    "data": {
      "id": "{{test_resource_id}}"
    }
  }
}

tests {
  test("Valid webhook returns 200", () => {
    expect(res.status).to.equal(200);
  });
}
```

```bru
meta {
  name: Simulate [Provider] Webhook (Invalid Signature — 401)
  type: http
  seq: 2
}

post {
  url: {{base_url}}/v1/webhooks/[provider]
  body: json
  auth: none
}

headers {
  [provider-signature-header]: invalid-signature-value
}

body:json {
  {
    "event": "[provider.event.name]"
  }
}

tests {
  test("Invalid signature returns 401", () => {
    expect(res.status).to.equal(401);
  });
  test("Error code is UNAUTHORIZED", () => {
    expect(res.body.error.code).to.equal("UNAUTHORIZED");
  });
}
```

### Layer 3: The Outbound Webhook Payload
For the webhooks your system sends to clients, use a test receiver service (e.g., `ngrok`, `hookdeck`, `webhook.site`) to capture the payload locally. Assert:
- The correct signature header is present (e.g., `X-[Service]-Signature`).
- The payload structure matches what `plan/technical/req-res.md` defines.
- All required fields in the payload are present.

## Strategy 2: Environment Variables for Async State

Carry state between an async trigger and a later verification step using Bruno variables:

```javascript
// In the trigger request's post-response script:
script:post-response {
  if (res.status === 202) {
    bru.setVar("async_request_id", res.body.data.request_id);
  }
}

// In a subsequent "fetch result" request (run manually after the worker completes):
// GET {{base_url}}/v1/[resource]?request_id={{async_request_id}}
```

## Strategy 3: Delayed Sequential Runs for Integration Tests

When running a full integration test locally where you need to wait for a worker to process:

```bash
# Add a delay between requests to allow the background job to complete
bru run [collection-folder]/ --env local --delay 5000
```

The `--delay` flag adds a pause (in milliseconds) between each request in the run. Adjust based on the expected processing time of the background worker.

## Idempotency Testing

For any async trigger that accepts an `X-Idempotency-Key` header, test the duplicate submission case:

```bru
meta {
  name: [Action] — Duplicate Idempotency Key (409)
  type: http
  seq: 4
}

post {
  url: {{base_url}}/v1/[resource]/[action]
  body: json
  auth: bearer
}

auth:bearer {
  token: {{auth_token}}
}

headers {
  X-Idempotency-Key: {{same_idempotency_key_as_previous_request}}
}

tests {
  test("Duplicate idempotency key returns 409", () => {
    expect(res.status).to.equal(409);
  });
  test("Error code is CONFLICT", () => {
    expect(res.body.error.code).to.equal("CONFLICT");
  });
}
```
