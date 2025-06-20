# Intilaq AI - Overview

## What is Intilaq AI?

**Intilaq AI** is a full-stack platform that empowers job seekers by offering realistic interview simulations, instant AI-based feedback, and smart CV building tools. It uses advanced AI models for natural language understanding and voice processing to help users improve their job readiness.

The system is built with **Clean Architecture**, **Dependency Injection (DI)**, and follows a **modular monolithic design** that enables easy maintenance, scalability, and testing.


---

##  Target Audience

- Job seekers looking to improve their interview performance.
-  University students and fresh graduates building professional resumes.
-  HR teams conducting remote, structured interviews and evaluations.

---

## Architecture Overview

Intilaq AI follows a **modular monolithic architecture** internally, while leveraging **external microservices** for AI-specific processing.

### Key Properties

- **Clean Architecture** with clear separation of logic and infrastructure.
- **Domain-based service layers** (auth_services, interview_services, cv_services, etc.)
- **Thin API routes** – logic is handled entirely within service classes.
- **Dependency Injection (DI)** via provider modules for full testability.
- **RQ Worker system** for offloading time-consuming tasks like audio extraction and AI scoring.
- **Microservices** for AI models:
  - **Whisper Service**: Speech-to-text transcription (runs separately, accessed via HTTP)
  - **Gemini Service**: Text feedback and scoring (accessed via external HTTP API)
- **Single repository** containing all domain logic, routes, services, workers, and tests.
- **Docker Compose** used to orchestrate all services into a unified network.

> This hybrid approach (modular monolith + external microservices) ensures fast local development and high scalability for resource-intensive AI tasks.

##  How to Run the System?

### 1. Using Docker (Recommended)
Each component runs in its own container:
- FastAPI (backend API)
- MongoDB + PostgreSQL
- Redis (session and token store)
- Whisper microservice (speech-to-text)
- RQ Worker (background jobs)
- NGINX (reverse proxy)
- Tailscale for secure HTTPS tunneling

### 2. Without Docker (Development Mode)
- Manually run services:
  - `uvicorn` for FastAPI
  - `rq worker` for background jobs
  - Local installations for databases
- Only recommended for local testing and debugging

---

##  Key Features

| Feature | Description |
|--------|-------------|
|  Interview Simulation | Full AI-powered job interview experience with voice/text answers |
|  AI Feedback | Real-time scoring and personalized feedback via Whisper + Gemini |
|  CV Builder | Intelligent CV generation with AI-enhanced content and export (PDF/HTML/DOCX) |
|  HR Interview Panel | Create custom interviews, assign to candidates, and review responses |
|  Background Jobs | Handle heavy tasks like video compression and transcription with RQ |
|  Monitoring Dashboard | Track performance using Prometheus and Grafana |

---

## Tech Stack

| Layer | Technologies |
|-------|--------------|
| Backend | FastAPI, Python  |
| Database | PostgreSQL (structured data), MongoDB (sessions), Redis (caching & tokens) |
| AI Integration | Whisper (speech-to-text), Gemini (text analysis) |
| Workers | RQ (Redis Queue) |
| DevOps | Docker, Docker Compose, NGINX, Tailscale|

---
