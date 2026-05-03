---
name: monorepo-nav
description: Orient inside a monorepo, locate the relevant package, and determine which packages are affected by a change before running work.
---

# Navigate Monorepo

## The Law
**Before touching any code in a monorepo, identify the package boundary your change belongs to, confirm its workspace tool, and run affected-package detection — never assume a change is isolated to one package.**

## When to Use
- Starting work in an unfamiliar monorepo and needing to understand the package layout
- Making a change to a shared package and needing to know which dependents will be affected
- Running tests or builds and unsure which packages to target
- **Never skip when:** a shared utility, library, or type package is being modified (changes there ripple — affected detection is required), or when you are about to run tests across the entire repo (targeted runs save significant time)

## Process

### Phase 1: Identify the Workspace Tool
1. Check for workspace configuration files at the repo root:
   ```bash
   ls package.json pnpm-workspace.yaml nx.json turbo.json \
      lerna.json rush.json pyproject.toml packages/ apps/ libs/
   ```
2. Map the file to the tool:

   | File present | Tool |
   |---|---|
   | `nx.json` | Nx |
   | `turbo.json` | Turborepo |
   | `pnpm-workspace.yaml` | pnpm workspaces |
   | `lerna.json` | Lerna |
   | `rush.json` | Rush |
   | `pyproject.toml` with `[tool.poetry.dependencies]` | Poetry monorepo |
   | `pyproject.toml` with `packages = [...]` | Hatch / uv workspaces |

3. Read the workspace package list:
   ```bash
   # pnpm
   pnpm list -r --depth -1

   # Nx
   npx nx show projects

   # Turborepo
   turbo ls

   # Python uv workspace
   uv workspace list
   ```

### Phase 2: Map the Package Structure
1. Print the top-level directory tree to one level of depth:
   ```bash
   ls packages/ apps/ libs/ 2>/dev/null | head -40
   # or
   find . -maxdepth 2 -name "package.json" -not -path "*/node_modules/*" | sort
   ```
2. Identify the three standard monorepo zones:
   - `apps/` or `services/` — deployable applications
   - `packages/` or `libs/` — shared libraries consumed by apps and other packages
   - `tools/` or `scripts/` — build tooling and repo-level utilities
3. Read the `package.json` or `pyproject.toml` of the package you are about to touch to confirm its name and dependencies:
   ```bash
   cat packages/auth/package.json | jq '{name, dependencies, devDependencies}'
   # or
   cat packages/auth/pyproject.toml
   ```

### Phase 3: Detect Affected Packages
Before running tests or builds, determine the blast radius of your change.

**Nx:**
```bash
# What packages does a change to 'auth' affect?
npx nx affected --base=main --head=HEAD --print-affected

# Run tests only for affected packages
npx nx affected --target=test --base=main --head=HEAD
```

**Turborepo:**
```bash
# Build only packages affected since main
turbo run build --filter=...[main]

# Test only affected packages
turbo run test --filter=...[HEAD^1]
```

**pnpm workspaces (manual approach):**
```bash
# Find which packages depend on 'auth'
grep -r '"@myorg/auth"' packages/ apps/ --include="package.json" -l
```

**Python uv / Poetry (manual approach):**
```bash
# Find which packages import from 'auth'
grep -r "from auth" packages/ apps/ --include="*.py" -l
grep -r "auth = " --include="pyproject.toml" -r . -l
```

### Phase 4: Run Targeted Commands
1. Run lint, test, and build commands scoped to the affected package(s) — not the whole repo:

   **Nx:**
   ```bash
   npx nx test auth
   npx nx build auth
   npx nx lint auth
   ```

   **Turborepo:**
   ```bash
   turbo run test --filter=@myorg/auth
   turbo run build --filter=@myorg/auth...  # include dependents
   ```

   **pnpm:**
   ```bash
   pnpm --filter @myorg/auth test
   pnpm --filter @myorg/auth... build       # include dependents
   ```

   **Python workspace (uv):**
   ```bash
   uv run --package auth pytest packages/auth/tests/
   ```

2. If the change is to a shared package, also run the affected dependents:
   ```bash
   # Turborepo: '...' suffix means "this package and all that depend on it"
   turbo run test --filter=@myorg/auth...
   ```

### Phase 5: Validate Cross-Package Imports
1. If you added or renamed an export, confirm all consumers import it correctly:
   ```bash
   # TypeScript: check for type errors across the repo
   npx tsc --noEmit -p tsconfig.base.json

   # Python: check imports
   python -m mypy packages/ apps/ --ignore-missing-imports
   ```
2. If the workspace tool has a graph command, render the dependency graph to spot unexpected connections:
   ```bash
   npx nx graph
   turbo run build --graph
   ```

## Red Flags — Stop Immediately
- Running `npm test` or `pytest` from the repo root without filtering — this runs every test in the monorepo
- A change to a package in `packages/` or `libs/` with no affected-detection step
- Editing a shared type or utility without checking which packages import it
- Installing a new dependency directly into a shared package without checking whether apps should receive it
- Running `npm install` at the repo root in a pnpm or Yarn Berry workspace — this breaks the lockfile

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "My change is small, it can't affect other packages" | Shared packages have dependents you may not know about; always check |
| "Running all tests is safer than running some" | Full runs are slow and mask which package actually broke |
| "I don't need the dependency graph for a quick fix" | Unknown dependency chains are exactly where unexpected breaks come from |
| "I'll just run tests in the root and see what fails" | Root test runs often miss package-local configs and give misleading results |
| "The workspace tool is optional for this project" | Running commands outside the workspace tool bypasses caching and filtering |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Tool | Identify workspace tool from root config files | Tool and package list confirmed |
| 2. Map | List packages; read target package manifest | Package boundary and deps understood |
| 3. Affected | Run affected-detection from main..HEAD | Blast radius known |
| 4. Targeted | Scope lint/test/build to affected packages only | Commands run without full-repo overhead |
| 5. Validate | Type-check cross-package imports | No broken consumers of changed exports |
