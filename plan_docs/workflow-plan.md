# Workflow Execution Plan: Project Setup

## 1. Overview

**Workflow Name:** `project-setup`  
**Workflow File:** `ai_instruction_modules/ai-workflow-assignments/dynamic-workflows/project-setup.md`  
**Project:** workflow-orchestration-queue (OS-APOW: Open Source Agentic Platform Orchestrating Workflows)  
**Repository:** `intel-agency/workflow-orchestration-queue-india42`

**Summary:**  
This workflow initializes the OS-APOW repository from the template, establishing the foundational project structure, documentation, and development environment for a headless agentic orchestration platform. The system transforms GitHub Issues into automated execution orders using a persistent background service (Sentinel), webhook receiver (Notifier), and DevContainer-based worker environment.

**Total Assignments:**  
- **1 pre-script event** (create-workflow-plan - currently executing)
- **6 main assignments** (init-existing-repository → create-app-plan → create-project-structure → create-agents-md-file → debrief-and-document → pr-approval-and-merge)
- **2 post-assignment events** (validate-assignment-completion, report-progress) - executed after each main assignment
- **1 post-script action** (apply orchestration:plan-approved label)

**Execution Flow:** 20 distinct steps total

---

## 2. Project Context Summary

### Project Overview
**OS-APOW (Open Source Agentic Platform Orchestrating Workflows)** is a groundbreaking headless agentic orchestration platform that transforms the current paradigm of interactive AI coding into an autonomous background production service. It eliminates the human-in-the-loop dependency by translating standard project management artifacts (GitHub Issues) into automated Execution Orders.

### Architecture (4 Pillars)
1. **The Ear (Notifier)**: FastAPI-based webhook receiver for event ingestion
2. **The State (Queue)**: GitHub Issues as distributed state management ("Markdown as a Database")
3. **The Brain (Sentinel)**: Persistent background polling service with distributed locking
4. **The Hands (Worker)**: DevContainer-based execution environment via shell bridge

### Technology Stack
- **Language:** Python 3.12+
- **Web Framework:** FastAPI with Uvicorn
- **Package Manager:** uv (Rust-based, not pip/poetry)
- **HTTP Client:** httpx (async)
- **Validation:** Pydantic
- **Containerization:** Docker + DevContainers
- **State Management:** GitHub Issues with labels (agent:queued, agent:in-progress, agent:success, agent:error, agent:infra-failure)
- **Shell Bridge:** scripts/devcontainer-opencode.sh

### Key Constraints
1. **Script-First Integration:** Orchestrator uses shell scripts, not Docker SDK
2. **Polling-First Resiliency:** Webhooks are optimization; polling is primary
3. **Provider-Agnostic Interface:** ITaskQueue ABC for future provider swapping
4. **SHA-Pinned Actions:** All GitHub Actions must use commit SHA, not version tags
5. **Self-Bootstrapping:** Phase 1 is manually seeded; Phases 2-3 are AI-built

### Phased Rollout
- **Phase 0 (Seeding):** Manual template clone and plan doc seeding ✅ (Already done)
- **Phase 1 (Sentinel MVP):** Persistent polling service with shell bridge ← **Current focus**
- **Phase 2 (Webhooks):** FastAPI webhook receiver for instant triage
- **Phase 3 (Deep Orchestration):** Hierarchical task decomposition and self-healing

### Repository State
- Template repository with existing shell scripts, devcontainer configs, and workflow templates
- `plan_docs/` contains comprehensive planning documents (Development Plan v4.2, Implementation Spec v1.2, Architecture Guide v3.2, Plan Review, Simplification Report)
- Reference implementations provided in `plan_docs/src/` (orchestrator_sentinel.py, notifier_service.py, models, queue)
- Template placeholders (`workflow-orchestration-queue-india42`) need replacement

---

## 3. Assignment Execution Plan

### Pre-Script Event

#### create-workflow-plan
**Goal:** Create a comprehensive workflow execution plan before any other assignments begin

**Key Acceptance Criteria:**
- Dynamic workflow file read and understood
- Every workflow assignment traced and read
- All plan_docs/ documents read
- Workflow execution plan produced with all required sections
- Plan presented for approval
- Plan committed to `plan_docs/workflow-plan.md`

