---
description: "Architecture guidance for keeping current mobile work reusable across future mobile and web surfaces in this monorepo."
applyTo: "**/*.{ts,tsx}"
---

# Shared Platform Guidance

## Current Delivery Scope

- Build for `apps/mobile` first.
- Validate design assumptions against both iPhone and Android.
- Keep future web compatibility in mind without forcing premature web abstractions.

## Package Placement Rules

- Put screen-specific composition, routing, gestures, and device-only behavior in `apps/mobile`.
- Put reusable presentational components, tokens, and cross-platform UI primitives in `packages/ui` when they are stable enough to reuse.
- Put business rules, data shaping, validation, and non-UI logic in `packages/core`.
- Put API contracts, endpoint clients, DTO mapping, transport wrappers, backend provider adapters, and remote content access in `packages/api`.

## Reuse Heuristics

- If the same logic or component is likely to be used by more than one screen or platform, prefer a shared package.
- If a piece is still highly screen-specific or experimental, keep it local until the shape is clearer.
- Favor interfaces and typed props that can later map to web without carrying mobile-only assumptions.
- Favor backend-managed content and configuration for data that should change without a binary release.

## Design Guidance

- Design UI as a component system, not as isolated screens.
- Keep domain models independent from transport models where that separation adds clarity.
- Prefer typed adapters at app boundaries rather than leaking API response shapes directly into screens.
- Assume Supabase is the likely initial backend choice unless the user specifies another platform, but isolate vendor-specific code behind typed `packages/api` boundaries.
- Distinguish clearly between remotely updatable behavior, OTA-safe JavaScript changes, and native or binary changes that still require store distribution.
- Prefer remote content, feature flags, and operational configuration for frequently changing product content when that improves delivery efficiency.

## Security Reminder

- Shared packages must never contain real secrets or private credentials.
- Authentication or secret-handling design must use secure configuration boundaries and placeholder values only in source.