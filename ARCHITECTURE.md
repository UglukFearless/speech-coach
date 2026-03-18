# Architecture & Design Decisions

## Overview

Speech Coach MVP is a local-first system for spoken English practice through a multi-turn dialogue with a native-speaker persona.

Per-turn workflow:
1. User sends an audio phrase from the web UI.
2. Orchestrator forwards audio to the STT service.
3. Orchestrator sends transcript and dialogue context to the LLM service.
4. LLM service returns assistant reply text, improved user phrase, and per-turn score.
5. Orchestrator sends assistant reply text to the TTS service.
6. TTS service returns synthesized audio reference.
7. Orchestrator persists the turn and returns the full turn result to the frontend.

System boundaries:
- Frontend (Vue 3): UI, recording/upload, playback, history rendering.
- Orchestrator (required .NET 8 service): session lifecycle, orchestration, contracts, persistence.
- STT service: speech-to-text processing.
- LLM service: response generation and scoring.
- TTS service: speech synthesis.
- Database: PostgreSQL for session and turn history.

AI services are intentionally language-agnostic at implementation level. They may be implemented in .NET or another language if it improves integration ergonomics with selected model runtimes.

## Architecture Patterns

### Primary patterns
- Clean Architecture
- Onion Architecture

### Core layers (inside orchestrator)
- Presentation layer: REST controllers and DTO contracts.
- Application/domain layer: dialogue orchestration, business rules, scoring model.
- Infrastructure layer: provider adapters (STT/LLM/TTS), repositories, database access.

### Dependency Injection
- All dependencies are injected through constructors.
- .NET built-in DI container is used in the orchestrator.

### Domain Layer
- Services use interfaces for testability.
- Domain services are separated from infrastructure services.
- Provider abstraction is mandatory: domain logic cannot depend on concrete STT/LLM/TTS implementations.

## Deployment Units

Minimum MVP containers:
- `frontend` (Vue 3)
- `orchestrator-api` (.NET 8)
- `stt-service` (implementation language optional)
- `llm-service` (implementation language optional)
- `tts-service` (implementation language optional)
- `postgres`

All units are composed through Docker Compose in local development.

## Input/Output Formats

### Audio Input
- **Format**: WAV (preferred), WebM/Opus (allowed from browser recording)
- **Sample Rate**: 16 kHz or 24 kHz
- **Channels**: mono preferred
- **Bit Depth**: 16-bit PCM for WAV
- **Max duration per turn (MVP default)**: configurable, recommended 20 seconds

### Audio Output
- **Temporary Storage**: `temp/` directory relative to working directory
- **Format**: WAV (MVP default)
- **Cleanup**: remove temporary files after successful turn persistence and response delivery; periodic cleanup job for leftovers

### API Contracts

Base path:
- `/api/v1`

Endpoints:

1. `POST /sessions`
- Purpose: start a new dialogue session
- Request:
```json
{
	"targetLanguage": "en",
	"sourceLanguage": "ru",
	"topic": "small-talk"
}
```
- Response:
```json
{
	"sessionId": "uuid",
	"createdAt": "2026-03-18T10:00:00Z"
}
```

2. `POST /sessions/{sessionId}/turns`
- Purpose: submit user audio for the next dialogue turn
- Content-Type: `multipart/form-data`
- Form fields:
	- `audio` (file, wav or webm/opus)
	- `clientTurnIndex` (int)
- Response:
```json
{
	"turnId": "uuid",
	"turnIndex": 3,
	"userTranscript": "I go to office yesterday",
	"improvedPhrase": "I went to the office yesterday",
	"assistantReplyText": "Nice. What did you do there?",
	"score": 72,
	"assistantReplyAudioUrl": "/api/v1/media/tts/turn-uuid.wav",
	"createdAt": "2026-03-18T10:01:20Z"
}
```

3. `GET /sessions`
- Purpose: history sidebar list
- Query params: `limit`, `offset`
- Response:
```json
{
	"items": [
		{
			"sessionId": "uuid",
			"startedAt": "2026-03-18T10:00:00Z",
			"summary": "Travel plans and daily routine",
			"averageScore": 76,
			"turnsCount": 8
		}
	],
	"total": 1
}
```

