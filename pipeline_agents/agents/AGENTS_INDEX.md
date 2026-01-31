# AGENTS_INDEX.md

## Назначение документа

AGENTS_INDEX.md описывает **полный состав агентов мультиагентского пайплайна**,
их роли, зоны ответственности, уровень profile-awareness и правила взаимодействия
с артефактами.

Документ является:

- контрактом взаимодействия агентов
- справочником для Pipeline Orchestrator
- контрольной точкой для human-in-the-loop
- основой для расширения системы новыми агентами

---

## Базовая модель агентов

Каждый агент определяется формулой:

Agent = Role + Profile Awareness + Artifact Contracts

Где:

- **Role** — неизменная ответственность агента
- **Profile Awareness** — правило работы с доменными профилями
- **Artifacts** — входные и выходные документы

---

## Разделение сущностей (КРИТИЧНО)

| Сущность                              | Регистрируется как агент | Исполняется |
|---------------------------------------|--------------------------|-------------|
| Agent (`developer-agent.md`)          | Да                       | Да          |
| Profile (`AGENT_PROFILE_*.md`)        | Нет                      | Нет         |
| Artifact (`ARCHITECTURE_OVERVIEW.md`) | Нет                      | Нет         |

**AGENT_PROFILE_*.md — это артефакты-конфиги, а не агенты.**

---

## Global Profile Rule (MANDATORY)

Для всех execution-агентов действует единое правило:

- Agent profiles MUST be loaded from:
  `~/.claude/agents/profiles/<category>/AGENT_PROFILE_<profile>.md`
- Profiles MUST NOT be loaded from project workspace
- Absence or unreadability of profile is a **fatal error**
- Profiles are selected via:
  `Feature.Domain → PROJECT_PROFILE.domains`

**Категории профилей:**

| Категория | Путь | Реестр |
|-----------|------|--------|
| Backend | `~/.claude/agents/profiles/backend/AGENT_PROFILE_*.md` | `backend/BACKEND_PROFILES.md` |
| Mobile | `~/.claude/agents/profiles/AGENT_PROFILE_mobile-*.md` | (будет добавлен) |
| Web | `~/.claude/agents/profiles/AGENT_PROFILE_web.md` | (будет добавлен) |
| CLI | `~/.claude/agents/profiles/AGENT_PROFILE_cli.md` | (будет добавлен) |

**Доступные backend профили:**
- `backend-base` — базовый профиль для всех backend стеков
- `spring-boot` — Kotlin + Spring Boot
- `nodejs` — TypeScript + Node.js (Express/Fastify)
- `python` — Python + Django/FastAPI
- `rust` — Rust + Actix/Axum

---

## Примечание по оркестрации

**Встроенный оркестратор Claude Code** (а не отдельный агент) отвечает за:
- управление пайплайном
- запуск и координацию агентов
- enforcement quality gates
- human-in-the-loop

Скилл `product-creator` превращает встроенный оркестратор в режим управления пайплайном.

---

## Pre-Pipeline агенты (инициализация)

### Project Profile Generator Agent

- **ID:** project-profile-generator
- **Role:** формализация человеческого намерения
- **Profile-aware:** ❌ нет

**Выход:**

- PROJECT_PROFILE.md (internal)
- PROJECT_PROFILE_HUMAN.md (human-readable)

---

### Pipeline Prompt Generator Agent

- **ID:** pipeline-prompt-generator
- **Role:** генерация master pipeline prompt
- **Profile-aware:** ❌ нет

**Выход:**

- PIPELINE_PROMPT.md

---

## Stage 1 — Аналитика

### Research Agent

- **ID:** research-agent
- **Role:** исследование и анализ контекста
- **Profile-aware:** ❌ нет

**Выход:**

- ANALYSIS.md

---

### System Analyst Agent

- **ID:** system-analyst
- **Role:** формализация требований и scope
- **Profile-aware:** ⚠️ частично

**Учитывает профиль:**

- нефункциональные приоритеты
- платформенные ограничения

**Выход:**

- TECH_REQUIREMENTS.md
- SCOPE.md

---

## Stage 2 — Архитектура и декомпозиция

### Solution Architect Agent

- **ID:** solution-architect
- **Role:** проектирование архитектуры
- **Profile-aware:** ✅ обязательно

**Выход:**

- ARCHITECTURE_OVERVIEW.md

---

### Feature Decomposition Agent

- **ID:** feature-decomposer
- **Role:** декомпозиция на фичи и стадии
- **Profile-aware:** ❌ нет

