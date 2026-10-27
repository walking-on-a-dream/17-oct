# AI Tutor — MVP Spec v0.1 (draft)

_Working name. Status: draft for team review. Date: 2026-10-27._

## 1. Summary

An AI tutor for a coaching institute that answers students' doubts using the
institute's own material: notes, DPPs and DPP solutions. It remembers each
student's weak topics, and gives teachers a live view of every student and
batch. The first customer is a single-building JEE Main/Advanced + NEET
institute: 15 batches (10 JEE, 5 NEET) of 40–50 students, about 675 students.
Schools and universities come later.

## 2. Why this matters

| Who | Today | With the tutor |
|---|---|---|
| **Student** | Gets stuck on a DPP problem at night and waits days for the next doubt session. Generic chatbots answer with methods the institute doesn't teach. | Gets an instant answer in Hinglish, explained the way their teacher explains it, with the exact page in their notes. |
| **Teacher** | Spends whole doubt sessions answering the same questions repeatedly, without knowing beforehand what the batch is stuck on. | Walks into doubt sessions knowing each batch's weak topics, and can ask "tell me about Rahul" to get a full picture of one student. |
| **Institute owner** | Pays teachers for doubt-session hours. Has no data on student progress between tests. | Covers more doubts with the same teachers, and has a selling point for admissions: "24×7 doubt solving". |

## 3. Users and roles

| Role | Can do |
|---|---|
| **Student** | Ask doubts (text or photo), have follow-up conversations, see their own weak and strong topics. |
| **Teacher** | Upload material for their courses, see stats for their batches, drill into any student in those batches, ask the teacher assistant about students. |
| **Institute admin** | Manage batches, students and teachers. Sees everything in the institution. |
| **Platform admin (us)** | Manage institutions, view costs and logs. |

## 4. Pilot success metrics

| Metric | Target | How it's measured |
|---|---|---|
| Answer correctness | ≥ 95% on the eval set | Teachers grade the M0 eval set (§10) |
| Doubts resolved without a teacher | ≥ 60% | Student taps "Solved" or "Still stuck" after each doubt |
| Weekly active students | ≥ 50% of pilot students by week 4 | Students with ≥ 1 doubt in the week |
| Time until the answer starts appearing | p50 < 3 s, p95 < 8 s | Server timing |
| AI cost | ≤ ₹150 per student per month at 10 doubts/day | Tokens and cost logged on every AI call (§9) |
| Teacher usage | Each pilot teacher opens the dashboard ≥ 2× per week | Page analytics |

## 5. Scope

### In scope (MVP)

1. Teachers upload PDFs (notes, DPPs, DPP solutions), including scanned pages.
2. Students ask doubts as typed text or a photo. Answers stream in, are grounded in the material, cite pages, and are written in Hinglish.
3. A student's problem is matched against DPP problems. When it matches, the answer is explained from the teacher's own solution.
4. Follow-up questions within a doubt conversation.
5. Each doubt is tagged against a JEE/NEET syllabus map: topic plus the kind of gap.
6. A per-student profile summary that feeds into every answer.
7. Teacher dashboard with batch stats and drill-down to individual students.
8. Teacher assistant: "ask about any student or batch" (read-only).
9. Phone-friendly web app that students can install to their home screen (PWA).
10. Cost tracking and a fair-use daily limit per student.

### Out of scope (MVP)

- Native Android/iOS apps (the installable web app covers this for now)
- WhatsApp bot
- Universities and schools, and subjects outside JEE/NEET
- Auto-generated tests, mock exams and rankings
- Video lectures
- Payments or billing inside the app (the institute is invoiced manually)
- Teachers editing AI answers inline (they can flag an answer instead, see M7)
- More than one institution in the pilot (the data model supports it; no UI for it yet)

## 6. How it works

```
TEACHER UPLOAD
PDF ─► extract text per page (scanned pages read by a vision model)
    ─► split into chunks with page numbers ─► embed ─► `chunks`
    ─► DPPs: extract each problem ─► `dpp_problems`
    ─► DPP solutions: link each solution to its problem

STUDENT DOUBT
question (text / photo → text)
    ─► 1. DPP match: nearest `dpp_problems` above a similarity threshold?
    ─► 2. retrieve top-k `chunks` from the student's courses
    ─► 3. load the `student_profiles` summary
    ─► 4. AI answers (streamed): uses the teacher solution if matched,
          cites pages, Hinglish, step by step
    ─► 5. save the conversation; tag the doubt in the background
          (topic + gap type); update the profile in the background

TEACHER
dashboard (batch stats, per-student view)
    └─► teacher assistant: read-only tools over doubts/profiles, scoped to the teacher's batches
```

