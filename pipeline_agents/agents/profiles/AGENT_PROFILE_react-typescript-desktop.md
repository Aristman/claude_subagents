# Agent Profile: React + TypeScript Desktop (Tauri)

## Profile Identity

- Profile ID: react-typescript-desktop
- Domain: D-FE-001 Frontend
- Scope: Desktop applications using Tauri + React + TypeScript
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `D-FE-001` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_react-typescript-desktop.md

---

## Supported Languages

The agent MUST use:
- TypeScript (strict mode)
- TSX for React components

The agent MUST NOT introduce JavaScript files without justification.

---

## Framework and Runtime Assumptions

The system is assumed to run:
- in Tauri WebView (desktop application)
- using React 18+ with TypeScript
- using Vite for building
- using Tauri API for IPC communication

The agent MUST:
- follow React best practices (hooks, functional components)
- use Tauri IPC for all backend communication
- respect the desktop application context (not web-specific features)

---

## Architectural Patterns

The agent MUST follow these principles:
- Clear separation between:
    - UI components
    - state management (React Context or Zustand)
    - Tauri IPC calls
- Predictable, explicit data flow
- Stateless UI components where possible
- Components organized by feature

The agent MUST NOT:
- embed business logic directly in UI components
- bypass Tauri IPC for data operations
- rely on global mutable state outside defined stores
- use web-only APIs without checking Tauri availability

---

## State Management Rules

- Application state MUST be explicit
- State updates MUST be predictable and traceable
- Tauri IPC calls MUST be isolated in custom hooks

Forbidden practices:
- implicit state mutations
- uncontrolled shared state
- direct Tauri command calls in render

---

## Tauri IPC Rules

- All backend communication MUST go through Tauri commands
- Command contracts MUST be respected exactly
- Error states MUST be handled explicitly in the UI
- Commands MUST be called through React hooks

The agent MUST NOT:
- bypass Tauri IPC abstraction
- hard-code API endpoints (use Tauri commands)
- silently ignore failed Tauri commands

---

## Error Handling and UX Safety

- UI MUST handle:
    - loading states
    - error states
    - empty states
- User-facing errors MUST be understandable on Russian language
- Crashes or blank screens are forbidden

The agent MUST:
- fail gracefully
- avoid unhandled promise rejections
- show meaningful error messages to users

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests (Vitest + React Testing Library)
    - components (logic-focused)
    - custom hooks
    - utility functions

2. Integration tests
    - component + state
    - component + Tauri IPC mocks

The agent MUST NOT:
- skip tests defined in the TDD roadmap
- rely only on manual testing

---

## Non-Functional Priorities

The agent MUST prioritize:
1. Correctness
2. Usability
3. Performance (< 100ms UI response)
4. Accessibility
5. Maintainability

The agent SHOULD:
- avoid unnecessary re-renders (React.memo, useMemo, useCallback)
- respect accessibility standards (ARIA, keyboard navigation)
- provide Russian language UI

---

## Performance Constraints

The agent MUST:
- avoid blocking the main thread
- avoid unnecessary heavy computations in render paths
- lazy-load non-critical resources where appropriate
- keep UI response time under 100ms

---

## Security Constraints

The agent MUST:
- treat all external input as untrusted
- prevent XSS and injection vectors
- avoid dangerous DOM manipulation patterns
- never expose API keys or secrets in frontend code

The agent MUST NOT:
- use dangerouslySetInnerHTML without sanitization
- expose sensitive data in client code
- store secrets in frontend artifacts

---

## Tooling and Code Quality

- Code MUST follow project formatting (Prettier) and linting (ESLint) rules
- Dead or unused code MUST NOT be introduced
- Code SHOULD be readable and intention-revealing
- TypeScript strict mode MUST be enabled

The agent SHOULD:
- prefer composition over inheritance
- avoid overly complex component trees
- use TypeScript types for all props and state

---

## Tauri-Specific Rules

The agent MUST:
- use @tauri-apps/api for all Tauri interactions
- invoke commands through the invoke() function
- handle Tauri-specific errors appropriately
- respect the async nature of Tauri commands

The agent MUST NOT:
- assume web APIs are available (use Tauri plugins instead)
- make direct fetch calls to backend (use Tauri commands)

---

## Russian Language Requirements

The agent MUST:
- provide all UI text in Russian
- provide all error messages in Russian
- use appropriate Russian terminology for DIY/construction context
- ensure proper character encoding (UTF-8)

---

## Forbidden Practices (Strict)

The agent MUST NOT:
- implement backend logic in frontend
- introduce new global state without justification
- mix presentation and Tauri IPC logic
- "fix" unrelated code opportunistically
- bypass established component structure

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
