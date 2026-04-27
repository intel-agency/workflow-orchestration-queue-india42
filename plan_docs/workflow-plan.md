# Workflow Execution Plan: project-setup

**Dynamic Workflow:** `project-setup`
**Project:** workflow-orchestration-queue (OS-APOW)
**Repository:** intel-agency/workflow-orchestration-queue-india42
**Branch:** `dynamic-workflow-project-setup`
**Created:** 2026-04-27
**Status:** Planning Complete

---

## 1. Overview

This plan governs the execution of the `project-setup` dynamic workflow for the **workflow-orchestration-queue** (OS-APOW) project — an autonomous, headless agentic orchestration platform that transforms GitHub Issues into automated Execution Orders fulfilled by specialized AI agents.

The workflow consists of **6 sequential main assignments** and **2 recurring post-assignment events** that execute after each main assignment completes.

| # | Assignment | Type | Est. Complexity |
|---|-----------|------|----------------|
| 1 | `init-existing-repository` | Infrastructure / Setup | Medium |
| 2 | `create-app-plan` | Planning / Documentation | High |
| 3 | `create-project-structure` | Scaffolding / Code Generation | High |
| 4 | `create-agents-md-file` | Documentation | Low |
| 5 | `debrief-and-document` | Retrospective / Reporting | Medium |
| 6 | `pr-approval-and-merge` | Review / Merge | Medium |

**Post-assignment events** (fire after each main assignment):
- `validate-assignment-completion` — Independent QA validation
- `report-progress` — Progress report, output capture, checkpoint

**Post-script-complete event** (fires after all assignments):
- Apply `orchestration:plan-approved` label to the application plan issue

---

## 2. Project Context Summary

### 2.1 What Is Being Built

**workflow-orchestration-queue (OS-APOW)** is a Python-based headless agentic orchestration platform with four conceptual pillars:

1. **The Ear (Notifier)** — FastAPI webhook receiver for GitHub event ingestion, HMAC signature verification, intelligent triaging
2. **The State (Work Queue)** — GitHub Issues as distributed state ("Markdown as a Database"), label-based state machine
3. **The Brain (Sentinel Orchestrator)** — Async polling service with assign-then-verify distributed locking, shell-bridge execution, heartbeat monitoring
4. **The Hands (Opencode Worker)** — DevContainer-based AI worker executing markdown instruction modules

### 2.2 Technology Stack

| Category | Technology |
|----------|-----------|
| Language | Python 3.12+ |
| Web Framework | FastAPI + Uvicorn |
| Data Validation | Pydantic v2 |
| HTTP Client | httpx (async) |
| Package Manager | uv (Rust-based) |
| Containerization | Docker / DevContainers |
| State Management | GitHub Issues + Labels |
| Authentication | GitHub App Installation Tokens |
| Logging | Python stdlib logging (stdout-only, S-10) |
| Shell Bridge | `./scripts/devcontainer-opencode.sh` |
| LLM Runtime | opencode CLI (GLM-5 / Claude) |

### 2.3 Key Architectural Decisions

- **ADR 07:** Shell-Bridge execution (reuse `devcontainer-opencode.sh`, not Docker SDK)
- **ADR 08:** Polling-first resiliency (webhooks are optimization, not requirement)
- **ADR 09:** Provider-agnostic `ITaskQueue` interface (kept per S-1 for future provider swapping)

### 2.4 Reference Code Already Provided

The `plan_docs/` directory includes working reference implementations:

| File | Purpose |
|------|---------|
| `orchestrator_sentinel.py` | Sentinel polling service (292 lines) |
| `notifier_service.py` | FastAPI webhook receiver (110 lines) |
| `src/models/work_item.py` | Unified `WorkItem`, `TaskType`, `WorkItemStatus`, `scrub_secrets()` (74 lines) |
| `src/queue/github_queue.py` | Consolidated `GitHubQueue` with connection pooling (249 lines) |

