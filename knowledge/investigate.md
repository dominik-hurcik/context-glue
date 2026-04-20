## Last updated: 20 April 2026
# investigate.md — Investigation Protocol

## Purpose

Defines the exact steps to follow when querying live systems or running commands during an investigation. Applies to any tool: databases, APIs, CLIs, logs, dashboards, or any other system.

---

## When to investigate

Investigate when:
- You need to verify actual data or system state before writing or changing something
- You suspect an issue and need evidence — counts, values, logs, API responses
- A result is unexpected and you need to inspect both sides of the problem
- You need to confirm that a value, record, or resource exists in a live system

Do NOT investigate when:
- You can answer the question from code, configuration, or documentation alone
- The user has already provided the data in the conversation
- The investigation would require mutating state

---

## Protocol

### Step 1 — Define the question

State the single specific question you need to answer before running anything.

Examples:
- "How many orders in the database have a null customer_id?"
- "Does the `/payments` endpoint return a 200 for this request payload?"
- "What is the current value of the `feature_flag_x` config in production?"
- "Which pipeline runs failed in the last 24 hours?"

### Step 2 — Write the query or command first

Write and review before executing. Check:
- Read-only — no mutations
- Scoped to the minimum needed
- If SQL: has a LIMIT clause and is fully lowercase
- If an API call: uses a read-only endpoint
- If a CLI command: is non-destructive

### Step 3 — Run it

Use the appropriate tool for the current system. Connection details come from `env.local`.

Announce what you are running and why before executing.

### Step 4 — Interpret and surface results

- State clearly what the result means in context
- If the result is unexpected, flag it before proceeding
- Show exactly what was run and what was returned

### Step 5 — Confirm before acting

If the result implies a change is needed, stop and confirm with the user before making it.

---

## Hard limits

- Maximum queries or commands per session: see `investigation.max_queries_per_session` in `settings.json`. Ask the user if more are needed.
- Never run mutations without explicit user confirmation
- Always add a result limit when querying large datasets unless told otherwise
- Record each step in PROGRESS.md as it happens (for ad-hoc analyses) or in CONTEXT.md (for ticket sessions)
