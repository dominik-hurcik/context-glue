## Complete ticket - context-glue

1. Read `context-glue/START_HERE.md` - follow the full instruction loading order. If START_HERE.md was already loaded this session, skip to step 2.

2. Ask:
   > "Which ticket are we completing?"

3. Read all agent files for that ticket:
   - `context-glue/tickets/{TICKET}/.agent/CONTEXT.md`
   - `context-glue/tickets/{TICKET}/.agent/CHECKLIST.md`
   - `context-glue/tickets/{TICKET}/.agent/TESTING.md`
   - `context-glue/tickets/{TICKET}/.agent/PROMOTE_CANDIDATES.md`

   (PULLREQUEST.md is kept current during the session but contains no promotable findings - skip it here.)

4. If any checklist items are still open, stop and flag them:
   > "{N} checklist items are still open. Complete them before closing, or confirm they are intentionally deferred."

   Do not proceed until the user confirms.

5. Read `PROMOTE_CANDIDATES.md`. This is the primary list of findings to promote - the agent built it in real time during the session. Present each candidate to the user for confirmation before writing:

   ```
   PROMOTE → context-glue/knowledge/stack/{system}.md
   Finding: {what was discovered}
   Will add to: Section "{target section}"

   Promote this? (yes / no / edit)
   ```

   Wait for confirmation before moving to the next. Do not batch writes.

6. After all PROMOTE_CANDIDATES entries are reviewed, do a lighter sanity-check scan of CONTEXT.md and TESTING.md for any findings not already captured in PROMOTE_CANDIDATES.md. Look for:

   | What to look for | Target file |
   |---|---|
   | System-specific patterns, conventions, gotchas | `context-glue/knowledge/stack/{relevant-system}.md` |
   | Cross-system contracts, integration failure modes, interface behavior | `context-glue/knowledge/ARCHITECTURE_MAP.md` |
   | Investigation techniques, query patterns that worked | `context-glue/knowledge/investigate.md` |

   Use `context-glue/knowledge/INDEX.md` to identify the right room for each finding. Present any additional findings using the same format as step 5.

7. Do NOT promote:
   - Decisions specific to this ticket only
   - Step-by-step debugging paths with no recurring pattern
   - One-off fixes that won't apply elsewhere
   - Anything already present in the target file

8. After all findings reviewed, write approved additions. Update `## Last updated:` on every modified file.

9. Run `context-glue/procedures/KNOWLEDGE_PR_CHECKLIST.md` - the "For the agent" section - against every file you modified. Report any flags (contradictions, duplication, wrong room, stale markers, size concern) before telling the user to push.

10. Update `## Last updated:` in `context-glue/knowledge/INDEX.md` if any knowledge files changed.

11. Confirm completion:
    - How many findings promoted and to which files
    - Any deferred checklist items
    - Any flags from the PR checklist
    - Ticket is closed

12. If any knowledge files were updated, remind the user:

    > Knowledge files updated. Push to share with the team:
    >
    > ```bash
    > cd context-glue
    > git add knowledge/
    > git commit -m "knowledge: {TICKET} findings"
    > git push
    > ```
    >
    > Then create a pull request. The reviewer should run `context-glue/procedures/KNOWLEDGE_PR_CHECKLIST.md` before approving.
