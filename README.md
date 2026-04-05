# AgenticWorkflows

Ready-to-use GitHub Copilot agentic workflow templates. Drop them into any project to get a structured, multi-agent pipeline that takes a feature idea all the way to reviewed, committed code.

---

## Why use this?

Copilot agents are powerful but unstructured by default — they invent architecture on the fly, skip steps, and don't produce reviewable artifacts. These workflows fix that by enforcing a strict pipeline:

```
Feature Vision → Requirements → Design → Tasks → Implementation → Review
```

Each stage is handled by a dedicated agent. Agents hand off through plain markdown artifacts (`requirements.md`, `design.md`, `tasks.md`) stored in your repo. No stage starts until the prior one's output exists and is approved.

This gets you:
- **Consistent, auditable AI output** — every decision is captured in a file, not just chat history.
- **Scope control** — agents ask targeted questions instead of guessing.
- **Clean git history** — one commit per task, every time.
- **Built-in QA** — a reviewer agent validates each task before it's marked done.

---

## What's included

### Workflows

| Folder | Target project type |
|---|---|
| `Copilot/C#` | General C# projects |
| `Copilot/C#-UnityLib` | C# Unity library projects |
| `Copilot/Unity` | Unity game projects (includes Unity Editor MCP integration) |

Each workflow lives under `.github/` and is self-contained. Copy only the one you need.

### Agents (per workflow)

| Agent | What it does |
|---|---|
| **Feature Vision** | Explores the project roadmap, surfaces gaps, updates `docs/vision.md` |
| **Requirements Planner** | Turns an idea into a reviewable `requirements.md`, creates the feature branch |
| **Design Planner** | Turns requirements into a `design.md` with architecture, data flow, and a review contract |
| **Task Planner** | Turns the design into an ordered, commit-aligned `tasks.md` |
| **Task Creator** | Orchestrator — runs Requirements → Design → Tasks in one go |
| **Task Developer** | Implements tasks sequentially, one commit per task |
| **Task Reviewer** | QA decision per task and final full-pass review |
| **Unity Integrator** *(Unity only)* | Applies Unity scene/prefab changes via MCP Unity tools |

### Supporting files

- **`instructions/`** — Coding conventions, memory, and project context loaded automatically by agents.
- **`prompts/remember.prompt.md`** — `/remember` shortcut to append notes to the project memory file.

---

## Usage

1. Pick the workflow that matches your project type (`C#`, `C#-UnityLib`, or `Unity`).
2. Copy its `.github/` folder into the root of your project repo.
3. Fill in `instructions/projectOverview.instructions.md` with your project's name, goals, and key paths.
4. In GitHub Copilot Chat, invoke any agent by name — start with **Task Creator** for a new feature, or **Feature Vision** to plan what to build next.

### Typical flow

```
# Plan a new feature end-to-end
@Task Creator  "Add a health bar UI component"

# Or step through manually
@Requirements Planner  "Add a health bar UI component"
@Design Planner
@Task Planner
@Task Developer
@Task Reviewer
```

Artifacts are committed to `.github/tasks/<feature-slug>/` on the feature branch as the pipeline runs.

---

## License

[MIT](LICENSE) — use freely, modify, redistribute. Attribution appreciated but not required.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
