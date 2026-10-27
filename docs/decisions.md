# Decisions and context

Everything the team has decided so far about AI Tutor, and why. Newest
decisions at the bottom. When a decision changes, add a new entry that says
which one it replaces instead of editing the old one.

_Started 2026-10-27._

## What we're building

An AI tutor that answers students' doubts using their own coaching institute's
material (notes, DPPs, DPP solutions). It remembers each student's weak topics
and gives teachers a live view of every student and batch. It started from the
team's pitch deck ([pitch-deck.md](pitch-deck.md)). The full plan is in
[spec/mvp-spec-v0.1.md](spec/mvp-spec-v0.1.md).

## Decisions

| # | Decision | Why |
|---|---|---|
| D1 | **This is a real startup**, not a college project. | Team decision. |
| D2 | **First customer:** a single-building coaching institute (no branches) where a team member studied. It prepares students for **JEE Main, JEE Advanced and NEET**: about 10 JEE batches and 5 NEET batches of 40–50 students each, roughly 675 students. | Warm introduction. The institute has agreed to share its material and pilot the product. |
| D3 | **Expansion order:** competitive exams (JEE/NEET) first, then universities and schools. | Start where the pain and the warm customer are. The data model (institution → courses → batches → students) already supports other kinds of institutions. |
| D4 | **The product's place:** today students clear doubts in dedicated in-person doubt sessions, and a few in WhatsApp groups. The tutor solves doubts the moment they come up. Teachers then spend doubt sessions on what the AI couldn't solve, already knowing what each batch is stuck on. | The institute already pays teacher hours for doubt sessions. That's the value we sell. |
| D5 | **Students ask in Hinglish.** The tutor replies in the student's own style. | Matches how students actually talk. |
| D6 | **Mobile-friendly web app (PWA), not a WhatsApp bot.** | Students rarely use WhatsApp for doubts. A web app can render equations and textbook citations properly. |
| D7 | **Language: TypeScript everywhere.** Stack: Next.js + Supabase (Postgres, pgvector, auth, storage) + Vercel. | One language for the web app, the server and a future mobile app. AI response time dominates performance anyway, so a "faster" language wouldn't help. |
| D8 | **How the AI works:** all teacher material (notes, DPPs, DPP solutions) goes into a searchable library. Each doubt pulls the relevant pages (RAG) and checks first whether the problem matches a DPP problem that has a teacher solution. | Material is far too large to send with every doubt. Teacher solutions are the most accurate and cheapest source of answers. |
| D9 | **Student profile:** every doubt is tagged against a fixed JEE/NEET syllabus map, with a topic and the kind of gap (concept, setup, calculation, missing earlier topic). A short per-student summary of weak and strong topics is sent with every new doubt. | Lets the tutor explain *why* a student got stuck. A short summary keeps prompts cheap. |
| D10 | **What teachers see:** rough stats per batch and per student, with the option to dig deeper by asking a teacher assistant about any student in their batches. The assistant is read-only and can read full doubts. | Team decision. Students must be told clearly what teachers can see. |
| D11 | **Usage estimate: about 10 doubts per student per day.** | Team estimate from their own coaching experience. Drives the cost model. |
| D12 | **AI provider (Claude vs Gemini) is decided by an eval, not by sticker price.** 50 real doubts from the institute are run through Claude Sonnet 5.5, Claude Opus 5.5, the Gemini flagship and Gemini Flash. Teachers grade the answers, and we pick the cheapest configuration that's ≥ 95% correct. All AI calls go through one module (`src/lib/ai/`) so switching providers is a small change. | A wrong JEE Advanced answer costs more than any API bill. See [research/llm-provider-comparison.md](research/llm-provider-comparison.md). |
| D13 | **Pilot on 2 batches first** (1 JEE + 1 NEET, about 90 students) before rolling out to all 15. | Keeps AI cost and risk small while we learn. |

## Still open

1. Who at the institute grades the eval and reviews flagged answers?
2. Are DPP solutions available for most DPPs, and are they typed or handwritten?
3. Price to the institute per student per month (proposed ₹300–500).
4. Which 2 batches run the pilot, and when?
5. Product name and domain. "AI Tutor" is a working name.
6. Legal check: most students are under 18. Confirm with a lawyer how India's DPDP Act 2023 and the DPDP Rules 2025 (children's data, parental consent, exemptions for educational institutions) apply before the pilot.
