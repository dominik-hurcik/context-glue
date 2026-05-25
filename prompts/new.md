## New ticket - context-glue

1. Read `context-glue/START_HERE.md` - follow the full instruction loading order. If START_HERE.md was already loaded this session, skip to step 2.

2. Read `context-glue/settings.json`. Check `workspace.ticket_id_example` and `workspace.ticket_tracker`.

   Ask:
   > "What is the ticket or task ID?"

   If `ticket_id_example` is set in settings, append: "(e.g. {ticket_id_example})"
   If `ticket_tracker` is "none" or empty, append: "(or a short name if you don't use a tracker)"

   Then ask on the next line:
   > "What is the task?"

3. Based on the task description, load the relevant knowledge rooms before doing any work.

4. Create `context-glue/tickets/{TICKET}/.agent/` with five living files:

   **CONTEXT.md** - the living record of this ticket. What was decided, why, and how pieces fit together. Start with a brief summary of the task and update continuously as work progresses.
   ```markdown
   # {TICKET} - Context

   ## Task
   {description from step 2}

   ## Decisions and findings
   <!-- Agent appends here as work progresses -->
   ```

   **CHECKLIST.md** - ordered task list. Break the work into concrete steps and check them off as you go.
   ```markdown
   # {TICKET} - Checklist

   ## Tasks
   - [ ] {first step}
   - [ ] {second step}
   <!-- Add more as the scope becomes clear -->
   ```

   **TESTING.md** - test cases, test plans, and results. Update as testing happens.
   ```markdown
   # {TICKET} - Testing

   ## Test cases
   <!-- Add test cases as they are identified -->

   ## Results
   <!-- Record outcomes as tests are run -->
   ```

   **PULLREQUEST.md** - PR or MR description. Keep it current so it's copy-paste ready when needed.
   ```markdown
   # {TICKET} - Pull Request

   ## Summary
   <!-- What this change does and why -->

   ## Changes
   <!-- List of files or components changed -->

   ## Testing done
   <!-- How it was tested -->
   ```

   **PROMOTE_CANDIDATES.md** - real-time log of findings worth promoting to shared knowledge. The agent appends to this file the moment a non-obvious discovery is made - do not wait until the end of the session.
   ```markdown
   # {TICKET} - Promote Candidates

   <!-- Agent appends one line per non-obvious discovery during the session. -->
   <!-- Format: - [target room tag] Brief statement of the finding. -->
   <!-- Example: - [backend] Retrying a failed job without clearing its lock causes silent duplicate processing. -->
   ```

5. All five files are alive - update them as work progresses, not just at the end.

6. **PROMOTE_CANDIDATES.md must be updated in real time.** Whenever a non-obvious discovery is made - a gotcha, a confirmed behavior, a corrected assumption, a useful pattern - append a one-line entry immediately. Format: `- [target room tag] Brief statement of the finding.` Do not batch these to the end of the session.

7. Never use git commands. The user handles all git operations.
