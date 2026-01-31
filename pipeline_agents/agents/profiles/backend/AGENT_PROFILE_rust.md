# Agent Profile: Backend Rust

## Profile Identity

- Profile ID: rust
- Domain: backend
- Stack: Rust
- Extends: `backend/AGENT_PROFILE_backend-base.md`
- Scope: Rust applications with Actix Web or Axum
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent

This profile is **mandatory** for any feature assigned to the `backend` domain with **Rust** stack.

---

## Inheritance

This profile extends `backend/AGENT_PROFILE_backend-base.md` with Rust specific rules.

General backend rules (layered architecture, error handling, testing, observability) still apply.

---

## Supported Languages and Frameworks

**Mandatory:**
- Language: Rust 1.75+ (latest stable)
- Framework: Actix Web 4.x OR Axum 0.7+
- Async runtime: Tokio 1.x
- Package manager: cargo

**The agent MUST NOT:**
- Use nightly Rust without explicit justification
- Mix frameworks without clear reason
- Use deprecated async runtimes (async-std)

---

## Project Structure

```
project/
├── Cargo.toml
├── Cargo.lock
├── src/
│   ├── main.rs              # Application entry
│   ├── lib.rs               # Library exports
│   ├── config/              # Configuration
│   │   ├── mod.rs
│   │   └── settings.rs
│   ├── handlers/            # HTTP handlers
│   │   ├── mod.rs
│   │   ├── notes.rs
│   │   └── auth.rs
│   ├── services/            # Business logic
│   │   ├── mod.rs
│   │   ├── note_service.rs
│   │   └── auth_service.rs
│   ├── repositories/        # Data access
│   │   ├── mod.rs
│   │   └── note_repository.rs
│   ├── models/              # Domain models
│   │   ├── mod.rs
│   │   ├── note.rs
│   │   └── user.rs
│   ├── dto/                 # Request/Response types
│   │   ├── mod.rs
│   │   ├── request.rs
│   │   └── response.rs
│   ├── errors/              # Error types
│   │   ├── mod.rs
│   │   └── app_error.rs
│   ├── middleware/          # Custom middleware
│   │   ├── mod.rs
│   │   └── auth.rs
│   └── db/                  # Database setup
│       ├── mod.rs
│       └── connection.rs
├── migrations/              # Database migrations
├── tests/                   # Integration tests
└── sqlx-data.json          # SQLx query data (if using sqlx)
```

---

## Actix Web Specific Patterns

### Handlers

```rust
// src/handlers/notes.rs
use actix_web::{web, HttpResponse, Responder};
use serde::{Deserialize, Serialize};
use crate::services::note_service::NoteService;
use crate::dto::request::{NoteCreateRequest, NoteUpdateRequest};
use crate::errors::AppError;

pub async fn get_all(
    service: web::Data<dyn NoteService>,
) -> impl Responder {
    match service.find_all().await {
        Ok(notes) => HttpResponse::Ok().json(notes),
        Err(e) => e.into(),
    }
}

pub async fn get_by_id(
    id: web::Path<i64>,
    service: web::Data<dyn NoteService>,
) -> impl Responder {
    match service.find_by_id(*id).await {
        Ok(Some(note)) => HttpResponse::Ok().json(note),
        Ok(None) => HttpResponse::NotFound().json(serde_json::json!({
            "error": "Note not found"
        })),
        Err(e) => e.into(),
    }
}

pub async fn create(
    req: web::Json<NoteCreateRequest>,
    service: web::Data<dyn NoteService>,
) -> impl Responder {
    match service.create(req.into_inner()).await {
        Ok(note) => HttpResponse::Created().json(note),
        Err(e) => e.into(),
    }
}

pub async fn update(
    id: web::Path<i64>,
    req: web::Json<NoteUpdateRequest>,
    service: web::Data<dyn NoteService>,
) -> impl Responder {
    match service.update(*id, req.into_inner()).await {
        Ok(note) => HttpResponse::Ok().json(note),
        Err(e) => e.into(),
    }
}

pub async fn delete(
    id: web::Path<i64>,
    service: web::Data<dyn NoteService>,
) -> impl Responder {
    match service.delete(*id).await {
        Ok(_) => HttpResponse::NoContent().finish(),
        Err(e) => e.into(),
    }
}
```

### Route Configuration

