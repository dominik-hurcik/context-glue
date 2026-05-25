<p align="center">
  <h1 align="center">context-glue</h1>
</p>

<p align="center">
  Persistent context for AI agents, across any stack.<br>
  One command. Every session.
</p>

<br>

<div align="left">

---

## The problem

AI agents start every session knowing nothing. Your stack, your ticket history, your investigation notes, your conventions - all gone. You spend the first part of every session re-explaining context, or the agent makes mistakes because it doesn't know how your systems fit together.

context-glue fixes this with no API integrations, no plugins, and no vendor lock-in. It is a folder of plain markdown files that sits next to your other repos. The agent reads what it needs at the start of each session and stays in context for the rest.

---

## Why I built this

In some temas, we can often work across several repos, which was often the case for my personal experience. Maybe a pipeline repo, an orchestration layer, a data warehouse, a handful of extractors. Each one has its own conventions, its own quirks, its own history of decisions.

Opening a new AI agent session could mean: explaining the stack from scratch, re-linking how the systems connect, reminding the agent what we tried last time and why it did not work. Half the time it would still confidently suggest something we had already ruled out two weeks earlier.

I looked at the existing solutions. Most of them require API integrations, maybe some paid plugins, or locking into a specific agent's ecosystem. I did not want any of that. I just wanted the agent to know what I know, every session, without me having to repeat myself and more importantly, structured only for myself and my team.

Context-glue provides exactly that! The idea is simple: keep a structured folder of markdown files next to your repos. The agent reads them at the start of every session. That is the whole thing.

What surprised me was how well it worked. Not just for giving the agent context, but for the team. When you have a place where knowledge is meant to land, where findings from one ticket feed into the next... the team gets smarter over time instead of each person starting from zero.

---

## How it works

You keep a `context-glue` folder next to your other repos. At the start of every AI session, you paste one command. The agent loads your team's knowledge, checks your repos are in sync, and asks what you want to do. No configuration beyond a one-time setup. No integrations to maintain.

**Knowledge compounds over time.** Every ticket you complete feeds findings back into shared knowledge files. The next person who works on a similar problem starts from a richer base than the person before them.

---

## Getting started

1. Clone this repo into the same folder as your other repos:
   ```bash
   git clone https://github.com/{org}/context-glue.git
   ```

2. Open your AI agent from that parent folder.

3. Run the setup wizard:
   ```
   Read context-glue/prompts/setup.md
   ```

The agent will scan your repos, ask about your stack and workflow, and generate a knowledge base tailored to your team. Takes about 10 minutes.

New team members follow the same three steps. The setup wizard walks them through cloning any missing repos and creating their personal credentials file.

---

## Every session

One command starts every session:

```
Read context-glue/init.md
```

The agent loads all context, checks your repos are in sync, then asks what you want to do:

| Option | What it does |
|---|---|
| **Ad-hoc** | Investigate, explore, or track findings without a ticket |
| **New ticket** | Start a tracked session for a task or ticket |
| **Resume ticket** | Pick up a ticket where you left off |
| **Complete ticket** | Close a ticket and promote findings to shared knowledge |
| **Quick question** | Ask something one-off with no tracking or session setup |

The agent will never run destructive commands, make git commits, or proceed past open questions without asking you first.

---

## Shared knowledge

The `knowledge/` folder is your team's shared asset. It is committed to git. When you complete a ticket, the agent reviews everything discovered during the session and asks which findings are worth keeping - then writes them into the right knowledge files so the next person benefits.

After completing a ticket, push the updates:

```bash
cd context-glue
git add knowledge/
git commit -m "knowledge: {TICKET} findings"
git push
```

Then open a pull request. The reviewer can run `procedures/KNOWLEDGE_PR_CHECKLIST.md` to check the additions are clean before merging.

**What is shared vs personal:**

| What | Shared (committed) | Personal (gitignored) |
|---|---|---|
| `knowledge/` | yes | |
| `env.template` | yes - shows what variables are needed | |
| `tickets/` | | yes - local only |
| `adhoc/` | | yes - local only |
| `env.local` | | yes - your credentials |

---

## How it works internally

### Session init flow

When you paste `Read context-glue/init.md`, the agent follows this chain:

```
init.md
  └── START_HERE.md       capability check + loading order + session rules
        ├── settings.json
        ├── knowledge/INDEX.md              <- palace map
        ├── knowledge/stack/overview.md     <- always-load
        └── env.local                       <- always-load (gitignored; your credentials)
  └── GIT_SYNC.md                          <- only loaded if a repo is behind
  └── Ask user: ad-hoc / new / resume / complete / quick question
  └── Route to selected prompt
        └── [ad-hoc]    -> adhoc/{name}/.agent/PROGRESS.md + FINDINGS.md
        └── [ticket]    -> tickets/{TICKET}/.agent/CONTEXT.md + CHECKLIST.md + ...
```

### Knowledge palace

The knowledge base uses a palace structure - a map of rooms, each tagged for a specific task type. The agent reads the palace map (`INDEX.md`) at every session start, then loads only the rooms it needs for the current task.

