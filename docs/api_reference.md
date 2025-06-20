# API Reference – Intilaq AI

This file documents the active API endpoints in the Intilaq AI project. All endpoints listed follow a consistent response pattern using the custom response template defined below.

---
## User Authentication Schemas

---

### SignupRequest

Used to register a new user account.

- `username` (str) – required  
- `email` (EmailStr) – required  
- `password` (str) – required  
- `confirm_password` (str) – required  

---

### LoginRequest

Used for user login.

- `email` (EmailStr) – required  
- `password` (str) – required  

---

### VerifyAccountRequest

Used to verify a newly registered account.

- `code` (str) – required  
- `new_password` (str) – optional (used for recovery flows)  

---

### ForgotPasswordRequest

Used to trigger a password reset via email.

- `email` (EmailStr) – required  

---

### ResetPasswordRequest

Used to reset the password using a verification code.

- `email` (EmailStr) – required  
- `reset_code` (str) – required  
- `new_password` (str) – required  

---

### ResendCodeRequest

Used to resend the account verification code.

- `email` (EmailStr) – required  

---

### RefreshTokenRequest

Used to request a new access token using a refresh token.

- `refresh_token` (str) – required  

---

### UpdateUserRequest

Used to update the current user's account credentials.

- `username` (str) – required  
- `email` (EmailStr) – required  
- `password` (str) – required  
- `confirm_password` (str) – required  

---

## HR Authentication Schemas

### `JobData`
Represents job metadata used for generating interview questions.

- `job_title`: *(str)* – Title of the job.
- `level`: *(Optional[str])* – Seniority level (e.g., Junior, Senior).
- `description`: *(Optional[str])* – Job description or notes.

---

### `HrSignupRequest`
Used for registering a new HR account.

- `name`: *(str)* – HR person's name.
- `company_name`: *(str)* – Name of the company.
- `business_email`: *(EmailStr)* – Official company email address.
- `company_field`: *(str)* – Industry or field of the company.
- `password`: *(str)* – Password.
- `confirm_password`: *(str)* – Confirmation of the password.

---

### `HrLoginRequest`
Used to authenticate an existing HR account.

- `business_email`: *(EmailStr)* – HR login email.
- `password`: *(str)* – HR login password.

---

### `HrVerifyRequest`
Used for verifying an HR account with a code.

- `code`: *(str)* – 6-digit or custom verification code.

---

### `HrResendCodeRequest`
Used to resend the verification code to the HR email.

- `business_email`: *(EmailStr)* – HR email.

---

### `InterviewLoginRequest`
Used when a candidate joins an interview.

- `name`: *(str)* – Candidate's name.
- `email`: *(EmailStr)* – Candidate's email.

---

### `InterviewAnswerRequest`
Used when a candidate submits a text answer.

- `text_answer`: *(Optional[str])* – Written answer (if response_type is `text`).

---

### `InterviewMetadataRequest`
Used by HR to define interview metadata and timing.

- `job_title`: *(str)* – Job title for the interview.
- `level`: *(str)* – Job level or experience requirement.
- `job_requirements`: *(Optional[str])* – Specific job requirements.
- `specific_date`: *(Optional[datetime])* – Scheduled interview date (if exact).
- `time`: *(Optional[str])* – Time of interview (if applicable).
- `date_range`: *(Optional[str])* – Flexible date range in format `"YYYY-MM-DD to YYYY-MM-DD"`.

---

### `HRAddQuestionRequest`
Used to add or update a question in an interview.

- `question_text`: *(Optional[str])* – The question itself.
- `response_type`: *(Optional[str])* – Must be `"text"` or `"video"`.
- `time_limit`: *(Optional[int])* – Duration in seconds to answer the question.

---

### `InterviewInvitationRequest`
Used to invite participants to an interview.

- `emails`: *(Optional[List[EmailStr]])* – List of participant emails.
- `email_description`: *(Optional[str])* – Optional message in the email.
- `interview_link`: *(str)* – URL the participant uses to access the interview.



## CV Builder Schemas

---

### HeaderRequest

Used to submit personal information for the resume header.

- `full_name` (str) – required  
- `job_title` (str) – optional  
- `email` (EmailStr) – required  
- `phone_number` (str) – optional  
- `address` (str) – optional  
- `linkedin_profile` (str) – optional  
- `github_profile` (str) – optional  
- `years_of_experience` (int) – optional  

---

### AwardsRequest

Used to submit an award entry.

- `award` (str) – required  
- `organization` (str) – optional  
- `start_date` (date) – required  
- `end_date` (date) – optional  

---

### CertificationRequest

Used to add a certification to the resume.

- `certification_title` (str) – required  
- `upload` (str) – optional  
- `link` (str) – optional  

---

### EducationRequest

Used to add an education entry.

