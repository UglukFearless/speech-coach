# Speech Coach

## Overview

Speech Coach is a personal English speaking trainer.

The application simulates a multi-turn dialogue with a native speaker persona and helps the user improve phrasing step by step.
Core loop: user speaks -> STT transcription -> LLM reply and correction -> per-turn score -> TTS voice response.

This repository currently focuses on a local-first MVP:
- Frontend: Vue 3 (separate app)
- Backend orchestrator: .NET 8 Web API (separate app)
- AI services: separate STT, LLM, and TTS services (polyglot allowed)
- Transport: REST
- AI runtime baseline: local models via Ollama
- Storage: PostgreSQL

## Features

- Audio recording/upload from browser
- Speech-to-text transcription
- Multi-turn dialogue (target: 5+ turns per session)
- Native-speaker style AI response
- Improved phrase suggestion for each user turn
- Per-turn score (0-100 integer)
- Text-to-speech voice response for model reply
- Session history in sidebar (datetime, one-line summary, average score, turns count)

## MVP Scope

Included in MVP:
- Single-user local setup
- English speaking practice dialogues
- Separate frontend, backend orchestrator, and AI services
- Session persistence in PostgreSQL

Out of scope for MVP:
- Registration/authentication
- Multi-tenant access
- Production hardening and enterprise security features
- Strict latency SLA

## Getting Started

### Prerequisites

- .NET 8.0 or later
- Node.js 20+
- Docker + Docker Compose
- Ollama installed and running locally

### Installation

```bash
# 1) Clone repository
git clone <repo-url>
cd speech-coach

# 2) Prepare environment variables for backend and frontend
# (details will be defined in app-specific env files)

# 3) Start infrastructure/services
docker compose up --build
```

### Usage

```bash
# 1) Open frontend in browser
# 2) Start a new dialogue session
# 3) Record or upload a phrase in English
# 4) Receive model reply + improved phrase + per-turn score + TTS audio
# 5) Continue conversation for multiple turns
# 6) Open history sidebar to review saved sessions
```

## Primary User Flow

1. User opens the main page.
2. User starts a dialogue session.
3. User sends voice input.
4. Backend processes STT -> LLM -> TTS and returns the turn result.
5. Frontend plays reply audio and displays correction + score.
6. Steps 3-5 repeat for 5+ turns.
7. Session appears in history sidebar and can be reopened.

## Project Structure

```
speech-coach/
├── frontend/         # Vue 3 application
├── backend/          # .NET 8 orchestrator API + domain services
├── services/         # STT/LLM/TTS services (language-agnostic)
├── docker-compose.yml
├── temp/             # Temporary audio files
└── docs or *.md      # Product/architecture/process documentation
```

## Contributing

Before implementation changes:
1. Keep architecture provider-agnostic.
2. Preserve clear separation between frontend and backend.
3. Follow engineering rules from RULES.md.
4. Keep TODO.md aligned with actual progress.

## License

TBD