```rust
// src/main.rs
use actix_web::{web, App, HttpServer};
use actix_cors::Cors;
use crate::handlers::notes;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // Initialize services, database, etc.
    let note_service = web::Data::new(/* ... */);

    HttpServer::new(move || {
        App::new()
            .wrap(
                Cors::permissive()
                    .allowed_origin("http://localhost:3000")
                    .allowed_methods(vec!["GET", "POST", "PUT", "DELETE"])
                    .allowed_header(actix_web::http::header::CONTENT_TYPE)
            )
            .app_data(note_service.clone())
            .service(
                web::scope("/api/notes")
                    .route("", web::get().to(notes::get_all))
                    .route("", web::post().to(notes::create))
                    .route("/{id}", web::get().to(notes::get_by_id))
                    .route("/{id}", web::put().to(notes::update))
                    .route("/{id}", web::delete().to(notes::delete))
            )
    })
    .bind(("0.0.0.0", 8080))?
    .run()
    .await
}
```

---

## Axum Specific Patterns

### Handlers

```rust
// src/handlers/notes.rs
use axum::{
    extract::{Path, State},
    http::StatusCode,
    Json,
};
use serde::{Deserialize, Serialize};
use crate::services::note_service::NoteService;
use crate::dto::request::{NoteCreateRequest, NoteUpdateRequest};
use crate::dto::response::NoteResponse;

pub async fn get_all(
    State(service): State<dyn NoteService>,
) -> Result<Json<Vec<NoteResponse>>, AppError> {
    let notes = service.find_all().await?;
    Ok(Json(notes))
}

pub async fn get_by_id(
    Path(id): Path<i64>,
    State(service): State<dyn NoteService>,
) -> Result<Json<NoteResponse>, AppError> {
    let note = service.find_by_id(id).await?
        .ok_or(AppError::NotFound("Note not found".into()))?;
    Ok(Json(note))
}

pub async fn create(
    State(service): State<dyn NoteService>,
    Json(req): Json<NoteCreateRequest>,
) -> Result<(StatusCode, Json<NoteResponse>), AppError> {
    let note = service.create(req).await?;
    Ok((StatusCode::CREATED, Json(note)))
}

pub async fn update(
    Path(id): Path<i64>,
    State(service): State<dyn NoteService>,
    Json(req): Json<NoteUpdateRequest>,
) -> Result<Json<NoteResponse>, AppError> {
    let note = service.update(id, req).await?;
    Ok(Json(note))
}

pub async fn delete(
    Path(id): Path<i64>,
    State(service): State<dyn NoteService>,
) -> Result<StatusCode, AppError> {
    service.delete(id).await?;
    Ok(StatusCode::NO_CONTENT)
}
```

### Route Configuration

```rust
// src/main.rs
use axum::{
    routing::{get, post, put, delete},
    Router,
};
use tower_http::cors::{CorsLayer, Any};
use crate::handlers::notes;

#[tokio::main]
async fn main() {
    let note_service: Arc<dyn NoteService> = Arc::new(/* ... */);

    let app = Router::new()
        .nest(
            "/api/notes",
            Router::new()
                .route("/", get(notes::get_all).post(notes::create))
                .route("/:id", get(notes::get_by_id).put(notes::update).delete(notes::delete))
        )
        .layer(CorsLayer::new().allow_origin(Any).allow_methods(Any))
        .with_state(note_service);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:8080")
        .await
        .unwrap();

    axum::serve(listener, app)
        .await
        .unwrap();
}
```

---

## Rust Specific Rules

**The agent MUST:**
- Use `Result<T, E>` for fallible operations
- Use `Option<T>` for nullable values
- Use `?` operator for error propagation
- Prefer `async`/`await` over manual futures
- Use trait objects (`dyn Trait`) for dependency injection
- Use `derive` macros for common traits
- Use `Arc<Mutex<T>>` or `tokio::sync::RwLock<T>` for shared state
- Use `tokio::spawn` for concurrent tasks

**The agent MUST NOT:**
- Use `.unwrap()` or `.expect()` in production code (use `?`)
- Use `panic!` for expected errors
- Use `Rc<RefCell<T>>` in async context (use `Arc`)
- Block on async with `.await` in loop
- Ignore compiler warnings
- Use unsafe without explicit justification

---

## Models

```rust
// src/models/note.rs
use chrono::{DateTime, Utc};
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize, sqlx::FromRow)]
pub struct Note {
    pub id: i64,
    pub user_id: i64,
    pub title: String,
    pub content: Option<String>,
    #[serde(rename = "createdAt")]
    pub created_at: DateTime<Utc>,
    #[serde(rename = "updatedAt")]
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize, sqlx::FromRow)]
pub struct NoteWithTags {
    #[sqlx(flatten)]
    pub note: Note,
    pub tags: Vec<Tag>,
}

#[derive(Debug, Clone, Copy, Serialize, Deserialize, sqlx::Type)]
#[repr(i32)]
pub enum NoteColor {
    Red = 0,
    Orange = 1,
    Yellow = 2,
    Green = 3,
    Blue = 4,
    Purple = 5,
    Pink = 6,
    Gray = 7,
}

impl Default for NoteColor {
    fn default() -> Self {
        NoteColor::Gray
    }
}
```

