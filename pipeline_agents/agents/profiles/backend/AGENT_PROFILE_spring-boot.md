# Agent Profile: Backend Spring Boot

## Profile Identity

- Profile ID: spring-boot
- Domain: backend
- Stack: JVM/Spring
- Extends: `backend/AGENT_PROFILE_backend-base.md`
- Scope: Spring Boot applications with Kotlin
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent

This profile is **mandatory** for any feature assigned to the `backend` domain with **Kotlin + Spring Boot** stack.

---

## Inheritance

This profile extends `backend/AGENT_PROFILE_backend-base.md` with Spring Boot specific rules.

General backend rules (layered architecture, error handling, testing, observability) still apply.

---

## Supported Languages and Frameworks

**Mandatory:**
- Language: Kotlin 1.9+
- JDK: 17 or 21 (LTS)
- Framework: Spring Boot 3.2+
- Build: Gradle (Kotlin DSL)

**The agent MUST NOT:**
- Use Java unless explicitly required
- Introduce other JVM languages (Scala, Groovy)
- Change Spring Boot version without architectural approval
- Use Maven unless existing project uses it

---

## Project Structure (Standard)

Follow this package structure:

```
com.notes.api/
├── config/          # @Configuration classes
├── controller/      # @RestController classes
├── service/         # @Service classes (business logic)
├── repository/      # Repository interfaces (data access)
├── entity/          # JPA @Entity classes
├── dto/             # Request/Response DTOs
│   ├── request/     # Request DTOs
│   └── response/    # Response DTOs
├── exception/       # Custom exceptions
└── util/            # Utility classes (if needed)
```

**The agent MUST:**
- Place classes in correct packages
- Follow Spring naming conventions (XxxController, XxxService, XxxRepository)
- Use internal/package-private visibility where possible

---

## Spring Boot Specific Patterns

### Controllers

```kotlin
@RestController
@RequestMapping("/api/notes")
class NoteController(
    private val noteService: NoteService
) {
    @GetMapping
    fun getAll(): List<NoteResponseDto> = noteService.findAll()

    @PostMapping
    fun create(@Valid @RequestBody request: NoteCreateRequestDto): NoteResponseDto =
        noteService.create(request)

    @GetMapping("/{id}")
    fun getById(@PathVariable id: Long): NoteResponseDto =
        noteService.findById(id) ?: throw NotFoundException("Note", id)

    @PutMapping("/{id}")
    fun update(
        @PathVariable id: Long,
        @Valid @RequestBody request: NoteUpdateRequestDto
    ): NoteResponseDto = noteService.update(id, request)

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    fun delete(@PathVariable id: Long) = noteService.delete(id)
}
```

**Rules:**
- Use `@RestController` with `@RequestMapping`
- Return DTOs, NOT entities directly
- Use `@Valid` for request body validation
- Throw exceptions, don't return HTTP codes directly
- Inject dependencies via constructor

### Services

```kotlin
@Service
@Transactional
class NoteService(
    private val noteRepository: NoteRepository,
    private val tagRepository: TagRepository
) {
    fun findAll(): List<NoteResponseDto> =
        noteRepository.findAll()
            .map { it.toResponseDto() }

    fun findById(id: Long): NoteResponseDto? =
        noteRepository.findById(id)?.toResponseDto()

    @Transactional
    fun create(request: NoteCreateRequestDto): NoteResponseDto {
        val entity = request.toEntity()
        return noteRepository.save(entity).toResponseDto()
    }

    @Transactional
    fun update(id: Long, request: NoteUpdateRequestDto): NoteResponseDto {
        val entity = noteRepository.findById(id)
            ?: throw NotFoundException("Note", id)
        entity.updateFrom(request)
        return noteRepository.save(entity).toResponseDto()
    }

    @Transactional
    fun delete(id: Long) {
        if (!noteRepository.existsById(id)) {
            throw NotFoundException("Note", id)
        }
        noteRepository.deleteById(id)
    }
}
```

**Rules:**
- Use `@Service` annotation
- Make methods `transactional` with `@Transactional`
- Business logic goes here, NOT in controllers or repositories
- Use repository methods for data access

### Repositories

```kotlin
@Repository
interface NoteRepository : JpaRepository<Note, Long>, JpaSpecificationExecutor<Note> {

    @Query("SELECT n FROM Note n WHERE n.userId = :userId")
    fun findAllByUserId(@Param("userId") userId: Long): List<Note>

    fun findByUserIdAndTitleContaining(userId: Long, title: String): List<Note>
}
```

**Rules:**
- Extend `JpaRepository` or `JpaSpecificationExecutor`
- Define custom queries with `@Query` only when necessary
- Use method naming conventions for simple queries
- Use `Optional<T>` for single results