- `degree_and_major` (str) – required  
- `school` (str) – required  
- `city` (str) – optional  
- `country` (str) – optional  
- `start_date` (date) – required  
- `end_date` (date) – optional  
- `description` (str) – optional  

---

### ExperienceRequest

Used to add a professional experience entry.

- `role` (str) – required  
- `start_date` (date) – required  
- `end_date` (date) – optional  
- `company_name` (str) – optional  

---

### ExperienceSaveRequest

Used to save the selected AI-generated experience description.

- `selected_description` (str) – required  

---

### ObjectiveSaveRequest

Used to submit or edit the objective statement.

- `description` (str) – optional  

---

### ProjectRequest

Used to add a project entry.

- `project_name` (str) – required  
- `link` (str) – optional  

---

### ProjectDescriptionSaveRequest

Used to save the selected project description generated by AI.

- `selected_description` (str) – required  

---

### SkillsLanguagesRequest

Used to submit user skills and languages.

- `languages` (str) – required  
- `skills` (str) – optional  
- `level` (str) – optional  

---

### SaveSkillsRequest

Used to save AI-suggested or selected skills and language levels.

- `selected_skills` (str) – required  
- `selected_language` (str) – required  
- `selected_level` (str) – required  

---

### VolunteeringRequest

Used to submit a volunteering experience.

- `organization` (str) – required  
- `role` (str) – required  
- `start_date` (date) – required  
- `end_date` (date) – optional  
- `description` (str) – optional  

---

### SaveVolunteeringRequest

Used to save the selected AI-generated volunteering description.

- `selected_description` (str) – required  

---

# Modules

## 1. User Authentication

### POST `/api/users/register/`
Registers a new user account.

- **Request Body**: `SignupRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/users/verify-account/`
Verifies the account using a 6-digit code.

- **Request Body**: `VerifyAccountRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/auth/login/`
Logs in a verified user and returns access and refresh tokens in cookies.

- **Request Body**: `LoginRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/auth/logout/`
Clears cookies and deletes the refresh token from Redis.

- **Request**: Cookie-based request  
- **Response**: `BaseResponseModel`

---

### POST `/api/auth/refresh-token/`
Generates new access and refresh tokens from a valid refresh token.

- **Request**: Cookie-based request  
- **Response**: `BaseResponseModel`

---

### POST `/api/security/forgot-password/`
Triggers the password-reset flow.

- **Request Body**: `ForgotPasswordRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/users/resend-verification-code/`
Resends the verification code for unverified users.

- **Request Body**: `ResendCodeRequest`  
- **Response**: `BaseResponseModel`

---

## 2. HR Authentication

### POST `/api/hr/register/`
Registers a new HR account.

- **Request Body**: `HrSignupRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/hr/verify-account/`
Verifies an HR account using a code.

- **Request Body**: `HrVerifyRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/hr/login/`
Logs in an HR user and returns token cookies.

- **Request Body**: `HrLoginRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/hr/resend-verification-code/`
Resends the HR verification code.

- **Request Body**: `HrResendCodeRequest`  
- **Response**: `BaseResponseModel`


---

## 3. CV Builder

> Manages user-resume data (experience, education, projects, skills) and AI-generated suggestions.

### POST `/api/headers/`
Creates a new CV header.

- **Auth Required**: Yes  
- **Request Body**: `HeaderRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/awards/`
Creates a new award entry.

- **Auth Required**: Yes  
- **Request Body**: `AwardsRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/certifications/`
Creates a certification entry.

- **Auth Required**: Yes  
- **Request Body**: `CertificationRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/educations/`
Creates an education entry.

- **Auth Required**: Yes  
- **Request Body**: `EducationRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/experiences/`
Creates a work-experience entry.

- **Auth Required**: Yes  
- **Request Body**: `ExperienceRequest`  
- **Response**: `BaseResponseModel`

---

### GET `/api/experiences/suggestions/{experience_id}/`
Generates AI experience suggestions.

- **Auth Required**: Yes  
- **Path Param**: `experience_id` (int)  
- **Response**: `BaseResponseModel`

---

### PUT `/api/experiences/save-description/{experience_id}/`
Saves an AI-generated experience description.

- **Auth Required**: Yes  
- **Path Param**: `experience_id` (int)  
- **Request Body**: `ExperienceSaveRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/objectives/suggestions/`
Generates AI objective suggestions.

- **Auth Required**: Yes  
- **Request Body**: `ObjectiveSaveRequest`  
- **Response**: `BaseResponseModel`

---

### PUT `/api/objectives/save-description/{objective_id}/`
Saves an AI-generated objective.

- **Auth Required**: Yes  
- **Path Param**: `objective_id` (int)  
- **Request Body**: `ObjectiveSaveRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/projects/`
Creates a project entry.

