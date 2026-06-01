# Requirements Analysis Document

**Project:** Agentic Health Coach
**Hackathon:** Google Cloud Rapid Agent Hackathon (May 1 – Jun 11, 2026)
**Document version:** 0.1 (draft for review)
**Last updated:** May 23, 2026

---

## 1. Project summary

### 1.1 One-liner

An agentic health coach that turns a user's body composition report and medical conditions into a Gemini-planned, calendar-scheduled week of workouts, meals, and recipes — grounded in a clinical knowledge base via Elastic, personalized through MongoDB Atlas vector search, and made trustworthy by Arize observability over every agent decision.

### 1.2 Problem statement

Body composition machines like InBody, Boditrix, and Evolt give consumers a rich snapshot of their physiology (body fat %, visceral fat, muscle mass distribution, basal metabolic rate, segmental water balance). Wearables like Fitbit and Amazfit log workouts and steps. Yet between the scan and the wearable lies a gap: **nobody turns this data into a personalized, condition-aware, calendar-scheduled plan that the user actually follows.** Existing apps either give generic meal plans, ignore medical conditions, or require an expensive human coach.

For users with diabetes, hypertension, PCOS, or thyroid conditions, generic plans are not just unhelpful — they can be unsafe.

### 1.3 Proposed solution

A multi-agent system that:
1. Ingests the user's body composition report (PDF or photo) and self-reported medical conditions.
2. Generates a personalized weekly plan covering workouts, meals, cheat days, and recipes — grounded in clinical guidelines.
3. Schedules the plan on the user's Google Calendar.
4. Tracks adherence via Google Fit / Health Connect and user-logged meals.
5. Proactively nudges when momentum drops.
6. Logs every agent decision for observability and safety review.

### 1.4 Target user

Primary persona: **adults aged 25–45 with a managed chronic condition (T2 diabetes, hypertension, PCOS, hypothyroidism) who own or have access to a body composition machine and a fitness wearable.** Tech-comfortable, goal-driven, frustrated with one-size-fits-all apps.

Secondary persona: general wellness users without a chronic condition who want structured, evidence-grounded plans rather than influencer-driven advice.

### 1.5 Non-goals (explicitly out of scope for hackathon v1)

- Direct hardware integration with InBody/Boditrix/Evolt machines (no public APIs).
- Direct integration with Fitbit/Amazfit/Huawei wearables (users sync through Google Fit / Health Connect).
- Image-based nutrition tracking (photo of plate → macros). Listed as future work.
- Nearby grocery / food store recommendation. Listed as future work.
- Clinical diagnosis or medication advice. The agent is a coach, not a doctor.
- Native mobile app. Web only for the demo.

---

## 2. Hackathon alignment

### 2.1 Judging criteria mapping

| Criterion | How this project earns it |
|---|---|
| **Technological Implementation** | Multi-agent orchestration via Google Cloud Agent Builder / ADK; three partner MCP integrations each with a distinct role; hybrid retrieval (Elastic) + vector search on user data (MongoDB Atlas); observability instrumentation (Arize) over every agent call. |
| **Design** | Clean web dashboard; a single "hero moment" (PDF upload → calendar populated in under 60 seconds); progressive disclosure of complexity; explicit visibility into *why* the agent made each recommendation. |
| **Potential Impact** | Addresses a real adherence gap for chronic-condition populations. The same architecture extends to mental health, post-op recovery, and pediatric care. |
| **Quality of the Idea** | Re-frames AI from "chat about your health" to "execute a plan on your behalf, with oversight." The Arize layer makes the safety story credible — most submissions will skip this. |

### 2.2 Partner MCP role assignment

Three partner integrations, each with a *distinct* job. No overlap. This division of labor is itself part of the technical story.

| Partner | Role | Why this partner specifically |
|---|---|---|
| **MongoDB Atlas** | User memory layer. Stores profile, health conditions, daily logs, body composition history, adherence data. Atlas Vector Search powers *personal* similarity — "find meals this user has logged and enjoyed that fit today's macros." | Private, mutable, per-user data with vector embeddings on personal preferences. Document model fits health records naturally. |
| **Elastic** | Knowledge corpus + retrieval layer. Indexes a curated corpus of recipes, exercises, condition-specific clinical guidelines, and food nutrition facts. Hybrid (BM25 + dense vector) retrieval grounds the planner agent's recommendations in evidence. | Hybrid search and relevance tuning are Elastic's core strength. Read-mostly knowledge corpus is the canonical Elastic use case. |
| **Arize** | Observability and evaluation over the agent system. Traces every agent invocation, evaluates planner outputs against condition-aware safety rules (e.g., "did this plan suggest >50g simple carbs to a T2 diabetic?"), surfaces failure dashboards. | Arize is built for LLM/agent observability. Health is the right domain to need this — trustworthiness is a feature, not a postscript. |

