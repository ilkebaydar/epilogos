# Project Charter: Epilogos

## 1. Project Purpose and Objectives
Epilogos is an intelligent post-meeting workflow platform that transforms technical discussions into actionable engineering items. By processing recorded engineering sessions, the system generates speaker-attributed transcripts, identifies core architectural decisions and automatically extracts structured tasks with strict schema validation. Through native integration with Atlassian Jira, Epilogos bridges the gap between verbal syncs and agile sprint backlogs, ensuring critical technical decisions are systematically captured and tracked.

## 2. Stakeholders & Roles
- **Product Owner:** Furkan Kural - kural21@itu.edu.tr
- **Software Engineering Team:**
    * **Furkan Kural** : Product Owner & Audio Pipeline Developer (Tracks customer requests and acceptance criteria; implements FFmpeg media preprocessing and Whisper/PyAnnote speech transcription pipeline)
    * **İlke Başak Baydar** : Project Manager & Integration Developer (Manages team backlog and milestone schedule; develops LLM extraction logic and Jira Cloud API integration)
    * **Muhammed Yunus Doğru** : Backend Lead & Core API Developer (Designs server architecture; develops FastAPI REST endpoints, database access services and background task processing)
    * **Abdul Akhundzada** : Security & Database Administrator (Designs PostgreSQL relational schemas, manages JWT authentication/authorization and implements basic PII data masking)
    * **Mehmet Enes Tekgöz** : QA & DevOps Lead (Sets up Docker environment, configures GitHub Actions CI/CD workflows and maintains automated test suites)
    * **Antigravity / Claude Code (AI Teammates)** : AI Code Assistant (Assists with boilerplate code, Jinja2/Tailwind UI components and unit test generation)
- **GH Repo/Project URL:** https://github.com/ilkebaydar/epilogos

## 3. Project Scope
### Project domain
The project operates in the domain of Developer Productivity Tools, Speech Processing and Applied NLP. Target users are agile software development teams and engineering managers who conduct recurring technical discussions, architecture reviews and sprint planning sessions across any meeting platform, managing their deliverables via Jira.

### In-Scope 
- **Media Ingestion & Async Processing:** Support for uploading common audio and video meeting formats (MP4, MKV, WAV) with background queue orchestration and task progress tracking.
- **Speaker-Attributed Transcription:** Generating timestamped transcripts separated by speaker identities using open-source speech recognition and diarization models.
- **Action Item & Decision Extraction:** Contextual parsing of transcripts into validated, structured items (title, assignee, description, priority) using strict schema enforcement.
- **Atlassian Jira Integration:** Direct creation of Jira tasks and sub-tasks from approved meeting action items via Atlassian Cloud REST APIs.
- **Synchronized Playback Interface:** A responsive web dashboard pairing meeting audio/video playback with interactive transcripts, allowing users to jump to timestamps by selecting text.
- **Data Protection & PII Sanitization:** Role-based access control (RBAC), secure credential handling (encryption at rest for Jira tokens) and rule-based masking of sensitive technical identifiers and personal secrets in meeting transcripts.

### Out-of-Scope
- **Active Meeting Participants (Live Call Bots):** Automated virtual bots actively joining live third-party calls (e.g., Zoom, Google Meet, MS Teams bots) or capturing live streaming audio in real time.
- **Biometric Speaker Enrollment & Profiling:** Pre-training voice fingerprints or maintaining cross-meeting persistent speaker identity databases (speaker identification is scoped strictly within the context of a single meeting session).
- **Two-Way Jira Synchronization:** Automated polling or bi-directional synchronization that mirrors downstream status changes from Jira back into Epilogos (integration is strictly one-way export/ticket creation).


## 4. Technology Stack

### Backend & Deployment Stack
- Python 3.11+ & FastAPI (Monolith core API)
- Celery & Redis (Task orchestration and GPU/IO queuing)
- PostgreSQL & SQLAlchemy (Relational schema with JSONB metadata support, Alembic migrations)
- FFmpeg (Audio extraction and 16 kHz mono normalization)
- Atlassian Jira REST API Client
- **Deployment & Hosting:** Docker & Docker Compose (Local dual-mode runtime environment); Render (Cloud mode deployment)

### Frontend Stack
- Jinja2 Server-Side Templates
- Tailwind CSS
- Vanilla JavaScript (Interactive playback and transcript sync)

### AI Stack & Inference Engine
- faster-whisper & pyannote.audio (Speech recognition and diarization)
- vLLM Server / Qwen 2.5 3B Instruct (AWQ) (Local LLM inference backend)
- Groq / OpenRouter Cloud Adapters (Cloud-mode fallback interface)
- Instructor & Pydantic (Structured extraction and schema enforcement)
- **AI Code Assistants:** Claude Code & Antigravity (Prompt logs tracked under `docs/ai-prompts/`)

### Testing & QA Stack
- Pytest, pytest-cov & unittest.mock (Backend unit/integration tests and mock suites)
- **Speech Evaluation:** `jiwer` (WER and DER accuracy benchmarking on AMI/ICSI datasets)
- **LLM Evaluation:** DeepEval (Structured extraction quality and schema compliance checks)

## 5. Project Milestones
| Deliverable                       | Deadline          |
|-----------------------------------|-------------------|
| Project Charter                   | 11.10.2026 23:59  |
| Project Plan                      | 25.10.2026 23:59  |
| Requirement Specification Report  | 08.11.2026 23:59  |
| Requirement Analysis Report       | 29.11.2026 23:59  |
| Software Design Report            | 13.12.2026 23:59  |
| Software Test Report              | 21.12.2026 23:59  |
| Codebase (Code Freeze)            | 25.12.2026 23:59  |
| Demo                               | 26.12.2026 23:59  |
| Presentation                      | 28.12.2026 23:59  |