These are **scaffold/reference** files that the `create-project-structure` assignment should use as a foundation, incorporating the fixes identified in the Plan Review.

### 2.5 Known Issues from Plan Review (to Address During Implementation)

| ID | Issue | Severity | Resolution Status |
|----|-------|----------|------------------|
| I-1 / R-3 | Divergent WorkItem models | High | ✅ Fixed in shared `src/models/work_item.py` |
| I-2 / R-2 | Race condition in task claiming | High | ✅ Fixed (assign-then-verify in `github_queue.py`) |
| I-3 | No jittered exponential backoff | High | ✅ Fixed in `orchestrator_sentinel.py` |
| I-4 / R-5 | No connection pooling | Medium | ✅ Fixed (single `httpx.AsyncClient` in `__init__`) |
| I-5 / R-6 | Hardcoded secrets in notifier | High | ✅ Fixed (env var validation at import time) |
| I-6 / R-1 | No heartbeat implementation | High | ✅ Fixed (async heartbeat coroutine) |
| I-7 | Cost guardrails not implemented | Medium | ⏳ Deferred (documented as future work) |
| I-8 | Single-repo only polling | Low | ⏳ Deferred (S-5: cross-repo is future phase) |
| I-9 | Bare `except: pass` in claim_task | Medium | ✅ Fixed (specific exception handling) |
| I-10 | No environment reset between tasks | Medium | ✅ Fixed (stop in `finally` block) |
| R-4 | No graceful shutdown | High | ✅ Fixed (SIGTERM/SIGINT handlers) |
| R-7 | No credential scrubber | High | ✅ Fixed (`scrub_secrets()` in `work_item.py`) |
| R-8 | No subprocess timeout | High | ✅ Fixed (`asyncio.wait_for` with SUBPROCESS_TIMEOUT) |

### 2.6 Applied Simplifications (from S-Report)

| ID | Simplification | Status |
|----|---------------|--------|
| S-1 | Keep `ITaskQueue` ABC | KEPT (user preference) |
| S-2 | Keep doc duplication | KEPT (aids autonomous agents) |
| S-3 | Reduce to 3 required env vars | IMPLEMENTED |
| S-4 | Hardcode env reset to "stop" | IMPLEMENTED |
| S-5 | Single-repo polling only | IMPLEMENTED (noted as future) |
| S-6 | Consolidated queue to `src/queue/github_queue.py` | IMPLEMENTED |
| S-7 | Removed IPv4 scrubbing pattern | IMPLEMENTED |
| S-8 | Removed "encrypted" log verbiage | IMPLEMENTED |
| S-9 | Phase 3 moved to appendix | IMPLEMENTED |
| S-10 | Stdout-only logging | IMPLEMENTED |
| S-11 | Removed `raw_payload` field | IMPLEMENTED |

### 2.7 Constraints & Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| `GH_ORCHESTRATION_AGENT_TOKEN` required for branch protection ruleset import | Medium — ruleset import will fail without it | Document in `init-existing-repository` notes; skip gracefully if unavailable |
| No `ai-new-app-template.md` in plan_docs — app spec is spread across multiple files | Medium — `create-app-plan` must synthesize from multiple docs | Use Development Plan v4.2 as primary, cross-reference Architecture Guide and Impl Spec |
| Reference code in `plan_docs/` may need adaptation from scaffold to production | Low — `create-project-structure` will refactor | Treat reference code as blueprint; create clean project structure |
| GitHub Project creation requires `project` and `read:project` scopes | Medium — project creation may fail | Verify auth scopes in `init-existing-repository` step |
| Long PR review cycle possible if branch protection is strict | Low — may delay merge | Plan for iterative comment resolution in `pr-approval-and-merge` |

---

## 3. Assignment Execution Plan

### Assignment 1: `init-existing-repository`

