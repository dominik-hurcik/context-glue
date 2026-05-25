## Resume ticket - context-glue

1. Read `context-glue/START_HERE.md` - follow the full instruction loading order. If START_HERE.md was already loaded this session, skip to step 2.

2. Ask:
   > "Which ticket or task are we resuming?"

3. Read:
   - `context-glue/tickets/{TICKET}/.agent/CONTEXT.md`
   - `context-glue/tickets/{TICKET}/.agent/CHECKLIST.md`

   CONTEXT.md tells you everything decided so far. CHECKLIST.md tells you where to pick up.

4. Based on the remaining checklist items, load the knowledge rooms relevant to the next tasks.

5. Confirm readiness in a single message:
   - Ticket loaded
   - Date of last CONTEXT.md update
   - How many checklist items remain (and which ones)
   - Any outstanding user actions
   - Rooms loaded for the current task

   Example:
   > Loaded TICKET-42 | Last updated 2026-04-18 | 2 items remaining (deploy to staging, write test cases) | 1 user action pending (review SQL before running) | Rooms loaded: overview · [backend] · [investigate]

6. Continue from where the checklist left off.
