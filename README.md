# Claude Cortex

Context management, project guidelines, and code review for AI coding assistants — all in one plugin.

Three systems that work independently, but integrate deeply when used together:

- **Context Framework** — Persistent project context that prevents AI assistants from losing track between sessions
- **Project Guidelines** — Define and enforce coding standards with automated compliance reviews
- **Code Review** — Structured, security-first code reviews with merge recommendations

Compatible with **Claude Code** and **GitHub Copilot**.

## The Guides

| Guide | What it covers |
|-------|---------------|
| [Context Framework](docs/context-framework.md) | What CDS is, how `.context/` works, daily workflow, templates |
| [Project Guidelines](docs/guidelines.md) | Simple vs full mode, severity system, review heuristics, subagent |
| [Code Review](docs/code-review.md) | `/code-review` vs `/ult-code-review`, severity tiers, 10-agent suite, testability assessment |

## Quick Start

### Install (Claude Code)

```bash
# Marketplace
/plugin marketplace add xanderscannell/claude-cortex
/plugin install claude-cortex

# Or manual
git clone https://github.com/xanderscannell/claude-cortex.git
/plugin marketplace add ./claude-cortex
/plugin install claude-cortex
```
### Install (VS Code)

```bash
# Open the Command Palette (F1) and run:
Chat: Install Plugin From Source
# then point to this directory.
```


### Set up a project

```bash
/cortex-init             # Full setup: .context/ framework + PROJECT_GUIDELINES.md
# Or run them separately:
/context-init            # Create .context/ framework + bootloaders
/guidelines-init         # Generate PROJECT_GUIDELINES.md
```

### Daily use

```bash
/context-session         # Load project context at session start; update at session end
/code-review             # Review current changes
/guidelines-review --pr  # Check PR against project guidelines
```

## Skills

| Command | What it does |
|---------|-------------|
| `/cortex-init` | Full framework setup — runs `/context-init` then `/guidelines-init` in series, with a single combined commit |
| `/context-init` | Scaffold `.context/` directory, `CLAUDE.md`, and `.github/copilot-instructions.md` with auto-detected project details |
| `/context-session` | Load context at session start, update status and guidelines at session end |
| `/guidelines-init` | Generate `PROJECT_GUIDELINES.md` — simple (flat rules) or `--full` (categorized + severity) |
| `/guidelines-review` | Review code against guidelines. Scopes: `--pr`, `--staged`, `--changed`, file, or directory |
| `/code-review` | Security-first code review with 4 severity tiers, testability analysis, and merge verdict |
| `/ult-code-review` | 10 parallel specialist agents for near-complete bug and vulnerability coverage |

## What's Inside

```
claude-cortex/
|-- .claude-plugin/
|   |-- plugin.json              # Plugin manifest
|-- skills/
|   |-- cortex-init/             # /cortex-init — full framework setup
|   |-- context-init/             # /context-init — scaffold context framework
|   |-- context-session/         # /context-session — load/update session context
|   |-- guidelines-init/         # /guidelines-init — generate guidelines
|   |-- guidelines-review/       # /guidelines-review — compliance review
|   |-- code-review/             # /code-review — structured code review
|   |-- ult-code-review/         # /ult-code-review — 10-agent parallel review
|-- agents/
|   |-- reviewer.md              # Subagent for parallel guideline reviews (51+ files)
|-- templates/
|   |-- context/                 # Templates for .context/ files
|       |-- CLAUDE.md            # Bootloader template (Claude Code)
|       |-- copilot-instructions.md  # Bootloader template (GitHub Copilot)
|       |-- CURRENT_STATUS.md    # Project status template
|       |-- ARCHITECTURE.md      # System design template
|       |-- CONVENTIONS.md       # Tooling and environment setup template
|       |-- DECISIONS.md         # ADR template
|       |-- MASTER_PLAN.md       # Roadmap template
|       |-- SETUP.md             # Dev setup template
|-- references/
|   |-- guideline-categories.md  # Per-stack example rules (JS/TS, Python, Go, Rust)
|-- docs/
|   |-- context-framework.md     # Deep dive: context system
|   |-- guidelines.md            # Deep dive: guidelines system
|   |-- code-review.md           # Deep dive: code review
|-- README.md
|-- LICENSE
```

## How the Systems Connect

Each system works standalone, but they're stronger together:

- **`/cortex-init`** sets up the full framework in one command — context first, then guidelines, cross-referenced automatically
- **`/context-session`** loads conventions and guidelines at session start, keeps them updated as the codebase evolves
- **`/guidelines-init`** codifies your standards → **`/guidelines-review`** enforces them automatically
- **`/code-review`** and **`/ult-code-review`** catch universal best-practice violations in real-time — pair with **`/guidelines-review`** to also enforce project-specific rules

## Key Concepts

**Context Framework** stores structured project knowledge in `.context/` — status, architecture, tooling conventions, decisions, and a master plan. Bootloader files (`CLAUDE.md` and `.github/copilot-instructions.md`) are read automatically, loading context without manual intervention. Each session the AI also loads `PROJECT_GUIDELINES.md` for code quality rules and updates it as the codebase evolves. [Learn more →](docs/context-framework.md)

**Project Guidelines** come in two modes: **simple** (flat list of rules, no ceremony) and **full** (categorized with severity tags). Simple mode asks one question and generates a file. Full mode auto-detects conventions from 13+ signal files, runs a structured interview, and produces categorized guidelines. [Learn more →](docs/guidelines.md)

**Code Review** reads complete files (not just diffs), finds related tests, and reviews against four severity tiers: Critical (security), High (will cause errors), Medium (code quality), Low (style). Includes testability assessment and coverage gap analysis. [Learn more →](docs/code-review.md)

## FAQ

<details>
<summary>Do I need all three systems?</summary>

Not necessarily. Each system works standalone. Use `/context-init` + `/context-session` for context management alone, `/guidelines-init` + `/guidelines-review` for standards enforcement alone, or `/code-review` for standalone code reviews. That said, since v1.2.0 the context framework loads and updates `PROJECT_GUIDELINES.md` as part of its session lifecycle, so the two frameworks integrate deeply when used together. `/cortex-init` sets both up in one command.
</details>

<details>
<summary>Does this work with GitHub Copilot?</summary>

Yes. `/context-init` creates both `CLAUDE.md` (for Claude Code) and `.github/copilot-instructions.md` (for GitHub Copilot). The `.context/` directory is plain markdown — any AI tool can read it. The skills themselves require Claude Code.
</details>

<details>
<summary>What's the difference between /code-review and /guidelines-review?</summary>

`/code-review` is a general-purpose review — security, bugs, code quality, testability. It applies universal best practices. `/guidelines-review` checks code against *your specific project rules* defined in `PROJECT_GUIDELINES.md`. Use both for comprehensive coverage.
</details>

<details>
<summary>What's Context Degradation Syndrome (CDS)?</summary>

When an AI assistant loses track of project state between sessions — forgetting architecture decisions, repeating mistakes, or ignoring established conventions. The context framework prevents this by storing structured knowledge in files the AI reads automatically.
</details>

<details>
<summary>Can I customize the context templates?</summary>

Yes. After `/context-init` creates the files, edit them freely. The templates are starting points. The only requirement is that the bootloader files (`CLAUDE.md` / `.github/copilot-instructions.md`) keep the context-loading instructions so the AI knows to read `.context/` at session start.
</details>

## License

MIT