**Выход:**

- WORK_BREAKDOWN.md
- FEATURES_INDEX.md  
  (каждая фича обязана иметь поле `Domain`)

---

## Stage 3 — Планирование (TDD)

### TDD Planner Agent

- **ID:** tdd-planner
- **Role:** пофичевые TDD-роадмапы
- **Profile-aware:** ✅ обязательно

**Правило выбора профиля:**

- profile определяется через `Feature.Domain`
- разрешается через `PROJECT_PROFILE.domains`
- глобальные профили запрещены

**Выход:**

- ROADMAP_<feature>.md

---

## Stage 4 — Разработка (Execution)

### Developer Agent

- **ID:** developer-<feature>
- **Role:** реализация фичи
- **Profile-aware:** ✅ обязательно

**Обязан:**

- загрузить профиль из `~/.claude/agents/profiles`
- строго соблюдать AGENT_PROFILE
- hard-fail при отсутствии профиля

**Выход:**

- IMPLEMENTATION_REPORT_<feature>.md

---

### Test Engineer Agent *(проектируется позже)*

- **Role:** тестирование фичи
- **Profile-aware:** ✅ обязательно
- **Обязан:** соблюдать Global Profile Rule

---

### Code Reviewer Agent *(проектируется позже)*

- **Role:** код-ревью и архитектурное соответствие
- **Profile-aware:** ✅ обязательно
- **Обязан:** проверять соответствие AGENT_PROFILE

---

### Feature Verifier Agent *(проектируется позже)*

- **Role:** финальная верификация фичи
- **Profile-aware:** ⚠️ частично

---

## Stage 5 — Системная верификация

### System Verifier Agent *(проектируется позже)*

- **Role:** проверка целостности системы
- **Profile-aware:** ⚠️ частично

---

## Stage 6 — Документация и релиз

### Documentation Agent *(проектируется позже)*

- **Role:** финальная документация
- **Profile-aware:** ⚠️ частично

---

### Release / DevOps Agent *(проектируется позже)*

- **Role:** деплой и релиз
- **Profile-aware:** ✅ обязательно
- **Обязан:** использовать AGENT_PROFILE_<profile>

---

## Refactoring Agents (standalone pipeline)

**Примечание:** Агенты рефакторинга работают в рамках отдельного standalone пайплайна
(REFACTORING_PROMPT.md) и НЕ зависят от основного product creation pipeline.

### Refactor Code Analyzer Agent

- **ID:** refactor-code-analyzer
- **Role:** анализ кода на code smells и метрики качества
- **Profile-aware:** ❌ нет
- **Pipeline:** Refactoring (standalone)

**Ответственность:**

- Сканирование кода на code smells (20+ типов)
- Измерение метрик качества (сложность, связность, связанность)
- Категоризация проблем по severity
- Анализ структуры кода

**Вход:**

- Запрос пользователя (текст)

**Выход:**

- CODE_ANALYSIS.md

---

### Refactoring Strategy Builder Agent

- **ID:** refactor-strategy-builder
- **Role:** создание пошагового плана рефакторинга
- **Profile-aware:** ❌ нет
- **Pipeline:** Refactoring (standalone)

**Ответственность:**

- Создание пошагового плана рефакторинга
- Выбор рефакторинг-паттернов (Martin Fowler)
- Определение зависимостей между шагами
- Планирование тестов для каждого шага

**Вход:**

- CODE_ANALYSIS.md

**Выход:**

- REFACTORING_STRATEGY.md

---

### Refactoring Executor Agent

- **ID:** refactor-executor
- **Role:** выполнение одного шага рефакторинга
- **Profile-aware:** ❌ нет
- **Pipeline:** Refactoring (standalone)

**Ответственность:**

- Выполнение ОДНОГО шага рефакторинга за раз
- Точное следование стратегии
- Минимальные изменения
- Документация изменений

**Вход:**

- REFACTORING_STRATEGY.md
- STEP_NUMBER (какой шаг выполнить)

**Выход:**

- STEP_REPORT_N.md
- Изменённый код

---

### Refactoring Verifier Agent

- **ID:** refactor-verifier
- **Role:** проверка шага после выполнения
- **Profile-aware:** ❌ нет
- **Pipeline:** Refactoring (standalone)

**Ответственность:**

- Проверка каждого шага после выполнения
- Запуск тестов (baseline + verification)
- Проверка сохранения поведения
- Оценка качества кода
- Возврат с инструкциями при ошибке

