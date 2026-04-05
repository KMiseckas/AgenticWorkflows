---
description: "Governs the UnityState.md scene snapshot: format, ownership, update rules, and agent read policy."
applyTo: ".github/**"
---

# UnityState.md Rules

## What It Is

`UnityState.md` is a generated, agent-readable snapshot of the live Unity scene and prefab state. It is the minimum representation needed to validate that the Unity Editor matches `design.md` (`## Unity Spec`).

It lives at: `docs/UnityState.md`

## Ownership

- **Written by**: `Unity Integrator` only, after each task's `[Unity]` phase completes.
- **Read by**: `task-reviewer`, `Unity Integrator` (for diffing prior state), and `Task Developer` (for context).
- **Never hand-edited** — if the Editor state changes manually, update `design.md` first, then re-run `Unity Integrator` to regenerate this file.

## Format

One section per Scene, one section per Prefab. Use only:

- Root GameObject names
- Component list (script/component type names only, in square brackets)
- Direct children as indented list items (depth capped at 2 levels unless Design explicitly requires more)

No GUIDs, no fileIDs, no transform data, no material or asset references.

```md
## Scene: <SceneName>
- <GameObjectName> [<Component>, <Component>]
  - <ChildName>
  - <ChildName>

## Prefab: <PrefabName>
- Root [<Component>, <Component>]
  - <ChildName>
  - <ChildName>
```

### Example

```md
## Scene: MainGame
- GameManager [GameManager]
- Player [PlayerController, PlayerHealth, Rigidbody, CapsuleCollider]
  - CameraPivot
  - WeaponSocket
- UI [Canvas]

## Prefab: Player
- Root [PlayerController, PlayerHealth, Rigidbody, CapsuleCollider]
  - CameraPivot
  - WeaponSocket
```

## Update Rules

- `Unity Integrator` regenerates the entire file after each task's Unity phase — no partial updates.
- If a scene or prefab is removed from the project, remove its section from the snapshot.
- Depth beyond 2 levels is only written when the corresponding `## Unity Spec` entry explicitly lists those children.

## Agent Read Policy

- Agents reading `UnityState.md` use it to answer: "Does the live scene match the Design?"
- Do not treat `UnityState.md` as the source of truth — `design.md` (`## Unity Spec`) is.
- If `UnityState.md` is missing or stale, report `BLOCKED` rather than inferring scene state.

## Staleness Detection

`Unity Integrator` records a timestamp comment at the top of the file:

```md
<!-- Generated: <ISO8601 timestamp> after task: <task-number> <task-title> -->
```

`task-reviewer` checks this timestamp against the last task completion time. If the snapshot predates the most recent Unity task, flag it as potentially stale.
