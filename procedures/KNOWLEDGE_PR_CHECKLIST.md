# KNOWLEDGE_PR_CHECKLIST.md — Knowledge PR Review

Use this checklist when reviewing a pull request that modifies files in `knowledge/`.

Both the agent (during "Complete ticket") and human reviewers should run through this before approving.

---

## For the agent — run before telling the user to push

For every knowledge file modified in this session, check each of the following. Report any flags before instructing the user to push.

### 1. Contradiction check
Read the target file in full. Does the new content contradict any existing statements?
- If yes: flag it. Do not write conflicting information — either reconcile the contradiction or ask the user which version is correct.

### 2. Duplication check
Is this finding already captured somewhere in `knowledge/`? Check the `INDEX.md` room descriptions.
- If it's a near-duplicate: flag it. Suggest merging or linking rather than duplicating.

### 3. Scope check
Is this finding in the right room? Cross-check the room's purpose in `INDEX.md`.
- If it belongs in a different room: say so and move it before writing.

### 4. Size check
After the addition, is the target file still a reasonable size (under ~300 lines)?
- If not: flag it as a candidate for splitting. Suggest where the content could be moved.

### 5. Last updated
Did you update the `## Last updated:` header on every modified file?
- If not: update it now.

---

## For human reviewers — run before approving the PR

When a knowledge PR arrives, check:

- [ ] **No contradictions** — new content doesn't conflict with existing content in the same file
- [ ] **No duplication** — same finding doesn't already exist in another room
- [ ] **Right room** — the content belongs in the file it was added to (matches the room's purpose in `INDEX.md`)
- [ ] **Reasonable size** — file isn't getting bloated; if so, flag for splitting
- [ ] **Last updated** — header is current on every modified file
- [ ] **No credentials or personal data** — `env.local` or any personal info was not accidentally included

If all checks pass, approve. If any fail, request changes with a note on what to fix.

---

## What belongs in knowledge/ vs what doesn't

**Promote to knowledge/:**
- Patterns and conventions discovered through real work
- Gotchas, edge cases, and hard-won lessons
- Cross-system contracts and failure modes
- Operational knowledge that would save time for the next person

**Do NOT promote:**
- Decisions specific to a single ticket with no recurring pattern
- Step-by-step debugging paths for a one-off bug
- Temporary workarounds that will be removed soon
- Anything already present in the target file
