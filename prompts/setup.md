## First-time setup — context-glue

This wizard configures context-glue for your team. It will scan your workspace, ask a few questions, and generate a knowledge base tailored to your stack. Follow every step in order.

---

### Step 1 — Capability check

Before anything else, confirm you can do the following. If any capability is missing, warn the user and note which steps will be skipped.

- Can you read files from disk? *(required for all workflows)*
- Can you write and edit files? *(required for knowledge generation)*
- Can you run terminal/shell commands? *(required for git and any CLI tools)*

---

### Step 2 — Check for existing setup

Read `context-glue/settings.json`. Check `workspace.setup_complete`.

- If `true`: say — "context-glue is already configured for this workspace. Re-run setup? This will overwrite the generated knowledge files. (yes / no)"
  - If no: stop. Say "Run `Read context-glue/init.md` to start a session."
  - If yes: continue.
- If `false` or missing: continue.

---

### Step 3 — Team info

Ask these two questions together in a single message:

> 1. What is your team or project name?
> 2. In one sentence, what does your team do?

Wait for both answers before continuing.

---

### Step 4 — Scan for repos

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

Wait for confirmation. Update the list based on the response. Remove `context-glue` itself from the list if it appears — it is tracked separately.

---

### Step 5 — Describe each repo

For each repo in the confirmed list, ask one at a time:

> "What does `{repo-name}` do? One sentence is fine."

Collect all answers before moving on.

---

### Step 6 — Tools and systems

Ask in a single message:

> "What are the primary tools and systems your team works with? List everything — languages, platforms, services, databases, clouds, SaaS tools, etc."

Examples of valid answers:
- "Python, Airflow, dbt, Snowflake, Azure Blob"
- "React, Node.js, PostgreSQL, AWS Lambda, GitHub Actions"
- "Go microservices, Kubernetes, Kafka, Datadog, Terraform"
- "HubSpot, Salesforce, Looker, BigQuery, Segment"
- "Figma, Notion, Google Analytics, Meta Ads API"

Wait for the response. This drives room generation and env.template creation.

---

### Step 7 — Credentials

Ask:

> "Do any of these tools require credentials or connection details to use from the command line or through your agent — for example: database connections, API keys, CLI auth tokens? (yes / no)"

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
| Custom/unknown | Ask: "What connection info does {tool} need?" — then generate `{TOOL}_API_KEY`, `{TOOL}_URL`, etc. |

If **no**: skip env.template creation.

---

### Step 8 — Confirm before generating

Present a summary and ask for confirmation before writing anything:

> "Here's what I'll create:
>
> **knowledge/stack/overview.md** — one-page summary of your team and stack
> **knowledge/INDEX.md** — palace map with rooms for: [list room tags]
> **knowledge/stack/{system}.md** — room files for: [list systems]
> **settings.json** — updated with workspace name and repo list
> **env.template** — credential template for: [list tools, or 'none needed']
>
> Ready to generate? (yes / no)"

Wait for confirmation.

---

### Step 9 — Generate files

On confirmation, write all files in this order.

#### 9a. Update settings.json

Read `context-glue/settings.json`, then write it back with these fields updated:
- `workspace.name` → team name from step 3
- `workspace.repos` → confirmed repo list from step 4 (as a JSON array of strings)
- `workspace.setup_complete` → `true`

#### 9b. Write knowledge/stack/overview.md

```markdown
## Last updated: {today's date}
# Stack Overview — {team name}

## What this team does
{one sentence from step 3}

## Repos in this workspace

| Repo | Purpose |
|---|---|
{one row per repo, using descriptions from step 5}

## Tool stack

| Category | Tool / System |
|---|---|
{one row per tool mentioned in step 6, with a reasonable category — e.g. Orchestration, Transformation, Warehouse, Source Systems, CI/CD, Monitoring, Frontend, Backend, Infra, Analytics, CRM, etc.}

## Navigation hints

| If you're working on... | Go to room |
|---|---|
{one row per room generated in 9c, with tag and file path}
| Cross-system questions, contracts, failure modes | `context-glue/knowledge/ARCHITECTURE_MAP.md` `[architecture]` |
| Data investigation or exploratory analysis | `context-glue/knowledge/investigate.md` `[investigate]` |
```

#### 9c. Write knowledge/INDEX.md

```markdown
## Last updated: {today's date}
# Knowledge Palace — Index

Read at session start. Load only the rooms tagged for the current task — do not pre-load everything.

---

## Always load (every session)

| Room | File | What it contains |
|---|---|---|
| Stack Overview | `context-glue/knowledge/stack/overview.md` | One-page team and system summary |

---

## Load by task type

| Tag | Load when... | File |
|---|---|---|
{one row per system room generated in 9d}
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
- Group closely related tools (e.g. "React + TypeScript" → one `[frontend]` room)
- Use descriptive tags that a teammate would recognize: `[orchestration]`, `[transformation]`, `[backend]`, `[frontend]`, `[infra]`, `[data-warehouse]`, `[analytics]`, `[ci-cd]`, `[monitoring]`, `[crm]`, etc.
- Always include `[investigate]` and `[architecture]` — do not generate room files for these, they already exist

#### 9d. Write one knowledge/stack/{system}.md per room

For each room generated above (skip `[investigate]` and `[architecture]`):

```markdown
## Last updated: {today's date}
# {System Name} — Knowledge Room

## What it does
{description — inferred from repo descriptions and tool names from steps 5 and 6}

## Key files / entry points
<!-- Add paths to important files as you discover them -->

## Patterns and conventions
<!-- Add team conventions, naming rules, and standard approaches as tickets are completed -->

## Gotchas and known issues
<!-- Add pitfalls, edge cases, and hard-won lessons -->

## Common operations
<!-- Add frequently run commands, queries, or workflows -->
```

#### 9e. Write env.template (only if credentials are needed)

```
# env.template — Credential variables for context-glue
# Copy this file to env.local and fill in your values.
# env.local is gitignored and will NEVER be committed.
#
# Setup: cp context-glue/env.template context-glue/env.local

{for each tool needing credentials:}

# ── {Tool name} ──────────────────────────────────────
{VAR_NAME}=
{VAR_NAME}=
```

---

### Step 10 — Next steps

After all files are written, confirm and give instructions in a single message:

> "Setup complete. Generated:
> - knowledge/stack/overview.md
> - knowledge/INDEX.md
> - knowledge/stack/{list of room files}
> - settings.json (updated)
> {- env.template (if applicable)}
>
> **Next steps:**
>
> {If env.template was created:}
> 1. Create your personal credentials file:
>    `cp context-glue/env.template context-glue/env.local`
>    Open env.local and fill in your values. This file is gitignored and stays on your machine.
>
> 2. The room files in `knowledge/stack/` are stubs. Add detail as you work — each completed ticket is a chance to feed findings back into these files.
>
> 3. Commit the generated files to share with your team:
>    ```bash
>    cd context-glue
>    git add knowledge/ settings.json
>    git commit -m 'feat: initialize context-glue for {team name}'
>    git push
>    ```
>
> 4. New team members clone this repo and run `Read context-glue/prompts/setup.md` — they'll be walked through creating their own env.local.
>
> Ready. Start your first session:
> `Read context-glue/init.md`"