**Short ID:** `init-existing-repository`
**Goal:** Create the working branch, configure GitHub repository settings (branch protection, project, labels), rename workspace/devcontainer files, and open the setup PR.

**Key Acceptance Criteria:**
- [ ] Branch `dynamic-workflow-project-setup` created from `main`
- [ ] Branch protection ruleset imported from `.github/protected branches - main - ruleset.json`
- [ ] GitHub Project (Board) created and linked to the repository with columns: Not Started, In Progress, In Review, Done
- [ ] Labels imported from `.github/.labels.json` via `scripts/import-labels.ps1`
- [ ] `.devcontainer/devcontainer.json` `name` property renamed to `workflow-orchestration-queue-india42-devcontainer`
- [ ] `workflow-orchestration-queue-india42.code-workspace` already correctly named (verify)
- [ ] PR created from `dynamic-workflow-project-setup` to `main`

**Project-Specific Notes:**
- The workspace file is already named `workflow-orchestration-queue-india42.code-workspace` — verify correctness but likely no rename needed.
- The ruleset file is at `.github/protected branches - main - ruleset.json` (note: spaces in filename).
- Use `GH_ORCHESTRATION_AGENT_TOKEN` for ruleset import (requires `administration: write` scope).
- Run `scripts/test-github-permissions.ps1` to verify auth scopes before proceeding.

**Prerequisites:** GitHub auth with `repo`, `project`, `read:project`, `read:user`, `user:email`, `administration: write` scopes.

**Risks/Challenges:**
- Ruleset import may fail if PAT lacks `administration: write` scope — stop and report, do not silently skip.
- PR creation requires at least one commit ahead of `main` — ensure file renames are committed before creating PR.
- GitHub Projects v2 API may require different commands than classic projects.

**Post-assignment events:**
- `validate-assignment-completion` — Verify branch exists, project is created, labels imported, PR is open
- `report-progress` — Capture PR number, project URL, branch name as outputs

---

### Assignment 2: `create-app-plan`

**Short ID:** `create-app-plan`
**Goal:** Analyze all plan_docs documents and create a comprehensive application plan documented as a GitHub Issue, with milestones for each phase.

**Key Acceptance Criteria:**
- [ ] All plan_docs documents analyzed and understood (Development Plan v4.2, Architecture Guide v3.2, Impl Spec v1.2, Plan Review, Simplification Report, reference code)
- [ ] `plan_docs/tech-stack.md` created documenting the technology stack
- [ ] `plan_docs/architecture.md` created documenting high-level architecture
- [ ] Application plan issue created using `.github/ISSUE_TEMPLATE/application-plan.md` template
- [ ] Plan covers all 5 phases: Foundation, Sentinel MVP, Ear/Webhook, Deep Orchestration, Testing/Deployment
- [ ] All mandatory requirements addressed (testing, documentation, containerization, CI/CD, security)
- [ ] All risks and mitigations identified
- [ ] Plan Review issues (I-1 through I-10) addressed as implementation tasks
- [ ] Simplification decisions (S-1 through S-11) reflected in the plan
- [ ] Milestones created for each phase (e.g., "Phase 1: Foundation", "Phase 2: Sentinel MVP")
- [ ] Plan issue linked to GitHub Project and assigned to Phase 1 milestone
- [ ] Labels applied: `planning`, `documentation`
- [ ] **NO code implementation** — planning only

**Project-Specific Notes:**
- There is no single `ai-new-app-template.md` file. The application spec is distributed across:
  - `OS-APOW Development Plan v4.2.md` (primary: phased roadmap, stories, acceptance criteria)
  - `OS-APOW Architecture Guide v3.2.md` (system design, ADRs, data flow)
  - `OS-APOW Implementation Specification v1.2.md` (requirements, test cases, project structure)
  - `OS-APOW Plan Review.md` (issues I-1 to I-10, recommendations R-1 to R-9)
  - `OS-APOW Simplification Report v1.md` (decisions S-1 to S-11)
