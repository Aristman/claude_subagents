# Backend Profiles Registry

## Назначение документа

BACKEND_PROFILES.md описывает **все доступные профили для backend-разработки**,
их иерархию, поддерживаемые технологии и правила выбора профиля для проекта.

Документ является:

- Справочником для Pipeline Orchestrator
- Контрактом для выбора правильного профиля
- Основой для добавления новых backend-профилей

---

## Иерархия профилей

```
backend-base (базовый профиль)
    ├── spring-boot (Kotlin + Spring Boot)
    ├── nodejs (TypeScript + Node.js)
    ├── python (Python + Django/FastAPI)
    └── rust (Rust + Actix/Axum)
```

**Правило:** Все специализированные профили наследуются от `backend-base` и
добавляют специфичные правила для своего стека.

---

## Выбор профиля

### Правило выбора через PROJECT_PROFILE

Профиль определяется через `PROJECT_PROFILE.domains`:

1. Считывается `Feature.Domain` из `FEATURES_INDEX.md`
2. По домену находится `Assigned Agent Profile` в `PROJECT_PROFILE.md`
3. Загружается соответствующий профиль из `~/.claude/agents/profiles/backend/`

**Пример:**

```yaml
# PROJECT_PROFILE.md
domains:
  - Domain ID: NOTES
    Responsibility: Управление заметками
    Assigned Agent Profile: spring-boot
```

→ Загружается `backend/AGENT_PROFILE_spring-boot.md`

---

## Описание профилей

### 1. backend-base

**Файл:** `backend/AGENT_PROFILE_backend-base.md`

**Описание:** Базовый профиль, определяющий универсальные backend-правила.

**Применяется:**
- Как базовый профиль для всех специализированных профилей
- Для generic backend-проектов без явного стека

**Содержит:**
- Архитектурные паттерны (layered architecture)
- Управление данными и состоянием
- Обработка ошибок
- Стратегию тестирования
- Нефункциональные приоритеты
- Безопасность
- API design principles
- Запрещённые практики

**НЕ содержит:**
- Специфичные для языка правила
- Специфичные фреймворки
- Конкретные библиотеки

---

### 2. spring-boot

**Файл:** `backend/AGENT_PROFILE_spring-boot.md`

**Наследует:** `backend-base`

**Стек:**
- Runtime: JVM (JDK 17 or 21 LTS)
- Language: Kotlin 1.9+
- Framework: Spring Boot 3.2+
- Build: Gradle (Kotlin DSL)
- Database: JPA/Hibernate + PostgreSQL
- Migrations: Flyway
- Testing: JUnit 5 + MockK

**Содержит специфичные правила:**
- Spring annotations (@RestController, @Service, @Repository)
- Project structure (controller, service, repository, entity, dto)
- Kotlin idioms (val over var, null safety, data classes)
- JPA entities and relationships
- Spring Security + JWT
- Error handling with @ControllerAdvice
- Validation (Jakarta Bean Validation)
- Testing strategy (@SpringBootTest, @DataJpaTest)

**Для каких проектов:**
- REST API сервисы
- Микросервисы
- Enterprise приложения
- Проекты, требующие строгой типизации

---

### 3. nodejs

**Файл:** `backend/AGENT_PROFILE_nodejs.md`

**Наследует:** `backend-base`

**Стек:**
- Runtime: Node.js 20 LTS (or 18 LTS)
- Language: TypeScript 5.x
- Framework: Express.js или Fastify
- Package manager: pnpm (preferred) или npm
- Database: PostgreSQL (pg, Knex, Prisma) или MongoDB (Mongoose)
- Testing: Vitest или Jest

**Содержит специфичные правила:**
- TypeScript strict mode
- Project structure (controllers, services, repositories, routes)
- Express middleware
- Error handling middleware
- Async/await patterns
- Dependency injection (constructor-based)
- Validation (Zod, Joi, class-validator)
- Testing (Vitest, Supertest)

**Для каких проектов:**
- REST API сервисы
- GraphQL API
- Real-time приложения (WebSocket)
- Serverless функции
- Microservices

---

### 4. python

**Файл:** `backend/AGENT_PROFILE_python.md`

**Наследует:** `backend-base`

**Стек:**
- Runtime: Python 3.11+ (3.12+ recommended)
- Framework: Django 5.x **ИЛИ** FastAPI 0.100+
- Package manager: Poetry (preferred) или uv
- Database: PostgreSQL + SQLAlchemy (FastAPI) или Django ORM
- Migrations: Alembic (FastAPI) или Django migrations
- Testing: pytest