## 7. Architecture

| Part | Choice | Why |
|---|---|---|
| Language | **TypeScript** everywhere | One language for the web app, server and a future mobile app |
| App | **Next.js** (App Router), mobile-first PWA | Student app, teacher app and API in one codebase |
| Database + vector search | **Postgres + pgvector** via **Supabase** | Database, auth, file storage and vector search in one service; free tier for the pilot |
| Auth | Supabase Auth, phone number + OTP | Students in India use phones; accounts are pre-created by the institute admin |
| File storage | Supabase Storage | PDFs and doubt photos |
| AI provider | **Decided by the M0 eval** (Claude vs Gemini) | Pick the cheapest model that meets the 95% correctness bar |
| Embeddings | Decided with the provider (Gemini embeddings, Voyage AI, or open-source) | Claude has no embeddings API |
| Hosting | Vercel | Free/cheap at pilot scale |

**Rule:** every AI call goes through one module, `src/lib/ai/`, with these
functions: `answerDoubt`, `tagDoubt`, `summarizeProfile`, `extractPage`,
`embed`, `teacherAgent`. No other file imports a provider SDK, so switching
providers means changing that one module.

## 8. Data model (Postgres)

```sql
create type role as enum ('student','teacher','admin');
create type exam as enum ('jee_main','jee_advanced','neet');
create type subject as enum ('physics','chemistry','maths','biology');
create type doc_kind as enum ('notes','dpp','dpp_solution','other');
create type gap_type as enum ('concept','setup','calculation','prerequisite','other');

create table institutions (id uuid primary key, name text not null);

create table users (
  id uuid primary key references auth.users,
  institution_id uuid not null references institutions,
  role role not null, name text not null, phone text unique
);

create table batches (
  id uuid primary key, institution_id uuid not null references institutions,
  name text not null, exam exam not null
);
create table batch_members (batch_id uuid references batches, user_id uuid references users,
  primary key (batch_id, user_id));

create table courses (
  id uuid primary key, institution_id uuid not null references institutions,
  exam exam not null, subject subject not null, name text not null
);
create table batch_courses (batch_id uuid references batches, course_id uuid references courses,
  primary key (batch_id, course_id));

-- syllabus map, seeded once per exam
create table topics (
  id serial primary key, exam exam not null, subject subject not null,
  chapter text not null, topic text not null, unique (exam, subject, chapter, topic)
);

create table documents (
  id uuid primary key, course_id uuid not null references courses,
  kind doc_kind not null, title text not null, storage_path text not null,
  page_count int, status text not null default 'processing', -- processing|ready|failed
  uploaded_by uuid references users, created_at timestamptz default now()
);

create table chunks (
  id bigserial primary key, document_id uuid not null references documents on delete cascade,
  page_from int not null, page_to int not null, text text not null,
  embedding vector(1024)  -- dimension fixed once the embedding model is chosen
);

create table dpp_problems (
  id bigserial primary key, document_id uuid not null references documents on delete cascade,
  problem_no text not null, page int not null, text text not null,
  solution_chunk_id bigint references chunks, embedding vector(1024)
);

create table conversations (            -- one doubt = one conversation
  id uuid primary key, student_id uuid not null references users,
  batch_id uuid not null references batches, course_id uuid references courses,
  matched_dpp_problem_id bigint references dpp_problems,
  status text not null default 'open', -- open|solved|stuck
  created_at timestamptz default now()
);

create table messages (
  id bigserial primary key, conversation_id uuid not null references conversations on delete cascade,
  role text not null,                  -- student|assistant
  text text not null, image_path text,
  citations jsonb,                     -- [{document_id, page, quote}]
  model text, tokens_in int, tokens_out int, tokens_cached int, cost_usd numeric(10,6),
  created_at timestamptz default now()
);

create table doubt_tags (
  conversation_id uuid references conversations on delete cascade,
  topic_id int references topics, gap gap_type not null,
  prerequisite_topic_id int references topics, primary key (conversation_id, topic_id)
);

create table student_profiles (
  student_id uuid primary key references users,
  summary text not null,               -- ≤ 120 words, injected into prompts
  weak_topics jsonb not null,          -- [{topic_id, score, doubts}]
  strong_topics jsonb not null,
  updated_at timestamptz default now()
);
```

Every table that holds student data gets row-level security: students see only
their own rows, teachers only their batches, admins only their institution.

