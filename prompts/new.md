## New ticket — context-glue

1. Read `context-glue/START_HERE.md` — follow the full instruction loading order.

2. Ask:
   > "What is the ticket or task ID? (e.g. JIRA-123, GH-42, LINEAR-7, or a short name if you don't use a tracker)"
   > "What is the task?"

3. Based on the task description, load the relevant knowledge rooms before doing any work.

4. Create `context-glue/tickets/{TICKET}/.agent/` with four living files:

   **CONTEXT.md** — the living record of this ticket. What was decided, why, and how pieces fit together. Start with a brief summary of the task and update continuously as work progresses.
   ```markdown
   # {TICKET} — Context

   ## Task
   {description from step 2}

   ## Decisions and findings
   <!-- Agent appends here as work progresses -->
   ```

   **CHECKLIST.md** — ordered task list. Break the work into concrete steps and check them off as you go.
   ```markdown
   # {TICKET} — Checklist

   ## Tasks
   - [ ] {first step}
   - [ ] {second step}
   <!-- Add more as the scope becomes clear -->
   ```

   **TESTING.md** — test cases, test plans, and results. Update as testing happens.
   ```markdown
   # {TICKET} — Testing

   ## Test cases
   <!-- Add test cases as they are identified -->

   ## Results
   <!-- Record outcomes as tests are run -->
   ```

   **PULLREQUEST.md** — PR or MR description. Keep it current so it's copy-paste ready when needed.
   ```markdown
   # {TICKET} — Pull Request

   ## Summary
   <!-- What this change does and why -->

   ## Changes
   <!-- List of files or components changed -->

   ## Testing done
   <!-- How it was tested -->
   ```

5. All four files are alive — update them as work progresses, not just at the end.

6. Never use git commands. The user handles all git operations.
