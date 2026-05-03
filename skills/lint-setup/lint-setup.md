---
name: lint-setup
description: Install and configure ESLint, Prettier, Ruff, and EditorConfig in a project that has none of these tools set up yet.
---

# Lint Setup

## The Law
**Configure linting and formatting once, correctly, at the start of a project — retrofitting these tools onto an established codebase is ten times harder and produces a noisy, unreviewable diff.**

## When to Use
- When starting a new Python, JavaScript, or TypeScript project
- When a project has code but no linter configuration at all
- When a project uses an outdated config that needs replacing (e.g., `.eslintrc.json` with deprecated rules)
- When onboarding a new contributor and discovering the project has no shared formatting baseline
- **Never skip when:** CI is being added to a project — linting must be configured locally first so the first CI run is not a wall of failures

## Process

### Phase 1: Audit What Already Exists
1. Check for existing config files before installing anything:
   ```bash
   ls .eslintrc* eslint.config* .prettierrc* prettier.config* \
      pyproject.toml ruff.toml .flake8 setup.cfg .editorconfig 2>/dev/null
   ```
2. Check `package.json` for existing lint scripts and devDependencies
3. Check `pyproject.toml` for `[tool.ruff]`, `[tool.black]`, or `[tool.isort]` sections
4. Document what exists — do not overwrite a working config without reading it first

### Phase 2: Set Up EditorConfig (all projects)
EditorConfig is language-agnostic and should be the first file created in every project.

1. Create `.editorconfig` at the project root:
   ```ini
   root = true

   [*]
   indent_style = space
   indent_size = 4
   end_of_line = lf
   charset = utf-8
   trim_trailing_whitespace = true
   insert_final_newline = true

   [*.{js,ts,jsx,tsx,json,yaml,yml,html,css}]
   indent_size = 2

   [Makefile]
   indent_style = tab
   ```
2. Confirm the editor or IDE respects `.editorconfig` — most do by default; VS Code requires the EditorConfig extension

### Phase 3: Set Up Ruff (Python projects)
Ruff replaces flake8, isort, and most pylint checks in a single fast tool.

1. Install Ruff:
   ```bash
   pip install ruff
   # or, to pin it in the project:
   pip install ruff && echo "ruff" >> requirements-dev.txt
   ```

2. Add Ruff configuration to `pyproject.toml` (create the file if it does not exist):
   ```toml
   [tool.ruff]
   line-length = 88
   target-version = "py311"

   [tool.ruff.lint]
   select = ["E", "F", "I", "UP", "B", "SIM"]
   ignore = []

   [tool.ruff.format]
   quote-style = "double"
   indent-style = "space"
   ```

3. Run Ruff to confirm the configuration works:
   ```bash
   ruff check . --select E,F
   ruff format --check .
   ```

4. If the project also uses Black, configure isort to be Black-compatible:
   ```toml
   [tool.isort]
   profile = "black"
   ```

### Phase 4: Set Up ESLint and Prettier (JavaScript / TypeScript projects)
1. Install ESLint and Prettier:
   ```bash
   npm install --save-dev eslint prettier eslint-config-prettier
   # For TypeScript projects, also install:
   npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin
   ```

2. Create `eslint.config.js` (flat config format, ESLint 9+):
   ```javascript
   import js from "@eslint/js";
   import tsPlugin from "@typescript-eslint/eslint-plugin";
   import tsParser from "@typescript-eslint/parser";
   import prettier from "eslint-config-prettier";

   export default [
     js.configs.recommended,
     {
       files: ["**/*.ts", "**/*.tsx"],
       languageOptions: { parser: tsParser },
       plugins: { "@typescript-eslint": tsPlugin },
       rules: {
         ...tsPlugin.configs.recommended.rules,
         "@typescript-eslint/no-explicit-any": "error",
       },
     },
     prettier,
   ];
   ```
   For ESLint 8 and below, use `.eslintrc.json` instead.

3. Create `.prettierrc`:
   ```json
   {
     "singleQuote": false,
     "semi": true,
     "tabWidth": 2,
     "trailingComma": "es5",
     "printWidth": 100
   }
   ```

4. Create `.prettierignore`:
   ```
   node_modules/
   dist/
   build/
   coverage/
   *.min.js
   ```

5. Add lint scripts to `package.json`:
   ```json
   {
     "scripts": {
       "lint": "eslint . --ext .ts,.tsx,.js,.jsx",
       "lint:fix": "eslint . --ext .ts,.tsx,.js,.jsx --fix && prettier --write .",
       "format:check": "prettier --check ."
     }
   }
   ```

### Phase 5: Verify the Setup
1. Run each tool with no source files to confirm configuration loads without errors:
   ```bash
   # Python
   ruff check . --statistics
   ruff format --check .

   # JS/TS
   npm run lint
   npm run format:check
   ```

2. Check that the tools agree on indentation — EditorConfig, Prettier, and Ruff must all use the same indent size for their respective file types

3. Run a quick smoke test by introducing a deliberate error and confirming the linter catches it:
   ```python
   # Python: an unused import should be flagged by Ruff rule F401
   import os  # unused
   ```
   ```typescript
   // TypeScript: explicit any should be flagged
   const x: any = 5;
   ```

4. Commit all config files together in a single "chore: add linting configuration" commit before writing any feature code

## Red Flags — Stop Immediately
- ESLint and Prettier configs conflict on quote style or semicolons — `eslint-config-prettier` must be the last entry in the ESLint config to disable all Prettier-conflicting rules
- Ruff `line-length` and Black `line-length` disagree — set both to the same value (88 is the standard)
- The project already has a hand-rolled linting script in a Makefile — read it before adding new config files to avoid running two conflicting setups
- A `.editorconfig` already exists with different indent settings — align all tools to the existing value rather than overwriting it

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "We can add linting later" | Later means a 10,000-line noisy diff; now means a 30-line config commit |
| "Everyone on the team uses a different formatter" | That is precisely why a shared config file exists |
| "Ruff is too strict out of the box" | Start with `["E", "F"]` rules and expand from there |
| "We already have flake8 so we don't need Ruff" | Ruff is a superset of flake8 and runs 100x faster; migrate and remove flake8 |
| "Prettier changes too much of the existing code" | Commit the format-only change first, then the feature — reviewers can skip the format commit |
| "ESLint flat config is confusing" | Use the migration guide; legacy `.eslintrc` is deprecated and will stop working |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Audit | Check what config already exists | No surprise overwrites, existing config documented |
| 2. EditorConfig | Create `.editorconfig` with indent and charset rules | File present, editor respects it |
| 3. Ruff | Install and configure in `pyproject.toml` | `ruff check .` runs clean on new project |
| 4. ESLint + Prettier | Install packages, create config files, add npm scripts | `npm run lint` and `npm run format:check` pass |
| 5. Verify | Tools run clean, smoke test catches deliberate errors | All tools agree, config committed |
