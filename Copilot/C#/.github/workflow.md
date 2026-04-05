# C# Agentic Workflow

This document describes the overall agentic workflow defined under `Copilot/C#/.github/`. It explains every agent, instruction file, and prompt — what each one does, how they interact, what artifacts they produce, and why the flow is structured the way it is.

---

## Overview

The workflow is a structured, multi-stage pipeline that takes a feature idea from raw intent all the way to reviewed, committed code. It is split into four main phases:

```
[Feature Vision] → [Planning] → [Implementation] → [Review]
```

Each phase is handled by one or more agents. Agents are coordinated through shared artifact files stored in `.github/tasks/<feature-slug>/`. The pipeline enforces a strict handoff sequence: no stage begins until the prior stage's artifact exists and is usable.

---

## File Structure

```
.github/
├── agents/
│   ├── feature-vision.agent.md        # Feature discovery and vision documentation
│   ├── requirements-planner.agent.md  # Turns intent into a requirements document
│   ├── design-planner.agent.md        # Turns requirements into a technical design
│   ├── task-planner.agent.md          # Turns design into an ordered task plan
│   ├── task-creator.agent.md          # Wrapper: runs requirements → design → tasks in one go
│   ├── task-developer.agent.md        # Implements tasks from tasks.md sequentially
│   └── task-reviewer.agent.md         # Reviews task completion and final feature quality
├── instructions/
│   ├── agentic-workflow-shared.instructions.md   # Non-negotiable rules for all planning/dev/review agents
│   ├── memory.instructions.md                    # Durable project memory shared across all agent runs
│   ├── UnityCsharpLib.instructions.md            # C#/Unity coding guidelines applied to all .cs files
│   └── projectOverview.instructions.md           # Project-wide context snapshot for agent orientation
├── prompts/
│   └── remember.prompt.md             # /remember shortcut: appends a note to memory.instructions.md
└── tasks/
    └── <feature-slug>/                # Created per feature during planning
        ├── requirements.md
        ├── design.md
        └── tasks.md
```

---

## Agents

### Feature Vision

**File:** `agents/feature-vision.agent.md`

**Purpose:** Helps the user discover, define, and refine what features to build. This is an optional but recommended first step when starting a new feature or revisiting project direction.

**What it does:**
- Reads `docs/vision.md` to understand the current feature roadmap.
- Asks targeted questions when intent is unclear.
- Searches the web for similar projects and common patterns (e.g. Unity command consoles) to surface expected or missing features.
- Proposes additions or changes to `docs/vision.md` and confirms with the user before editing.
- Keeps feature descriptions at the "what", not the "how".

**When to use it:** When you have a rough idea but are unsure what to build, when you want to identify gaps in the current plan, or when you want to update the project roadmap.

**Output:** Updated `docs/vision.md` — the living feature roadmap for the project.

**Interacts with:** No other agents. This is a standalone discovery step.

---

### Requirements Planner

**File:** `agents/requirements-planner.agent.md`

**Purpose:** Converts a proposed feature or task into a clear, reviewable requirements document. This is the entry point for all planning work.

**What it does:**
- Asks focused clarifying questions to surface ambiguities in scope, acceptance conditions, and constraints.
- Determines the branch type prefix (`feat_`, `bug_`, `refactor_`) and derives a kebab-case `<feature-slug>`.
- Creates and checks out the feature branch (branching from `origin/main` when available).
- Writes `.github/tasks/<feature-slug>/requirements.md` using a fixed document structure.
- Commits and pushes the requirements file to the feature branch.

**Why it creates the branch:** The requirements stage owns branch creation so that all subsequent planning artifacts (design, tasks) and implementation work are committed to the correct branch from the start.

**Output:** `.github/tasks/<feature-slug>/requirements.md` and the feature branch.

**Interacts with:** Invoked by **Task Creator** as step 1. Output consumed by **Design Planner** and **Task Planner**.

---

### Design Planner

**File:** `agents/design-planner.agent.md`

**Purpose:** Turns an approved requirements document into a concrete technical design that implementation agents and reviewers can use directly.

**What it does:**
- Reads `requirements.md` and inspects the relevant codebase area.
- Asks clarifying questions only where the answer will materially change the architecture.
- Produces `.github/tasks/<feature-slug>/design.md` with architecture decisions, component responsibilities, data flow, code snippets, and a testing strategy.
- Includes a final review contract — a checklist of critical behaviors, design invariants, required test evidence, and blocking conditions — for `taskReviewer` to use during the final full-pass review.
- Evaluates whether third-party dependencies are needed; defaults to no new dependency unless the in-project solution would be error-prone.
- Commits and pushes the design file to the existing feature branch.

