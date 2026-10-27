## Project

AI Tutor (working name): an AI doubt-solver for JEE/NEET coaching institutes, grounded in each institute's own notes, DPPs and solutions, with per-student weak-topic tracking for teachers. Read these before planning or building:

- `docs/decisions.md`: decisions so far and why. Don't silently reverse them; add a new entry instead.
- `docs/spec/mvp-spec-v0.1.md`: MVP spec, data model and milestones.
- `docs/research/llm-provider-comparison.md`: Claude vs Gemini and the cost model.

## Team skills

This repo's own skills live in `.claude/skills/`. They hold the team's conventions and take precedence over gstack's defaults where they overlap: when a gstack skill such as /ship commits or opens a PR, follow `commit-pr-conventions`.

## gstack (recommended)

This project uses [gstack](https://github.com/garrytan/gstack) for AI-assisted workflows.
Install it for the best experience:

```bash
git clone --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup --team
```

Skills like /qa, /ship, /review, /investigate, and /browse become available after install.
Use /browse for all web browsing (Aside first, the bundled gstack browser as fallback). Use ~/.claude/skills/gstack/... for gstack file paths.
