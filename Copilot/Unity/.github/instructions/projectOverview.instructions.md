---
description: "Concise Unity project overview so agents can understand where systems live, how scripts connect to Unity assets, and what constraints matter without scanning the repository. Agents fill this out as they work."
applyTo: "**"
---

# [Project Name] Project Overview

## What This Project Is

[Short description of what the project does, who it is for, and what kind of Unity project it is.]

## Current Repository State

- [Summarize what is currently implemented — active systems, major folders, editor tooling, and current test status.]

## Unity Project Shape

- [List the main script roots and what they contain: runtime code, editor scripts, tests, UI, ScriptableObjects, integration helpers, docs.]
- [Call out folders or assets that are user-managed and therefore likely to drift from documentation unless kept current.]

## Key Paths

- `.github/instructions/`: reusable workflow and project guidance.
- `.github/agents/`: custom planner/developer/reviewer agents.
- `.github/tasks/<feature-slug>/`: `requirements.md`, `design.md`, `tasks.md`, `unity_instructions.md`.
- [Add important runtime, editor, test, and docs paths with a one-line purpose for each.]

## Runtime Systems

### [System Name]

- Purpose: [What this system does at runtime.]
- Main scripts: [Where the core and Unity-layer code lives.]
- Unity touchpoints: [What scenes, prefabs, ScriptableObjects, UI documents, or input assets it depends on.]

## Editor Tooling And Setup

- [List any custom inspectors, editor windows, menu items, setup helpers, or validation tools and where they live.]
- [If a feature ships helper scripts for setup, note their menu path and intended use.]

## Dependencies And Target Versions

- [List Unity version, package dependencies, target C#/.NET level, test frameworks, and compatibility constraints.]

## Project Goals

- [List the primary goals and capabilities the project aims to deliver.]

## Non-Goals

- [List what this project explicitly does not do.]

## Systems In Action

- [Describe the major gameplay, UI, tooling, or data systems and how they interact.]

## Public Script Contracts

- [Document the public runtime/editor APIs, extension points, and script contracts other systems rely on.]

## Integration Hotspots

- [Describe recurring hookup points that matter for development: scene bindings, prefab wiring, UI setup, ScriptableObjects, input assets, addressables, save data, etc.]

## Critical Constraints

- [List hard constraints: compatibility requirements, performance rules, architectural invariants, and any manual Unity setup expectations that repeatedly matter.]

## Implementation Direction

- [Capture active implementation notes, known editor-side manual steps, and decisions that affect day-to-day development.]