- The reference code (`orchestrator_sentinel.py`, `notifier_service.py`, `src/`) should be documented as the starting baseline.
- Key phases for milestones:
  - Phase 0: Seeding & Bootstrapping (may be partially done)
  - Phase 1: The Sentinel (MVP) — polling, claiming, shell-bridge, status feedback
  - Phase 2: The Ear — FastAPI webhook receiver, HMAC validation, triaging
  - Phase 3: Deep Orchestration — Architect Sub-Agent, self-correction (appendix/future)
  - Phase 4: Testing, Documentation, CI/CD & Deployment

**Prerequisites:** Assignment 1 complete (branch, project, labels exist).

**Risks/Challenges:**
- Synthesizing 5+ documents into a coherent plan requires careful cross-referencing.
- Plan Review and Simplification Report contain contradictory guidance in places (e.g., S-1 says drop ABC vs. "KEPT").
- Must ensure deferred items (I-7, I-8, Phase 3 features) are explicitly marked as future work.

**Post-assignment events:**
- `validate-assignment-completion` — Verify plan issue exists, milestones created, tech-stack.md and architecture.md exist
- `report-progress` — Capture issue number, milestone names, plan document paths

---

### Assignment 3: `create-project-structure`

**Short ID:** `create-project-structure`
**Goal:** Create the complete project scaffolding including Python package structure, Dockerfiles, CI/CD workflows, documentation, and development tooling based on the application plan.

**Key Acceptance Criteria:**
- [ ] Python project structure created following Impl Spec project layout
- [ ] `pyproject.toml` with all dependencies (fastapi, uvicorn, pydantic, httpx, etc.)
- [ ] `uv.lock` generated for deterministic builds
- [ ] Source files created:
  - `src/orchestrator_sentinel.py` — Sentinel polling service
  - `src/notifier_service.py` — FastAPI webhook receiver
  - `src/models/__init__.py`, `src/models/work_item.py` — Unified data model
  - `src/models/github_events.py` — GitHub webhook payload schemas
  - `src/queue/__init__.py`, `src/queue/github_queue.py` — Consolidated queue
  - `src/__init__.py`
- [ ] Reference code from `plan_docs/` refactored into proper project structure
- [ ] `.python-version` pinned to 3.12+
- [ ] Dockerfile for each service (Sentinel, Notifier)
- [ ] `docker-compose.yml` for local development (healthchecks use Python stdlib, not curl)
- [ ] `.env.example` with required variables documented (GITHUB_TOKEN, GITHUB_ORG, GITHUB_REPO)
- [ ] CI/CD workflows created in `.github/workflows/` with actions pinned to commit SHA
- [ ] `tests/` directory with initial test structure
- [ ] `README.md` comprehensive project documentation
- [ ] `.ai-repository-summary.md` created per `create-repository-summary.md` spec
- [ ] Repository summary linked from README.md
- [ ] Solution builds successfully (`uv sync` or equivalent)
- [ ] All GitHub Actions workflows have actions pinned to specific commit SHA

**Project-Specific Notes:**
- This is a Python project using `uv` — not .NET. Adapt generic structure guidance accordingly.
- The target project structure (from Impl Spec):
  ```
  workflow-orchestration-queue/
  ├── pyproject.toml
  ├── uv.lock
  ├── src/
  │   ├── __init__.py
  │   ├── notifier_service.py
  │   ├── orchestrator_sentinel.py
  │   ├── models/
  │   │   ├── __init__.py
  │   │   ├── work_item.py
  │   │   └── github_events.py
  │   └── queue/
  │       ├── __init__.py
  │       └── github_queue.py
  ├── tests/
  │   ├── __init__.py
  │   ├── test_sentinel.py
  │   ├── test_notifier.py
  │   ├── test_work_item.py
  │   └── test_github_queue.py
  ├── scripts/
  ├── Dockerfile
  ├── docker-compose.yml
  ├── .env.example
  ├── .python-version
  └── README.md
  ```
