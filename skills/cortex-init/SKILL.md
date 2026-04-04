---
name: cortex-init
description: >
  Full framework initialization — sets up both .context/ (project knowledge)
  and PROJECT_GUIDELINES.md (coding standards) in one command. Use when the
  user wants to set up the complete claude-cortex framework, says "initialize
  everything", "full setup", or "set up cortex".
---

# Initialize Full Cortex Framework

Initialize both the context framework and project guidelines in one command.
Runs context-init first, then guidelines-init. Because context-init creates
CONVENTIONS.md before guidelines-init runs, guidelines-init's existing
CONVENTIONS.md integration logic handles the cross-reference automatically —
no orchestrator wiring needed.

## Arguments

| Argument | Guidelines mode |
|---|---|
| *(empty)* | Simple — flat rules list, minimal interview |
| `--simple` | Simple — same as empty |
| `--full` | Full — categorized rules with severity tags, comprehensive interview |

---

## Step 1: Pre-flight

### 1a. Detect existing artifacts

Check for:
- `.context/` directory
- `CLAUDE.md` or `.github/copilot-instructions.md` in the project root
- `PROJECT_GUIDELINES.md` in the project root

If any exist, tell the user what was found and ask:
> "These files already exist. Do you want to reinitialize? This will overwrite them."

If the user declines, stop.

### 1b. Determine guidelines mode

- If `--full` was passed: use full mode
- If `--simple` was passed or no argument: use simple mode

---

## Step 2: Run Context Init

Read `skills/context-init/SKILL.md` and follow its instructions with these
adjustments:

- **Skip** its "Check if already initialized" step — already handled in pre-flight
- **Do NOT** suggest a commit at the end — the combined commit happens in Step 4

Everything else runs as documented: environment detection, file selection prompt,
directory creation, template copying, codebase analysis, placeholder filling.

---

## Step 3: Run Guidelines Init

Read `skills/guidelines-init/SKILL.md` and follow its instructions with these
adjustments:

- Pass the guidelines mode determined in Step 1b (simple or full)
- If reinitializing and `PROJECT_GUIDELINES.md` already exists: follow the
  existing-guidelines edge case logic from the skill
- **Do NOT** suggest a commit at the end — the combined commit happens in Step 4

Everything else runs as documented. Because context-init already created
CONVENTIONS.md in Step 2, guidelines-init's existing CONVENTIONS.md integration
logic will naturally detect it, migrate any code quality rules, and add the
cross-reference.

---

## Step 4: Summary and Commit

1. **List created files** grouped by system:
   - Context Framework: bootloader file, `.context/` files created
   - Guidelines: `PROJECT_GUIDELINES.md` (mode used, rule count or category count)

2. **Note the cross-reference**: "CONVENTIONS.md references PROJECT_GUIDELINES.md
   as the source of truth for code quality rules."

3. **Suggest a single commit**:
   ```
   git add .context/ [bootloader] PROJECT_GUIDELINES.md
   git commit -m "Initialize cortex framework (context + guidelines)"
   ```

4. **Mention next steps**:
   - Run `/context-session` at the start of each session
   - Run `/guidelines-review` to check code against the generated guidelines

---

## Rules

- Always run context-init before guidelines-init (CONVENTIONS.md must exist for
  guidelines-init to detect)
- Do NOT suggest separate commits — always suggest a single combined commit
- If context-init fails, ask the user if they want to continue with just
  guidelines-init
- If guidelines-init fails, the context framework is still intact — suggest
  running `/guidelines-init` separately to retry
