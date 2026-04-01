# Claude Cortex

Context management, project guidelines, and code review for AI coding assistants — all in one plugin.

Three systems that work independently or together:

- **Context Framework** — Persistent project context that prevents AI assistants from losing track between sessions
- **Project Guidelines** — Define and enforce coding standards with automated compliance reviews
- **Code Review** — Structured, security-first code reviews with merge recommendations

Compatible with **Claude Code** and **GitHub Copilot**.

## The Guides

| Guide | What it covers |
|-------|---------------|
| [Context Framework](docs/context-framework.md) | What CDS is, how `.context/` works, daily workflow, templates |
| [Project Guidelines](docs/guidelines.md) | Simple vs full mode, severity system, review heuristics, subagent |
| [Code Review](docs/code-review.md) | Severity tiers, testability assessment, output format |

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
/context-init            # Create .context/ framework + bootloaders
/guidelines-init         # Generate PROJECT_GUIDELINES.md
```

### Daily use

```bash
/context                 # Load project context at session start
/code-review             # Review current changes
/guidelines-review --pr  # Check PR against project guidelines
/context-checkpoint      # Save session progress
```

## Skills

| Command | What it does |
|---------|-------------|
| `/context-init` | Scaffold `.context/` directory, `CLAUDE.md`, and `.github/copilot-instructions.md` with auto-detected project details |
| `/context` | Load context at session start, update status at session end |
| `/context-checkpoint` | Create a timestamped session checkpoint in `.context/CHECKPOINTS/` |
| `/guidelines-init` | Generate `PROJECT_GUIDELINES.md` — simple (flat rules) or `--full` (categorized + severity) |
| `/guidelines-review` | Review code against guidelines. Scopes: `--pr`, `--staged`, `--changed`, file, or directory |
| `/code-review` | Security-first code review with 4 severity tiers, testability analysis, and merge verdict |

## What's Inside

```
claude-cortex/
|-- .claude-plugin/
|   |-- plugin.json              # Plugin manifest
|-- skills/
|   |-- context-init/             # /context-init — scaffold context framework
|   |-- context/                 # /context — load/update session context
|   |-- context-checkpoint/      # /context-checkpoint — save session snapshot
|   |-- guidelines-init/         # /guidelines-init — generate guidelines
|   |-- guidelines-review/       # /guidelines-review — compliance review
|   |-- code-review/             # /code-review — structured code review
|-- agents/
|   |-- reviewer.md              # Subagent for parallel guideline reviews (51+ files)
|-- templates/
|   |-- context/                 # Templates for .context/ files
|       |-- CLAUDE.md            # Bootloader template (Claude Code)
|       |-- copilot-instructions.md  # Bootloader template (GitHub Copilot)
|       |-- CURRENT_STATUS.md    # Project status template
|       |-- ARCHITECTURE.md      # System design template
|       |-- CONVENTIONS.md       # Coding standards template
|       |-- DECISIONS.md         # ADR template
|       |-- MASTER_PLAN.md       # Roadmap template
|       |-- SETUP.md             # Dev setup template
|       |-- CHECKPOINT_TEMPLATE.md  # Checkpoint format
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

- **`/context`** loads your project's conventions → **`/code-review`** catches violations in real-time
- **`/guidelines-init`** codifies your standards → **`/guidelines-review`** enforces them automatically
- **`/context-checkpoint`** records what changed → **`/context`** picks it up next session

Future `/cortex` skills will combine these systems for deeper integration.

## Key Concepts

**Context Framework** stores structured project knowledge in `.context/` — status, architecture, conventions, decisions, and a master plan. Bootloader files (`CLAUDE.md` and `.github/copilot-instructions.md`) are read automatically by their respective AI tools, loading context without manual intervention. [Learn more →](docs/context-framework.md)

**Project Guidelines** come in two modes: **simple** (flat list of rules, no ceremony) and **full** (categorized with severity tags). Simple mode asks one question and generates a file. Full mode auto-detects conventions from 13+ signal files, runs a structured interview, and produces categorized guidelines. [Learn more →](docs/guidelines.md)

**Code Review** reads complete files (not just diffs), finds related tests, and reviews against four severity tiers: Critical (security), High (will cause errors), Medium (code quality), Low (style). Includes testability assessment and coverage gap analysis. [Learn more →](docs/code-review.md)

## FAQ

<details>
<summary>Do I need all three systems?</summary>

No. Each system is independent. Use `/context-init` + `/context` for context management alone, `/guidelines-init` + `/guidelines-review` for standards enforcement alone, or `/code-review` for standalone code reviews. They complement each other but don't require each other.
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
