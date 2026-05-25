## First-time setup - context-glue

This wizard configures context-glue for your team. It will scan your workspace, ask a few questions, and generate a knowledge base tailored to your stack. Follow every step in order.

---

### Step 1 - Capability check

Before anything else, confirm you can do the following. If any capability is missing, warn the user and note which steps will be skipped.

- Can you read files from disk? *(required for all workflows)*
- Can you write and edit files? *(required for knowledge generation)*
- Can you run terminal/shell commands? *(required for git and any CLI tools)*

---

### Step 2 - Check for existing setup

Read `context-glue/settings.json`. Check `workspace.setup_complete`.

- If `true`: say - "context-glue is already configured for this workspace. Re-run setup? This will overwrite the generated knowledge files. (yes / no)"
  - If no: stop. Say "Run `Read context-glue/init.md` to start a session."
  - If yes: continue.
- If `false` or missing: continue.

---

### Step 3 - Team info

Ask these two questions together in a single message:

> 1. What is your team or project name?
> 2. In one sentence, what does your team do?

Wait for both answers before continuing.

---

### Step 4 - Scan for repos

If terminal is available, list all sibling directories of `context-glue/` that contain a `.git` folder:

```bash
# bash/zsh
ls -d ../*/  | xargs -I{} sh -c 'test -d "{}/.git" && echo {}'

# PowerShell
Get-ChildItem -Path .. -Directory | Where-Object { Test-Path (Join-Path $_.FullName ".git") } | Select-Object -ExpandProperty Name
```

If terminal is unavailable, ask the user: "What repos are in your workspace? List their folder names."

Present the list and ask:
> "Found these repos in your workspace:
> - repo-a
> - repo-b
>
> Any to exclude? Any missing? (or say 'all good')"

Wait for confirmation. Update the list based on the response. Remove `context-glue` itself from the list if it appears - it is tracked separately.

---

### Step 5 - Describe each repo

Ask in a single message, listing all confirmed repos:

> "Tell me about each repo - what it does, and any key files or folders worth knowing about. One or two sentences per repo is fine.
>
> repo-a:
> repo-b:
> ..."

Wait for all answers before continuing.

---

### Step 6 - Tools and systems

Ask in a single message:

> "What are the primary tools and systems your team works with? List everything - languages, platforms, services, databases, clouds, SaaS tools, etc."

Examples of valid answers:
- "Python, Airflow, dbt, Snowflake, Azure Blob"
- "React, Node.js, PostgreSQL, AWS Lambda, GitHub Actions"
- "Go microservices, Kubernetes, Kafka, Datadog, Terraform"
- "HubSpot, Salesforce, Looker, BigQuery, Segment"
- "Figma, Notion, Google Analytics, Meta Ads API"

Wait for the response. This drives room generation and env.template creation.

---

### Step 7 - Workflow conventions

Ask these questions together in a single message:

> 1. Do you use a ticket tracker? (e.g. Jira, Linear, GitHub Issues, Notion, or none)
> 2. What does a ticket ID look like in your system? (e.g. PROJ-123, #42 - or skip if you don't use a tracker)
> 3. What is your default branch name? (main / master / develop / other)
> 4. Do you follow a commit message convention? (e.g. Conventional Commits, a custom prefix, or none)

Wait for all answers before continuing. These configure the session prompts and git sync behavior.

---

### Step 8 - Seed knowledge for each room

Based on the tools from step 6, determine which knowledge rooms will be generated (same rules as step 11c below - one room per major system or tool category).

Present the room list and ask in a single message:

> "I'll create knowledge rooms for: [list of room tags]. Before generating, tell me anything you already know that's worth capturing for each one - key files or entry points, common commands, gotchas, patterns. One or two sentences per room is fine. Skip any you want to leave empty and build up over time.
>
> [room-tag]:
> [room-tag]:
> ..."

Wait for the full response before continuing. The user can answer as many or as few rooms as they like.

---

### Step 9 - Credentials

Ask:

> "Do any of these tools require credentials or connection details to use from the command line or through your agent - for example: database connections, API keys, CLI auth tokens? (yes / no)"

If **yes**:
> "For each tool that needs credentials, what connection variables are needed? Describe or list them and I'll generate the template."

For common tools, use standard variable names. Examples:

| Tool | Variables |
|---|---|
| Snowflake CLI | `SNOW_ACCOUNT`, `SNOW_USER`, `SNOW_AUTHENTICATOR`, `SNOW_WAREHOUSE`, `SNOW_ROLE` |
| PostgreSQL | `PG_HOST`, `PG_PORT`, `PG_USER`, `PG_PASSWORD`, `PG_DATABASE` |
| MySQL | `MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE` |
| AWS CLI | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION` |
| dbt Cloud | `DBT_CLOUD_API_KEY`, `DBT_ACCOUNT_ID` |
| GitHub | `GITHUB_TOKEN` |
| OpenAI | `OPENAI_API_KEY` |
| Custom/unknown | Ask: "What connection info does {tool} need?" - then generate `{TOOL}_API_KEY`, `{TOOL}_URL`, etc. |

If **no**: skip env.template creation.

---

### Step 10 - Confirm before generating

Present a summary and ask for confirmation before writing anything:

> "Here's what I'll create:
>
> **settings.json** - updated with workspace name, repo list, ticket tracker, and default branch
> **knowledge/stack/overview.md** - one-page summary of your team and stack
> **knowledge/INDEX.md** - palace map with rooms for: [list room tags]
> **knowledge/stack/{system}.md** - room files for: [list systems]
> **env.template** - credential template for: [list tools, or 'none needed']
>
> Ready to generate? (yes / no)"

Wait for confirmation.

---

### Step 11 - Generate files

On confirmation, write all files in this order.

#### 11a. Update settings.json

Read `context-glue/settings.json`, then write it back with these fields updated:
- `workspace.name` -> team name from step 3
- `workspace.repos` -> confirmed repo list from step 4 (as a JSON array of strings)
- `workspace.setup_complete` -> `true`
- `workspace.ticket_tracker` -> ticket tracker from step 7 (e.g. "Jira", "Linear", "GitHub Issues", "none")
- `workspace.ticket_id_example` -> example ticket ID from step 7 (e.g. "PROJ-123", "#42", or "" if none)
- `git.default_branch` -> default branch from step 7 (e.g. "main", "master", "develop")

#### 11b. Write knowledge/stack/overview.md

```markdown
## Last updated: {today's date}
# Stack Overview - {team name}

## What this team does
{one sentence from step 3}

## Repos in this workspace

| Repo | Purpose |
|---|---|
{one row per repo, using descriptions from step 5}

## Tool stack

| Category | Tool / System |
|---|---|
{one row per tool mentioned in step 6, with a reasonable category - e.g. Orchestration, Transformation, Warehouse, Source Systems, CI/CD, Monitoring, Frontend, Backend, Infra, Analytics, CRM, etc.}

## Workflow

| Setting | Value |
|---|---|
| Ticket tracker | {from step 7, or "none"} |
| Ticket ID format | {from step 7, or "n/a"} |
| Default branch | {from step 7} |
| Commit convention | {from step 7, or "none"} |

## Navigation hints

| If you're working on... | Go to room |
|---|---|
{one row per room generated in 11c, with tag and file path}
| Cross-system questions, contracts, failure modes | `context-glue/knowledge/ARCHITECTURE_MAP.md` `[architecture]` |
| Investigation or exploratory analysis | `context-glue/knowledge/investigate.md` `[investigate]` |
```

#### 11c. Write knowledge/INDEX.md

```markdown
## Last updated: {today's date}
# Knowledge Palace - Index

Read at session start. Load only the rooms tagged for the current task - do not pre-load everything.

---

## Always load (every session)

| Room | File | What it contains |
|---|---|---|
| Stack Overview | `context-glue/knowledge/stack/overview.md` | One-page team and system summary |

---

## Load by task type

| Tag | Load when... | File |
|---|---|---|
{one row per system room generated in 11d}
| `[investigate]` | Starting any investigation, analysis, or exploratory work | `context-glue/knowledge/investigate.md` |
| `[architecture]` | Cross-system questions, contracts, integration bugs | `context-glue/knowledge/ARCHITECTURE_MAP.md` |

---

## Declaring loaded rooms

After loading, state rooms in one line before proceeding:

> Rooms loaded: overview · [system-a] · [investigate]

Update the declaration if the task scope shifts mid-session.
```

**Room generation rules:**
- One room per major system or tool category from step 6
- Group closely related tools (e.g. "React + TypeScript" -> one `[frontend]` room)
- Use descriptive tags that a teammate would recognize: `[orchestration]`, `[transformation]`, `[backend]`, `[frontend]`, `[infra]`, `[data-warehouse]`, `[analytics]`, `[ci-cd]`, `[monitoring]`, `[crm]`, etc.
- Always include `[investigate]` and `[architecture]` - do not generate room files for these, they already exist

#### 11d. Write one knowledge/stack/{system}.md per room

For each room generated above (skip `[investigate]` and `[architecture]`):

```markdown
## Last updated: {today's date}
# {System Name} - Knowledge Room

## What it does
{Description inferred from repo descriptions (step 5) and tool names (step 6). If the user's step 8 answer for this room describes the system, incorporate it here.}

## Key files / entry points
{If the user provided key files or entry points for this room in step 8: write them as a bullet list.}
{If not provided: leave the placeholder comment below.}
<!-- Add paths to important files as you discover them -->

## Patterns and conventions
{If the user provided patterns or conventions for this room in step 8: write them as a bullet list.}
{If not provided: leave the placeholder comment below.}
<!-- Add team conventions, naming rules, and standard approaches as tickets are completed -->

## Gotchas and known issues
{If the user provided gotchas or known issues for this room in step 8: write them as a bullet list.}
{If not provided: leave the placeholder comment below.}
<!-- Add pitfalls, edge cases, and hard-won lessons -->

## Common operations
{If the user provided common commands or workflows for this room in step 8: write them as a bullet list or code block as appropriate.}
{If not provided: leave the placeholder comment below.}
<!-- Add frequently run commands, queries, or workflows -->
```

Use judgment when placing seeded content from step 8 - a single answer may contain material for multiple sections, split it accordingly.

#### 11e. Write env.template (only if credentials are needed)

```
# env.template - Credential variables for context-glue
# Copy this file to env.local and fill in your values.
# env.local is gitignored and will NEVER be committed.
#
# Setup: cp context-glue/env.template context-glue/env.local

{for each tool needing credentials:}

# -- {Tool name} ------------------------------------------
{VAR_NAME}=
{VAR_NAME}=
```

---

### Step 12 - Next steps

After all files are written, confirm and give instructions in a single message:

> "Setup complete. Generated:
> - settings.json (updated)
> - knowledge/stack/overview.md
> - knowledge/INDEX.md
> - knowledge/stack/{list of room files}
> {- env.template (if applicable)}
>
> **Next steps:**
>
> {If env.template was created:}
> 1. Create your personal credentials file:
>    `cp context-glue/env.template context-glue/env.local`
>    Open env.local and fill in your values. This file is gitignored and stays on your machine.
>
> 2. The room files in `knowledge/stack/` now have a starting point based on what you told me. Add more as you work - each completed ticket feeds findings back into these files.
>
> 3. Commit the generated files to share with your team:
>    ```bash
>    cd context-glue
>    git add knowledge/ settings.json
>    git commit -m 'feat: initialize context-glue for {team name}'
>    git push
>    ```
>
> 4. New team members clone this repo and run `Read context-glue/prompts/setup.md` - they'll be walked through creating their own env.local.
>
> Ready. Start your first session:
> `Read context-glue/init.md`"
