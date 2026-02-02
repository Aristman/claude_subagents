# Rust Profiles Registry

## Назначение документа

RUST_PROFILES.md описывает **все доступные профили для Rust-разработки**,
их иерархию, поддерживаемые технологии и правила выбора профиля для проекта.

Документ является:

- Справочником для Pipeline Orchestrator
- Контрактом для выбора правильного профиля
- Основой для добавления новых Rust-профилей

---

## Категории профилей

```
Rust Profiles
├── Game Development (Bevy Engine)
│   ├── rust-simcore (SIMCORE domain)
│   ├── rust-ai-engineer (AI domain)
│   └── rust-gameplay-engineer (GAME domain)
│
├── Desktop Applications (Tauri)
│   └── rust-tauri-backend (Backend for Tauri apps)
│
└── Specialized Components
    ├── rust-network-engineer (Networking)
    ├── rust-sqlite-storage (Storage)
    ├── rust-api-integration (API integration)
    └── rust-document-export (Document export)
```

---

## Выбор профиля

### Правило выбора через PROJECT_PROFILE

Профиль определяется через `PROJECT_PROFILE.domains`:

1. Считывается `Feature.Domain` из `FEATURES_INDEX.md`
2. По домену находится `Assigned Agent Profile` в `PROJECT_PROFILE.md`
3. Загружается соответствующий профиль из `~/.claude/agents/profiles/rust/`

**Пример:**

```yaml
# PROJECT_PROFILE.md
domains:
  - Domain ID: AI
    Responsibility: Robot Intelligence
    Assigned Agent Profile: rust-ai-engineer
```

Загружается `rust/AGENT_PROFILE_rust-ai-engineer.md`

---

## Описание профилей

### Game Development (Bevy Engine)

#### 1. rust-simcore

**Файл:** `rust/AGENT_PROFILE_SIMCORE.md`

**Domain:** SIMCORE (Simulation Core)

**Стек:**
- Language: Rust (stable)
- Engine: Bevy 0.14+ (ECS-driven)
- Platforms: Windows 10+, macOS 12+, Linux (Ubuntu 22.04+)

**Ответственность:** Ядро симуляции (физика, сущности, компоненты, системы)

**Для чего:** Базовый слой Bevy-игры

---

#### 2. rust-ai-engineer

**Файл:** `rust/AGENT_PROFILE_rust-ai-engineer.md`

**Domain:** AI (Robot Intelligence)

**Стек:**
- Language: Rust (stable)
- Engine: Bevy 0.14+
- AI: Custom feedforward neural network
- GA: In-house genetic algorithm

**Ответственность:** AI система роботов (neural networks, genetic algorithm, sensors)

**Для чего:** Интеллект роботов, эволюция, сенсорная система

---

#### 3. rust-gameplay-engineer

**Файл:** `rust/AGENT_PROFILE_rust-gameplay-engineer.md`

**Domain:** GAME (Gameplay Mechanics)

**Стек:**
- Language: Rust (stable)
- Engine: Bevy 0.14+
- Pattern: Entity Component System

**Ответственность:** Игровая логика (фабрика, исследования, мутации, дикие фабрики)

**Для чего:** Игровые механики, производство, исследования

---

### Desktop Applications (Tauri)

#### 4. rust-tauri-backend

**Файл:** `rust/AGENT_PROFILE_rust-tauri-backend.md`

**Domain:** Desktop Backend

**Стек:**
- Language: Rust (stable)
- Framework: Tauri 1.x / 2.x
- Frontend: React/Vue/Svelte (веб-технологии)

**Ответственность:** Backend для десктопных приложений

**Для чего:** Кроссплатформенные десктопные приложения

---

### Specialized Components

#### 5. rust-network-engineer

**Файл:** `rust/AGENT_PROFILE_rust-network-engineer.md`

**Domain:** NET (Networking)

**Стек:**
- Language: Rust (stable)
- Runtime: Tokio (full features)
- Serialization: serde + bincode

**Ответственность:** Сетевой слой (сервер, протокол, 24/7 симуляция, персистентность)

**Для чего:** Сетевое взаимодействие, персистентность состояния

---

#### 6. rust-sqlite-storage

**Файл:** `rust/AGENT_PROFILE_rust-sqlite-storage.md`

**Domain:** Storage

**Стек:**
- Language: Rust (stable)
- Database: SQLite (rusqlite)
- Migrations: Собственная система

**Ответственность:** Локальное хранение данных

