# Agent Profile: Backend (Base)

## Profile Identity

- Profile ID: backend-base
- Domain: backend
- Scope: server-side applications and services
- Type: Base profile (inherits to specialized profiles)
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile defines **universal backend rules** that apply to ALL backend stacks.
Specialized profiles (spring-boot, nodejs, python, rust) extend this profile.

---

## Architectural Patterns (MANDATORY)

The agent MUST follow these patterns regardless of technology:

### Layered Architecture

```
Transport Layer (Controllers/Handlers)
         ↓
Business Logic Layer (Services/Use Cases)
         ↓
Data Access Layer (Repositories/DAOs)
```

### Separation of Concerns

The agent MUST:

- Keep transport concerns separate from business logic
- Isolate data access behind repository abstraction
- Use dependency inversion at layer boundaries
- Make side effects explicit and observable

The agent MUST NOT:

- Put business logic in controllers/handlers
- Mix database queries with business rules
- Introduce hidden cross-module coupling
- Rely on global mutable state

---

## Data and State Management

The agent MUST:

- Make all persistent state explicit
- Isolate database access behind repositories
- Use explicit transactions for write operations
- Ensure operations are deterministic or explicitly non-deterministic

Forbidden practices:

- Implicit state changes
- Hidden shared mutable state
- Unbounded resource retention
- Silent failures in state mutations

---

## Error Handling Rules

The agent MUST:

- Make errors explicit (no silent failures)
- Propagate or handle errors intentionally
- Return meaningful error information
- Distinguish between recoverable and non-recoverable errors

The agent MUST NOT:

- Use catch-all error suppression without logging
- Return error codes as successful responses
- Expose internal implementation details in errors
- Lose error context during propagation

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in this order:

### 1. Unit Tests
- Business logic in isolation
- Fast and deterministic
- Mock external dependencies

### 2. Integration Tests
- Database operations
- External service integration
- API contract validation
- Reproducible with test containers/fixtures

### Optional (if defined in roadmap):

- Contract tests (API consumers)
- Load tests (performance critical paths)
- Chaos tests (distributed systems)

The agent MUST NOT:

- Skip tests defined in the roadmap
- Rely only on manual testing
- Write flaky/non-deterministic tests

---

## Non-Functional Priorities

The agent MUST prioritize in this order:

1. **Correctness** — system behaves as specified
2. **Reliability** — system handles failures gracefully
3. **Observability** — system state is inspectable
4. **Performance** — within reasonable bounds

The agent MUST:

- Add logging where failures may occur
- Expose meaningful metrics if required
- Avoid premature optimization
- Measure before optimizing

---

## Security and Safety Constraints

The agent MUST:

- Validate ALL external input
- Sanitize data for storage/output
- Avoid unsafe deserialization
- Never hard-code secrets
- Respect authentication/authorization boundaries
- Use parameterized queries for database
- Implement rate limiting for public APIs

The agent MUST NOT:

- Bypass security mechanisms
- Introduce insecure defaults
- Trust client-side validation
- Log sensitive data (passwords, tokens, PII)

---

## API Design Principles

For REST APIs:

- Use appropriate HTTP verbs (GET, POST, PUT, DELETE, PATCH)
- Return correct status codes (2xx, 3xx, 4xx, 5xx)
- Version APIs explicitly (/v1/, /v2/)
- Use pagination for list responses
- Implement idempotency where applicable
- Use standardized error response format

For GraphQL:

- Type everything explicitly
- Use mutations for writes, queries for reads
- Implement query complexity analysis
- Rate limit at query level

---

## Database Guidelines

The agent MUST:

- Use transactions for multi-step operations
- Define indexes for query performance
- Handle connection pooling appropriately
- Use migrations for schema changes
- Consider foreign keys for referential integrity

The agent MUST NOT:

- Use N+1 queries
- Forget to close connections/resources
- Put business logic in database
- Skip migration testing

---

## Tooling and Code Quality

The agent MUST:

- Follow project formatting standards
- Respect linting rules
- Remove dead code
- Write self-documenting code

The agent SHOULD:

- Prefer clarity over cleverness
- Use meaningful names
- Keep functions focused (single responsibility)
- Maintain low cyclomatic complexity

---

## Observability

The agent MUST ensure:

- Structured logging (JSON if possible)
- Log levels used correctly (ERROR, WARN, INFO, DEBUG)
- Correlation IDs for request tracing
- Metrics for critical operations
- Health check endpoints

---

## Forbidden Practices (Strict)

The agent MUST NOT:

- Introduce new dependencies without justification
- Block async contexts with sync IO
- Violate architectural boundaries
- Implement functionality outside feature scope
- "Fix" unrelated code opportunistically
- Ignore memory leaks/resource leaks
- Hard-code configuration
- Use deprecated APIs without justification

---

## Deviation Policy

If deviation from this profile is unavoidable:

- The deviation MUST be explicitly documented
- The reason MUST be technical, not convenience-based
- The deviation will be reviewed during verification

Undocumented deviations are considered defects.

---

## Profile Authority

This profile:

- Is the base for ALL backend specialized profiles
- Defines universal backend rules
- Is overridden by specialized profiles where they have specific patterns
- Is subordinate only to ARCHITECTURE_OVERVIEW.md

Failure to comply with this profile is grounds for rejection during review or verification.

---

## Version

- **Version:** 1.0
- **Date:** 2025-01-24
- **Status:** Base profile
