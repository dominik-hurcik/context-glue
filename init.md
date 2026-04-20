# context-glue — Session Init

1. Read `context-glue/START_HERE.md` — follow the full instruction loading order before continuing.

2. Check `context-glue/settings.json` for `workspace.setup_complete`.
   - If `false` or the field is missing, stop immediately and say:
     > "context-glue hasn't been set up yet for this workspace. Run the setup wizard first:
     > `Read context-glue/prompts/setup.md`"
   - If `true`, continue.

3. Read `context-glue/procedures/GIT_SYNC.md` and run the sync check for all repositories listed in `settings.json` under `workspace.repos`.

   Present the sync status and handle per-repo sync as described in the procedure.

4. Once the loading order and sync check are complete, output this exact welcome screen in a single message. Fill in the repo sync status lines with actual results from step 3 (e.g. `up to date`, `synced`, `skipped (user choice)`, `not found`). Then present the menu immediately below — no extra text between the banner and the menu:

```
------------------------------------------------------------
       ___ ___  _ __ | |_ _____  __ |_   _|__ | |_   _  ___
      / __/ _ \| '_ \| __/ _ \ \/ /   | |/ _ \| | | | |/ _ \
     | (_| (_) | | | | ||  __/>  <    | | (_) | | |_| |  __/
      \___\___/|_| |_|\__\___/_/\_\   |_|\___/|_|\__,_|\___|

      ________________________________________________________________
       context-glue — One prompt. Full context. Any stack.
      ________________________________________________________________

       -- Sync complete --

       {repo 1}:   {status}
       {repo 2}:   {status}
       {repo 3}:   {status}

      ________________________________________________________________
       Ready.
------------------------------------------------------------

  What are we doing today?

  1. Ad-hoc — investigate, explore, track findings
  2. New ticket — start a new task or ticket session
  3. Resume ticket — pick up where we left off
  4. Complete ticket — close out a ticket and promote findings
```

5. Based on the user's choice, read the matching prompt file and follow it:

   | Choice | Read |
   |---|---|
   | Ad-hoc | `context-glue/prompts/adhoc.md` |
   | New ticket | `context-glue/prompts/new.md` |
   | Resume ticket | `context-glue/prompts/resume.md` |
   | Complete ticket | `context-glue/prompts/complete.md` |
