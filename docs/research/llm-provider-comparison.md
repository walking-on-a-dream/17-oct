# Claude vs Gemini for AI Tutor

_Researched 2026-10-27. Prices change often; re-check before deciding._

**Decision:** pick the provider using the M0 eval (decision D12 in
[../decisions.md](../decisions.md)), not on price alone.

## Price per million tokens

| Tier | Claude | Gemini |
|---|---|---|
| Top | Opus 5.5: $4 in / $20 out (cached input $0.20) | Flagship: $2 / $10 until 30 Nov 2026, reportedly $4 / $20 after |
| Middle | Sonnet 5.5: $2 / $10 (cached input $0.20) | Newest Flash: $0.75 / $3.75 (introductory), reportedly $1.50 / $7.50 from Jan 2027 |
| Cheapest | Haiku 5.5: $0.10 / $0.50 | Flash-Lite: $0.25 / $1.50 |

Claude prices are Anthropic's published API rates. The Gemini prices come from
third-party sites (Google's official page,
ai.google.dev/gemini-api/docs/pricing, couldn't be reached during research),
and those sites disagree on model names, so verify the official page.

## Pros and cons

**Claude**
- ✅ Built-in citations: send PDF pages and it returns the exact page each sentence came from. That covers the "source tags" feature directly.
- ✅ Strong step-by-step reasoning, which JEE Advanced needs.
- ✅ Prompt caching makes re-sending the same material cheap.
- ✅ Handles Hinglish and photos of problems.
- ✅ Stable model lineup. Prices aren't introductory offers.
- ❌ No embeddings API, so a second provider is needed for search (Voyage AI, Gemini embeddings, or an open-source model).
- ❌ No ongoing free tier.

**Gemini**
- ✅ Free tier for prototyping. Check whether free-tier data is used to improve Google products; that's not acceptable for student data.
- ✅ Embeddings in the same API, so one provider covers everything.
- ✅ Strong with images, video and long documents.
- ✅ Google for Startups credits are available.
- ❌ Introductory prices are ending, so the budget could double within months.
- ❌ Frequent model renames and retirements (e.g. 2.5 Flash-Lite retired 16 Oct 2026).
- ❌ Page-level citations for uploaded files not verified. Check before choosing.

_Note: this comparison was written with the help of Claude, made by Anthropic.
That's why the decision rests on an eval that teachers grade._

## Cost model

Assumptions: 675 students, **10 doubts per student per day**, 25 days a month,
which is 168,750 doubts a month. Each doubt sends about 20K tokens of cached
course material plus 2K new tokens, and gets back an answer of about 1.5K
tokens.

| Model tier | Per doubt | Per month (675 students) | Per student per month |
|---|---|---|---|
| Claude Opus 5.5 | ~$0.042 | ~$7,090 | ~₹890 |
| Claude Sonnet 5.5 / Gemini flagship | ~$0.023 | ~$3,880 | ~₹490 |
| Gemini Flash (introductory price) | ~$0.01 | ~$1,690 | ~₹210 |
| Claude Haiku 5.5 | ~$0.003 | ~$510 | ~₹64 |

**Target: ≤ ₹150 per student per month.** Ways to get there:

1. **Reuse DPP solutions.** A whole batch does the same DPP, so problems that match a teacher solution are answered from it. That's cheaper, and more accurate too.
2. **Route by difficulty:** factual doubts go to a cheap model, multi-step numericals to a strong one.
3. **Prompt caching** for course material (already counted above).
4. **A fair-use daily limit** per student.

The pilot on 2 batches (about 90 students) costs roughly $500 a month even on
the middle tier.

## Sources

- [Gemini API Pricing (October 2026), BenchLM](https://benchlm.ai/google/api-pricing)
- [Google Vertex AI pricing in 2026, CloudZero](https://www.cloudzero.com/blog/google-vertex-ai-pricing/)
- [Gemini API Pricing: Full Breakdown of Costs (Oct 2026), Puter](https://developer.puter.com/tutorials/gemini-api-pricing/)
- [Gemini API Pricing in 2026, GeoToolbox](https://geotoolbox.ai/blog/gemini-api-pricing)
- [Gemini API Pricing 2026, AIFreeAPI](https://www.aifreeapi.com/en/posts/gemini-api-pricing-2026)
- [Gemini API Pricing May 2026, MetaCTO](https://www.metacto.com/blogs/the-true-cost-of-google-gemini-a-guide-to-api-pricing-and-integration)
