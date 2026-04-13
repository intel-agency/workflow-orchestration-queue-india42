# Workflow Execution Plan: project-setup

**Generated:** 2026-04-13
**Workflow:** project-setup (dynamic workflow)
**Repository:** intel-agency/workflow-orchestration-queue-india42
**Working Branch:** dynamic-workflow-project-setup
**Trigger:** `workflow_run` event — Pre-build devcontainer image completed successfully on `main`

---

## 1. Overview

The `project-setup` dynamic workflow initializes the `workflow-orchestration-queue` repository for active development. It transforms the seeded plan documents and reference code into a structured project ready for autonomous agent-driven implementation.

| Metric | Value |
|--------|-------|
| Total Main Assignments | 6 |
| Pre-Script Events | 1 (`create-workflow-plan`) |
| Post-Assignment Events (per assignment) | 2 (`validate-assignment-completion`, `report-progress`) |
| Post-Script Events | 1 (apply `orchestration:plan-approved` label) |
| Estimated Event Handlers | 1 + (6 × 2) + 1 = 14 event handler invocations |
| Total Steps | 6 main + 14 event + 1 plan = **21 orchestration steps** |

---

## 2. Project Context Summary

### 2.1 Application Description

**workflow-orchestration-queue** (OS-APOW) is a headless agentic orchestration platform that transforms GitHub Issues into "Execution Orders" autonomously fulfilled by specialized AI agents. The system replaces manual human-in-the-loop AI coding with a persistent, event-driven infrastructure that runs as a background production service.

### 2.2 Architecture: Four Pillars

| Pillar | Component | Technology | Role |
|--------|-----------|------------|------|
| **The Ear** | Work Event Notifier | Python 3.12, FastAPI, Pydantic | Webhook receiver — secure ingestion, HMAC verification, event triage |
| **The State** | Work Queue | GitHub Issues + Labels | "Markdown as a Database" — distributed state via `agent:*` labels |
| **The Brain** | Sentinel Orchestrator | Python async service, httpx | Polling, task claiming (assign-then-verify), worker lifecycle management |
| **The Hands** | Opencode Worker | opencode CLI, LLM, DevContainer | Isolated execution environment for AI-driven code generation |

### 2.3 Tech Stack

| Category | Technology |
|----------|------------|
| Primary Language | Python 3.12+ |
| Web Framework | FastAPI + Uvicorn |
| Data Validation | Pydantic |
| Async HTTP Client | httpx |
| Package Manager | uv (Rust-based) |
| Containerization | Docker + DevContainers |
| Shell Bridge | bash / PowerShell Core (pwsh) |
| AI Runtime | opencode CLI + GLM-5 |
| CI/CD | GitHub Actions (SHA-pinned) |
| State Management | GitHub Issues, Labels, Milestones |

### 2.4 Repository Details

- **Owner:** `intel-agency`
- **Repo:** `workflow-orchestration-queue-india42`
- **Template:** Cloned from `intel-agency/workflow-orchestration-queue-india42`
- **Branch Strategy:** `main` (stable), `develop` (integration)
- **Pre-built Devcontainer:** `ghcr.io/intel-agency/workflow-orchestration-queue-india42/devcontainer`

### 2.5 Key Architectural Decisions (from Plan Docs)

1. **Shell-Bridge Execution (ADR 07):** Orchestrator interacts with worker exclusively via `./scripts/devcontainer-opencode.sh` — no Docker SDK in Python.
2. **Polling-First Resiliency (ADR 08):** Webhooks are optimization; polling is the primary discovery mechanism. System self-heals on restart.
3. **Provider-Agnostic Interface (ADR 09):** `ITaskQueue` ABC enables future provider swapping (Linear, Jira, etc.).
4. **Assign-then-Verify Locking:** Distributed locking via GitHub Assignees prevents race conditions between multiple Sentinel instances.
5. **Credential Scrubbing:** All public-facing output is sanitized via `scrub_secrets()` in `src/models/work_item.py`.

### 2.6 Applied Simplifications (from Simplification Report)

