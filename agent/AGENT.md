## Last updated: 20 April 2026
# AGENT.md - Agent Identity and Session Behavior

You are an AI assistant running inside **context-glue**.

Return to `context-glue/START_HERE.md` and complete the full instruction loading order before continuing below.

---

## Capability self-check

Before starting any session, assess what you are able to do in this environment. Work through each capability and state the result openly.

| Capability | How to check | If missing |
|---|---|---|
| **Read files** | Can you read files from disk? | Almost all context-glue workflows require this. Warn the user. |
| **Write files** | Can you write and edit files? | Ticket tracking (CONTEXT.md, CHECKLIST.md, etc.) will not work. Warn the user. |
| **Run terminal commands** | Can you execute shell/CLI commands? | Git sync and any command-based queries will not work. Note which workflows are affected. |
| **Credentials available** | Does `env.local` exist and contain values? | Tell the user which tools will be unavailable and how to fix it. |

Report the result in a single block before proceeding:

```
## Capability check
- [x] File read        - full access
- [x] File write       - full access
- [x] Terminal         - available
- [x] env.local        - loaded (Snowflake CLI, AWS)
```

Or with gaps:

```
## Capability check
- [x] File read        - full access
- [x] File write       - full access
- [ ] Terminal         - not available in this environment
                         → git sync will be skipped; command-based queries will not run
- [ ] env.local        - missing; copy env.template to env.local and fill in credentials
```

If a missing capability blocks the user's intended workflow, say so clearly and offer alternatives where possible.

---

## Session start protocol

1. **Confirm settings** - state active toggles in one line (see START_HERE.md "Applying settings").
2. **Confirm ticket context** - if a ticket ID was provided:
   - State the ticket key, date of last CONTEXT.md update, and how many checklist items remain.
   - Call out any items marked as "user action required" that are still pending.
3. **If no ticket folder exists** - say: "No context found for {TICKET}. Should we start a new session? Read `context-glue/prompts/new.md` to begin."
4. **Ready message** - one concise line confirming you are in the loop.

Example:
> Loaded TICKET-42 | Last updated 2026-04-14 | 3 items remaining | 1 user action pending

---

## During a session

- **Auto-update checklist** (if `workflow.auto_update_checklist: true`): mark `[x]` on completion without waiting to be asked.
- **Auto-update context** (if `workflow.auto_update_context: true`): append decisions and findings to CONTEXT.md as work progresses.
- **Auto-update PR** (if `workflow.auto_update_pullrequest: true`): keep PULLREQUEST.md current as changes accumulate.
- **Investigations**: follow `context-glue/knowledge/investigate.md` strictly. Never exceed `investigation.max_actions_per_session` without asking first.
- **Git**: never run git commands unless `git.enable_commits: true` in settings.json. The user handles all git. Exception: git sync is handled separately via `procedures/GIT_SYNC.md` at session start.

---

## Session end protocol

When the user signals the session is ending:

1. Ensure CHECKLIST.md, CONTEXT.md, and PULLREQUEST.md are current.
2. **Write session summary** (if `workflow.summarize_on_session_end: true`):
   - What was completed this session
   - What remains open
   - Any outstanding user actions
   - Any risks or open questions
3. **Git commit** (if `git.enable_commits: true` AND `git.commit_strategy: "session_end"`): stage and commit.

---

## Behavior rules

- Concise by default. Don't explain basics unless asked.
- State assumptions explicitly.
- One clarifying question at a time.
- Never silently skip an instruction. If you can't follow a rule, say why.
- Never use git unless settings allow it.
- **Load Receipt is mandatory.** Produce the structured load receipt exactly as specified in `START_HERE.md`. A narrative sentence does not satisfy it.
