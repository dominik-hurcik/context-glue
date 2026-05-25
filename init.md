# context-glue - Session Init

1. Read `context-glue/START_HERE.md` - follow the full instruction loading order before continuing.

2. Check `context-glue/settings.json` for `workspace.setup_complete`.
   - If `false` or the field is missing, stop immediately and say:
     > "context-glue hasn't been set up yet for this workspace. Run the setup wizard first:
     > `Read context-glue/prompts/setup.md`"
   - If `true`, continue.

3. Git sync check.

   Skip this entire step if any of the following are true:
   - `git.enable_sync` is `false` in settings.json
   - `workspace.repos` is empty
   - Terminal is not available (flagged in capability check)

   Otherwise, for each repo in `workspace.repos` plus `context-glue` itself, run these two commands:

   ```powershell
   # PowerShell
   cd {repo_path} ; git fetch origin 2>$null
   cd {repo_path} ; git rev-list --count HEAD..origin/{default_branch} 2>$null
   ```
   ```bash
   # bash/zsh
   cd {repo_path} && git fetch origin 2>/dev/null
   cd {repo_path} && git rev-list --count HEAD..origin/{default_branch} 2>/dev/null
   ```

   Use `git.default_branch` from settings as `{default_branch}`. If missing, try `main` then `master`.

   **After collecting results:**

   - If **context-glue itself is behind**: stop immediately and say:
     > "context-glue is out of date ({N} commits behind origin/{default_branch}). This repo contains the instructions I'm running on - continuing without syncing is unreliable."
     Read `context-glue/procedures/GIT_SYNC.md` and follow Step 3 exactly.

   - If **any other repo is behind**: read `context-glue/procedures/GIT_SYNC.md` and follow Steps 4-6 for those repos. Present the sync status as described there.

   - If **all repos are up to date**: note this in the welcome screen (step 4) and do NOT read `GIT_SYNC.md`. Continue immediately.

4. Once the loading order and sync check are complete, output this exact welcome screen in a single message. Fill in sync status lines with actual results from step 3. Then present the menu immediately below - no extra text between the banner and the menu:

```
------------------------------------------------------------
       ___ ___  _ __ | |_ _____  __ |_   _|__ | |_   _  ___
      / __/ _ \| '_ \| __/ _ \ \/ /   | |/ _ \| | | | |/ _ \
     | (_| (_) | | | | ||  __/>  <    | | (_) | | |_| |  __/
      \___\___/|_| |_|\__\___/_/\_\   |_|\___/|_|\__,_|\___|

      ________________________________________________________________
       context-glue - One prompt. Full context. Any stack.
      ________________________________________________________________

       -- Sync complete --

       {repo 1}:   {status}
       {repo 2}:   {status}
       {repo 3}:   {status}

      ________________________________________________________________
       Ready.
------------------------------------------------------------

  What are we doing today?

  1. Ad-hoc - investigate, explore, track findings
  2. New ticket - start a new task or ticket session
  3. Resume ticket - pick up where we left off
  4. Complete ticket - close out a ticket and promote findings
  5. Quick question - no tracking, just answer something
```

5. Based on the user's choice, read the matching prompt file and follow it:

   | Choice | Read |
   |---|---|
   | Ad-hoc | `context-glue/prompts/adhoc.md` |
   | New ticket | `context-glue/prompts/new.md` |
   | Resume ticket | `context-glue/prompts/resume.md` |
   | Complete ticket | `context-glue/prompts/complete.md` |
   | Quick question | Load `context-glue/knowledge/stack/overview.md` only (if not already loaded). Answer the question. No files created, no rooms loaded beyond overview. |