| ID | Change | Status |
|----|--------|--------|
| S-3 | Reduced to 3 env vars (`GITHUB_TOKEN`, `GITHUB_ORG`, `SENTINEL_BOT_LOGIN`) | IMPLEMENTED |
| S-4 | Hardcoded `"stop"` env reset mode | IMPLEMENTED |
| S-5 | Single-repo polling only (cross-repo deferred) | IMPLEMENTED |
| S-6 | Consolidated queue into `src/queue/github_queue.py` | IMPLEMENTED |
| S-7 | Removed IPv4 scrubbing pattern | IMPLEMENTED |
| S-8 | Removed "encrypted" log prose | IMPLEMENTED |
| S-9 | Moved Phase 3 to appendix | IMPLEMENTED |
| S-10 | Stdout-only logging (no FileHandler) | IMPLEMENTED |
| S-11 | Removed `raw_payload` field from WorkItem | IMPLEMENTED |
| S-1 | Kept `ITaskQueue` ABC for future provider swapping | KEPT |
| S-2 | Kept doc duplication for agent reinforcement | KEPT |

### 2.7 Plan Documents Inventory

| File | Description |
|------|-------------|
| `OS-APOW Architecture Guide v3.2.md` | System architecture, ADRs, data flow, security model |
| `OS-APOW Development Plan v4.2.md` | Phased roadmap (Phase 0–3), user stories, risk assessment, cross-cutting directions |
| `OS-APOW Implementation Specification v1.2.md` | Detailed features, test cases, acceptance criteria, project structure |
| `OS-APOW Plan Review.md` | Code review of reference implementations (I-1 through I-10, R-1 through R-9) |
| `OS-APOW Simplification Report v1.md` | Simplification decisions with user feedback |
| `orchestrator_sentinel.py` | Reference Sentinel implementation (292 lines) |
| `notifier_service.py` | Reference Notifier implementation (110 lines) |
| `src/models/work_item.py` | Unified WorkItem, TaskType, WorkItemStatus, scrub_secrets() (74 lines) |
| `src/queue/github_queue.py` | ITaskQueue ABC + GitHubQueue with connection pooling (249 lines) |

---

## 3. Assignment Execution Plan

### 3.1 Pre-Script Event: `create-workflow-plan`

| Field | Value |
|-------|-------|
| **Short ID** | `create-workflow-plan` |
| **Goal** | Produce this comprehensive workflow execution plan before any other assignment begins |
| **Type** | Pre-script event (runs first) |
| **Output** | `plan_docs/workflow-plan.md` committed to `dynamic-workflow-project-setup` branch |
| **Acceptance Criteria** | Plan document exists, covers all 6 assignments, branch pushed to remote |
| **Prerequisites** | Read all `plan_docs/` files; create working branch from `main` |
| **Dependencies** | None (first step) |
| **Risks** | Plan docs may be incomplete or ambiguous; mitigated by reading all files |
| **Output Record** | `#events.pre-script-begin.create-workflow-plan` |

---

### 3.2 Assignment 1: `init-existing-repository`

| Field | Value |
|-------|-------|
| **Short ID** | `init-existing-repository` |
| **Goal** | Set up the repository: create working branch, configure labels, initialize project board, rename template files, and open the setup PR |
| **Type** | Main assignment |
| **Acceptance Criteria** | (1) Working branch `dynamic-workflow-project-setup` exists; (2) Labels imported from `.github/.labels.json`; (3) Template placeholders (`workflow-orchestration-queue-india42`) replaced with repo-specific names; (4) Setup PR opened targeting `main`; (5) PR number captured as `$pr_num` for downstream use |
| **Prerequisites** | Workflow plan completed (pre-script) |
| **Dependencies** | None (first main assignment) |
| **Key Output** | `$pr_num` — the setup PR number, required by `pr-approval-and-merge` |
| **Risks** | (1) Labels JSON may be missing — create from spec; (2) Template replacement may miss embedded references — use `grep` to verify; (3) PR creation may fail if branch protections prevent direct push |
| **Output Record** | `#initiate-new-repository.init-existing-repository` |

