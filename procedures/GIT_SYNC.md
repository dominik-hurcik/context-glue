# GIT_SYNC.md - Repository Sync Check

This procedure runs at the start of every session.
It requires `git.enable_sync: true` in `settings.json`. If sync is disabled, skip this procedure entirely.
It also requires terminal access. If the agent's capability check flagged terminal as unavailable, skip this procedure and note it.

---

## Repositories to check

Read `context-glue/settings.json`. The `workspace.repos` array lists every repo name to check. All repos are siblings of `context-glue/` in the same parent folder.

Also read `git.default_branch` from settings - this is the branch to sync to. If the field is missing or empty, default to trying `main` then `master`.

If `workspace.repos` is empty, say:
> "No repos configured for sync. Run `Read context-glue/prompts/setup.md` to configure them."

Then skip this procedure.

---

## Procedure

### Step 1 - Collect status for all repos

For each repo in `workspace.repos`, run these git commands (read-only, always allowed).

Use `{default_branch}` (from `git.default_branch` in settings) as the target branch. If the field is missing, try `main` first and fall back to `master`.

**Important - PowerShell compatibility:**
`git fetch` writes progress to stderr. To avoid false failures, run it separately with stderr suppressed.

```powershell
# Call 1 - fetch (stderr suppressed)
cd {repo_path} ; git fetch origin 2>$null

# Call 2 - branch, uncommitted changes, and how far behind
cd {repo_path} ; Write-Output "BRANCH: $(git branch --show-current)" ; Write-Output "STATUS: $(git status --short)" ; Write-Output "BEHIND: $(git rev-list --count HEAD..origin/{default_branch} 2>$null)"
```

If the `BEHIND` line is empty and `default_branch` is not `master`, retry with `origin/master`:
```powershell
cd {repo_path} ; Write-Output "BEHIND: $(git rev-list --count HEAD..origin/master 2>$null)"
```

**On Unix/macOS (bash/zsh):**
```bash
cd {repo_path} && git fetch origin 2>/dev/null
cd {repo_path} && echo "BRANCH: $(git branch --show-current)" && echo "STATUS: $(git status --short)" && echo "BEHIND: $(git rev-list --count HEAD..origin/{default_branch} 2>/dev/null)"
```

Collect per repo:
- **Current branch**
- **Uncommitted changes** (any unstaged or staged changes)
- **Commits behind** origin/{default_branch}

### Step 2 - Present status

Show all repos in a single status block:

```
Repository sync check:

  context-glue          main    up to date
  repo-a                main    3 commits behind origin/main
  repo-b                feature/TICKET-42    not on main
```

If all repos are on main and up to date, say so and move on - no further action needed.

### Step 3 - Handle context-glue first (special case)

**context-glue is the tool itself.** If it is behind, the agent is running on stale instructions - continuing the session would be unreliable.

**If context-glue is behind origin/main:**

Do NOT include it in the multi-select prompt with other repos. Instead, stop and require a sync immediately:

> ⚠️ **context-glue is out of date** ({N} commits behind origin/main).
> This repo contains the instructions I'm running on right now - if it's stale, this session cannot be trusted.
> I need to sync it before we continue. Shall I sync context-glue now? (yes / no)

- If the user says **no**: end the session. Remind them to sync context-glue and restart.
- If the user says **yes**: sync context-glue (follow Step 4 below for the sync commands), then say:

> ✓ context-glue synced. **Please close and relaunch your agent from your workspace folder**, then paste `Read context-glue/init.md` to start a fresh session with the updated instructions.

Do not proceed further. The session must restart so the agent loads the updated files.

### Step 4 - Offer sync for remaining repos

After confirming context-glue is up to date (or after syncing it - in which case the session ends here), offer sync for the other repos.

If only one needs syncing, present it as a simple yes/no.

If **multiple repos** need syncing, list them all in a single prompt so the user can pick which to sync in one action:

> **Which repos should we sync to latest?**
>
> - [ ] repo-a - on `main`, 3 commits behind origin/main
> - [ ] repo-b - on `feature/TICKET-42`, 5 commits behind origin/main
>
> Reply with repo names, or "none".

### Step 5 - Execute sync (per confirmed repo)

Before syncing, check for uncommitted changes:

```bash
cd {repo_path} && git status --short
```

**If uncommitted changes exist:**
> ⚠️ `{repo}` has uncommitted changes. Syncing would risk losing them. Please commit or stash your changes first.

Do NOT proceed with sync for that repo. Move to the next.

**If clean:**

```bash
# PowerShell
cd {repo_path} ; git checkout {default_branch} 2>$null ; if (-not $?) { git checkout master }
cd {repo_path} ; git pull origin {default_branch} 2>$null ; if (-not $?) { git pull origin master }

# bash/zsh
cd {repo_path} && (git checkout {default_branch} 2>/dev/null || git checkout master) && (git pull origin {default_branch} 2>/dev/null || git pull origin master)
```

Confirm:
> ✓ `{repo}` synced to {default_branch} (now at {short_commit_hash})

### Step 6 - Summary

After processing all repos, state the final status in one line:

> Sync complete - all repos on main and up to date.

Or:

> Sync complete - context-glue: synced | repo-a: synced | repo-b: skipped (user choice)

---

## Edge cases

- **Repo directory not found:** skip and warn: "`{repo}` not found at expected path. Clone it as a sibling of `context-glue/`."
- **No remote named origin:** warn and skip: "`{repo}` has no remote named `origin`. Skipping sync."
- **Neither main nor master on remote:** warn and skip: "Could not determine default branch for `{repo}`. Skipping sync."
- **Merge conflict during pull:** stop and warn: "⚠️ Merge conflict in `{repo}`. Resolve manually, then re-run the session."
- **Terminal unavailable:** skip entire procedure and note: "Git sync skipped - terminal not available in this environment."
- **context-glue has uncommitted changes and is also behind:** warn and do not sync: "⚠️ context-glue has uncommitted changes and cannot be safely synced. Please commit or stash your changes, then relaunch and restart the session." Do not continue.
