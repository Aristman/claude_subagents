# Agent Profile: Mobile Android

## Profile Identity

- Profile ID: mobile-android
- Domain: mobile-android
- Platform: Android
- Scope: native Android mobile applications
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `mobile-android` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_mobile-android.md

---

## Supported Languages

The agent MUST use one of the following languages
(as determined by architecture and repository context):

- Kotlin (preferred)
- Java (only if legacy codebase requires it)

The agent MUST NOT introduce additional languages without architectural approval.

---

## Framework and Runtime Assumptions

The application is assumed to be:

- a native Android application
- running on Android OS
- distributed via Google Play or enterprise channels

Primary frameworks and libraries (if present in the project):

- Android SDK
- Jetpack libraries
- Jetpack Compose or XML-based UI (as defined by architecture)

The agent MUST:

- follow existing project conventions
- respect the chosen UI toolkit (Compose vs XML)
- follow Android lifecycle rules strictly

---

## Architectural Patterns

The agent MUST follow these patterns where applicable:

- Clear separation between:
    - UI layer
    - state / view-model layer
    - domain / business logic
    - data sources
- Unidirectional data flow
- Explicit lifecycle-aware components

Recommended patterns:

- MVVM or MVI (as defined by architecture)

The agent MUST NOT:

- place business logic in Activities or Composables
- create tight coupling between UI and data layers
- ignore lifecycle constraints

---

## State and Lifecycle Management

- UI state MUST be explicit and immutable
- State MUST survive configuration changes where required
- Background work MUST be lifecycle-aware

Forbidden practices:

- leaking Activities or Context
- unmanaged background threads
- relying on static mutable state for UI

---

## Navigation Rules

- Navigation MUST be explicit and predictable
- Back-stack behavior MUST be well-defined
- Deep links MUST be handled explicitly if required

The agent MUST NOT:

- hard-code navigation logic inside UI components
- create hidden navigation side effects

---

## API and Data Integration Rules

- All network access MUST go through defined data layers
- API contracts MUST be respected exactly
- Local storage MUST be abstracted (e.g., Room, DataStore)

The agent MUST:

- handle offline and error states explicitly
- avoid direct network calls from UI components

---

## Error Handling and UX Safety

The application MUST:

- handle loading, error, and empty states
- avoid application crashes due to unhandled exceptions
- provide user-friendly error feedback

The agent MUST:

- handle permission errors gracefully
- avoid blocking the UI thread

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests
    - ViewModels
    - domain logic
    - data mappers

2. Integration tests
    - repository + data source
    - ViewModel + data layer

Optional (if defined in roadmap or architecture):

- UI tests (Espresso / Compose testing)
- End-to-end tests

The agent MUST NOT:

- skip tests defined in the TDD roadmap
- rely only on manual testing or emulator checks

---

## Non-Functional Priorities

The agent MUST prioritize:

1. Correctness
2. Stability
3. Performance
4. Battery efficiency
5. Maintainability

The agent SHOULD:

- minimize unnecessary background work
- avoid excessive wake locks
- optimize recompositions (Compose)

---

## Performance and Resource Constraints

The agent MUST:

- avoid blocking the main (UI) thread
- optimize memory usage
- avoid unnecessary object allocations in hot paths

The agent MUST NOT:

- perform heavy computation on the UI thread
- cause memory leaks or excessive GC pressure

---

## Security Constraints

The agent MUST:

- treat all external input as untrusted
- use secure storage for sensitive data
- respect Android permission model

The agent MUST NOT:

- store secrets in plain text
- log sensitive user data
- bypass platform security mechanisms

---

## Tooling and Code Quality

- Code MUST follow Kotlin/Java style guidelines
- Lint warnings MUST be addressed
- Dead or unused code MUST NOT be introduced

The agent SHOULD:

- prefer idiomatic Kotlin
- keep code readable and modular

---

## Forbidden Practices (Strict)

The agent MUST NOT:

- mix UI and business logic
- introduce hidden lifecycle dependencies
- bypass repository or domain layers
- “fix” unrelated code opportunistically
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