Rooms are generated for your specific stack during setup. Examples across different team types:

| Team type | Example rooms |
|---|---|
| Data engineering | `[orchestration]` `[transformation]` `[warehouse]` `[extraction]` |
| Software engineering | `[backend]` `[frontend]` `[infra]` `[api]` |
| Platform / DevOps | `[ci-cd]` `[infra]` `[monitoring]` `[services]` |
| Analytics / BI | `[data-sources]` `[reporting]` `[warehouse]` `[pipelines]` |

Every team also gets `[investigate]` and `[architecture]` rooms automatically.

### Repository sync

At the start of every session the agent runs a quick check on all configured repos. If any repo is behind its default branch, the agent tells you and offers to sync. All up to date - it moves on without loading the full sync procedure.

Controlled by `git.enable_sync` in `settings.json`.

---

## File structure

```
context-glue/
├── init.md                       single entry point - start here
├── README.md
├── settings.json                 agent behavior toggles
├── env.template                  credential variable template (generated by setup)
├── .gitignore
├── START_HERE.md                 capability check, loading order, session rules, permissions
│
├── procedures/                   reusable agent procedures
│   ├── GIT_SYNC.md               repository sync procedure (loaded only when needed)
│   └── KNOWLEDGE_PR_CHECKLIST.md knowledge PR review checklist
│
├── knowledge/                    shared team knowledge - committed to git
│   ├── INDEX.md                  palace map - always-load rooms + tagged rooms
│   ├── investigate.md            investigation protocol (all system types)
│   ├── ARCHITECTURE_MAP.md       cross-system dependency map
│   └── stack/                    platform knowledge, split by room
│       ├── overview.md           one-page system summary (always loaded)
│       └── {system}.md           one file per system (generated by setup)
│
├── prompts/                      session workflows
│   ├── setup.md                  first-time setup wizard
│   ├── adhoc.md                  ad-hoc investigation - track findings
│   ├── new.md                    start a new ticket session
│   ├── resume.md                 resume an existing ticket session
│   └── complete.md               close a ticket; promote findings to shared knowledge
│
├── tickets/                      ticket work - gitignored, stays local
│   └── {TICKET}/
│       └── .agent/
│           ├── CONTEXT.md        living record of decisions and findings
│           ├── CHECKLIST.md      ordered task list
│           ├── TESTING.md        test cases and results
│           └── PULLREQUEST.md    PR/MR description; copy-paste ready
│
└── adhoc/                        ad-hoc analyses - gitignored, stays local
    └── {analysis-name}/
        └── .agent/
            ├── PROGRESS.md       chronological investigation log
            └── FINDINGS.md       key findings and open questions
```

---

## Supported agents

context-glue uses a single `START_HERE.md` that works with any AI agent. At session start, the agent runs a capability self-check and tells you upfront what works in its environment (file access, terminal, credentials). No agent-specific configuration needed.

Works with: Claude, GPT/Codex, Gemini, Copilot, Cursor, and any other agent that can read files and follow instructions.

---

## settings.json reference

Controls agent behavior. Edit once - applies every session.

| Setting | Default | Description |
|---|---|---|
| `workspace.name` | `""` | Team or project name (set by setup) |
| `workspace.repos` | `[]` | Repo names to sync at session start (set by setup) |
| `workspace.setup_complete` | `false` | Set to `true` by setup wizard on first run |
| `workspace.ticket_tracker` | `""` | Ticket system your team uses (set by setup, e.g. "Jira", "Linear", "GitHub Issues", "none") |
| `workspace.ticket_id_example` | `""` | Example ticket ID shown when starting a new ticket (set by setup, e.g. "PROJ-123", "#42") |
| `git.enable_sync` | `true` | Check repos at session start and offer to sync |
| `git.default_branch` | `"main"` | Default branch name used for git sync (set by setup) |
| `git.enable_commits` | `false` | Master switch - `false` means user handles all git commits |
| `git.commit_strategy` | `"never"` | `"per_task"`, `"session_end"`, or `"never"` |
| `workflow.auto_update_checklist` | `true` | Agent marks `[x]` on completion without being asked |
| `workflow.auto_update_context` | `true` | Agent appends decisions to CONTEXT.md as it works |
| `workflow.auto_update_pullrequest` | `true` | Agent keeps PULLREQUEST.md current |
| `workflow.summarize_on_session_end` | `true` | Agent writes a handoff summary when you say you are done |
| `investigation.max_actions_per_session` | `50` | Hard cap on queries, commands, or API calls per investigation |
| `investigation.always_cap_results` | `true` | Agent always adds a result cap (SQL LIMIT, API page size, log line count, etc.) unless told otherwise |
| `agent.verbose_context_loading` | `false` | `false` = single summary line at startup; `true` = full fenced load receipt with one row per file |

---

## Rules

- Never use git commits unless `git.enable_commits: true` - user handles all git
- Git sync (fetch/checkout/pull) is allowed when `git.enable_sync: true` and the user confirms per-repo
- Never run destructive or mutating commands without explicit confirmation
- Never commit `env.local` - it contains personal credentials
- Always state which knowledge rooms are loaded at the start of a session
