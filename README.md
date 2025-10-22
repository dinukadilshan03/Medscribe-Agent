# MedScribe Agent — Project Overview

MedScribe Agent is an agentic RAG (Retrieval-Augmented Generation) system for extracting, normalizing, indexing, and explaining medical report content to support patient-facing, educational insights. It is designed as a privacy-conscious research/educational tool (not a medical-diagnostic product).

This repository implements a multi-service architecture (React frontend, Express API, Python FastAPI AI services) and a set of agents and pipelines for ingesting clinical documents, extracting structured facts, creating embeddings, and answering grounded user questions with verifiable citations.

---

## Key capabilities

- Document ingestion (PDFs/images) with OCR and structured parsing
- Unit canonicalization and reference-range normalization for lab values
- Status classification (normal / borderline / abnormal)
- Structured facts storage (timeseries lab results and report sections)
- Per-user vector namespaces and semantic retrieval (Chroma/FAISS/Qdrant)
- Agentic RAG chat with citations, safety checks, and trend computation
- Plain-language translation and glossary generation
- Async processing pipeline (upload → process → enriched report)
- Audit-friendly JSON APIs for agent interactions

---

## High-level architecture

- Web client (React/Vite): upload, reports, chat, profile
- Orchestrator (Express + MongoDB): auth, uploads, report APIs, agent proxy
- AI services (FastAPI): OCR, normalizer, classifier, indexer, summarizer, translator, RAG agent, safety/citation checker
- Vector DB: Chroma / FAISS / Qdrant (per-user namespaces)
- Object storage: S3 or MinIO for raw artifacts
- Job queue: BullMQ / RabbitMQ for asynchronous pipeline steps

Principles:
- Strict per-user isolation (filter by userId everywhere)
- Grounded outputs with explicit source citations
- Educational-only messaging and safety guardrails

---

## Core components & responsibilities

- Ingestion & OCR: extract text, pages, tables → produce parsed.json
- Units & Range Normalizer: canonicalize units, apply age/sex reference ranges
- Status Classifier: label results using tolerance bands and clinical rules
- Indexer & Embeddings: chunk sections, embed, write vectors to per-user namespace
- Summary Agent: produce concise, structured report summaries and findings
- Term Translator: generate plain-language glossary (multilingual support)
- Education/Advice Agent: provide neutral, non-diagnostic explanations and next-step education
- RAG Chat Agent: conversational QA grounded in user reports + structured facts
- Safety & Citation Checker: verify numeric grounding, add disclaimers, surface red flags

---

## Data model (overview)

- Raw artifacts: reports/{userId}/{reportId}/... (PDFs + parsed.json)
- Mongo collections:
  - lab_results(ownerId, reportId, panel, test, value, unitCanonical, refLow, refHigh, status, measuredAt, ...)
  - report_sections(ownerId, reportId, section, page, text)
- Embeddings: per-user namespace with metadata { userId, reportId, section, page, test?, measuredAt }

---

## Selected API endpoints (summary)

- POST /api/uploads — upload report (returns reportId)
- GET /api/reports — list reports and processing status
- GET /api/reports/:id — processed summary, findings, glossary
- POST /api/agents/chat — RAG chat: { userId, messages, reportIds?, mode, useHistory }
- Internal AI endpoints (versioned):
  - POST /ai/v1/ingest
  - POST /ai/v1/normalize
  - POST /ai/v1/index
  - POST /ai/v1/retrieve
  - POST /ai/v1/chat
  - POST /ai/v1/verify

Protocol: JSON request/response envelopes with canonical error types. Service-to-service auth via bearer tokens. User JWTs must remain backend-local.

---

## Local development (macOS)

Prerequisites:
- Node.js (recommended LTS)
- Python 3.10+
- Docker (for optional local vector DB and object storage)
- Redis (if using BullMQ)

Typical steps:
1. Install dependencies
   - Backend: cd api && npm install
   - Frontend: cd web && npm install
   - AI services: cd ai && python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
2. Start supporting services (optional)
   - docker compose up (Chroma/Redis/MinIO/Mongo)
3. Run dev servers
   - npm run dev (frontend)
   - npm run dev (express API)
   - uvicorn ai.main:app --reload (FastAPI services)
4. Upload test reports via the UI or API and follow pipeline events in logs.

---

## Testing & CI

- Unit tests for parsers, normalizers, and classifiers (PyTest / Jest)
- Contract tests for agent endpoints (use canned vectors and fixtures)
- CI checks: lint, type checks, unit tests, API smoke tests

---

## Privacy, safety & compliance

- Per-user data isolation and namespace scoping
- Consent gating for historical/comparative analyses
- Data deletion purges raw files, DB records, and vector entries
- Educational-only outputs; explicit disclaimers to avoid medical advice
- Keep prompts, reference ranges, and safety rules in version control

---

## Roadmap (short)

- M1: Auth, user profiles, consents
- M2: Uploads + ingestion pipeline
- M3: Structured memory & retrieval
- M4: Agentic RAG chat with safety/citation checks
- M5: Production readiness (observability, secrets, managed DBs)

---

## Contributing

- Use trunk-based workflow with short feature branches
- PR checks: lint, unit tests, API contract validations
- Add/update diagrams and README when changing APIs or agents

---

## License & contacts

- This project is intended for research/educational use. Review LICENSE file in repo root for specifics.
- For questions, open an issue or review the project board for priorities.
