# [PROJECT_NAME]

> [ONE_SENTENCE_DESCRIPTION]

## Context System

This project uses a context framework in `.context/` to prevent context degradation across sessions. You MUST follow these instructions every session.

### First-Time Setup (Placeholder Detection)

If you see `[PLACEHOLDER]` markers (like `[CURRENT_TASK]`, `[PHASE_NAME]`, etc.) in this file or in `.context/` files, the framework hasn't been initialized for this project yet. Before doing anything else:

1. Explore the codebase to understand the project structure, purpose, tech stack, and patterns
2. Fill in all `[PLACEHOLDER]` markers in this file (`CLAUDE.md`)
3. Fill in `.context/CURRENT_STATUS.md` with actual project state
4. Fill in `.context/ARCHITECTURE.md` with the real system design
5. Fill in `.context/CONVENTIONS.md` with the actual coding standards you observe
6. Fill in `.context/MASTER_PLAN.md` if the user provides a roadmap
7. Suggest a commit message for the user to commit the initialized context

### Every Session — Start

Before writing any code, read these files in order:

1. **`.context/CURRENT_STATUS.md`** — what was accomplished last session, what's in progress, what's next
2. **`.context/MASTER_PLAN.md`** — full roadmap; confirm which phase is active and which tasks are already checked off
3. **`.context/CONVENTIONS.md`** — coding standards to follow
4. **`.context/ARCHITECTURE.md`** — system design and how components connect

Read as needed based on the task:

- `.context/DECISIONS.md` — past architectural decisions, to avoid re-debating settled questions

### Every Session — During Work

- **Follow CONVENTIONS.md** for all code you write
- **Check DECISIONS.md** before proposing architectural changes — the decision may already be made
- **Record new decisions** in `DECISIONS.md` when significant technical choices are made (use the ADR template)
- **Tick off completed tasks** in `MASTER_PLAN.md` as soon as they are done — change `- [ ]` to `- [x]`

### Every Session — End

Before the session ends:

1. **Update `.context/CURRENT_STATUS.md`** with:
   - What was completed
   - What's in progress
   - What should happen next
   - Any new blockers or open questions
2. **Update `.context/MASTER_PLAN.md`** — ensure all tasks completed this session are checked off; if the active phase is now complete, mark its milestones done and note the next phase as active
3. **Update this file's Current Focus section** if priorities changed
4. **Create a checkpoint** in `.context/CHECKPOINTS/` if the session was long or made significant progress
5. **Suggest a commit message** that includes both code and context changes — never commit or push automatically

## Current Focus

**Phase**: [CURRENT_PHASE]
**Working on**: [CURRENT_TASK]
**Key constraint**: [KEY_CONSTRAINT]

## Reference

| File | Purpose |
|------|---------|
| `.context/CURRENT_STATUS.md` | Where the project stands right now |
| `.context/MASTER_PLAN.md` | Implementation roadmap |
| `.context/ARCHITECTURE.md` | System design and components |
| `.context/DECISIONS.md` | Architecture Decision Records |
| `.context/CONVENTIONS.md` | Coding standards and patterns |
| `.context/SETUP.md` | Dev environment setup |
| `.context/CHECKPOINTS/` | Session summaries |
