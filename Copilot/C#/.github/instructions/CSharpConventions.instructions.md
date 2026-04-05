---
description: "General C# coding conventions, naming rules, formatting standards, and documentation expectations."
applyTo: "**/*.cs"
---

# C# Conventions

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
- Prefer `is null` / `is not null` over `== null` for standard C# types
- Avoid unnecessary casts; prefer pattern matching
- Keep methods short and focused on a single responsibility
- Avoid deep nesting; prefer early returns

## Documentation

- XML doc comments are required for all public types, methods, properties, and events
- Keep XML docs concise — describe *what* and *why*, not *how*
- Use `<summary>`, `<param>`, `<returns>`, and `<exception>` tags where applicable
- Internal and private members do not require XML docs unless the logic is non-obvious

## Error Handling

- Validate inputs at public API boundaries; throw `ArgumentException`/`ArgumentNullException` for invalid inputs
- Do not use exceptions for normal control flow
- Prefer returning structured result types (bool, enum, result struct) for expected failure cases
- Catch only exceptions you can meaningfully handle; avoid broad `catch (Exception)`
- Do not swallow exceptions silently

## Performance

- Avoid unnecessary allocations in hot paths
- Cache frequently accessed values rather than recomputing them
- Avoid closures in tight loops
- Prefer `Span<T>` and `Memory<T>` over array slices where appropriate
- Profile before optimizing; do not prematurely pessimize readable code

## Dependencies

- Minimize external dependencies; prefer solving problems in-project when the solution is simple and maintainable
- Document the purpose and rationale for each dependency added
- Keep dependencies up to date and ensure they are actively maintained

## Testing

- Keep logic testable — minimize dependencies on infrastructure, I/O, or platform APIs in core code
- Prefer pure functions and dependency injection over static global state
- Name tests: `MethodOrScenario_Condition_ExpectedOutcome`
- Each test should validate one behavior

## General Rules

- Do not commit commented-out code
- Remove unused usings and imports
- Do not use `#region` unless the codebase already uses them consistently
- Avoid `static` mutable state; if required, document lifecycle and reset behavior explicitly