**Why design comes before tasks:** Tasks must map to concrete design decisions. Without a design, task agents would need to invent architecture on the fly, leading to inconsistent or incorrect implementations.

**Output:** `.github/tasks/<feature-slug>/design.md`.

**Interacts with:** Invoked by **Task Creator** as step 2. Output consumed by **Task Planner** and **Task Developer**.

---

### Task Planner

**File:** `agents/task-planner.agent.md`

**Purpose:** Converts the approved requirements and design into an ordered, commit-aligned implementation plan.

**What it does:**
- Reads both `requirements.md` and `design.md`.
- Splits work into numbered tasks where each task delivers a distinct, independently-validatable increment and targets one commit.
- Each task includes: objective, inputs, implementation steps, validation checks (including a `taskReviewer` handoff block), documentation sync expectations, a completion gate checklist, and a suggested commit message.
- Verifies full coverage: every requirement and every key design component must be mapped to at least one task.
- Ensures documentation update tasks are included whenever behavior, API, or architecture changes.
- Ensures `projectOverview.instructions.md` sync is planned when project-level facts are affected.
- After writing `tasks.md`, explicitly asks the user: *"Start implementation with Task Developer now?"* — never auto-starts implementation.

**Output:** `.github/tasks/<feature-slug>/tasks.md`.

**Interacts with:** Invoked by **Task Creator** as step 3. Output consumed by **Task Developer**.

---

### Task Creator

**File:** `agents/task-creator.agent.md`

**Purpose:** An orchestrating wrapper that runs the three planning agents (Requirements Planner → Design Planner → Task Planner) in sequence in a single invocation.

**What it does:**
- Accepts a feature description from the user.
- Invokes `Requirements Planner`, verifies branch creation, then invokes `Design Planner`, then invokes `Task Planner`.
- Validates consistency between artifacts at each stage transition: branch names must match, design must trace to requirements, tasks must cover both.
- Stops and reports if any stage fails or produces unusable output, then asks whether to retry before moving on.
- After all three artifacts exist and are consistent, presents a summary and asks whether to start implementation with `Task Developer`.

**When to use it:** When you want to go from a feature idea to a ready-to-implement task plan in one go, without invoking each planning agent manually.

**Does not:** Write code, start implementation, or skip any stage.

**Interacts with:** Orchestrates **Requirements Planner**, **Design Planner**, and **Task Planner**. Hands off to **Task Developer** only with explicit user approval.

---

### Task Developer

**File:** `agents/task-developer.agent.md`

**Purpose:** Implements the tasks in `tasks.md` sequentially, one commit per completed task.

**What it does:**
- Reads `requirements.md`, `design.md`, `tasks.md`, all `instructions/*.instructions.md` files, and `/docs/**` for project context.
- Confirms the active branch matches the planned branch before touching any code.
- Asks the user once at the start to choose an execution mode:
  - `auto-all`: runs all tasks end-to-end without pausing.
  - `confirm-each-task`: pauses after each task and waits for a go/no-go before continuing.
- For each task: implements the scoped objective, runs the task's validation steps, then stages and commits with the suggested commit message.
- Invokes `taskReviewer` when the task's completion gate requires a QA quick pass. Does not mark the task complete if `taskReviewer` returns `BLOCKED`.
- Updates `tasks.md` when reality requires a plan amendment, without silently changing scope.
- Adds or updates Doxygen-style XML comments for public APIs as required.

**Why one commit per task:** Keeps git history clean and makes it easy to bisect, revert individual changes, or review incremental progress during code review.

**Interacts with:** Reads artifacts produced by all three planning agents. Invokes **taskReviewer** for per-task QA passes.

---

### taskReviewer

**File:** `agents/task-reviewer.agent.md`

**Purpose:** Provides a QA decision on whether a task increment is truly complete, or whether the full feature is acceptable for final sign-off.

**What it does:**

**Task Quick Pass (during implementation):**
- Reviews a specific task number from `tasks.md`.
- Verifies the task objective is implemented, validation steps were executed, unit tests pass, completion-gate checkboxes are all checked, and no obvious conflict exists with requirements or design.
- Returns one of three decisions: `PASS`, `PASS WITH NOTES`, or `BLOCKED`.

**Final Full Pass (after all tasks are done):**
- Reviews the full feature against requirements coverage, design alignment, and the final review contract defined in `design.md`.
- Identifies remaining gaps and confirms the feature is ready for PR merge.

**Why a separate reviewer agent:** Separates the "did I write it" concern from the "is it correct and complete" concern. The reviewer applies the review contract written during design, ensuring the final code is evaluated against the original intended behavior — not just against the implementation itself.

**Output:** A structured review report: Decision, Findings (severity-ordered), Gate Check Summary, Coverage Notes, and Recommended Next Step.

