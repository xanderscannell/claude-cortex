---
name: context
description: Load and update project context across sessions
---

# CDS Prevention Framework

Prevent Context Degradation Syndrome (CDS) — when an AI assistant loses track of project state between sessions, forgets architecture decisions, repeats mistakes, or ignores established conventions.

This skill reads and updates your project's persistent context files so the AI assistant maintains complete awareness across sessions.

## Session Start: Load Context

At the START of each session, this skill loads:

1. **Bootloader file** — Check for `CLAUDE.md` and/or `.github/copilot-instructions.md` in the project root. Read whichever exist (both may be present). These contain the Current Focus and context-loading instructions.
2. **`.context/CURRENT_STATUS.md`** — What was accomplished last session, what's in progress, what's next
3. **`.context/CONVENTIONS.md`** — Coding standards and patterns to follow
4. **`.context/ARCHITECTURE.md`** — System design and how components connect

Then reads as needed based on your task:
- **`.context/MASTER_PLAN.md`** — Full roadmap, to understand where current work fits
- **`.context/DECISIONS.md`** — Architecture Decision Records to avoid re-debating settled questions

## During Work: Follow Context

While working:

- **Follow CONVENTIONS.md** for all code you write
- **Check DECISIONS.md** before proposing architectural changes — the decision may already be made
- **Record new decisions** in `DECISIONS.md` when significant technical choices are made

## Session End: Update Context

At the END of each session:

1. **Update `.context/CURRENT_STATUS.md`** with:
   - What was completed this session
   - What's in progress
   - What should happen next
   - Any new blockers or open questions

2. **Update the Current Focus section** in whichever bootloader files exist (`CLAUDE.md` and/or `.github/copilot-instructions.md`) if priorities changed

3. **Create a checkpoint** in `.context/CHECKPOINTS/` if the session was long or made significant progress

4. **Commit the changes** (git add + commit with message describing what was done)

## File Reference

| File | Purpose | Updated |
|------|---------|---------|
| `CLAUDE.md` | Bootloader for Claude Code (read automatically) | Every few sessions |
| `.github/copilot-instructions.md` | Bootloader for GitHub Copilot (read automatically) | Every few sessions |
| `.context/CURRENT_STATUS.md` | Where the project stands right now | Every session |
| `.context/MASTER_PLAN.md` | Implementation roadmap | When phases change |
| `.context/ARCHITECTURE.md` | System design and components | When architecture evolves |
| `.context/DECISIONS.md` | Architecture Decision Records | When decisions are made |
| `.context/CONVENTIONS.md` | Coding standards and patterns | Rarely |
| `.context/SETUP.md` | Dev environment setup | Rarely |
| `.context/CHECKPOINTS/` | Session summaries and snapshots | End of long sessions |

## Need to Initialize?

If you see `[PLACEHOLDER]` markers (like `[CURRENT_TASK]`, `[PHASE_NAME]`, etc.) in `CLAUDE.md`, `.github/copilot-instructions.md`, or `.context/` files, run `/context-init` to initialize the framework with your project details.