**Post-Assignment Events:**
- `validate-assignment-completion` — verify branch exists, labels applied, PR opened
- `report-progress` — post progress summary

---

### 3.3 Assignment 2: `create-app-plan`

| Field | Value |
|-------|-------|
| **Short ID** | `create-app-plan` |
| **Goal** | Analyze all plan documents in `plan_docs/` and create an Application Plan GitHub Issue summarizing the project scope, phases, and acceptance criteria |
| **Type** | Main assignment |
| **Acceptance Criteria** | (1) All 5 plan markdown documents read and analyzed; (2) Reference code (`orchestrator_sentinel.py`, `notifier_service.py`, `src/`) analyzed; (3) Application Plan issue created with: title prefixed `[Application Plan]`, body contains phases (0–3), features, test cases, acceptance criteria, tech stack; (4) Plan issue linked to repo milestones; (5) Issue number captured for `post-script-complete` event |
| **Prerequisites** | Repository initialized (Assignment 1 complete) |
| **Dependencies** | `init-existing-repository` (needs branch, labels) |
| **Key Output** | Plan issue number — needed for `post-script-complete` event to apply `orchestration:plan-approved` label |
| **Project-Specific Notes** | The plan must reflect: Python 3.12+ primary language, FastAPI for webhooks, httpx for async HTTP, uv for package management, 3 required env vars (S-3), single-repo polling (S-5), consolidated queue (S-6), stdout-only logging (S-10). Phase 3 features (Architect Sub-Agent, self-healing loop, cross-repo polling) go in "Future Work" appendix (S-9). |
| **Risks** | (1) Plan docs have redundant info across 3 files — deduplicate in issue body; (2) Plan Review findings (I-1 through I-10) must be incorporated as implementation requirements, not ignored |
| **Output Record** | `#initiate-new-repository.create-app-plan` |

**Post-Assignment Events:**
- `validate-assignment-completion` — verify plan issue exists and is well-formed
- `report-progress` — post progress summary

---

### 3.4 Assignment 3: `create-project-structure`

| Field | Value |
|-------|-------|
| **Short ID** | `create-project-structure` |
| **Goal** | Create the project scaffolding — directory structure, `pyproject.toml`, configuration files, and skeleton source files |
| **Type** | Main assignment |
| **Acceptance Criteria** | (1) `pyproject.toml` with dependencies: fastapi, uvicorn, pydantic, httpx, using uv; (2) `src/` directory structure matching spec: `src/models/work_item.py`, `src/models/github_events.py`, `src/queue/github_queue.py`, `src/notifier_service.py`, `src/orchestrator_sentinel.py`; (3) Reference code from `plan_docs/` integrated into `src/` (not left in `plan_docs/`); (4) `.env.example` with 3 required vars; (5) `uv.lock` generated; (6) All files committed to `dynamic-workflow-project-setup` branch |
| **Prerequisites** | Repository initialized (Assignment 1), Application Plan issue created (Assignment 2) |
| **Dependencies** | `init-existing-repository`, `create-app-plan` |
| **Project-Specific Notes** | Target structure from Implementation Spec: `src/` with `models/`, `queue/` subdirs; `scripts/` for shell bridge; `local_ai_instruction_modules/` for markdown workflows. Must NOT create `.NET` project files (this is a Python project — `global.json` should NOT be included). The reference implementations in `plan_docs/` (sentinel, notifier, work_item, github_queue) should be used as the starting point, incorporating all Plan Review fixes (R-1 through R-9). |
| **Risks** | (1) `global.json` exists in repo root from template — must be removed or noted as template artifact; (2) Reference code has known issues (I-1 through I-10) that must be fixed during integration; (3) `pyproject.toml` must pin exact versions for reproducibility |
| **Output Record** | `#initiate-new-repository.create-project-structure` |

**Post-Assignment Events:**
- `validate-assignment-completion` — verify directory structure, pyproject.toml, imports resolve
- `report-progress` — post progress summary

---

### 3.5 Assignment 4: `create-agents-md-file`

