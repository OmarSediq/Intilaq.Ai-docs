# Architecture

This document explains the **Clean Architecture** design of the Intilaq AI backend project.

Unlike `project_structure.md` which describes folder and file organization, this file focuses on the logical **layers**, how data flows, where responsibilities lie, and how modules are separated using principles like **Dependency Injection**, **Service Separation**, and **Repository Pattern**.

---

## 1. Overview of Architecture Layers

Intilaq AI follows a layered architecture inspired by Clean Architecture principles. It separates concerns into distinct layers:

- **API Layer (Routing)**: Handles HTTP requests/responses.
- **Service Layer (Business Logic)**: Handles logic, validation, AI integration, etc.
- **Repository Layer (Data Access)**: Communicates with databases (MongoDB, PostgreSQL, Redis).
- **Utility Layer**: Common helpers (logging, exception handling, JWT, etc).
- **Job Layer**: Asynchronous background processing via Redis Queue (RQ).

Each module (CV, Interview, Auth, HR, etc.) follows this layered separation independently.

---

## 2. Layer Details

### 2.1 API Layer (`api/`)

- Contains FastAPI `APIRouter`s for each domain.
- Does **not** include any business logic.
- Delegates to the corresponding service layer via dependency injection.

---

### 2.2 Service Layer (`domain_services/`)

Each domain (CV, Auth, HR, AI, etc.) has its own service package.

Examples:
- `cv_services/`: Handles logic for education, experience, etc.
- `auth_services/`: Password hashing, user login/registration.
- `ai_services/`: Whisper + Gemini logic for text/audio processing.
- `interview_services/`: Full control over session flow, feedback, and scoring.
- `hr_services/`: Contains client-side interview handling, HR-side creation, and summary.

Each service:
- Accepts sanitized inputs from the route.
- Calls repositories if needed.
- May trigger jobs (via job_triggers).
- Returns structured responses.

---

### 2.3 Repository Layer (`data_access/`)

This layer **abstracts all direct database communication**.

- **PostgreSQL (users, HR)**: `data_access/postgres/`
- **MongoDB (sessions, answers)**: `data_access/mongo/`
- **Redis (tokens, sessions)**: `data_access/redis/`

All services **depend on repositories** to fetch/write/update data.

Benefits:
- Easily mockable.
- Swappable DB tech in future.

---

### 2.4 Dependency Injection Layer (`core/providers/`)

- All services and repositories are injected via `Depends(...)` in route layer.
- Providers centralize how each service is constructed.
- Allows flexible swapping or mocking of services.