**Содержит специфичные правила:**
- Type hints (mypy strict mode)
- Django patterns (models, views, serializers, services)
- FastAPI patterns (Pydantic schemas, dependency injection)
- Async/await (FastAPI)
- ORM usage (SQLAlchemy или Django ORM)
- Security (JWT, bcrypt)
- Testing (pytest, pytest-asyncio)

**Для каких проектов:**
- REST API (FastAPI)
- Монолитные веб-приложения (Django)
- Data-heavy приложения
- ML/AI сервисы

---

### 5. rust

**Файл:** `backend/AGENT_PROFILE_rust.md`

**Наследует:** `backend-base`

**Стек:**
- Runtime: Rust 1.75+ (latest stable)
- Framework: Actix Web 4.x **ИЛИ** Axum 0.7+
- Async runtime: Tokio 1.x
- Database: SQLX (compile-time checked queries)
- Testing: Built-in test framework + mockall

**Содержит специфичные правила:**
- Ownership и borrowing
- Error handling (Result<T, E>, Option<T>)
- Async/await (.await)
- Trait objects для dependency injection
- Shared state (Arc<Mutex<T>>, Arc<RwLock<T>>)
- SQLX для type-safe queries
- Testing strategy (unit + integration)

**Для каких проектов:**
- High-performance API
- Systems programming
- Memory-critical приложения
- Microservices с высокой нагрузкой

---

## Сравнительная таблица

| Характеристика | spring-boot | nodejs | python | rust |
|----------------|-------------|--------|--------|------|
| **Язык** | Kotlin | TypeScript | Python | Rust |
| **Типизация** | Строгая (compile-time) | Строгая (compile-time) | Динамическая + hints | Строгая (compile-time) |
| **Runtime** | JVM | V8 | CPython | Native |
| **Асинхронность** | WebFlux / Coroutines | Event loop | asyncio | Tokio |
| **Производительность** | Высокая | Средняя | Средняя | Очень высокая |
| **Потребление памяти** | Среднее-высокое | Среднее | Среднее-низкое | Очень низкое |
| **Стартап время** | Среднее | Быстрое | Быстрое | Очень быстрое |
| **Экосистема** | Очень богатая | Очень богатая | Очень богатая | Растущая |
| **Learning curve** | Средняя | Низкая | Низкая | Высокая |

---

## Добавление нового профиля

Для добавления нового backend-профиля:

1. Создать файл `backend/AGENT_PROFILE_<new-stack>.md`
2. Унаследовать от `backend-base`
3. Описать:
   - Stack (язык, runtime, фреймворк, версия)
   - Project structure
   - Специфичные паттерны
   - Testing strategy
   - Forbidden practices
4. Добавить запись в этот реестр
5. Обновить версию документа

---

## Правила для execution-агентов

### Developer Agent

**Обязан:**

1. Определить профиль через `Feature.Domain`
2. Загрузить профиль из `~/.claude/agents/profiles/backend/AGENT_PROFILE_<profile>.md`
3. Соблюдать ВСЕ правила профиля
4. Hard-fail при отсутствии профиля

**НЕ должен:**

1. Использовать другой профиль без явного указания
2. Игнорировать правила профиля
3. Смешивать стеки без архитектурного обоснования

### Test Engineer Agent

**Обязан:**

1. Использовать тот же профиль, что и Developer Agent
2. Следовать testing strategy из профиля
3. Проверять соответствие покрытия тестами (coverage targets)

### Code Reviewer Agent

**Обязан:**

1. Проверять соответствие AGENT_PROFILE
2. Проверять соблюдение forbidden practices
3. Проверять соответствие коду стилю профиля

---

## Mapping к старым профилям

Для обратной совместимости:

| Старый профиль | Новый профиль | Действие |
|----------------|---------------|----------|
| `AGENT_PROFILE_backend.md` | `backend/AGENT_PROFILE_backend-base.md` | Устарел, удалить |
| `AGENT_PROFILE_backend-spring-boot-developer.md` | `backend/AGENT_PROFILE_spring-boot.md` | Перенести, удалить старый |

---

## Обслуживание

### Ответственный

Обновление профилей — ответственность владельца каждого стека.

### Версионирование

Каждый профиль имеет свою версию.
Изменения в `backend-base` требуют пересмотра всех специализированных профилей.

---

## Версия документа

- **Версия:** 1.0
- **Дата:** 2025-01-24
- **Статус:** Production-ready

---

## Связанные документы

- `~/.claude/agents/AGENTS_INDEX.md` — реестр агентов
- `PROJECT_PROFILE.md` — профиль проекта
- `PIPELINE_PROMPT.md` — мастер-промпт пайплайна
