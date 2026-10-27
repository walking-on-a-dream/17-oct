---
name: code-review-checklist
description: Review code changes against the team's checklist before they are merged. Use when asked to review a pull request, a branch, a diff, or "my changes", or before merging anything into main.
---

# Code review checklist

Review the change, not the whole codebase. Start by getting the diff (`git diff main...HEAD` for a branch, or the PR's changed files), and read enough surrounding code to understand it.

## Checklist

### Correctness
- [ ] Does the code do what the PR description says?
- [ ] Are edge cases handled: empty inputs, `null`/`None`, errors, network failures?
- [ ] Any obvious bugs, off-by-one errors or race conditions?

### Security
- [ ] No secrets, API keys, tokens or passwords in code or config.
- [ ] User input is validated; no SQL injection, XSS or command injection.
- [ ] New dependencies are well known and maintained.

### Tests
- [ ] New behaviour has tests, and bug fixes have a test that would have caught the bug.
- [ ] The test suite and linter pass. Run them; don't assume.

### Readability
- [ ] Names are clear and consistent with the surrounding code.
- [ ] No dead code, leftover debug prints or commented-out blocks.
- [ ] Complex logic has a short comment explaining *why*.

### Scope and hygiene
- [ ] The PR is focused on one change; unrelated edits are split out.
- [ ] Commits and the PR title follow the `commit-pr-conventions` skill.
- [ ] Docs and READMEs are updated if setup or behaviour changed.

## Reporting

Group findings by severity, most important first:

- **Blocking:** bugs, security issues, failing tests. Must be fixed before merge.
- **Should fix:** missing tests, unclear code, convention violations.
- **Nit:** small style suggestions. Optional.

For each finding, give the file and line, what is wrong, and a concrete suggested fix. If nothing is blocking, say so clearly. Briefly note what was done well, too.
