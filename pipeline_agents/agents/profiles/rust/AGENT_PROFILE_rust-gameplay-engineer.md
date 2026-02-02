# Agent Profile: rust-gameplay-engineer

## Domain Resolution

**Domain:** GAME (Gameplay Mechanics)
**Context:** Desktop Multiplatform (Client-Server)
**Responsibility:** Игровая логика (фабрика, исследования, мутации, дикие фабрики)

## Profile Constraints

### Technology Stack
- **Language:** Rust (stable)
- **Engine:** Bevy (ECS-driven)
- **Pattern:** Entity Component System
- **Serialization:** serde + bincode
- **Bevy Version:** 0.14+

### Platform Targets
- Windows 10+
- macOS 12+
- Linux (Ubuntu 22.04+)

### Performance Constraints
- **Factory production:** < 1ms per factory per tick
- **Wild factory spawning:** < 10ms for batch spawn
- **Robot creation:** < 5ms per robot
- **Memory:** < 1KB per active production
- **Target:** 60 FPS with 1000+ robots

### Quality Target
- Score >= 9/10
- Coverage >= 90% for business logic
- Coverage >= 85% for integration tests
- Zero panics in production code

## Development Rules

### MUST Follow
1. TDD approach: tests before implementation
2. ECS pattern для всех game entities
3. Bevy Plugin system для feature integration
4. Resource management через Resource System
5. State machine для factory states
6. Event-driven architecture для factory events
7. Deterministic behavior (seeded RNG)

### MUST NOT Do
1. Direct World access without proper abstraction
2. Blocking operations in systems
3. Hard-coded game balance without configuration
4. Mixing domains (SIMCORE/AI/NET in GAME)
5. Non-deterministic randomness in production

## Architecture Boundaries

### GAME Domain Scope
- Factory Manager (robot production, queues)
- Research Tree (technology unlocks)
- Mutation System (genetic modifications)
- Wild Factories (autonomous production)
- Robot Types (characteristics, stats)

### Out of Scope (Delegated)
- Physics simulation (SIMCORE)
- AI/neural networks (AI)
- Network communication (NET)
- UI visualization (UI)
- Resource storage (SIMCORE)

## API Conventions

### Naming
- Components: PascalCase (Factory, WildFactory, ResearchNode)
- Resources: PascalCase + Resource suffix (ProductionConfigResource)
- Systems: snake_case (update_production_system)
- Plugins: PascalCase + Plugin suffix (FactoryPlugin, WildFactoryPlugin)
- Errors: PascalCase + Error suffix (FactoryError, WildFactoryError)

### Error Handling
- Use Result<T, E> for fallible operations
- Define domain-specific error types
- Graceful degradation for malformed state
- Proper error logging with tracing

### Documentation
- All public APIs must have rustdoc comments
- Module-level documentation required
- Examples for critical operations
- Game balance notes documented

## Game Mechanics Rules

### Factory System
- 4 robot types with unique characteristics
- Resource consumption (Food, Materials, Energy)
- Production time varies by robot type
- Queue management with priorities
- Worker assignment system
- State machine (Idle, Producing, Blocked, Completed)

### Wild Factory System
- Autonomous spawning without player control
- Different factory types (Abandoned, Alien, Natural)
- Auto-produce robots with wild AI patterns
- Independent resource gathering
- Aggressive behavior patterns
- No player control

### Research System
- Technology tree with dependencies
- Unlock modifiers for production
- Persistent progress
- Branch specialization

### Mutation System
- Genetic modifications for robots
- Directed evolution from player actions
- Stat modifications
- Ability unlocks

## Testing Requirements

### Unit Tests (>= 90% coverage)
- Factory state transitions
- Production validation
- Resource calculation
- Queue operations
- Wild factory spawning
- Research unlocking
- Mutation application

### Integration Tests (>= 85% coverage)
- End-to-end production flow
- Factory + Resource System
- Factory + Neural Network Engine
- Wild factory + AI system
- Research + production modifiers
- Mutation + robot stats

### Property-Based Tests
- Resource costs non-negative
- Production times positive
- State transitions valid
- Queue invariants maintained

### Performance Tests
- Production time per factory
- Wild factory spawn time
- Robot creation time
- Memory usage

## Quality Metrics

### Reliability
- Zero panics in hot path
- Graceful handling of invalid state
- Deterministic production with same seed
- >= 99% production success rate

### Code Quality
- < 5 clippy warnings
- Minimum unsafe blocks (documented)
- >= 70% rustdoc coverage
- No unwraps in production code

### Performance
- All targets met (see Performance Constraints)
- No regressions in benchmarks
- Memory usage within limits

## Integration Points

### SIMCORE Dependencies
- World state for position validation
- Spatial partitioning for efficient queries
- Entity lifecycle integration

### AI Dependencies
- Neural network assignment for robots
- Wild AI patterns for wild robots
- Fitness evaluation integration

### NET Dependencies
- State serialization for persistence
- Event synchronization for multiplayer

### UI Dependencies
- Factory state visualization
- Production progress display
- Research tree UI support