---

## DTOs

```rust
// src/dto/request.rs
use serde::{Deserialize, Serialize};
use validator::Validate;

#[derive(Debug, Deserialize, Validate)]
pub struct NoteCreateRequest {
    #[validate(length(min = 1, max = 200))]
    pub title: String,
    #[validate(length(max = 10000))]
    pub content: Option<String>,
    pub color: Option<NoteColor>,
    pub tag_ids: Vec<i64>,
}

#[derive(Debug, Deserialize, Validate)]
pub struct NoteUpdateRequest {
    #[validate(length(min = 1, max = 200))]
    pub title: Option<String>,
    #[validate(length(max = 10000))]
    pub content: Option<String>,
    pub color: Option<NoteColor>,
}

// src/dto/response.rs
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize)]
pub struct NoteResponse {
    pub id: i64,
    pub title: String,
    pub content: Option<String>,
    pub color: NoteColor,
    #[serde(rename = "createdAt")]
    pub created_at: DateTime<Utc>,
    #[serde(rename = "updatedAt")]
    pub updated_at: DateTime<Utc>,
}
```

---

## Services

```rust
// src/services/note_service.rs
use async_trait::async_trait;
use crate::dto::request::{NoteCreateRequest, NoteUpdateRequest};
use crate::dto::response::NoteResponse;
use crate::errors::AppError;

#[async_trait]
pub trait NoteService: Send + Sync {
    async fn find_all(&self) -> Result<Vec<NoteResponse>, AppError>;
    async fn find_by_id(&self, id: i64) -> Result<Option<NoteResponse>, AppError>;
    async fn create(&self, req: NoteCreateRequest) -> Result<NoteResponse, AppError>;
    async fn update(&self, id: i64, req: NoteUpdateRequest) -> Result<NoteResponse, AppError>;
    async fn delete(&self, id: i64) -> Result<(), AppError>;
}

pub struct NoteServiceImpl {
    repo: PgPool,
}

#[async_trait]
impl NoteService for NoteServiceImpl {
    async fn find_all(&self) -> Result<Vec<NoteResponse>, AppError> {
        let notes = sqlx::query_as::<_, Note>("SELECT * FROM notes ORDER BY created_at DESC")
            .fetch_all(&self.repo)
            .await?;

        Ok(notes.into_iter().map(|n| n.to_response()).collect())
    }

    async fn find_by_id(&self, id: i64) -> Result<Option<NoteResponse>, AppError> {
        let note = sqlx::query_as::<_, Note>("SELECT * FROM notes WHERE id = $1")
            .bind(id)
            .fetch_optional(&self.repo)
            .await?;

        Ok(note.map(|n| n.to_response()))
    }

    // ... other methods
}
```

---

## Error Handling

```rust
// src/errors/app_error.rs
use actix_web::{error::ResponseError, http::StatusCode, HttpResponse};
use serde::Serialize;

#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),

    #[error("Not found: {0}")]
    NotFound(String),

    #[error("Conflict: {0}")]
    Conflict(String),

    #[error("Unauthorized")]
    Unauthorized,

    #[error("Validation error: {0}")]
    Validation(String),
}

impl Serialize for AppError {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: serde::ser::Serializer,
    {
        use serde::ser::SerializeStruct;

        let status = self.status_code();
        let mut s = serializer.serialize_struct("ErrorResponse", 5)?;
        s.serialize_field("timestamp", &chrono::Utc::now().to_rfc3339())?;
        s.serialize_field("status", &status.as_u16())?;
        s.serialize_field("error", &status.canonical_reason().unwrap_or("Error"))?;
        s.serialize_field("message", &self.to_string())?;
        s.end()
    }
}

impl ResponseError for AppError {
    fn status_code(&self) -> StatusCode {
        match self {
            AppError::Database(_) => StatusCode::INTERNAL_SERVER_ERROR,
            AppError::NotFound(_) => StatusCode::NOT_FOUND,
            AppError::Conflict(_) => StatusCode::CONFLICT,
            AppError::Unauthorized => StatusCode::UNAUTHORIZED,
            AppError::Validation(_) => StatusCode::BAD_REQUEST,
        }
    }

    fn error_response(&self) -> HttpResponse {
        HttpResponse::build(self.status_code()).json(self)
    }
}
```

---

## Security

