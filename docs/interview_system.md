# Interview System

This document outlines the full backend structure, AI integration, and data flow for the **Interview System** feature in the Intilaq AI platform. It covers MongoDB models, Redis session handling, microservices, and AI evaluation flow.

---

## Database Models (MongoDB + Redis)

### MongoDB Collections

#### `questions`
Represents a full interview session with job metadata and AI-generated content.

- `session_id`: Unique session identifier.
- `user_id`: ID of the user taking the interview.
- `job_title`: Target job title.
- `level`: Optional job level (e.g., Junior, Senior).
- `questions`: List of:
  - `question_index`
  - `question`
  - `best_model_answer`: Gemini-generated ideal response.
- `current_question_index`: Active pointer during session.
- `created_at`: Timestamp.

---

#### `answers`
Each document represents one answer submitted by the user.

- `session_id`, `user_id`, `question_index`
- `answer_text`: Whisper-generated transcription.
- `similarity_score`: Float (0.00 to 10.00).
- `feedback`: Structured dict with:
  - `strengths`
  - `weaknesses`
  - `constructive_feedback`
- `timestamp`: UTC datetime of submission.

---

#### `session_results`
Final summary stored when session ends.

- `session_id`, `user_id`
- `final_score`: Out of 100.
- `answered_questions`: Count of submitted answers.
- `accuracy`: Averaged % across questions.
- `status`: `"ended"`
- `ended_at`: Completion datetime.

---

#### `user_home_summary`
Aggregated statistics for user dashboard.

- `user_id`
- `total_interviews`
- `total_answers`
- `avg_score`: Overall average score.
- `accuracy`: Average similarity across all answers.
- `last_session`: Dict with last session metadata.
- `updated_at`

---

### Redis Keys

Used for in-memory session control.

- `user:{user_id}:session_ids` → Active session ID list.
- `session:{session_id}:current_index` → Pointer to current question.
- `session:{session_id}:status` → "active", "ended", etc.

---

## Design Notes

- MongoDB supports nested `questions[]` and `answers[]` documents.
- Redis accelerates state transitions during interviews.
- All lookups use `{session_id, user_id}`—no joins required.
- Final evaluation is triggered by user or timeout via `InterviewScoreService`.

---

## AI Service: `GeminiAIService`

Located at:  
`backend/domain_services/ai_services/gemini_ai_service.py`

Uses **Gemini 1.5 Flash** to generate and analyze interview questions and responses.

### Supported Methods

| Method | Description | Output |
|--------|-------------|--------|
| `generate_interview_questions(role, level, description)` | Generates 10 tailored questions | List of strings |
| `generate_best_answer(question)` | Strong ideal answer for scoring | String |
| `generate_feedback(user_answer, question, ideal_answer)` | Strengths, weaknesses, feedback | Dict |
| `analyze_similarity_score(answer, model_answer)` | TF-IDF similarity (0–10) | Float |

> Prompts are engineered for clarity and completeness. No placeholders are returned.

---

## AI Workflow

1. **Frontend** sends job metadata (title, level, description).
2. **InterviewSessionService** uses Gemini to generate questions.
3. Questions and model answers stored in MongoDB (`questions`).
4. User answers via audio. Whisper converts to text.
5. Transcription stored in `answers`.
6. Feedback and score generated, stored in same answer record.
7. When session ends, stats saved to `session_results` and `user_home_summary`.

---

## Whisper Transcriber Service

Located at:  
`backend/domain_services/ai_services/whisper_transcriber_service.py`

### Key Features

- Accepts `.webm`, `.mp3`, or raw audio `bytes`.
- Uses FFmpeg + Whisper for accurate, fast transcription.
- Returns `{ text, language, error }`.

This is part of a **standalone microservice**, not the main FastAPI app.

---

## Speech-to-Text Micro-service (Whisper)

The Interview System delegates audio processing to an external FastAPI microservice for performance and modularity.

### Why a separate service?

| Reason | Benefit |
|--------|---------|
| GPU / CPU isolation | Whisper uses CUDA without locking main app threads |
| Cold-start caching | Model is loaded once, reused across all requests |
| Language agnostic | Any backend can POST to the transcriber endpoint |

---

## Service Layer (CRUD + AI)

All services follow Clean Architecture with clear separation between validation, business logic, and storage access.

| Service | Role | Repositories Used | AI Used |
|---------|------|-------------------|---------|
| `InterviewSessionService` | Creates, stores, and starts new sessions | `InterviewRepository`, `SessionRedisRepository` | ✅ |
| `InterviewAnswerService` | Accepts audio, sends to Whisper, stores text | `InterviewRepository`, `WhisperTranscriberService` | ✅ |
| `InterviewFeedbackService` | Uses Gemini to provide feedback on answers | `InterviewRepository`, `GeminiAIService` | ✅ |
| `InterviewScoreService` | Computes final session score and summary | `InterviewRepository` | ✅ |
| `InterviewQuestionService` | Returns next or full list of questions | `InterviewRepository`, `SessionRedisRepository` | ❌ |
| `InterviewValidatorService` | Session validation and permission checks | `InterviewRepository`, `SessionRedisRepository` | ❌ |

---

## Validation Pattern

Each service begins with session validation using `InterviewValidatorService`.

This ensures that the `session_id` belongs to the authenticated `user_id` before proceeding with logic or storage operations.