### Entities

```kotlin
@Entity
@Table(name = "notes", indexes = [
    Index(name = "idx_notes_user_id", columnList = "user_id"),
    Index(name = "idx_notes_created_at", columnList = "created_at")
])
data class Note(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,

    @Column(name = "user_id", nullable = false)
    val userId: Long,

    @Column(nullable = false)
    val title: String,

    @Column(length = 10000)
    val content: String? = null,

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    val color: NoteColor = NoteColor.GRAY,

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", insertable = false, updatable = false)
    val user: User? = null,

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "note_tags",
        joinColumns = [JoinColumn(name = "note_id")],
        inverseJoinColumns = [JoinColumn(name = "tag_id")]
    )
    var tags: MutableList<Tag> = mutableListOf(),

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    val createdAt: LocalDateTime = LocalDateTime.now(),

    @LastModifiedDate
    @Column(name = "updated_at", nullable = false)
    var updatedAt: LocalDateTime = LocalDateTime.now()
)
```

**Rules:**
- Use JPA annotations: `@Entity`, `@Table`, `@Id`, `@GeneratedValue`
- Use Kotlin data classes with `no-arg` compiler plugin
- Define relationships explicitly: `@OneToMany`, `@ManyToOne`, `@ManyToMany`
- Use `fetch = FetchType.LAZY` for collections
- Define indexes in `@Table` annotation

### Configuration

```kotlin
@Configuration
class SecurityConfig(
    private val jwtAuthenticationFilter: JwtAuthenticationFilter
) {
    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        return http
            .csrf { it.disable() }
            .sessionManagement { it.sessionCreationPolicy(SessionCreationPolicy.STATELESS) }
            .authorizeHttpRequests {
                it.requestMatchers("/api/auth/**").permitAll()
                    .anyRequest().authenticated()
            }
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter::class.java)
            .build()
    }
}
```

**Rules:**
- Use `@Configuration` classes for beans
- Use `application.yml` for properties (not .properties)
- Externalize secrets to environment variables
- Prefer constructor injection over field injection

---

## Kotlin Specific Rules

**The agent MUST:**
- Use idiomatic Kotlin (null safety, extension functions, data classes)
- Avoid Java-style getters/setters (Kotlin properties auto-generate them)
- Use `val` over `var` (immutability preferred)
- Use Kotlin stdlib functions (map, filter, let, run, also, apply)
- Use `?.let` for null-safe operations
- Use Elvis operator `?:` for default values

**The agent MUST NOT:**
- Use `!!` operator without strong justification
- Write Java-style null checks
- Ignore compiler warnings
- Use `lateinit` when nullable property is better

---

## Database (JPA/Hibernate)

**The agent MUST:**
- Use Flyway for migrations in `src/main/resources/db/migration/`
- Name migrations: `V{version}__{description}.sql`
- Define indexes in migrations (not `@Index` annotations)
- Use `@Transactional` for write operations
- Use JOIN FETCH to avoid N+1 queries

**The agent MUST NOT:**
- Use raw SQL in repository methods unless necessary
- Use `@Lob` for large text (use `@Column(length = ...)` instead)
- Forget cascade settings for relationships
- Load lazy collections outside transaction

---

## Security (Spring Security + JWT)

**For AUTH features:**
- Use `BCryptPasswordEncoder` for passwords (strength 10-12)
- Generate JWT with reasonable expiry:
  - Access token: 15 minutes
  - Refresh token: 7 days
- Store refresh tokens in database for revocation
- Access tokens are stateless (not stored)

**For protected endpoints:**
- Use JWT filter to validate tokens on each request
- Extract userId from claims and set in SecurityContext
- Return 401 for missing/invalid tokens
- Return 403 for authorization failures
- Use `@PreAuthorize` for method-level security

---

## Error Handling

**The agent MUST:**
- Create custom exceptions for each error case
- Use `@ControllerAdvice` with `@ExceptionHandler`
- Return standardized error response DTO
- Map exceptions to correct HTTP status codes

```kotlin
// Custom exceptions
class NotFoundException(resource: String, id: Long) :
    RuntimeException("$resource with id $id not found")

class ConflictException(message: String) : RuntimeException(message)

class AuthException(message: String) : RuntimeException(message)

// Global exception handler
@ControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(NotFoundException::class)
    fun handleNotFound(ex: NotFoundException, request: HttpServletRequest): ResponseEntity<ErrorResponse> {
        val error = ErrorResponse(
            timestamp = Instant.now(),
            status = HttpStatus.NOT_FOUND.value(),
            error = "Not Found",
            message = ex.message ?: "Resource not found",
            path = request.requestURI
        )
        return ResponseEntity(error, HttpStatus.NOT_FOUND)
    }

    // ... other handlers
}

// Standardized error response
data class ErrorResponse(
    val timestamp: Instant,
    val status: Int,
    val error: String,
    val message: String,
    val path: String
)
```