**The agent MUST:**
- Use `bcrypt` or `argon2` for password hashing
- Use `jsonwebtoken` for JWT
- Validate ALL input
- Use parameterized queries (SQLX does this)
- Never log sensitive data

**Example JWT validation:**

```rust
// src/middleware/auth.rs
use actix_web::{dev::Payload, Error, FromRequest, HttpRequest};
use futures::future::{ready, Ready};
use jsonwebtoken::{decode, Validation, DecodingKey};

pub struct AuthenticatedUser {
    pub user_id: i64,
}

impl FromRequest for AuthenticatedUser {
    type Error = Error;
    type Future = Ready<Result<Self, Error>>;

    fn from_request(req: &HttpRequest, _: &mut Payload) -> Self::Future {
        let auth_header = match req.headers().get("Authorization") {
            Some(header) => header,
            None => return ready(Err(AppError::Unauthorized.into())),
        };

        let token = match auth_header.to_str() {
            Ok(token) if token.starts_with("Bearer ") => &token[7..],
            _ => return ready(Err(AppError::Unauthorized.into())),
        };

        match decode_token(token) {
            Ok(claims) => ready(Ok(AuthenticatedUser { user_id: claims.user_id })),
            Err(_) => ready(Err(AppError::Unauthorized.into())),
        }
    }
}
```

---

## Testing Strategy

**Unit Tests:**

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use mockall::{predicate::*, Mock};

    #[tokio::test]
    async fn test_find_by_id_found() {
        let mut mock_service = MockNoteService::new();
        mock_service
            .expect_find_by_id()
            .with(eq(1))
            .returning(|_| Ok(Some(NoteResponse { /* ... */ })));

        let result = mock_service.find_by_id(1).await;
        assert!(result.is_ok());
        assert!(result.unwrap().is_some());
    }
}
```

**Integration Tests:**

```rust
// tests/integration_tests.rs
use axum::http::StatusCode;
use serde_json::json;

#[tokio::test]
async fn test_create_note() {
    let app = create_test_app().await;

    let response = app
        .oneshot(
            Request::builder()
                .method("POST")
                .uri("/api/notes")
                .header("Content-Type", "application/json")
                .body(Body::from(json!({"title": "Test"}).to_string()))
                .unwrap(),
        )
        .await
        .unwrap();

    assert_eq!(response.status(), StatusCode::CREATED);
}
```

**Coverage targets:**
- Services: 90%+
- Handlers: 80%+

---

## Database

**The agent MUST:**
- Use SQLX for type-safe database access
- Use migrations
- Use connection pooling
- Use transactions for multi-step operations

```rust
// migrations/V001__create_notes.sql
CREATE TABLE notes (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id),
    title VARCHAR(200) NOT NULL,
    content TEXT,
    color VARCHAR(20) DEFAULT 'GRAY',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notes_user_id ON notes(user_id);
CREATE INDEX idx_notes_created_at ON notes(created_at DESC);
```

---

## Non-Functional Requirements

**Performance:**
- Use connection pooling (SQLX does this)
- Use async/await throughout
- Consider caching (Redis) for hot paths
- Profile before optimizing

**Observability:**
- Use tracing crate for structured logging
- Use metrics (prometheus crate)
- Log important operations

**Code Quality:**
- Use `cargo fmt` for formatting
- Use `cargo clippy` for lints
- Fix all warnings
- Use `cargo test` for testing

---

## Dependency Management

**The agent MUST:**
- Use `Cargo.toml` for dependencies
- Specify versions carefully
- Run `cargo audit` for vulnerabilities
- Keep dependencies up to date

**The agent MUST NOT:**
- Add dependencies without justification
- Use git dependencies unless necessary
- Ignore security advisories

---

## Forbidden Practices (Rust Specific)

The agent MUST NOT:

- Use `.unwrap()` in production code
- Use `.expect()` in production code
- Ignore compiler warnings
- Use `unsafe` without explicit justification
- Block on async in event loop
- Use `Rc<RefCell<T>>` in async context
- Use `std::time::Instant` for timeout (use tokio::time::Instant)
- Use `panic!` for expected errors
- Create deadlocks with mutex ordering
- Forget `Send` + `Sync` bounds for shared state

---

## Profile Authority

This profile:
- Extends `backend/AGENT_PROFILE_backend-base.md`
- Is specific to Rust projects (Actix Web or Axum)
- Overrides general backend rules where Rust has specific patterns
- Is subordinate only to `ARCHITECTURE_OVERVIEW.md` and explicit roadmap instructions

Failure to comply with this profile is grounds for rejection during review or verification.

---

## Version

- **Version:** 1.0
- **Date:** 2025-01-24
- **Extends:** backend-base v1.0
