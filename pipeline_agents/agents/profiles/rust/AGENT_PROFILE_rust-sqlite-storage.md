# Agent Profile: Rust SQLite Storage

## Profile Identity

- Profile ID: rust-sqlite-storage
- Domain: D-DS-001 Data Storage
- Scope: SQLite database operations using Rust
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `D-BE-001` domain with storage responsibilities.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_rust-sqlite-storage.md

---

## Supported Languages

The agent MUST use:
- Rust (stable channel, edition 2021)
- SQL for queries (when applicable)

The agent MUST NOT introduce other languages without architectural approval.

---

## Framework and Runtime Assumptions

The system is assumed to run:
- using SQLite as embedded database
- using sqlx or rusqlite for database access
- using migrations for schema versioning
- targeting Windows 10/11, macOS 12+, Linux Ubuntu 22.04+

The agent MUST:
- follow Rust best practices and idioms
- use parameterized queries (prevent SQL injection)
- respect transaction boundaries

---

## Architectural Patterns

The agent MUST follow these principles:
- Clear separation between:
    - database layer (SQL operations)
    - domain models (structs/enums)
    - migration logic
- Repository pattern for data access
- Explicit transaction management

The agent MUST NOT:
- embed SQL queries outside repository layer
- create hidden dependencies between tables
- rely on global database connections

---

## Database Rules

- Queries MUST use parameterized statements
- Migrations MUST be reversible
- Schema changes MUST go through migrations
- Indexes MUST be used for query optimization

The agent MUST NOT:
- construct SQL by string concatenation
- use SELECT * in production code
- ignore foreign key constraints

---

## Error Handling

- All database operations MUST return Result<T, E>
- Database-specific error types MUST be defined
- Errors MUST be properly logged
- Transaction errors MUST result in rollback

The agent MUST:
- use thiserror for error definitions
- handle connection errors gracefully
- provide context for database errors

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests
    - migration logic
    - query building (if applicable)
    - model validation

2. Integration tests
    - CRUD operations
    - transaction behavior
    - migration rollback

The agent MUST NOT:
- skip tests defined in the TDD roadmap
- rely only on manual database inspection

---

## Non-Functional Priorities

The agent MUST prioritize:
1. Data integrity
2. Correctness
3. Performance (appropriate indexing)
4. Maintainability
5. Migration safety

The agent SHOULD:
- use indexes for frequently queried columns
- avoid N+1 queries
- keep transactions short

---

## Performance Constraints

The agent MUST:
- use appropriate indexes
- avoid full table scans
- use transactions for multi-step operations
- respect SQLite limitations

The agent SHOULD:
- profile slow queries
- batch operations when appropriate

---

## Security Constraints

The agent MUST:
- use parameterized queries (prevent SQL injection)
- never trust input from other layers
- encrypt sensitive data (API keys) before storage
- validate data before database operations

The agent MUST NOT:
- construct SQL with user input
- store plaintext secrets
- expose database errors directly to users

---

## Migration Rules

- Migrations MUST be versioned
- Migrations MUST be reversible
- Migrations MUST NOT destroy data without backup
- Migrations MUST be tested

The agent MUST:
- document migration purpose
- test both upgrade and rollback
- handle migration errors gracefully

---

## Tooling and Code Quality

- Code MUST pass clippy with minimal warnings
- Code MUST be formatted with rustfmt
- Public APIs MUST have rustdoc comments
- SQL queries MUST be readable

The agent SHOULD:
- use meaningful table/column names
- keep migrations focused
- document complex queries

---

## Data Integrity

The agent MUST:
- use foreign key constraints
- use appropriate column types
- define NOT NULL constraints where applicable
- use UNIQUE constraints for unique data

The agent SHOULD:
- add CHECK constraints for validation
- use transactions for multi-table operations

---

## File System Storage (for images)

The agent MUST:
- store metadata in SQLite
- store binary data (images) in file system
- manage file paths correctly
- handle missing files gracefully

The agent MUST NOT:
- store large blobs in SQLite
- leak file handles
- create orphaned files

---

## Russian Language Requirements

The agent MUST:
- handle UTF-8 correctly for Russian text
- support Unicode in text fields
- ensure proper collation for Russian strings

---

## Forbidden Practices (Strict)

The agent MUST NOT:
- construct SQL by string concatenation
- ignore foreign key constraints
- store unencrypted secrets
- "fix" unrelated data opportunistically
- skip migration testing

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
