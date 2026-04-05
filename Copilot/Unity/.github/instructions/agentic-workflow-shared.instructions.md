---
description: "Shared rules for custom planning/development/review agents to reduce duplication and keep behavior consistent."
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

## Output Discipline

- Keep outputs concise and findings-first for review workflows.
- Separate confirmed facts, assumptions, and open questions.
- Prefer minimal, scoped changes over broad rewrites.

## Unity Hard Gates

- Do NOT infer scene or prefab structure. If `## Unity Spec` is absent or incomplete in `design.md`, stop and report `BLOCKED`.
- Do NOT read, write, or edit `.unity` or `.prefab` files directly. Use MCP Unity tools or generated editor scripts only.
- `UnityState.md` is read-only for all agents except `Unity Integrator`. No agent may write to it except `Unity Integrator`.
- Design is the source of truth. The Unity scene is a derived artifact of Design — not the reverse.
- If a manual editor change is discovered that conflicts with Design, update Design first, then re-run `Unity Integrator`.
