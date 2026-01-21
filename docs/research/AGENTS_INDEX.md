# AGENTS_INDEX.md

## Назначение документа

AGENTS_INDEX.md описывает полный состав субагентов, используемых в пайплайне разработки,
их роли, зоны ответственности, входные и выходные артефакты, а также правила работы
с учетом профилей разработки.

Документ является:
- контрактом взаимодействия агентов
- справочником для оркестратора пайплайна
- точкой контроля для человека (human-in-the-loop)

---

## Базовая концепция

Каждый агент в системе определяется формулой:

Agent = Role + Profile + Artifacts

Где:
- Role — универсальная ответственность агента
- Profile — доменная и технологическая специализация
- Artifacts — контракт входных и выходных документов

Роль агента неизменна.
Профиль влияет только на способ исполнения роли.

---

## Профили разработки (Agent Profiles)

Agent Profile — это формализованное описание:
- технологического стека
- архитектурных паттернов
- нефункциональных требований
- best practices
- типовых ошибок и ограничений

Профиль не является агентом и не выполняется самостоятельно.
Он используется execution-агентами как входная спецификация.

### Поддерживаемые профили (базовый набор)

- backend — серверная разработка
- web — web frontend
- mobile-android — Android
- mobile-ios — iOS
- multiplatform — shared core + adapters
- cli — CLI инструменты

Добавление нового направления разработки = добавление нового AGENT_PROFILE_<profile>.md.

---

## PROJECT_PROFILE.md

PROJECT_PROFILE.md — обязательный входной артефакт пайплайна.

Он определяет:
- тип проекта
- профиль для каждой группы ролей
- используемый технологический стек

Все execution-агенты обязаны учитывать PROJECT_PROFILE.md.
Агент без профиля считается некорректно сконфигурированным.

---

## Общая схема пайплайна

Human
→ Pipeline Orchestrator
→ Stage 1: Analysis
→ Stage 2: Decomposition
→ Stage 3: Roadmaps (TDD)
→ Stage 4: Development (Loop)
→ Stage 5: System Verification
→ Stage 6: Release / Deploy

---

## Классификация агентов по работе с профилями

- Profile-unaware — полностью универсальный
- Profile-aware (partial) — частично учитывает профиль
- Profile-aware (required) — не может работать без профиля

---

## Уровень 0 — Оркестрация

Pipeline Orchestrator Agent
ID: pipeline-orchestrator
Тип: system / controller
Profile-aware: нет

Ответственность:
- управление стадиями пайплайна
- запуск агентов
- передача артефактов
- остановка процесса для human review

Ограничения:
- не пишет код
- не принимает продуктовых решений
- не интерпретирует профиль

---

## Этап 1 — Аналитика и требования

Research Agent
ID: research-agent
Тип: analytical
Profile-aware: нет

System Analyst Agent
ID: system-analyst
Тип: analytical
Profile-aware: частично

Учет профиля:
- нефункциональные требования
- платформенные ограничения

---

## Этап 2 — Архитектура и декомпозиция

Solution Architect Agent
ID: solution-architect
Тип: architectural
Profile-aware: обязательно

Учет профиля:
- архитектурные паттерны
- допустимые технологии
- способы интеграции

Feature Decomposition Agent
ID: feature-decomposer
Тип: planning
Profile-aware: нет

---

## Этап 3 — Планирование (TDD)

TDD Planner Agent
ID: tdd-planner
Тип: planning
Profile-aware: обязательно

Учет профиля:
- типы тестов
- обязательные сценарии
- требования к покрытию

---

## Этап 4 — Разработка (циклический)

Developer Agent
ID: developer-<feature>
Тип: implementation
Profile-aware: обязательно

Определяется профилем:
- язык и фреймворки
- архитектурные правила
- инструменты сборки
- coding conventions
- стратегия тестирования

Test Engineer Agent
ID: test-engineer
Тип: qa
Profile-aware: обязательно

Code Reviewer Agent
ID: code-reviewer
Тип: reviewer
Profile-aware: обязательно

Feature Verifier Agent
ID: feature-verifier
Тип: verifier
Profile-aware: частично

---

## Этап 5 — Системная проверка

System Verifier Agent
ID: system-verifier
Тип: verifier
Profile-aware: частично

Проверяет:
- интеграцию фич
- системные ограничения с учетом профиля

---

## Этап 6 — Документация и релиз

Documentation Agent
ID: documentation-agent
Тип: documentation
Profile-aware: частично

Release / DevOps Agent
ID: release-agent
Тип: devops
Profile-aware: обязательно

Профиль определяет:
- тип деплоя
- CI/CD
- каналы распространения

---

## Общие правила для всех агентов

1. Агент работает строго в рамках своей роли
2. Профиль задается только через PROJECT_PROFILE.md
3. Агент не может менять профиль
4. Агент обязан отказать в работе при неподдерживаемом профиле
5. Каждый агент создает новый артефакт, а не изменяет чужой
6. Stage-gate проверки обязаны учитывать профиль

---

## Execution Preconditions

Execution-агенты — агенты, непосредственно реализующие,
проверяющие или выпускающие результат (Developer, Tester,
Reviewer, Release).

Любой execution-агент не может быть запущен, если не задан
и не валидирован профиль проекта.

---

## Связанные документы

- ARTIFACTS_INDEX.md
- PROJECT_PROFILE.md
- AGENT_PROFILE_<profile>.md
- QUALITY_SCORING.md
- PIPELINE_PROMPT.md

Статус документа: Production-ready
Версия: 3.0