| Field | Value |
|-------|-------|
| **Short ID** | `create-agents-md-file` |
| **Goal** | Update or rewrite `AGENTS.md` to reflect the actual project (workflow-orchestration-queue), removing template placeholders and adding project-specific instructions |
| **Type** | Main assignment |
| **Acceptance Criteria** | (1) All template placeholders (`workflow-orchestration-queue-india42`, `intel-agency`) replaced with actual repo names; (2) Repository map updated to reflect new project structure; (3) Tech stack section updated: Python 3.12+, FastAPI, httpx, uv (NOT .NET); (4) Testing section updated with Python-specific commands (`uv run pytest`, `uv run ruff check`); (5) Verification commands reflect actual project tooling; (6) Agent-specific guardrails updated for OS-APOW context |
| **Prerequisites** | Repository initialized (Assignment 1), project structure created (Assignment 3) |
| **Dependencies** | `init-existing-repository`, `create-project-structure` |
| **Project-Specific Notes** | AGENTS.md is the primary instruction file for AI agents working in this repo. It must accurately describe: the Python project structure, how to run the sentinel and notifier, how to run tests, and the agent delegation rules. The `.NET` references in the current template AGENTS.md must be completely replaced. |
| **Risks** | (1) Template AGENTS.md has extensive .NET references that could mislead agents; (2) Must preserve the opencode/orchestration infrastructure sections that are still relevant |
| **Output Record** | `#initiate-new-repository.create-agents-md-file` |

**Post-Assignment Events:**
- `validate-assignment-completion` — verify AGENTS.md is accurate, no stale placeholders
- `report-progress` — post progress summary

---

### 3.6 Assignment 5: `debrief-and-document`

| Field | Value |
|-------|-------|
| **Short ID** | `debrief-and-document` |
| **Goal** | Review all work completed, document learnings, capture decisions made during setup, and produce a summary of the project state |
| **Type** | Main assignment |
| **Acceptance Criteria** | (1) Summary of all assignments completed with outputs; (2) Decisions log (what was changed from plan docs, what was kept); (3) Known issues or TODOs identified during setup; (4) Recommendations for next steps (Phase 1 implementation); (5) Document committed to repo |
| **Prerequisites** | All previous assignments complete (1–4) |
| **Dependencies** | All preceding assignments |
| **Risks** | Low risk — this is a documentation task |
| **Output Record** | `#initiate-new-repository.debrief-and-document` |

**Post-Assignment Events:**
- `validate-assignment-completion` — verify debrief document exists and is comprehensive
- `report-progress` — post progress summary

---

### 3.7 Assignment 6: `pr-approval-and-merge`

| Field | Value |
|-------|-------|
| **Short ID** | `pr-approval-and-merge` |
| **Goal** | Merge the setup PR after validating CI passes |
| **Type** | Main assignment |
| **Special Input** | `$pr_num` from `#initiate-new-repository.init-existing-repository` |
| **Acceptance Criteria** | (1) PR `$pr_num` located; (2) CI checks verified green; (3) If CI fails, up to 3 remediation cycles attempted (Phase 0.5 loop); (4) PR approved (self-approval acceptable for automated setup PR); (5) PR merged; (6) Setup branch `dynamic-workflow-project-setup` deleted; (7) Related setup issues closed |
| **Prerequisites** | All previous assignments complete and committed |
| **Dependencies** | All preceding assignments (the PR contains all their work) |
| **Special Handling** | Self-approval by orchestrator is acceptable — this is an automated setup PR, not a feature PR requiring stakeholder review. However, the CI remediation loop MUST still run. |
| **Risks** | (1) CI may fail on new Python project structure — linting, import errors; (2) Template CI workflows may expect .NET project — may need adjustment; (3) Branch protection rules may block merge without external approval — may need admin override |
| **Output Record** | `#initiate-new-repository.pr-approval-and-merge` |

**Post-Assignment Events:**
- `validate-assignment-completion` — verify PR merged, branch deleted
- `report-progress` — post final progress summary

---

