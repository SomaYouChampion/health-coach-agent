# Health Coach Agent

> **Upload your InBody scan. Get a personalized week on your Google Calendar in 60 seconds.**

An agentic health coach that turns your body composition report and medical conditions into a Gemini-planned weekly schedule of workouts, meals, and recipes — grounded in your personal history via MongoDB Atlas Vector Search, and made auditable through Arize Phoenix observability over every agent decision.

Built for the [Google Cloud Rapid Agent Hackathon](https://rapid-agent.devpost.com/) 2026.

---

## The problem

People who get body composition scans at gyms or clinics receive a detailed snapshot of their health — body fat, muscle mass, BMR, segmental analysis — and then almost nothing happens with it. They walk out with a PDF, no follow-through plan, no integration with their daily life, and no accountability. For people managing chronic conditions like type 2 diabetes or hypertension, the gap between *knowing their numbers* and *acting on them* is even more consequential.

Existing fitness apps don't bridge this gap. They don't read those reports. They treat all users identically. They don't respect medical conditions when generating plans. And they're dashboards, not coaches.

## The solution

A multi-agent system built on Google's Agent Development Kit (ADK) that:

1. **Ingests** a body composition report (PDF or image) via Gemini multimodal parsing
2. **Understands** your medical conditions, goals, and dietary preferences
3. **Plans** a full week of workouts, meals, recipes, and cheat days — grounded in a curated clinical knowledge base
4. **Schedules** the plan directly to your Google Calendar
5. **Audits** every agent decision through Arize Phoenix traces and safety evaluations

## Architecture

*(Diagram coming soon)*

**Multi-agent system orchestrated via Google ADK:**

- **Orchestrator agent** — routes user requests to the right specialist agent
- **Profile agent** — parses scans, captures conditions and goals, writes user state to MongoDB
- **Planner agent** — retrieves relevant recipes/exercises/guidelines, generates the weekly plan with Gemini, writes events to Google Calendar

**Why these partners:**

- **MongoDB Atlas** — the user's longitudinal health state lives in a document model (a profile naturally nests conditions, scan history, plan history, daily logs). Atlas Vector Search powers personal meal similarity ("find me meals like the ones I've enjoyed before, that fit today's macros"). This is a genuine vector use case, not a Postgres-substitute use case.
- **Arize Phoenix** — health advice has real consequences, which makes observability not optional. Every agent run is traced. Plan outputs run through safety evaluations (e.g., "does this plan respect the user's listed conditions?") before reaching the user. The trust story is structural, not bolted on.

## Tech stack

- **Backend:** Python 3.11+, FastAPI, Google ADK
- **Models:** Gemini 2.5 Pro (multimodal PDF parsing), Gemini 2.5 Flash (plan generation)
- **Frontend:** React + Vite + Tailwind CSS
- **Data:** MongoDB Atlas (user state + knowledge corpus + vector search)
- **Observability:** Arize Phoenix (ADK auto-instrumentation + MCP tracing)
- **Integrations:** Google OAuth, Google Calendar API, Google Fit (stretch goal)
- **Deployment:** Google Cloud Run + Firebase Hosting

## Status

🚧 Under active development for the hackathon. Build log in `/docs`.

## License

MIT. See [LICENSE](LICENSE).

## Important disclaimer

This is a hackathon project and not a medical product. It does not provide medical advice, diagnosis, or treatment. Always consult a qualified healthcare professional before making changes to diet, exercise, or medication.