- **Auth Required**: Yes  
- **Request Body**: `ProjectRequest`  
- **Response**: `BaseResponseModel`

---

### GET `/api/projects/generate-description/{project_id}/`
Generates an AI description for a project.

- **Auth Required**: Yes  
- **Path Param**: `project_id` (int)  
- **Response**: `BaseResponseModel`

---

### PUT `/api/projects/save-description/{project_id}/`
Saves an AI-generated project description.

- **Auth Required**: Yes  
- **Path Param**: `project_id` (int)  
- **Request Body**: `ProjectDescriptionSaveRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/skills-languages/`
Creates skills and languages.

- **Auth Required**: Yes  
- **Request Body**: `SkillsLanguagesRequest`  
- **Response**: `BaseResponseModel`

---

### GET `/api/skills/suggestions/`
Generates AI skill suggestions.

- **Auth Required**: Yes  
- **Response**: `BaseResponseModel`

---

### PUT `/api/skills/save/{skills_id}/`
Saves selected skills and language level.

- **Auth Required**: Yes  
- **Path Param**: `skills_id` (int)  
- **Request Body**: `SaveSkillsRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/volunteerings/`
Creates a volunteering entry.

- **Auth Required**: Yes  
- **Request Body**: `VolunteeringRequest`  
- **Response**: `BaseResponseModel`

---

### GET `/api/volunteerings-suggestions/{volunteering_id}/`
Generates AI volunteering suggestions.

- **Auth Required**: Yes  
- **Path Param**: `volunteering_id` (int)  
- **Response**: `BaseResponseModel`

---

### PUT `/api/volunteerings-save-description/{volunteering_id}/`
Saves an AI-generated volunteering description.

- **Auth Required**: Yes  
- **Path Param**: `volunteering_id` (int)  
- **Request Body**: `SaveVolunteeringRequest`  
- **Response**: `BaseResponseModel`

---

### GET `/api/generate-cv/`
Generates the full resume in HTML.

- **Auth Required**: Yes  
- **Response Type**: `HTMLResponse`

---

### GET `/api/download-cv/pdf/`
Exports the resume as PDF and stores it.

- **Auth Required**: Yes  
- **Response**: `BaseResponseModel`

---

### GET `/api/download-cv/docx/`
Downloads the resume in DOCX format.

- **Auth Required**: Yes  
- **Response**: `BaseResponseModel`

---

### GET `/api/resumes/{file_id}/download`
Downloads a saved resume file.

- **Auth Required**: Yes  
- **Path Param**: `file_id` (str)  
- **Response**: `BaseResponseModel`

---

### GET `/api/regenerate-cv/`
Regenerates the HTML CV from the latest data.

- **Auth Required**: Yes  
- **Response Type**: `HTMLResponse`

---

## 4. Interview System

### POST `/api/sessions/`
Creates a new interview session.

- **Auth Required**: Yes  
- **Request Body**: `JobData`  
- **Response**: `BaseResponseModel`

---

### POST `/api/sessions/{session_id}/start`
Starts the interview session.

- **Auth Required**: Yes  
- **Path Param**: `session_id` (int)  
- **Response**: `BaseResponseModel`

---

### GET `/api/sessions/{session_id}/questions`
Retrieves all questions in a session.

- **Auth Required**: Yes  
- **Path Param**: `session_id` (int)  
- **Response**: `BaseResponseModel`

---

### GET `/api/sessions/{session_id}/questions/next`
Retrieves the next question in the session.

- **Auth Required**: Yes  
- **Path Param**: `session_id` (int)  
- **Response**: `BaseResponseModel`

---

### POST `/api/sessions/{session_id}/answers`
Submits an answer (video file) to a question.

- **Auth Required**: Yes  
- **Path Param**: `session_id` (int)  
- **Form Data**: `file: UploadFile`  
- **Response**: `BaseResponseModel`

---

### GET `/api/sessions/{session_id}/answers/{question_index}/feedback`
Retrieves feedback for a given answer.

- **Auth Required**: Yes  
- **Path Params**: `session_id`, `question_index`  
- **Response**: `BaseResponseModel`

---

### POST `/api/sessions/{session_id}/end`
Ends the session and triggers scoring.

- **Auth Required**: Yes  
- **Path Param**: `session_id` (int)  
- **Response**: `BaseResponseModel`

---

### GET `/api/sessions/{session_id}/score`
Retrieves the final score of the session.

- **Auth Required**: Yes  
- **Path Param**: `session_id` (int)  
- **Response**: `BaseResponseModel`

---

### GET `/api/home/summary`
Returns user statistics for the dashboard.

- **Auth Required**: Yes  
- **Response**: `BaseResponseModel`

---

### GET `/api/home/interview-sessions`
Returns past sessions the user attended.

- **Auth Required**: Yes  
- **Response**: `BaseResponseModel`

