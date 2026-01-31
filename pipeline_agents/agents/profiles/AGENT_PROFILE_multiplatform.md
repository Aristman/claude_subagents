# Agent Profile: Multiplatform

## Profile Identity

- Profile ID: multiplatform
- Domain: multiplatform
- Scope: shared business logic and cross-platform codebases
- Platforms: Android, iOS, Web, Desktop (as defined by architecture)
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `multiplatform` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_multiplatform.md

---

## Supported Technologies

The agent MUST use one of the following multiplatform stacks
(as determined by architecture and repository context):

- Kotlin Multiplatform (KMP)
- Flutter (Dart)
- React Native (TypeScript)
- Shared core libraries (language depends on target platforms)

The agent MUST NOT introduce a new multiplatform technology
without architectural approval.

---

## Architectural Intent

The primary goal of a multiplatform codebase is:

- maximize shared logic
- minimize platform-specific divergence
- keep platform adapters thin and explicit

Shared code MUST contain:

- business logic
- domain models
- validation
- non-UI workflows

Platform-specific code MUST contain:

- UI
- platform APIs
- lifecycle handling

---

## Code Organization Rules

- Shared code MUST be isolated from platform code
- Platform adapters MUST depend on shared code
- Shared code MUST NOT depend on platform APIs

Forbidden practices:

- platform-specific conditionals inside shared core
- leaking UI concepts into shared modules
- hidden platform dependencies

---

## State and Data Management

- State in shared code MUST be explicit and deterministic
- Side effects MUST be abstracted behind interfaces
- Platform implementations MUST be injected

The agent MUST NOT:

- perform IO directly in shared business logic
- rely on global mutable state
- introduce non-deterministic behavior

---

## Concurrency and Async Rules

- Concurrency primitives MUST be appropriate for the chosen stack
- Async boundaries MUST be explicit
- Shared code MUST remain thread-safe

The agent MUST:

- avoid blocking calls in shared code
- document threading assumptions

---

## API and Contract Rules

- Shared APIs MUST be stable and versioned if exposed
- Data models MUST be platform-agnostic
- Serialization formats MUST be explicitly defined

The agent MUST NOT:

- expose platform-specific types in shared APIs
- change contracts without architectural alignment

---

## Error Handling Rules

- Errors in shared code MUST be explicit and typed
- Platform layers MUST map errors to user-facing behavior
- Silent failures are forbidden

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Shared unit tests
    - domain logic
    - validation rules
    - business workflows

2. Shared integration tests
    - use mocked platform adapters

Optional (if defined in roadmap or architecture):

- Platform-specific integration tests
- End-to-end tests per platform

The agent MUST NOT:

- rely solely on platform-level manual testing
- skip shared logic tests

---

## Non-Functional Priorities

The agent MUST prioritize:

1. Correctness
2. Consistency across platforms
3. Maintainability
4. Performance of shared logic
5. Testability

The agent SHOULD:

- minimize platform branching
- keep shared code simple and portable

---

## Performance Constraints

The agent MUST:

- avoid unnecessary abstractions in hot paths
- minimize serialization overhead
- avoid excessive data copying between layers

Premature optimization is discouraged.

---

## Security Constraints

The agent MUST:

- treat all external input as untrusted
- avoid leaking sensitive data across platform boundaries
- delegate secure storage to platform-specific layers

The agent MUST NOT:

- store secrets in shared code
- bypass platform security mechanisms

---

## Tooling and Code Quality

- Code MUST follow stack-specific style guides
- Shared code MUST be well-documented
- Dead or unused code MUST NOT be introduced

The agent SHOULD:

- favor explicitness over clever abstractions
- keep APIs small and intention-revealing

---

## Forbidden Practices (Strict)

The agent MUST NOT:

- mix platform-specific logic into shared core
- introduce hidden coupling between platforms
- “fix” unrelated platform code opportunistically
- violate architectural boundaries

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
