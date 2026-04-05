---
description: "Concise project overview so agents can understand scope, architecture, and constraints without scanning the repository. Agents fill this out as they work."
applyTo: "**"
---

# [Project Name] Project Overview

## What This Project Is

[Short description of what the project does, its primary purpose, and intended consumers.]

## Current Repository State

- [Summarize what is currently implemented — main systems, folder structure, test status.]

## Key Paths

- `.github/instructions/`: reusable workflow and project guidance.
- `.github/agents/`: custom planner/developer/reviewer agents.
- `.github/tasks/<feature-slug>/`: `requirements.md`, `design.md`, `tasks.md`.
- [Add project-specific source, test, and docs paths as the project grows.]

## Dependencies And Target Versions

- [List runtime and tooling dependencies, target framework versions, and compatibility constraints.]

## Project Goals

- [List the primary goals and capabilities the project aims to deliver.]

## Non-Goals

- [List what this project explicitly does not do.]

## Systems In Action

- [Describe major runtime subsystems as they are built out.]

## API Layer Summary

- [Document the public API surface as it is defined and stabilized.]

## Critical Constraints

- [List hard constraints: compatibility requirements, performance rules, architectural invariants.]

## Implementation Direction

- [Capture active implementation notes and decisions that affect day-to-day development.]