### 2.3 Prize-track strategy

Three eligible tracks ($5K / $3K / $2K per partner). Primary track: **MongoDB** — the integration is the deepest and most central. Secondary: **Elastic** — strong story on grounding and RAG quality. Tertiary: **Arize** — differentiator most teams will skip.

---

## 3. Functional requirements

### 3.1 User onboarding (FR-1)

| ID | Requirement | Priority |
|---|---|---|
| FR-1.1 | User can create an account via Google OAuth. | Must |
| FR-1.2 | User can upload a body composition report as PDF or image (JPG/PNG, up to 10MB). | Must |
| FR-1.3 | System parses the report using Gemini multimodal and extracts: body fat %, muscle mass, visceral fat rating, BMR, weight, height, body water, segmental analysis if present. | Must |
| FR-1.4 | User reviews and confirms the extracted values, with the ability to correct any field. | Must |
| FR-1.5 | User selects medical conditions from a curated list (T2 diabetes, T1 diabetes, hypertension, PCOS, hypothyroidism, hyperthyroidism, high cholesterol, none). Multi-select. | Must |
| FR-1.6 | User selects a primary goal (fat loss, muscle gain, recomposition, maintenance, condition management). | Must |
| FR-1.7 | User specifies dietary preferences (vegetarian, vegan, non-veg, halal, kosher, no preference) and allergies (free text + common picklist). | Must |
| FR-1.8 | User specifies workout context (home / gym / both), available time per day, weekly availability. | Must |
| FR-1.9 | Onboarding data is persisted to MongoDB as the user's profile document. | Must |
| FR-1.10 | User can manually enter body composition values without uploading a report. | Should |

### 3.2 Plan generation (FR-2)

| ID | Requirement | Priority |
|---|---|---|
| FR-2.1 | On first profile completion, the planner agent generates a 7-day plan covering meals (breakfast, lunch, dinner, two snacks), workouts, and one designated cheat meal slot. | Must |
| FR-2.2 | The planner agent retrieves condition-appropriate guidelines from Elastic before generating any recommendation. | Must |
| FR-2.3 | The planner agent retrieves the user's past meal preferences from MongoDB Atlas Vector Search to personalize suggestions. | Must |
| FR-2.4 | Each meal in the plan includes: name, ingredients, macros (kcal, protein, carbs, fat), prep time, and a link to a full recipe. | Must |
| FR-2.5 | Each workout in the plan includes: name, target muscle groups, duration, exercises with sets/reps, equipment needed. | Must |
| FR-2.6 | Total daily kcal is set within ±10% of the calculated target based on BMR, goal, and activity level. | Must |
| FR-2.7 | The plan is written to MongoDB and to the user's Google Calendar as discrete events. | Must |
| FR-2.8 | User can request a regenerate of any single day or single meal/workout. | Should |
| FR-2.9 | User can mark items as "don't suggest again" and the preference is stored in MongoDB. | Should |
| FR-2.10 | Every plan generation logs trace data to Arize, including retrieved sources, prompt, and output. | Must |

### 3.3 Daily tracking (FR-3)

| ID | Requirement | Priority |
|---|---|---|
| FR-3.1 | System pulls workout and step data from Google Fit / Health Connect at least once per day. | Must |
| FR-3.2 | User can log meals against the planned items with one tap ("ate as planned") or with substitutions. | Must |
| FR-3.3 | User can log free-form meals (typed name + macro estimate) when off-plan. | Must |
| FR-3.4 | Daily adherence score is computed and stored: % of planned workouts completed, % of planned meals logged, kcal variance from target. | Must |
| FR-3.5 | Body weight can be logged daily; trend is shown on dashboard. | Should |

### 3.4 Nudges (FR-4)

| ID | Requirement | Priority |
|---|---|---|
| FR-4.1 | A nudge agent runs on schedule (e.g., 7 PM local) and reviews the day's adherence. | Must |
| FR-4.2 | If adherence is below threshold (configurable, default 60%), the nudge agent generates a personalized, supportive message via Gemini. | Must |
| FR-4.3 | Nudges are delivered in-app (notification badge) and via email. Push notifications are out of scope for web v1. | Must |
| FR-4.4 | Nudges reference specific gaps ("you missed your strength session today") rather than generic encouragement. | Must |
| FR-4.5 | User can configure nudge frequency, channel, and quiet hours. | Should |
| FR-4.6 | Nudge generation is traced in Arize, with evaluation for tone (must remain supportive, never shame-based). | Must |

### 3.5 Dashboard (FR-5)

