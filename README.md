# AI Tutor

_Working name; a proper product name will come later._

An AI tutor for coaching institutes. Students ask doubts, typed or as a photo,
in Hinglish. The tutor answers from their own institute's notes, DPPs and DPP
solutions, with page references. It remembers each student's weak topics, and
teachers get a live view of every student and batch. We start with JEE and NEET
coaching, then expand to universities and schools.

## Docs

- [Decisions and context](docs/decisions.md): what we've decided and why
- [MVP spec v0.1](docs/spec/mvp-spec-v0.1.md): the full build plan
- [Claude vs Gemini comparison](docs/research/llm-provider-comparison.md): provider options and cost model
- [Pitch deck](docs/pitch-deck.md): the original idea

Team:
  1. Kalp
  2. Divyansh
  3. Akshat
  4. Muni
  5. Pranav


Ideas:

### AI Tutor
- **Pitch:** An AI doubt-solver for JEE/NEET coaching students that answers from their own institute's material and tracks each student's weak topics for their teachers.
- **Problem:** Students wait days for doubt sessions, generic chatbots don't teach the way their institute does, and teachers don't know what each student is stuck on.
- **MVP:** Teacher uploads notes/DPPs/solutions; student asks a doubt by text or photo and gets a cited, Hinglish answer; each doubt is tagged by topic to build a student profile; teacher dashboard with batch stats; teacher assistant to ask about any student.
- **Stretch goals:** Native mobile app, practice tests from weak topics, universities and schools, WhatsApp access.
- **Tech stack:** TypeScript, Next.js, Supabase (Postgres + pgvector), Claude or Gemini API (chosen by eval), Vercel.
- **Risks:** Wrong answers on hard problems, AI cost at 10 doubts/student/day, poor scans of handwritten material, student data privacy (most students are minors).
- **Proposed by:** The team
