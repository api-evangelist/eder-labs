---
name: Get structured insights from a user's memory
description: >-
  Ask Persona a question over a user's memory and get back a structured JSON
  result, optionally shaped by your own output_schema.
api: openapi/eder-labs-persona-openapi.yml
operations: [create_user, ingest_data, ask_insights]
---

# Get structured insights from a user's memory

Use the `ask` endpoint when you want machine-readable fields rather than a prose
answer — e.g. extracting a user's preferences into a fixed shape.

## Steps

1. **Ensure the user and memory exist.** If needed, `create_user`
   (`POST /users/{user_id}`) and `ingest_data`
   (`POST /users/{user_id}/ingest`) first — `ask_insights` over an empty user
   returns little of value, and a missing user returns `404`.

2. **Ask for structured insights.** `ask_insights` —
   `POST /users/{user_id}/ask`. Body: `{ "query": "What are my preferences?",
   "output_schema": { "preferences": ["example"], "summary": "string" } }`.
   The `output_schema` is optional; when provided, Persona shapes the result to
   match it. A `200` returns `{ "result": { ... } }`.

## Error handling

FastAPI `{ "detail": "<message>" }` envelope:
- `400` — missing body or empty query.
- `404` — user not found; create/ingest first.
- `502` — LLM provider error; retry with backoff.
- `503` — Neo4j unavailable; retry later.