**Project-Specific Notes:**
- Currently executing this assignment
- Must account for existing comprehensive planning docs in plan_docs/
- Must identify that this is a template repo with existing infrastructure
- Must clarify relationship between existing plan_docs/ and new GitHub Issue plan

**Prerequisites:** None

**Dependencies:** None (first assignment)

**Risks/Challenges:**
- Agent may be confused by extensive existing documentation
- May not understand self-bootstrapping nature of OS-APOW
- May not recognize template repository context

**Events:** None

---

### Main Assignments

#### 1. init-existing-repository

**Goal:** Initialize the repository by creating a new branch, configuring GitHub settings (branch protection, project, labels), and establishing the administrative structure

**Key Acceptance Criteria:**
- New branch created: `dynamic-workflow-project-setup`
- Branch protection ruleset imported from `.github/protected-branches_ruleset.json`
- GitHub Project created and linked to repository
- Project columns created: Not Started, In Progress, In Review, Done
- Labels imported from `.github/.labels.json`
- Workspace and devcontainer files renamed to match project name
- PR created from branch to main (stays open until final assignment)

**Project-Specific Notes:**
- This is a template repository - many files already exist
- Branch protection import requires `GH_ORCHESTRATION_AGENT_TOKEN` with `administration:write` scope
- Project creation requires `GITHUB_TOKEN` with `project` scope
- Template placeholder `workflow-orchestration-queue-india42` must be replaced with actual repo name
- Existing labels in `.github/.labels.json` include OS-APOW-specific labels (agent:queued, agent:in-progress, etc.)
- PR will remain open throughout all subsequent assignments

**Prerequisites:**
- GitHub authentication with appropriate scopes
- Access to create GitHub Projects
- Permissions for repository administration

**Dependencies:** None (first main assignment)

**Risks/Challenges:**
- Branch protection import may fail due to insufficient permissions
- Project creation may fail if project scope missing from token
- Template placeholder replacement may be missed
- PR creation requires at least one commit first

**Mitigation:**
- Run `scripts/test-github-permissions.ps1` before starting
- Verify `GH_ORCHESTRATION_AGENT_TOKEN` and `GITHUB_TOKEN` are set correctly
- Ensure at least one commit is made before attempting PR creation

**Events:**
- **post-assignment-complete:** validate-assignment-completion, report-progress

---

#### 2. create-app-plan

**Goal:** Create a comprehensive application plan documented as a GitHub Issue, based on the existing planning documents in `plan_docs/`

**Key Acceptance Criteria:**
- Application template (plan_docs/) thoroughly analyzed
- Plan documented in GitHub Issue using application-plan.md template
- Milestones created for implementation phases
- Issue linked to GitHub Project
- Issue assigned to appropriate milestone (typically "Phase 1: Foundation")
- Appropriate labels applied (planning, documentation)
- NO implementation code written (planning only)
- tech-stack.md and architecture.md created in plan_docs/ (if not present)

**Project-Specific Notes:**
- `plan_docs/` already contains extensive planning documents - these should be referenced, not recreated
- The GitHub Issue plan should summarize and link to existing docs, not duplicate them
- Focus should be on Phase 1 (Sentinel MVP) implementation
- Tech stack is already defined: Python 3.12+, FastAPI, uv, httpx, Pydantic, Docker
- Architecture is already defined in Architecture Guide v3.2
- This is PLANNING ONLY - no code implementation

