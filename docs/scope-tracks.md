# Scope vs Schedule: Two Build Tracks

**Decision needed by:** Mon May 25 (so coding starts on a locked plan)
**Available hours:** ~45–75 across 20 days, solo
**Submission deadline:** Thu Jun 11, 2026

This document presents two builds — **Track A (Ambitious)** and **Track B (Lean)** — so you can pick one with eyes open. Both are designed to win something at this hackathon. They optimize for different risks.

---

## The honest framing

A hackathon submission can win on **breadth** (look at everything we integrated) or **depth** (look at how well this one thing works). With 45–75 solo hours, breadth is risky because anything half-finished hurts more than it helps — a judge who sees a broken Arize trace remembers the broken trace, not the working calendar integration.

**Track A** bets on breadth. Three partner MCPs, all wired, plan generation + tracking + nudges all functional. Eligible for three prize tracks. High ceiling, real risk of something being broken on submission day.

**Track B** bets on depth. One partner MCP done extremely well, plus a clean hero demo. Eligible for one prize track but with a much stronger shot at winning it. Lower ceiling, much lower risk.

There's no objectively right answer. It depends on your appetite for risk and how the next three weeks actually feel.

---

## Side-by-side comparison

| Dimension | Track A (Ambitious) | Track B (Lean) |
|---|---|---|
| **Partner MCPs** | MongoDB + Elastic + Arize | MongoDB only (Elastic stubbed, Arize aspirational) |
| **Prize tracks eligible** | 3 ($15K top-prize ceiling) | 1 ($5K top-prize ceiling) |
| **Agent topology** | Orchestrator + 4 sub-agents | Orchestrator + 2 sub-agents (Profile + Planner) |
| **PDF parsing** | Gemini multimodal, full extraction | Gemini multimodal, basic fields only |
| **Plan generation** | Workouts + meals + cheat days + recipes, RAG-grounded via Elastic | Workouts + meals, grounded in a small hand-curated JSON of recipes/guidelines |
| **Calendar write** | Full week, every event with rationale | Full week, simple titles |
| **Google Fit tracking** | Hourly poll, adherence scoring | Manual "did you do it?" checkbox; Fit as stretch goal |
| **Nudges** | Scheduled, Gemini-drafted, email delivery | Skipped for v1, mentioned in future work |
| **Dashboard** | Today's plan + 7-day adherence + body comp trend | Today's plan only |
| **Observability** | Arize traces every agent run + safety evals | Console logging; mention Arize as roadmap |
| **Demo hero moment** | "Upload → calendar in 60s, with adherence, nudge, Arize trace" | "Upload → calendar in 60s, with the plan rationale clearly visible" |
| **Total hours (estimate)** | 90–110 | 45–60 |
| **Hours buffer** | Negative (over budget) | Positive (room to polish) |
| **Risk of unfinished work** | High — likely something looks half-baked | Low — every shown feature is solid |
| **Risk of looking generic** | Low — multi-partner integration stands out | Medium — needs strong design + demo to differentiate |

---

## Track A — Ambitious (three partners)

### What you submit

A multi-agent system on Google Cloud Agent Builder with Orchestrator, Profile, Planner, Tracker, and Nudge agents. MongoDB stores user state. Elastic indexes a recipe + exercise + guideline corpus and serves hybrid search. Arize traces every agent run and shows safety eval results. Web app on Cloud Run + Firebase. Hosted URL, public GitHub repo with MIT license, three-minute demo.

### Where the hours go (rough cut, ~95 hours)

- Account setup, repo, OAuth, Google Cloud Agent Builder spike: **8 hrs**
- MongoDB schema + writes + Atlas Vector Search setup: **10 hrs**
- Elastic Cloud setup + corpus curation + ingestion + hybrid search wiring: **15 hrs**
- Profile agent + Gemini multimodal PDF parsing + confirmation UI: **12 hrs**
- Planner agent + RAG retrieval + Gemini plan generation + calendar write: **15 hrs**
- Tracker agent + Google Fit pull + adherence scoring: **10 hrs**
- Nudge agent + scheduler + email delivery: **8 hrs**
- Arize instrumentation + safety evals + trace dashboard: **8 hrs**
- Web dashboard (today + trends): **6 hrs**
- Deployment, debugging, polish: **6 hrs**
- Demo video + README + Devpost form: **5 hrs**

### Honest risks

Your hours are ~45–75. Track A needs ~95. The math doesn't work without one of: more hours than estimated, AI-pair-programming velocity gains (real but uneven), or cutting corners on testing/polish in the final week. The hardest cuts always happen in the last 48 hours when you're tired — that's exactly when things break in front of judges.

