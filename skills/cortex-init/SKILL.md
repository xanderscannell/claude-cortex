---
name: cortex-init
description: >
  Full framework initialization — sets up both .context/ (project knowledge)
  and PROJECT_GUIDELINES.md (coding standards) in one command using parallel
  agents. Use when the user wants to set up the complete claude-cortex
  framework, says "initialize everything", "full setup", or "set up cortex".
---

# Initialize Full Cortex Framework

Initialize both the context framework and project guidelines in one command.
Two subagents scan the codebase simultaneously through different lenses, then
the orchestrator wires the two systems together.

## Arguments

| Argument | Guidelines mode |
|---|---|
| *(empty)* | Simple — flat rules list, minimal interview |
| `--simple` | Simple — same as empty |
| `--full` | Full — categorized rules with severity tags, comprehensive interview |

Context file selection is always interactive.

---

## Step 1: Pre-flight Checks

Run all checks before spawning any agents.

### 1a. Detect existing artifacts

Check for:
- `.context/` directory
- `CLAUDE.md` or `.github/copilot-instructions.md` in the project root
- `PROJECT_GUIDELINES.md` in the project root

If any exist, tell the user what was found and ask:
> "These files already exist. Do you want to reinitialize? This will overwrite them."

If the user declines, stop.

Special case: If `.context/CONVENTIONS.md` exists with code quality rules (naming
conventions, error handling, testing standards, git workflow, architecture) AND
`PROJECT_GUIDELINES.md` does not exist, reassure the user: those rules will NOT be
lost — Agent B will incorporate them into the generated guidelines. Explain this
before asking whether to reinitialize.

### 1b. Detect AI environment

Check your system prompt to determine which AI tool is running this skill:

| Environment | Bootloader to create |
|---|---|
| Claude Code | `CLAUDE.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |

If indeterminate, ask the user. Store the result — it will be passed to Agent A.

### 1c. Determine guidelines mode

- If `--full` was passed: use full mode
- If `--simple` was passed or no argument: use simple mode
- Store the result — it will be passed to Agent B

### 1d. Ask context file selection

Present the same selection prompt as `/context-init`:

> How would you like to set up context?
>
> 1. **All context files** — Full framework with status, architecture, conventions, decisions, master plan, and setup
> 2. **Pick context files** — Choose only the ones you need

If the user picks 2, present the list:

> Which context files do you want? (list the numbers)
>
> 1. `CURRENT_STATUS.md` — Track what's done, in progress, and next
> 2. `ARCHITECTURE.md` — Document system design and components
> 3. `CONVENTIONS.md` — Tooling, environment, and build commands
> 4. `DECISIONS.md` — Architecture Decision Records
> 5. `MASTER_PLAN.md` — Implementation roadmap and phases
> 6. `SETUP.md` — Dev environment setup instructions

**`CONVENTIONS.md` is mandatory** — if the user does not select it, add it back and
explain: "CONVENTIONS.md is required because it links the context framework to
PROJECT_GUIDELINES.md."

Store the selection — it will be passed to Agent A.

---

## Step 2: Parallel Agent Execution

Read both skill files now, before spawning agents:
- Read `skills/context-init/SKILL.md` — full content for Agent A
- Read `skills/guidelines-init/SKILL.md` — full content for Agent B

Spawn both agents simultaneously. Both need full tool access (Read, Write, Edit,
Glob, Grep, Bash).

### Agent A: Context Agent

**Brief**: Follow the instructions in `skills/context-init/SKILL.md` with the
overrides below.

**Pass to agent**:
- The full content of `skills/context-init/SKILL.md`
- Detected environment (from step 1b)
- User's context file selection (from step 1d)
- Whether reinitializing (from step 1a)

**Overrides** (state these explicitly in the agent prompt):
- Skip step 1 (environment detection) — already determined: [detected environment]
- Skip step 2 (check if already initialized) — already handled in pre-flight
- Skip step 3 (ask user for file selection) — selection is: [selected files]
- When filling in CONVENTIONS.md's `## Project Guidelines Reference` section:
  write the version that states PROJECT_GUIDELINES.md IS the source of truth
  (not the "run /guidelines-init to create it" version) — Agent B is creating
  it simultaneously