| ID | Requirement | Priority |
|---|---|---|
| FR-5.1 | Today view shows: today's planned meals and workouts, adherence so far, next scheduled item. | Must |
| FR-5.2 | Week view shows: full 7-day plan grid, completion ticks per item. | Must |
| FR-5.3 | Progress view shows: weight trend, adherence trend, body composition delta since onboarding. | Must |
| FR-5.4 | "Why this recommendation?" panel exposes the Elastic retrieval sources used by the planner. | Should |
| FR-5.5 | Settings view allows update of conditions, goals, preferences, nudge config. | Must |

### 3.6 Plan revision (FR-6)

| ID | Requirement | Priority |
|---|---|---|
| FR-6.1 | A revision agent runs weekly to assess the past week's adherence and outcomes. | Should |
| FR-6.2 | If adherence is consistently below threshold, the revision agent proposes a modified plan (less aggressive, more flexible). | Should |
| FR-6.3 | User must approve plan revisions before they take effect — the agent never silently changes the plan. | Must |

---

## 4. Non-functional requirements

### 4.1 Performance

- PDF/image parsing returns within 15 seconds for a single body comp report.
- Full week plan generation returns within 45 seconds end-to-end.
- Dashboard initial load under 2 seconds on a typical connection.

### 4.2 Reliability

- All agent calls are idempotent at the request level. Retry on transient failure with exponential backoff.
- Plan generation is atomic — partial writes to MongoDB or Google Calendar must be rolled back on failure.

### 4.3 Security and privacy

- All user data encrypted at rest (MongoDB Atlas default) and in transit (TLS).
- Health data is *not* shared with third parties beyond the partner services strictly required for the function.
- User can export and delete all their data on request.
- No PII appears in Arize traces — health data is hashed or redacted before logging.
- The system displays a disclaimer that it is not a substitute for medical advice. Required before plan generation.

### 4.4 Safety

- Every plan is evaluated by an internal safety check before being shown to the user:
  - Carb load per meal vs. diabetic threshold.
  - Sodium per meal vs. hypertensive threshold.
  - Total deficit/surplus within physiologically safe range (e.g., never exceed 1000 kcal deficit/day).
- Failed safety checks trigger regeneration with stricter constraints, logged to Arize.

### 4.5 Cost (hackathon constraint)

- Operating cost under $50 per week during development. Track Gemini token usage, MongoDB Atlas tier, Elastic deployment, Arize plan.

---

## 5. System architecture

### 5.1 Multi-agent design

Four agents, each with narrow responsibility. Coordinated through Google Cloud Agent Builder (ADK).

```
                    ┌─────────────────┐
                    │  Orchestrator    │
                    │  (router agent)  │
                    └────────┬─────────┘
                             │
        ┌────────────┬───────┼───────┬──────────────┐
        ▼            ▼               ▼              ▼
  ┌──────────┐  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ Intake   │  │ Planner  │   │ Tracker  │   │ Nudge    │
  │ agent    │  │ agent    │   │ agent    │   │ agent    │
  └──────────┘  └──────────┘   └──────────┘   └──────────┘
```

- **Intake agent.** Parses body comp reports (Gemini multimodal), extracts structured fields, writes to MongoDB.
- **Planner agent.** Generates and revises weekly plans. Calls Elastic MCP (knowledge retrieval), MongoDB MCP (user history, vector search), Google Calendar tool (write events).
- **Tracker agent.** Pulls Google Fit data on schedule, computes adherence, writes to MongoDB.
- **Nudge agent.** Runs on schedule, reads adherence, generates and delivers messages.

The orchestrator routes user requests to the right agent and handles handoffs.

### 5.2 Component diagram

```
┌────────────────────────────────────────────────────┐
│                 Web app (React)                    │
│  onboarding · dashboard · plan view · settings    │
└─────────────────────┬──────────────────────────────┘
                      │ HTTPS / REST
                      ▼
┌────────────────────────────────────────────────────┐
│           Backend API (Node or Python)             │
│         FastAPI / Express + auth middleware        │
└─────┬──────────────┬──────────────┬────────────────┘
      │              │              │
      ▼              ▼              ▼
┌───────────┐  ┌──────────────┐  ┌─────────────────┐
│  Agent    │  │  Google APIs │  │  Scheduled jobs │
│  runtime  │  │  Calendar,   │  │  (Cloud Run     │
│  (ADK)    │  │  Fit, OAuth  │  │   Jobs / cron)  │
└─────┬─────┘  └──────────────┘  └─────────────────┘
      │
      ├──── Gemini 2.x (Vertex AI)
      ├──── MongoDB Atlas MCP (user data + vector search)
      ├──── Elastic MCP (knowledge corpus retrieval)
      └──── Arize (tracing + evaluation)
```

### 5.3 Data model (MongoDB collections)

- `users` — profile, conditions, goals, preferences, auth.
- `body_comp_reports` — historical scans with timestamps.
- `plans` — generated weekly plans, version history.
- `daily_logs` — meal logs, workout completions, weight, adherence score.
- `meal_embeddings` — per-user vector embeddings of meals the user has rated, for vector similarity.
- `nudges` — sent nudges with reasoning and user response.

