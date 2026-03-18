# Rules for Cursor

## Architecture & Design

- All service, agent, and adapter classes must have corresponding interfaces (e.g., `ISttService` for `SttService`). Models and controllers should not have interfaces.
- Code is written in C# 12 using the latest language features (primary constructors, required members, etc.).
- All dependencies are injected through constructors (constructor injection).
- Do not use global static utilities — everything should be dependency injection or part of the domain.
- When generating HTTP clients, use `IHttpClientFactory` and typed clients.
- Backend and frontend are separate deployable units. Backend must not depend on frontend runtime internals.
- Runtime provider implementations (Ollama or alternatives) must be hidden behind adapter interfaces.
- Domain/application layers must not reference concrete provider SDKs directly.
- The orchestrator service is implemented in .NET.
- STT/LLM/TTS services may use other languages when integration with model runtimes is significantly simpler.
- Cross-service contracts must remain stable regardless of service implementation language.

## API Design Rules

- Expose REST endpoints under versioned path prefix (for MVP: `/api/v1`).
- Request and response DTOs are explicit contracts and must not expose persistence entities directly.
- Each dialogue turn response must include a per-turn score as integer in range 0-100.
- Session history list contract must include: datetime, one-line summary, average score, turns count.
- Error responses must follow a normalized model with stable error code and trace id.
- Internal orchestrator-to-service APIs must be versioned independently from public APIs.

## Data & Persistence Rules

- Use PostgreSQL for persistent session/turn storage.
- Schema changes must be applied through tracked migrations.
- Never perform manual production-like schema drift changes outside migration pipeline.
- Retention policy is configuration-driven; do not hardcode retention days in business logic.
- Temporary audio files must be placed under `temp/` and cleaned up safely.

## Code Style & Documentation

- Comments and XML documentation must be in English only.
- Use meaningful names for classes, methods, and variables following C# naming conventions.
- Prefer explicit types over `var` when the type is not immediately obvious from the right-hand side.

## Logging & Error Handling

- Logging is performed through `ILogger<T>` from Microsoft.Extensions.Logging.
- Use appropriate log levels (Trace, Debug, Information, Warning, Error, Critical).
- Exceptions should be caught and logged with context before rethrowing or handling.
- Use custom exception types for domain-specific errors.
- Log provider name and operation for STT/LLM/TTS failures to simplify adapter debugging.
- Propagate correlation id across orchestrator and all downstream services.

## Async/Await

- Prefer async/await over synchronous I/O operations.
- Use `ConfigureAwait(false)` in library code when appropriate.
- Avoid `async void` — use `async Task` instead.
- Long-running inference or I/O operations must support cancellation tokens.

## Resource Management

- Implement `IDisposable` or `IAsyncDisposable` for classes that manage unmanaged resources.
- Use `using` statements or `await using` for disposable resources.
- Temporary audio files are created in the `temp/` subdirectory relative to the application's working directory.

## Configuration & Constants

- All string literals related to paths, URLs, or configuration keys must be extracted to constants or `appsettings.json`.
- Use `IOptions<T>` pattern for strongly-typed configuration.
- Never hardcode secrets or sensitive data — use configuration providers or secret management.
- Provider selection (STT/LLM/TTS) must be configurable without domain code changes.

## Validation

- Validate input parameters in public methods (use `ArgumentNullException`, `ArgumentException`, etc.).
- Use data annotations or FluentValidation for model validation.
- Validate uploaded audio constraints (format, duration, size) before provider calls.

## Testing (if applicable)

- Write unit tests for business logic.
- Use dependency injection to enable testability.
- Mock external dependencies in tests.
- Add contract tests for REST DTOs of sessions/turns/history.
- Add integration tests for adapter boundary behavior (at least one baseline path with Ollama).
- Maintain one E2E smoke test for a 5+ turn dialogue scenario.

## MVP Acceptance Criteria

- User can start a session and complete at least 5 turns via REST API.
- Each turn returns transcript, improved phrase, assistant text reply, synthesized audio reference, and per-turn score.
- Session history endpoint returns required sidebar fields.
- Provider can be switched via configuration with no domain-layer refactor.