- Skip step 5 of "After Initialization" (suggesting to run `/guidelines-init`) —
  guidelines are being created in parallel
- Do NOT suggest a commit — the orchestrator handles that

**Lens**: "What IS this codebase?" — config files, tooling, tech stack, package
manager, formatter, linter, test runner, build commands. Look at configuration
and project metadata, not source code patterns.

**Produces**: `.context/` directory with selected files, bootloader, all
placeholders filled.

### Agent B: Guidelines Agent

**Brief**: Follow the instructions in `skills/guidelines-init/SKILL.md` with the
overrides below.

**Pass to agent**:
- The full content of `skills/guidelines-init/SKILL.md`
- Guidelines mode (from step 1c): simple or full
- Whether reinitializing with an existing PROJECT_GUIDELINES.md (from step 1a)

**Overrides** (state these explicitly in the agent prompt):
- Skip the `## CONVENTIONS.md Integration` section entirely — do NOT scan for
  CONVENTIONS.md, do NOT migrate from it, do NOT write to it. CONVENTIONS.md is
  being created fresh by Agent A simultaneously; the orchestrator handles the
  cross-reference after both agents complete
- Do NOT suggest a commit — the orchestrator handles that
- If PROJECT_GUIDELINES.md already exists (reinitializing): follow the existing-
  guidelines edge case logic from the skill (mode mismatch warnings, merge vs replace)

**Lens**: "How should code BE WRITTEN here?" — existing source code patterns,
naming conventions already in use, architectural patterns, error handling
approaches, testing patterns. Look at the actual code, not config files.

**Produces**: `PROJECT_GUIDELINES.md` in the project root.

**Note**: Agent B's interview (simple or full mode) will run while Agent A scans
in the background. This is intentional — Agent A's scan happens during the user's
think time, reducing total wall-clock time.

---

## Step 3: Coordination

After BOTH agents have completed:

1. **Verify Agent A output** — check that `.context/` exists and CONVENTIONS.md
   was created

2. **Verify Agent B output** — check that `PROJECT_GUIDELINES.md` exists in the
   project root

3. **Wire the cross-reference** — read `.context/CONVENTIONS.md` and check the
   `## Project Guidelines Reference` section:
   - If present and correctly references PROJECT_GUIDELINES.md as the source of
     truth: no action needed
   - If present but says "run /guidelines-init to create it": update it to the
     source-of-truth version
   - If missing: append it using this content:

     ```
     ## Project Guidelines Reference

     `PROJECT_GUIDELINES.md` in the project root is the source of truth for code
     quality rules: naming conventions, architecture patterns, error handling,
     testing standards, security, performance, and git workflow. Follow those
     rules for all code you write.
     ```

4. **Verify bootloader** — read the created bootloader and confirm it lists
   `PROJECT_GUIDELINES.md` in the session-start reading order (the template
   already includes this — this is a sanity check)

---

## Step 4: Failure Handling

**If Agent A fails, Agent B succeeds:**
- Tell the user: "PROJECT_GUIDELINES.md was created, but the context framework
  setup failed."
- Show the Agent A error
- Do NOT perform the coordination step
- Suggest running `/context-init` separately to retry

**If Agent B fails, Agent A succeeds:**
- Tell the user: "The context framework was set up, but guidelines initialization
  failed."
- Show the Agent B error
- Update CONVENTIONS.md's `## Project Guidelines Reference` section to say:
  "Run `/guidelines-init` to create PROJECT_GUIDELINES.md."
- Suggest running `/guidelines-init` separately to retry

**If both agents fail:**
- Report both errors
- Suggest running `/context-init` and `/guidelines-init` separately for better
  diagnostics

---

## Step 5: Summary and Commit Suggestion

After successful completion:

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

- Do NOT merge the two scans — each agent scans independently through its own lens
- Agent A owns `.context/` and the bootloader; Agent B owns `PROJECT_GUIDELINES.md`
- Only the orchestrator writes to `CONVENTIONS.md` post-completion
- Always suggest a single combined commit, never two separate commits
- CONVENTIONS.md must always be included in the context file selection