**Для чего:** Локальные базы данных, кэширование

---

#### 7. rust-api-integration

**Файл:** `rust/AGENT_PROFILE_rust-api-integration.md`

**Domain:** Integration

**Стек:**
- Language: Rust (stable)
- HTTP: Reqwest / Hyper
- Async: Tokio

**Ответственность:** Интеграция с внешними API

**Для чего:** HTTP клиенты, вебхуки, интеграции

---

#### 8. rust-document-export

**Файл:** `rust/AGENT_PROFILE_rust-document-export.md`

**Domain:** Export

**Стек:**
- Language: Rust (stable)
- Formats: PDF, DOCX, XLSX

**Ответственность:** Экспорт документов в различные форматы

**Для чего:** Генерация отчётов, экспорт данных

---

## Общие правила для всех Rust профилей

### MUST Follow

1. **Rust Edition:** 2021 или позднее
2. **Error Handling:** `Result<T, E>` для fallible operations
3. **Unsafe:** Минимум, с документацией `# Safety`
4. **Testing:** Unit + интеграционные тесты
5. **Documentation:** rustdoc для всех public API
6. **Clippy:** < 5 warnings

### MUST NOT Do

1. `.unwrap()` / `.expect()` в production коде
2. Блокирующие операции в async контексте
3. Memory leaks (корректная работа с жизненным циклом)
4. Ignoring warnings компилятора

### Качество кода

- Покрытие тестами: >= 80% для бизнес-логики
- rustdoc покрытие: >= 70%
- Zero panics в production коде
- Graceful degradation при ошибках

---

## Сравнительная таблица специализированных профилей

| Профиль | Domain | Основной фокус | Ключевые технологии |
|---------|--------|----------------|---------------------|
| `rust-simcore` | SIMCORE | Физическая симуляция | Bevy ECS, Rapier |
| `rust-ai-engineer` | AI | Нейросети + генетика | Custom NN, GA |
| `rust-gameplay-engineer` | GAME | Игровые механики | Bevy ECS |
| `rust-tauri-backend` | Desktop | Десктоп приложения | Tauri |
| `rust-network-engineer` | NET | Сеть + персистентность | Tokio, bincode |
| `rust-sqlite-storage` | Storage | Локальное хранение | SQLite, rusqlite |
| `rust-api-integration` | Integration | Внешние API | Reqwest, HTTP |
| `rust-document-export` | Export | Экспорт документов | PDF, DOCX |

---

## Добавление нового профиля

Для добавления нового Rust-профиля:

1. Создать файл `rust/AGENT_PROFILE_<name>.md`
2. Указать:
   - Domain (область ответственности)
   - Stack (язык, фреймворки, версии)
   - Responsibilities (что входит в зону ответственности)
   - Out of Scope ( что делегируется другим профилям)
   - Performance Constraints
   - Quality Target
3. Добавить запись в этот реестр
4. Обновить версию документа

---

## Правила для execution-агентов

### Developer Agent

**Обязан:**

1. Определить профиль через `Feature.Domain`
2. Загрузить профиль из `~/.claude/agents/profiles/rust/AGENT_PROFILE_<profile>.md`
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
3. Проверять соответствие покрытия тестами

### Code Reviewer Agent

**Обязан:**

1. Проверять соответствие AGENT_PROFILE
2. Проверять соблюдение forbidden practices
3. Проверять соответствие Rust идиомам

---

## Взаимодействие с backend профилями

Для server-side Rust проектов можно использовать:

**`backend/AGENT_PROFILE_rust.md`** — общий профиль для backend-сервисов на Rust

Отличия от специализированных профилей:
- Backend профиль фокусируется на API, бизнес-логике, данных
- Специализированные профили фокусируются на конкретных доменах (AI, GAME, NET)

Выбор зависит от типа проекта:
- Серверный API → `backend/rust`
- Игровой движок → `rust/simcore`, `rust/ai-engineer`, `rust/gameplay-engineer`
- Десктопное приложение → `rust/rust-tauri-backend`

---

## Версия документа

- **Версия:** 1.0
- **Дата:** 2025-01-31
- **Статус:** Production-ready

---

## Связанные документы

- `~/.claude/agents/AGENTS_INDEX.md` — реестр агентов
- `~/.claude/agents/profiles/backend/BACKEND_PROFILES.md` — backend профили
- `PROJECT_PROFILE.md` — профиль проекта
- `PIPELINE_PROMPT.md` — мастер-промпт пайплайна
