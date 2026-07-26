---
name: Ingest content and recall it with a RAG query
description: >-
  Create a Persona user (memory namespace), ingest one or more pieces of content
  into their Graph-Vector memory, then ask a natural-language RAG question and get
  an answer grounded in that memory.
api: openapi/eder-labs-persona-openapi.yml
operations: [create_user, ingest_data, ingest_batch_data, rag_query]
---

# Ingest content and recall it with a RAG query

Persona is a self-hosted memory API. Base URL is your deployment's `/api/v1`
(local default `http://localhost:8000/api/v1`). There is no built-in auth — the
service is expected to run behind your own gateway, so send requests to your
protected host.

## Steps

1. **Create the user (memory namespace).** `create_user` — `POST /users/{user_id}`.
   `user_id` must match `^[a-zA-Z0-9_-]+$`. A `201` means newly created; `200`
   means it already existed (both are fine to proceed). A `422` means the id
   format is invalid — fix the id and retry.

2. **Ingest content.** For a single item use `ingest_data`
   (`POST /users/{user_id}/ingest`) with a JSON body `{ "content": "...",
   "source_type": "conversation" }`. For several items in one call use
   `ingest_batch_data` (`POST /users/{user_id}/ingest/batch`) with
   `{ "items": [ { "content": "...", "source_type": "notes" }, ... ] }`.
   A `201` returns `memories_created`. Do not send empty content (→ `400`).

3. **Recall with RAG.** `rag_query` — `POST /users/{user_id}/rag/query` with
   `{ "query": "What projects am I working on?" }`. Keep the query ≤ 1000
   characters (→ `400` otherwise). A `200` returns `{ "answer": "..." }`.

## Error handling

Errors use the FastAPI envelope `{ "detail": "<message>" }`:
- `404` — the user does not exist; create it first.
- `502` — upstream LLM provider error; retry with backoff.
- `503` — Neo4j is unavailable; retry later.

Persona documents no idempotency key, so guard against duplicate ingests on
your side (e.g. dedupe content before calling).