## 9. AI behaviour

**Answering a doubt (`answerDoubt`)**

1. If the photo or text matches a DPP problem with similarity ≥ threshold (tuned in M0), explain using the linked teacher solution. Don't re-solve it differently.
2. Otherwise, answer using the retrieved chunks (top-k = 8, filtered to the student's batch courses).
3. Cite pages for definitions, formulas and worked examples taken from the material. If the material doesn't cover the doubt, say so ("Yeh tumhare notes mein nahi hai, general method:") and answer anyway.
4. Reply in the student's style: Hinglish if they wrote in Hinglish, English if English. Equations are rendered with KaTeX.
5. Use the student profile: if the doubt touches a weak topic or a missing earlier topic, say so explicitly ("Yeh step isliye mushkil laga kyunki vectors ka cross product…").
6. Solve step by step for numericals, and end with one similar practice problem from the DPPs when one exists.
7. Prompt caching: the stable system prompt and instructions come first, and the per-doubt content comes after the cache breakpoint.

**Tagging (`tagDoubt`, runs in the background after the first answer)** uses a cheap model with structured output:
`{ topics: [{topic_id, gap, prerequisite_topic_id?}], confidence }`. Only
`topic_id`s from the seeded syllabus are accepted, and anything else is rejected.

**Profile update (`summarizeProfile`)** runs after every 5 new doubts or nightly, whichever comes first.
It recomputes `weak_topics` from `doubt_tags` (doubt count weighted by
"stuck" status and how recent they are) and rewrites the ≤ 120-word summary. A
student with few doubts is shown as "not enough data", never as "strong".

**Teacher assistant (`teacherAgent`)** uses tool use with read-only tools, scoped server-side to
the teacher's batches:

| Tool | Returns |
|---|---|
| `list_students(batch_id)` | names, doubt counts, top weak topics |
| `get_student_profile(student_id)` | profile and topic scores |
| `search_doubts(student_id?, batch_id?, topic_id?, gap?, from?, to?)` | doubt list (question + status + tags) |
| `get_conversation(conversation_id)` | the full conversation |
| `batch_topic_stats(batch_id, from, to)` | per-topic doubt and stuck counts |

The assistant must cite which doubts its claims come from, and must refuse
questions about students outside the teacher's batches.

**Cost model at 10 doubts per student per day** (675 students × 10 × 25 days = 168,750 doubts/month):

| Model tier | Per doubt | Per month | Per student per month |
|---|---|---|---|
| Strong (Claude Opus 5.5) | ~\$0.042 | ~\$7,090 | ~₹890 |
| Middle (Claude Sonnet 5.5 / Gemini flagship) | ~\$0.023 | ~\$3,880 | ~₹490 |
| Gemini Flash (introductory price) | ~\$0.01 | ~\$1,690 | ~₹210 |
| Cheap (Claude Haiku 5.5) | ~\$0.003 | ~\$510 | ~₹64 |

Two levers have to get us to ≤ ₹150 per student per month:
- **DPP-solution reuse:** a matched DPP problem is answered from the stored solution, using a shorter prompt or a cheaper model.
- **Routing:** simple factual doubts go to the cheap tier, and multi-step numericals go to the strong tier.

M0 and the pilot will measure whether this target is realistic. The **pilot
runs on 2 batches (1 JEE, 1 NEET, ≈ 90 students)**, which costs about \$500/month
even on the middle tier.

## 10. Milestones (child issues)

| # | Milestone | Effort (human team / with Claude Code) | Depends on |
|---|---|---|---|
| M0 | **Model eval:** 50 real doubts (JEE Adv, JEE Main, NEET, photos, Hinglish) run through 4 models; teachers grade correctness and clarity; pick provider and routing | 1 wk / 2 days | — |
| M1 | **Foundation:** repo, Next.js, Supabase, schema + row-level security, phone OTP login, roles, seed institution/batches/students | 1.5 wk / 2 days | — |
| M2 | **Library:** upload, page text extraction including scanned pages, chunking, embeddings, DPP problem extraction + solution linking, upload status UI | 2 wk / 4 days | M1, embedding choice from M0 |
| M3 | **Doubt chat:** text/photo input, DPP match, retrieval, streamed answer with citations + KaTeX, follow-ups, Solved/Stuck buttons | 2 wk / 4 days | M0, M2 |
| M4 | **Tagging + profiles:** JEE/NEET syllabus seed, `tagDoubt`, `summarizeProfile`, profile used in answers | 1.5 wk / 3 days | M3 |
| M5 | **Teacher dashboard:** batch topic heatmap, student list, student detail page | 1.5 wk / 3 days | M4 |
| M6 | **Teacher assistant** over the M5 data | 1 wk / 2 days | M5 |
| M7 | **Pilot hardening:** fair-use limit, cost dashboard, error logging, "flag wrong answer" button, PWA install, student privacy notice | 1 wk / 2 days | M3 |

```
M0 Eval ──────────────┐
M1 Foundation ─► M2 Library ─► M3 Doubt chat ─► M4 Tags+profiles ─► M5 Dashboard ─► M6 Teacher assistant
                                    └─► M7 Pilot hardening
```

**Why this order:** M0 comes first because the provider choice affects the
embedding size (M2) and every prompt (M3). M3 before M4 because tagging needs
real doubts to tune against. The dashboard needs tags.

## 11. Acceptance criteria (MVP)

1. A teacher uploads a 50-page scanned DPP PDF. Within 10 minutes it shows "ready", with ≥ 90% of its problems extracted into `dpp_problems` (spot-checked against the PDF).
2. A student photographs a problem from an uploaded DPP. The answer cites that DPP's problem number and uses the linked teacher solution, in ≥ 9 of 10 test photos.
3. Every answer that uses a definition or formula from the material includes a citation with document and page, and tapping it opens that page.
4. A student who writes in Hinglish gets a Hinglish reply, in ≥ 9 of 10 test doubts.
5. M0 eval: the chosen configuration scores ≥ 95% correct on the 50-doubt set, as graded by 2 teachers.
6. Every conversation has ≥ 1 `doubt_tags` row with a `topic_id` from the seeded syllabus, within 60 s of the first answer.
7. After 5 doubts on one topic marked "stuck", that topic appears in the student's top-3 weak topics.
8. A teacher sees only their own batches. A request for another batch's student returns 403 (tested through the API and through the teacher assistant).
9. A student can never read another student's conversations (row-level security test).
10. Every AI call logs model, tokens and `cost_usd`. The cost dashboard total matches the sum of `messages.cost_usd` within 1%.
11. Hitting the daily fair-use limit shows a clear message and blocks further AI calls until midnight IST.
12. Each test in §12 states what it protects, what wrong result it catches, and what it does not prove.

## 12. Testing plan

| Layer | What | Count |
|---|---|---|
| Unit | chunking keeps page numbers; DPP-match threshold; profile scoring; cost calculation; fair-use limit | +15 |
| Integration | upload → ready → retrievable; doubt → answer with citations → tags → profile; row-level security per role | +10 |
| Eval | M0 50-doubt set, re-run on every prompt or model change | 50 cases |
| E2E | student login → photo doubt → answer → Solved; teacher login → batch → student → ask the assistant | +2 |

## 13. Privacy and safety

- **Most students are under 18.** India's DPDP Act 2023 has extra rules for children's data, including parental consent and limits on tracking and behavioural monitoring. The DPDP Rules 2025 include exemptions for educational institutions. **Get legal advice before the pilot** on whether those exemptions cover this product, with the institute as data owner and us as processor.
- Tell students clearly what teachers can see: stats, plus full doubts through the teacher assistant.
- The contract with the institute must say who owns the uploaded material and the student data, and that the material isn't used for other customers.
- Choose an AI provider plan whose terms don't use our data for training. Free tiers often do.
- A "flag wrong answer" button sends the answer to a teacher review queue (M7).

## 14. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Wrong answers on hard JEE Advanced problems | Students and the institute lose trust | M0 eval bar, answers from teacher solutions first, flag button |
| Poor text extraction from scanned or handwritten PDFs | Bad retrieval and citations | Vision-model extraction; ask the institute for typed/digital originals |
| AI cost over budget | Margins disappear | DPP reuse, routing, fair-use limit, pilot on 2 batches |
| Students don't adopt it | No data, no value | Teachers promote it in class; doubt sessions start from the dashboard |
| Students avoid asking because teachers can see doubts | Fewer doubts | Clear notice; framing as "your teacher uses this to help you" |
| Single customer dependency | Startup risk | Treat the pilot as a case study for the next 3 institutes |

## 15. Open questions

1. Who at the institute signs off on answer quality (grades M0, reviews flags)?
2. Are DPP solutions available for most DPPs, and are they typed or handwritten?
3. Price to the institute, per student per month? (Proposed range: ₹300–500.)
4. Which 2 batches run the pilot, and when does it start?
5. Product name and domain.
