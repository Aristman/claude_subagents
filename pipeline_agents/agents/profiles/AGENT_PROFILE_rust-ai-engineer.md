# Agent Profile: rust-ai-engineer

## Domain Resolution

**Domain:** AI (Robot Intelligence)
**Context:** Desktop Multiplatform (Client-Server)
**Responsibility:** AI система роботов (neural networks, genetic algorithm, sensors, AI serialization)

## Profile Constraints

### Technology Stack
- **Language:** Rust (stable)
- **Engine:** Bevy (ECS-driven)
- **AI Framework:** Custom feedforward neural network
- **Genetic Algorithm:** In-house implementation
- **Serialization:** serde + bincode
- **Bevy Version:** 0.14+

### Platform Targets
- Windows 10+
- macOS 12+
- Linux (Ubuntu 22.04+)

### Performance Constraints
- **Inference time:** < 10ms per robot (1000+ robots)
- **Sensor read:** < 5ms per robot
- **Mutation time:** < 1ms per network
- **Evolution cycle:** < 10s for 1000 robots
- **Memory:** < 1GB for 1000 robots

### Quality Target
- Score >= 9/10
- Coverage >= 90% for business logic
- Coverage >= 80% for integration tests
- Deterministic AI behavior (seeded RNG)
- Zero panics in production code

## Development Rules

### MUST Follow
1. TDD approach: tests before implementation
2. Feedforward neural network architecture
3. Genetic algorithm with weight/architecture mutations
4. Sensor system (vision, proximity, resource detector)
5. ECS components for AI state
6. Bevy Plugin system for integration
7. Serialization support for AI networks
8. Deterministic RNG (seeded StdRng)

### MUST NOT Do
1. Implement training (only inference)
2. Use external ML frameworks (keep it simple)
3. Hard-coded network architectures
4. Non-deterministic mutations in production
5. Blocking operations in AI systems
6. Direct World access without proper abstraction

## Architecture Boundaries

### AI Domain Scope
- Neural Network Engine (feedforward inference)
- Sensor System (vision, proximity, resource detection)
- Genetic Algorithm (mutations, selection, fitness)
- AI Serialization (save/load networks)
- AI Components (RobotAI, NeuralNetwork, FitnessScore)
- AI Systems (InferenceSystem, SensorSystem, EvolutionSystem)

### Out of Scope (Delegated)
- Physics simulation (SIMCORE)
- Network communication (NET)
- UI visualization (UI)
- Game logic (GAME)
- Training/backpropagation (future feature)

## API Conventions

### Naming
- Components: PascalCase (RobotAI, NeuralNetwork)
- Resources: PascalCase + Resource suffix (GeneticConfigResource)
- Systems: snake_case (run_neural_inference, evaluate_fitness)
- Plugins: PascalCase + Plugin suffix (SensorSystemPlugin)
- Errors: PascalCase + Error suffix (InferenceError)

### Error Handling
- Use Result<T, E> for fallible operations
- Define domain-specific error types
- Graceful degradation for malformed AI networks
- Proper error logging with tracing

### Documentation
- All public APIs must have rustdoc comments
- Module-level documentation required
- Examples for critical operations
- Performance characteristics documented

## AI-Specific Rules

### Neural Network Architecture
- Feedforward only (no RNN/LSTM for MVP)
- Activation functions: ReLU, Sigmoid, Tanh, Softmax
- Configurable layers and neurons
- Weight clamping to prevent overflow

### Genetic Algorithm
- Weight mutations (gaussian, uniform)
- Architecture mutations (add/remove layers, neurons)
- Selection methods (tournament, fitness proportionate)
- Elitism for top performers
- Fitness evaluation (survival, resources, reproduction)

### Sensor System
- Vision sensor (object detection in radius)
- Proximity sensor (nearest object distance)
- Resource detector (resource type filtering)
- Raycasting for line-of-sight (vision)
- Efficient spatial queries

### Determinism
- Seeded RNG for all random operations
- Reproducible mutations with same seed
- Deterministic inference (no floating-point non-determinism)
- Testable AI behavior

## Testing Requirements

### Unit Tests (>= 90% coverage)
- Neural network inference
- Activation functions
- Mutation operations
- Selection mechanisms
- Fitness evaluation
- Sensor readings

### Integration Tests (>= 80% coverage)
- Full evolution cycle
- Sensor + Neural Network integration
- AI + SIMCORE integration
- Serialization/deserialization

### Property-Based Tests
- Weight bounded invariant
- Network validity invariant
- Fitness non-negative invariant
- Population size invariant

### Performance Tests
- Inference time per robot
- Sensor read time
- Mutation time
- Evolution cycle for 1000 robots

## Quality Metrics

### Reliability
- Zero panics in hot path
- Graceful handling of malformed networks
- Deterministic behavior with same seed
- >= 99% inference success rate

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
- World state queries for sensors
- Spatial partitioning for efficient queries
- Entity lifecycle integration

### NET Dependencies
- AI serialization for persistence
- Network synchronization for AI state

### GAME Dependencies
- Fitness evaluation based on game events
- Directed mutations from player actions

### UI Dependencies
- AI visualization support
- Debug rendering for sensors
