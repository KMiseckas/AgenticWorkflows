# Contributing

Contributions are welcome — new workflow variants, agent improvements, or bug fixes.

## How to contribute

1. Fork the repo and create a branch (`feat_your-change`).
2. Make your changes. Keep agents focused and scoped; avoid combining unrelated concerns in one agent file.
3. Test manually by running the workflow in a real project before submitting.
4. Open a pull request with a short description of what changed and why.

## Adding a new workflow variant

- Create a new folder under `Copilot/` (e.g. `Copilot/Python`).
- Copy an existing `.github/` structure as a starting point.
- Update `instructions/` files for the target language/framework.
- Document the new variant in the root `README.md`.

## Guidelines

- Keep agent instructions concise — agents read them on every run.
- Don't duplicate rules that already live in `agentic-workflow-shared.instructions.md`; reference or extend them instead.
- Prefer explicit over implicit — agents should never have to guess what to do next.
