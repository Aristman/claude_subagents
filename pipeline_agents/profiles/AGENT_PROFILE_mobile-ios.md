# Agent Profile: Mobile iOS

## Profile Identity

- Profile ID: mobile-ios
- Domain: mobile-ios
- Platform: iOS
- Scope: native iOS mobile applications
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `mobile-ios` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_mobile-ios.md

---

## Supported Languages

The agent MUST use one of the following languages
(as determined by architecture and repository context):

- Swift (preferred)
- Objective-C (only if legacy codebase requires it)

The agent MUST NOT introduce additional languages without architectural approval.

---

## Framework and Runtime Assumptions

The application is assumed to be:

- a native iOS application
- running on iOS / iPadOS
- distributed via App Store or enterprise channels

Primary frameworks and libraries (if present in the project):

- SwiftUI or UIKit (as defined by architecture)
- Combine or async/await for async flows

The agent MUST:

- follow existing project conventions
- respect the chosen UI framework (SwiftUI vs UIKit)
- follow iOS lifecycle and memory rules strictly

---

## Architectural Patterns

The agent MUST follow these patterns where applicable:

- Clear separation between:
    - UI layer
    - state / view-model layer
    - domain / business logic
    - data sources
- Unidirectional data flow
- Explicit ownership of state and side effects

Recommended patterns:

- MVVM or MVI (as defined by architecture)

The agent MUST NOT:

- place business logic inside Views or ViewControllers
- create tight coupling between UI and networking
- rely on implicit shared mutable state

---

## State and Lifecycle Management

- UI state MUST be explicit and observable
- State MUST be predictable and testable
- Memory ownership MUST be clear and intentional

Forbidden practices:

- retain cycles
- leaking ViewControllers or Views
- unmanaged background tasks

---

## Navigation Rules

- Navigation MUST be explicit and predictable
- Back navigation behavior MUST be well-defined
- Deep links MUST be handled explicitly if required

The agent MUST NOT:

- embed navigation logic deeply inside UI views
- create hidden navigation side effects

---

## API and Data Integration Rules

- All network access MUST go through defined data layers
- API contracts MUST be respected exactly
- Local persistence MUST be abstracted (e.g., Core Data, file storage)

The agent MUST:

- handle offline and error states explicitly
- avoid direct network calls from UI code

---

## Error Handling and UX Safety

The application MUST:

- handle loading, error, and empty states
- avoid crashes due to unhandled errors
- provide clear, user-friendly error feedback

The agent MUST:

- handle permission and privacy errors gracefully
- avoid blocking the main thread

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests
    - ViewModels
    - domain logic
    - data mappers

2. Integration tests
    - data layer + networking
    - ViewModel + data layer

Optional (if defined in roadmap or architecture):

- UI tests (XCUITest)
- Snapshot tests

The agent MUST NOT:

- skip tests defined in the TDD roadmap
- rely only on manual simulator testing

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
- optimize view updates (SwiftUI diffing / UIKit layout)
- reduce memory footprint

---

## Performance and Resource Constraints

The agent MUST:

- avoid blocking the main thread
- manage memory explicitly
- avoid unnecessary object retention

The agent MUST NOT:

- perform heavy computation on the main thread
- introduce memory leaks or excessive allocations

---

## Security and Privacy Constraints

The agent MUST:

- treat all external input as untrusted
- use secure storage for sensitive data (Keychain)
- respect iOS privacy and permission model

The agent MUST NOT:

- store secrets in plain text
- log sensitive user data
- bypass platform security mechanisms

---

## Tooling and Code Quality

- Code MUST follow Swift / Objective-C style guidelines
- Lint warnings MUST be addressed
- Dead or unused code MUST NOT be introduced

The agent SHOULD:

- prefer idiomatic Swift
- keep code modular and readable

---

## Forbidden Practices (Strict)

The agent MUST NOT:

- mix UI and business logic
- introduce hidden lifecycle dependencies
- bypass domain or data layers
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
