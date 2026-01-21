# Agent Profile: CLI

## Profile Identity

- Profile ID: cli
- Domain: cli
- Scope: command-line applications and developer tools
- Environment: local machine, CI environments, servers
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `cli` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_cli.md

---

## Supported Languages

The agent MUST use one of the following languages
(as determined by architecture and repository context):

- Rust
- Go
- Python
- Node.js (TypeScript / JavaScript)

The agent MUST NOT introduce a new language without architectural approval.

---

## Runtime and Environment Assumptions

CLI applications are assumed to:

- run in terminal environments
- be scriptable and automatable
- support non-interactive usage (CI / piping)

The agent MUST:

- respect POSIX conventions where applicable
- support execution without a graphical environment
- behave predictably in headless environments

---

## CLI Interface Design Rules

- Commands MUST be explicit and discoverable
- Flags and arguments MUST be well-documented
- Defaults MUST be safe and conservative
- Output MUST be machine-readable where appropriate

The agent MUST:

- provide `--help` output
- use standard exit codes (0 = success, non-zero = error)

The agent MUST NOT:

- rely on interactive prompts unless explicitly required
- produce ambiguous or undocumented output
- break scripts by changing output format silently

---

## Input / Output Behavior

- STDIN MAY be used for input when appropriate
- STDOUT MUST be used for normal output
- STDERR MUST be used for errors and diagnostics

The agent MUST NOT:

- mix structured output with logs on STDOUT
- hide errors in verbose-only modes

---

## Error Handling Rules

- Errors MUST be explicit and descriptive
- Error messages MUST be concise and actionable
- Exit codes MUST reflect failure types where possible

Silent failure is forbidden.

---

## Configuration Management

- Configuration MUST be explicit
- Environment variables MAY be supported
- Config files MUST be optional and documented

The agent MUST:

- clearly define configuration precedence
- avoid hidden or implicit configuration sources

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests
    - command parsing
    - business logic
    - utility functions

2. Integration tests
    - command execution
    - filesystem or external interactions (mocked where possible)

Optional (if defined in roadmap or architecture):

- End-to-end CLI tests
- Snapshot tests for output

The agent MUST NOT:

- rely only on manual CLI testing
- skip tests defined in the TDD roadmap

---

## Non-Functional Priorities

The agent MUST prioritize:

1. Correctness
2. Predictability
3. Scriptability
4. Performance
5. Maintainability

The agent SHOULD:

- keep startup time low
- avoid unnecessary dependencies
- support quiet / verbose modes where appropriate

---

## Performance Constraints

The agent MUST:

- avoid unnecessary blocking operations
- handle large input/output streams efficiently
- avoid excessive memory usage

Premature optimization is discouraged.

---

## Security Constraints

The agent MUST:

- treat all external input as untrusted
- avoid shell injection vulnerabilities
- sanitize file paths and arguments

The agent MUST NOT:

- execute shell commands with untrusted input
- store secrets in plain text
- log sensitive data

---

## Tooling and Code Quality

- Code MUST follow language-specific style guidelines
- Lint warnings MUST be addressed
- Dead or unused code MUST NOT be introduced

The agent SHOULD:

- write self-documenting code
- keep command handlers small and focused

---

## Forbidden Practices (Strict)

The agent MUST NOT:

- change output formats without versioning
- break backward compatibility silently
- rely on interactive-only flows
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
