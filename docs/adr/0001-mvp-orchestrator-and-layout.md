# ADR 0001: MVP Orchestrator, Service Split, and Source Layout

## Status

Accepted

## Date

2026-03-19

## Context

The project targets a local-first MVP for spoken English practice with a multi-turn conversation loop.
We need:
- clear backend orchestration,
- easy provider replacement,
- language flexibility for model-facing services,
- and a predictable repository layout for AI-assisted implementation.

## Decision

1. Use a .NET orchestrator service as the main backend API.
2. Keep STT, LLM, and TTS as separate services behind stable contracts.
3. Allow STT/LLM/TTS implementation language to vary when integration with model runtimes is simpler outside .NET.
4. Keep public API in orchestrator (`/api/v1`) and version internal service contracts independently.
5. Place all source code under `src/`.

## Source Layout Convention

Target repository layout:

```text
speech-coach/
├── src/
│   ├── frontend/              # Vue 3 application
│   ├── orchestrator/          # .NET API and domain/application/infrastructure layers
│   └── services/
│       ├── stt-service/       # STT service (language optional)
│       ├── llm-service/       # LLM service (language optional)
│       └── tts-service/       # TTS service (language optional)
├── docs/
│   └── adr/
├── README.md
├── ARCHITECTURE.md
├── RULES.md
└── TODO.md
```

## Consequences

Positive:
- Better separation of concerns and easier provider replacement.
- Independent evolution of AI-facing services.
- Cleaner repository navigation for humans and AI agents.

Trade-offs:
- More moving parts in local Docker Compose.
- Additional internal API contracts to maintain.

## Follow-up

1. Keep README and ARCHITECTURE aligned with this ADR.
2. Keep TODO tasks mapped to `src/` paths.
3. Add internal API contract artifacts when implementation starts.
