# Contributing to context-glue

Thanks for wanting to improve this. Contributions of all sizes are welcome.

---

## What counts as a contribution

- Fixing unclear or incorrect instructions in any `.md` file
- Improving the setup wizard to handle edge cases better
- Adding examples or coverage for stack types that aren't well represented
- Improving the investigation protocol, prompts, or procedures
- Fixing typos or broken formatting

---

## What this repo is (and isn't)

context-glue is a set of plain text files that an AI agent reads and follows. There is no code to compile, no dependencies to install, and no tests to run. A contribution is a change to markdown or JSON that makes the tool work better for more people.

Keep that in mind when scoping a contribution - the goal is always "does an AI agent reading this file behave better as a result?"

---

## How to contribute

1. Fork the repo
2. Create a branch with a short descriptive name (e.g. `fix/investigate-examples`, `feat/setup-docker-credentials`)
3. Make your changes
4. Open a pull request with a clear title and a one or two sentence description of what changed and why

No issue required before opening a PR for small fixes. For larger changes (reworking a full prompt, adding a new procedure, changing the knowledge palace structure) it is worth opening an issue first to discuss the direction.

---

## Guidelines

**Keep files agent-readable.** Instructions should be unambiguous. If a step could be interpreted two ways by an AI agent, rewrite it until it can't.

**Stay stack-agnostic.** The generic files (`investigate.md`, `AGENT.md`, `START_HERE.md`, the prompt files) should work for any team on any stack. Do not bake in tool-specific conventions. Stack-specific knowledge belongs in the files that `setup.md` generates for each user, not in the shared files.

**One change per PR.** Easier to review, easier to revert if needed.

**Update `## Last updated:` headers.** Any `.md` file you touch that has a `## Last updated:` header at the top should have that date updated to today.

---

## Questions

Open an issue and ask. No question is too small.
