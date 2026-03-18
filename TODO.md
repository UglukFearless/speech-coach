# TODO List

## High Priority

- [ ] Define orchestrator solution structure (domain/application/infrastructure/api projects) and wire DI composition root.
- [ ] Implement REST session endpoints: `POST /api/v1/sessions`, `GET /api/v1/sessions`, `GET /api/v1/sessions/{id}`.
- [ ] Implement turn endpoint `POST /api/v1/sessions/{id}/turns` with multipart audio upload and response DTO contract.
- [ ] Define and implement orchestrator clients/contracts for separate STT, LLM, and TTS services.
- [ ] Implement STT service container and contract (audio -> transcript).
- [ ] Implement LLM service container and contract (transcript + context -> reply + improved phrase + score).
- [ ] Implement TTS service container and contract (reply text -> audio).
- [ ] Implement dialogue orchestration service in orchestrator (STT service -> LLM service -> TTS service -> persistence).
- [ ] Design PostgreSQL schema for sessions and turns, add first migration, and connect repositories.
- [ ] Build Vue 3 frontend skeleton with main dialogue page and history sidebar layout.
- [ ] Implement frontend recording/upload flow and turn result rendering (transcript, improved phrase, score, audio playback).
- [ ] Add docker-compose orchestration for frontend, orchestrator-api, stt-service, llm-service, tts-service, and postgres.

## Medium Priority

- [ ] Add frontend API client module with typed contracts for sessions and turns.
- [ ] Implement session replay view from history sidebar.
- [ ] Add backend validation for audio constraints (size/duration/format) with normalized error responses.
- [ ] Add contract tests for REST DTOs and endpoint payloads.
- [ ] Add integration tests for Ollama adapters behind interface boundaries.
- [ ] Add integration tests for orchestrator-to-service contracts (STT/LLM/TTS) and error mapping.
- [ ] Add retention cleanup job for old session data and temporary audio files.

## Low Priority / Future Enhancements

- [ ] Add alternate provider implementations (cloud STT/LLM/TTS) without domain changes.
- [ ] Introduce WebSocket streaming mode for near-real-time dialogue.
- [ ] Add advanced feedback dimensions (pronunciation, grammar, vocabulary subscores).
- [ ] Add authentication and user profiles when moving beyond personal mode.
- [ ] Add production deployment hardening and observability dashboards.

## In Progress

- [ ] Define internal service API artifacts (request/response schema drafts) when implementation starts.

## Completed

- [x] Locked MVP product direction: personal English dialogue trainer, multi-turn focus.
- [x] Locked technical direction: separate Vue 3 frontend, .NET backend, REST transport.
- [x] Locked provider baseline: local models via Ollama with provider-agnostic architecture.
- [x] Locked scoring decision: per-turn score as integer 0-100.
- [x] Locked history sidebar fields: datetime, one-line summary, average score, turns count.
- [x] Locked decomposition direction: .NET orchestrator + separate STT/LLM/TTS services.
- [x] Added ADR 0001 with accepted architecture and repository layout decisions.
- [x] Finalized documentation baseline (README, ARCHITECTURE, RULES, TODO) and removed placeholders.

---

Note: Keep this file aligned with actual implementation status. Move items between sections during development.