**Interacts with:** Invoked by **Task Developer** per-task, and optionally invoked directly by the user for final full-pass review.

---

## Instructions Files

Instructions files are automatically applied by the agent runtime based on their `applyTo` glob pattern. They provide persistent rules and context that every agent reads on every run — without requiring the user to re-state conventions each time.

### `agentic-workflow-shared.instructions.md`

**Applies to:** `.github/agents/*.agent.md`

**Purpose:** Contains non-negotiable rules shared across all planning, development, and review agents to keep behavior consistent.

**Key rules:**
- Treat `requirements.md`, `design.md`, and `tasks.md` as source-of-truth artifacts.
- Do not skip evidence-based gates — completion claims must be backed by explicit validation output.
- Do not auto-start implementation unless the user explicitly approves that handoff.
- Ask only high-impact clarifying questions that materially affect scope, correctness, or validation.
- If evidence is missing or contradictory, report `BLOCKED` instead of assuming completion.
- Keep outputs concise, findings-first, and prefer minimal scoped changes over broad rewrites.

**Why it exists:** Without shared rules, individual agents might apply inconsistent standards — one agent might approve a task without running tests, another might spam the user with low-value questions. This file enforces a common contract across the whole pipeline.

---

### `memory.instructions.md`

**Applies to:** `.github/**`

**Purpose:** Acts as durable project memory that persists across agent runs. Agents (especially `Task Developer`) write notes here when they discover new preferences, project rules, or recurring constraints that should influence future runs.

**Key contents:**
- Preferences for planning artifact quality and evidence-driven completion.
- Project-level rules (e.g. "do not mark tasks complete without completion-gate evidence").
- Conventions for keeping memory notes concise and non-duplicated.
- A "Do Not" section preventing misuse (no transient logs, no speculative rules).

**How to add to it:** Use the `/remember` prompt (see below), or `Task Developer` can append directly under a relevant heading.

**Why it exists:** Agents have no persistent memory across sessions by default. This file provides a structured way to accumulate project-specific facts that would otherwise be lost or need to be re-stated by the user every time.

---

### `UnityCsharpLib.instructions.md`

**Applies to:** `**/*.cs`

**Purpose:** Defines the technical coding standards and hard constraints for all C# source files in the project. Every agent that writes or reviews C# code must follow these rules.

**Key rules:**
- Target Unity 2021+ with C# 8 features only; must be IL2CPP/AOT safe.
- Hard bans: no LINQ in runtime code, no reflection-heavy logic, no dynamic or runtime code generation, no `System.Reflection.Emit`.
- Architecture: separate core (engine-agnostic) from the Unity integration layer; core must not depend on `UnityEngine`.
- API design: minimal stable public API, explicit lifecycle (`Initialize()` / `Shutdown()`), no hidden static initialization.
- Performance: avoid allocations in hot paths, avoid closures and virtual calls in tight loops.
- Error handling: validate at boundaries, avoid exceptions in hot paths, return structured result types.
- XML docs required for all public APIs.
- Core logic must be testable outside Unity.

**Why it exists:** As a compiled DLL library consumed by Unity projects, breaking API changes are costly, and IL2CPP/AOT failures are silent at authoring time but fatal at runtime on mobile/console platforms. These rules prevent common pitfalls before they reach code review.

---

### `projectOverview.instructions.md`

**Applies to:** `**`

**Purpose:** Provides a concise, always-current snapshot of the `kmCommands` project — its architecture, key paths, API surface, constraints, and current implementation state — so agents can orient themselves without scanning the entire repository.

**Key contents:**
- What the project is and what it explicitly does not do (no UI, no input handling).
- Current repository state: which systems are implemented, how many tests pass, what docs exist.
- Key source paths and their responsibilities.
- The complete public API layer summary.
- Critical constraints (IL2CPP safety, minimal allocations, explicit lifecycle, public API stability).
- The required source file header for new `src/` files.
- Docs index (`architecture.md`, `unity-integration.md`, `commands.md`).

**When it must be updated:** Any time project scope, architecture, folder structure, dependencies, API surface, or major implementation facts change — `Task Planner` enforces this as part of the documentation sync check in every task.

**Why it exists:** Without a reliable project overview, agents make incorrect assumptions about what already exists, re-invent patterns that are already established, or miss critical constraints like the IL2CPP ban on LINQ. This file is the single authoritative source of project context.

---

## Prompts

### `remember.prompt.md`

**Usage:** `/remember <what to remember>`

**Purpose:** A shortcut prompt that appends a note to `memory.instructions.md`. Agents (especially `Task Developer`) use this to save durable preferences, project rules, or recurring constraints discovered during implementation.

