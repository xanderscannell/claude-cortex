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

2. **Create the directory structure**:
   ```
   .context/
   ├── CURRENT_STATUS.md
   ├── ARCHITECTURE.md
   ├── CONVENTIONS.md
   ├── DECISIONS.md
   ├── MASTER_PLAN.md
   ├── SETUP.md
   └── CHECKPOINTS/
   ```

3. **Create files from templates** — Read each template from the plugin's `templates/` directory and copy it to the target project location. The mapping is:

   | Template source | Target location |
   |----------------|-----------------|
   | `templates/context/CLAUDE.md` | `CLAUDE.md` (project root) |
   | `templates/context/copilot-instructions.md` | `.github/copilot-instructions.md` |
   | `templates/context/CURRENT_STATUS.md` | `.context/CURRENT_STATUS.md` |
   | `templates/context/ARCHITECTURE.md` | `.context/ARCHITECTURE.md` |
   | `templates/context/CONVENTIONS.md` | `.context/CONVENTIONS.md` |
   | `templates/context/DECISIONS.md` | `.context/DECISIONS.md` |
   | `templates/context/MASTER_PLAN.md` | `.context/MASTER_PLAN.md` |
   | `templates/context/SETUP.md` | `.context/SETUP.md` |

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

1. Tell the user what was created
2. Summarize detected project details (name, tech stack, etc.)
3. Explain that **both `CLAUDE.md` and `.github/copilot-instructions.md`** were created:
   - `CLAUDE.md` is read by Claude Code
   - `.github/copilot-instructions.md` is read by GitHub Copilot
   - Both contain the same context-loading instructions
4. Suggest they review and customize:
   - `CLAUDE.md` and `.github/copilot-instructions.md` — Update the Current Focus section
   - `MASTER_PLAN.md` — Define implementation phases
   - `CONVENTIONS.md` — Verify detected conventions
   - `ARCHITECTURE.md` — Refine system design
5. Suggest the commit command shown above
