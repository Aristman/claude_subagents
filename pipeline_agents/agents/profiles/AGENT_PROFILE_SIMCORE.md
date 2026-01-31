# Agent Profile: SIMCORE

## Domain Resolution

**Domain:** SIMCORE (Simulation Core)
**Context:** Desktop Multiplatform (Client-Server)

## Profile Constraints

### Technology Stack
- **Language:** Rust (stable)
- **Engine:** Bevy (ECS-driven)
- **Pattern:** Entity Component System
- **Storage:** ECS-based storage для клеток
- **Bevy Version:** 0.14+

### Platform Targets
- Windows 10+
- macOS 12+
- Linux (Ubuntu 22.04+)

### Performance Constraints
- **Target:** 100K+ cells
- **Frame budget:** <= 16.67ms (60 FPS)
- **Memory:** Conservative allocation
- **Iteration:** < 5ms for 100K cells
- **Random query:** < 1μs

### Quality Target
- Score >= 9/10
- Coverage >= 95% for core logic
- Coverage >= 80% for integration tests

## Development Rules

### MUST Follow
1. ECS pattern для всех game entities
2. Bevy Plugin system для feature integration
3. Coordinate system с bounds checking
4. Serialization support (serde + bincode)
5. TDD approach: tests before implementation

### MUST NOT Do
1. Direct entity spawning without ECS components
2. Circular dependencies between domains
3. Blocking operations in systems
4. Hard-coded dimensions without configuration

## Architecture Boundaries

### SIMCORE Domain Scope
- Grid data structures (ECS components)
- Coordinate system (GridBounds, GridCoordinate)
- Cell creation/destruction
- Cell state queries
- Spatial validation

### Out of Scope (Delegated)
- Terrain types (FEAT-003)
- Resources (FEAT-004)
- Rendering (FEAT-021)
- Physics simulation (FEAT-005)
- Spatial partitioning (FEAT-007)

## API Conventions

### Naming
- Components: PascalCase (Cell, Position)
- Resources: PascalCase + Resource suffix (WorldGridResource)
- Systems: snake_case (create_grid_cells)
- Plugins: PascalCase + Plugin suffix (WorldGridPlugin)

### Error Handling
- Use assertions for invariants in debug builds
- Return Result for external APIs
- Use Bevy's error handling for systems

### Documentation
- All public APIs must have rustdoc comments
- Module-level documentation required
- Examples for complex operations
