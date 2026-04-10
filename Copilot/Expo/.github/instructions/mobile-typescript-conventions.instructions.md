---
description: "Small coding conventions for Expo, React Native, and shared TypeScript code in this monorepo."
applyTo: "**/*.{ts,tsx}"
---

# Mobile TypeScript Conventions

## General

- Use strict TypeScript and prefer explicit types at module boundaries.
- Prefer small, focused modules over large mixed-responsibility files.
- Keep platform-specific code in `apps/mobile` unless it is truly reusable.
- Keep reusable contracts and pure logic portable so they can later serve web too.

## React And React Native

- Prefer function components and hooks.
- Keep screens focused on composition and delegate reusable pieces to shared components or hooks when justified.
- Avoid deeply nested JSX by extracting meaningful UI units.
- Do not introduce ad hoc state containers unless the feature truly needs them.

## Naming

- Components, types, and interfaces: `PascalCase`
- Functions, variables, props, and hooks: `camelCase`
- Hooks must start with `use`
- Constants that are exported broadly should use clear `camelCase` or `PascalCase`, not opaque abbreviations

## Imports And Boundaries

- Prefer absolute or configured aliases when the project already supports them.
- Do not create circular dependencies between `apps/mobile` and shared packages.
- `packages/core` must stay platform-agnostic.
- `packages/api` should own transport-facing types, client boundaries, backend provider adapters, and remote content access.
- `packages/ui` should not contain mobile-only business rules.
- Do not spread direct backend vendor calls across screens or presentational components when a shared API boundary is more appropriate.

## Error Handling

- Validate inputs at boundaries.
- Do not swallow errors silently.
- Prefer typed result handling or explicit error states over hidden failures in UI flows.

## Security

- Never hardcode tokens, JWTs, secrets, private IDs, or credentials.
- Use placeholders in examples and docs.
- Treat any sensitive config handling as a reviewed boundary.
- Keep environment and backend configuration centralized so remote-update behavior stays explicit and auditable.

## Testing And Validation

- Keep pure logic easy to test outside the app shell.
- When tests are practical, prefer targeted coverage around changed behavior.
- Run lint and type-checks for the changed scope whenever practical.

## Style

- Follow existing repo formatting and lint rules.
- Prefer early returns over deep nesting.
- Remove unused imports, dead code, and starter leftovers when touching a file.