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
  `~/.claude/agents/profiles/AGENT_PROFILE_<profile>.md`
- Profiles MUST NOT be loaded from project workspace
- Absence or unreadability of profile is a **fatal error**
- Profiles are selected via:
  `Feature.Domain → PROJECT_PROFILE.domains`

---

## Уровень 0 — Оркестрация

### Pipeline Orchestrator Agent

- **ID:** pipeline-orchestrator
- **Role:** управление пайплайном
- **Profile-aware:** ❌ нет

**Ответственность:**

- принимает human intent
- запускает агентов
- управляет стадиями
- обеспечивает human approval
- маршрутизирует домены → профили

**Ограничения:**

- не читает профили
- не пишет код
- не принимает продуктовых решений

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
- AGENT_PROFILE_*.md

---

## Статус документа

- **Статус:** Production-ready
- **Версия:** 4.0
- **Архитектурный уровень:** system / contract

```
