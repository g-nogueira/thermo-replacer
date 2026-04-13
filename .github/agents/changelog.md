# Agent Customization Changelog

> Versioned record of all changes to agent customization files (`.agent.md`, `.instructions.md`, `SKILL.md`, `copilot-instructions.md`, `AGENTS.md`).
> Each entry captures **what** changed, **why**, **what worked/didn't** from the retro, and **lessons learned** for future sessions.

---

<!-- 
## Entry Template (copy this for each new entry)

### YYYY-MM-DD — [Agent/File Name] — [Short Title]

**Retro trigger:** [What prompted this change — user feedback, observed failure, workflow gap]

**Files modified:**
- `path/to/file.md` — [brief description of change]

**What changed & why:**
| # | Section | Change | Rationale |
|---|---------|--------|-----------|
| 1 | [section] | [what changed] | [why — traced to retro feedback] |

**What was working (kept):**
- [Behaviors/rules preserved because they were effective]

**What wasn't working (fixed):**
- [Specific symptoms and how the change addresses them]

**Lessons learned:**
- [Insights about what makes agents more effective — generalizable takeaways]

**Risks & watch items:**
- [Things to monitor after this change]

---
-->

### 2026-04-26 — All Agents + Skills + Instructions — Full project migration from MonthlyBudget to Genial-T31 Bridge