- **IMPORTANT:** When using `uv pip install -e .`, ensure `COPY src/ ./src/` appears BEFORE the install command in Dockerfile.
- **IMPORTANT:** Healthchecks in docker-compose.yml must use Python stdlib (`python -c "import urllib.request; ..."`), NOT `curl`.
- Reference code in `plan_docs/` already incorporates Plan Review fixes (I-1 through I-10, R-1 through R-8). Use it as the baseline.
- The existing `scripts/` directory already contains shell bridge scripts — preserve and reference them.

**Prerequisites:** Assignment 2 complete (application plan issue exists with documented tech stack and architecture).

**Risks/Challenges:**
- Existing repository has template infrastructure (`scripts/`, `.github/`, `.devcontainer/`) that must be preserved alongside new project files.
- The `plan_docs/` directory should remain as-is (seeded documentation) — do not move or modify its contents.
- CI/CD workflows must be compatible with the existing GitHub Actions setup in the template.
- Docker build must work with `uv` for dependency installation.

**Post-assignment events:**
- `validate-assignment-completion` — Verify build succeeds, files exist, Docker configs valid
- `report-progress` — Capture file count, build status, Docker compose validation

---

### Assignment 4: `create-agents-md-file`

**Short ID:** `create-agents-md-file`
**Goal:** Create a comprehensive `AGENTS.md` file at the repository root providing AI coding agents with precise, actionable project context.

**Key Acceptance Criteria:**
- [ ] `AGENTS.md` exists at repository root
- [ ] Contains project overview (OS-APOW, autonomous orchestration, Python/FastAPI/Docker)
- [ ] Contains verified build/test/lint commands:
  - `uv sync` (install dependencies)
  - `uv run pytest` (run tests)
  - `uv run python -m src.orchestrator_sentinel` (run sentinel)
  - `uv run uvicorn src.notifier_service:app` (run notifier)
- [ ] Contains project structure / directory layout section
- [ ] Contains code style conventions (PEP 8, type hints, docstrings)
- [ ] Contains testing instructions (pytest, coverage, where to place tests)
- [ ] Contains PR/commit guidelines
- [ ] All commands validated by running them
- [ ] Complements (not duplicates) existing `AGENTS.md` template file

