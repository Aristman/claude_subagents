# Agent Profile: rust-network-engineer

## Domain Resolution

**Domain:** NET (Networking)
**Context:** Desktop Multiplatform (Client-Server)
**Responsibility:** Сетевой слой (сервер, протокол, 24/7 симуляция, персистентность)

## Profile Constraints

### Technology Stack
- **Language:** Rust (stable)
- **Engine:** Bevy (ECS-driven)
- **Serialization:** serde + bincode (binary format)
- **Async Runtime:** tokio (full features)
- **Storage:** Local file system (для MVP)
- **Bevy Version:** 0.14+

### Platform Targets
- Windows 10+
- macOS 12+
- Linux (Ubuntu 22.04+)

### Performance Constraints
- **Snapshot creation:** <= 1s (100K cells, 1000 robots)
- **Load time:** <= 2s (snapshot + 1000 events)
- **Event replay rate:** >= 5000 events/s
- **Snapshot size:** <= 10MB
- **Memory spike:** <= 50MB during load

### Quality Target
- Score >= 9/10
- Coverage >= 90% for core logic
- Coverage >= 80% for integration tests
- Zero data loss on graceful shutdown
- < 5 minutes data loss on crash

## Development Rules

### MUST Follow
1. Binary serialization (bincode) для persistence
2. Checksum validation (CRC32) для corruption detection
3. Atomic file operations (temp file + rename)
4. Event log append-only pattern
5. TDD approach: tests before implementation
6. Snapshot + event log hybrid persistence
7. Graceful shutdown with state preservation

### MUST NOT Do
1. Blocking I/O in async systems
2. In-memory-only state without persistence
3. Unvalidated file reads (checksum required)
4. Hard-coded file paths (use configuration)
5. Direct World access without proper abstraction

## Architecture Boundaries

### NET Domain Scope
- Persistence Manager (snapshot + event log)
- Protocol Handler (serialization)
- Network Server (future)
- Network Client (future)
- Crash recovery system
- Graceful shutdown handling

### Out of Scope (Delegated)
- World state management (SIMCORE)
- AI persistence (AI domain)
- Game logic persistence (GAME domain)
- Cloud storage (Phase 2)

## API Conventions

### Naming
- Components: PascalCase
- Resources: PascalCase + Resource suffix
- Systems: snake_case
- Plugins: PascalCase + Plugin suffix
- Errors: PascalCase + Error suffix

### Error Handling
- Use Result<T, E> для fallible operations
- Define domain-specific error types
- Proper error propagation
- Logging для all errors

### Documentation
- All public APIs must have rustdoc comments
- Module-level documentation required
- Examples for critical operations
- Performance characteristics documented

## Persistence-Specific Rules

### File Structure
```
saves/
├── autosave_<timestamp>.snapshot
├── autosave_<timestamp>.events
├── manual_save_<name>.snapshot
└── manual_save_<name>.events
```

### Snapshot Format
1. Header: version, timestamp, checksum
2. World state: binary serialized
3. Checksum: CRC32

### Event Log Format
1. Append-only binary log
2. Each event: timestamp, type, data
3. Rotation after snapshot

### Recovery Strategy
1. Load latest valid snapshot
2. Replay events from log
3. Fallback to previous snapshot if corruption detected
4. Create new world if no snapshot exists

## Testing Requirements

### Unit Tests (>= 90% coverage)
- Snapshot creation/validation
- Event log recording/replay
- Checksum calculation
- Error handling

### Integration Tests (>= 80% coverage)
- Load/restore world state
- Crash recovery scenarios
- Cross-domain integration (SIMCORE, AI)

### Performance Tests
- Snapshot creation time
- Load operation time
- Event replay rate
- Memory usage during load

### Property-Based Tests
- Corruption detection
- Randomized file operations

## Quality Metrics

### Reliability
- Zero data loss (graceful shutdown)
- < 5 minutes data loss (crash)
- 100% corruption detection (deterministic)
- >= 99% recovery success rate

### Code Quality
- < 5 clippy warnings
- Minimum unsafe blocks (documented)
- >= 70% rustdoc coverage
- No unwraps in production code

### Performance
- All targets met (see Performance Constraints)
- No regressions in benchmarks
- Memory usage within limits
