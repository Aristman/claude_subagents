# Agent Profile: Rust API Integration

## Profile Identity

- Profile ID: rust-api-integration
- Domain: D-AI-001 AI Integration
- Scope: External API clients using Rust (LLM, Image Generation)
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `D-AI-001` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_rust-api-integration.md

---

## Supported Languages

The agent MUST use:
- Rust (stable channel, edition 2021)

The agent MUST NOT introduce other languages without architectural approval.

---

## Framework and Runtime Assumptions

The system is assumed to run:
- using reqwest or similar for HTTP clients
- using Tokio for async runtime
- using Serde for JSON serialization
- integrating with LLM APIs (OpenAI, Anthropic, etc.)
- integrating with Image Generation APIs (DALL-E, etc.)

The agent MUST:
- follow Rust async best practices
- respect API rate limits
- handle network errors gracefully

---

## Architectural Patterns

The agent MUST follow these principles:
- Clear separation between:
    - API client layer (HTTP operations)
    - domain models (request/response types)
    - error handling (API-specific errors)
- Provider abstraction (support multiple providers)
- Streaming support for LLM responses

The agent MUST NOT:
- embed API keys in code
- create tight coupling to specific providers
- ignore rate limiting

---

## API Client Rules

- Clients MUST use async/await
- Clients MUST respect rate limits
- Clients MUST handle timeouts appropriately
- Clients MUST support streaming (for LLM APIs)
- API keys MUST be passed as parameters (not stored in client)

The agent MUST NOT:
- hardcode API endpoints
- ignore API errors
- retry indefinitely without backoff

---

## Error Handling

- All API operations MUST return Result<T, E>
- API-specific error types MUST be defined
- Errors MUST distinguish between:
    - network errors
    - API errors
    - rate limit errors
    - authentication errors

The agent MUST:
- use thiserror for error definitions
- implement exponential backoff for retries
- log API calls (without sensitive data)

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests
    - request/response serialization
    - error parsing
    - utility functions

2. Integration tests (with mocks)
    - API client behavior
    - error handling
    - retry logic

The agent MUST NOT:
- skip tests defined in the TDD roadmap
- make real API calls in unit tests

---

## Non-Functional Priorities

The agent MUST prioritize:
1. Reliability (handle network failures)
2. Correctness (proper data parsing)
3. Performance (streaming, connection pooling)
4. Security (no leaked credentials)
5. Maintainability (provider abstraction)

The agent SHOULD:
- use connection pooling
- implement proper timeouts
- respect API quotas

---

## Performance Constraints

The agent MUST:
- use async operations for all network calls
- implement appropriate timeouts (30-120 seconds)
- use streaming for LLM responses when available
- avoid unnecessary allocations

The agent SHOULD:
- reuse connections when possible
- implement request batching when appropriate

---

## Security Constraints

The agent MUST:
- never log API keys or tokens
- never expose API keys in error messages
- validate all input before sending to APIs
- use HTTPS for all API calls

The agent MUST NOT:
- hardcode API credentials
- store API keys in the API client
- send secrets in query parameters

---

## LLM API Specific Rules

- Support streaming responses when available
- Handle chunked responses appropriately
- Parse structured outputs (JSON, markdown)
- Support different models (GPT-4, Claude, etc.)

The agent SHOULD:
- implement progress callbacks
- handle incomplete responses gracefully

---

## Image Generation API Specific Rules

- Handle async generation (polling if needed)
- Support different image sizes/types
- Save images to file system (not in memory)
- Store metadata (prompt, model, parameters)

The agent MUST:
- validate image URLs before returning
- handle download failures
- clean up temporary files

---

## Rate Limiting

The agent MUST:
- respect API rate limits
- implement exponential backoff
- handle 429 (Too Many Requests) responses
- provide rate limit status to caller

The agent SHOULD:
- implement request queuing if appropriate
- warn user about rate limit approach

---

## Observability

The agent MUST:
- log all API calls (without credentials)
- log request duration
- log token usage (when available)
- log errors with context

The agent MUST NOT:
- log API keys or tokens
- log full request/response bodies (may contain sensitive data)

---

## Russian Language Requirements

The agent MUST:
- handle UTF-8 correctly for Russian text
- support Unicode in prompts and responses
- ensure proper encoding for API calls

---

## Provider Abstraction

The agent SHOULD:
- define common traits for API clients
- support multiple providers (OpenAI, Anthropic, etc.)
- allow provider selection at runtime
- hide provider-specific details

---

## Forbidden Practices (Strict)

The agent MUST NOT:
- hardcode API keys or tokens
- ignore rate limits
- retry indefinitely without backoff
- log sensitive data
- create provider-tight coupling without abstraction

---

## Deviation Policy

If deviation from this profile is unavoidable:
- the deviation MUST be explicitly documented
- the reason MUST be technical, not convenience-based
- the deviation will be reviewed and verified

Undocumented deviations are considered defects.

---

## Profile Authority

This profile:
- constrains implementation behavior
- overrides agent preferences
- is subordinate only to:
    - ARCHITECTURE_OVERVIEW.md
    - explicit feature roadmap instructions

Failure to comply with this profile is grounds for rejection
during review or verification.
