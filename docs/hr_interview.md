# HR Interview

This chapter documents the recruiter-side module that lets HR teams design, send, and evaluate asynchronous interviews.  
You will see how data moves between MongoDB, GridFS, Redis, RQ-workers, the Whisper micro-service, and Google Gemini—  
all stitched together with Clean Architecture service classes.

The goal is to offer a narrative you can copy straight into a graduation thesis without extra editing.

---

## 1 Authentication & Account Lifecycle (Recruiter)

- **Registration** – `HRRegisterService` hashes the password and writes a six-digit code into `reset_codes`.
- **Verification** – `HRVerificationService` confirms the code and flips `is_verified` to 1.
- **Login** – `HRAuthService` checks credentials, then drops an **access** and **refresh** JWT in *Secure, HttpOnly* cookies (`SameSite=None`, TTL 7 days).
- **Resend code** – available until the account is verified.

> **Why it matters:** No recruiter API—including interview creation—is exposed until the email is positively verified.

---

## 2 Persistent Storage Design

### 2-A. MongoDB Collections

- **`hr_interviews`**  
  Stores interview metadata.  
  Fields: `interview_token`, `job_title`, `level`, `specific_date / date_range`, `hr_id`, `created_at`.

- **`hr_interview_questions`**  
  One document per interview.  
  Contains `questions[]`, where each entry includes:  
  `index`, raw text, `response_type`, `time_limit`, `ideal_answer`.

- **`hr_answers`**  
  Created when a candidate opens the interview link.  
  Contains: `user_email`, `login_time`, `review_status`, and a dynamic `answers[]` array.  
  Once all answers are scored, the `overall_score` percentage is written.

- **`hr_summary`**  
  A denormalised dashboard document.  
  Stores per-interview participant lists, pending/accepted counters, and timestamps for a performant HR dashboard.

### 2-B. GridFS Bucket

Large binary objects stored include:

- Raw video uploads  
- Compressed `.webm` video  
- Extracted audio snippets

`HRGridFSStorageService` is the only service allowed to read/write those ObjectIDs securely.

### 2-C. Embedded `answers[]` Schema

Each element inside `answers[]` holds:

- `question_index` – pointer to the master question list
- `response_type` – `"video"` or `"text"`
- `video_file_id`, `audio_file_id` – GridFS IDs for media (if present)
- `answer_text` – filled for text-based answers
- `answer_duration`, `question_time_limit`, `time_exceeded`
- `transcript` – Whisper-generated transcript
- `score` – Gemini similarity score (0–10)
- `feedback` – three-part structured feedback: strengths, weaknesses, advice

---

## 3 Service Layer (Clean Architecture)

- **HRInterviewService**  
  Creates interview metadata and triggers Gemini to generate 10 tailored questions.

- **HRInvitationService**  
  Saves invitation data and dispatches the email job, avoiding SMTP delays.

- **HRAnswerService**  
  Starts sessions, saves answers, and triggers background processing (video or inline text evaluation).

- **HRInterviewEvaluationService**  
  Fetches video and transcript data for reviewers.

- **HRUserSummaryService**  
  Rebuilds the dashboard after submissions or reviews.

> All services use dependency-injection to remain framework-agnostic and easily testable.

---

## 4 Background Processing with RQ Workers

- **`process_video` job**  
  Triggered on video upload:
  1. Compress raw video  
  2. Extract audio  
  3. Send audio to Whisper  
  4. Get feedback from Gemini  
  5. Save results into `hr_answers`

- **`evaluate_transcription` job**  
  For text responses:
  - Sends content to Gemini  
  - Stores `score` and `feedback` in the answer

> Both jobs run inside an isolated **rq-worker** container to avoid CPU/GPU bottlenecks on the FastAPI API server.

---

## 5 E-mail Invitation Infrastructure

1. `EmailTemplateService` renders `interview_invitation.html` with name, job, date, and a secure invite link.
2. `EmailSenderService` sends the rendered HTML via SMTP using credentials in `settings.*`.
3. `send_invitation_job()` runs inside the RQ worker, looping through all recipients without blocking the main thread.
4. `EmailJobTriggerService` provides a public method the API can call to enqueue the job.

---

## 6 End-to-End Candidate Flow

1. **Link Activation**  
   When the candidate opens the invite link, `HRAnswerService.create_session` creates a document in `hr_answers` with `login_time`.

2. **Response Upload**  
   - *Video Path:* the browser uploads `.webm`, which is stored in GridFS. The `process_video` job is triggered.  
   - *Text Path:* the answer is sent to Gemini directly for real-time feedback and scoring.

3. **Per-question Scoring**  
   Gemini evaluates the answer and returns a score (0–10) plus structured feedback in three blocks:  
   *Strengths*, *Weaknesses*, *Suggestions*.

4. **Overall Score Calculation**  
   When all responses are scored, `overall_score` is computed as a percentage.

5. **HR Review Process**  
   The HR panel streams video or text answers and updates `review_status` to `accepted` or `rejected`.

6. **Dashboard Refresh**  
   `HRUserSummaryService` updates `hr_summary` with totals and reviewed counts.

---

## 7 Security & Token Governance

- JWTs use HS256 with a payload of `user_id` and role `"hr"`.
- Refresh tokens are stored in Redis and deleted on logout (`DEL` operation).
- GridFS file access is only permitted with valid JWTs.
- SMTP credentials are injected at runtime from secure env vars.
- Whisper runs in an isolated, GPU-backed service with no API access.
- Invite URLs contain only a secure `interview_token` (no PII).

---

## 8 Why this Architecture Matters for Your Thesis

- **Scalability**  
  Audio and video processing is pushed to workers, keeping the API sub-100ms.

- **Observability**  
  MongoDB stores every interaction, allowing easy audit trails and analytics.

- **Extensibility**  
  New response types (e.g., code editors, diagrams) can be supported via new workers without modifying the API layer.

> Together, these decisions create a **production-grade hiring system**, aligned with modern software engineering practices—  
perfect for showcasing in your graduation thesis.
