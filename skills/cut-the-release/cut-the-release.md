---
name: cut-the-release
description: Use when preparing to ship a release — automate changelog generation, tag creation, and deployment sequence. Ensures every release is documented and traceable.
---

# Cut the Release

## The Law
**A release without a changelog entry, a git tag, and a deployment record is invisible to users and unmaintainable in production. Every release must have all three.**

## When to Use
- Merging the final PR before a release
- Deploying to production after testing is complete
- Coordinating multi-service or multi-environment deployments
- **Never skip when:** the code you're shipping is user-facing or affects production infrastructure

## Process

### Phase 1: Generate the Changelog

1. Run the changelog generation command for your stack:
   - **Node/TypeScript:** `npm run changelog` or equivalent conventional-changelog tool
   - **Python:** `python -m towncrier` or check `CHANGELOG.md` manually and add entry
   - **Go:** Review recent commits and compose entries manually
   - **Ruby:** `bundle exec changelog` or manual entry
2. Verify the changelog entry:
   - Lists all breaking changes first
   - Groups features, fixes, and deprecations separately
   - Includes issue/PR references
   - Does not contain "WIP", "TBD", or placeholder text
3. Commit the changelog:
   ```
   git add CHANGELOG.md
   git commit -m "chore: update changelog for v<version>"
   ```

### Phase 2: Create the Release Tag

1. Determine the version number using semantic versioning:
   - **Major** — breaking changes, incompatible API updates
   - **Minor** — new features, backward-compatible
   - **Patch** — bug fixes, no new features
2. Create an annotated git tag (not lightweight):
   ```
   git tag -a v<version> -m "Release v<version>

   $(git log --oneline <previous-tag>..HEAD)"
   ```
3. Push the tag to remote:
   ```
   git push origin v<version>
   ```
4. Verify the tag is visible:
   ```
   git tag -l v<version>
   git show v<version>
   ```

### Phase 3: Deploy the Release

1. Build the artifact:
   - **Web:** `npm run build` or equivalent; test the bundle
   - **Docker:** `docker build -t <image>:<version> .`
   - **Binary:** `go build -o bin/<name>` or equivalent; sign if required
2. Run smoke tests on the artifact:
   - Can you start the service?
   - Can you reach a health endpoint?
   - Are there any obvious errors in logs?
3. Deploy to staging first (if applicable):
   - Verify the deployment worked
   - Run integration tests against staging
   - Check metrics and logging
4. Deploy to production:
   - Use zero-downtime deployment (blue-green, canary, or rolling updates)
   - Monitor error rates and latency during rollout
   - Have a rollback procedure ready (revert tag, redeploy previous version)
5. Verify production health:
   - Check application logs for errors
   - Verify monitoring dashboards show normal metrics
   - Test at least one critical user flow manually
6. Announce the release:
   - Update the release notes on GitHub/GitLab
   - Post in team Slack channel
   - Notify stakeholders if needed

## Red Flags — Stop Immediately

- Changelog entry is missing or contains placeholder text — do not tag
- Tag already exists for this version — version number conflict, resolve first
- Build fails during artifact creation — fix the build before releasing
- Smoke tests fail on the artifact — the binary is broken, do not deploy
- Rollback procedure is untested — practice it in staging first
- No monitoring dashboard or alerting configured — you cannot detect production issues

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "I'll write the changelog after release" | Released code is immediately in production. Without a changelog, users don't know what changed and you can't explain bugs. |
| "The tag is just for ourselves, we don't need a tag" | Tags are recovery points. Without them, finding "the version that broke X" becomes guesswork. |
| "Staging is down, let's deploy straight to prod" | Staging catches build issues and integration failures before they affect users. Skipping it is reckless. |
| "We've deployed this code a hundred times, no need for smoke tests" | Familiarity breeds complacency. Smoke tests catch deployment-specific failures, not just code bugs. |
| "The deployment looks fine, monitoring is not important" | Monitoring is how you discover silent failures — high error rates, memory leaks, cascading timeouts. Missing them until users report is unprofessional. |
| "Rollback is too complicated, let's just hope it works" | If you cannot reliably rollback in 10 minutes, you are not ready to deploy. Practice the procedure. |
| "This is a minor change, no need to version it" | Every change to production code is versioned. "Minor" changes break things too. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Changelog** | Generate and review entries, commit | Changelog updated, no placeholders, no WIP markers |
| **Tag** | Create annotated tag, push to remote | Tag is visible on remote, `git show <tag>` works |
| **Deploy** | Build artifact, smoke test, stage, prod, verify | Prod is healthy, monitoring shows normal metrics, users can access |
