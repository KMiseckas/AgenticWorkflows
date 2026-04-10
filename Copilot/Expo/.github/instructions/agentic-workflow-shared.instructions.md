---
description: "Shared rules for the mobile planning, implementation, and review agents so behavior stays consistent across the workflow."
applyTo: ".github/agents/*.agent.md"
---

# Shared Agentic Workflow Rules

## Non-Negotiable

- Treat `requirements.md`, `design.md`, and `tasks.md` as source-of-truth artifacts.
- Keep the active implementation scope focused on `apps/mobile` for now.
- Design and review with future reuse across `packages/ui`, `packages/core`, `packages/api`, and `apps/web` in mind.
- Treat backend-connected, remotely updatable product behavior as the default direction when it improves delivery speed or content agility.
- Assume Supabase is the likely initial backend platform unless the user states otherwise, but keep integration boundaries portable.
- Ask only high-impact clarification questions that materially affect scope, correctness, validation, or security.
- Do not auto-start implementation unless the user explicitly approves that handoff.

## Security

- Never place secrets in committed source, config files, examples, or planning artifacts.
- Never commit auth tokens, refresh tokens, JWTs, API keys, client secrets, private IDs, or similar sensitive values.
- If a workflow needs configuration, prefer environment-based configuration and placeholder examples only.
- Treat security regressions and secret leaks as blocking issues.

## Completion Integrity

- A task is complete only when implementation and validation evidence are present.
- If tests cannot be run, document why and require the best available alternative evidence.
- If evidence is missing or contradictory, report `BLOCKED` instead of assuming completion.
- Call out whether the shipped change is backend-managed, OTA-eligible, or binary-only when that affects rollout expectations.

## Reuse Discipline

- Keep `apps/mobile` focused on composition, navigation, and mobile-specific behavior.
- Move reusable UI, domain logic, and typed API contracts into shared packages when that meaningfully reduces duplication.
- Do not extract to shared packages prematurely when the behavior is still speculative or single-use.
- Prefer typed backend and content-delivery boundaries in `packages/api` over scattering provider SDK usage through screens.
- Prefer remotely managed content, configuration, and operational data over hardcoded app content when the product expects frequent updates.

## Output Discipline

- Keep outputs concise.
- Separate confirmed facts, assumptions, and open questions.
- Prefer minimal, scoped changes over broad rewrites.
- For reviews, keep findings first and avoid low-signal narration.
- Do not imply that all changes can avoid app store releases; native dependency, entitlement, and binary-level changes still require a new build.