**HTTP Status Mapping:**
- 400 Bad Request — Validation errors, malformed input
- 401 Unauthorized — Missing/invalid token
- 403 Forbidden — Valid token, insufficient permissions
- 404 Not Found — Resource doesn't exist
- 409 Conflict — Duplicate resource, constraint violation
- 500 Internal Server Error — Unexpected errors

**The agent MUST NOT:**
- Return error codes as successful responses
- Expose stack traces in API responses
- Use generic `Exception` for business logic errors

---

## Validation

**The agent MUST:**
- Use Jakarta Bean Validation annotations
- Create custom validators for complex rules
- Return validation errors as 400 with field messages

```kotlin
// Request DTO with validation
data class NoteCreateRequestDto(
    @field:NotBlank(message = "Title is required")
    @field:Size(min = 1, max = 200, message = "Title must be between 1 and 200 characters")
    val title: String,

    @field:Size(max = 10000, message = "Content must not exceed 10000 characters")
    val content: String? = null,

    @field:Enumerated(EnumType.STRING)
    val color: NoteColor = NoteColor.GRAY,

    val tagIds: List<Long> = emptyList()
)
```

---

## Testing Strategy (Spring Boot Test)

**Unit Tests:**
- Use `@Test` with plain JUnit 5
- Mock dependencies with MockK
- Test service methods in isolation
- Focus on business logic

**Integration Tests:**
- Use `@SpringBootTest` for full context
- Use `@DataJpaTest` for repository tests
- Use `TestEntityManager` for database operations
- Use `@MockMvc` or `TestRestTemplate` for API tests
- Use `@TestConfiguration` for test-specific beans

**Test Database:**
- Configure H2 in-memory for tests (`application-test.yml`)
- Clean data between tests (`@DirtiesContext` or `@Transactional`)
- Use Testcontainers for PostgreSQL integration tests

**Coverage targets:**
- Services: 90%+
- Controllers: 80%+
- Repositories: 70%+ (mostly integration tested)

---

## Non-Functional Requirements

**Performance:**
- API response p95 < 500ms for CRUD operations
- API response p95 < 1000ms for search operations
- Use database indexes properly
- Avoid N+1 queries (use JOIN FETCH)
- Consider caching for frequently accessed data (Redis)

**Security:**
- Never log passwords or tokens
- Hash passwords with bcrypt (strength 10-12)
- Validate all input
- Use HTTPS in production
- Set security headers (CSP, X-Frame-Options, etc.)

**Observability:**
- Use SLF4J for logging
- Log important operations (register, login, errors)
- Don't log sensitive data
- Use structured logging (JSON in production)
- Expose metrics for Spring Boot Actuator

---

## Dependency Management

**Use these versions (unless specified otherwise):**
- Spring Boot: 3.2.0+
- Kotlin: 1.9.20+
- PostgreSQL Driver: latest
- Flyway: latest (managed by Spring Boot)
- SpringDoc OpenAPI: 2.x
- MockK: latest
- JUnit 5: latest (managed by Spring Boot)

**The agent MUST NOT:**
- Add dependencies without justification
- Use versions conflicting with Spring Boot BOM
- Ignore security vulnerabilities in dependencies

---

## Code Quality

**Kotlin style:**
- Use ktlint formatting (if configured)
- Follow Kotlin coding conventions
- Prefer expression over statement
- Use data classes for DTOs

**Spring Boot style:**
- Constructor injection (not field injection)
- `@Autowired` is optional with constructor injection
- Use `lateinit` or constructor injection for required dependencies
- Prefer composition over inheritance

---

## Forbidden Practices (Spring Boot Specific)

The agent MUST NOT:

- Use `@Autowired` on fields (use constructor injection)
- Use `new` for Spring-managed beans
- Put business logic in controllers
- Return entities from controllers (use DTOs)
- Use `System.out.println` (use logger)
- Hard-code configuration values (use application.yml)
- Ignore transaction boundaries
- Use `synchronized` for concurrency (use database transactions)
- Use `@Transactional` on controller methods
- Forget to handle lazy initialization exceptions

---

## Profile Authority

This profile:
- Extends `backend/AGENT_PROFILE_backend-base.md`
- Is specific to Kotlin + Spring Boot projects
- Overrides general backend rules where Spring Boot has specific patterns
- Is subordinate only to `ARCHITECTURE_OVERVIEW.md` and explicit roadmap instructions

Failure to comply with this profile is grounds for rejection during review or verification.

---

## Version

- **Version:** 1.0
- **Date:** 2025-01-24
- **Extends:** backend-base v1.0
