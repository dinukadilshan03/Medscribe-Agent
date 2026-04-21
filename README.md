# MedScribe Agent

> AI-powered healthcare report assistant for patient comprehension, bilingual communication, and evidence-grounded follow-up guidance.

## 1) Executive Summary
MedScribe Agent is a healthcare AI assistant system designed to bridge the gap between technical medical reports and patient understanding.

- **Mission:** Make medical report interpretation accessible, safe, and understandable for every patient.
- **Vision:** Build a trustworthy, multilingual patient-facing intelligence layer for clinical data.
- **Unique patient value proposition:** Patients can upload reports and receive plain-language summaries, medical term explanations, bilingual translation (Sinhala/English), and grounded conversational Q&A without replacing clinical decision-making.

## 2) Problem Statement
Many patients receive high-complexity reports (lab panels, imaging results, specialist notes) with terminology that is difficult to understand.

Core gaps MedScribe addresses:
- **Medical literacy gap:** Technical wording creates confusion and anxiety.
- **Report comprehension challenges:** Patients often cannot identify what is normal, abnormal, or urgent.
- **Language accessibility barriers:** Non-English speakers may miss critical context.
- **Actionability gap:** Patients need educational next steps before consulting licensed clinicians.

## 3) Solution Overview
MedScribe Agent uses a **multi-agent AI pipeline** to process medical documents and deliver:
- structured report understanding,
- plain-language summaries,
- explainable term definitions with grounding,
- educational recommendations,
- bilingual patient communication,
- and compliance-friendly artifact generation.

## 4) Key Features

### A. Report Summarization
- Plain-language summaries from uploaded clinical documents.
- Highlights key findings for patient readability.

### B. Classification
- Medical domain tagging (e.g., lab/hematology/imaging context).
- Organizes findings into understandable categories.

### C. Term Explanation
- Medical glossary style explanations.
- Definitions are linked to report context for interpretability.

### D. Educational Advice
- Non-diagnostic recommendations for patient education.
- Encourages appropriate follow-up with healthcare professionals.

### E. Bilingual Translation (Sinhala/English)
- Translation for summaries and advice outputs.
- Focuses on preserving medical meaning and tone.

### F. Conversational QA
- Chat interface for report-based questions.
- Retrieval-grounded responses using indexed report chunks.

### G. Audit Trail
- Artifact-oriented processing (raw/cleaned/derived outputs).
- Designed for traceability and healthcare governance workflows.

## 5) Architecture Overview

```text
+------------------------+         +--------------------------+
|      Frontend UI       |  HTTP   |      Express Backend     |
| React 19 + Vite + MUI  +-------->+  Auth, Cases, Proxy APIs |
+-----------+------------+         +------------+-------------+
            |                                      |
            |                                      | internal HTTP
            |                                      v
            |                          +-------------------------+
            |                          | FastAPI AI Services     |
            |                          | Multi-agent processing  |
            |                          +-----------+-------------+
            |                                      |
            |                                      |
            v                                      v
+------------------------+              +------------------------+
|  MongoDB (users/cases) |              | Qdrant Vector Database |
+------------------------+              +------------------------+
            |
            v
+------------------------+
| Azure Blob Storage     |
| raw/cleaned/panels/... |
+------------------------+
```

## 6) Tech Stack
- **Frontend:** React 19, Vite, Material-UI, React Router, Axios
- **Backend:** Node.js, Express 5, Mongoose, JWT auth, Multer uploads
- **AI Services:** FastAPI, LangChain ecosystem, sentence-transformers, Gemini integration
- **Vector DB:** Qdrant (semantic retrieval)
- **Storage & Data:** MongoDB + Azure Blob Storage

## 7) Agent System Design (9+ Specialized Agents)

1. **Ingest/OCR Agent**
   - Extracts text from PDF/image reports.
   - Supports OCR and document parsing workflows.

2. **Normalizer Agent**
   - Cleans/normalizes clinical values and units.
   - Prepares machine-consistent report content.

3. **Indexer Agent**
   - Chunks processed content and generates embeddings.
   - Persists vectors into Qdrant.

4. **Retriever Agent**
   - Executes semantic retrieval over indexed report chunks.
   - Applies metadata filters (e.g., case/user context).

5. **Summarizer Agent**
   - Produces patient-friendly summary output.

6. **Classifier Agent**
   - Tags medical report content by domain/context.

7. **Explainer Agent**
   - Generates terminology explanations and contextual clarity.

8. **Advicer Agent**
   - Produces educational, evidence-aware recommendations.

9. **Translator Agent**
   - Sinhala/English translation for summaries/advice.

10. **Validator/Safety Agent**
    - Input/output checks and quality/safety guardrails.

## 8) Data Flow (End-to-End)

```text
[User Upload]
    |
    v
[Backend /api/cases] --> [AI pipeline /pipeline/run]
    |                              |
    |                              v
    |                     Ingest -> Normalize -> Classify
    |                              |
    |                              v
    |                         Index into Qdrant
    |                              |
    v                              v
[Persist case metadata]     [Summary/Explain/Advice/Translate]
    |                              |
    +---------------+--------------+
                    |
                    v
           [Frontend dashboard + chat]
```

