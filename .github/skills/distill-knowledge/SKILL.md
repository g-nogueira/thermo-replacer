---
name: distill-knowledge
description: "Compress completed issue memory files into reusable knowledge. Reads all Tier 1 (active) memory files for an issue, extracts decisions, patterns, conventions, and gotchas, writes atomic facts to knowledge.md, and archives raw files to Tier 3. Use after an issue is merged/closed or when the user asks to distill learnings."
user-invocable: true
---

# Distill Knowledge Skill

This skill compresses verbose Tier 1 memory files into reusable Tier 2 knowledge after an issue is completed. It also archives the raw files to Tier 3.

**Trigger:** Invoke after an issue is merged/closed, or when the user explicitly asks to distill learnings. Can also be invoked periodically to clean up accumulated active memory files.

## Step 1: Identify the Issue

If the issue number isn't provided, check:
```powershell
# List active memory files
Get-ChildItem .github/agents/memory/active/ -Name
```

Ask the user which issue to distill if multiple are present.

## Step 2: Read All Tier 1 Memory Files

Read every memory file for the issue:

| File | Agent | Key content to extract |
|---|---|---|
| `task-context-<N>.md` | Task Context Skill | Issue scope, acceptance criteria, component mapping |
| `plan-<N>.md` | Planner | Implementation approach, file structure decisions, open questions resolved |
| `implementation-<N>.md` | Implementor | Patterns discovered, conventions learned, gotchas encountered |
| `code-reviewer-<N>.md` | Reviewer | Review findings, common mistakes, quality patterns |

Not all files will exist for every issue. Read what's available.

## Step 3: Read the Learnings Sections

Each agent appends a `## Learnings` section to its memory file during work. These are your primary extraction source. Look for:

- **Decisions made** — choices between alternatives, with rationale
- **Patterns discovered** — codebase conventions, naming patterns, file organization
- **Conventions confirmed** — things that worked and should be repeated
- **Gotchas encountered** — surprising behavior, workarounds, things that didn't work
- **BLE protocol insights** — any new protocol understanding beyond `Docs/FINDINGS.md`
- **Tooling/build insights** — Gradle tricks, ADB commands, test patterns that worked

## Step 4: Deduplicate Against Existing Knowledge

Read `.github/agents/memory/knowledge.md` and check if any extracted facts already exist. Skip duplicates. If an existing entry needs updating (new nuance discovered), update it rather than adding a duplicate.

## Step 5: Write to knowledge.md

Append new facts to `.github/agents/memory/knowledge.md` using this format:

```markdown
## <Category>

### <Date> — <Short title> (from #<issue>)
- <Atomic fact 1>
- <Atomic fact 2>
```

**Categories** (use existing ones, create new only if needed):

| Category | What goes here |
|---|---|
| `Architecture Decisions` | Component design choices, technology selections, trade-offs |
| `BLE Protocol` | Protocol insights, timing observations, edge cases |
| `Codebase Conventions` | Naming patterns, file organization, package structure |
| `Build & Tooling` | Gradle configs, ADB tricks, test patterns |
| `Android Patterns` | Compose patterns, WorkManager usage, DataStore patterns |
| `Review Findings` | Common mistakes, quality patterns, things reviewers should watch for |
| `Gotchas` | Surprising behavior, workarounds, things that look right but aren't |

**Rules for writing facts:**
- **Atomic:** One fact per bullet point — no compound sentences
- **Actionable:** Write as instructions, not observations ("Use X" not "We found that X")
- **Source-linked:** Include issue number for traceability
- **No narratives:** Delete context, keep only the reusable insight

**Example:**
```markdown
## BLE Protocol

### 2026-05-10 — Temperature polling timing (from #12)
- Poll command (0x01) must wait ≥4s between sends — faster polling causes the device to stop responding
- If no response within 3s of a poll, the device has likely disconnected — do not retry, close GATT

## Codebase Conventions

### 2026-05-10 — Worker naming (from #12)
- WorkManager workers use `<Purpose>Worker.kt` naming (e.g., `TemperatureWorker.kt`)
- Worker tags use the format `genial_<purpose>_work` for unique work identification
```

## Step 6: Archive Tier 1 Files

Move all memory files for the issue from `active/` to `archive/`:

```powershell
Move-Item .github/agents/memory/active/*-<issue-number>.md .github/agents/memory/archive/
```

## Step 7: Report

Present a summary to the user:

```markdown
## Distillation Summary — Issue #<N>

**Files processed:** <count>
**Facts extracted:** <count>
**Facts deduplicated (skipped):** <count>
**Facts written to knowledge.md:** <count>
**Files archived:** <list>
```

## Lightweight Inline Distillation (For Working Agents)

Working agents don't run this full skill. Instead, they append a `## Learnings` section to their own memory file at the end of their workflow. This section is the raw material this skill later compresses.

**Format agents should use in their memory files:**

```markdown
## Learnings

### Decisions
- <decision>: <rationale>

### Patterns
- <pattern observed>

### Gotchas
- <thing that was surprising or didn't work as expected>
```

Agents should write facts here as they work — not batch them at the end. If no learnings were generated, write `## Learnings\nNone.` to confirm the section was considered.

## When knowledge.md Gets Too Large

If `knowledge.md` exceeds ~200 lines, consider:
1. Consolidating related entries under fewer headings
2. Removing entries that are now obvious from the codebase itself (e.g., naming conventions that are self-evident from existing files)
3. Moving rarely-referenced categories to a separate file (e.g., `knowledge-ble.md`)
