---
description: "Shared rules for custom planning, development, and review agents so the workflow stays aligned and evidence-based."
applyTo: ".github/agents/*.agent.md"
---

# Shared Agentic Workflow Rules

## Non-Negotiable

- Treat `requirements.md`, `design.md`, and `tasks.md` as source-of-truth artifacts.
- Do not skip evidence-based gates: completion claims must be backed by explicit validation output.
- Do not auto-start implementation unless the user explicitly approves that handoff.
- Ask only high-impact clarification questions that materially affect scope, correctness, or validation.

## Completion Integrity

- A task is complete only when implementation, validation, and review-gate evidence are present.
- If evidence is missing or contradictory, report `BLOCKED` (or equivalent blocked state) instead of assuming completion.
- If test execution is not possible, document why and require the best available alternative evidence.
- For Unity-related work, distinguish between `instructions prepared` and `manual editor work executed`. Do not collapse those states into one claim.

## Output Discipline

- Keep outputs concise and findings-first for review workflows.
- Separate confirmed facts, assumptions, and open questions.
- Prefer minimal, scoped changes over broad rewrites.

## Unity Workflow Rules

- Do NOT use editor automation or live scene inspection as part of this workflow.
- Do NOT read, write, or edit `.unity` or `.prefab` files directly.
- Do NOT infer live scene or prefab structure from repository files, screenshots, or stale notes.
- If `## Unity Spec` is absent or incomplete in `design.md` for Unity-affecting work, stop and report `BLOCKED`.
- Treat `design.md` `## Unity Spec` as the source of truth for required Unity-side setup.
- Treat `.github/tasks/<feature-slug>/unity_instructions.md` as the user-facing execution guide derived from design and task scope.
- The workflow's Unity deliverable is accurate guidance, not editor mutation. Agents may prepare instructions and optional helper scripts, but must not claim the Unity editor work was executed unless the user explicitly confirms it.
- Optional Unity editor scripts are allowed only when they reduce repetitive or error-prone setup. They must be documented in `unity_instructions.md`, exposed under `Tools/<feature-slug>/...`, and described with a manual alternative when feasible.
- If a manual editor change conflicts with design or task artifacts, update design and the affected planning/instruction documents first. Do not treat the manual scene state as the source of truth.
