---
name: context-init
description: Initialize CDS framework in a new project
---

# Initialize CDS Prevention Framework

Initialize the Context Degradation System prevention framework in a project. This creates the `.context/` directory structure and the appropriate bootloader file(s) for the user's AI tool, then analyzes the codebase to fill in project-specific details.

## How to Use

When the user runs `/context-init`:

1. **Detect the current environment** — Check your system prompt to determine which AI tool is running this skill. Create the matching bootloader:

   | Environment | Bootloader |
   |-------------|-----------|
   | Claude Code | `CLAUDE.md` |
   | GitHub Copilot | `.github/copilot-instructions.md` |

   If you can't determine the environment, ask the user. All bootloaders use the same template content — only the file location differs.

2. **Check if already initialized** — Look for existing `.context/` directory and bootloader files
   - If they exist, ask if the user wants to reinitialize (this will overwrite existing files)
   - **Important:** If reinitializing and `.context/CONVENTIONS.md` exists with code quality rules (naming conventions, error handling, testing standards, git workflow) AND `PROJECT_GUIDELINES.md` does not exist, warn the user: "Your current CONVENTIONS.md contains code quality rules that will be removed during reinitialization. Run `/guidelines-init` first to migrate them to PROJECT_GUIDELINES.md, or they will be lost."
   - If they don't exist, proceed with initialization

3. **Ask the user which context files to include**:

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
   > 3. `CONVENTIONS.md` — Tooling, environment, and build commands
   > 4. `DECISIONS.md` — Architecture Decision Records
   > 5. `MASTER_PLAN.md` — Implementation roadmap and phases
   > 6. `SETUP.md` — Dev environment setup instructions

   The bootloader and `CHECKPOINTS/` directory are always created regardless of selection.

   Only create the selected context files in the steps below.

4. **Create the directory structure** — Create `.context/` and `CHECKPOINTS/` subdirectory. Only include selected context files:
   ```
   .context/
   ├── [selected files...]
   └── CHECKPOINTS/
   ```

5. **Create files from templates** — Copy from `templates/context/` to target locations. Create the bootloader for the detected environment (step 1), and only the context files the user selected (step 3):

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

   When using a subset of context files, update the bootloader to only reference the files that were created.

6. **Analyze the codebase** to detect:
   - Project name (from package.json, Cargo.toml, pyproject.toml, go.mod, or directory name)
   - Description (from README.md if present)
   - Tech stack (languages, frameworks, libraries)
   - Tooling (formatter, linter, test framework, build commands)
   - Architecture (key modules, directory structure)
   - Whether `PROJECT_GUIDELINES.md` already exists in the project root

7. **Fill in placeholders** — Replace `[PLACEHOLDER]` markers in the created files with detected values. When filling in `CONVENTIONS.md`, populate only tooling and environment fields (language, runtime, formatter, linter, test framework, build commands). Do not add naming conventions, error handling patterns, testing standards, git workflow, or architecture rules — those belong in `PROJECT_GUIDELINES.md`.
   - If `PROJECT_GUIDELINES.md` already exists: the `## Project Guidelines Reference` section in `CONVENTIONS.md` should confirm it as the source of truth for code quality rules.
   - If `PROJECT_GUIDELINES.md` does not exist: the `## Project Guidelines Reference` section should note that `/guidelines-init` can create it.

8. **Suggest a commit** — Include only the files that were actually created:
   ```bash
   git add .context/ [bootloader files created]
   git commit -m "Initialize CDS prevention context framework"
   ```

---

## After Initialization

Once files are created and placeholders filled:

1. Tell the user what was created (list only the files that were actually created)
2. Summarize detected project details (name, tech stack, etc.)
3. Explain which bootloader was created and that it's read automatically by the detected tool
4. Suggest they review and customize the created files (only mention files that exist):
   - Bootloader — Update the Current Focus section
   - `MASTER_PLAN.md` — Define implementation phases (if created)
   - `CONVENTIONS.md` — Verify detected tooling and environment info (if created)
   - `ARCHITECTURE.md` — Refine system design (if created)
5. **Mention PROJECT_GUIDELINES.md**:
   - If `PROJECT_GUIDELINES.md` already exists: note that it was detected and `CONVENTIONS.md` references it as the source of truth for code quality rules
   - If `PROJECT_GUIDELINES.md` does not exist: recommend running `/guidelines-init` to create code quality rules (naming, architecture, error handling, testing, security, performance, git workflow) — explain that `CONVENTIONS.md` covers tooling/environment while `PROJECT_GUIDELINES.md` covers code quality standards
6. If the user chose a subset of context files, mention they can add more later by running `/context-init` again
6. Suggest the commit command shown above
