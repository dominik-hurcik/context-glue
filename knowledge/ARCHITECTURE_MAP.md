## Last updated: —
# Architecture Map

> **Template — fill this in as you work.** This file grows over time as the team discovers cross-system contracts, failure modes, and integration details. The agent adds to it when `[architecture]` rooms are loaded and cross-system findings are promoted from completed tickets.

This document maps the cross-system dependencies in your workspace.

---

## Verification legend

| Label | Meaning |
|---|---|
| Verified | Directly confirmed by code, config, or documentation in this workspace |
| Assumption | Strong inference, but final value lives outside the repo |
| Uncertain | Could not be confirmed from available sources |

---

## System overview

*Describe each major system and its role. One row per system.*

| System | Repo | Role |
|---|---|---|
| *(add systems here)* | | |

---

## End-to-end flow

*Map how data, requests, or events flow between systems.*

```
{System A}
    ↓  {protocol or method}
{System B}
    ↓  {protocol or method}
{System C}
```

---

## Contract map

*Capture explicit and implicit contracts between components — things that break if either side changes without coordination.*

| Producer | Consumer | Contract type | Artifact | Status | Risk if changed |
|---|---|---|---|---|---|
| *(add contracts here)* | | | | | |

---

## Known fragile points

*Integration points that are convention-based, undocumented, or known to be brittle.*

---

## Integration points

*HTTP endpoints, queue topics, file paths, shared tables, or any other explicit integration surface.*

| Integration point | Producer side | Consumer side | Status |
|---|---|---|---|
| *(add here)* | | | |

---

## Risks and unknowns

*Things that could break silently, or that depend on runtime state outside the repo.*

---

## Glossary

*Terms, table names, endpoint paths, topic names, and identifiers that appear across multiple systems.*

| Term | Meaning |
|---|---|
| *(add here)* | |
