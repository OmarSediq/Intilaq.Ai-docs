# Intilaq AI - Overview

## What is Intilaq AI?

**Intilaq AI** is a full-stack platform that empowers job seekers by offering realistic interview simulations, instant AI-based feedback, and smart CV building tools.

The platform utilizes advanced AI models for natural language understanding and voice transcription to improve job readiness.

It is designed using **Clean Architecture**, **Dependency Injection (DI)**, and a **modular monolithic structure** that ensures maintainability, scalability, and testing efficiency.

---

## Target Audience

Intilaq AI is designed for:

- Individuals preparing for job interviews.
- University students and recent graduates building professional CVs.
- HR teams conducting structured, remote interview evaluations.

---

## Architecture Overview

The architecture follows a **modular monolithic** structure combined with **external microservices** for heavy AI processing.

Key architectural features include:

- Clean separation of concerns between logic and infrastructure.
- Domain-specific service layers (e.g., `auth_services`, `cv_services`, `interview_services`).
- API routes are kept minimal, delegating business logic to service classes.
- Dependency Injection (DI) is applied to all core services via provider modules.
- Background job processing is handled by RQ Worker for tasks such as:
  - Audio extraction
  - Whisper transcription
  - Gemini feedback scoring
  - Updating participant summaries
- Microservices include:
  - **Whisper Service** for speech-to-text transcription.
  - **Gemini Service** for feedback and similarity scoring.

All components run together in a single repository using Docker Compose to orchestrate services into one network.

> This hybrid model combines modular monolith benefits for local development and microservice flexibility for AI processing.

---

## How to Run the System

### 1. Using Docker (Recommended)

To run the platform with Docker Compose:

- FastAPI backend container
- MongoDB and PostgreSQL containers
- Redis for caching and token storage
- Whisper microservice for transcription
- RQ Worker container for background tasks
- NGINX for HTTPS reverse proxying
- Tailscale for secure private domain access

### 2. Without Docker (For Development)

You can run components manually:

- Use `uvicorn` to run FastAPI
- Use `rq worker` to launch job workers
- Start local Redis, PostgreSQL, and MongoDB instances manually

This method is best for isolated debugging, not production.

---

## Key Features

Some of the main features offered by Intilaq AI include:

- **Interview Simulation**: Simulate a complete interview with voice and text responses.
- **AI Feedback**: Get real-time scoring and feedback using Whisper and Gemini.
- **CV Builder**: Build professional CVs with smart suggestions and export to PDF, HTML, or DOCX.
- **HR Interview System**: Allow HR teams to create interviews, send invitations, and review answers.
- **Background Processing**: Offload time-consuming tasks like video processing to RQ workers.
- **Performance Monitoring**: Track metrics using integrated Prometheus and Grafana dashboards.

---

## Technology Stack

The platform is built with the following tools:

- **Backend**: FastAPI, Python
- **Databases**: PostgreSQL (structured), MongoDB (sessions), Redis (tokens, cache)
- **AI Services**: Whisper for transcription, Gemini for analysis
- **Workers**: RQ + Redis
- **DevOps**: Docker, Docker Compose, NGINX, Tailscale

---