### 5.4 Knowledge corpus (Elastic indices)

- `recipes` — curated recipe dataset with macros, condition tags, dietary tags.
- `exercises` — exercise library with muscle groups, difficulty, equipment.
- `guidelines` — clinical guidelines for diabetes, hypertension, PCOS, etc. (from open sources: ADA, AHA, NHS).
- `foods` — food nutrition facts (e.g., USDA FoodData Central subset).

Each index uses hybrid search: BM25 + dense vector (embeddings via Vertex AI text-embedding model).

---

## 6. Build plan and milestones

Working backwards from **June 11, 2026** submission deadline. Today is May 23.

| Week | Dates | Goals |
|---|---|---|
| W1 | May 23–29 | Lock requirements, set up GCP project, MongoDB Atlas cluster, Elastic deployment, Arize account. Stub Agent Builder skeleton. Seed Elastic with initial knowledge corpus (recipes + exercises + 1 condition's guidelines). |
| W2 | May 30 – Jun 5 | Build intake agent (PDF/image parsing). Build planner agent end-to-end against MongoDB + Elastic. Web app onboarding flow. |
| W3 | Jun 6–11 | Tracker + nudge agents. Dashboard. Google Calendar write integration. Arize traces wired into every agent. Demo polish, video, Devpost write-up. |

This is tight — three weeks. If anything slips, the cut order is: revision agent (FR-6), then nudge channel diversity (email-only), then plan regeneration UX (regenerate full week only, not per-item).

### 6.1 Demo hero moment

In the 3-minute demo, the moment to nail: **user uploads an InBody PDF → 45 seconds later their week of meals and workouts is visible on their Google Calendar, with a sidebar showing the Elastic sources used to ground each recommendation.** Everything else in the demo serves this moment.

---

## 7. Risks and open questions

### 7.1 Risks

| Risk | Mitigation |
|---|---|
| Gemini multimodal extraction fails on uncommon report layouts | Manual entry fallback (FR-1.10); test against 5+ real InBody/Boditrix/Evolt sample PDFs in W1. |
| Elastic knowledge corpus quality is too thin to ground plans well | Pre-curate a focused corpus (3 conditions × 50 recipes × 30 exercises) rather than going broad. |
| Google Fit / Health Connect API friction in OAuth setup | Have a fallback "manual log" path so the demo doesn't depend on a third-party token at runtime. |
| Multi-agent coordination introduces latency that breaks the hero moment | Pre-warm the planner on the dashboard transition; show progress UI; cache where safe. |
| Arize integration becomes a sink for time without visible payoff | Time-box Arize wiring to 1.5 days in W3; treat as MVP "every call traced + one safety eval running." |
| Three partners is one too many for the build window | Drop priority order if needed: keep MongoDB and Elastic, cut Arize to "wired but minimal." |

### 7.2 Open questions

1. **Architecture detail:** ADK in Python or Node? Python has better Gemini/Vertex SDK maturity; Node aligns with a React frontend monorepo. Decision needed by end of W1.
2. **Knowledge corpus sourcing:** which open licenses can we use for recipes? USDA FoodData is fine; recipe sources need checking. May need to write synthetic recipes against a known-good macro structure.
3. **Hosting:** Cloud Run for the backend? Firebase Hosting for the web app? Decision needed by end of W1.
4. **Auth scope:** Google OAuth — what scopes do we request? Minimum is Calendar.write + Fit.read. Avoid scope creep.
5. **Safety rules:** who authors the condition-specific safety rules used in the eval? Need a clear source (e.g., "carb threshold per meal per ADA guidelines"). Citable sources only.
6. **Demo data:** do we use a real seeded test account for the demo or generate fresh on stage? Real account is safer; generate-fresh is more impressive.

---

## 8. Success criteria

Hackathon success is defined as:

1. A working hosted demo URL where a judge can complete the onboarding flow with a sample PDF and see a generated week of plans on a calendar within 60 seconds.
2. All three partner MCPs (MongoDB, Elastic, Arize) demonstrably wired — visible in the demo or in the repo with clear evidence of use.
3. A 3-minute video that lands the hero moment in the first 45 seconds.
4. A public repo with an open-source license at the top level (likely MIT or Apache 2.0), a clean README, and architecture diagrams matching this document.
5. Devpost write-up that maps each judging criterion to specific implementation details.

---

## 9. Next documents to produce

After this requirements doc is approved:

1. **Architecture spec** — agent prompt designs, tool schemas, MCP wiring details.
2. **Build schedule** — day-by-day task list for the 3-week window, with owner assignments if a team forms.
3. **Demo script** — minute-by-minute breakdown of the 3-minute video.
4. **Devpost submission narrative** — judging-criterion-aligned write-up.
