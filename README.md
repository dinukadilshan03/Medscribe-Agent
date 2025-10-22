# MedScribe Agent

Overview
--------
MedScribe is an assistant built to help patients understand their medical reports. It ingests clinical documents (PDFs, scans), extracts and normalizes key facts (lab values, findings), and presents patient-friendly summaries, plain-language explanations, and evidence‑linked advice. MedScribe is intended for educational and research use only — it does not provide medical diagnoses or treatment recommendations. 🩺📄

Core features at a glance
- Summaries & findings — clear, citation-linked plain-language summaries for patients. 🔍
- Classification & tagging — organizes report sections and flags items needing attention. 🏷️⚠️
- Explanations & glossary — plain-language definitions linked to report locations. 💬📌
- Advice (educational) — empathetic, non-prescriptive next steps and red-flag highlights. 🤝
- Bilingual outputs — Sinhala ↔ English translations that preserve tone and meaning. 🌐
- RAG chat & retrieval — conversational Q&A grounded in report passages and structured facts. 💬🔗
- Per-user isolation & auditability — data scoped per user with audit-friendly artifacts. 🔒

## Quick summary
- Purpose: Provide patient-facing, plain-language summaries, explanations, translations, and evidence-grounded educational advice from clinical reports.
- Not a diagnostic tool. Follow safety, disclaimer, and per-user isolation principles.

## Features

Learn what each tool does and how to use it — from uploading reports to getting clear summaries, translations, and helpful next steps.

- 📝 Summarizer — Plain-language summary
  - A concise, patient-friendly summary that highlights key results and explains what they mean.
  - How it helps:
    - Key findings listed up front
    - Simple explanation of important values
    - Shareable summary for your doctor

- 🧭 Classifier — Organize findings
  - Groups report content by medical domain (e.g., blood, imaging, cardiovascular) so you can quickly see relevant sections.
  - How it helps:
    - Groups results by specialty
    - Flags items that may need attention
    - Makes long reports easy to scan

- 💡 Explainer — Glossary & explanations
  - Provides simple explanations for medical terms and links each definition back to where it appears in your report.
  - How it helps:
    - Plain-language definitions
    - Short contextual examples
    - Linked to report locations

- ⚕️ Advicer — Practical next steps
  - Non-prescriptive, empathetic suggestions such as lifestyle tips, follow-up ideas, and when to seek urgent care.
  - How it helps:
    - Actionable tips
    - Follow-up recommendations
    - Highlights urgent items

- 🔁 Translate Summarizer — Bilingual summaries
  - Translates the generated summary into Sinhala (or English) so non-English speakers can understand their results easily.
  - How it helps:
    - Sinhala ↔ English summaries
    - Preserves clinical meaning
    - Easy to share with family or providers

- 🌐 Translate Advicer — Bilingual advice
  - Translates the suggested next steps and advice between English and Sinhala while keeping the tone supportive and clear.
  - How it helps:
    - Sinhala ↔ English advice
    - Neutral, empathetic tone
    - Keeps medical nuance intact

### How to use these features

1. Upload your report from the Home page. Supported: PDF, JPG, PNG.
2. Wait for the pipeline to run — you’ll see Summary, Classifier, Explainer, Translator, and Advice panels.
3. Use the Term Translator to get Sinhala translations where needed.
4. Download or share the generated PDF summaries (available on paid plans).


## High-level architecture
- Frontend: React (Vite) — upload, reports, chat, features UI.
- Orchestrator API: Express + MongoDB — auth, uploads, agent proxy, user-facing APIs.
- AI services: FastAPI (Python) — modular agent endpoints (ingest, normalize, index, retrieve, summarize, translate, advise, verify).
- Vector DB: Chroma / FAISS / Qdrant (per-user namespaces).
- Storage: S3 / MinIO for raw artifacts and parsed outputs.
- Queue: BullMQ / RabbitMQ for async pipeline steps.

## Agents & agent chains (LangChain-style design)
Agents are implemented as discrete services/tools and composed into deterministic chains. The design follows common LangChain patterns: tools, retrievers, chains, and memory.

Primary agents
- Ingest/OCR agent — PDF/image ingestion, OCR, table extraction → parsed.json.
- Normalizer agent — canonicalize units, compute age/sex-aware reference ranges.
- Classifier agent — report/section classification and domain tagging.
- Indexer agent — chunking, embedding, and storing vectors in per-user namespaces.
- Retriever (RAG) agent — vector retrieval, passage ranking, and context assembly.
- Summarizer agent — produce structured, citation-linked plain-language summaries.
- Translator agent — bilingual rendering of summaries/advice.
- Advisor agent — evidence-grounded, empathetic next-step suggestions.
- Verifier / Safety agent — numeric grounding checks, fidelity/safety rules, disclaimers.

Common agent chains
- Ingest → Normalize → Classify → Index
  - Full ingestion pipeline to populate parsed artifacts, DB records, and vectors.
- Retriever → Verifier → Summarizer → Translator (optional)
  - RAG query chain for grounded Q&A and bilingual output.
- Retriever + Summarizer + Advisor + Safety
  - Produces evidence-linked advice with safety checks and disclaimers.

LangChain-style patterns used
- Agent-as-tool: services expose discrete tool endpoints (embed, retrieve, explain).
- Chains: composed sequences of tools + LLM calls for multi-step reasoning.
- Retriever + LLM: vectors supply grounded context to LLMs for faithful generation.
- Memory: short-term conversation memory + long-term structured facts in DB.
- Post-generation safety: verifiers that re-check values and add mandated disclaimers.