## 9) Setup & Installation

### Prerequisites
- Node.js 18+
- Python 3.10+
- MongoDB instance
- Qdrant instance (local/cloud)
- Azure Blob Storage credentials (for file artifacts)

## 10) Frontend Setup
```bash
cd /home/runner/work/Medscribe-Agent/Medscribe-Agent/frontend
npm install
npm run dev
```
Frontend default dev URL: `http://localhost:5173`

## 11) Backend Setup
```bash
cd /home/runner/work/Medscribe-Agent/Medscribe-Agent/backend
npm install
npm run server
```
Backend default URL: `http://localhost:4000`

## 12) AI Services Setup
```bash
cd /home/runner/work/Medscribe-Agent/Medscribe-Agent/ai_services
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8001
```
AI service URL: `http://localhost:8001`

## 13) Database Configuration
- **MongoDB:** user accounts, case metadata, usage/conversation data.
- **Qdrant:** semantic vectors for report retrieval.
- **Azure Blob:** report artifacts (`raw`, `cleaned`, `panels`, exports).

## 14) Running Locally (All Services)
Open 3 terminals:
1. `frontend`: `npm run dev`
2. `backend`: `npm run server`
3. `ai_services`: `uvicorn app.main:app --reload --port 8001`

Then use the app from `http://localhost:5173`.

## 15) API Endpoints (Key)

### Backend (Express)
- `POST /api/cases` → upload case/report
- `GET /api/cases/:id/data` → processed case data
- `GET /api/cases/:id/meta` → case metadata
- `GET /api/cases/:id/export` → export case output
- `POST /api/pipeline/run` → stream AI pipeline execution
- `POST /api/chat/rag/chat` → chat with report context
- `POST /api/index/:caseId/build` → build vector index for case

### AI Services (FastAPI)
- `POST /pipeline/run`
- `POST /summary/process`
- `POST /advice/process`
- `POST /explain/process`
- `POST /classify/process-medical-report`
- `POST /translate-summary/process`
- `POST /translate-advice/process`
- `DELETE /vector/cleanup/{case_id}`
- `GET /health/`

## 16) Vector Database Design (Qdrant)
Current collection is environment-driven (`QDRANT_COLLECTION`, default `medscribe_cases`).

Payload design includes keys such as:
- `case_id`
- `user_id`
- `report_name`
- `doctor`
- `hospital`
- `chunk`

Isolation model:
- **Per-case and per-user filtering** is supported through payload filters in retrieval.
- Recommended production practice: namespace by tenant/user and enforce strict backend-side ownership checks.

## 17) Model Selection
- **Embedding model:** `sentence-transformers/all-MiniLM-L6-v2` (fast + compact for semantic retrieval).
- **LLM integration:** Gemini-family configurable via environment variables.
- **Reasoning approach:** retrieval-grounded generation to reduce unsupported answers.

## 18) Bilingual NLP (Sinhala/English)
- Dedicated translation routes for summary and advice outputs.
- Domain intent: preserve clinical entities, units, and cautionary phrasing.
- Recommended production hardening: term-level glossary locking for critical medical vocabulary.

## 19) User Interface
Primary workflows:
1. **Upload:** create case and attach report file.
2. **Dashboard/report views:** inspect summary, explanation, advice, translations.
3. **Chat:** ask follow-up questions grounded in indexed report content.
4. **Export:** retrieve generated artifacts for review/sharing.

## 20) Features Grid

| Feature | Patient Value | Agent/Route Support |
|---|---|---|
| Summarization | Understand report quickly | `/summary/process`, summarizer |
| Classification | Organize report by domain | `/classify/process-medical-report` |
| Term Explanation | Clarify medical terms | `/explain/process` |
| Educational Advice | Safe non-diagnostic guidance | `/advice/process` |
| Bilingual Translation | Sinhala/English accessibility | `/translate-summary/process`, `/translate-advice/process` |
| Conversational QA | Ask context-specific questions | `/api/chat/rag/chat`, `/rag/chat` |
| Auditability | Track generated artifacts | case metadata + storage outputs |

## 21) Healthcare Compliance Considerations
- Educational-assistant posture (not diagnostic automation).
- Need-to-know data handling and ownership filters.
- Audit-friendly artifacts and traceable processing chain.
- Recommended: formal HIPAA/GDPR controls (BAA, DLP, retention policies, SIEM monitoring) before production PHI usage.

## 22) Error Handling
- Backend proxies return upstream status where possible.
- AI service timeouts and failures are surfaced as user-readable API errors.
- Recommended UI behavior: fallback messaging + retry options + partial-result rendering.

## 23) Performance & Scalability
Current repository does not yet publish formal benchmark reports. Practical production targets:
- P50 summary generation under 10–20s (report-size dependent).
- Concurrent pipeline processing via queue/workers.
- Horizontal scaling for FastAPI inference workers and retrieval tier.