4. `GET /sessions/{sessionId}`
- Purpose: session details with turns
- Response includes all turns, transcripts, improved phrases, scores, and assistant replies.

Common error model:
```json
{
	"code": "stt_processing_failed",
	"message": "Failed to transcribe audio",
	"traceId": "..."
}
```

Score contract:
- Integer in range 0-100 for each turn.

Internal service contracts (orchestrator -> AI services):
- STT service: accepts audio payload, returns transcript text and confidence metadata.
- LLM service: accepts transcript + session context, returns assistant reply, improved phrase, and score.
- TTS service: accepts assistant reply text, returns audio binary or media reference.
- Internal contracts should be versioned independently from public API.

## Key Components

### Services
- **Orchestrator API**: public REST API, persistence, and cross-service orchestration.
- **STT Service**: speech-to-text conversion.
- **LLM Service**: assistant reply + improved phrase + score generation.
- **TTS Service**: assistant reply speech synthesis.
- **Session Service (inside orchestrator)**: session lifecycle and history queries.

### Adapters
- `ISttProviderAdapter`
- `ILlmProviderAdapter`
- `ITtsProviderAdapter`
- `Ollama*Adapter` implementations as MVP baseline
- Remote service clients (`ISttClient`, `ILlmClient`, `ITtsClient`) for orchestrator-to-service calls

### Agents
- Not used in runtime architecture for MVP.

## Data Flow

```
Vue 3 UI
	-> POST /api/v1/sessions (start)
	-> POST /api/v1/sessions/{id}/turns (audio)
		 -> STT service (local model backend)
		 -> LLM service (local model backend)
		 -> TTS service (local model backend)
		 -> Save turn in PostgreSQL
	<- turn result (text + score + audio URL)
	-> GET /api/v1/sessions (history sidebar)
	-> GET /api/v1/sessions/{id} (session replay)
```

## Technology Stack

- **Framework**: .NET 8.0
- **Orchestrator Language**: C# 12
- **Frontend**: Vue 3
- **Logging**: Microsoft.Extensions.Logging
- **HTTP Client**: IHttpClientFactory
- **Database**: PostgreSQL
- **Containerization**: Docker Compose
- **AI Services Language**: Polyglot allowed
- **Local AI baseline**: Ollama

## Configuration

- Configuration stored in `appsettings.json`
- Environment-specific settings via `appsettings.{Environment}.json`
- Secrets/keys via environment variables in local setup
- Service endpoint routing via config keys (example):
	- `Services:Stt:BaseUrl=http://stt-service:8080`
	- `Services:Llm:BaseUrl=http://llm-service:8080`
	- `Services:Tts:BaseUrl=http://tts-service:8080`
- Optional provider selection behind each service via config (example):
	- `Providers:Stt=Ollama`
	- `Providers:Llm=Ollama`
	- `Providers:Tts=Ollama`
- Retention settings are configurable (example):
	- `DataRetention:Days=14`

## Error Handling Strategy

- Use domain-specific exception types for STT/LLM/TTS and persistence failures.
- Return normalized API error payload with `code`, `message`, and `traceId`.
- Log all external service failures with context, service name, and operation.
- For recoverable provider errors, support fallback adapter chain in future versions.

## Performance Considerations

- Keep all provider calls asynchronous.
- Stream file uploads to avoid loading full audio into memory when possible.
- Start with correctness and observability first; strict latency optimization is out of MVP scope.
- Add correlation ID propagation across orchestrator, STT, LLM, and TTS services.

## Persistence Model (MVP)

Core entities:
- `Session`: id, started_at, source_language, target_language, topic, summary, average_score
- `Turn`: id, session_id, turn_index, user_transcript, improved_phrase, assistant_reply_text, score, assistant_reply_audio_path, created_at

Retention:
- Retention window is configurable and can be changed without domain logic changes.
- For current personal mode, any practical value is acceptable.