### 3.8 Post-Script Event: Apply `orchestration:plan-approved` Label

| Field | Value |
|-------|-------|
| **Goal** | Signal that the plan is ready for epic creation |
| **Action** | Apply `orchestration:plan-approved` label to the application plan issue (from Assignment 2) |
| **Prerequisites** | All assignments complete, PR merged |
| **Output Record** | `#events.post-script-complete.plan-approved` |

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│  PRE-SCRIPT-BEGIN                                                   │
│  ┌──────────────────────┐                                           │
│  │ create-workflow-plan │ ──► plan_docs/workflow-plan.md committed  │
│  └──────────────────────┘                                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ASSIGNMENT 1: init-existing-repository                             │
│  ┌──────────────────────────┐                                       │
│  │ Create branch, labels,   │                                       │
│  │ replace placeholders,    │ ──► OUTPUT: $pr_num                   │
│  │ open setup PR            │                                       │
│  └──────────────────────────┘                                       │
│       │                                                             │
│       ├── validate-assignment-completion                            │
│       └── report-progress                                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ASSIGNMENT 2: create-app-plan                                      │
│  ┌──────────────────────────┐                                       │
│  │ Analyze plan_docs/,      │                                       │
│  │ create Application Plan  │ ──► OUTPUT: plan_issue_number         │
│  │ GitHub Issue             │                                       │
│  └──────────────────────────┘                                       │
│       │                                                             │
│       ├── validate-assignment-completion                            │
│       └── report-progress                                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ASSIGNMENT 3: create-project-structure                             │
│  ┌──────────────────────────┐                                       │
│  │ pyproject.toml, src/     │                                       │
│  │ directory structure,     │ ──► OUTPUT: project scaffolding       │
│  │ integrate reference code │                                       │
│  └──────────────────────────┘                                       │
│       │                                                             │
│       ├── validate-assignment-completion                            │
│       └── report-progress                                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ASSIGNMENT 4: create-agents-md-file                                │
│  ┌──────────────────────────┐                                       │
│  │ Update AGENTS.md: remove │                                       │
│  │ template placeholders,   │ ──► OUTPUT: updated AGENTS.md         │
│  │ update tech stack/lang   │                                       │
│  └──────────────────────────┘                                       │
│       │                                                             │
│       ├── validate-assignment-completion                            │
│       └── report-progress                                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ASSIGNMENT 5: debrief-and-document                                 │
│  ┌──────────────────────────┐                                       │
│  │ Review all work, capture │                                       │
│  │ decisions, document      │ ──► OUTPUT: debrief document          │
│  │ learnings                │                                       │
│  └──────────────────────────┘                                       │
│       │                                                             │
│       ├── validate-assignment-completion                            │
│       └── report-progress                                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ASSIGNMENT 6: pr-approval-and-merge  (input: $pr_num)              │
│  ┌──────────────────────────┐                                       │
│  │ Verify CI → approve →   │                                       │
│  │ merge → delete branch →  │ ──► OUTPUT: merged PR, clean repo    │
│  │ close setup issues       │                                       │
│  └──────────────────────────┘                                       │
│       │                                                             │
│       ├── validate-assignment-completion                            │
│       └── report-progress                                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  POST-SCRIPT-COMPLETE                                               │
│  ┌──────────────────────────────────────────────────────┐           │
│  │ Apply `orchestration:plan-approved` label to          │           │
│  │ plan issue (from Assignment 2)                        │           │
│  └──────────────────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────────────┘
```

**Critical Path:**
```
create-workflow-plan → init-existing-repository → create-app-plan → create-project-structure → create-agents-md-file → debrief-and-document → pr-approval-and-merge → plan-approved
```

**Key Data Flow:**
```
init-existing-repository.$pr_num ──────────────────────────────► pr-approval-and-merge
create-app-plan.$plan_issue_number ────────────────────────────► post-script-complete
```

---

## 5. Dependency Graph

```
create-workflow-plan (pre-script)
         │
         ▼
init-existing-repository ──────► $pr_num
         │
         ▼
create-app-plan ────────────────► $plan_issue_number
         │
         ▼
