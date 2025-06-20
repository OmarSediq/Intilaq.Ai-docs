# CV Builder

This document outlines the full backend structure and AI integration for the **CV Builder** feature in the Intilaq AI platform. It covers database schema, service layers, AI generation logic, and extensibility plans.

---

## Database Models (PostgreSQL + SQLAlchemy)

### User Model
Represents each registered user.

- `id`: Unique user ID.
- `username`: Unique name.
- `email`: Used for login and password reset.
- `hashed_password`: Encrypted using bcrypt.
- `is_verified`: Indicates if the email is verified.
- `headers`: One-to-many with the `Header` model.

---

### ResetCode Model
Used for password recovery.

- `id`: Auto-increment ID.
- `email`: Destination email for reset code.
- `code`: 6-digit verification code.
- `created_at`: Timestamp of request.

---

### Header Model (Main CV Record)
Represents one full CV/resume.

- `id`: Primary key.
- `full_name`, `job_title`, `email`, `phone_number`, `address`
- `years_of_experience`
- `github_profile`, `linkedin_profile`
- `user_id`: Foreign key to User.

**Relationships:**  
One-to-many with Education, Experience, SkillsLanguages, Certifications, Projects, VolunteeringExperience, Awards, Objective.

---

### Education
Educational background.

- `header_id`: FK to Header.
- `degree_and_major`
- `school`, `city`, `country`
- `start_date`, `end_date`
- `description`

---

### SkillsLanguages

- `header_id`: FK to Header.
- `skills`: Free-text field.
- `languages`: Spoken languages.
- `level`: Proficiency level (optional).

---

### Certifications

- `header_id`: FK to Header.
- `certification_title`
- `upload`: File reference (e.g., GridFS path).
- `link`: Optional certificate link.

---

### Projects

- `header_id`: FK to Header.
- `project_name`
- `description`
- `link`: GitHub or external.

---

### Experience

- `header_id`: FK to Header.
- `role`
- `company_name`
- `start_date`, `end_date`
- `description`

---

### VolunteeringExperience

- `header_id`: FK to Header.
- `organization`
- `role`
- `start_date`, `end_date`
- `description`

---

### Awards

- `header_id`: FK to Header.
- `award`
- `organization`
- `start_date`, `end_date`

---

### Objective

- `header_id`: FK to Header.
- `description`: Career goal or personal statement.

---

## Design Notes

- Only `Education` and `SkillsLanguages` are required.
- A user can have multiple CVs (multiple `Header` entries).
- `upload` and `link` fields are designed to integrate with **GridFS**, PDF export, and external storage.
- Fully normalized design for portability and integrity.

---

## Security

- Passwords are hashed using `bcrypt` via `passlib`.
- All foreign key relationships are secured using constraints.
- File uploads are stored as text references (e.g., ObjectIDs for GridFS).

---

## AI Service: `GeminiAIService`

Located in:  
`backend/domain_services/ai_services/gemini_ai_service.py`

This service uses **Google GenerativeAI SDK (Gemini 1.5 Flash)** to help users auto-generate professional CV sections.

### Supported Methods

| Method | Description | Output |
|--------|-------------|--------|
| `generate_objective(job_title, years_of_experience)` | Generates 4 career objectives | List of strings |
| `fetch_project_descriptions(project_name)` | 4 short project summaries | List of strings |
| `generate_experience(role, company_name, start_date, end_date)` | Up to 5 full experience blocks | List of text blocks |
| `generate_skills(job_title, years_of_experience)` | Technical, programming, and language skills | Dict of lists |
| `generate_volunteering_description(activity_role)` | 4 tailored volunteering descriptions | List of strings |

> Prompt templates are designed to **avoid placeholders** or unfinished outputs. All results are ready-to-use.

---

## AI Integration Workflow

1. Frontend requests content generation (e.g., objective).
2. FastAPI injects the `GeminiAIService`.
3. The service method is invoked.
4. Result is returned as 4–5 suggestions.
5. User selects one → stored in the database.

---

## Error Handling

- Each method uses try/except.
- If an error occurs: `HTTPException(500)` is raised.
- All logs are handled via `TraceableService`.

---

## Security & API Key Handling

- **API Key is injected** via dependency injection — never hardcoded.
- Gemini calls are **never exposed to the frontend**.
- Rate limits can be managed in Google AI Console to avoid abuse.

---

## CRUD + AI Services Overview

Each database model has its own dedicated **Service Class**, which:

- Handles validation and business logic.
- Optionally calls `GeminiAIService`.
- Saves user selections via repository layer.
- Is injected using `providers`.

| Service | Role | Repository | AI Calls? |
|---------|------|------------|-----------|
| `CVHeaderService` | Creates CV headers | `CVHeaderRepository` | ❌ |
| `CVEducationService` | Manages education entries | `EducationRepository` | ❌ |
| `CVExperienceService` | Adds experience blocks + bullet points | `ExperienceRepository` | ✅ |
| `CVProjectService` | Adds projects + description suggestions | `ProjectRepository` | ✅ |
| `CVObjectiveService` | Stores objective + AI suggestions | `CVObjectiveRepository` | ✅ |
| `CVSkillsService` | Skills/languages + categorized AI suggestions | `SkillsLanguagesRepository` | ✅ |
| `CVVolunteeringService` | Volunteering + AI generation | `VolunteeringRepository` | ✅ |
| `CVAwardService` | Awards and recognitions | `AwardRepository` | ❌ |
| `CVCertificationService` | Adds certifications (file or link) | `CertificationRepository` | ❌ |
| `CVResumeExportService` | Generates full HTML → PDF/DOCX and stores it | `ResumeRepository`, `HTMLRenderer`, `GridFSStorageService` | ❌ |

---

## Validation Pattern (Standard)

Example from service layer:

```python
header = await header_repo.get_by_user_id(user_id)
if not header:
    return error_response(404, "Header not found")
