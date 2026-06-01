# Day-by-Day Build Schedule

**Track:** B+ (MongoDB + Arize Phoenix, ~65 hrs)
**Stack:** Python 3.11+, FastAPI, Google ADK, Gemini 2.5, React+Vite+Tailwind, MongoDB Atlas, Arize Phoenix Cloud
**Submission:** Thu Jun 11, 2026
**Hero demo:** "Upload InBody PDF → personalized week on Google Calendar in 60 seconds"

---

## Schedule overview

The schedule is organized into four phases, mapped against your holiday/weekday rhythm.

| Phase | Days | Goal | Hours |
|---|---|---|---|
| **Phase 0 — Foundations** | Sat 23 – Tue 26 | Accounts, repo, ADK hello-world, decisions locked | ~8 |
| **Phase 1 — Core agents** | Wed 27 – Sun 31 | Profile agent + Planner agent working end-to-end locally | ~22 |
| **Phase 2 — Integration + UI** | Mon 1 – Sun 7 | Calendar, frontend, dashboard, deploy to Cloud Run | ~25 |
| **Phase 3 — Polish + submit** | Mon 8 – Thu 11 | Demo video, README, submission, buffer | ~10 |

---

## Phase 0 — Foundations (Sat 23 – Tue 26)

### Sat May 23 (today, 1–2 hrs, low energy)

**Goal: locked decisions, repo exists, you don't touch code.**

1. **Lock the hero demo moment** in writing somewhere you'll see it. Put it as the first line of the README.
2. **Create the GitHub repo.** Name it something honest — `health-coach-agent` or `agentic-health-coach` works. MIT license. Add a 3-line README placeholder.
3. **Sign up for Devpost**, register for the hackathon, fill out the basic team info (just you).
4. Done. Rest.

### Sun May 24 (1–2 hrs, low energy)

**Goal: all accounts exist, nothing configured yet.**

1. **Google Cloud account** — claim $300 free credit if eligible. Create a new project named `health-coach-hackathon`.
2. **MongoDB Atlas free tier** — create a free M0 cluster. Save the connection string. Don't model data yet.
3. **Arize Phoenix Cloud account** at `app.phoenix.arize.com`. Save your API key and space ID.
4. **Verify Phoenix qualifies for the Arize partner track.** Check the hackathon Discord or partner page. If unclear, post a question and move on — fallback is Arize AX free trial.
5. **Resolve one open question:** decide Fitness REST API (recommended for web app). Note the deprecation in the README's risk section.

### Mon May 25 (weekday, 1–2 hrs)

**Goal: dev environment running, ADK hello-world working locally.**

