# Questioning Patterns: How to Pressure-Test a Design

Use these question patterns to force the user to reveal gaps in their thinking. Never accept "it should be fine" as an answer.

## Pattern 1: The Failure Mode Question
Force the user to think about what happens when things go wrong.

> "You've designed the happy path. Now tell me: what happens when `<external service>` goes down at the exact moment a user is trying to connect their bank account? Walk me through every step of the failure."

## Pattern 2: The Scale Question
Force the user to quantify their assumptions.

> "This works fine with 10 users. What happens at 10,000 concurrent requests? Where is the first bottleneck? What breaks first?"

## Pattern 3: The Replay Attack Question
Force the user to think about security at the boundary.

> "An attacker intercepts a valid signed webhook from Mono. They replay it 10 minutes later with the same valid HMAC signature. Does your system process it again? What's your defense?"

## Pattern 4: The Data Corruption Question
Force the user to think about atomic writes.

> "You write to `credit_agreements` and then write to `audit_logs`. Between those two writes, the server crashes. What is the state of the database? Is it corrupt? How do you recover?"

## Pattern 5: The "Who Has Access?" Question
Force the user to think about authorization at every layer.

> "Lender A generates a report for User X. Lender B knows the `public_id` of that report. Can Lender B call `GET /v1/reports/{id}` and see Lender A's report? Walk me through the authorization check."

## Pattern 6: The Stale Data Question
Force the user to think about cache coherence.

> "A Consumer's Shard Score is cached for 30 days. On Day 2, they are reported as `NON_PERFORMING` on a new loan. Does the cached score get invalidated? How does the next lender who queries this Consumer get a fresh, accurate score?"

## Pattern 7: The Dispute Deadline Question
Force the user to think about time-sensitive state machines.

> "A dispute is created with a `deadline_at` of 30 days. The lender never responds. On Day 31, what happens? Is there a cron job? What triggers the status transition from `OPEN` to `RESOLVED`? Who does it? When?"

## After Every Answer: Validate or Probe Deeper

- If the answer is specific and backed by code/logic: `"Accepted. Moving to the next risk."`
- If the answer is vague: `"That's not specific enough. Tell me exactly which line of code handles this case."`
- If the user says "we'll handle that later": `"Accepted as a known risk. I'm creating a note in the architecture review. Let's flag this in `plan/TRD.md` as a known gap."`