**Retro trigger:** The entire agent pipeline was inherited from the MonthlyBudget project (C#/.NET modular monolith + SvelteKit frontend). The new project is a Kotlin/Jetpack Compose Android app for BLE thermometer integration. Every customization file contained domain-specific references to .NET, hexagonal architecture, bounded contexts, DDD, EF Core, SvelteKit, and PostgreSQL that no longer apply. User explicitly requested: (1) add HITL to UI Designer, (2) check if frontend/backend agent split is needed, (3) check which skills are necessary, (4) ensure Staff Engineer persona is correct, (5) review changelog for historical context.

**Files modified:**
- `.github/copilot-instructions.md` — Full rewrite: MonthlyBudget/.NET Staff Engineer → Genial-T31/Android/Kotlin Staff Engineer. Updated project description, repo name (`g-nogueira/thermo-replacer`), grounding rules (BLE protocol refs instead of invariants/API contracts), git discipline (`gradlew` instead of `dotnet`/`pnpm`), architectural constraints (BLE coexistence, Android native BLE, MVP focus), architecture reference table, HITL Gates table, all 5 workflows (collapsed frontend/backend split).
- `.github/agents/planner.agent.md` — **Created** (unified from `backend-planner` + `frontend-planner`). Android/Kotlin focused. Analyzes BLE/UI/Data/Worker/Integration layers. HITL gates at Step 4 (present plan) and Step 6 (confirm issue creation).
- `.github/agents/implementor.agent.md` — **Created** (unified from `backend-implementor` + `frontend-implementor`). Kotlin/Android code executor. Gradle build/test/lint. HITL at Step 0 (confirm approach) and Step 4 (confirm before PR).
- `.github/agents/reviewer.agent.md` — **Created** (unified from `backend-reviewer` + `frontend-reviewer`). Additive review model. Review checklist adapted for BLE protocol, Android, Kotlin, Compose. HITL at Step 10 (confirm findings).
- `.github/agents/product-manager.agent.md` — Changed project name, simplified tools list. Workflow preserved.
- `.github/agents/software-architect.agent.md` — Full rewrite: removed hexagonal/DDD/bounded contexts. New architecture overview with 6 Android components (BLE Client, Data Layer, Worker, HA Client, Health Connect, UI). HITL at Phase 3 (confirm plan) and Phase 4 (suggest changes).
- `.github/agents/issue-reader.agent.md` — Repo changed to `thermo-replacer`. Bounded context refs → component-based. Memory template updated. Single handoff to Planner.
- `.github/agents/issue-writer.agent.md` — Labels changed from bounded contexts to components (`ble`, `ui`, `worker`, etc.). Removed Project Board steps. Architecture refs updated.
- `.github/agents/ui-designer.agent.md` — Device type DESKTOP → MOBILE. Added 3 HITL gates: confirm screen plan, confirm between each screen, confirm before handoff.
- `.github/agents/agent-improver.agent.md` — Updated Known Customization File Inventory to reflect new agent/skill/instruction structure.
- `.github/skills/android-dev/SKILL.md` — **Created.** Gradle build/test/lint, ADB, emulator, TDD loop, dependency management.
- `.github/skills/ble-protocol/SKILL.md` — **Created.** Full BLE protocol quick reference: GATT services, packet format, all commands, temperature decoding, connection flow, coexistence rules.
- `.github/skills/additive-review/SKILL.md` — Adapted: replaced .NET checklist items (hexagonal purity, domain invariants, API contracts, EF Core) with Android equivalents (BLE protocol correctness, Android best practices, Kotlin quality, Compose UI). Build commands changed from `dotnet` to `gradlew`. Memory file template updated.
- `.github/skills/resume/SKILL.md` — Adapted: build commands changed from `dotnet`/`pnpm` to `gradlew`. Removed backend/frontend stack detection. Single Implementor handoff.
- `.github/instructions/android.instructions.md` — **Created** with `applyTo: 'app/**/*.kt'`. Kotlin idioms, coroutines, null safety, architecture conventions, BLE code rules, Compose rules, testing, security.
- **Pending user deletion:** 6 old agent files (backend-planner, backend-implementor, backend-reviewer, frontend-planner, frontend-implementor, frontend-reviewer), 2 old instruction files (backend.instructions.md, frontend.instructions.md), 5 context pattern files (budget/forecast/frontend/identity/shared-patterns.md), 4 old skill directories (dotnet-tdd, hexagonal-validation, api-exercise, sveltekit-dev).

**What changed & why:**
| # | Section | Change | Rationale |
|---|---------|--------|-----------|
| 1 | All files | Domain refs: .NET/hexagonal/DDD/bounded contexts → Kotlin/Android/BLE/components | Project changed from C#/.NET monolith to Android BLE app |
| 2 | Agent split | 6 agents (3 backend + 3 frontend) → 3 unified agents (planner/implementor/reviewer) | Single-stack project (Kotlin/Android) doesn't need frontend/backend split |
| 3 | UI Designer | Added 3 HITL gates: screen plan, between screens, before handoff | User explicitly requested HITL for UI Designer; was missing |
| 4 | Skills | Removed 4 (.NET/SvelteKit), created 2 (android-dev, ble-protocol) | Old skills are irrelevant; new skills match project tech stack |
| 5 | Instructions | Removed 2 (.NET/SvelteKit), created 1 (android.instructions.md) | Conditional loading for Kotlin files via `applyTo: 'app/**/*.kt'` |
| 6 | Additive review | Checklist: hexagonal/invariants/API → BLE/Android/Kotlin/Compose | Review criteria must match actual codebase technology |
| 7 | Resume skill | Build commands: dotnet/pnpm → gradlew | Single-stack project |
| 8 | copilot-instructions.md | Staff Engineer persona reoriented for Android/BLE | Global rules must reflect actual project constraints |
| 9 | Issue labels | budget-management/forecast/identity → ble/ui/worker/ha-integration/health-connect/settings | Labels must match actual project components |
| 10 | Architecture | Hexagonal layers/bounded contexts → Android components (BLE/Data/Worker/HA/Health/UI) | Project architecture is fundamentally different |

**What was working (kept):**
- 3-tier memory model (active → distilled → archive) — preserved unchanged
- Activity log conventions — preserved unchanged
- Agent workflow structure (context loading priority, pre-flight checks, grounding rules, execution steps) — preserved across all agents
- Additive review model (baseline, delta, review points, staleness threshold) — core algorithm preserved, only checklist items changed
- Artifact-driven handoffs between agents — preserved
- HITL gates pattern using `vscode/askQuestions` — existing gates preserved, new ones added
- Knowledge graph MCP integration — preserved
- Product Manager state machine (States 1-4) — preserved with minor project name change
- Git discipline conventions (commit format, branch naming, never push during implementation) — preserved with Gradle commands

**What wasn't working (fixed):**
- All agent files referenced .NET types, C# namespaces, hexagonal layers, bounded contexts, EF Core migrations — none of these exist in the new project
- Frontend/backend agent split forced artificial task division for a single-stack Android project
- Old skills (dotnet-tdd, hexagonal-validation, api-exercise, sveltekit-dev) referenced technologies not in the project
- Old instruction files (backend.instructions.md, frontend.instructions.md) contained C#/.NET and SvelteKit rules — irrelevant for Kotlin/Android
- Context pattern files described MonthlyBudget domain patterns (budget, forecast, identity) — completely wrong domain
- UI Designer had no HITL gates between screen generations or before handoff
- Review checklist validated hexagonal purity and domain invariants instead of BLE protocol correctness and Android best practices

**Lessons learned:**
- When migrating an agent pipeline to a new project, the structural patterns (memory model, handoff protocols, HITL gates, review workflows) are highly reusable — the technology-specific content (build commands, architecture rules, review checklists, domain labels) is what changes. Treat the pipeline as a framework with swappable technology modules.
- Collapsing frontend/backend agent splits into unified agents when the project is single-stack dramatically reduces coordination overhead and context switching. The split should be driven by actual technology diversity, not assumed separation.
- BLE protocol details (exact hex values, GATT UUIDs, packet formats) should be captured in a dedicated skill file rather than scattered across agent instructions — agents can load the protocol reference on demand when working on BLE code, keeping other agents' context windows lean.
- HITL gates should exist at every artifact production boundary (screen plan → screen generation → screen batch → handoff). The UI Designer was missing intermediate gates, allowing quality to degrade silently across a batch of screens.
- The `applyTo` glob pattern for conditional instructions (`app/**/*.kt`) is the right mechanism to ensure Kotlin-specific rules only load when editing Kotlin files — this prevents context pollution when working on build scripts, XML resources, or documentation.

**Risks & watch items:**
- Old files (6 agents, 2 instructions, 5 context patterns, 4 skills) must be manually deleted by the user — if left in place, VS Code may still load them and create conflicts with the new unified agents
- The new `android-dev` and `ble-protocol` skills are written from project spec knowledge — they'll need validation against actual implementation once code exists
- Agent Improver's file inventory is now updated, but the mode instructions in `.github/agents/agent-improver.agent.md` still reference "the backend reviewer" as an example — minor but could confuse
- No actual Android code exists yet — all agent adaptations are based on the project spec. Once implementation starts, some assumptions in the review checklist or architecture references may need refinement
- The `android.instructions.md` uses `applyTo: 'app/**/*.kt'` — if the project structure uses a different root (not `app/`), the glob won't match

---

### 2026-04-11 — Issue Creation — Replace dedicated issue-writer with shared github-issues skill

**Retro trigger:** Both the Planner and Issue Writer agents could create GitHub issues, leading to confusion about ownership. In real teams, any team member files issues — having a dedicated "issue writer" agent is artificial. User proposed making issue creation a shared skill with standardized templates that any agent can use.

**Files modified:**
- `.github/skills/github-issues/SKILL.md` — **Created.** Shared issue creation workflow: template selection, enrichment from project docs, duplicate detection, HITL confirmation, label conventions, batch creation support.
- `.github/ISSUE_TEMPLATE/feature.md` — **Created.** Feature template with user story format, acceptance criteria, technical references.
- `.github/ISSUE_TEMPLATE/bug.md` — **Created.** Bug template with repro steps, expected/actual behavior, technical context.
- `.github/ISSUE_TEMPLATE/tech-debt.md` — **Created.** Tech debt template with problem, proposed change, affected components, verification.
- `.github/ISSUE_TEMPLATE/spike.md` — **Created.** Spike template with goal, timebox, research questions, expected output.
- `.github/agents/planner.agent.md` — Added `github-issues` skill reference. Simplified Step 6 to delegate to the skill instead of inline workflow.
- `.github/agents/implementor.agent.md` — Added `github-issues` skill reference for filing follow-up issues during implementation.
- `.github/agents/reviewer.agent.md` — Added `github-issues` skill reference for filing tech debt or bugs found during review.
- `.github/agents/software-architect.agent.md` — Added `github-issues` skill reference. Removed "Hand off to Issue Writer" handoff — architect creates issues directly via skill.
- `.github/copilot-instructions.md` — Updated Workflow 5 to remove Issue Writer, replaced with "creates issues directly (using github-issues skill)".
- `.github/agents/agent-improver.agent.md` — Removed `issue-writer.agent.md` from inventory, added `github-issues/SKILL.md`.
- `.github/agents/issue-writer.agent.md` — **Pending deletion** by user.

**What changed & why:**
| # | Section | Change | Rationale |
|---|---------|--------|-----------|
| 1 | New skill | Created `github-issues` skill with full workflow | Centralizes issue creation logic — any agent can use it |
| 2 | Issue templates | Created 4 templates in `.github/ISSUE_TEMPLATE/` | Standardizes issue format, also powers GitHub web UI |
| 3 | Issue Writer agent | Marked for deletion | Artificial agent — the capability is better as a shared skill |
| 4 | Planner Step 6 | Simplified to reference skill | Was inline workflow; now delegates to shared skill |
| 5 | Implementor | Added skill reference | Engineers should be able to file follow-up issues |
| 6 | Reviewer | Added skill reference | Reviewers should be able to file tech debt / bug issues |
| 7 | Architect | Removed Issue Writer handoff, added skill | Architect creates issues directly, no intermediate agent |
| 8 | Workflow 5 | Updated to show direct issue creation | Removed Issue Writer from the pipeline |

**What was working (kept):**
- Label conventions (ble, ui, worker, ha-integration, health-connect, settings) — preserved in the skill
- HITL gate before issue creation — preserved as Step 5 in the skill
- Duplicate detection — preserved as Step 4 in the skill
- Source traceability (PRD → issue, gap → issue, review → issue) — preserved in the skill
- Issue enrichment from project docs — preserved as Step 3 in the skill

**What wasn't working (fixed):**
- Dedicated Issue Writer agent created artificial separation — in real teams, anyone files issues; the workflow should be a skill, not an agent
- Planner and Issue Writer both had issue creation logic — duplicated and confusing ownership
- Issue format was hardcoded in agent files — now standardized via reusable templates
- Architect had to hand off to Issue Writer for creating issues — unnecessary intermediary removed

**Lessons learned:**
- When a capability (like issue creation) is needed by multiple agents, it should be a **skill**, not a dedicated agent. Skills are composable and loadable on-demand; agents are heavyweight and create unnecessary handoff friction.
- GitHub issue templates in `.github/ISSUE_TEMPLATE/` serve dual purpose: they guide AI agents AND human developers using the GitHub web UI. This alignment between AI tooling and human tooling reduces drift.
- The "any team member can file issues" model from real software teams translates directly to "any agent can use the skill" — organizational patterns from human teams are a good heuristic for agent pipeline design.
- Removing a dedicated agent simplifies the workflow graph but requires careful cleanup: handoffs, workflow docs, HITL tables, and inventory lists all need updating. A checklist helps avoid stale references.

**Risks & watch items:**
- `issue-writer.agent.md` must be manually deleted by user — if left in place, it may still be invocable and create confusion
- The `github-issues` skill references templates via relative paths — if templates are moved, the paths break
- Agents now have `github/*` tools for issue creation — ensure they don't create issues outside of the skill workflow (the HITL gate in the skill should prevent this)
- The Planner's team lead role (batch issue creation from PRD decomposition) is now partially delegated to the skill — monitor if the Planner correctly invokes the skill's batch workflow

---

### 2026-04-11 — All Agents + Global Instructions — Add distill-knowledge skill and agent learnings capture

**Retro trigger:** Knowledge distillation was defined in `copilot-instructions.md` as a vague rule ("distill learnings into knowledge.md AND the MCP knowledge graph") with no procedure, no format, and no agent ownership. Only the Implementor had a Step 6 mentioning it (2 lines). No other agent had any learnings capture. The knowledge graph MCP was configured but unused. User confirmed: hybrid approach (lightweight inline capture + full skill post-merge), skill-based, `knowledge.md` only for MVP.

**Files modified:**
- `.github/skills/distill-knowledge/SKILL.md` — **Created.** Full 7-step procedure: identify issue, read all Tier 1 files, extract learnings sections, dedup against existing knowledge, write categorized atomic facts to `knowledge.md`, archive Tier 1 files, report summary. Defines the `## Learnings` section format agents should use. Includes knowledge.md growth management guidelines.
- `.github/agents/planner.agent.md` — Added Step 7 (Record Learnings) before handoff. Renumbered Step 7→8.
- `.github/agents/implementor.agent.md` — Rewrote Step 6 from vague "note for knowledge.md" to structured learnings capture with Decisions/Patterns/Gotchas format. Added note explaining this is inline capture, full compression happens via skill.
- `.github/agents/reviewer.agent.md` — Added learnings capture (Patterns/Gotchas/Review insights) to Step 11 (Write Review Memory & Post).
- `.github/agents/software-architect.agent.md` — Added "Record Learnings" section before Critical Rules.
- `.github/agents/ui-designer.agent.md` — Added Step 9 (Record Learnings) before handoff. Renumbered Step 9→10.
- `.github/agents/product-manager.agent.md` — Added "Record Learnings" section before Handoff Chain.
- `.github/copilot-instructions.md` — Updated Tier 2 definition: removed MCP knowledge graph dual-write requirement, pointed to `distill-knowledge` skill, explained the agent `## Learnings` convention. Updated Agent Conventions to reference the skill.
- `.github/agents/agent-improver.agent.md` — Added `distill-knowledge/SKILL.md` to skill inventory.

**What changed & why:**
| # | Section | Change | Rationale |
|---|---------|--------|-----------|
| 1 | New skill | Created `distill-knowledge` with full procedure | Vague "distill" rule had no actionable procedure — skill defines the how |
| 2 | All agents | Added learnings capture step with structured format | Agents weren't recording learnings — skill needs raw material to compress |
| 3 | Tier 2 definition | Removed MCP graph dual-write, pointed to skill | Graph untested, adds complexity — `knowledge.md` only for MVP |
| 4 | Agent conventions | Added `## Learnings` convention and skill reference | Makes the expectation explicit across all agents |

**What was working (kept):**
- 3-tier memory architecture (active/distilled/archive) — preserved, skill formalizes the tier transitions
- MCP knowledge graph configuration in `mcp.json` — kept for future use, just not required for MVP
- Activity log conventions — unchanged
- Tier 1 and Tier 3 definitions — unchanged

**What wasn't working (fixed):**
- "Distill learnings" was a vague instruction with no procedure — now defined in a 7-step skill
- Only the Implementor had any learnings step (2 lines, unstructured) — now all 6 workflow agents have structured capture
- Tier 2 required dual-write to `knowledge.md` AND knowledge graph — simplified to markdown only for MVP
- No defined format for what a "learning" looks like — skill defines categories (Architecture Decisions, BLE Protocol, Codebase Conventions, etc.) and atomic fact format
- No dedup mechanism — skill includes Step 4 for checking existing knowledge before writing

**Lessons learned:**
- Vague instructions like "distill learnings" are effectively ignored by AI agents — they need a concrete procedure with specific steps, formats, and examples. "What" without "how" is useless.
- The hybrid model (lightweight inline + full skill post-merge) mirrors how human engineers work: jot quick notes during work, write the postmortem later with full context. AI agents benefit from the same two-phase pattern.
- Starting with `knowledge.md` only (no graph) follows YAGNI — the knowledge graph can be added when there's enough structured data to make querying valuable. Premature graph writes add complexity without proven benefit.
- Every agent needs the same `## Learnings` section format so the distillation skill can parse them consistently. Standardizing the schema across agents is critical for downstream automation.

**Risks & watch items:**
- Agents may write shallow or empty learnings sections ("None.") to satisfy the step without genuine reflection — monitor quality of early entries
- `knowledge.md` will grow over time — the skill includes a "too large" section but this needs human judgment to trigger
- MCP knowledge graph is now disconnected from the distillation flow — if graph queries become valuable later, the skill needs a Step 5b for graph writes
- The distill-knowledge skill is user-invocable but not auto-triggered — user must remember to invoke it after issue closure

---

### 2026-04-11 — Issue Reader Agent → task-context Skill — Replace dedicated agent with universal context-gathering skill

**Retro trigger:** The Issue Reader was a dedicated agent whose sole job was to fetch GitHub issue context and write a memory file for the Planner. This mirrors the earlier Issue Writer pattern — an artificial agent for a capability that any team member should have. In real teams, any engineer gathers task context before starting work. Additionally, downstream agents (Implementor, Reviewer) had no way to verify context completeness — they blindly trusted what was in the memory file. User also noted that agents writing issues should ensure they provide complete enough information to reduce future context-gathering work.

**Files modified:**
- `.github/skills/task-context/SKILL.md` — **Created.** Two-mode skill: Mode 1 (Primary Gather) fetches full issue context from GitHub + enriches from docs + writes structured memory file. Mode 2 (Completeness Check) verifies existing context is sufficient and fills gaps. Includes completeness checklist (8 items) and context quality section for issue writing. Available to all agents with primary (Planner) and secondary (others) roles.
- `.github/agents/planner.agent.md` — Added `task-context` skill reference. Pre-flight now invokes Mode 1 (if no memory file) or Mode 2 (if exists). Step 1 renamed from "Read Memory" to "Gather Context". Context loading priority updated. Input section updated.
- `.github/agents/implementor.agent.md` — Added `task-context` skill reference. Pre-flight now runs Mode 2 completeness check before coding.
- `.github/agents/reviewer.agent.md` — Added `task-context` skill reference. Pre-flight now runs Mode 2 completeness check. Context loading priority updated from `issue-reader-*` to `task-context-*`.
- `.github/skills/github-issues/SKILL.md` — Added Step 3b (Verify Context Quality) between enrichment and duplicate search. Cross-references the `task-context` skill's quality bar for issue writing.
- `.github/copilot-instructions.md` — Workflow 2 updated: `Planner (gathers context via task-context skill) → Implementor → Reviewer`. Tier 1 naming example updated from `issue-reader-5.md` to `task-context-5.md`.
- `.github/agents/agent-improver.agent.md` — Removed `issue-reader.agent.md` from inventory, added `task-context/SKILL.md`.
- `.github/skills/distill-knowledge/SKILL.md` — Updated memory file table: `issue-reader-<N>.md` → `task-context-<N>.md`.
- `.github/skills/resume/SKILL.md` — Updated memory file path: `issue-reader-*` → `task-context-*`.
- `.github/agents/issue-reader.agent.md` — **Pending deletion** by user.

**What changed & why:**
| # | Section | Change | Rationale |
|---|---------|--------|-----------|
| 1 | New skill | Created `task-context` with Primary Gather + Completeness Check modes | In real teams, gathering task context is a universal skill, not a separate role |
| 2 | Issue Reader agent | Marked for deletion | Artificial agent — the capability belongs as a skill |
| 3 | Planner pre-flight | Now invokes task-context skill (Mode 1 or 2) | Planner is the primary context gatherer in Workflow 2 |
| 4 | Implementor/Reviewer pre-flight | Added Mode 2 completeness check | Downstream agents should verify context, not blindly trust |
| 5 | github-issues Step 3b | Added context quality verification | Issue writers must ensure issues are complete enough for future context gathering |
| 6 | Workflow 2 | Removed Issue Reader from pipeline | One fewer handoff step, Planner starts directly |
| 7 | Memory file naming | `issue-reader-*` → `task-context-*` | Reflects the new skill name |

**What was working (kept):**
- Structured memory file template (verbatim ACs, affected components, BLE refs, codebase snapshot) — preserved exactly in the skill
- Verbatim copy rule for acceptance criteria — preserved
- Project doc enrichment steps (PROJECT_SPEC.md, FINDINGS.md, architecture docs) — preserved
- Activity log cross-team event logging — preserved in the skill

**What wasn't working (fixed):**
- Dedicated Issue Reader agent was an artificial role — real engineers gather their own task context; now it's a skill any agent can use
- Issue Reader → Planner handoff added unnecessary latency — now Planner gathers context directly
- Implementor and Reviewer blindly trusted the memory file without verifying completeness — now they run a completeness check
- Agents creating issues had no quality bar for context completeness — now github-issues skill cross-references the task-context skill's quality checklist
- No mechanism to fill context gaps after initial gathering — Mode 2 (Completeness Check) appends missing information without overwriting

**Lessons learned:**
- The pattern of replacing dedicated agents with shared skills (Issue Writer → github-issues, Issue Reader → task-context) consistently improves the pipeline. A good heuristic: if an agent's sole job is a single capability that multiple agents need, it should be a skill.
- Context gathering has two distinct phases: initial fetch (Primary Gather) and verification (Completeness Check). Modeling these as two modes of the same skill keeps them colocated while serving different consumers.
- Making every downstream agent verify context completeness (not just the first agent in the chain) prevents information gaps from compounding. This mirrors how real engineers re-read the ticket before starting work.
- Issue quality and context gathering are two sides of the same coin — when agents write better issues, future agents spend less time gathering context. The cross-reference between github-issues and task-context skills encodes this relationship.
- The "primary gatherer vs. secondary consumer" role distinction avoids redundant full-fetches while keeping the skill universally accessible. This matches how real teams work: the lead does the deep context gathering, team members verify they have enough.

**Risks & watch items:**
- `issue-reader.agent.md` must be manually deleted — if left in place, it may still be invocable and create confusion
- The completeness checklist adds pre-flight overhead for Implementor and Reviewer — monitor if it becomes a bottleneck or if agents skip it
- Mode 2 appends to existing memory files (`## Additional Context`) — these files could grow large over multiple rounds; monitor size
- The task-context skill is available to all agents but most effective when the Planner is the primary gatherer — if multiple agents independently run Mode 1 for the same issue, they'd overwrite each other's memory file

---

### 2026-04-12 — Reviewer + Implementor + Global Instructions — PR feedback flow, sub-agents, and tool UX fixes

**Retro trigger:** Four user-reported issues: (1) Reviewer kept trying REQUEST_CHANGES on own PR and getting API errors — should know this is a permanent setup constraint. (2) PR conversation addressing should be owned by the Implementor (developer), not routed through the Planner — matches how real dev teams work. (3) Agents couldn't consult each other mid-task because all had `disable-model-invocation: true` — needed sub-agent capability for quick consultations. (4) `vscode/askQuestions` tool truncates long question text, making HITL unusable when agents cram too much detail into it.

**Files modified:**
- `.github/agents/reviewer.agent.md` — Added "Always use COMMENT event" as top Critical Rule with full explanation of the setup constraint. Updated Step 10 HITL gate to reference COMMENT-only. Updated Step 11 handoff to point to Implementor instead of Planner. Removed `disable-model-invocation: true`. Added `agent` tool and `agents: ['Planner']` for sub-agent invocation. Added Sub-agent Invocation section with when-to/when-not-to guidelines.
- `.github/agents/implementor.agent.md` — Removed `disable-model-invocation: true`. Added `agent` tool and `agents: ['Planner', 'Reviewer']` for sub-agent invocation. Added `address-pr-feedback` skill reference. Added Sub-agent Invocation section with escalation criteria, anti-chattiness rules, and HITL inheritance note.
- `.github/agents/planner.agent.md` — Removed `disable-model-invocation: true` so it can be invoked as a sub-agent by Implementor and Reviewer.
- `.github/skills/address-pr-feedback/SKILL.md` — **Created.** 10-step workflow for addressing PR review feedback: load context, classify review points (direct-fix vs. escalate-to-planner), HITL confirmation, fix code, escalate where needed, full verification, reply to GitHub threads, commit/push, update memory, request re-review. Includes escalation signals checklist (5 questions), GitHub thread reply conventions, and critical rules.
- `.github/copilot-instructions.md` — Updated Workflow 3 to route Reviewer→Implementor (was Reviewer→Planner→Implementor). Added HITL table entry for Implementor PR feedback gate. Added "Sub-agent Invocation Convention" section with allowed pairs table, anti-chattiness rule, and HITL inheritance note. Added "`vscode/askQuestions` Tool — Long Content Rule" section with the write-to-chat-then-ask pattern.
- `.github/agents/agent-improver.agent.md` — Added `address-pr-feedback/SKILL.md` to skill inventory.

**What changed & why:**
| # | Section | Change | Rationale |
|---|---------|--------|-----------|
| 1 | Reviewer Critical Rules | Added "Always use COMMENT event" with explanation of setup constraint | Reviewer repeatedly hit GitHub API error — the constraint is permanent (same account), not transient |
| 2 | Reviewer Step 10 | Removed "request changes" from recommended actions | Aligns with COMMENT-only rule |
| 3 | Reviewer Step 11 | Handoff target: Planner → Implementor | The developer should address feedback directly, consulting Planner only when needed |
| 4 | New skill | Created `address-pr-feedback` skill | Implementor needs a structured workflow for addressing review comments with escalation criteria |
| 5 | Agent frontmatter | Removed `disable-model-invocation: true` from Planner, Reviewer | Enables them to be invoked as sub-agents by other agents |
| 6 | Agent frontmatter | Added `agent` tool + `agents` whitelist to Implementor, Reviewer | Enables controlled sub-agent invocation (specific pairs only) |
| 7 | Sub-agent guidelines | Added to Implementor, Reviewer, and global instructions | Agents need clear rules about when to use sub-agents vs. handoffs vs. just doing it themselves |
| 8 | Workflow 3 | `Reviewer → Implementor` (was `Reviewer → Planner → Implementor`) | Removes unnecessary Planner intermediary for straightforward PR fixes |
| 9 | HITL table | Added Implementor PR feedback gate; added COMMENT note on Reviewer | Reflects new skill's HITL gate and COMMENT constraint |
| 10 | Global askQuestions rule | Added long-content workaround | Tool truncates long text; writing to chat first solves the visibility problem |

**What was working (kept):**
- Additive review model (baseline, delta, review points) — preserved, skill integrates with it
- 3-tier memory architecture — preserved
- Pre-flight checks and self-verification checkpoints — preserved
- Handoff buttons for full context switches — preserved alongside new sub-agent capability
- HITL gates at all decision points — preserved, added new ones
- Git discipline conventions — preserved
- Skill-based capability pattern — extended with new skill
- All existing Critical Rules on reviewer — preserved, new rule added at top

**What wasn't working (fixed):**
- Reviewer repeatedly got "can't request changes on own PR" API error — now hardcoded to always use COMMENT event with clear explanation of why
- PR feedback addressing required unnecessary Planner intermediary — now Implementor handles directly with Planner available as sub-agent for design questions
- All agents had `disable-model-invocation: true` preventing any inter-agent consultation — now Planner and Reviewer can be invoked as sub-agents by specific agents
- `vscode/askQuestions` tool questions were too long to read — now agents write full context to chat first and use the tool for a short summary question only

**Lessons learned:**
- **Permanent setup constraints should be elevated to Critical Rules, not left as runtime discoveries.** When an agent encounters the same error repeatedly (like "can't REQUEST_CHANGES on own PR"), the constraint should be baked into its instructions so it never attempts the failing action. This is the "remember when you fail" pattern — errors that stem from environment constraints (not code bugs) should become permanent rules.
- **Sub-agent invocation requires a three-part framework: (1) who can invoke whom, (2) when to invoke vs. handle yourself, (3) what the sub-agent should do vs. what the parent continues.** Without all three, agents either never use sub-agents or over-use them. The `agents` whitelist handles (1), the anti-chattiness rules handle (2), and the escalation criteria in each agent handle (3).
- **The "developer addresses reviewer feedback" pattern from real teams maps cleanly to "Implementor addresses PR feedback with Planner sub-agent for escalation."** The Planner-as-intermediary model was an over-formalization that didn't match how real engineering teams work — developers fix their own code.
- **Tool limitations should be addressed with explicit workaround conventions, not vague guidance.** The `vscode/askQuestions` truncation issue was causing real HITL failures — the "write to chat first, then ask short question" pattern is a concrete, testable convention that agents can follow consistently.
- **The `agents` frontmatter property + `agent` tool is the idiomatic VS Code mechanism for sub-agent invocation.** It provides context isolation (each sub-agent gets a clean context window), controlled access (whitelist prevents unintended invocations), and visibility (sub-agent calls appear as collapsible tool calls in the chat).

**Risks & watch items:**
- The `agents` property and sub-agent invocation are marked **experimental** in VS Code docs (as of April 2026) — API could change in future updates
- Agents might **over-invoke sub-agents** instead of making decisions independently — monitor for chattiness and add stricter guidelines if needed
- The `address-pr-feedback` skill's **escalation checklist** (5 questions) needs validation against real PR review rounds — some criteria may be too sensitive or not sensitive enough
- Removing `disable-model-invocation: true` from Planner means **any agent with the `agent` tool could theoretically invoke it** — the `agents` whitelist on the calling agents is the control, but if a new agent is added without a whitelist, it could invoke Planner unintentionally
- The **long-content workaround** relies on agent judgment about "too long" — different models may interpret the ~500-character threshold differently; monitor if agents are consistent
- The Reviewer's handoff now goes to **Implementor instead of Planner** — if the Implementor is not available or the user prefers the Planner to plan fixes first, the workflow may need an alternative path