**Вход:**

- STEP_REPORT_N.md
- REFACTORING_STRATEGY.md

**Выход:**

- VERIFICATION_REPORT_N.md (PASS/FAIL + инструкции)

---

### Refactoring Project Verifier Agent

- **ID:** refactor-project-verifier
- **Role:** финальная проверка после всех шагов
- **Profile-aware:** ❌ нет
- **Pipeline:** Refactoring (standalone)

**Ответственность:**

- Финальная проверка после ВСЕХ шагов
- Сравнение метрик до/после
- Проверка разрешения всех code smells
- Оценка общего улучшения качества
- Финальное решение approve/reject

**Вход:**

- REFACTORING_REQUEST.md
- CODE_ANALYSIS.md
- REFACTORING_STRATEGY.md
- Все STEP_REPORT_*.md
- Все VERIFICATION_REPORT_*.md

**Выход:**

- FINAL_VERIFICATION.md (APPROVED/REJECTED)

---

## Analysis Agents (standalone)

**Примечание:** Агенты анализа работают в режиме standalone и НЕ зависят от основного product creation pipeline.
Они могут быть вызваны в любое время для глубокого анализа существующего проекта.

### Deep Analysis Agent

- **ID:** deep-analysis-agent
- **Role:** глубокий анализ проекта методом обратного регресса
- **Profile-aware:** ❌ нет
- **Pipeline:** Standalone

**Ответственность:**

- Проведение комплексного анализа проекта от общего к частному
- Создание архитектурной карты с полным объяснением
- Документирование структуры проекта
- Технологическая инвентаризация
- Выявление паттернов и практик

**Методология:**

Обратный регресс (4 уровня):
1. **Макро-архитектура** — общее назначение, высокоуровневые компоненты
2. **Мезо-архитектура** — доменные области, группы модулей
3. **Микро-архитектура** — структура модулей, ключевые классы
4. **Детализация** — конкретные реализации, алгоритмы

**Вход:**

- Рабочая директория проекта
- Опционально: области для углублённого анализа

**Выход (в `docs/research/`):**

- **ARCHITECTURE_MAP.md** (обязательный) — карта архитектуры со всеми уровнями
- **PROJECT_STRUCTURE.md** (обязательный) — полная структура проекта
- **TECH_NOTES.md** (обязательный) — технологическая записка
- **PATTERNS_AND_PRACTICES.md** (обязательный) — паттерны и практики
- **DATA_FLOW.md** (опциональный) — потоки данных
- **API_REFERENCE.md** (опциональный) — справочник API
- **GLOSSARY.md** (опциональный) — глоссарий
- **ISSUES_AND_IMPROVEMENTS.md** (опциональный) — проблемы и улучшения

**Использование:**

```bash
# Пример запуска через Task tool
# Task subagent_type=general-purpose prompt="Запусти deep-analysis-agent для анализа текущего проекта"
```

---

## Execution Preconditions

Любой execution-агент (Developer, Tester, Reviewer, Release):

- MUST receive resolved profile
- MUST load profile from global profile directory
- MUST refuse execution if profile is missing
- MUST NOT infer or invent profile rules

---

## Связанные документы

- ARTIFACTS_INDEX.md
- PROJECT_PROFILE.md
- PIPELINE_PROMPT.md
- QUALITY_SCORING.md
- `profiles/backend/BACKEND_PROFILES.md` — реестр backend профилей
- `profiles/backend/AGENT_PROFILE_*.md` — специализированные профили

---

## Статус документа

- **Статус:** Production-ready
- **Версия:** 8.0
- **Архитектурный уровень:** system / contract
- **Изменения v8.0:**
  - Добавлен новый раздел "Analysis Agents (standalone)"
  - Добавлен Deep Analysis Agent для комплексного анализа проектов
  - Новый агент использует методологию обратного регресса (4 уровня)
  - Создаёт комплект документации в `docs/research/`
- **Изменения v7.0:**
  - Добавлена категоризация профилей
  - Создана папка `backend/` для специализированных профилей
  - Обновлён Global Profile Rule с путями к категориям
  - Добавлен реестр `backend/BACKEND_PROFILES.md`
- **Изменения v6.0:** Удалён pipeline-orchestrator agent — оркестрация выполняется встроенным оркестратором Claude Code через скилл product-creator
- **Изменения v5.0:** Добавлены Refactoring Agents (5 агентов для standalone REFACTORING_PROMPT.md)

```
