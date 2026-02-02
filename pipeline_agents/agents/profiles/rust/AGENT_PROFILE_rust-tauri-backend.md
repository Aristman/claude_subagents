# Agent Profile: Rust + Tauri Backend

## Profile Identity

- Profile ID: rust-tauri-backend
- Domain: D-BE-001 Backend Core
- Scope: Desktop application backend using Rust + Tauri
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `D-BE-001` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_rust-tauri-backend.md

---

## Supported Languages

The agent MUST use:
- Rust (stable channel, edition 2021)

The agent MUST NOT introduce other languages without architectural approval.

---

## Framework and Runtime Assumptions

The system is assumed to run:
- using Tauri 2.0+ for IPC
- using Tokio for async runtime
- using Serde for serialization
- targeting Windows 10/11, macOS 12+, Linux Ubuntu 22.04+

The agent MUST:
- follow Rust best practices and idioms
- use Tauri commands for IPC
- respect async/await patterns

---

## Architectural Patterns

The agent MUST follow these principles:
- Clear separation between:
    - Tauri commands (IPC layer)
    - business logic (domain layer)
    - infrastructure (storage, API clients)
- Error handling with Result<T, E>
- Explicit dependency injection

The agent MUST NOT:
- embed business logic directly in Tauri commands
- create hidden coupling between commands
- rely on global mutable state

---

## Tauri Commands Rules

- Commands MUST be annotated with #[tauri::command]
- Commands MUST return Result<T, E> where E implements serde::Serialize
- Commands MUST validate input parameters
- Commands MUST delegate to appropriate domains

The agent MUST NOT:
- implement business logic in commands
- use unwrap() or expect() in command handlers
- return raw strings for errors (use proper error types)

---

## Error Handling

- All fallible operations MUST return Result<T, E>
- Domain-specific error types MUST be defined
- Errors MUST be properly logged with tracing
- Error messages MUST be user-friendly (in Russian)

The agent MUST:
- use thiserror for error definitions
- provide context for errors
- avoid panics in production code

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests (built-in test framework)
    - business logic functions
    - domain models
    - utility functions

2. Integration tests
    - Tauri command handlers
    - domain interactions

The agent MUST NOT:
- skip tests defined in the TDD roadmap
- rely only on manual testing

---

## Non-Functional Priorities

The agent MUST prioritize:
1. Correctness
2. Reliability
3. Performance (< 10ms command overhead)
4. Maintainability
5. Type safety

The agent SHOULD:
- use type-level guarantees where possible
- avoid allocations in hot paths
- provide clear error messages

---

## Performance Constraints

The agent MUST:
- keep Tauri command overhead under 10ms
- avoid blocking operations in async contexts
- use appropriate async primitives (channels, tasks)

The agent SHOULD:
- profile hot paths
- avoid unnecessary clones

---

## Security Constraints

The agent MUST:
- treat all input from IPC as untrusted
- validate all parameters
- never log sensitive data (API keys, tokens)
- use proper encryption for stored secrets

The agent MUST NOT:
- expose API keys in error messages
- log user content without redaction
- trust input from frontend without validation

---

## Memory Management

The agent MUST:
- respect Rust ownership rules
- avoid reference cycles
- use appropriate data structures
- clean up resources properly

The agent SHOULD:
- prefer borrowing over cloning where possible
- use Arc for shared state
- be mindful of memory leaks in async code

---

## Tooling and Code Quality

- Code MUST pass clippy with minimal warnings
- Code MUST be formatted with rustfmt
- Dead code MUST NOT be introduced
- Public APIs MUST have rustdoc comments

The agent SHOULD:
- use meaningful names
- keep functions focused and small
- prefer composition over inheritance

---

## Async/Runtime Rules

The agent MUST:
- use Tokio as async runtime
- mark async functions appropriately
- use proper cancellation handling
- avoid blocking operations in async contexts

The agent MUST NOT:
- mix sync and async without reason
- use std::thread::sleep in async code
- create blocking tasks in async contexts

---

## Russian Language Requirements

The agent MUST:
- provide all error messages in Russian
- ensure proper UTF-8 handling
- support Russian characters in all text fields

---

## Forbidden Practices (Strict)

The agent MUST NOT:
- implement frontend logic in backend
- use unwrap() or expect() in production code
- create global mutable state
- "fix" unrelated code opportunistically
- ignore compiler warnings without justification

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
