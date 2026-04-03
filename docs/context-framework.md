# Context Framework

Prevent Context Degradation Syndrome (CDS) when working with AI coding assistants across sessions.

## What is CDS?

Context Degradation Syndrome happens when an AI assistant loses track of project state between sessions — forgetting architecture decisions, repeating mistakes, or ignoring established conventions. This framework gives your AI assistant persistent context by storing structured project knowledge in files it reads automatically at the start of every session.

## How It Works

```
Your Project
├── CLAUDE.md               ← Claude Code reads this automatically
├── .github/
│   └── copilot-instructions.md  ← GitHub Copilot reads this automatically
├── .context/               ← Persistent project context
│   ├── CURRENT_STATUS.md   ← Updated every session
│   ├── ARCHITECTURE.md     ← System design
│   ├── CONVENTIONS.md      ← Tooling and environment setup
│   ├── DECISIONS.md        ← Architecture Decision Records
│   ├── MASTER_PLAN.md      ← Implementation roadmap
│   ├── SETUP.md            ← Dev environment setup
│   └── CHECKPOINTS/        ← Session summaries
└── ...
```

The bootloader files (`CLAUDE.md` and `.github/copilot-instructions.md`) are read automatically by their respective AI tools. They instruct the assistant to load the relevant `.context/` files, follow conventions, and update status at the end of each session. Both files contain the same instructions — `CLAUDE.md` targets Claude Code, while `.github/copilot-instructions.md` targets GitHub Copilot.

## Skills

| Command | Description |
|---------|-------------|
| `/context-init` | Initialize the framework in a new project |
| `/context-session` | Load context at session start, update at session end |

## Daily Workflow

**Start of session**: Run `/context` (or let it auto-invoke). The assistant reads context files and understands project state.

**During work**: The assistant follows `CONVENTIONS.md` for tooling setup and `PROJECT_GUIDELINES.md` for code quality rules, checks `DECISIONS.md` before proposing architectural changes, records new decisions when significant choices are made, and updates `PROJECT_GUIDELINES.md` when new patterns are adopted.

**End of session**: The assistant updates `CURRENT_STATUS.md`, checks whether `PROJECT_GUIDELINES.md` needs updating based on what changed, creates a checkpoint if the session was significant, and suggests a commit.

**Multi-machine sync**: Just `git pull`. Context travels with the code.

## File Reference

| File | Purpose | Update Frequency |
|------|---------|-----------------|
| `CLAUDE.md` | Bootloader for Claude Code | Every few sessions |
| `.github/copilot-instructions.md` | Bootloader for GitHub Copilot | Every few sessions |
| `.context/CURRENT_STATUS.md` | Where the project stands now | Every session |
| `.context/MASTER_PLAN.md` | Implementation roadmap | When phases change |
| `.context/ARCHITECTURE.md` | System design | When architecture evolves |
| `.context/DECISIONS.md` | Architecture Decision Records | When decisions are made |
| `.context/CONVENTIONS.md` | Tooling and environment setup | Rarely |
| `PROJECT_GUIDELINES.md` | Code quality rules (naming, architecture, testing, security) | When patterns evolve |
| `.context/SETUP.md` | Dev environment setup | Rarely |
| `.context/CHECKPOINTS/` | Session summaries | End of long sessions |

## What to Customize

After running `/context-init`, these files need your input:

1. **`CLAUDE.md` / `.github/copilot-instructions.md`** — Fill in `[ONE_SENTENCE_DESCRIPTION]` and the Current Focus section
2. **`.context/MASTER_PLAN.md`** — Define your implementation phases and goals
3. **`.context/CONVENTIONS.md`** — Document your language, tooling (formatter, linter, test runner), and build commands
4. **`PROJECT_GUIDELINES.md`** — Run `/guidelines-init` to create code quality rules: naming, architecture, error handling, testing standards, security, and performance
5. **`.context/ARCHITECTURE.md`** — Describe your system components and how they connect
6. **`.context/SETUP.md`** — Document how to set up the dev environment

You can also let your AI assistant fill these in — it will detect the `[PLACEHOLDER]` markers and offer to initialize them by exploring your codebase.

## Compatibility

| AI Tool | Bootloader | Status |
|---------|-----------|--------|
| Claude Code | `CLAUDE.md` | Fully supported |
| GitHub Copilot | `.github/copilot-instructions.md` | Fully supported |

The `.context/` directory and its files work with **any** AI coding assistant — the framework is just structured markdown. The bootloader files and skills are the delivery mechanism that makes it automatic.