**Project-Specific Notes:**
- This repository already has an `AGENTS.md` file (the template repo's instructions). The new `AGENTS.md` should **replace** it with project-specific content, preserving the general agent coordination instructions where applicable.
- Key commands to verify and document:
  - `uv sync` — install all dependencies
  - `uv run pytest tests/` — run test suite
  - `uv run python -c "from src.models.work_item import WorkItem"` — verify imports
  - `docker compose build` — build Docker images
  - `docker compose up` — run locally
- Document the `plan_docs/` directory as read-only context material.
- Document the relationship between the existing template infrastructure (`.github/`, `.opencode/`, `scripts/`) and the new project code (`src/`, `tests/`).

**Prerequisites:** Assignment 3 complete (project structure exists, build works).

**Risks/Challenges:**
- Must not duplicate the existing AGENTS.md content — project-specific content replaces template content.
- Commands must be verified to work; if the project doesn't build yet, note that in the file.

**Post-assignment events:**
- `validate-assignment-completion` — Verify AGENTS.md exists, commands work, sections complete
- `report-progress` — Capture AGENTS.md path, command validation results

---

### Assignment 5: `debrief-and-document`

**Short ID:** `debrief-and-document`
**Goal:** Create a comprehensive debriefing report capturing lessons learned, metrics, deviations, and recommendations from the entire project-setup workflow.

**Key Acceptance Criteria:**
- [ ] Debrief report created following the structured 12-section template
- [ ] All sections complete: Executive Summary, Workflow Overview, Key Deliverables, Lessons Learned, What Worked Well, What Could Be Improved, Errors Encountered, Complex Steps, Suggested Changes, Metrics & Statistics, Future Recommendations, Conclusion
- [ ] All deviations from assignments documented with explanations
- [ ] Plan-impacting findings flagged as ACTION ITEMS with recommended follow-up
- [ ] Execution trace saved to `debrief-and-document/trace.md`
- [ ] Report committed and pushed to the working branch
- [ ] Stakeholder approval obtained

**Project-Specific Notes:**
- The debrief should specifically assess:
  - Whether the reference code from `plan_docs/` was sufficient for scaffolding
  - Whether the Plan Review issues were adequately addressed
  - Whether the simplification decisions (S-1 through S-11) were correctly applied
  - Any challenges with the Python/uv toolchain in the devcontainer
- Action items should reference specific Plan Review issues if unresolved.
- Metrics to capture: total files created, lines of code, dependencies, build time.

**Prerequisites:** Assignment 4 complete (all project files created, AGENTS.md written).

**Risks/Challenges:**
- The debrief requires honest assessment of what went wrong — resist the temptation to only document successes.
- Trace file must capture actual commands run and their outputs.

**Post-assignment events:**
- `validate-assignment-completion` — Verify debrief report exists with all 12 sections, trace.md present
- `report-progress` — Capture report path, deviations count, action items filed

---

### Assignment 6: `pr-approval-and-merge`

**Short ID:** `pr-approval-and-merge`
**Goal:** Complete the full PR approval and merge process — resolve all review comments, obtain approval, merge the setup PR, and verify CI.

**Key Acceptance Criteria:**
- [ ] All CI/CD status checks pass (CI remediation loop: up to 3 attempts)
- [ ] Code review delegated to `code-reviewer` subagent (NOT self-review)
- [ ] Auto-reviewer comments (Copilot, CodeQL, etc.) waited for before resolution
- [ ] `ai-pr-comment-protocol.md` workflow executed and logged
- [ ] All review threads resolved with unique replies and explicit resolution
- [ ] GraphQL verification: `pr-unresolved-threads.json` is empty
- [ ] Stakeholder approval obtained after presenting resolution evidence
- [ ] PR merged to `main`
- [ ] Source branch deleted (if policy allows)
- [ ] Related issues updated with resolution notes
- [ ] Run report updated with final status

**Project-Specific Notes:**
- The PR number will be captured from Assignment 1 output.
- The PR will contain all changes from Assignments 1–5 (branch setup, app plan, project structure, AGENTS.md, debrief).
- Branch protection ruleset (if successfully imported in Assignment 1) may require specific checks to pass.
- Commit hygiene is critical — all changes must be committed and pushed before merge.

**Prerequisites:** Assignment 5 complete (all work committed to the branch).

**Risks/Challenges:**
- If branch protection was not imported successfully (e.g., PAT scope issue in Assignment 1), merge may be unprotected.
- Large PR with many files may generate many automated review comments.
- CI workflows must exist and pass — if `create-project-structure` didn't create working CI, this step may need to fix CI first.

**Post-assignment events:**
- `validate-assignment-completion` — Verify PR is merged, branch deleted, issues updated
- `report-progress` — Capture merge commit SHA, final status

**Post-script-complete event:**
- Apply `orchestration:plan-approved` label to the application plan issue (from Assignment 2)

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    project-setup Dynamic Workflow                           │
│                    Branch: dynamic-workflow-project-setup                    │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────┐
  │ 1. init-existing-    │
  │    repository         │
  │ • Create branch       │
  │ • Import ruleset      │
  │ • Create GH Project   │
  │ • Import labels       │
  │ • Rename files        │
  │ • Create PR ───────────────────────► PR #N created
  └──────────┬───────────┘
             │
             ▼
  ┌──────────────────────────┐     ┌───────────────────────────┐
  │ [validate-assignment-1]  │     │ [report-progress-1]       │
  │  QA: Branch, project,    │     │  Output: PR#, project URL │
  │  labels, PR verified     │     │  Checkpoint saved         │
  └──────────────────────────┘     └───────────────────────────┘
             │
             ▼
  ┌──────────────────────┐
  │ 2. create-app-plan    │
  │ • Analyze plan_docs   │
  │ • Create tech-stack.md│
  │ • Create architecture │
  │   .md                 │
  │ • Create plan issue   │
  │ • Create milestones   │
  │ • Link to project     │
  │ (NO CODE)             │
  └──────────┬───────────┘
             │
             ▼
  ┌──────────────────────────┐     ┌───────────────────────────┐
  │ [validate-assignment-2]  │     │ [report-progress-2]       │
  │  QA: Issue exists,       │     │  Output: Issue#, milestones│
  │  milestones, docs        │     │  Checkpoint saved         │
  └──────────────────────────┘     └───────────────────────────┘
             │
             ▼
  ┌──────────────────────┐
  │ 3. create-project-    │
  │    structure          │
  │ • Python package init │
  │ • pyproject.toml      │
  │ • Source code files   │
  │ • Docker/compose      │
  │ • CI/CD workflows     │
  │ • Tests structure     │
  │ • README.md           │
  │ • .ai-repo-summary    │
  │ • Verify build        │
  └──────────┬───────────┘
             │
             ▼
  ┌──────────────────────────┐     ┌───────────────────────────┐
  │ [validate-assignment-3]  │     │ [report-progress-3]       │
  │  QA: Build succeeds,     │     │  Output: File count, build│
  │  files exist, Docker OK  │     │  status, Docker valid     │
  └──────────────────────────┘     └───────────────────────────┘
             │
             ▼
  ┌──────────────────────┐
  │ 4. create-agents-     │
  │    md-file            │
  │ • Gather context      │
  │ • Validate commands   │
  │ • Write AGENTS.md     │
  │ • Cross-reference docs│
  │ • Final validation    │
  └──────────┬───────────┘
             │
             ▼
  ┌──────────────────────────┐     ┌───────────────────────────┐
  │ [validate-assignment-4]  │     │ [report-progress-4]       │
  │  QA: AGENTS.md exists,   │     │  Output: Path, validation │
  │  commands verified       │     │  results                  │
  └──────────────────────────┘     └───────────────────────────┘
             │
             ▼
  ┌──────────────────────┐
  │ 5. debrief-and-       │
  │    document           │
  │ • 12-section report   │
  │ • Execution trace     │
  │ • Deviations logged   │
  │ • Action items filed  │
  │ • Stakeholder review  │
  └──────────┬───────────┘
             │
             ▼
  ┌──────────────────────────┐     ┌───────────────────────────┐
  │ [validate-assignment-5]  │     │ [report-progress-5]       │
  │  QA: Report complete,    │     │  Output: Report path,     │
  │  trace.md present        │     │  deviations, action items │
  └──────────────────────────┘     └───────────────────────────┘
             │
             ▼
  ┌──────────────────────┐
  │ 6. pr-approval-and-   │
  │    merge              │
  │ • CI verification     │
  │ • Code review         │
  │ • Resolve comments    │
  │ • Obtain approval     │
  │ • Merge PR            │
  │ • Delete branch       │
  │ • Close issues        │
  └──────────┬───────────┘
             │
             ▼
  ┌──────────────────────────┐     ┌───────────────────────────┐
  │ [validate-assignment-6]  │     │ [report-progress-6]       │
  │  QA: PR merged, branch   │     │  Output: Merge SHA,       │
  │  deleted, CI green       │     │  final status             │
  └──────────────────────────┘     └───────────────────────────┘
             │
             ▼
  ┌──────────────────────────────────────────┐
  │ POST-SCRIPT-COMPLETE EVENT:              │
  │ Apply `orchestration:plan-approved`      │
  │ label to the application plan issue      │
  │ (from Assignment 2)                      │
  └──────────────────────────────────────────┘
```

---

## 5. Dependency Chain

```
init-existing-repository
    │
    ├──► (creates branch, PR, project, labels)
    │
    ▼
create-app-plan
    │
    ├──► (creates plan issue, milestones, tech-stack.md, architecture.md)
    │
    ▼
create-project-structure
    │
    ├──► (uses tech-stack + architecture docs to scaffold project)
    │
    ▼
create-agents-md-file
    │
    ├──► (uses existing project structure to document commands)
    │
    ▼
debrief-and-document
    │
    ├──► (documents entire workflow execution)
    │
    ▼
pr-approval-and-merge
    │
    └──► (merges all accumulated changes)
```

**Critical Path:** All assignments are strictly sequential. Each depends on the output of the previous one.

---

## 6. Open Questions

| # | Question | Impact | Recommendation |
|---|----------|--------|----------------|
| 1 | **Will `GH_ORCHESTRATION_AGENT_TOKEN` be available?** The ruleset import requires `administration: write` scope on the PAT. | If unavailable, branch protection ruleset import in Assignment 1 will fail. | Test with `scripts/test-github-permissions.ps1` first. If unavailable, skip ruleset import with explicit documentation and file an issue. |
| 2 | **Should the existing `AGENTS.md` be replaced or merged?** The template repo's AGENTS.md contains opencode/orchestration instructions. | The new project-specific AGENTS.md may lose important agent coordination context. | Preserve the orchestration-specific sections (agent delegation, prompt assembly) in the new AGENTS.md while adding project-specific content. |
| 3 | **How should `plan_docs/` reference code be used?** The scaffold code has known issues (I-7, I-8) that are documented as deferred. | `create-project-structure` must decide whether to implement or defer. | Use reference code as-is for scaffolding. Mark deferred items with TODO comments and file issues for future work. |
| 4 | **What CI/CD workflows should be created?** The existing template has `validate`, `publish-docker`, `prebuild-devcontainer` workflows. | New project-specific CI may conflict with template CI. | Create project-specific CI (build, test, lint) that complements (not replaces) the existing template workflows. |
| 5 | **Is Phase 3 in scope for the plan issue?** The Simplification Report (S-9) moved Phase 3 to an appendix/future work. | The `create-app-plan` assignment must decide the phase boundary. | Include Phase 3 as documented future work in the plan issue, but do not create milestones or tasks for it. |
| 6 | **What GitHub Project columns are needed?** The assignment specifies Not Started, In Progress, In Review, Done. | May need additional columns for agent-specific states. | Use the specified columns. Agent state labels (agent:queued, agent:in-progress, etc.) track AI workflow state separately from project board columns. |
| 7 | **Should `src/models/github_events.py` be created?** The Impl Spec lists it but the reference code doesn't include it. | Missing webhook payload schemas may complicate Phase 2. | Create as a stub with basic schemas in Assignment 3; implement fully in Phase 2. |

---

## 7. Output Artifact Tracking

| Assignment | Key Outputs | Where Stored |
|-----------|-------------|-------------|
| 1 | Branch name, PR number, Project URL, labels imported | Git branch, GitHub |
| 2 | Plan issue number, milestone names, tech-stack.md, architecture.md | GitHub Issues, `plan_docs/` |
| 3 | File count, build status, Docker validation, .ai-repo-summary.md | `src/`, `tests/`, root files |
| 4 | AGENTS.md path, command validation results | Repository root |
| 5 | Debrief report path, trace.md, action item issue numbers | `debrief-and-document/`, GitHub Issues |
| 6 | Merge commit SHA, final status, closed issues | Git main branch |
| Post-script | `orchestration:plan-approved` label on plan issue | GitHub Issue labels |

---

*Plan prepared by: Planner Agent*
*Date: 2026-04-27*
*Status: Ready for Execution*
