Cadence: Build Guide (20th → 25th July)

Day 1- Today, 20th: Get the agent talking
Goal by end of day: you can have a live voice conversation with your agent in the browser, and it can look up a fake order.
Sign up / log into the OneInbox dashboard, grab your API key (Settings → API Keys).
Run the Business Logic API locally:
 cd backendpip install fastapi uvicorn slowapiuvicorn business_logic_api:app --reload --port 8000
Confirm it works: curl http://localhost:8000/orders/ORD1001 -H "x-api-key: sk_demo_tenant_acme"
Expose it publicly so OneInbox can reach it:
 npx ngrok http 8000
Copy the https://...ngrok.app URL.
Fill in setup_agent.py with your API key and the ngrok URL, then run it. This creates your agent, wires up both tools, and sets the system prompt.
Test with a raw web call first (no frontend needed yet):
 curl -X POST https://api.oneinbox.ai/v1/calls/web \  -H "Authorization: Bearer <api_key>" \  -d '{"agent_id": "<agent_id>"}'
This confirms the agent + tool wiring works before you touch frontend code.
End-of-day check: agent responds, and asking "what's the status of order ORD1001" triggers a real lookup through your API.

Day 2- 21st: Frontend + interruption tuning
Goal: a clickable demo page, and barge-in behavior that feels right.
Set up the frontend:
 cd frontendnpm install @oneinbox/web-sdk-react @oneinbox/web-sdk livekit-client react


Get a publishable key (Dashboard → Settings → Publishable Keys), register your dev origin (e.g. http://localhost:3000).
Drop in CallWidget.tsx, fill in AGENT_ID and PUBLISHABLE_KEY.
Talk to it. Deliberately interrupt it mid-sentence, several times, in different ways (cutting in immediately vs. after a pause).
Tune interruption_sensitivity on the agent (PATCH /v1/agents/<id>)- start at 0.7, adjust based on how it actually feels. Write down what value you land on and why- this becomes a real, defensible line in your Solution Overview instead of a guess.
Confirm silence_timeout_seconds behavior (Flow C)- go quiet mid-sentence, see what happens.
End-of-day check: you have a working browser demo, and you've personally felt the barge-in behavior enough to describe it confidently to judges.

Day 3- 22nd: Observability
Goal: your PRD's KPIs are actually being measured, not just described.
Run the Observability API:
 cd backenduvicorn observability_api:app --reload --port 8001
Expose it with a second ngrok tunnel, update your OneInbox webhook URL (or re-run the webhook step of setup_agent.py with the new URL).
Wire up the barge-in tracking in CallWidget.tsx- this is the fiddliest part; read the note at the bottom of that file about speech-start event wiring, since it may need the lower-level SDK client rather than the hook.
Make several test calls, deliberately interrupting some, staying quiet on others, completing the "task" (getting an order status) on some.
Hit GET /metrics on your Observability API and confirm real numbers come back- barge-in latency, false-interruption rate, task completion rate.
End-of-day check: /metrics returns real numbers from real calls, not placeholders.

Day 4- 23rd: Business logic depth + polish
Goal: the agent feels like a real support agent, not a demo shell.
Add 3-5 more fake orders and troubleshooting topics to the seed data in business_logic_api.py so the demo doesn't look thin.
Consider adding a transfer_call or end_call tool (from the Tools guide) so the agent can escalate or hang up cleanly- this directly demonstrates a flow from your PRD.
If you have slack: add the stretch-goal security polish- the API key check is already in place; consider adding a second demo tenant to show the isolation actually works (tenant A can't see tenant B's orders).
Sanity-check every flow from your PRD one more time: normal turn-taking, interruption, silence, and- if using phone calls instead of/alongside web- a dropped-call scenario.
End-of-day check: you'd be comfortable demoing this to a stranger without narrating around rough edges.

Day 5- 24th: Freeze
Goal: nothing new. Everything that exists, works reliably.
Stop building features.
Run the full demo start-to-finish, at least 3 times, in the actual environment/browser you'll present from.
Have a fallback: record a video of a clean run, in case live audio hiccups during the actual presentation.
Update your PRD/Solution Overview one last time to match what was actually built (not what was planned)- judges notice when the doc and the demo disagree.

Day 6- 25th: Present
Final polish only- nothing structural.
Rehearse the narrative: problem → what OneInbox handles vs. what you built → live demo → metrics from /metrics as evidence, not just claims.
Lead the demo with an actual interruption- it's the single most memorable 5 seconds of the whole presentation.

