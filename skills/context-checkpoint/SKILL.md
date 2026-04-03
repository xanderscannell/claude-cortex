---
name: context-checkpoint
description: Create session checkpoint documenting progress
---

# Create Session Checkpoint

Create a snapshot of your session's work. Checkpoints document what was accomplished, decisions made, issues solved, and what should happen next. This helps future sessions understand the project's progress and context.

## Process

1. **Gather session context** by reading:
   - `.context/CURRENT_STATUS.md` — what was planned
   - `.context/MASTER_PLAN.md` — current phase and task completion state
   - Recent git log and diff — what was actually done
   - `.context/DECISIONS.md` — any new decisions made

2. **Create a checkpoint file** at `.context/CHECKPOINTS/YYYY-MM-DD-[brief-description].md` with:
   - Session summary (2-3 sentences)
   - Completed tasks and milestones
   - Files changed with descriptions
   - Issues encountered and how they were solved
   - New decisions made
   - What the next session should focus on
   - Any open questions or unresolved issues

3. **Update `.context/CURRENT_STATUS.md`** to reflect:
   - What was completed this session
   - What's now in progress
   - What comes next
   - Any new blockers or questions

4. **Update `.context/MASTER_PLAN.md`** — tick off all tasks completed this session (`- [ ]` → `- [x]`); if the phase is complete, mark its milestones done and update the active phase

5. **Update the Current Focus section** in whichever bootloader files exist (`CLAUDE.md` and/or `.github/copilot-instructions.md`) if priorities changed

6. **Suggest a commit** — provide a title and description based on what was worked on this session

Use the template from `templates/context/CHECKPOINT_TEMPLATE.md` for the checkpoint file format.

## Output Format

Plain ASCII only. Non-ASCII characters and emojis are not allowed in any output.
