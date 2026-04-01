---
name: context-init
description: Initialize CDS framework in a new project
---

# Initialize CDS Prevention Framework

Initialize the Context Degradation System prevention framework in a project. This creates the `.context/` directory structure, `CLAUDE.md` bootloader (for Claude Code), and `.github/copilot-instructions.md` bootloader (for GitHub Copilot), then analyzes your codebase to fill in project-specific details.

## How to Use

When the user runs `/context-init`:

1. **Check if already initialized** — Look for existing `.context/` directory, `CLAUDE.md`, and `.github/copilot-instructions.md`
   - If they exist, ask if the user wants to reinitialize (this will overwrite existing files)
   - If they don't exist, proceed with initialization

2. **Ask the user which context files to include**:

   > How would you like to set up context?
   >
   > 1. **All context files** — Full framework with status, architecture, conventions, decisions, master plan, and setup
   > 2. **Pick context files** — Choose only the ones you need

   - If the user picks **1**, proceed with all files (steps 3-4 below use the full file list).
   - If the user picks **2**, present the available context files and let them choose:

   > Which context files do you want? (list the numbers, e.g. "1, 3, 5")
   >
   > 1. `CURRENT_STATUS.md` — Track what's done, in progress, and next
   > 2. `ARCHITECTURE.md` — Document system design and components
   > 3. `CONVENTIONS.md` — Coding standards and patterns
   > 4. `DECISIONS.md` — Architecture Decision Records
   > 5. `MASTER_PLAN.md` — Implementation roadmap and phases
   > 6. `SETUP.md` — Dev environment setup instructions

   The bootloader files (`CLAUDE.md` and `.github/copilot-instructions.md`) and the `CHECKPOINTS/` directory are always created regardless of selection — they are required for the framework to function.

   Only create the selected context files in the steps below.

3. **Create the directory structure** — Create `.context/` and `CHECKPOINTS/` subdirectory. Only include selected context files:
   ```
   .context/
   ├── [selected files...]
   └── CHECKPOINTS/
   ```

4. **Create files from templates** — Read each template from the plugin's `templates/` directory and copy it to the target project location. Only create files the user selected (bootloaders are always created):

   | Template source | Target location | Always created? |
   |----------------|-----------------|-----------------|
   | `templates/context/CLAUDE.md` | `CLAUDE.md` (project root) | Yes |
   | `templates/context/copilot-instructions.md` | `.github/copilot-instructions.md` | Yes |
   | `templates/context/CURRENT_STATUS.md` | `.context/CURRENT_STATUS.md` | Only if selected |
   | `templates/context/ARCHITECTURE.md` | `.context/ARCHITECTURE.md` | Only if selected |
   | `templates/context/CONVENTIONS.md` | `.context/CONVENTIONS.md` | Only if selected |
   | `templates/context/DECISIONS.md` | `.context/DECISIONS.md` | Only if selected |
   | `templates/context/MASTER_PLAN.md` | `.context/MASTER_PLAN.md` | Only if selected |
   | `templates/context/SETUP.md` | `.context/SETUP.md` | Only if selected |

   When using a subset of files, update the bootloader files (`CLAUDE.md` and `.github/copilot-instructions.md`) to only reference the context files that were created. Remove references to files that were not selected so the AI assistant doesn't try to read nonexistent files.

4. **Analyze the codebase** to detect:
   - Project name (from package.json, Cargo.toml, pyproject.toml, go.mod, or directory name)
   - Description (from README.md if present)
   - Tech stack (languages, frameworks, libraries)
   - Conventions (linting rules, naming patterns, test framework)
   - Architecture (key modules, directory structure)

5. **Fill in placeholders** — Replace `[PLACEHOLDER]` markers in the created files with detected values

6. **Suggest a commit**:
   ```bash
   git add CLAUDE.md .github/copilot-instructions.md .context/
   git commit -m "Initialize CDS prevention context framework"
   ```

---

## After Initialization

Once files are created and placeholders filled:

1. Tell the user what was created (list only the files that were actually created)
2. Summarize detected project details (name, tech stack, etc.)
3. Explain that **both `CLAUDE.md` and `.github/copilot-instructions.md`** were created:
   - `CLAUDE.md` is read by Claude Code
   - `.github/copilot-instructions.md` is read by GitHub Copilot
   - Both contain the same context-loading instructions
4. Suggest they review and customize the created files (only mention files that exist):
   - `CLAUDE.md` and `.github/copilot-instructions.md` — Update the Current Focus section
   - `MASTER_PLAN.md` — Define implementation phases (if created)
   - `CONVENTIONS.md` — Verify detected conventions (if created)
   - `ARCHITECTURE.md` — Refine system design (if created)
5. If the user chose a subset, mention they can add more context files later by running `/context-init` again
6. Suggest the commit command shown above
