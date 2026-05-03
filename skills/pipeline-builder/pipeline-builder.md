---
name: pipeline-builder
description: Design and implement a CI/CD pipeline for a project, covering lint, test, build, and deploy stages for Python and Node stacks.
---

# Build the Pipeline

## The Law
**Every pipeline must run tests before it touches a deployment target — no exceptions. A pipeline that ships without verified tests is an automated way to break production.**

## When to Use
- A project has no CI/CD configuration and needs one
- The existing pipeline is missing lint, test, or deploy stages
- The stack has changed (new language, new runtime, new cloud target) and the pipeline needs to match
- **Never skip when:** the project has tests that are not yet running automatically (idle tests are the same as no tests), or when a deploy step exists without a preceding test gate

## Process

### Phase 1: Understand the Project
1. Identify the language(s) and runtime: Python, Node, Go, or mixed.
2. Find the test commands by reading `package.json` (scripts), `pyproject.toml`, `Makefile`, or `Taskfile`.
3. Find the build output — a Docker image, a static bundle, a wheel/sdist, a binary.
4. Identify the deploy target — container registry, PaaS, serverless, bare VMs.
5. Note any secrets required (registry credentials, cloud keys, API tokens).

### Phase 2: Choose the Runner
Select based on where the code lives and what the team already uses.

| Runner | Best for | Config file location |
|--------|----------|----------------------|
| GitHub Actions | GitHub repos; free for public repos | `.github/workflows/<name>.yml` |
| GitLab CI | GitLab repos; built-in container registry | `.gitlab-ci.yml` |
| CircleCI | Multi-cloud; Docker-native pipelines | `.circleci/config.yml` |
| Buildkite | On-premise agents; large-scale parallelism | `pipeline.yml` (repo) |
| Jenkins | Self-hosted; existing enterprise setup | `Jenkinsfile` |

### Phase 3: Define Pipeline Stages
Structure every pipeline with these four mandatory stages in order:

1. **Lint** — catch style and static errors before running anything
2. **Test** — run the full test suite; fail hard if any test fails
3. **Build** — produce the deployable artifact only after tests pass
4. **Deploy** — push to the target environment; gated by all prior stages

### Phase 4: Write the Configuration

**GitHub Actions — Python project:**
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install ruff
      - run: ruff check .

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -e ".[dev]"
      - run: pytest --tb=short -q

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Push to registry
        env:
          REGISTRY_TOKEN: ${{ secrets.REGISTRY_TOKEN }}
        run: |
          echo "$REGISTRY_TOKEN" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          docker push ghcr.io/${{ github.repository }}/myapp:${{ github.sha }}
```

**GitHub Actions — Node project:**
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
      - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
```

**GitLab CI — Python project:**
```yaml
# .gitlab-ci.yml
stages:
  - lint
  - test
  - build
  - deploy

default:
  image: python:3.12

lint:
  stage: lint
  script:
    - pip install ruff
    - ruff check .

test:
  stage: test
  script:
    - pip install -e ".[dev]"
    - pytest --tb=short -q

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
  only:
    - main

deploy:
  stage: deploy
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main
```

### Phase 5: Handle Secrets Safely
1. Never commit credentials to the repository — use the runner's secret store.
   - GitHub: Settings → Secrets and variables → Actions
   - GitLab: Settings → CI/CD → Variables
2. Reference secrets via environment variables in the config, never as literals.
3. Add `.env*` and any credential files to `.gitignore` before writing the pipeline config.

### Phase 6: Validate the Pipeline
1. Open a PR with the pipeline config and watch the first run.
2. Confirm each stage produces the expected output and that a failing test actually fails the pipeline.
3. Verify the deploy stage only fires on the intended branch (`main` or a release tag).
4. Confirm secrets are masked in the log output — never printed in plaintext.

## Red Flags — Stop Immediately
- A deploy step has no `needs:` dependency on the test job — tests can be bypassed
- Secrets are hardcoded in the YAML file instead of referenced from the secret store
- The pipeline has no test stage at all, only lint and build
- The deploy job runs on every branch, not just the default or release branch
- Cached dependency layers include credentials or token files
- A `continue-on-error: true` is applied to the test job — failures are being silenced

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "Tests are slow so we skip them in CI" | Slow tests mean the suite needs optimization, not removal from the gate |
| "Deploy on every branch is fine for now" | Accidental deploys of feature branches to production happen fast |
| "I'll add secrets management later" | Committed secrets cannot be un-committed from git history safely |
| "We only have one stage; we'll add more later" | Stages added after the first deploy are always harder to enforce |
| "The pipeline is optional for this project type" | Every project that ships code needs a test gate before it ships |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Understand | Identify language, test command, build output, deploy target | All four are documented |
| 2. Runner | Select GitHub Actions, GitLab CI, or other | Config file location decided |
| 3. Stages | Define lint → test → build → deploy in order | Stage ordering is explicit |
| 4. Config | Write the YAML with correct `needs:` chaining | Each stage depends on the previous |
| 5. Secrets | Move all credentials to the runner secret store | No literals in YAML |
| 6. Validate | PR triggers a run; fail a test to confirm gate works | Pipeline enforces test gate |
