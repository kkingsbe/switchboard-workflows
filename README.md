# Switchboard Workflows

Pre-built prompt systems for [Switchboard](https://github.com/kkingsbe/switchboard-rs-oss) — coordinated multi-agent pipelines you can install and run with a single command.

---

## What are workflows?

A **workflow** is a collection of agent prompts designed to work together. Where a single Switchboard agent performs one task on a schedule, a workflow orchestrates multiple agents that coordinate through file-based contracts — passing state, delegating subtasks, and building on each other's output.

Each workflow ships as a directory of prompt files and a README explaining the agent roles, scheduling recommendations, and any prerequisites.

## Available Workflows

| Workflow | Agents | Description |
|----------|--------|-------------|
| **[bmad](bmad/)** | 8 | Full BMAD-method implementation — sprint planning, parallel development, code review, and knowledge curation from planning artifacts |
| **[development](development/)** | 5 | General-purpose development pipeline with architect planning and parallel dev agents |
| **[codebase-maintenance](codebase-maintenance/)** | 3 | Automated auditing, improvement planning, and refactoring of existing code |
| **[qa](qa/)** | 2 | Quality assurance testing and automated fix generation |
| **[documentation](documentation/)** | 1 | Codebase summarization and documentation generation |

## Quick Start

### Install a workflow

```bash
switchboard workflows install bmad
```

This copies the workflow's prompt files into your project at `.switchboard/prompts/workflows/<name>/`.

### Add agents to your config

Each workflow README includes a recommended `switchboard.toml` snippet. For example, after installing the `development` workflow:

```toml
[settings]
image_name = "switchboard-agent:latest"

[[agent]]
name = "architect"
schedule = "15 */2 * * *"
timeout = "30m"
prompt_file = ".switchboard/prompts/workflows/development/ARCHITECT.md"
env = { AGENT_COUNT = "2" }

[[agent]]
name = "dev-1"
schedule = "*/30 * * * *"
timeout = "30m"
prompt_file = ".switchboard/prompts/workflows/development/DEV_PARALLEL.md"
env = { AGENT_ID = "1" }

[[agent]]
name = "dev-2"
schedule = "10,40 * * * *"
timeout = "30m"
prompt_file = ".switchboard/prompts/workflows/development/DEV_PARALLEL.md"
env = { AGENT_ID = "2" }
```

### Run

```bash
switchboard validate
switchboard up
```

## Workflow Structure

Every workflow follows the same layout:

```
workflow-name/
├── README.md          # Overview, agent roles, config snippet, prerequisites
├── input/             # (optional) Input files or templates the workflow expects
│   └── .gitkeep
└── prompts/
    ├── AGENT_ONE.md   # Prompt file for each agent role
    ├── AGENT_TWO.md
    └── ...
```

Prompt files are referenced via `prompt_file` in your `switchboard.toml`. Each agent prompt defines its own coordination protocol — what state files it reads, what it writes, and how it signals completion to downstream agents.

## CLI Commands

```bash
switchboard workflows list              # List available workflows
switchboard workflows install <name>    # Install a workflow into your project
switchboard workflows info <name>       # Show details about a workflow
```

## Workflow Details

### bmad

Implements the [BMAD method](https://github.com/bmad-code-org/BMAD-METHOD) for Switchboard. Expects planning artifacts (PRD, architecture docs, epics) in `_bmad-output/planning-artifacts/` and orchestrates sprint-based development through eight coordinated agents: architect, solution architect, sprint planner, parallel dev agents, code reviewer, scrum master, knowledge curator, and work chronicler.

### development

A lighter-weight development pipeline for projects that don't need the full BMAD ceremony. An architect agent breaks work into tasks, then parallel dev agents pick up and implement them. Includes both standard and feature-branch architect variants.

### codebase-maintenance

Continuous code health automation. An auditor scans for issues and technical debt, an improvement planner prioritizes and organizes the findings, and refactor agents execute the planned improvements. Designed to run alongside other workflows without conflict.

### qa

Two-agent quality assurance loop. A QA agent runs tests and identifies failures, then a fix agent addresses the discovered issues. Useful as a standalone pipeline or layered on top of development workflows.

### documentation

A single summarizer agent that generates and maintains project documentation from your codebase. Runs on a daily or weekly schedule to keep docs in sync with code changes.

## Creating Your Own Workflow

1. Create a new directory with a `README.md` and a `prompts/` subdirectory
2. Write one prompt file per agent role in `prompts/`
3. Document the agent coordination protocol — which state files each agent reads and writes
4. Include a recommended `switchboard.toml` snippet in your README with scheduling and env vars
5. Add an `input/` directory if your workflow expects seed files or templates

### Design Principles

**File-based coordination** — Agents communicate through state files and signal files (e.g., `.spec_ready`, `.agent_done_1`). Keep JSON schemas explicit in every prompt that reads or writes shared state.

**Idempotent resumption** — Every agent should detect its current phase from existing state files and resume correctly after interruption. Never assume a clean slate.

**Explicit negative guardrails** — Tell agents what *not* to do. "Do NOT search for missing files — fail immediately if the expected input doesn't exist" prevents agents from silently working around problems.

**Task sizing** — Design tasks to complete within a single agent session (~15 minutes). Rich upfront context in TODO items pays off more than vague descriptions that require agents to re-orient.

## Contributing

We welcome new workflows. To contribute:

1. Fork this repository
2. Create your workflow directory following the structure above
3. Include a thorough README with agent roles, scheduling recommendations, and a config snippet
4. Open a pull request

## License

MIT