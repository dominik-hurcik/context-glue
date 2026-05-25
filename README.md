<p align="center">
  <h1 align="center">context-glue</h1>
</p>

<p align="center">
  Context-keeping system for multi-repo teams.<br>
  One prompt. Full context. Any stack.
</p>

<br>

<div align="left">

---

## What is it?

**context-glue** is a structured set of files that gives an AI agent persistent, shared context across all of your team's tools and repos - without API integrations, without plugins, without vendor lock-in.

Your team probably works across multiple systems: a ticketing tool, a source repo, an orchestration platform, a database, a data warehouse, a CI/CD pipeline. Each system holds context that matters. When you open a new AI session, all of that is gone. context-glue fixes that.

It works by being a repo that lives alongside your other repos. It holds your stack knowledge, your ticket history, your investigation logs, and your team's accumulated findings - all as plain markdown files, structured so an AI agent can load exactly what it needs for the current task.

**The key insight:** no API connections needed. No JIRA integration, no Snowflake plugin, no GitHub app. The agent reads files, writes files, and runs commands you already have access to. That's it.

- AI agents start every session cold. context-glue makes that irrelevant.
- Glues together any combination of systems - data engineering, software engineering, platform, DevOps, analytics, marketing ops, and more.
- Not tied to any specific tool, language, or cloud provider.
- Knowledge grows over time - each completed ticket feeds back into shared context that every team member benefits from.
- Zero setup overhead for new systems - describe a tool in a sentence, get a knowledge room for it.

---

## First time?

1. Clone this repository into the same folder as your other repos:
   ```bash
   git clone https://github.com/{org}/context-glue.git
   ```
2. From that parent folder, open your AI agent's CLI.
3. Run the setup wizard:
   ```
   Read context-glue/prompts/setup.md
   ```

The agent will scan your repos, ask a few questions about your stack, and generate a knowledge base tailored to your team. Takes about 10 minutes.

---

## Quick reference

One command starts every session:

```
Read context-glue/init.md
```

The agent will:
1. Load all context and settings
2. Check if your repositories are up to date (and offer to sync them)
3. Ask what you want to do:

| Option | What it does |
|---|---|
| **Ad-hoc** | Investigate, run queries, track findings |
| **New ticket** | Start a new task or ticket session |
| **Resume ticket** | Pick up where you left off |
| **Complete ticket** | Close a ticket and promote findings to shared knowledge |

**Power users:** jump directly to a specific workflow:

| When | Paste this |
|---|---|
| Ad-hoc analysis | `Read context-glue/prompts/adhoc.md` |
| New ticket | `Read context-glue/prompts/new.md` |
| Resume ticket | `Read context-glue/prompts/resume.md` |
| Complete ticket | `Read context-glue/prompts/complete.md` |
| First-time setup | `Read context-glue/prompts/setup.md` |

**What the AI will never do without asking you first:** run destructive or mutating commands, make git commits, or proceed past a task that has open questions or risks.

**If something feels wrong:** the agent always states which files it loaded at startup - if it loaded the wrong context, say "reload" and tell it the correct ticket.

**After completing a ticket**, push your knowledge updates:

```bash
cd context-glue
git add knowledge/
git commit -m "knowledge: {TICKET} findings"
git push
```

Then create a pull request.

---

## How it works

### Session init flow

When you paste `Read context-glue/init.md`, the agent follows this chain:

```
init.md
  └── START_HERE.md       capability check + loading order + session rules
        ├── settings.json
        ├── knowledge/INDEX.md              ← palace map
        ├── knowledge/stack/overview.md     ← always-load
        └── env.local                       ← always-load (gitignored; your credentials)
  └── GIT_SYNC.md                          ← only loaded if a repo is behind
  └── Ask user: ad-hoc / new / resume / complete / quick question
  └── Route to selected prompt
        └── [ad-hoc]    → adhoc/{name}/.agent/PROGRESS.md + FINDINGS.md
        └── [ticket]    → tickets/{TICKET}/.agent/CONTEXT.md + CHECKLIST.md + ...
```

### Knowledge Palace

The knowledge base uses a **palace structure** - a map of rooms, each tagged for a specific task type. The agent reads the palace map (`INDEX.md`) at every session start, then walks into only the rooms it needs for the current task.

Rooms are generated for your specific stack during setup. Examples across different team types:

| Team type | Example rooms |
|---|---|
| Data engineering | `[orchestration]` · `[transformation]` · `[warehouse]` · `[extraction]` |
| Software engineering | `[backend]` · `[frontend]` · `[infra]` · `[api]` |
| Platform / DevOps | `[ci-cd]` · `[infra]` · `[monitoring]` · `[services]` |
| Analytics / BI | `[data-sources]` · `[reporting]` · `[warehouse]` · `[pipelines]` |

Every team also gets `[investigate]` and `[architecture]` rooms automatically.

The more your team uses context-glue, the richer these rooms become.

### Repository sync

At the start of every session, the agent checks all workspace repositories (configured during setup). If any repo is behind `origin/main` or on a different branch, the agent will tell you and offer to sync - one repo at a time, so you stay in control.

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
│   ├── GIT_SYNC.md              repository sync check
│   └── KNOWLEDGE_PR_CHECKLIST.md  knowledge PR review checklist
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

## Team onboarding

### For new team members - let the AI walk you through it

After cloning the repo, paste this into your AI agent:

```
Read context-glue/prompts/setup.md
```

The agent will:
- Check if git is installed (and help install it if not)
- Help you clone any missing sibling repos
- Create your personal `env.local` from `env.template`
- Explain the git sync feature

Takes about 10 minutes.

---

### Manual setup

#### 1. Clone all repos into the same folder

```bash
mkdir workspace && cd workspace
git clone https://github.com/{org}/context-glue.git
git clone https://github.com/{org}/{your-repo}.git
# ... repeat for all repos
```

#### 2. Create your credentials file

```bash
cp context-glue/env.template context-glue/env.local
```

Open `env.local` and fill in your values. This file is gitignored and will never be committed.

#### 3. Start working

```
Read context-glue/init.md
```

---

## Team workflow - shared knowledge

The `knowledge/` folder is the team's shared asset. It is committed to git and everyone benefits when it is updated.

**Before starting any session:**

The git sync feature handles this automatically at session start. Or manually:

```bash
cd context-glue && git pull
```

**After completing a ticket:**

When you run the "Complete ticket" workflow, the agent promotes findings to shared knowledge files and then reminds you to push:

```bash
cd context-glue
git add knowledge/
git commit -m "knowledge: {TICKET} findings"
git push
```

Then create a pull request.

**Reviewing a knowledge PR:**

When a knowledge PR arrives, the reviewer should run through `procedures/KNOWLEDGE_PR_CHECKLIST.md` before approving. It covers four things:

- Does the new content contradict anything already in the target file?
- Is there duplication with another room?
- Is the finding scoped to the right room?
- Is the file still a reasonable size after the addition?

The agent runs the same checklist automatically during the "Complete ticket" workflow before telling you to push - so by the time the PR lands, the obvious issues should already be caught.

**What is shared vs personal:**

| What | Shared (committed) | Personal (gitignored) |
|---|---|---|
| `knowledge/` | yes | |
| `env.template` | yes - shows what's needed | |
| `tickets/` | | yes - local only |
| `adhoc/` | | yes - local only |
| `env.local` | | yes - your credentials |

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