The **Elastic corpus** is the hidden hour-sink. Curating 100+ recipes with conditions tags, 50+ exercises with contraindications, and a clinical guideline set — that's where the schedule blows up. You'd want to use existing datasets (Open Food Facts, public exercise lists) and minimize curation, but even ingestion + tuning hybrid search takes real time.

### When Track A makes sense

If you can find 10–15 more hours than you estimated (a hackathon often pulls more out of people than they predict, in both directions), if you're comfortable with the risk of one of three integrations being shaky on submission day, and if maximizing prize-track exposure matters more to you than maximizing odds of placing in any one track.

---

## Track B — Lean (one partner, depth)

### What you submit

A two-agent system on Google Cloud Agent Builder: Profile agent and Planner agent, orchestrated. MongoDB stores user state and the recipe/exercise/guideline corpus (small, hand-curated JSON loaded into a MongoDB collection — using Atlas Vector Search where it shines, e.g., "find a meal similar to one this user enjoyed"). Web app on Cloud Run + Firebase. Hosted URL, public GitHub repo, three-minute demo.

Elastic and Arize are **not in the build**. They appear in the README's "roadmap" section with a clear story for why they'd be added next.

### Where the hours go (rough cut, ~52 hours)

- Account setup, repo, OAuth, Agent Builder spike: **8 hrs**
- MongoDB schema + writes + Atlas Vector Search for personal similarity: **8 hrs**
- Curated corpus (30 recipes, 20 exercises, condition tags) loaded into MongoDB: **5 hrs**
- Profile agent + Gemini multimodal PDF parsing + confirmation UI: **10 hrs**
- Planner agent + Gemini plan generation grounded in MongoDB corpus + calendar write: **12 hrs**
- Web dashboard (today's plan with rationale, simple "completed" checkboxes): **5 hrs**
- Deployment + debugging + polish: **3 hrs**
- Demo video + README + Devpost form: **5 hrs**

### Why this is actually competitive

The MongoDB judges are looking for projects that demonstrate **why MongoDB specifically** — not projects that use MongoDB as a Postgres substitute. Track B uses Atlas Vector Search for a genuinely good reason (personal meal similarity is a vector problem) and uses MongoDB's document model for naturally nested health data (a profile with conditions, scans, history is a perfect document). That story, well told, beats a project that wired up three partners but used MongoDB as a generic key-value store.

The depth-over-breadth bet also pays off on the **Design** and **Quality of Idea** judging criteria. A polished single-partner project shows better than three rushed integrations.

### When Track B makes sense

If you want to sleep before the demo, if you want every feature shown in the video to actually work, if you'd rather have a 30% chance at MongoDB's $5K than a 10% chance each at three $5K prizes (the math is roughly equivalent in expected value, but the variance is very different), and if the design + idea-quality criteria matter to you as much as technological breadth.

---

## The hybrid I'd actually recommend: Track B+

If you want my honest read after thinking through this — neither track as written is quite right. Here's the version I'd build:

**Track B+ (~65 hours):**
- Everything in Track B
- **Plus** Arize lightweight: just trace every agent run, no safety evals, no custom dashboard — Arize's own UI shows the traces. This is maybe 4 extra hours and unlocks the Arize prize track. The story is "we instrumented for observability from day one because health advice deserves auditability."
- **Plus** a one-paragraph honest framing in the README: "We chose to integrate MongoDB and Arize deeply rather than spread thin across all partners. Here's why each one is structurally necessary for a trustworthy health agent."

This wins on:
- MongoDB track: Vector Search + document model used meaningfully
- Arize track: Real traces of real agent decisions, with the trust angle
- Design track (in any partner judging): Depth shows
- Quality of Idea: The condition-aware health agent thesis is strong

This loses on:
- Elastic track: Not eligible
- "Look at all three partners" optics: Mitigated by the honest README framing

**Total hours: ~65, which fits comfortably in your 45–75 window with buffer.**

---

## Decision matrix

| Your situation | Pick |
|---|---|
| You find energy and time you didn't expect; you want max prize exposure | **Track A** |
| You want the safest path to placing in one track; you value polish | **Track B** |
| You want the best risk-adjusted shot at placing, with two prize tracks live | **Track B+ (recommended)** |

---

## What this means for today and tomorrow

Regardless of track:

**Today (1–2 hrs, low energy):** Read this doc. Decide track. That's it.

**Tomorrow (1–2 hrs, low energy):**
- Create GitHub repo with MIT license
- Register on Devpost
- Sign up for Google Cloud, MongoDB Atlas, (and Arize if going B+ or A; Elastic if going A)
- Resolve open questions: recipe corpus source, Fitness REST vs Health Connect

Track decision drives everything from Monday onward, so we don't lock the day-by-day schedule until you decide.
