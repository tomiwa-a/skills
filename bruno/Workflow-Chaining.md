# Bruno Workflow Chaining & Scripting

Use these patterns to create multi-step API tests that share data.

## 1. Capturing Data (Post-Response)
Add these scripts to "Provider" requests to store variables for later use:

```javascript
script:post-response {
  if (res.status === 200 || res.status === 201) {
    // Auth Tokens
    if (res.body.token) {
      bru.setVar("authToken", res.body.token);
    }
    
    // Resource IDs (Look for 'id', '_id', or 'uuid')
    const id = res.body.id || res.body.data?.id || res.body.uuid;
    if (id) {
      bru.setVar("activeResourceId", id);
    }
  }
}
```

## 2. Using Data (Chaining)
Update "Consumer" requests to use the variables stored in the environment:

- **Authentication**: Set the `Authorization` header to `Bearer {{authToken}}`.
- **Dynamic Paths**: Use `{{baseUrl}}/resource/{{activeResourceId}}`.
- **JSON Bodies**: Pass IDs from previous steps:
  ```json
  {
    "userId": "{{activeResourceId}}",
    "updatedAt": "now"
  }
  ```

## 3. Workflow Validation
- Always add a basic test to verify the chain is working:
  ```javascript
  tests {
    test("Variable was captured", () => {
      expect(bru.getVar("authToken")).to.be.a("string");
    });
  }
  ```
