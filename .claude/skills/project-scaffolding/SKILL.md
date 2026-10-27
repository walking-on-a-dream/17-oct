---
name: project-scaffolding
description: Scaffold a new project or app in this repository with a consistent structure and tooling. Use when someone wants to start building an idea, set up a new web app, API, CLI or script project, or add a new package to the repo.
---

# Project scaffolding

Set up new projects so any team member can clone the repo and run them within minutes.

## 1. Confirm the essentials

Get these from the request, or the idea's entry in `README.md`, before generating files. Ask only for what's missing:

- **Kind of project:** web frontend, backend API, full-stack app, CLI, data/ML notebook, etc.
- **Stack:** if the team hasn't chosen one, propose a default from the table below and say why.
- **Location:** put each project in its own top-level folder named in kebab-case (e.g. `study-buddy/`), so several ideas can live in the repo side by side.

| Kind | Suggested default |
| --- | --- |
| Web frontend | Vite + React + TypeScript |
| Backend API | Python + FastAPI, or Node + Express + TypeScript |
| Full-stack | Next.js + TypeScript |
| CLI / scripts | Python 3 with `argparse` |
| Data / ML | Python + Jupyter, `pandas`, `scikit-learn` |

Use the official generator where one exists (e.g. `npm create vite@latest`) rather than writing boilerplate by hand.

## 2. Required files

Every project folder must have:

- `README.md` with: one-line description, prerequisites, setup steps, how to run, how to test, and the folder layout.
- `.gitignore` suited to the stack (dependencies, build output, `.env`, editor/OS files).
- `.env.example` listing every environment variable with placeholder values, if the project uses any. Never commit a real `.env`.
- A dependency manifest with pinned or locked versions (`package.json` + lockfile, `requirements.txt` or `pyproject.toml`).
- A formatter and linter configured (e.g. Prettier + ESLint, or Ruff for Python) with a script to run them.
- At least one passing test and a script to run the tests.

## 3. Structure

Keep it simple and conventional for the stack, for example:

```
<project-name>/
├── README.md
├── .gitignore
├── .env.example
├── src/          # application code
└── tests/        # tests
```

## 4. Verify before finishing

1. Install dependencies from scratch.
2. Run the app and confirm it starts.
3. Run the linter and tests; they must pass.
4. Follow the steps in the project's README exactly as written, to make sure they work.

Then add a link to the project folder under its idea in the root `README.md`, and commit following the `commit-pr-conventions` skill.
