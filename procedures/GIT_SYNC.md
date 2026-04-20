# GIT_SYNC.md — Repository Sync Check

This procedure runs at the start of every session.
It requires `git.enable_sync: true` in `settings.json`. If sync is disabled, skip this procedure entirely.
It also requires terminal access. If the agent's capability check flagged terminal as unavailable, skip this procedure and note it.

---

## Repositories to check

Read `context-glue/settings.json`. The `workspace.repos` array lists every repo name to check. All repos are siblings of `context-glue/` in the same parent folder.

If `workspace.repos` is empty, say:
> "No repos configured for sync. Run `Read context-glue/prompts/setup.md` to configure them."

Then skip this procedure.

---

## Procedure

### Step 1 — Collect status for all repos

For each repo in `workspace.repos`, run these git commands (read-only, always allowed).

**Important — PowerShell compatibility:**
`git fetch` writes progress to stderr. To avoid false failures, run it separately with stderr suppressed.

```powershell
# Call 1 — fetch (stderr suppressed)
cd {repo_path} ; git fetch origin 2>$null

# Call 2 — branch, uncommitted changes, and how far behind
cd {repo_path} ; Write-Output "BRANCH: $(git branch --show-current)" ; Write-Output "STATUS: $(git status --short)" ; Write-Output "BEHIND: $(git rev-list --count HEAD..origin/main 2>$null)"
```

If the `BEHIND` line is empty, retry with `origin/master`:
```powershell
cd {repo_path} ; Write-Output "BEHIND: $(git rev-list --count HEAD..origin/master 2>$null)"
```

**On Unix/macOS (bash/zsh):**
```bash
cd {repo_path} && git fetch origin 2>/dev/null
cd {repo_path} && echo "BRANCH: $(git branch --show-current)" && echo "STATUS: $(git status --short)" && echo "BEHIND: $(git rev-list --count HEAD..origin/main 2>/dev/null)"
```

Collect per repo:
- **Current branch**
- **Uncommitted changes** (any unstaged or staged changes)
- **Commits behind** origin/main (or origin/master)

### Step 2 — Present status

Show all repos in a single status block:

```
Repository sync check:

  context-glue          main    up to date
  repo-a                main    3 commits behind origin/main
  repo-b                feature/TICKET-42    not on main
```

If all repos are on main and up to date, say so and move on — no further action needed.

### Step 3 — Offer sync

If only one repo needs syncing, present it as a simple yes/no.

If **multiple repos** need syncing, list them all in a single prompt so the user can pick which to sync in one action:

> **Which repos should we sync to latest?**
>
> - [ ] repo-a — on `main`, 3 commits behind origin/main
> - [ ] repo-b — on `feature/TICKET-42`, 5 commits behind origin/main
>
> Reply with repo names, or "none".

### Step 4 — Execute sync (per confirmed repo)

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
cd {repo_path} ; git checkout main 2>$null ; if (-not $?) { git checkout master }
cd {repo_path} ; git pull origin main 2>$null ; if (-not $?) { git pull origin master }

# bash/zsh
cd {repo_path} && (git checkout main 2>/dev/null || git checkout master) && (git pull origin main 2>/dev/null || git pull origin master)
```

Confirm:
> ✓ `{repo}` synced to main (now at {short_commit_hash})

### Step 5 — Summary

After processing all repos, state the final status in one line:

> Sync complete — all repos on main and up to date.

Or:

> Sync complete — context-glue: synced | repo-a: synced | repo-b: skipped (user choice)

---

## Edge cases

- **Repo directory not found:** skip and warn: "`{repo}` not found at expected path. Clone it as a sibling of `context-glue/`."
- **No remote named origin:** warn and skip: "`{repo}` has no remote named `origin`. Skipping sync."
- **Neither main nor master on remote:** warn and skip: "Could not determine default branch for `{repo}`. Skipping sync."
- **Merge conflict during pull:** stop and warn: "⚠️ Merge conflict in `{repo}`. Resolve manually, then re-run the session."
- **Terminal unavailable:** skip entire procedure and note: "Git sync skipped — terminal not available in this environment."
