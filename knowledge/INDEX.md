## Last updated: (not set)
# Knowledge Palace - Index

> **Not configured yet.** Run `Read context-glue/prompts/setup.md` to generate this file for your stack.

This is the palace map. The agent reads it at session start, then loads only the rooms tagged for the current task.

---

## Always load (every session)

| Room | File | What it contains |
|---|---|---|
| Stack Overview | `context-glue/knowledge/stack/overview.md` | One-page team and system summary |

---

## Load by task type

*This section is generated during setup based on your team's tools and repos.*

| Tag | Load when... | File |
|---|---|---|
| `[investigate]` | Starting any investigation, analysis, or exploratory work | `context-glue/knowledge/investigate.md` |
| `[architecture]` | Cross-system questions, contracts, integration bugs, failure modes | `context-glue/knowledge/ARCHITECTURE_MAP.md` |

---

## Declaring loaded rooms

After loading, state rooms in one line before proceeding:

> Rooms loaded: overview · [system-a] · [investigate]

Update the declaration if task scope shifts mid-session.
