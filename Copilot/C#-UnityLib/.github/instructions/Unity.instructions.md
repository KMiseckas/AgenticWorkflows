---
description: "Guidelines for the Unity integration layer — MonoBehaviours, scene bindings, Unity lifecycle, and integration validation. Complements UnityCsharpLib.instructions.md which covers the engine-agnostic C# core layer."
applyTo: "**/*.cs"
---

# Unity Integration Layer Guidelines

## Layer Responsibility

The Unity integration layer connects the engine-agnostic C# core to the Unity runtime. Its sole role is to:

- Wire Unity lifecycle events (`Awake`, `Start`, `OnEnable`, `OnDisable`, `OnDestroy`) to core `Initialize` / `Shutdown` calls
- Bind serialized scene fields and configuration to core data structures
- Route Unity input and engine callbacks into core logic
- Expose core results back to Unity (UI updates, audio triggers, animation, effects)

The integration layer must **not** contain business logic. Logic belongs in the core layer.

## MonoBehaviour Rules

- Keep MonoBehaviours thin — lifecycle wiring and delegation only
- Call `Initialize(...)` on core objects from `Awake`; call `Shutdown()` from `OnDestroy`
- Do not duplicate or re-implement core logic inside a MonoBehaviour
- Cache component and object references in `Awake`; never call `GetComponent` in `Update` or other per-frame callbacks
- Avoid `FindObjectOfType`, tag lookups, or name-based searches at runtime; resolve dependencies via serialized fields

## Scene Binding Rules

- Serialize all scene bindings via `[SerializeField]` — do not resolve objects by name, tag, or layer at runtime
- Keep the scene hierarchy minimal and purposeful; avoid hidden runtime state embedded in the scene
- Document scene setup expectations in `docs/` whenever a feature requires a specific object hierarchy, component configuration, or prefab arrangement

## Unity Lifecycle Rules

- All `UnityEngine` API calls must happen on the main thread
- Do not rely on constructor initialization for `MonoBehaviour` or `ScriptableObject`; use explicit initialization methods instead
- Do not rely on Unity's default script execution order without an explicit `DefaultExecutionOrder` attribute or documented rationale
- Null-check `UnityEngine.Object` references with `== null`, not `is null`; standard null semantics do not apply to Unity objects

## Validation Approach

The Unity integration layer (MonoBehaviours, scene bindings) is currently validated via:

- **`UnityState.md` snapshot comparison** — record expected state (active objects, component field values, binding wires) before and after the task; flag any deviations as `BLOCKED`
- **Manual PlayMode spot checks** — verify the targeted integration behavior in-editor before task completion

This is the accepted evidence ceiling for integration-layer task completion. `taskReviewer` must not block integration-layer tasks solely for lacking Unity Test Runner output.

## Future Milestone: Unity Test Runner Integration

Unity Test Runner (EditMode and PlayMode automated tests) is a **planned Phase 2 validation gate** and is not part of the current completion criteria for integration-layer tasks.

When Unity Test Runner integration is added:

- **EditMode tests** will cover MonoBehaviour lifecycle and scene binding correctness without requiring Play mode
- **PlayMode tests** will cover full game-loop behavior and coroutine-driven flows using `[UnityTest]`
- The `taskReviewer` gate will require Unity Test Runner output as required evidence for integration-layer task approval

Until then, `UnityState.md` snapshot comparison and manual PlayMode spot checks are the required and sufficient evidence for integration-layer task completion.

## Documentation

- Document scene setup requirements in `docs/` for any feature that requires a specific scene configuration, object hierarchy, or prefab arrangement
- Update `.github/instructions/projectOverview.instructions.md` when integration-layer subsystems are added or structurally changed
