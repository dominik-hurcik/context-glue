## Last updated: 20 April 2026
# investigate.md — Investigation Protocol

## Purpose

Defines the exact steps to follow when querying live systems or running commands during an investigation. Applies to any tool: databases, APIs, CLIs, logs, dashboards, or any other system.

---

## When to investigate

Investigate when:
- You need to verify actual system state before writing or changing something
- You suspect an issue and need evidence (counts, values, logs, API responses, config values, etc.)
- A result is unexpected and you need to inspect both sides of the problem
- You need to confirm that a value, record, resource, or service state exists in a live system

Do NOT investigate when:
- You can answer the question from code, configuration, or documentation alone
- The user has already provided the data in the conversation
- The investigation would require mutating state

---

## Protocol

### Step 1 - Define the question

State the single specific question you need to answer before running anything.

Examples vary by stack type:

| Stack type | Example questions |
|---|---|
| Data / SQL | "How many rows in `orders` have a null `customer_id`?" |
| Backend / API | "Does the `/payments` endpoint return a 200 for this request payload?" |
| Platform / Infra | "How many pods in the `payments` namespace are currently not ready?" |
| Frontend | "Does the production build include the updated `featureFlags.js` bundle?" |
| Config / Feature flags | "What is the current value of `feature_flag_x` in production?" |
| CI/CD | "Which pipeline runs failed in the last 24 hours and at which step?" |
| Logs | "Are there any ERROR-level entries for `payment-service` in the last hour?" |
| CRM / SaaS | "How many open deals in this pipeline have no close date set?" |

Pick the form that fits your system. The protocol below applies to all of them.

### Step 2 - Write the query or command first

Write and review before executing. Check:
- Read-only - no mutations
- Scoped to the minimum needed
- If SQL: has a LIMIT clause
- If an API call: uses a read-only endpoint
- If a CLI command: is non-destructive
- If reading logs or files: scoped to a time range or line count

### Step 3 - Run it

Use the appropriate tool for the current system. Connection details come from `env.local`.

Announce what you are running and why before executing.

### Step 4 - Interpret and surface results

- State clearly what the result means in context
- If the result is unexpected, flag it before proceeding
- Show exactly what was run and what was returned

### Step 5 - Confirm before acting

If the result implies a change is needed, stop and confirm with the user before making it.

---

## Hard limits

- Maximum actions per session: see `investigation.max_actions_per_session` in `settings.json`. Ask the user if more are needed.
- Never run mutations without explicit user confirmation
- Always cap results when reading from large datasets, APIs, or logs unless told otherwise (SQL LIMIT, API page size, log line count, etc.)
- Record each step in PROGRESS.md as it happens (for ad-hoc analyses) or in CONTEXT.md (for ticket sessions)