## API surface (internal AI services & orchestrator)
The ai_services/routes folder contains FastAPI route modules. The list below summarizes the primary endpoints exposed by AI services and the orchestrator. See ai_services/routes for exact request/response contracts and field names.

Core AI/internal endpoints (versioned under /ai/v1)
- POST /ai/v1/ingest
  - Purpose: upload or reference raw artifact → perform OCR, parsing, produce parsed.json and page/text outputs.
  - Example payload: { "userId": "...", "reportId": "...", "sourceUrl": "...", "fileBytes"? }
- POST /ai/v1/normalize
  - Purpose: canonicalize units, compute reference ranges, enrich lab results.
  - Example payload: { "reportId": "...", "labResults": [...] }
- POST /ai/v1/index
  - Purpose: chunk text, create embeddings, write to per-user vector namespace.
  - Example payload: { "reportId": "...", "sections": [...], "namespace": "user:{userId}" }
- POST /ai/v1/retrieve
  - Purpose: vector retrieval for a query, returns ranked passages + metadata.
  - Example payload: { "userId": "...", "query": "...", "k": 10, "reportIds"? }
- POST /ai/v1/summary  (routes: summarizer.py / summary_route.py)
  - Purpose: generate structured, citation-linked summaries of reports or passages.
  - Example payload: { "userId": "...", "reportId": "...", "mode": "concise|detailed" }
- POST /ai/v1/translate/summary  (translate_sum_route.py)
  - Purpose: translate generated summaries (Sinhala ↔ English).
  - Example payload: { "text": "...", "target": "si|en" }
- POST /ai/v1/translate/advice  (translate_adv_route.py)
  - Purpose: translate advice outputs while preserving tone and nuance.
  - Example payload: { "text": "...", "target": "si|en" }
- POST /ai/v1/advice  (advisor.py / advice_route.py)
  - Purpose: generate educational, non-diagnostic next steps tied to evidence.
  - Example payload: { "userId": "...", "reportIds": [...], "context": "..." }
- POST /ai/v1/classify  (classifier.py / classify_route.py)
  - Purpose: classify report sections, tests, and tag domains/flags.
  - Example payload: { "reportId": "...", "text": "..." }
- POST /ai/v1/explain  (explain.py / explaine_route.py)
  - Purpose: generate plain-language definitions and link them to report locations.
  - Example payload: { "terms": [...], "reportId": "..." }
- POST /ai/v1/validate or /ai/v1/verify  (validator.py)
  - Purpose: post-generation verification (numeric grounding, citation checks).
  - Example payload: { "response": "...", "reportId": "..." }
- POST /ai/v1/vector/cleanup  (vector_cleanup.py)
  - Purpose: remove or reindex vectors for a report or user namespace.
  - Example payload: { "namespace": "user:{userId}", "reportId"? }
- GET /ai/v1/health  (health.py)
  - Purpose: service health and readiness checks.

Orchestrator / user-facing endpoints (Express)
- POST /api/uploads — upload report (returns reportId)
- GET /api/reports — list reports and processing status
- GET /api/reports/:id — processed summary, findings, glossary
- POST /api/agents/chat — RAG chat: { userId, messages, reportIds?, mode, useHistory }
- Conversation & cases endpoints (routes: conversations.py, cases.py)
  - Example: POST /api/conversations, GET /api/conversations/:id, POST /api/cases

Note: consult ai_services/routes and the Express routes folder for precise request/response schemas and auth requirements (service-to-service bearer tokens and backend-local user JWTs).

## Data model (concise)
- Raw artifacts: reports/{userId}/{reportId}/... (PDFs + parsed.json)
- Mongo collections: lab_results, report_sections, users, conversations, cases
- Embeddings: per-user namespace with metadata { userId, reportId, section, page, test?, measuredAt }

## Local development (macOS)
Prereqs: Node.js (LTS), Python 3.10+, Docker (optional), Redis (optional).
Setup (quick)
1. Backend: cd api && npm install
2. Frontend: cd frontend && npm install
3. AI services: cd ai_services && python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
4. Optional infra: docker compose up (Chroma/Redis/MinIO/Mongo)
5. Run dev servers:
   - npm run dev (frontend)
   - npm run dev (express API)
   - uvicorn app.main:app --reload (ai_services FastAPI)

## Testing & CI
- Unit tests: parsers, normalizers, classifiers (pytest / jest)
- Contract tests: agent endpoints with canned vectors/fixtures
- CI: lint, type checks, unit tests, API smoke tests

## Privacy, safety & compliance
- Per-user isolation (namespaces + ownerId scoping)
- Consent gating for historical/comparative analyses
- Data deletion purges raw files, DB records, and vectors
- Educational-only outputs with explicit disclaimers and safety rules in source control

## Contributing
- Trunk-based workflow, short feature branches
- PR checks: lint, unit tests, contract validation
- Keep prompts, reference ranges, and safety rules in source control

## Where to look next
- ai_services/routes — authoritative API contracts for AI services
- ai_services/agents — individual agent implementations and tools
- api/routes — orchestrator / user-facing endpoints
- frontend/src/pages/Features.jsx — canonical feature descriptions used in the UI

## License
See LICENSE in repository root.
