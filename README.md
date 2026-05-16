# Sentinel AI

**The control layer for autonomous sales agents.**

Sentinel researches leads using live web data, generates personalized outreach with an AI agent, logs every step the agent takes, and flags risky language before anything goes out.

> Most teams are building one agent. Sentinel is the system companies need when those agents start acting in the real world. The future is not just autonomous agents — it is **governed autonomy**.

## Live demo

- **Frontend:** https://sentinel-ai.zeabur.app
- **Backend API:** https://sentinel.zeabur.app
- **Demo login:** `demo@sentinel.ai` / `sentinel2026`

Click **Research** on the seeded Lattice lead and watch the agent fetch lattice.com via Bright Data, synthesize a cited fact with OpenAI, and weave that fact into a personalized cold email that goes out via Resend.

## Built for Agent Forge / AI Builders @ Llama Ventures (May 16, 2026)

### Sponsor stack

| Layer | Sponsor / tool |
|---|---|
| Live web fetch | **Bright Data** Web Unlocker |
| LLM (research + drafting) | **OpenAI** (`gpt-4o-mini`) |
| Email delivery | **Resend** |
| Hosting (backend + frontend) | **Zeabur** |

### Minimum winning flow

`Lead → live website fetch → LLM-generated research summary → AgentEvent reasoning trace → personalized draft → governance flag → real email send`

## What's inside

- **`backend/`** — FastAPI app, SQLite, JWT auth, single-file monolith with all endpoints (`/research_lead`, `/generate_researched_draft`, `/send_draft`, `/agent_events`, `/governance_events`, etc.)
- **`backend/agents/`** — `llm.py` (one OpenAI-compatible helper with a DEMO_MODE fail-safe), `research_agent.py` (fetch via Bright Data → LLM → structured research with `cited_fact`)
- **`web/`** — Next.js 16 + Tailwind dashboard with 8 tabs: Dashboard, Leads, Drafts, Activity, Agents, Monitoring, Governance, ROI
- **`web/app/page.tsx`** — Reasoning Trace modal that surfaces every agent step (input, output preview, latency, token usage) and an "Add Real Lead" button to demo any company on demand

## Key design choices

1. **Governed autonomy as the product.** Every agent step is logged to `AgentEvent` so the Monitoring / Governance / ROI tabs render real data from real runs — not mocked screens.
2. **One LLM helper, sponsor-pluggable.** `backend/agents/llm.py` is ~50 lines, OpenAI-compatible (`OPENAI_BASE_URL` + `OPENAI_API_KEY` + `OPENAI_MODEL`). Swap to any sponsor LLM (TokenRouter, Qwen Cloud, Z.ai, etc.) with one env var.
3. **DEMO_MODE fail-safe.** Every external call (Bright Data fetch, LLM, email send) has a prewritten fallback so the stage demo never crashes on a flaky API.
4. **Self-healing demo data.** A startup hook re-creates the demo user + seeds the Lattice + Quick Wins leads on every container restart — Zeabur free-tier SQLite wipes can't kill the demo.
5. **Regex governance, not LLM-as-judge.** Catches "guarantee", "ACT NOW", "limited time", excessive `!!!`, "100%", "no risk", "zero downside", and ALL-CAPS pressure language — fast, deterministic, ships in 30 minutes, visible in the Governance tab.

## Local dev

```bash
# Backend
cd backend
cp .env.example .env   # fill in OPENAI_API_KEY at minimum
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
.venv/bin/python -m uvicorn main:app --reload --port 8000

# Frontend
cd web
npm install
NEXT_PUBLIC_API_URL=http://127.0.0.1:8000 npm run dev
```

Or via Docker:

```bash
docker compose up --build
```

## Environment variables

| Var | Purpose | Required |
|---|---|---|
| `OPENAI_API_KEY` | OpenAI-compatible LLM key | yes |
| `OPENAI_BASE_URL` | LLM endpoint (default `https://api.openai.com/v1`) | no |
| `OPENAI_MODEL` | Model id (default `gpt-4o-mini`) | no |
| `OPENAI_PROVIDER_NAME` | Display name for the "Powered by" badge | no |
| `BRIGHT_DATA_API_KEY` | Bright Data customer token | no (falls back to `httpx`) |
| `BRIGHT_DATA_ZONE` | Bright Data zone name (e.g. `web_unlocker1`) | no |
| `RESEND_API_KEY` | Resend bearer token | no (falls back to SMTP / Gmail) |
| `RESEND_FROM` | From header (default `Sentinel AI <onboarding@resend.dev>`) | no |
| `RESEND_REPLY_TO` | Reply-to override | no |
| `DEMO_MODE` | `true` enables fail-safe fallbacks | no |
| `DEMO_LEAD_EMAIL` | Email used for the seeded demo leads | no |
| `DEMO_AUTOBOOTSTRAP` | `true` auto-creates demo user + leads on startup | no |
| `AUTH_SECRET_KEY` | JWT signing secret | yes (in prod) |
| `ALLOW_REGISTRATION` | `true` enables `/auth/register` | no |
| `CORS_ORIGINS` | Comma-separated extra allowed origins | no |
| `NEXT_PUBLIC_API_URL` | Frontend → backend URL (build-time) | yes for FE |
| `NEXT_PUBLIC_LLM_PROVIDER` | Badge text on the dashboard | no |

## License

MIT.
