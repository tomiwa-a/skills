# Language-Agnostic API Extraction

When tasked with generating Bruno endpoints, follow this investigative pattern regardless of the programming language:

## 1. Locate the "Entry Point" (Controller/Router)
- Look for files containing API routes (e.g., `@Controller`, `@Route`, `app.get`, `router.post`).
- Identify the **Base Path** (class/module level) and the **Endpoint Path** (method level).

## 2. Analyze the "Contract" (Request & Response DTOs)
- Identify the data structure used for the request body and the expected response.
- **Mapping**: Convert the code properties (String, Int, Boolean, UUID) into a sample JSON object for the Bruno `body:json` block.
- Look for validation decorators to understand required vs. optional fields.

## 3. Understand the "Intent" (The Service Layer)
- **Deep Trace**: Look into the Service/Business Logic layer that the controller calls.
- **Behavior Mapping**: 
  - If the service calls a Database -> It's a CRUD operation.
  - If the service calls an External API (e.g., Paystack, AWS) -> Include necessary headers or external variables.
  - If the service handles Auth/Encryption -> Note this for Bruno's `script:pre-request`.

## 4. Output Goal
- Your goal is to understand *why* the endpoint exists so the generated Bruno request has a meaningful name and relevant test assertions.
