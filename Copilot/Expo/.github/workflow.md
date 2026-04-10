# Mobile Agentic Workflow

This document describes the workspace-level Copilot workflow defined under `.github/` for this monorepo.

It is modeled after the reference agentic workflow, but adapted for a mobile-first Expo project with future web support and shared `ui`, `core`, and `api` packages.

## Overview

The workflow is a staged pipeline:

```text
[Feature Vision] -> [Planning] -> [Implementation] -> [Final Review]
```

Current implementation work is limited to `apps/mobile` and related shared packages. Web is intentionally deferred, but every planning and design step should preserve future reuse where practical.

## File Structure

```text
.github/
├── agents/
│   ├── feature-vision.agent.md
│   ├── mobile-requirements-planner.agent.md
│   ├── mobile-design-planner.agent.md
│   ├── mobile-task-planner.agent.md
│   ├── mobile-task-creator.agent.md
│   ├── mobile-task-developer.agent.md
│   └── mobile-task-reviewer.agent.md
├── instructions/
│   ├── agentic-workflow-shared.instructions.md
│   ├── projectOverview.instructions.md
│   ├── mobile-typescript-conventions.instructions.md
│   └── mobile-shared-platform.instructions.md
└── tasks/
    └── <feature-slug>/
        ├── requirements.md
        ├── design.md
        └── tasks.md
```

Related product documentation:

```text
docs/
└── vision.md
```

## Agents

### Feature Vision

- Purpose: maintain `docs/vision.md` and help shape product direction
- Scope: mobile-first roadmap with a separate future web vision
- Output: updates to `docs/vision.md`

### Mobile Requirements Planner

- Purpose: turn a requested feature into `.github/tasks/<feature-slug>/requirements.md`
- Scope: `apps/mobile` first, shared package implications included where relevant
- Branch ownership: creates and checks out the feature branch

### Mobile Design Planner

- Purpose: turn requirements into `.github/tasks/<feature-slug>/design.md`
- Scope: package boundaries, data flow, UI reuse, API contract shape, testing strategy, and security notes

### Mobile Task Planner

- Purpose: turn requirements and design into `.github/tasks/<feature-slug>/tasks.md`
- Scope: ordered tasks, validation gates, shared package coverage, docs sync, and one final review handoff

### Mobile Task Creator

- Purpose: orchestrate requirements, design, and task planning in one run
- Scope: planning only, no implementation

### Mobile Task Developer

- Purpose: implement tasks sequentially with focused validation and one commit per completed task
- Difference from the reference flow: it does not default to reviewer handoff after every task

### Mobile Task Reviewer

- Purpose: run one efficient final pass across the completed feature
- Secondary mode: review a single task when explicitly requested

## Why The Review Flow Is Different

The reference workflow performs more frequent review handoffs during implementation. This mobile workflow is intentionally leaner:

- Per-task validation still happens inside `Mobile Task Developer`
- `Mobile Task Reviewer` is optimized for a single combined pass near the end
- This reduces token usage and review churn while still preserving an independent QA check before the feature is considered complete

## Shared Instructions

### agentic-workflow-shared.instructions.md

- Shared non-negotiable rules for all custom agents
- Enforces evidence-based completion, security hygiene, and mobile-first scope

### projectOverview.instructions.md

- Snapshot of the repo structure, active surfaces, package roles, and constraints

### mobile-typescript-conventions.instructions.md

- Small code conventions for Expo, React Native, and shared TypeScript code

### mobile-shared-platform.instructions.md

- Guidance on when logic belongs in `apps/mobile` versus `packages/ui`, `packages/core`, and `packages/api`

## Product And Architecture Expectations

- Mobile work must consider both iPhone and Android behavior because the active app ships through Expo
- UI should be designed as reusable components where that creates real cross-platform value
- API integration should be designed for reuse across platforms
- Security is non-negotiable: no secrets, auth tokens, JWTs, API keys, or private identifiers in code, commits, or planning artifacts

## Recommended Usage

1. Use `Feature Vision` when the product direction or roadmap needs refinement.
2. Use `Mobile Task Creator` to generate `requirements.md`, `design.md`, and `tasks.md` for a feature.
3. Start `Mobile Task Developer` only after the task plan is approved.
4. Run `Mobile Task Reviewer` after implementation is complete, or earlier only if a targeted checkpoint is needed.
