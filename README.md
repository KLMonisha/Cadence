# Cadence

An interruption-aware voice AI agent for customer support, built on [OneInbox](https://oneinbox.ai). Cadence handles order-status lookups and troubleshooting over a natural, real-time voice conversation and reacts instantly when a caller talks over it, instead of finishing its sentence first.

Built for the OneInbox AI Internship Hackathon 2026.

## How it works

OneInbox's managed voice platform handles the hard real-time infrastructure — speech-to-text, LLM turn-taking, text-to-speech, and barge-in detection (tuned via `interruption_sensitivity`). Cadence's own code covers everything OneInbox doesn't provide out of the box:

- **`backend/business_logic_api.py`** : FastAPI service for order lookups and troubleshooting steps, called by the agent via a OneInbox `api_call` tool. Includes API-key auth and rate limiting.
- **`backend/observability_api.py`** : receives OneInbox webhooks (`call.started`, `call.ended`, `transcript.final`) and client-reported barge-in events, and exposes `/metrics` with real KPIs: barge-in reaction latency, false-interruption rate, task completion rate.
- **`frontend/CallWidget.tsx`** : browser demo UI using the OneInbox Web SDK, including client-side barge-in latency measurement (OneInbox doesn't emit this as a webhook event, so it's tracked in the browser and reported back).
- **`setup_agent.py`** : one-time script that provisions the agent, both tools, and the webhook via the OneInbox API.

## Setup

```bash
# Backend
cd backend
pip install fastapi uvicorn slowapi requests
uvicorn business_logic_api:app --reload --port 8000
uvicorn observability_api:app --reload --port 8001

# Expose both publicly (OneInbox needs to reach them)
ngrok http 8000
ngrok http 8001

# Provision the agent (fill in API key + ngrok URLs first)
python setup_agent.py

# Frontend
cd frontend
npm install @oneinbox/web-sdk-react @oneinbox/web-sdk livekit-client react
# fill in AGENT_ID and PUBLISHABLE_KEY in CallWidget.tsx, then run your React app
```

## Tech stack

OneInbox (voice pipeline) · FastAPI · SQLite · React · OneInbox Web SDK

## Status

See [`BUILD_GUIDE.md`](./BUILD_GUIDE.md) for the day-by-day build plan and current progress.
