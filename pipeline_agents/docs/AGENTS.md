# 1. Список агентов проектируемого пайплайна

## Уровень 0 — Управление процессом

### Pipeline Orchestrator Agent

* **Роль:** управление жизненным циклом пайплайна
* **Profile-aware:** нет
* **Единственная точка взаимодействия с человеком**
* **Критичность:** 🔴 ядро системы

---

## Уровень −1 — Подготовка пайплайна (pre-pipeline)

### Project Profile Generator Agent

* **Роль:** формализация человеческого намерения
* **Выход:**

    * `PROJECT_PROFILE.md` (internal)
    * `PROJECT_PROFILE_HUMAN.md` (human-readable)
* **Profile-aware:** нет (он профиль формирует)
* **Критичность:** 🔴 обязательный

---

### Pipeline Prompt Generator Agent

* **Роль:** генерация исполнимого master-prompt пайплайна
* **Выход:** `PIPELINE_PROMPT.md`
* **Profile-aware:** нет (использует готовый профиль)
* **Критичность:** 🔴 обязательный

---

## Этап 1 — Аналитика и требования

### Research Agent

* **Роль:** исследование и анализ контекста
* **Profile-aware:** нет
* **Артефакты:** `ANALYSIS.md`

### System Analyst Agent

* **Роль:** формирование требований и ТЗ
* **Profile-aware:** частично
* **Артефакты:** `TECH_REQUIREMENTS.md`, `SCOPE.md`

---

## Этап 2 — Архитектура и декомпозиция

### Solution Architect Agent

* **Роль:** проектирование архитектуры
* **Profile-aware:** обязательно
* **Артефакты:** `ARCHITECTURE_OVERVIEW.md`

### Feature Decomposition Agent

* **Роль:** декомпозиция на фичи и стадии
* **Profile-aware:** нет
* **Артефакты:** `WORK_BREAKDOWN.md`, `FEATURES_INDEX.md`

---

## Этап 3 — Планирование (TDD)

### TDD Planner Agent

* **Роль:** пофичевые TDD-роадмапы
* **Profile-aware:** обязательно
* **Артефакты:** `ROADMAP_<feature>.md`

---

## Этап 4 — Разработка (многопоточная)

### Developer Agent (`developer-<feature>`)

* **Роль:** реализация фичи
* **Profile-aware:** обязательно
* **Артефакты:** `IMPLEMENTATION_REPORT_<feature>.md`

### Test Engineer Agent

* **Роль:** тестирование
* **Profile-aware:** обязательно
* **Артефакты:** `TEST_REPORT_<feature>.md`

### Code Reviewer Agent

* **Роль:** ревью кода
* **Profile-aware:** обязательно
* **Артефакты:** `CODE_REVIEW_<feature>.md`

### Feature Verifier Agent

* **Роль:** финальная верификация фичи
* **Profile-aware:** частично
* **Артефакты:** `FEATURE_VERIFICATION_<feature>.md`
* **Правило:** score < 9 → возврат в разработку

---

## Этап 5 — Системная верификация

### System Verifier Agent

* **Роль:** проверка целостности системы
* **Profile-aware:** частично
* **Артефакты:** `SYSTEM_VERIFICATION.md`

---

## Этап 6 — Документация и релиз

### Documentation Agent

* **Роль:** сбор и оформление документации
* **Profile-aware:** частично
* **Артефакты:** `README.md`, `ARCHITECTURE.md`, `USAGE.md`

### Release / DevOps Agent

* **Роль:** деплой и релиз
* **Profile-aware:** обязательно
* **Артефакты:** `DEPLOY.md`, `RELEASE_NOTES.md`

---

# 2. Другие элементы, необходимые для полноценной работы системы

## 2.1 Контрактные документы (обязательные)

* `AGENTS_INDEX.md` — кто есть кто
* `ARTIFACTS_INDEX.md` — контракты файлов
* `PROJECT_PROFILE.md` — профиль проекта (internal)
* `PROJECT_PROFILE_HUMAN.md` — интерпретация для человека
* `PIPELINE_PROMPT.md` — master-prompt пайплайна
* `QUALITY_SCORING.md` — правила оценки 0–10

---

## 2.2 Профили разработки (расширяемые)

* `AGENT_PROFILE_backend.md`
* `AGENT_PROFILE_web.md`
* `AGENT_PROFILE_mobile-android.md`
* `AGENT_PROFILE_mobile-ios.md`
* `AGENT_PROFILE_multiplatform.md`
* `AGENT_PROFILE_cli.md`

➡️ **Добавление нового направления = добавление профиля, не агентов**

---

## 2.3 Системные механизмы (логические)

* state-machine оркестратора
* versioning артефактов
* human feedback log
* stage-gate checker
* parallel execution controller

---

# 3. Схема работы пайплайна (каноническая)

## 3.1 Полная схема (end-to-end)

```text
[Human Intent]
      ↓
[Pipeline Orchestrator]
      ↓
[Project Profile Generator]
      ↓
PROJECT_PROFILE.md (internal)
PROJECT_PROFILE_HUMAN.md
      ↓
[Pipeline Prompt Generator]
      ↓
PIPELINE_PROMPT.md
      ↓
[Human Approval]
      ↓
══════════ PIPELINE EXECUTION ══════════

Stage 1: Analysis
  Research Agent
  System Analyst

Stage 2: Decomposition
  Solution Architect
  Feature Decomposer

Stage 3: TDD Planning
  TDD Planner

Stage 4: Development (PARALLEL)
  Feature A → Dev → Test → Review → Verify
  Feature B → Dev → Test → Review → Verify
  Feature C → Dev → Test → Review → Verify

Stage 5: System Verification
  System Verifier

Stage 6: Documentation & Release
  Documentation Agent
  Release Agent
```

---

## 3.2 Ключевые архитектурные свойства

* 🔹 **Один вход** — человеческое намерение
* 🔹 **Один интерфейс с человеком** — оркестратор
* 🔹 **Документы = API между агентами**
* 🔹 **Многопоточность только на уровне фич**
* 🔹 **Качество обеспечивается циклом, а не доверием**

---

# 4. Итоговое резюме

Ты спроектировал не просто пайплайн, а:

> **управляемую, расширяемую, профилируемую
> мультиагентскую фабрику разработки
> с человеческим контролем и формальными контрактами**

Система:

* масштабируется по направлениям
* устойчива к ошибкам агентов
* прозрачна для человека
* готова к автоматизации