1. Local Python 3.11+ environment with `uv` (the fast package manager Google's blog uses).
2. `pip install google-adk google-genai fastapi uvicorn pymongo arize-phoenix openinference-instrumentation-google-adk`
3. Write the smallest possible ADK agent that responds to "hello" — just to verify the toolchain works. Use the example from the ADK GitHub README.
4. Run it with `adk web` to confirm the local UI works.
5. Commit. You now have a working baseline.

### Tue May 26 (weekday, 1–2 hrs)

**Goal: Gemini API + MongoDB connection both verified, Phoenix tracing the hello-world agent.**

1. Get a Gemini API key (via AI Studio is fastest for hackathon use).
2. Connect to your MongoDB Atlas cluster from a quick Python script — insert one document, read it back. Confirm the connection string and IP allowlist work.
3. Wire Phoenix tracing into the hello-world agent (3 lines of setup from the Phoenix ADK integration docs). Send a message, see the trace appear in Phoenix Cloud.
4. **Checkpoint:** all four pieces (ADK, Gemini, MongoDB, Phoenix) verified working in isolation. This is the most important checkpoint in the whole schedule — if anything here is broken, fix it before Wednesday.

---

## Phase 1 — Core agents (Wed 27 – Sun 31)

### Wed May 27 (holiday, 3–5 hrs)

**Goal: Profile agent skeleton + PDF parsing prototype.**

1. Design the MongoDB schema for the user profile (use the data model in the requirements doc).
2. Write a `User` and `Profile` pydantic model in Python.
3. **Profile agent v0:** an ADK agent with one tool — `extract_scan_from_pdf(file_path)` — that calls Gemini 2.5 Pro with the PDF and structured output to return body composition fields.
4. **Test with a real InBody report.** If you don't have one, search for "InBody result sample PDF" online — many gyms publish samples.
5. **Critical:** verify Gemini reliably extracts at least body fat %, skeletal muscle mass, weight, BMR. If extraction is shaky on test PDFs, add an explicit JSON schema in the prompt and a confidence field.

### Thu May 28 (weekday, 1–2 hrs)

**Goal: Profile agent end-to-end locally.**

1. Add `save_profile(user_id, profile_data)` tool wired to MongoDB.
2. Profile agent now: accepts PDF → extracts → asks confirming questions about conditions/goals/preferences → saves complete profile.
3. Test the full flow in `adk web`. Capture the Phoenix trace as a screenshot — you'll want this for the demo.

### Fri May 29 (weekday, 1–2 hrs)

**Goal: small but real knowledge corpus loaded into MongoDB.**

1. Hand-curate **30 recipes** as JSON: name, ingredients, macros (cal/protein/carbs/fat), prep time, dietary tags, condition tags (diabetic-friendly, low-sodium, etc.). Use ChatGPT/Claude to generate the first draft, then review and prune. Don't agonize over quality — these are demo content.
2. Hand-curate **20 exercises** the same way: name, muscle groups, equipment, difficulty, contraindications.
3. Hand-curate **5 clinical guideline summaries** for diabetes and hypertension. Paraphrase from public ADA/AHA sources, cite the source in each document. Don't copy verbatim.
4. Load all three collections into MongoDB. Verify queries work.

### Sat May 30 (holiday, 3–5 hrs)

**Goal: Planner agent generating real plans.**

1. **Planner agent v0:** an ADK agent with tools `get_user_profile`, `query_recipes(filters)`, `query_exercises(filters)`, `query_guidelines(condition)`, all wired to MongoDB.
2. Planner agent prompt: takes a profile, retrieves relevant content from MongoDB, asks Gemini 2.5 Flash to generate a 7-day plan as structured JSON.
3. Test with the test profile from Wednesday. Inspect the output JSON for: condition-awareness (no high-glycemic for diabetic), macro reasonableness, variety across the week.
4. Iterate the prompt until the plans look good. **This is where you'll spend the most prompt-engineering time — budget for it.**

### Sun May 31 (holiday, 3–5 hrs)

**Goal: MongoDB Atlas Vector Search for personalized similarity + agent orchestration.**

1. In MongoDB Atlas, create a vector search index on the recipes collection. Use Google's `text-embedding-004` to embed recipe names + descriptions. Backfill embeddings for your 30 recipes.
2. Add a `find_similar_recipes(reference_recipe, user_constraints)` tool to the Planner agent. **This is your MongoDB hero feature — the moment that justifies "why MongoDB."**
3. **Orchestrator agent:** a parent ADK agent that delegates to Profile or Planner based on the request. Use ADK's hierarchical sub-agent pattern.
4. End-of-Phase-1 checkpoint: from the ADK web UI, you can say "I just uploaded my scan" → profile created → "now plan my week" → plan generated. All traced in Phoenix.

---

## Phase 2 — Integration + UI (Mon 1 – Sun 7)

### Mon Jun 1 (holiday, 3–5 hrs)

**Goal: Google Calendar write working.**

1. Set up Google OAuth in your GCP project. Configure consent screen, get client ID/secret. Add Calendar and Fit scopes (Fit scope even though we may not implement Fit — having the scope saves a re-consent later if you add it).
2. Implement OAuth flow in FastAPI: `/auth/login` → Google → callback → store refresh token in MongoDB.
3. Add `write_plan_to_calendar(plan, user_id)` tool to the Planner agent. Creates one event per workout and per meal, with the plan rationale in the description.
4. Test end-to-end from `adk web`: plan generates, events appear in your real Google Calendar.

### Tue Jun 2 (holiday, 3–5 hrs)

**Goal: FastAPI backend wrapping the agents.**

1. Wrap the ADK orchestrator agent in a FastAPI service. Endpoints: `/auth/*`, `/profile/upload`, `/profile/confirm`, `/plan/generate`, `/plan/current`.
2. Each endpoint invokes the appropriate agent and returns structured JSON.
3. Test all endpoints with curl or Postman.
4. Commit. Backend is feature-complete for the demo.

### Wed Jun 3 (weekday, 1–2 hrs)

**Goal: frontend scaffolding.**

1. Create React+Vite+Tailwind app in a `frontend/` directory.
2. Set up routing (React Router): `/`, `/onboarding`, `/dashboard`.
3. Build the landing page with Google Sign-In button only. Don't style yet — wire the auth flow.

### Thu Jun 4 (weekday, 1–2 hrs)

**Goal: onboarding flow UI.**

1. Onboarding screen: file upload area, then a confirmation form for the extracted values, then a short form for conditions/goals/preferences.
2. Submit hits `/profile/upload` and `/profile/confirm`.
3. Don't worry about styling — just get the flow working.

### Fri Jun 5 (weekday, 1–2 hrs)

**Goal: dashboard UI.**

1. Dashboard screen: today's plan card (workout + meals with rationale), "regenerate this week" button.
2. Fetches from `/plan/current`.
3. Style with Tailwind — clean, single-column, generous whitespace. Reference the [frontend-design skill] when you start — don't ship default-AI-aesthetic UI.

### Sat Jun 6 (weekend, 3–5 hrs)

**Goal: deployment to Cloud Run, frontend hosting, end-to-end working in the cloud.**

1. Dockerize the FastAPI backend.
2. Deploy to Cloud Run. Configure env vars for Mongo URI, Gemini API key, Phoenix credentials, Google OAuth.
3. Deploy frontend to Firebase Hosting (or Vercel — whatever's faster).
4. Update Google OAuth redirect URIs to include the deployed frontend URL.
5. **Run the full hero flow end-to-end in production.** Upload a PDF, confirm, generate plan, see it on calendar. Fix whatever breaks.

### Sun Jun 7 (weekend, 3–5 hrs)

**Goal: Phoenix evals, polish, buffer for whatever's broken.**

1. Define **two Phoenix evals** for the Planner output: (a) "does the plan respect the user's listed medical conditions?" — boolean LLM-as-judge; (b) "is the macro split appropriate for the user's goal?" — boolean LLM-as-judge.
2. Run the evals against the last 5 plan generations. Capture screenshots for the demo.
3. Polish the dashboard UI — typography, spacing, color, the rationale display. This is where the project either looks polished or looks like a hackathon submission.
4. Test the demo flow start-to-finish three times. Time it. If it's over 90 seconds, find what's slow.

---

## Phase 3 — Polish + submit (Mon 8 – Thu 11)

### Mon Jun 8 (weekday, 1–2 hrs)

**Goal: demo video script written, README drafted.**

1. **Write the 3-minute demo script** as a tight narration. Beat-by-beat:
   - 0:00–0:15: problem (10 seconds) + solution one-liner (5 seconds)
   - 0:15–0:45: upload + extract + confirm
   - 0:45–1:30: plan generates with rationale visible
   - 1:30–1:45: calendar event shown on a phone or web view
   - 1:45–2:15: Phoenix trace shown — "here's how the agent reasoned, with safety evals passing"
   - 2:15–2:45: dashboard with regenerate
   - 2:45–3:00: closing — partner tracks, repo, hosted URL
2. Draft the **README** with these sections: hero demo (with screenshot/GIF), problem, solution, architecture diagram, partner integrations with "why this partner" rationale per partner, setup instructions, demo video link.

### Tue Jun 9 (weekday, 1–2 hrs)

**Goal: record demo video.**

1. Record 2-3 takes of the 3-minute demo. Use Loom, OBS, or screen recording software you trust.
2. Pick the best take. Light edit only — trim dead air, add a title card.
3. Upload to YouTube as unlisted. Get the link.

### Wed Jun 10 (weekday, 1–2 hrs)

**Goal: Devpost submission filled out, every detail final.**

1. Fill out the Devpost form:
   - Title
   - Tagline (the one-liner)
   - Description (lift from README)
   - Inspiration, what it does, how we built it, challenges, accomplishments, what we learned (Devpost asks all of these — answer briefly and honestly)
   - Built with (tags: Python, ADK, Gemini, MongoDB, Phoenix, Cloud Run, React)
   - Try it out links (hosted URL + GitHub)
   - Demo video URL
   - **Select partner prize tracks: MongoDB + Arize**
2. Submit. **Do not wait until Thu Jun 11.**

### Thu Jun 11 (submission day, buffer only)

**Goal: nothing breaks. Sleep.**

1. Verify the hosted URL is up.
2. Verify the demo video loads from the Devpost page.
3. Verify the GitHub repo is public.
4. If anything is broken: hours 0–4 are your buffer. After noon, stop.
5. Submit any final updates to Devpost before the deadline closes.

---

## Buffer and risk management

**Built-in buffer:** Sun Jun 7 has slack baked in, and Phase 3 is intentionally light. You have roughly 6–8 hours of unallocated buffer.

**If you fall behind:** drop these in this order — (1) the second Phoenix eval, (2) the dashboard's trend display, (3) the "regenerate with feedback" feature, (4) the macro display on each meal. Each cut keeps the hero demo intact.

**If you fall ahead:** in priority order — (1) polish the rationale display so it's the most beautiful thing on the page, (2) add a Phoenix safety-eval pass-rate badge to the dashboard ("plan passed 12/12 checks"), (3) start on Google Fit integration as a stretch goal.

**Single biggest risk:** the OAuth + Calendar flow on Mon Jun 1. If that day blows up, your demo's hero moment is at risk. Front-load any Google OAuth experimentation — even a 30-minute spike on Tue May 26 to make sure you understand the consent flow would be insurance.

---

## What today (Sat May 23) looks like specifically

You said you're up for working. Here's the today-only checklist:

1. **Right now:** read this schedule once, top to bottom.
2. **Hour 1:** create the GitHub repo with MIT license. Add a `README.md` with just the hero demo one-liner at the top. Commit, push.
3. **Hour 2:** register on Devpost for the Rapid Agent Hackathon. Sign up for Google Cloud, MongoDB Atlas, Phoenix Cloud (don't configure them, just create accounts and save credentials in a `.env.local.notes` file — gitignored).
4. **Stop.** If you have energy left, read the [ADK GitHub README](https://github.com/google/adk-python) and the [Phoenix ADK integration page](https://arize.com/docs/phoenix/integrations/frameworks-and-platforms/google-adk). No coding today.

The point of today is that tomorrow you wake up with everything provisioned and ready, and Monday morning you start coding without administrative friction.

Want me to draft the GitHub README first commit content, or the architecture diagram for the README? Either is a good 30-minute task for tonight if you have the energy.