## 24) Testing Strategy
Current repository has limited explicit automated test setup in package scripts.
Recommended strategy:
- **Unit tests:** parser/normalizer/classifier utilities.
- **Integration tests:** backend ↔ AI route contract tests.
- **Retrieval tests:** deterministic fixture queries against Qdrant test collection.
- **Safety tests:** disclaimer enforcement and hallucination guard checks.

Example test cases:
- Upload valid PDF and verify case creation + artifact paths.
- Run pipeline and confirm summary/explain/advice keys exist.
- Chat query must return grounded response for known chunk.
- Translation output preserves numbers/units and key entities.

## 25) Deployment Considerations
- Separate frontend, backend, and AI services into independent deployable units.
- Use managed MongoDB + managed Qdrant for operational reliability.
- Apply strict secrets management and service-to-service auth.
- Add observability: tracing, structured logs, and health probes.

## 26) Docker Setup (Reference)
This repository currently does not include production-ready Docker files. Suggested baseline:
- `frontend` container (Vite build + static serving)
- `backend` container (Express API)
- `ai_services` container (FastAPI/uvicorn)
- `mongo` and `qdrant` services via compose for local orchestration

## 27) Configuration (Environment Variables)

### Backend
- `PORT`
- `MONGODB_URI`
- `JWT_SECRET`
- `AI_SERVICE_URL`
- `AI_SERVICE_TOKEN`
- `NODE_ENV`
- `SMTP_USER`, `SMTP_PASS`, `SENDER_EMAIL`
- `MAX_UPLOAD_MB`

### AI Services
- `QDRANT_URL`, `QDRANT_API_KEY`, `QDRANT_COLLECTION`
- `HF_EMBED_MODEL`
- `GOOGLE_API_KEY`, `LLM_MODEL`, `LLM_FALLBACK_MODEL`
- `MONGODB_URI`, `MONGO_DB_NAME`
- `AZURE_BLOB_ACCOUNT_URL`, `AZURE_BLOB_CONTAINER`, `AZURE_BLOB_SAS_TOKEN`
- `AZURE_OCR_URL`, `AZURE_OCR_KEY`, `AZURE_OCR_MODEL`
- `SERVICE_TOKEN`, `MAX_UPLOAD_MB`

## 28) Limitations
- LLM outputs can still be imperfect or incomplete.
- Hallucination risk exists without strict grounding checks.
- OCR quality impacts downstream accuracy.
- Should explicitly encourage clinical consultation for abnormal or urgent findings.

## 29) Medical Disclaimer
**MedScribe Agent is an educational assistant and does not provide medical diagnosis, treatment, or emergency triage. Always consult a licensed healthcare professional for medical decisions. If you have urgent symptoms, seek immediate clinical care.**

## 30) Privacy & Security
- Enforce HTTPS in production.
- Encrypt data in transit and at rest.
- Apply user-scoped data access checks at backend and vector retrieval layers.
- Implement retention/deletion workflows and access audits.

## 31) Contributing
Contributions are welcome, especially for:
- New or improved clinical agents.
- Better Sinhala/English medical translation quality.
- Safety/grounding and evaluation improvements.
- Developer tooling and test automation.

Suggested contribution flow:
1. Fork and create a feature branch.
2. Implement focused changes.
3. Add/update tests and docs.
4. Open a PR with clear rationale and validation steps.

## 32) Learning Outcomes Demonstrated
This project demonstrates:
- Multi-agent orchestration patterns for healthcare AI.
- RAG design with vector retrieval.
- Multilingual NLP for clinical communication.
- Practical integration of React + Express + FastAPI + Qdrant.

## 33) Future Enhancements
- Cross-document longitudinal trend analysis.
- Provider/hospital workflow integrations (EHR/FHIR adapters).
- Mobile-first patient companion app.
- Stronger automated evaluation and safety scoring.
- Role-based clinician dashboards and review controls.

## 34) Real-World Analysis Examples

### Example A: Lab Panel (CBC)
- **Input:** Hemoglobin low, MCV low, ferritin low.
- **Output:** Plain-language summary notes pattern consistent with possible iron deficiency and recommends discussing iron studies/follow-up with clinician.

### Example B: Lipid Profile
- **Input:** LDL elevated, HDL borderline.
- **Output:** Educational advice emphasizes lifestyle discussions and clinician-guided risk review; no direct treatment claims.

### Example C: Bilingual Family Communication
- **Input:** English report, Sinhala-speaking family member.
- **Output:** Summary and advice translated to Sinhala while preserving values, units, and caution language.

## 35) License & Author
- **License:** MIT (see `/home/runner/work/Medscribe-Agent/Medscribe-Agent/LICENSE`)
- **Repository Owner/Author:** [@dinukadilshan03](https://github.com/dinukadilshan03)

---

### Investor/Clinical-Evaluator Note
MedScribe Agent is positioned as a patient-understanding layer, not a diagnostic replacement. Its architectural strength is in combining ingestion, retrieval, multilingual explanation, and safety-aware educational output into a single workflow that can be audited and improved over time.
