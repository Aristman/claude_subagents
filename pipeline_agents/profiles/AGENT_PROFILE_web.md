# Agent Profile: Web

## Profile Identity

- Profile ID: web
- Domain: web-frontend
- Scope: browser-based client applications (SPA / MPA)
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `web` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_web.md

---

## Supported Languages

The agent MUST use one of the following languages
(as determined by architecture and repository context):

- TypeScript
- JavaScript

The agent MUST NOT introduce a new language without architectural approval.

---

## Framework and Runtime Assumptions

The system is assumed to run:

- in modern web browsers
- in a client-side or hybrid (SSR/CSR) environment

Typical frameworks (used only if present in the project):

- React
- Vue
- Svelte
- Angular

The agent MUST:

- follow existing framework and project conventions
- respect the chosen rendering strategy (CSR / SSR / SSG)

---

## Architectural Patterns

The agent MUST follow these principles:

- Clear separation between:
    - UI components
    - state management
    - side effects (API, browser APIs)
- Predictable, explicit data flow
- Stateless UI components where possible

The agent MUST NOT:

- embed business logic directly in UI components
- create hidden coupling between UI and network layers
- rely on global mutable state outside defined stores

---

## State Management Rules

- Application state MUST be explicit
- State updates MUST be predictable and traceable
- Side effects MUST be isolated

Forbidden practices:

- implicit state mutations
- uncontrolled shared state
- logic inside rendering expressions

---

## API and Integration Rules

- All external communication MUST go through defined API clients
- API contracts MUST be respected exactly
- Error states MUST be handled explicitly in the UI

The agent MUST NOT:

- bypass API abstraction layers
- hard-code API endpoints without configuration
- silently ignore failed requests

---

## Error Handling and UX Safety

- UI MUST handle:
    - loading states
    - error states
    - empty states
- User-facing errors MUST be understandable
- Crashes or blank screens are forbidden

The agent MUST:

- fail gracefully
- avoid unhandled promise rejections

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests
    - components (logic-focused)
    - state reducers / stores
    - utility functions

2. Integration tests
    - component + state
    - component + API mocks

Optional (if defined in roadmap or architecture):

- E2E tests
- Accessibility tests

The agent MUST NOT:

- skip tests defined in the TDD roadmap
- rely only on manual browser testing

---

## Non-Functional Priorities

The agent MUST prioritize:

1. Correctness
2. Usability
3. Accessibility
4. Performance
5. Maintainability

The agent SHOULD:

- avoid unnecessary re-renders
- optimize bundle size where applicable
- respect accessibility standards (ARIA, keyboard navigation)

---

## Performance Constraints

The agent MUST:

- avoid blocking the main thread
- avoid unnecessary heavy computations in render paths
- lazy-load non-critical resources where appropriate

Premature micro-optimizations are discouraged.

---

## Security Constraints

The agent MUST:

- treat all external input as untrusted
- prevent XSS and injection vectors
- avoid dangerous DOM manipulation patterns

The agent MUST NOT:

- use `innerHTML` without sanitization
- expose sensitive data in client code
- store secrets in frontend artifacts

---

## Tooling and Code Quality

- Code MUST follow project formatting and linting rules
- Dead or unused code MUST NOT be introduced
- Code SHOULD be readable and intention-revealing

The agent SHOULD:

- prefer composition over inheritance
- avoid overly complex component trees

---

## Forbidden Practices (Strict)

The agent MUST NOT:

- implement backend logic in frontend
- introduce new global state without justification
- mix presentation and side-effect logic
- “fix” unrelated UI code opportunistically
- bypass established design or component systems

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
