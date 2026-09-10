cat << 'EOF' > my-integration-note.md
# Integration note: team-2 (v1, go-live)

Copy this file, fill every angle bracket, and hand it to your paired Agentic AI team. It is Part A of the cross-cohort runbook; the full operating rules for the window live there.

- **base_url** (client form, ends in `/v1` - paste into an OpenAI client): `http://localhost:8000/v1`
- **service root** (no `/v1` - the runbook's triage curls and `verify.sh` build paths from this): `http://localhost:8000`
- **model id:** `Qwen/Qwen2.5-1.5B-Instruct-AWQ`
- **auth:** bearer key, handed over in person / DM to their on-call, never in this file
- **modalities:** text in, text out, tool calls per the OpenAI schema. text only
- **example call:** the exact `curl` from your green check, with the key redacted:
  `curl http://localhost:8000/v1/chat/completions -H "Authorization: Bearer REDACTED" -H "Content-Type: application/json" -d '{"model": "Qwen/Qwen2.5-1.5B-Instruct-AWQ", "messages": [{"role": "user", "content": "hello from outside"}]}'`
- **SLOs we publish:** availability 99% over the window • TTFT p95 < 500 ms (tier 1) • error rate < 1%
- **limits, declared honestly:** max_tokens clamp 2048 • concurrency knee ~10
- **on-call:** Shurooq • #d5-team • response within 15 minutes during the window
EOF