**Prerequisites:**
- Repository initialized (assignment #1)
- Access to create GitHub Issues and Milestones

**Dependencies:**
- Should wait for assignment #1 to complete (needs project structure)
- Can technically run in parallel but logically should follow

**Risks/Challenges:**
- Agent may be confused by existing comprehensive docs and try to recreate them
- Agent may attempt to implement code instead of just planning
- Agent may not focus on Phase 1 specifically
- May not understand that plan_docs/ are reference material, not targets

**Mitigation:**
- Explicitly state that plan_docs/ are reference material
- Emphasize PLANNING ONLY - no implementation
- Clarify that GitHub Issue should summarize and link to existing docs
- Direct focus to Phase 1 (Sentinel MVP) implementation

**Events:**
- **pre-assignment-begin:** gather-context
- **on-assignment-failure:** recover-from-error
- **post-assignment-complete:** report-progress

---

#### 3. create-project-structure

**Goal:** Create the actual project scaffolding including solution structure, configuration files, Docker setup, CI/CD foundation, and documentation structure

**Key Acceptance Criteria:**
- Solution/project structure created following Python + uv tech stack
- All required project files and directories established
- Initial configuration files created (pyproject.toml, .python-version, etc.)
- Docker and docker-compose.yml configured
- Basic CI/CD pipeline structure established in .github/workflows/
- Documentation structure created (README.md, docs/, etc.)
- Development environment properly configured
- Repository summary document created (.ai-repository-summary.md)
- Initial commit made with complete scaffolding
- All GitHub Actions workflows have SHA-pinned actions
- Stakeholder approval obtained

**Project-Specific Notes:**
- **Tech Stack:** Python 3.12+ with uv package manager (NOT pip/poetry)
- **Project Type:** FastAPI async web service + background polling service
- **Structure should follow Implementation Spec v1.2:**
  ```
  workflow-orchestration-queue/
  ├── pyproject.toml
  ├── uv.lock
  ├── src/
  │   ├── notifier_service.py
  │   ├── orchestrator_sentinel.py
  │   ├── models/
  │   │   ├── work_item.py
  │   │   └── github_events.py
  │   └── queue/
  │       └── github_queue.py
  ├── scripts/
  │   ├── devcontainer-opencode.sh
  │   ├── gh-auth.ps1
  │   └── update-remote-indices.ps1
  ├── local_ai_instruction_modules/
  ├── .devcontainer/
  └── docs/
  ```
- **Docker Healthcheck:** Must use Python stdlib, NOT curl (base image may not have curl)
  ```yaml
  healthcheck:
    test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"]
  ```
- **Existing Files:** scripts/ and .devcontainer/ already exist - preserve them
- **Reference Implementations:** plan_docs/src/ contains reference code - use as guide, don't copy directly
- **CI/CD:** GitHub Actions workflows must use SHA-pinned actions per project-setup directive

**Prerequisites:**
- Repository initialized (assignment #1)
- Application plan created (assignment #2)

**Dependencies:**
- Requires assignment #1 complete (needs initialized repository)
- Should follow assignment #2 (understand what's being built)

**Risks/Challenges:**
- Agent may default to .NET/C# structure instead of Python
- May use pip/poetry instead of uv
- Docker healthcheck may use curl incorrectly
- May overwrite existing scripts/ and .devcontainer/ files
- May not preserve template repository structure
- GitHub Actions may not use SHA-pinned actions

**Mitigation:**
- Explicitly specify Python + uv stack in instructions
- Reference Implementation Spec v1.2 for correct structure
- Warn about Docker healthcheck curl issue
- Emphasize preserving existing scripts/ and .devcontainer/
- Require SHA pinning validation for all workflows

**Events:**
- **post-assignment-complete:** validate-assignment-completion, report-progress

---

#### 4. create-agents-md-file

**Goal:** Create an AGENTS.md file at the repository root that provides AI coding agents with the context and instructions they need to work effectively on the project

**Key Acceptance Criteria:**
- AGENTS.md file exists at repository root
- Contains project overview, tech stack, setup commands
- Contains project structure / directory layout
- Contains code style and conventions
- Contains testing instructions
- Contains PR / commit guidelines
- All listed commands have been validated
- File is committed and pushed to working branch
- Stakeholder approval obtained

**Project-Specific Notes:**
- **Target Audience:** AI coding agents (Copilot, Codex, Aider, Cursor, etc.)
- **Complement Existing Docs:** Should complement README.md, not duplicate it
- **Tech Stack Commands:**
  - Install dependencies: `uv sync`
  - Run tests: `uv run pytest`
  - Lint: `uv run ruff check .`
  - Type check: `uv run mypy src/`
  - Run sentinel: `uv run python src/orchestrator_sentinel.py`
  - Run notifier: `uv run uvicorn src.notifier_service:app --reload`
- **Project Structure:** Reference the structure created in assignment #3
- **Key Conventions:**
  - Python 3.12+ with type hints
  - Async/await for I/O operations
  - Pydantic for data validation
  - Docstrings in Google/Sphinx format
  - Shell scripts use bash/pwsh (cross-platform)
  - All Actions pinned to SHA
- **Validation Challenge:** Project is newly scaffolded - some commands may not work without dependencies installed

**Prerequisites:**
- Repository initialized (assignment #1)
- Project structure created (assignment #3)

**Dependencies:**
- Requires assignment #3 complete (needs project structure to document)
- Should follow assignment #3 closely

**Risks/Challenges:**
- Commands may not be validated if dependencies not installed
- May duplicate content from README.md
- May document commands that don't exist yet
- May not account for uv-specific workflow

**Mitigation:**
- Document commands that SHOULD work, note validation status
- Reference README.md for user-focused content, keep AGENTS.md agent-focused
- Emphasize uv-specific commands
- Cross-reference with .ai-repository-summary.md

**Events:**
- **post-assignment-complete:** validate-assignment-completion, report-progress

---

#### 5. debrief-and-document

**Goal:** Perform a comprehensive debriefing session that captures key learnings, insights, deviations from plan, and areas for improvement from the completed project setup workflow

**Key Acceptance Criteria:**
- Detailed report created following structured template
- Report documented in .md file format
- All required sections complete (12 sections minimum)
- All deviations from assignments documented
- Report reviewed and approved by stakeholders
- Report committed and pushed to project repo
- Execution trace saved in repository (debrief-and-document/trace.md)
- Plan-impacting findings flagged as ACTION ITEMS with recommended follow-up

**Project-Specific Notes:**
- **Comprehensive Review:** This is a complex project setup with 6 assignments - expect many learnings
- **Execution Trace:** Capture all commands run, files created/modified, terminal output
- **Deviations:** Document any steps that weren't completed as specified
- **Action Items:** Per Simplification Report, several items were already identified:
  - Model unification (I-1/R-3)
  - Distributed locking implementation (I-2/R-2)
  - Exponential backoff (I-3)
  - Connection pooling (I-4/R-5)
  - Heartbeat implementation (I-6/R-1)
  - Credential scrubber (R-7)
  - Graceful shutdown (R-4)
  - Subprocess timeout (R-8)
- **Future Recommendations:** Should identify work for Phase 2 (Webhooks) and Phase 3 (Deep Orchestration)

**Prerequisites:**
- All previous assignments complete (1-4)

**Dependencies:**
- Must run after all main work assignments complete
- Should capture full execution history

**Risks/Challenges:**
- Report may be very large given complexity
- May not capture all deviations
- Execution trace may be incomplete
- May not identify actionable improvements

**Mitigation:**
- Use structured template to ensure completeness
- Review each assignment's acceptance criteria systematically
- Capture terminal output throughout execution
- Focus on actionable findings, not just observations

**Events:**
- **post-assignment-complete:** validate-assignment-completion, report-progress

---

#### 6. pr-approval-and-merge

**Goal:** Complete the full PR approval and merge process including resolving all PR comments, obtaining approval, merging the PR, and closing associated issues

**Key Acceptance Criteria:**
- All required CI/CD status checks pass before code review
- CI remediation loop executed (up to 3 attempts) if checks fail
- Code review delegated to code-reviewer subagent (NOT self-review)
- Auto-reviewer comments (Copilot, CodeQL) waited for and addressed
- PR review comment protocol executed (ai-pr-comment-protocol.md)
- All review comments resolved via GraphQL/UI
- Stakeholder/Delegating Agent approval obtained
- Merge performed successfully
- Source branch deleted
- Related issues/tickets closed
- Run report updated with final status

**Project-Specific Notes:**
- **Input Required:** `$pr_num` from assignment #1 output
- **Self-Approval Acceptable:** This is an automated setup PR - orchestrator self-approval is acceptable
- **CI Remediation:** Expect potential failures on first run (new project, untested configs)
- **Auto-Reviewers:** GitHub Copilot and CodeQL may flag issues in new code
- **Branch:** `dynamic-workflow-project-setup` → `main`
- **Post-Merge:** Delete setup branch, close setup issues
- **Critical:** All local changes MUST be committed and pushed BEFORE merge

**Prerequisites:**
- All previous assignments complete (1-5)
- All work committed to PR branch
- PR number available from assignment #1

**Dependencies:**
- Requires PR number from assignment #1
- Requires all work from assignments #1-5 to be committed
- Must wait for CI checks to complete

**Risks/Challenges:**
- CI may fail on first run (new project scaffolding)
- Auto-reviewers may flag many issues
- Review comments may require significant rework
- Merge conflicts if main branch changed
- May forget to commit local changes before merge

**Mitigation:**
- Allow up to 3 CI remediation cycles
- Address auto-reviewer comments systematically
- Follow ai-pr-comment-protocol.md strictly
- Verify all changes committed before requesting merge
- Capture GraphQL verification artifacts

**Events:**
- **post-assignment-complete:** validate-assignment-completion, report-progress

---

### Post-Assignment Events

#### validate-assignment-completion

**Goal:** Validate that each completed assignment has successfully met all its acceptance criteria through independent QA verification

**Key Acceptance Criteria:**
- All required files from assignment exist
- All verification commands pass (build, test, lint, etc.)
- Validation report created documenting results
- Pass/fail status determined
- If failed, specific remediation steps provided
- **Must be delegated to independent qa-test-engineer agent**

**Project-Specific Notes:**
- **Independent QA:** Must use qa-test-engineer, NOT the agent that did the work
- **GitHub Operations:** For GitHub state changes (issues, PRs, projects), query live GitHub state
- **Validation Commands:** Python-specific (uv sync, uv run pytest, uv run mypy, uv run ruff)
- **Report Location:** `docs/validation/VALIDATION_REPORT_<assignment-name>_<timestamp>.md`

**Prerequisites:**
- Assignment just completed
- Assignment acceptance criteria documented

**Dependencies:**
- Runs after each main assignment completes

**Risks/Challenges:**
- May not have independent agent available
- Validation commands may fail on newly scaffolded project
- GitHub state verification may require API access

**Mitigation:**
- Ensure qa-test-engineer agent is available
- Adjust validation commands for project state
- Use gh CLI for GitHub state queries

**Events:** None

---

#### report-progress

**Goal:** Provide progress reporting, output capture, and validation checkpoints after each workflow step completes successfully

**Key Acceptance Criteria:**
- Structured progress report generated with step name, duration, status
- All step outputs captured and recorded
- Step acceptance criteria validated
- Workflow state saved for resume-from-checkpoint
- Action items filed as GitHub issues (MANDATORY)
- User notification provided (optional)

**Project-Specific Notes:**
- **Progress Format:**
  ```
  === STEP COMPLETE: <assignment-name> ===
  Status: ✓ COMPLETE
  Duration: <elapsed-time>
  Outputs:
    - <key-output-1>: <value-or-location>
    - <key-output-2>: <value-or-location>
  Progress: X/6 (XX%)
  Next: <next-assignment-name>
  ```
- **Deviations & Findings:** REQUIRED section - document any deviations from plan
- **Plan-Impacting Discoveries:** REQUIRED section - new findings affecting subsequent work
- **Action Items:** MUST be filed as GitHub issues, not just noted
- **Labels:** Apply `priority:low` and `needs-triage` to filed issues

**Prerequisites:**
- Workflow step just completed successfully
- Access to step execution context

**Dependencies:**
- Runs after each main assignment and validate-assignment-completion

**Risks/Challenges:**
- May not capture all outputs systematically
- May skip filing GitHub issues for action items
- May not identify plan-impacting discoveries

**Mitigation:**
- Use structured template for progress reports
- Enforce GitHub issue filing for all action items
- Review upcoming assignments for continued validity

**Events:** None

---

### Post-Script Action

#### Apply orchestration:plan-approved Label

**Goal:** Signal that the project-setup workflow is complete and the plan is ready for epic creation

**Key Acceptance Criteria:**
- Locate the application plan issue created in assignment #2
- Apply label `orchestration:plan-approved` to that plan issue
- Record output as `#events.post-script-complete.plan-approved`

**Project-Specific Notes:**
- **Trigger:** This label triggers the next phase of orchestration pipeline
- **Target Issue:** The plan issue from assignment #2 (recorded as `#initiate-new-repository.create-app-plan`)
- **Label:** Must be `orchestration:plan-approved` exactly

**Prerequisites:**
- All assignments complete
- Plan issue exists

**Dependencies:**
- Requires plan issue number from assignment #2

**Risks/Challenges:**
- May not find the correct plan issue
- Label may not exist in repository

**Mitigation:**
- Verify label exists in .github/.labels.json
- Use recorded issue number from assignment #2

**Events:** None

---

## 4. Sequencing

### Execution Flow

```
[PRE-SCRIPT-BEGIN]
  └─> create-workflow-plan
      └─> Produce plan_docs/workflow-plan.md
      └─> Present for approval
      └─> Commit approved plan

[MAIN SCRIPT]
  └─> 1. init-existing-repository
      ├─> Create branch: dynamic-workflow-project-setup
      ├─> Import branch protection ruleset
      ├─> Create GitHub Project
      ├─> Import labels
      ├─> Rename workspace/devcontainer files
      └─> Create PR (OUTPUT: PR number)
      
      [POST-ASSIGNMENT-COMPLETE]
        ├─> validate-assignment-completion
        │   └─> Independent QA verification
        └─> report-progress
            └─> Capture outputs, file action items
      
  └─> 2. create-app-plan
      ├─> Analyze plan_docs/
      ├─> Create GitHub Issue with plan
      ├─> Create milestones
      └─> Link to project (OUTPUT: Plan issue number)
      
      [POST-ASSIGNMENT-COMPLETE]
        ├─> validate-assignment-completion
        └─> report-progress
      
  └─> 3. create-project-structure
      ├─> Create Python/uv project structure
      ├─> Configure Docker & docker-compose
      ├─> Set up development environment
      ├─> Create documentation structure
      ├─> Initialize CI/CD workflows (SHA-pinned)
      └─> Create .ai-repository-summary.md
      
      [POST-ASSIGNMENT-COMPLETE]
        ├─> validate-assignment-completion
        └─> report-progress
      
  └─> 4. create-agents-md-file
      ├─> Gather project context
      ├─> Validate build/test commands
      ├─> Draft AGENTS.md
      └─> Commit to working branch
      
      [POST-ASSIGNMENT-COMPLETE]
        ├─> validate-assignment-completion
        └─> report-progress
      
  └─> 5. debrief-and-document
      ├─> Create debrief report (12 sections)
      ├─> Document deviations
      ├─> Capture execution trace
      └─> File action items as GitHub issues
      
      [POST-ASSIGNMENT-COMPLETE]
        ├─> validate-assignment-completion
        └─> report-progress
      
  └─> 6. pr-approval-and-merge (INPUT: PR number from #1)
      ├─> [Phase 0.5] CI Verification & Remediation (up to 3 cycles)
      ├─> [Phase 0.75] Code Review Delegation & Auto-Reviewer Wait
      ├─> [Phase 1] Resolve Review Comments (ai-pr-comment-protocol.md)
      ├─> [Phase 2] Secure Approval
      ├─> [Phase 3] Merge Execution
      └─> Post-merge: Delete branch, close issues
      
      [POST-ASSIGNMENT-COMPLETE]
        ├─> validate-assignment-completion
        └─> report-progress

[POST-SCRIPT-COMPLETE]
  └─> Apply orchestration:plan-approved label to plan issue
```

### Dependency Graph

```
create-workflow-plan
    ↓
init-existing-repository (OUTPUT: PR #, Project ID, Branch)
    ↓
    ├─> create-app-plan (OUTPUT: Plan Issue #)
    │       ↓
    └─> create-project-structure
            ↓
        create-agents-md-file
            ↓
        debrief-and-document
            ↓
        pr-approval-and-merge (INPUT: PR # from init)
            ↓
        Apply orchestration:plan-approved (INPUT: Plan Issue #)
```

### Parallelization Opportunities

- **Limited parallelization:** Most assignments have sequential dependencies
- **Potential parallel:** create-app-plan could theoretically run parallel with create-project-structure, but logically should follow to understand what's being built
- **Post-assignment events:** validate-assignment-completion and report-progress can run in parallel after each main assignment

---

## 5. Open Questions

### Q1: Relationship Between Existing plan_docs/ and New GitHub Issue Plan

**Question:** The `plan_docs/` directory already contains comprehensive planning documents (Development Plan v4.2, Implementation Spec v1.2, Architecture Guide v3.2, Plan Review, Simplification Report). Should the `create-app-plan` assignment create a new comprehensive plan, or create a GitHub Issue that summarizes and links to these existing documents?

**Recommendation:** Create a GitHub Issue that serves as the "plan tracker" - it should summarize the existing comprehensive plans, link to the relevant documents in `plan_docs/`, define the Phase 1 (Sentinel MVP) scope, and establish milestones. The Issue should NOT duplicate the extensive content already in `plan_docs/`.

**Impact:** Affects how `create-app-plan` assignment is executed and what deliverables are expected.

---

### Q2: Handling Reference Implementations in plan_docs/src/

**Question:** The `plan_docs/src/` directory contains reference implementations (orchestrator_sentinel.py, notifier_service.py, models/work_item.py, queue/github_queue.py). Should `create-project-structure` copy these files into the actual `src/` directory, or create fresh scaffolding?

**Recommendation:** Create fresh scaffolding in `src/` that follows the structure and patterns from the reference implementations, but don't directly copy the files. The reference implementations should guide the structure and demonstrate patterns (e.g., connection pooling, assign-then-verify locking, heartbeat implementation), but the actual implementation should be done in subsequent phases. The `create-project-structure` assignment is about scaffolding, not full implementation.

**Impact:** Affects what files are created in assignment #3 and whether reference code is treated as production-ready or educational.

---

### Q3: Validating Build/Test Commands in AGENTS.md

**Question:** The `create-agents-md-file` assignment requires validating all listed commands, but this is a newly scaffolded project. Should commands be documented if they can't be fully validated yet?

**Recommendation:** Document the commands that SHOULD work based on the project structure (e.g., `uv sync`, `uv run pytest`, `uv run mypy src/`), but include a note in AGENTS.md that full validation requires dependency installation and may need adjustment. The goal is to provide accurate guidance for future AI agents, not to have a fully functional system at the end of project-setup.

**Alternative:** Run `uv sync` to install dependencies during assignment #3 or #4 to enable command validation.

**Impact:** Affects whether AGENTS.md contains validated commands or "best effort" documentation.

---

### Q4: Template Placeholder Replacement Scope

**Question:** The repository name in many files references "workflow-orchestration-queue-india42" (the template repo name). How extensively should template placeholders be replaced with the actual repository name?

**Current Scope:** Assignment #1 step 5 only renames:
- `.devcontainer/devcontainer.json` name property
- `ai-new-app-template.code-workspace` → `<repo-name>.code-workspace`

**Potential Additional Replacements:**
- AGENTS.md references
- README.md references
- Documentation references
- Shell script references

**Recommendation:** For project-setup, focus only on the explicitly required replacements in assignment #1. Additional placeholder replacements can be identified in the debrief and addressed in subsequent work.

**Impact:** Affects how thorough the template-to-project transition is during setup.

---

### Q5: CI/CD Workflow Scope for project-setup

**Question:** What CI/CD workflows should be established during `create-project-structure`? The OS-APOW system will eventually need several workflows (build, test, lint, security scan, Docker build, etc.), but should all of these be created during scaffolding?

**Recommendation:** Create minimal CI/CD scaffolding focused on:
1. **validate.yml** - Basic build, lint, test workflow (Python/uv specific)
2. **publish-docker.yml** - Docker image build and publish (referenced in Architecture Guide)
3. **prebuild-devcontainer.yml** - DevContainer prebuild (referenced in Architecture Guide)

Additional workflows (security scanning, deployment, etc.) can be added in subsequent phases. All workflows MUST use SHA-pinned actions per the dynamic workflow directive.

**Impact:** Affects the scope of assignment #3 and what validation is possible.

---

## 6. Summary

This workflow execution plan provides a comprehensive roadmap for executing the `project-setup` dynamic workflow on the OS-APOW repository. The plan covers:

- **1 pre-script event** to create this execution plan
- **6 main assignments** to initialize the repository, create planning artifacts, scaffold the project structure, document for AI agents, debrief on learnings, and merge the setup PR
- **12 post-assignment event executions** (2 events × 6 assignments) for validation and progress reporting
- **1 post-script action** to signal plan approval

**Total execution steps:** 20

**Key Success Factors:**
1. Understanding that this is a template repository with existing infrastructure
2. Recognizing that `plan_docs/` contains reference material, not implementation targets
3. Using Python 3.12+ with uv package manager (not pip/poetry)
4. Preserving existing shell scripts and devcontainer configurations
5. Ensuring all GitHub Actions use SHA-pinned versions
6. Completing thorough validation and debrief to capture learnings

**Next Steps:**
1. Obtain stakeholder approval of this workflow execution plan
2. Commit approved plan to `plan_docs/workflow-plan.md`
3. Begin execution of `init-existing-repository` assignment

---

**Plan Status:** Approved  
**Prepared By:** Planner Agent  
**Date:** 2026-04-06  
**Workflow:** project-setup  
**Repository:** intel-agency/workflow-orchestration-queue-india42