**What it does:**
1. Reads the current `memory.instructions.md`.
2. Appends the note under a matching heading (or creates a new one if needed).
3. Keeps entries to one concise bullet per fact.
4. Refuses to duplicate existing bullets — refines instead.
5. Moves stale notes to an archive file if `memory.instructions.md` grows beyond ~200 lines.

**Why it exists:** Allows any agent or the user to incrementally grow the project memory without manually editing the instructions file or risking overwriting existing content.

---

## Task Artifacts

Each feature produces three planning files under `.github/tasks/<feature-slug>/`:

| File | Produced by | Consumed by | Purpose |
|---|---|---|---|
| `requirements.md` | Requirements Planner | Design Planner, Task Planner, Task Developer, taskReviewer | Defines what must be accomplished, scope boundaries, acceptance criteria, and testing expectations |
| `design.md` | Design Planner | Task Planner, Task Developer, taskReviewer | Defines the technical approach, architecture decisions, code contracts, and the final review contract |
| `tasks.md` | Task Planner | Task Developer, taskReviewer | Ordered implementation plan with per-task completion gates, validation steps, and reviewer handoff blocks |

These files are the source of truth for the entire pipeline. No agent invents scope or design decisions outside these documents.

---

## Full Workflow

### When to start with Feature Vision

Use **Feature Vision** when you are exploring what to build, are uncertain about scope, or want to research common patterns before committing to a feature. It updates `docs/vision.md` — a separate document from the per-feature planning artifacts.

Feature Vision is optional. You can skip it and go directly to planning if the feature is already well-understood.

### Planning Phase

1. **Invoke Task Creator** (or invoke Requirements Planner, Design Planner, and Task Planner manually in order) with a feature description.
2. **Requirements Planner** clarifies scope, creates the feature branch, writes and commits `requirements.md`.
3. **Design Planner** reads `requirements.md`, inspects the codebase, writes and commits `design.md` with the review contract.
4. **Task Planner** reads both artifacts, writes and commits `tasks.md` with per-task completion gates and reviewer handoff blocks. Asks: *"Start implementation with Task Developer now?"*

After this phase, three committed planning artifacts exist on the feature branch. Implementation does not begin until the user approves.

### Implementation Phase

5. **Task Developer** is invoked with the path to `tasks.md` and an execution mode choice (`auto-all` or `confirm-each-task`).
6. For each task in order:
   - Implements the scoped objective.
   - Runs the task's validation steps.
   - If the task requires a QA quick pass, invokes **taskReviewer** with the task's review-request block.
   - Only marks the task complete when all completion-gate checkboxes are satisfied.
   - Commits with the suggested commit message and pushes to the remote.
7. If `taskReviewer` returns `BLOCKED`, Task Developer fixes findings before marking the task complete.

### Review Phase

8. After all tasks are marked complete, **taskReviewer** is invoked for a **Final Full Pass** against the complete feature: requirements coverage, design alignment, and the final review contract from `design.md`.
9. If all checks pass, the feature branch is ready for pull request and merge.

---

## Why This Flow

**Separation of concerns:** Each stage has a distinct, bounded responsibility. Requirements agents do not write architecture. Design agents do not write tasks. Task agents do not implement code. This prevents early-stage assumptions from leaking into later stages where they are harder to correct.

**Artifact-driven handoffs:** Every stage produces a file that the next stage reads. This makes the pipeline auditable — you can inspect exactly what each agent was working from, reproduce any stage independently, and identify where a misalignment was introduced.

**Mandatory gates:** Completion gates at every task prevent an agent from reporting "done" without evidence. This is enforced by both `Task Developer` (won't commit without gates) and `taskReviewer` (will return `BLOCKED` if gates are unmet). The shared instructions file reinforces this as a non-negotiable across all agents.

**User control at the planning-to-implementation boundary:** Task Creator and Task Planner both stop and ask the user before implementation begins. This prevents an agent from spending large amounts of compute on code changes based on a plan the user has not reviewed and approved.

**Incremental commits:** One commit per completed task keeps the git history readable, makes bisection easy, and ensures each commit leaves the codebase in a valid, reviewed state.

**Persistent instructions:** Rather than re-stating project rules on every invocation, the instructions files encode the project's constraints permanently. New agents, new sessions, and new contributors all start with the same shared context.

---

## Quick Reference: Who Does What

| Goal | Agent to invoke |
|---|---|
| Explore what to build or update the feature roadmap | Feature Vision |
| Turn a feature idea into requirements, design, and tasks in one go | Task Creator |
| Write only the requirements document | Requirements Planner |
| Write only the design document | Design Planner |
| Write only the task plan | Task Planner |
| Implement tasks from an existing tasks.md | Task Developer |
| Review a specific task or final feature quality | taskReviewer |
| Save a project rule or preference for future runs | `/remember` prompt |