---

### GET `/api/sessions/{session_id}/details`
Returns detailed metadata about a session.

- **Auth Required**: Yes  
- **Path Param**: `session_id` (int)  
- **Response**: `BaseResponseModel`

---

### GET `/api/resumes/download`
Downloads the most recent resume.

- **Auth Required**: Yes  
- **Response**: `BaseResponseModel`

---

## 5. HR Interview

### POST `/api/hr/interview/create`
Creates interview metadata (title, description, timings, etc.).

- **Auth Required**: Yes  
- **Request Body**: `InterviewMetadataRequest`  
- **Response**: `BaseResponseModel`

---

### PUT `/api/hr/interview/{interview_token}/questions/{index}`
Updates or replaces a specific interview question.

- **Auth Required**: Yes  
- **Path Params**:  
  - `interview_token` (str)  
  - `index` (int)  
- **Request Body**: `HRAddQuestionRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/hr/interview/{interview_token}/invitations/send`
Sends email invitations to candidates for the interview.

- **Auth Required**: Yes  
- **Path Param**: `interview_token` (str)  
- **Request Body**: `InterviewInvitationRequest`  
- **Response**: `BaseResponseModel`

---

### GET `/api/hr/interview/video-stream/{interview_token}/{index}`
Streams a candidate’s video answer by question index.

- **Auth Required**: Yes  
- **Path Params**:  
  - `interview_token` (str)  
  - `index` (int)  
- **Query Param**: `user_email` (str)  
- **Response**: `StreamingResponse` (video)

---

### GET `/api/hr/interview/answer-indexes/{interview_token}`
Returns indexes of all answers grouped by type (video / text).

- **Auth Required**: Yes  
- **Path Param**: `interview_token` (str)  
- **Query Param**: `user_email` (str)  
- **Response**: `BaseResponseModel`

---

### GET `/api/hr/interview/video-question/{interview_token}/{index}`
Returns the video question, ideal answer, and candidate’s answer.

- **Auth Required**: Yes  
- **Path Params**: `interview_token` (str), `index` (int)  
- **Query Param**: `user_email` (str)  
- **Response**: `BaseResponseModel`

---

### GET `/api/hr/interview/text-question/{interview_token}/{index}`
Returns a text-based question with candidate’s answer.

- **Auth Required**: Yes  
- **Path Params**: `interview_token` (str), `index` (int)  
- **Query Param**: `user_email` (str)  
- **Response**: `BaseResponseModel`

---

### PUT `/api/hr/interview/{interview_token}/review-status`
Updates a candidate’s review status (e.g., accepted / rejected).

- **Auth Required**: Yes  
- **Path Param**: `interview_token` (str)  
- **Query Params**:  
  - `user_email` (str)  
  - `status` (str)  
- **Response**: `BaseResponseModel`

---

### GET `/api/hr/dashboard`
Returns high-level statistics for the HR dashboard.

- **Auth Required**: Yes  
- **Response**: `BaseResponseModel`

---

### GET `/api/hr/interview/{interview_token}/participants`
Lists all candidates participating in the interview.

- **Auth Required**: Yes  
- **Path Param**: `interview_token` (str)  
- **Response**: `BaseResponseModel`

---

### GET `/api/hr/questions/basic/{interview_token}/`
Retrieves all basic (non-custom) questions defined for the interview.

- **Auth Required**: Yes  
- **Path Param**: `interview_token` (str)  
- **Response**: `BaseResponseModel`

---

### POST `/api/hr/interview/login/{interview_token}`
Candidate login to an interview (creates a session).

- **Path Param**: `interview_token` (str)  
- **Request Body**: `InterviewLoginRequest`  
- **Response**: `BaseResponseModel`

---

### POST `/api/hr/interview/{interview_token}/question/{index}/answer`
Uploads a video or text answer for a given question.

- **Path Params**: `interview_token` (str), `index` (int)  
- **Form Data**:  
  - `user_email` (str)  
  - `file` (UploadFile) *or* `json_data` (text)  
- **Response**: `BaseResponseModel`

---

### POST `/upload`
Enqueues a background job to process a video.

- **Query Param**: `video_id` (str)  
- **Response**: `{ "message": "Video processing started in background" }`

---

### GET `/api/hr/interview/{interview_token}/overall-score`
Returns the overall score for a candidate’s interview.

- **Auth Required**: Yes  
- **Path Param**: `interview_token` (str)  
- **Query Param**: `user_email` (str)  
- **Response**: `BaseResponseModel`

---

## Response Format

All responses use the custom `BaseResponseModel`:

```json
{
  "status": "success",
  "error": null,
  "code": 200,
  "data": { ... }
}
```

Or in case of failure:

```json
{
  "status": "error",
  "error": "Invalid credentials",
  "code": 401,
  "data": null
}
```