---
description: "Unity-specific coding conventions, MonoBehaviour rules, editor-script guidance, and core vs Unity-layer separation guidelines."
applyTo: "**/*.cs"
---

# Unity C# Conventions

## Naming

- Types (classes, structs, interfaces, enums, delegates): `PascalCase`
- Methods and properties: `PascalCase`
- Public fields: `PascalCase`
- Private and protected fields: `_camelCase` (underscore prefix)
- Local variables and parameters: `camelCase`
- Constants: `PascalCase`
- Interfaces: prefix with `I` — `IMyInterface`
- Generic type parameters: `T`, or descriptive `TPascalCase` for multi-parameter generics
- Prefer descriptive names over abbreviations

## Formatting

- Follow project `.editorconfig` where present
- Indent with spaces (4 per level)
- Opening braces on the same line for methods and types
- One blank line between members; no trailing blank lines at end of file
- Keep lines within a reasonable width (120 characters)
- One type per file; file name matches type name

## Code Style

- Use `var` when the type is obvious from the right-hand side; use explicit types otherwise
- Use `nameof(...)` instead of string literals for member references
- Prefer expression-bodied members for single-expression methods and properties
- Use `== null` (not `is null`) for `UnityEngine.Object` types — Unity overrides the `==` operator
- For non-Unity types, prefer `is null` / `is not null` over `== null`
- Avoid unnecessary casts; prefer pattern matching for non-Unity types
- Keep methods short and focused on a single responsibility
- Avoid deep nesting; prefer early returns

## Architecture: Core vs Unity Layer

- Separate all logic into two layers:
  - **Core**: pure C#, no `UnityEngine` dependency, fully testable outside Unity
  - **Unity Layer**: MonoBehaviours, adapters, bindings — depends on Core, not the reverse
- Core must not reference `UnityEngine`
- Unity layer adapts Core → Unity (initialization, event wiring, serialization)
- Design always defines this split explicitly in `## Unity Spec`

## MonoBehaviour Rules

- Do not use constructors on `MonoBehaviour` or `ScriptableObject` — use `Awake`, `OnEnable`, or explicit init methods
- Do not cache references to other `MonoBehaviour`s in field initializers; cache in `Awake`
- Null-check Unity objects with `== null`, not `?.` — `?.` bypasses Unity's destroyed-object detection
- Do not call Unity APIs from non-main threads
- Keep `Update`, `FixedUpdate`, and `LateUpdate` implementations minimal; delegate work to Core
- Prefer `[SerializeField] private` fields over public fields for Inspector exposure

## Scene and Prefab Policy

- Do NOT hand-edit `.unity` or `.prefab` YAML files
- Do NOT reconstruct scene or prefab structure by reading raw Unity YAML in agents
- Describe required Unity setup in `design.md` (`## Unity Spec`) and user-facing execution steps in `.github/tasks/<feature-slug>/unity_instructions.md`
- Treat user-performed editor setup as external to the agent workflow unless the user explicitly reports the result
- If a manual editor change conflicts with design or `unity_instructions.md`, update the design and instructions first

## Editor Script Policy

- Unity editor scripts are optional helpers, not the primary source of truth
- Use editor scripts only when they reduce repetitive or error-prone setup work
- Place editor scripts under an `Editor` folder and expose entry points under `Tools/<feature-slug>/...`
- Keep editor scripts idempotent where practical so users can re-run them safely
- Document what each editor script changes and the manual alternative in `unity_instructions.md`

## Documentation

- XML doc comments required for all public types, methods, properties, and events
- Keep XML docs concise — describe _what_ and _why_, not _how_
- Use `<summary>`, `<param>`, `<returns>`, and `<exception>` tags where applicable
- Internal and private members do not require XML docs unless the logic is non-obvious

## Error Handling

- Validate inputs at public API boundaries; throw `ArgumentException`/`ArgumentNullException` for invalid inputs
- Do not use exceptions for normal control flow
- Prefer returning structured result types for expected failure cases
- Catch only exceptions you can meaningfully handle; avoid broad `catch (Exception)`
- Do not swallow exceptions silently
- Avoid exceptions in hot paths (Update, physics callbacks)

## Performance

- Avoid allocations in `Update`, `FixedUpdate`, and physics callbacks
- Cache `GetComponent` results in `Awake`; never call in `Update`
- Do not use LINQ in runtime hot paths
- Avoid closures in tight loops
- Avoid `string` concatenation in per-frame code; use `StringBuilder` or avoid entirely
- Profile before optimizing; do not prematurely pessimize readable code

## IL2CPP and AOT Safety

- Use C# 8 compatible features only (Unity 2021+)
- Do NOT use `System.Reflection.Emit` or runtime code generation
- Do NOT use `dynamic`
- All generic types used must be explicitly referenced somewhere (avoid stripping)
- Avoid interface-based generic dispatch in performance-critical paths
- Avoid code paths that rely on runtime type discovery
- Avoid boxing/unboxing

## Dependencies

- Minimize external dependencies
- All dependencies must be Unity-compatible and IL2CPP-safe
- Document the purpose and rationale for each dependency added

## Testing

- Core logic must be testable outside Unity — no `MonoBehaviour` in Core
- Separate logic from Unity-specific behaviour
- Name tests: `MethodOrScenario_Condition_ExpectedOutcome`
- Each test validates one behavior

## General Rules

- Do not commit commented-out code
- Remove unused usings and imports
- Avoid `static` mutable state; if required, document lifecycle and domain-reload reset behavior explicitly
