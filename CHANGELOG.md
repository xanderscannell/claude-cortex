# Changelog

All notable changes to claude-cortex are documented here.

## [1.3.1] — 2026-04-03

### Removed
- Checkpoint functionality — `CHECKPOINT_TEMPLATE.md` deleted, `.context/CHECKPOINTS/` directory no longer created by `/context-init`, checkpoint creation step removed from `/context-session` and both bootloader templates. `CURRENT_STATUS.md` already captures session state; checkpoints added file bloat without practical benefit.

### Fixed
- Both bootloader templates (`CLAUDE.md`, `copilot-instructions.md`) session-end steps now match `/context-session` exactly — added missing step 4 (PROJECT_GUIDELINES.md staleness check) and renumbered remaining steps to 4 and 5
- `docs/context-framework.md` stale `/context` reference updated to `/context-session` (renamed in v1.1.1)
- README code-review description no longer claims `/code-review` is "informed by the loaded context" — reworded to accurately describe it as universal best-practice review, with a pointer to `/guidelines-review` for project-specific enforcement
- README and FAQ updated to accurately reflect context/guidelines integration depth since v1.2.0 — "work independently" language now reads "work independently, but integrate deeply when used together"
- `context-init` After Initialization section now mentions `/cortex-init` as the preferred single-command setup for new projects
- `context-init` After Initialization section duplicate step number (two items numbered "6") corrected to 6 and 7
- `guidelines-init` simple and full mode generation now adds a CONVENTIONS.md back-reference note in the Overview when `.context/CONVENTIONS.md` exists, so users editing guidelines can find tooling documentation
- `guidelines-review` full mode now instructs the reviewer to read `.context/CONVENTIONS.md` for reliable tooling detection rather than guessing

---

## [1.3.0] — 2026-04-03

### Added
- `/cortex-init` skill — full framework initialization that runs `/context-init` and `/guidelines-init` in parallel via two subagents (each scanning through a different lens), then coordinates the CONVENTIONS.md to PROJECT_GUIDELINES.md cross-reference. Supports `--simple` (default) and `--full` arguments for guidelines mode.

---

## [1.2.0] — 2026-04-03

### Removed
- `/context-checkpoint` skill — all session-end behavior is now handled by `/context-session`. Checkpoint files in `.context/CHECKPOINTS/` continue to be created by `/context-session` and the directory is preserved.

### Changed
- `/context-session` session-end now includes a PROJECT_GUIDELINES.md staleness check — if the session introduced new patterns, architecture changes, or new conventions, the assistant updates `PROJECT_GUIDELINES.md` directly or flags it for next session
- `/context-session` checkpoint creation now references `templates/context/CHECKPOINT_TEMPLATE.md` for consistent checkpoint format
- `CONVENTIONS.md` template narrowed to tooling and environment only (language, runtime, formatter, linter, test runner, build commands) — code quality rules (naming, architecture, error handling, testing standards, security, performance, git workflow) belong in `PROJECT_GUIDELINES.md`
- Both bootloader templates (`CLAUDE.md`, `copilot-instructions.md`) now load `PROJECT_GUIDELINES.md` at session start and instruct agents to keep it updated as the codebase evolves
- `guidelines-init` now detects and migrates code quality rules from existing `CONVENTIONS.md` into the generated `PROJECT_GUIDELINES.md`, then narrows `CONVENTIONS.md` to tooling-only

---

## [1.1.1] — 2026-04-03

### Changed
- Renamed `/context` skill to `/context-session` to avoid conflict with Claude Code's built-in `/context` command
- MASTER_PLAN.md is now a mandatory session-start read (previously optional/as-needed)
- Tasks in MASTER_PLAN.md are now ticked off during work as they are completed, not just at phase boundaries
- CURRENT_STATUS.md moved ahead of MASTER_PLAN.md in session-start reading order (highest priority)
- Standardised commit guidance across all context skills — all skills now suggest a commit title and description rather than running git commands or providing hardcoded messages
- Output-producing skills (`context-checkpoint`, `guidelines-review`, `code-review`, `ult-code-review`) now explicitly prohibit non-ASCII characters and emojis in their output

---

## [1.1.0] — 2026-04-02

### Added
- `/ult-code-review` skill — orchestrates 10 parallel specialist agents (security, architecture, performance, error handling, dependencies, idioms, triple-pass correctness, coverage gap analysis) for near-100% bug catch rate

---

## [1.0.0] — 2026-04-01

### Added
- Initial release of the claude-cortex plugin
- **Context framework** — CDS (Context Degradation Syndrome) prevention system using `.context/` files
  - `/context-session` skill — load and update project context across sessions
  - `/context-init` skill — initialise the context framework for a new project
  - `/context-checkpoint` skill — create session checkpoint documenting progress
  - Templates for `CLAUDE.md`, `copilot-instructions.md`, `CURRENT_STATUS.md`, `MASTER_PLAN.md`, `ARCHITECTURE.md`, `DECISIONS.md`, `CONVENTIONS.md`, `SETUP.md`, `CHECKPOINT_TEMPLATE.md`
- **Code review** — `/code-review` skill for structured code quality review
- **Guidelines** — `/guidelines-init` and `/guidelines-review` skills for project guideline management
- **Reviewer agent** — parallel specialist agent for batch file review
- Plugin manifest for Claude Code marketplace installation (`plugin.json`, `marketplace.json`)

### Changed (1.0.x patches)
- `context-init`: Added selective context file initialisation — users can now choose which `.context/` files to create rather than always generating all of them
- `context-init`: Auto-detect running AI environment to create only the relevant bootloader (`CLAUDE.md` for Claude Code, `copilot-instructions.md` for GitHub Copilot)
- README: Added installation instructions for Claude Code and VS Code
- `plugin.json`: Fixed manifest schema — converted `author` field from string to object, added `./` prefix to skill and agent paths for marketplace compatibility
