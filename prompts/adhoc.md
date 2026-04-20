## Ad-hoc analysis — context-glue

1. Read `context-glue/START_HERE.md` — follow the full instruction loading order.

2. Ask:
   > "What do you need to investigate or explore?"

   Let the user describe the analysis in their own words.

3. Based on the description, determine which knowledge rooms are relevant and load them now. Examples:

   | If the user is... | Load these rooms |
   |---|---|
   | Investigating a data quality issue | `[investigate]` + relevant system rooms |
   | Debugging a pipeline or workflow | `[investigate]` + relevant system rooms |
   | Exploring an API or service | `[investigate]` + `[architecture]` if cross-system |
   | Analysing metrics or reports | `[investigate]` + relevant system rooms |
   | Tracing a value across systems | `[investigate]` + `[architecture]` + all relevant system rooms |

   State which rooms you loaded and why.

4. Ask:
   > "Give this analysis a short name to use as a folder name (e.g. `orders-null-customer-id`, `api-latency-spike`, `q1-conversion-drop`)."

5. Check if `context-glue/adhoc/{name}/` already exists.
   - If it exists: ask "An analysis with this name already exists. Resume it, or pick a different name?"
   - If resuming: read `.agent/PROGRESS.md` and `.agent/FINDINGS.md`, summarise where we left off, and continue from there.

6. If starting fresh, create:

   `context-glue/adhoc/{name}/.agent/PROGRESS.md`
   ```markdown
   # {name} — Analysis Progress

   ## Investigation question
   {user's description from step 2}

   ## Rooms loaded
   {list of rooms loaded in step 3}

   ## Steps
   <!-- Agent updates this as the investigation progresses -->
   ```

   `context-glue/adhoc/{name}/.agent/FINDINGS.md`
   ```markdown
   # {name} — Findings

   ## Summary
   <!-- Updated as findings emerge -->

   ## Key findings
   <!-- Numbered list of discoveries -->

   ## Open questions
   <!-- Anything unresolved -->
   ```

7. Work through the investigation. For each step:
   - **Before running a query or command**: state what you are about to check and why.
   - **After getting a result**: state what you found and what it means.
   - **Update PROGRESS.md** with each step as it happens — include the command or query, the result, and the conclusion.
   - **Update FINDINGS.md** when a discovery is worth recording.

8. Follow the investigation protocol in `context-glue/knowledge/investigate.md` for all queries and commands.

9. When the user signals they are done (or the question is answered):
   - Ensure PROGRESS.md and FINDINGS.md are current.
   - Write a short summary of what was found and any remaining open questions.
   - Do not promote findings to shared knowledge — ad-hoc analyses are personal and stay local.
