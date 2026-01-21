# Agent Profile: Backend

## Profile Identity

- Profile ID: backend
- Domain: backend
- Scope: server-side application and services
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `backend` domain.

---

## Supported Languages

The agent MUST use one of the following languages
(as determined by architecture and repository context):

- Rust
- TypeScript
- Go
- Python

The agent MUST NOT introduce a new language without architectural approval.

---

## Framework and Runtime Assumptions

- The system is assumed to be:
    - networked
    - request-driven (API, events, jobs)
    - running in a server or container environment

Framework choice is constrained by:

- existing codebase
- architectural overview
- repository conventions

The agent MUST follow existing framework conventions.

---

## Architectural Patterns

The agent MUST follow these patterns where applicable:

- Clear separation of concerns
- Explicit boundaries between:
    - transport layer
    - business logic
    - data access
- Dependency inversion at module boundaries

The agent MUST NOT:

- mix business logic with transport concerns
- introduce hidden cross-module coupling
- rely on global mutable state

---

## Data and State Management

- All persistent state MUST be explicit
- Database access MUST be isolated
- Transactions MUST be explicit and bounded
- Side effects MUST be observable

Forbidden practices:

- implicit state changes
- hidden shared mutable state
- non-deterministic behavior without justification

---

## Error Handling Rules

- Errors MUST be explicit
- Errors MUST be propagated or handled intentionally
- Silent failure is forbidden

The agent MUST:

- return meaningful error information
- avoid catch-all error suppression

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests according to this order:

1. Unit tests
    - for business logic
    - fast and deterministic

2. Integration tests
    - for database, external services, IO
    - isolated and reproducible

Optional (if defined in roadmap or architecture):

- Contract tests
- API tests

The agent MUST NOT:

- skip tests defined in the roadmap
- rely only on manual testing

---

## Non-Functional Priorities

The agent MUST prioritize:

1. Correctness
2. Reliability
3. Observability
4. Performance (within reasonable bounds)

The agent MUST:

- add logging where failures may occur
- expose meaningful metrics if required
- avoid premature optimization

---

## Security and Safety Constraints

The agent MUST:

- validate external input
- avoid unsafe deserialization
- avoid hard-coded secrets
- respect authentication and authorization boundaries

The agent MUST NOT:

- bypass security mechanisms
- introduce insecure defaults

---

## Tooling and Code Quality

- Code MUST be formatted according to project standards
- Linting rules MUST be respected
- Dead code MUST NOT be introduced

The agent SHOULD:

- prefer clarity over cleverness
- write readable, maintainable code

---

## Forbidden Practices (Strict)

The agent MUST NOT:

- introduce new dependencies without justification
- introduce blocking IO in async contexts
- violate architectural boundaries
- implement functionality outside the feature scope
- “fix” unrelated code opportunistically

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
    - explicit roadmap instructions

Failure to comply with this profile is grounds for rejection
during review or verification.
