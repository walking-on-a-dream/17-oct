---
name: commit-pr-conventions
description: The team's conventions for branch names, commit messages and pull requests. Use whenever creating a branch, writing a commit message, or opening or updating a pull request in this repository.
---

# Commit and PR conventions

## Branches

- Never commit directly to `main`. All changes go through a pull request.
- Name branches `<type>/<short-description>` in kebab-case, e.g. `feat/user-login`, `fix/navbar-overflow`, `docs/setup-guide`.

## Commit messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<optional scope>): <summary>

<optional body: what changed and why>
```

- **Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci`.
- **Summary:** imperative mood ("add", not "added"), lowercase, no trailing period, at most 72 characters.
- **Body:** explain *why* the change was made when it isn't obvious from the diff. Wrap at 72 characters.
- One logical change per commit. Don't mix refactors with features.
- Never commit secrets, API keys, `.env` files or large generated artifacts.

Examples:

```
feat(auth): add Google sign-in
fix: prevent crash when ideas list is empty
docs: add local setup steps to README
```

## Pull requests

- **Title:** same format as a commit summary, e.g. `feat(auth): add Google sign-in`.
- **Body:** use this structure:

  ```markdown
  ## What
  Short description of the change.

  ## Why
  The problem it solves or the idea it implements.

  ## How to test
  Steps a reviewer can follow to verify it.

  ## Screenshots
  (For UI changes; otherwise delete this section.)
  ```

- Keep PRs small and focused; split large work into several PRs.
- Request review from at least one other team member before merging.
- Make sure the branch is up to date with `main` and checks pass before merging.