create-project-structure
         │
         ▼
create-agents-md-file
         │
         ▼
debrief-and-document
         │
         ▼
pr-approval-and-merge ◄──────── $pr_num
         │
         ▼
post-script-complete ◄───────── $plan_issue_number
  (apply orchestration:plan-approved)
```

All main assignments are strictly sequential. Post-assignment events (`validate-assignment-completion`, `report-progress`) run after each assignment but are not blocking dependencies for subsequent assignments.

---

## 6. Risk Register

| ID | Risk | Impact | Probability | Mitigation |
|----|------|--------|-------------|------------|
| R-1 | Template CI workflows expect .NET project | High | High | Adjust or replace `validate` workflow for Python project structure before PR merge |
| R-2 | `global.json` in root confuses agents | Medium | High | Remove during `init-existing-repository` or `create-project-structure` |
| R-3 | Branch protection blocks self-merge | High | Medium | Use admin token or disable branch protection for setup PR |
| R-4 | Plan docs have redundant info causing verbose plan issue | Low | High | Deduplicate during `create-app-plan` — reference by section |
| R-5 | Reference code has known bugs (I-1 through I-10) | Medium | Certain | Fix during `create-project-structure` integration; track in debrief |
| R-6 | AGENTS.md still references opencode/zhipu infrastructure | Medium | Medium | Careful review during `create-agents-md-file` to preserve what's relevant |
| R-7 | CI remediation loop exceeds 3 attempts | Medium | Low | Escalate to human; document in debrief |

---

## 7. Open Questions

1. **CI Workflow Alignment:** The existing `validate` workflow in `.github/workflows/` is built for the template (devcontainer build, shell tests). For the Python project, do we need to add Python-specific CI steps (ruff lint, pytest, mypy) during this setup, or defer to Phase 1 implementation?

2. **Template File Disposition:** The repo root contains `global.json` (a .NET file), `workflow-orchestration-queue-india42.code-workspace`, and other template artifacts. Should these be removed during `init-existing-repository`, or left for backward compatibility?

3. **Reference Code Location:** The reference implementations currently live in `plan_docs/` (e.g., `plan_docs/orchestrator_sentinel.py`, `plan_docs/src/`). Should `create-project-structure` move them to the root `src/` directory and remove them from `plan_docs/`, or keep both copies?

4. **Existing Shell Scripts:** The `scripts/` directory already contains `devcontainer-opencode.sh` (referenced in the architecture). Does the `create-project-structure` assignment need to create additional scripts (e.g., `start_dev_notifier.sh`, `run_sentinel.sh`), or are those deferred to Phase 1/2?

5. **GitHub Project Board:** Does `init-existing-repository` need to create a GitHub Project (V2) for tracking, or just labels and milestones?

6. **Simplification S-3 (3 env vars only):** The reference `orchestrator_sentinel.py` in `plan_docs/` already reflects this (only `GITHUB_TOKEN`, `GITHUB_ORG`, `GITHUB_REPO` as required). Should `GITHUB_REPO` also be removed from the required set since the notifier derives the repo from the webhook payload?

7. **`.env.example` Scope:** Should the `.env.example` include the 3 sentinel vars only, or also the notifier vars (`WEBHOOK_SECRET`, `GITHUB_TOKEN`) and optional tuning knobs?

---

## 8. Constraints & Directives

- **Action SHA Pinning:** Any GitHub Actions workflows created or modified during this workflow MUST pin all actions to the specific commit SHA of their latest release. No major/minor version tags.
- **Orchestrator Delegation Depth:** ≤ 2 levels. No limit on concurrent delegations.
- **Orchestrator Never Writes Code:** The orchestrator delegates all implementation to specialist agents.
- **Self-Approval:** The setup PR (Assignment 6) may be self-approved by the orchestrator — no human stakeholder required.
- **CI Remediation:** Up to 3 fix cycles on CI failure before escalation.

---

*This plan was generated by the `create-workflow-plan` assignment as part of the `project-setup` dynamic workflow pre-script-begin event.*
