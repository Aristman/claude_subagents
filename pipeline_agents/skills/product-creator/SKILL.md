---
name: product-creator
description: Запускает полный мультиагентный пайплайн создания нового продукта от идеи до релиза
---

# PRODUCT_CREATOR — Создание нового продукта

## Версия
- version: 3.0.0
- standalone: true
- purpose: product_creation

---

## Назначение

Этот скилл превращает ТЕБЯ (встроенный оркестратор Claude Code) в **оркестратор мультиагентского пайплайна создания продуктов**.

Ты отвечаешь за:
- планирование задач
- запуск и координацию агентов
- параллелизацию
- управление зависимостями
- enforcement quality gates

Ты ОБЯЗАН рассматривать следующие документы как
**авторитетные системные контракты**:

- AGENTS_INDEX.md
- ARTIFACTS_INDEX.md
- QUALITY_SCORING.md

Спецификации агентов (*.md) определяют поведение и ограничения агентов.
Файлы AGENT_PROFILE_*.md определяют доменные правила выполнения.

Любые предположения вне этих документов ЗАПРЕЩЕНЫ.

---

## НАЗНАЧЕНИЕ ПАЙПЛАЙНА

Преобразовать **один человеческий запрос** в полностью:
- реализованную
- проверенную
- задокументированную
- готовую к релизу систему

путём исполнения **строгого, артефактно-ориентированного,
quality-gated пайплайна**.

---

## ОСНОВНЫЕ ПРАВИЛА ПАЙПЛАЙНА (ОБЯЗАТЕЛЬНЫ)

### 1. Модель взаимодействия агентов
- Агенты НЕ общаются напрямую
- Всё взаимодействие происходит ТОЛЬКО через артефакты
- ТЫ (встроенный оркестратор) управляешь выполнением

### 2. Авторитет артефактов
- Артефакты считаются неизменяемыми после утверждения
- У каждого артефакта есть ровно один агент-автор
- Агентам ЗАПРЕЩЕНО изменять артефакты, которые они не создавали

### 3. Profile-aware выполнение
- Execution-агенты ОБЯЗАНЫ определять профиль через:
  Feature.Domain → PROJECT_PROFILE.domains
- Профили загружаются ТОЛЬКО из:
  ~/.claude/agents/profiles/AGENT_PROFILE_<profile>.md
- Отсутствие или недоступность профиля = ЖЁСТКИЙ СБОЙ (hard fail)

### 4. Quality Gates
- Порог приёмки фичи: score ≥ 9 / 10
- Порог приёмки стадии: score ≥ 9 / 10
- Порог приёмки системы: score ≥ 9 / 10
- Score < 9 ВСЕГДА запускает цикл возврата
- Оценка качества строго регулируется QUALITY_SCORING.md

### 5. Отображение артефактов для Human-in-the-Loop
При показе PROJECT_PROFILE_HUMAN.md или PIPELINE_PROMPT.md:
- Прочитать файл через Read tool
- Включить ПОЛНОЕ содержимое в текстовый ответ между "---"
- Запросить явное подтверждение через AskUserQuestion

### 6. Human-in-the-Loop
Явное одобрение человека ОБЯЗАТЕЛЬНО для:
- PROJECT_PROFILE (человекочитаемая версия)
- PIPELINE_PROMPT

В этих точках ты ОБЯЗАН остановиться и ждать подтверждения.

---

## ФАЙЛОВАЯ СТРУКТУРА АРТЕФАКТОВ (ОБЯЗАТЕЛЬНА)

Все артефакты пайплайна сохраняются в директории `docs/`:

```
docs/
├── project/              # Артефакты уровня проекта (Фазы 1-4, 7-9)
│   ├── PROJECT_PROFILE.md
│   ├── PROJECT_PROFILE_HUMAN.md
│   ├── PIPELINE_PROMPT.md
│   ├── ANALYSIS.md
│   ├── TECH_REQUIREMENTS.md
│   ├── SCOPE.md
│   ├── ARCHITECTURE_OVERVIEW.md
│   ├── FEATURES_INDEX.md         # Декомпозиция на ФИЧИ
│   ├── TASKS_INDEX.md            # Индекс всех задач
│   ├── SYSTEM_VERIFICATION.md
│   ├── README.md
│   ├── ARCHITECTURE.md
│   ├── USAGE.md
│   ├── DEPLOY.md
│   └── RELEASE_NOTES.md
│
├── roadmaps/             # TDD roadmaps для задач фич (Фаза 5)
│   ├── ROADMAP_TASKS_<feature>.md    # Задачи фичи (TDD)
│   └── ...
│
└── develop/              # Артефакты разработки задач (Фаза 6)
    └── <FEATURE>/        # Артефакты фичи
        └── <TASK>/       # Артефакты задачи
            ├── IMPLEMENTATION_REPORT_<TASK>.md
            ├── TEST_REPORT_<TASK>.md
            ├── CODE_REVIEW_<TASK>.md
            └── FEATURE_VERIFICATION_<TASK>.md
```

**Правила:**
- Все проектные артефакты → `docs/project/`
- Roadmaps задач → `docs/roadmaps/ROADMAP_TASKS_<feature>.md`
- Артефакты разработки задачи → `docs/develop/<FEATURE>/<TASK>/`

---

## GIT WORKFLOW (ОБЯЗАТЕЛЬНЫЙ)

**Основная ветка разработки:** `{MAIN_BRANCH}` — определяется в Фазе 0, формат `<PROJECT>-DEV`

### Структура веток (ФИЧИ ПОСЛЕДОВАТЕЛЬНО, ЗАДАЧИ ПАРАЛЛЕЛЬНО)

```
{MAIN_BRANCH} (основная ветка, например SW-DEV)
│
├── (коммиты на каждой стадии)
│   ├── Фаза 1: Project Profile
│   ├── Фаза 2: Pipeline Definition
│   ├── Фаза 3: Analytics
│   ├── Фаза 4: Architecture (FEATURES_INDEX.md)
│   ├── Фаза 5: TDD Planning (ROADMAP_TASKS для фич)
│   └──
│
├─────────────────────────────────────────────────────────┐
│ ФАЗА 6: Реализация фич (ПОСЛЕДОВАТЕЛЬНО)                 │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │ ФИЧА F-001: Authentication System              │   │
│  │ Ветка: feature/auth-system                     │   │
│  ├─────────────────────────────────────────────────┤   │
│  │ Задачи (параллельно, до 3 штук):               │   │
│  │   ├─ T-001: User Registration → commit        │   │
│  │   ├─ T-002: Login/Logout → commit             │   │
│  │   ├─ T-003: Password Reset → commit           │   │
│  │   └─ T-004: JWT Tokens → commit               │   │
│  │                                                 │   │
│  │ Все коммиты → feature/auth-system              │   │
│  └─────────────────────────────────────────────────┘   │
│           │                                              │
│           ↓ (все задачи score ≥ 9)                       │
│        Merge → {MAIN_BRANCH}                             │
│           │                                              │
│           ↓                                              │
│  ┌─────────────────────────────────────────────────┐   │
│  │ ФИЧА F-002: User Profile                       │   │
│  │ Ветка: feature/user-profile                    │   │
│  ├─────────────────────────────────────────────────┤   │
│  │ Задачи (параллельно, до 3 штук):               │   │
│  │   ├─ T-005: Profile View → commit             │   │
│  │   ├─ T-006: Profile Edit → commit             │   │
│  │   └─ T-007: Avatar Upload → commit            │   │
│  │                                                 │   │
│  │ Все коммиты → feature/user-profile             │   │
│  └─────────────────────────────────────────────────┘   │
│           │                                              │
│           ↓ (все задачи score ≥ 9)                       │
│        Merge → {MAIN_BRANCH}                             │
│           │                                              │
│           ↓                                              │
│        [Фича F-003...]                                  │
│                                                          │
└─────────────────────────────────────────────────────────┘
    │
    ↓
├── Фаза 7: System Verification
├── Фаза 8: Documentation
└── Фаза 9: Release
```

### Общий алгоритм работы

**В начале пайплайна (Фаза 0):**
1. Определить имя основной ветки `{MAIN_BRANCH}`
2. Проверить существование ветки `{MAIN_BRANCH}`
3. Если не существует — создать от текущей ветки
4. Переключиться на `{MAIN_BRANCH}`

**Для фаз 1-5, 7-9:**
1. Всё работает в ветке `{MAIN_BRANCH}`
2. **Агенты сами делают коммиты** при завершении своей работы

**Для фазы 6 (Реализация фич):**
1. **Каждая фича = отдельная ветка `feature/<name>`**
2. Фичи выполняются **строго последовательно**
3. Задачи внутри фичи могут выполняться **параллельно** (до 3 штук)
4. Все коммиты задач → в ветку фичи
5. После завершения всех задач фичи → merge в `{MAIN_BRANCH}`
6. **Оркестратор делает merge** после успешного завершения фичи

### Ответственность за коммиты

| Кто | Когда делает коммит | Что коммитит |
|-----|---------------------|--------------|
| **Агенты фаз 1-5, 7-9** | После завершения работы фазы | Созданные артефакты |
| **developer-agent** | После реализации задачи | Код + IMPLEMENTATION_REPORT |
| **test-engineer** | После тестирования задачи | TEST_REPORT |
| **code-reviewer** | После ревью задачи | CODE_REVIEW |
| **feature-verifier** | После верификации задачи | FEATURE_VERIFICATION |
| **Оркестратор** | **Merge фичи в {MAIN_BRANCH}** | Вся фича |

### Git команды (выполняют агенты)

**Проверка статуса:**
\`\`\`bash
git status
\`\`\`

**Добавление файлов:**
\`\`\`bash
git add <файлы>
\`\`\`

**Коммит:**
\`\`\`bash
git commit -m "<сообщение>"
\`\`\`

**Push (опционально):**
\`\`\`bash
git push
\`\`\`

---

## СТАДИИ ПАЙПЛАЙНА (СТРОГИЙ ПОРЯДОК)

### Фаза 0 — Приём намерения и инициализация

**Приём намерения:**
- Принять сырой человеческий запрос
- Сохранить его ДОСЛОВНО
- НЕ интерпретировать и НЕ уточнять

**Инициализация Git:**
1. Определи основную ветку разработки:
   - Если пользователь не указал — спроси через AskUserQuestion
   - Если указал в запросе — используй это значение
   - Формат: `<PROJECT>-DEV` (например, `SW-DEV`, `AUTH-DEV`, `CRM-DEV`)

2. Проверь существование ветки:
```bash
git branch --show-current
git branch -a | grep <MAIN_BRANCH>
```

3. Если ветка не существует — создай от текущей:
```bash
git checkout -b <MAIN_BRANCH>
git push -u origin <MAIN_BRANCH>
```

4. Запомни имя ветки для использования во всех фазах как `{MAIN_BRANCH}`

---

#### Фаза 1 — Формализация проекта

Выполни через Task tool:
\`\`\`
subagent_type: project-profile-generator
\`\`\`

**Агент сам сделает коммит** после создания артефактов.

Выход: `docs/project/PROJECT_PROFILE.md`, `docs/project/PROJECT_PROFILE_HUMAN.md`

**Обработка вопросов от агента:**

Если агент создал `docs/project/CLARIFICATION_NEEDED.md`:
```python
if exists("docs/project/CLARIFICATION_NEEDED.md"):
    questions = parse_clarification_needed("docs/project/CLARIFICATION_NEEDED.md")
    user_answers = AskUserQuestion(questions=questions["Вопросы"], ...)
    create_file("docs/project/USER_ANSWERS.md", user_answers)
    remove("docs/project/CLARIFICATION_NEEDED.md")

    # Перезапускаем агента с ответами
    agent_result = Task(subagent_type="project-profile-generator", prompt="""
    ПЕРЕЗАПУСК С ОТВЕТАМИ:

    Файл docs/project/USER_ANSWERS.md содержит ответы пользователя.
    Используй эти ответы для создания финальных PROJECT_PROFILE.md.
    НЕ задавай те же вопросы повторно.
    """)

    # После завершения - удаляем USER_ANSWERS.md
    remove("docs/project/USER_ANSWERS.md")
```

ДЕЙСТВИЯ:
- Прочитать файл `docs/project/PROJECT_PROFILE_HUMAN.md` через Read tool
- Включить ПОЛНОЕ содержимое в текстовый ответ между "---"
- Запросить явное подтверждение через AskUserQuestion

При правках — ПОВТОРИТЬ эту фазу.
---

#### Фаза 2 — Определение пайплайна

Выполни через Task tool:
```
subagent_type: pipeline-prompt-generator
```

**Агент сам сделает коммит** после создания артефактов.

Выход: `docs/project/PIPELINE_PROMPT.md`

ДЕЙСТВИЯ:
- Прочитать файл `docs/project/PIPELINE_PROMPT.md` через Read tool
- Включить ПОЛНОЕ содержимое в текстовый ответ между "---"
- Запросить явное подтверждение через AskUserQuestion

При правках — ПОВТОРИТЬ Фазу 1 и Фазу 2.

### Фаза 3 — Аналитика

Выполни через Task tool **с обработкой вопросов**:

1. **research-agent** → `docs/project/ANALYSIS.md`
   ```
   Task(subagent_type="research-agent", prompt="...")
   ```

2. **system-analyst** — ЦИКЛ с обработкой вопросов:

   ```
   ЦИКЛ while (true):
     Task(subagent_type="system-analyst", prompt="Проанализируй docs/project/ANALYSIS.md и создай системные требования.")

     ПРОВЕРКА: существует ли файл docs/project/CLARIFICATION_NEEDED.md?

     Если ДА:
       → Читаешь CLARIFICATION_NEEDED.md
       → Задаю вопросы пользователю через AskUserQuestion
       → Получаю ответы
       → Создаю docs/project/USER_ANSWERS.md с ответами
       → Повторяю цикл (агент перезапустится с ответами)

     Если НЕТ:
       → Артефакты TECH_REQUIREMENTS.md и SCOPE.md созданы
       → Выход из цикла
   ```

**Псевдокод обработки вопросов:**
\`\`\`python
# После завершения system-analyst:
if exists("docs/project/CLARIFICATION_NEEDED.md"):
    questions = parse_clarification_needed("docs/project/CLARIFICATION_NEEDED.md")

    # Задаю вопросы пользователю
    user_answers = AskUserQuestion(
        questions=questions["Вопросы"],
        options=[...]
    )

    # Создаю файл с ответами
    create_user_answers("docs/project/USER_ANSWERS.md", user_answers)

    # УДАЛЯЮ CLARIFICATION_NEEDED.md (временный артефакт)
    remove("docs/project/CLARIFICATION_NEEDED.md")

    # Перезапускаю system-analyst с контекстом ответов
    agent_result = Task(subagent_type="system-analyst", prompt="""
    ПЕРЕЗАПУСК С ОТВЕТАМИ:

    Файл docs/project/USER_ANSWERS.md содержит ответы пользователя.

    Используй эти ответы для создания финальных TECH_REQUIREMENTS.md и SCOPE.md.
    НЕ задавай повторно те же вопросы.
    """)

    # После завершения агента - УДАЛЯЮ USER_ANSWERS.md
    remove("docs/project/USER_ANSWERS.md")
\`\`\`

**Агенты сами сделают коммит** после создания артефактов.

### Фаза 4 — Архитектура и декомпозиция на ФИЧИ

Выполни через Task tool (последовательно):

1. **solution-architect** → `docs/project/ARCHITECTURE_OVERVIEW.md`
   ```
   Task(subagent_type="solution-architect", prompt="...")
   ```

2. **feature-decomposer** → `docs/project/FEATURES_INDEX.md` (ЦИКЛ с обработкой вопросов):

   ```
   ЦИКЛ while (true):
     Task(subagent_type="feature-decomposer", prompt="Выполни декомпозицию архитектуры на ФИЧИ (крупные обособленные блоки функционала).")

     ПРОВЕРКА: существует ли файл docs/project/CLARIFICATION_NEEDED.md?

     Если ДА:
       → Читаешь CLARIFICATION_NEEDED.md
       → Задаю вопросы пользователю через AskUserQuestion
       → Получаю ответы
       → Создаю docs/project/USER_ANSWERS.md с ответами
       → Удаляю CLARIFICATION_NEEDED.md
       → Повторяю цикл (агент перезапустится с ответами)

     Если НЕТ:
       → Артефакт FEATURES_INDEX.md создан
       → Выход из цикла
   ```

**Псевдокод обработки вопросов:**
\`\`\`python
# После завершения feature-decomposer:
if exists("docs/project/CLARIFICATION_NEEDED.md"):
    questions = parse_clarification_needed("docs/project/CLARIFICATION_NEEDED.md")

    # Задаю вопросы пользователю
    user_answers = AskUserQuestion(
        questions=questions["Вопросы"],
        options=[...]
    )

    # Создаю файл с ответами
    create_user_answers("docs/project/USER_ANSWERS.md", user_answers)

    # УДАЛЯЮ CLARIFICATION_NEEDED.md (временный артефакт)
    remove("docs/project/CLARIFICATION_NEEDED.md")

    # Перезапускаю feature-decomposer с контекстом ответов
    agent_result = Task(subagent_type="feature-decomposer", prompt="""
    ПЕРЕЗАПУСК С ОТВЕТАМИ:

    Файл docs/project/USER_ANSWERS.md содержит ответы пользователя.

    Используй эти ответы для создания финального FEATURES_INDEX.md.
    НЕ задавай повторно те же вопросы.
    """)

    # После завершения агента - УДАЛЯЮ USER_ANSWERS.md
    remove("docs/project/USER_ANSWERS.md")
\`\`\`

Каждая фича ОБЯЗАНА иметь поле Domain.

**Агенты сами сделают коммит** после создания артефактов.

### Фаза 5 — TDD планирование для ФИЧ

Выполни через Task tool последовательно для **КАЖДОЙ фичи**.

**Путь назначения:** `docs/roadmaps/`

**Формат имени файла:** `ROADMAP_TASKS_<feature>.md`

где:
- `<feature>` — ID фичи (например, auth-system, user-profile, content)

**Алгоритм:**

1. Прочитай `docs/project/FEATURES_INDEX.md` и получи список всех фич
2. Для каждой фичи запусти `tdd-planner` **последовательно**:
   - Каждая фича разбивается на **задачи** (2-4 часа каждая)
   - Фичи обрабатываются по очереди (одна за другой)
3. Дождись завершения tdd-planner для текущей фичи
4. Повторяй для следующей фичи, пока все фичи не будут обработаны

**Пример запуска:**
\`\`\`
// Для каждой фичи последовательно:
Task(subagent_type="tdd-planner", prompt="... Feature F-001: auth-system ...")
// Ждём завершения
Task(subagent_type="tdd-planner", prompt="... Feature F-002: user-profile ...")
// Ждём завершения
...
\`\`\`

**Промпт для каждого tdd-planner:**
\`\`\`
Создай TDD roadmap с задачами для фичи:

Feature ID: {feature_id}
Feature Name: {feature_name}
Feature Description: {feature_description}
Domain: {feature_domain}
Dependencies: {feature_dependencies}

Входные артефакты:
- docs/project/FEATURES_INDEX.md
- docs/project/ARCHITECTURE_OVERVIEW.md
- docs/project/PROJECT_PROFILE.md

ТРЕБОВАНИЯ К ROADMAP:

1. Разбей фичу на **ЗАДАЧИ** (не фичи!):
   - Каждая задача = 2-4 часа работы
   - Всего 3-10 задач на фичу
   - Задачи должны быть мелкими и быстрыми

2. Формат задачи:

   ## Task T-XXX: <Task Name>

   - Description: [описание]
   - Estimated Time: 2-4 hours
   - Dependencies: [зависимости от других задач]
   - In scope: [что входит]
   - Out scope: [что НЕ входит]

3. Для каждой задачи определи:
   - Test Strategy (unit, integration, build & run)
   - Test Cases
   - Implementation Plan
   - Acceptance Criteria

4. ⚠️ КРИТИЧЕСКО: Build & Run Verification
   - Команда сборки проекта
   - Команда запуска проекта
   - Критерии успешности

Создай ROADMAP_TASKS_{feature_id}.md в директории docs/roadmaps/
в соответствии с твоим process workflow.

**После создания roadmap — сделай git commit.**
\`\`\`

**Агенты сами сделают коммит** после создания roadmaps.
Dependencies: {dependencies_list}  ← ОБЯЗАТЕЛЬНОЕ ПОЛЕ

Входные артефакты:
- docs/project/FEATURES_INDEX.md
- docs/project/WORK_BREAKDOWN.md
- docs/project/ARCHITECTURE_OVERVIEW.md
- docs/project/PROJECT_PROFILE.md

ТРЕБОВАНИЯ К ROADMAP:

1. Секция Dependencies ДОЛЖНА содержать:
   - Список ID фичей от которых зависит данная фича
   - Если зависимостей нет — явно указать "None"

2. Формат секции Dependencies:

   ## Dependencies

   ### Feature Dependencies
   - F-001: User Authentication (blocking)  ← пример
   - F-005: Database Layer (blocking)

   ### External Dependencies
   - None (или список внешних зависимостей)

3. Зависимости используются для:
   - Определения порядка разработки фич
   - Параллельной разработки независимых фич
   - Блокировки разработки зависимых фич до завершения родительских

Создай ROADMAP_{feature_id}_01.md в директории docs/roadmaps/
в соответствии с твоим process workflow.

**После создания roadmap — сделай git commit.**
\`\`\`

**Агенты сами сделают коммит** после создания roadmaps.

### Фаза 6 — Реализация ФИЧ (ПОСЛЕДОВАТЕЛЬНО) и ЗАДАЧ (ПАРАЛЛЕЛЬНО)

**⚠️ КРИТИЧЕСКИ ВАЖНО:**
- **ФИЧИ выполняются ПОСЛЕДОВАТЕЛЬНО** (одна за другой)
- **ЗАДАЧИ внутри фичи могут выполняться ПАРАЛЛЕЛЬНО** (до 3 штук)
- Каждая фича = отдельная ветка `feature/<name>`
- После завершения всех задач фичи → merge в `{MAIN_BRANCH}`

---

## ⚠️ КРИТИЧЕСКИ ВАЖНЫЕ ТРЕБОВАНИЯ

### 1. Полный цикл для КАЖДОЙ задачи (НЕПРИЕМЛЕМО ПРОПУСКАТЬ ШАГИ)

**КАЖДАЯ задача ДОЛЖНА иметь ВСЕ артефакты:**
- ✅ `IMPLEMENTATION_REPORT_<task>.md` — от developer-agent
- ✅ `TEST_REPORT_<task>.md` — от test-engineer
- ✅ `CODE_REVIEW_<task>.md` — от code-reviewer
- ✅ `FEATURE_VERIFICATION_<task>.md` — от feature-verifier (с score ≥ 9)

❌ **ЗАПРЕЩЕНО:**
- Создавать только IMPLEMENTATION_REPORT и пропускать остальные
- "Устать" и делать задачи в ускоренном режиме
- Создавать задачи больше 4-6 часов работы

### 2. Параллельная разработка задач (до 3 одновременно)

✅ **МОЖНО разрабатывать до 3 задач параллельно** при соблюдении условий:
- У задач **нет зависимостей** друг от друга (проверь Dependencies в ROADMAP_TASKS)
- Каждая задача проходит **ПОЛНЫЙ цикл** разработки
- Все задачи разрабатываются в ветке фазы, каждая задача = отдельный коммит

❌ **ЗАПРЕЩЕНО:**
- Разрабатывать более 3 задач параллельно
- Игнорировать зависимости между задачами
- Запускать разработку зависимой задачи до завершения родительской

### 3. Учёт зависимостей задач

**Порядок разработки задач внутри фазы:**
1. Сначала задачи без зависимостей (Level 0)
2. Затем задачи, зависящие от Level 0 (Level 1)
3. И так далее по дереву зависимостей

**Пример:**
```
T-001 (Registration API)    → нет зависимостей → Level 0
T-002 (Login API)           → нет зависимостей → Level 0
T-003 (JWT Tokens)          → зависит от T-002 → Level 1

Порядок: (T-001, T-002 параллельно) → (T-003)
```

---

## Алгоритм выполнения Фазы 6

**⚠️ ФИЧИ ВЫПОЛНЯЮТСЯ ПОСЛЕДОВАТЕЛЬНО, ЗАДАЧИ ПАРАЛЛЕЛЬНО**

```
ДЛЯ КАЖДОЙ ФИЧИ (по очереди):

┌─────────────────────────────────────────────────────────────────┐
│ 1. Создать ветку фичи: feature/<name>                           │
│    git checkout -b feature/<name>                               │
├─────────────────────────────────────────────────────────────────┤
│ 2. Прочитать ROADMAP_TASKS_<feature>.md                        │
│ 3. Разбить задачи на группы по зависимостям                    │
├─────────────────────────────────────────────────────────────────┤
│ 4. Разрабатывать задачи (до 3 параллельно)                     │
│    - Задачи без зависимостей → параллельно                     │
│    - Задачи с зависимостями → последовательно                  │
├─────────────────────────────────────────────────────────────────┤
│ 5. После завершения ВСЕХ задач фичи:                           │
│    - Проверить что все задачи score ≥ 9                         │
│    - Merge feature/<name> → {MAIN_BRANCH}                      │
└─────────────────────────────────────────────────────────────────┘

ПЕРЕЙТИ К СЛЕДУЮЩЕЙ ФИЧЕ (повторить шаги 1-5)
```

---

## Git Workflow для Фазы 6 (Фичи последовательно, Задачи параллельно)

### Ответственность за коммиты

**Оркестратор делает merge ПОСЛЕ успешного завершения ВСЕХ задач фичи:**

| Агент | Действие | Коммит? |
|-------|----------|---------|
| developer-agent | Реализует задачу | ❌ Нет (оркестратор) |
| test-engineer | Тестирует задачу | ❌ Нет (оркестратор) |
| code-reviewer | Делает ревью задачи | ❌ Нет (оркестратор) |
| feature-verifier | Верифицирует задачу | ❌ Нет (оркестратор) |
| **Оркестратор** | **Делает коммит после каждой задачи (score ≥ 9)** | ✅ **Да** |
| **Оркестратор** | **Делает merge фазы после всех задач** | ✅ **Да** |

### Когда делать коммит

**Разработка задачи (в ветке feature/<name>):**
```
developer-agent → test-engineer → code-reviewer → feature-verifier
                                                                  ↓
                                                           score ≥ 9?
                                                                ✅ Да
                                                        ┌───────────────┐
                                                        │ ОРКЕСТРАТОР   │
                                                        │ делает коммит │
                                                        │ в feature/<name>│
                                                        └───────────────┘
```

**Параллельная разработка 3 задач:**
```
Задача T-001: dev → test → review → verify → score ≥ 9 → КОММИТ в feature/<name>
Задача T-002: dev → test → review → verify → score ≥ 9 → КОММИТ в feature/<name>
Задача T-003: dev → test → review → verify → score ≥ 9 → КОММИТ в feature/<name>
```

### Что коммитить

**Для задачи `{task_id}` коммитить в ветку `feature/<name>`:**
```
docs/develop/{feature}/{task_id}/IMPLEMENTATION_REPORT_{task_id}.md
docs/develop/{feature}/{task_id}/TEST_REPORT_{task_id}.md
docs/develop/{feature}/{task_id}/CODE_REVIEW_{task_id}.md
docs/develop/{feature}/{task_id}/FEATURE_VERIFICATION_{task_id}.md
```

### Команды для коммита (выполняет оркестратор)

```bash
# После успешной верификации задачи (score ≥ 9)
git add docs/develop/{feature}/{task_id}/
git commit -m "feat: {task_name} ({task_id})

- Implementation: developer-agent
- Test: test-engineer
- Review: code-reviewer
- Verification: feature-verifier (score ≥ 9)
"
```

### Команда для merge фичи (выполняет оркестратор)

```bash
# После успешного завершения ВСЕХ задач фичи
git checkout {MAIN_BRANCH}
git merge feature/{name}
git branch -d feature/{name}
```

### Псевдокод для параллельной разработки задач

```python
# ============================================================
# ПОЛНЫЙ ЦИКЛ РАЗРАБОТКИ ФИЧИ (последовательно)
# ============================================================

# Читаем FEATURES_INDEX.md
features = parse_features("docs/project/FEATURES_INDEX.md")

# ============================================================
# ДЛЯ КАЖДОЙ ФИЧИ (ПОСЛЕДОВАТЕЛЬНО)
# ============================================================
for feature in features:
    feature_id = feature["id"]
    feature_name = feature["name"]

    # ─────────────────────────────────────────────────────────────────
    # ЭТАП 1: Создать ветку фичи
    # ─────────────────────────────────────────────────────────────────
    bash_command(f"git checkout -b feature/{feature_id}")

    # Читаем roadmap задач для фичи
    tasks = parse_tasks(f"docs/roadmaps/ROADMAP_TASKS_{feature_id}.md")

    # ─────────────────────────────────────────────────────────────────
    # ЭТАП 2: Группируем задачи по зависимостям
    # ─────────────────────────────────────────────────────────────────
    # Level 0: без зависимостей
    # Level 1: зависят от Level 0
    # и т.д.

    # ─────────────────────────────────────────────────────────────────
    # ЭТАП 3: Разрабатываем задачи (до 3 параллельно)
    # ─────────────────────────────────────────────────────────────────
    for level_tasks in group_by_level(tasks):
        # level_tasks = задачи одного уровня (максимум 3)

        # ─────────────────────────────────────────────────────────────────
        # ШАГ 3.1: developer-agent (ПАРАЛЛЕЛЬНО)
        # ─────────────────────────────────────────────────────────────────
        dev_tasks = []
        for task in level_tasks:
            task_dev = Task(
                subagent_type="developer-agent",
                prompt=f"Реализуй задачу {task['id']}: {task['name']}"
            )
            dev_tasks.append((task, task_dev))

        # Ждём завершения ВСЕХ
        for task, task_dev in dev_tasks:
            result = TaskOutput(task_id=task_dev["id"], block=True, timeout=600000)

        # ─────────────────────────────────────────────────────────────────
        # ШАГ 3.2: test-engineer + code-reviewer (ПАРАЛЛЕЛЬНО)
        # ─────────────────────────────────────────────────────────────────
        test_review_tasks = []
        for task in level_tasks:
            task_test = Task(subagent_type="test-engineer", prompt=f"Тестируй {task['id']}")
            task_review = Task(subagent_type="code-reviewer", prompt=f"Ревью {task['id']}")
            test_review_tasks.append((task, task_test, task_review))

        # Ждём завершения ВСЕХ
        for task, task_test, task_review in test_review_tasks:
            TaskOutput(task_id=task_test["id"], block=True, timeout=600000)
            TaskOutput(task_id=task_review["id"], block=True, timeout=600000)

        # ─────────────────────────────────────────────────────────────────
        # ШАГ 3.3: feature-verifier (ПОСЛЕДОВАТЕЛЬНО для каждой задачи)
        # ─────────────────────────────────────────────────────────────────
        for task in level_tasks:
            task_verify = Task(subagent_type="feature-verifier", prompt=f"Верифицируй {task['id']}")
            result = TaskOutput(task_id=task_verify["id"], block=True, timeout=600000)

            # Проверяем score
            score = extract_score(result)

            if score >= 9:
                # Оркестратор делает коммит задачи
                bash_command(f"""
                    git add docs/develop/{feature_id}/{task['id']}/
                    git commit -m "feat: {task['name']} ({task['id']})"
                """)
                print(f"✅ {task['id']}: коммит создан (score={score})")
            else:
                # score < 9 — перезапуск developer-agent с доработкой
                # [код доработки...]

    # ─────────────────────────────────────────────────────────────────
    # ЭТАП 4: ВСЕ задачи фичи завершены (score ≥ 9)
    # ─────────────────────────────────────────────────────────────────
    # Merge фичи в {MAIN_BRANCH}
    bash_command(f"""
        git checkout {MAIN_BRANCH}
        git merge feature/{feature_id}
        git branch -d feature/{feature_id}
    """)
    print(f"✅ Фича {feature_id} завершена и смержена")

    # Переходим к следующей фиче
```

### Критические правила

1. **Коммит ТОЛЬКО после score ≥ 9** от feature-verifier
2. **Агенты НЕ делают коммиты** — только оркестратор
3. **Каждая задача = отдельный коммит** в ветку фичи
4. **Каждая фича = merge в {MAIN_BRANCH}** после всех задач
5. **Фичи выполняются ПОСЛЕДОВАТЕЛЬНО** (одна за другой)
6. **НЕ использовать Skill(commit)** — оркестратор делает напрямую через `git commit`


### Шаг 1: Анализ фич и зависимостей

```
1. Прочитай docs/project/WORK_BREAKDOWN.md
2. Прочитай docs/project/FEATURES_INDEX.md
3. Прочитай docs/roadmaps/ROADMAP_*.md для всех фич
4. Построй таблицу зависимостей:

   Feature ID | Dependencies | Level | Can Parallel
   -----------|--------------|-------|--------------
   F-001      | None         | 0     | F-002, F-003
   F-002      | F-001        | 1     | F-003
   F-003      | F-001        | 1     | F-002
   F-004      | F-002        | 2     | (после F-002)

5. Сгруппируй фичи по Level для последовательной обработки
6. Внутри каждого Level выдели до 3 фич для параллельной разработки
```

### Шаг 2: Параллельная разработка группы фич (до 3 штук)

### Шаг 2: Параллельная разработка группы задач (до 3 штук)

┌─────────────────────────────────────────────────────────────────┐
│ ГРУППА ЗАДАЧ Level N (до 3 задач параллельно)                  │
└─────────────────────────────────────────────────────────────────┘

**Для каждой задачи в группе запускаем ПОЛНЫЙ цикл:**

┌─────────────────────────────────────────────────────────────────┐
│ ЗАДАЧА {task_id_1}                                             │
├─────────────────────────────────────────────────────────────────┤
│ 1. developer-agent → IMPLEMENTATION_REPORT                     │
│ 2. test-engineer + code-reviewer (параллельно)                 │
│ 3. feature-verifier → FEATURE_VERIFICATION (score)             │
│ 4. Если score < 9 → доработка (повтор 1-3)                     │
│ 5. Если score ≥ 9 → оркестратор делает коммит в feature/<name>    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ ЗАДАЧА {task_id_2} (ПАРАЛЛЕЛЬНО с {task_id_1})                │
├─────────────────────────────────────────────────────────────────┤
│ 1. developer-agent → IMPLEMENTATION_REPORT                     │
│ 2. test-engineer + code-reviewer (параллельно)                 │
│ 3. feature-verifier → FEATURE_VERIFICATION (score)             │
│ 4. Если score < 9 → доработка (повтор 1-3)                     │
│ 5. Если score ≥ 9 → оркестратор делает коммит в feature/<name>    │
└─────────────────────────────────────────────────────────────────┘

⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ВСЕХ ЗАДАЧ В ГРУППЕ перед переходом к следующему Level
```

**После завершения ВСЕХ задач фичи:**
```
git checkout {MAIN_BRANCH}
git merge feature/{feature_name}
git branch -d feature/{feature_name}
```

---

## Визуальная схема флоу для ОДНОЙ ЗАДАЧИ

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLOW ДЛЯ ОДНОЙ ЗАДАЧИ                        │
└─────────────────────────────────────────────────────────────────┘

    developer-agent          (последовательно)
           │
           ▼
    ┌──────┴──────┐
    │             │
    ▼             ▼
test-engineer  code-reviewer   (ПАРАЛЛЕЛЬНО ⚡)
    │             │
    └──────┬──────┘
           │
           ▼
  feature-verifier             (последовательно, после ОБИХ)
           │
           ▼
      score ≥ 9?
           │
     ┌─────┴─────┐
     │           │
    ДА          НЕТ
     │           │
     ▼           ▼
  Оркестратор делает коммит   developer-agent (повтор с контекстом доработки)
  в feature/<name>                │
                                ▼ (цикл повторяется)
```

**Ключевые моменты:**
1. Developer идёт первым (создаёт код)
2. Test + Review идут параллельно (независимые проверки)
3. Verifier идёт последним (консолидирует ОБА результата)
4. При score < 9 — возврат к developer с задачами доработки

            Выполни доработку. Обнови IMPLEMENTATION_REPORT_{feature_id}.md
            """
        )
        TaskOutput(task_id=task["id"], block=True, timeout=600000)

        # Повторяем test + review + verifier для этой фичи
        # (контекст: доработка готова)
```

---


┌─────────────────────────────────────────────────────────────────┐
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 1. developer-agent (последовательно)                           │
│    Task(subagent_type="developer-agent", ...)                   │
│    ЖДЁМ ЗАВЕРШЕНИЯ (block=true)                                │
│    Выход: IMPLEMENTATION_REPORT.md                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2+3. test-engineer + code-reviewer (ПАРАЛЛЕЛЬНО)                │
│    [ОДНО сообщение с ДВУМЯ Task]                                │
│    ЖДЁМ ЗАВЕРШЕНИЯ ОБИХ (block=true)                           │
│    Выход: TEST_REPORT.md + CODE_REVIEW.md                       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 4. feature-verifier (последовательно)                           │
│    Task(subagent_type="feature-verifier", ...)                  │
│    ЖДЁМ ЗАВЕРШЕНИЯ (block=true)                                 │
│    Выход: FEATURE_VERIFICATION.md (score)                       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                    ┌─────────────────┐
                    │ ПРОВЕРКА SCORE  │
                    └─────────────────┘
                              ↓
              ┌───────────────┴───────────────┐
              │                               │
        score ≥ 9                      score < 9
              │                               │
              ↓                               ↓
      Оркестратор делает коммит                    Повтор 1-4 (доработка)
    (коммит через git)                    с контекстом
```

```
┌─────────────────────────────────────────────────────────────────┐
│ ЦИКЛ для фичи {feature_id} (повторять пока score < 9)          │
└─────────────────────────────────────────────────────────────────┘

                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Шаг 1: developer-agent (ПОСЛЕДОВАТЕЛЬНО)                       │
├─────────────────────────────────────────────────────────────────┤
│ Task(                                                            │
│   subagent_type="developer-agent",                              │
│   prompt="Реализуй фичу {feature_id}..."                        │
│ )                                                                │
│                                                                  │
│ ⚠️ ЖДЁМ ЗАВЕРШЕНИЯ (block=true, timeout=600000)                │
│                                                                  │
│ Выход: docs/develop/{feature_id}/IMPLEMENTATION_REPORT.md       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Шаг 2+3: test-engineer + code-reviewer (ПАРАЛЛЕЛЬНО) ⚡        │
├─────────────────────────────────────────────────────────────────┤
│ [ОДНО сообщение с ДВУМЯ Task вызовами]                          │
│                                                                  │
│ Task(                                                            │
│   subagent_type="test-engineer",                                │
│   prompt="Протестируй реализованную фичу {feature_id}..."       │
│ )                                                                │
│ Task(                                                            │
│   subagent_type="code-reviewer",                                │
│   prompt="Выполни code review для фичи {feature_id}..."         │
│ )                                                                │
│                                                                  │
│ ⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ОБИХ (block=true для каждого)               │
│                                                                  │
│ Выход: TEST_REPORT.md + CODE_REVIEW.md                          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Шаг 4: feature-verifier (ПОСЛЕДОВАТЕЛЬНО после ОБИХ)           │
├─────────────────────────────────────────────────────────────────┤
│ Task(                                                            │
│   subagent_type="feature-verifier",                             │
│   prompt="Выполни финальную верификацию фичи {feature_id}..."    │
│ )                                                                │
│                                                                  │
│ ⚠️ ЖДЁМ ЗАВЕРШЕНИЯ (block=true, timeout=600000)                │
│                                                                  │
│ Требует: TEST_REPORT.md И CODE_REVIEW.md                        │
│ Выход: docs/develop/{feature_id}/FEATURE_VERIFICATION.md        │
│ ИЗВЛЕКАЕМ: score из FEATURE_VERIFICATION.md                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                    ┌─────────────────┐
                    │ ПРОВЕРКА SCORE  │
                    └─────────────────┘
                              ↓
              ┌───────────────┴───────────────┐
              │                               │
        score ≥ 9                      score < 9
              │                               │
              ↓                               ↓
┌─────────────────────────┐     ┌───────────────────────────────┐
│ Шаг 5a: Успех            │     │ Шаг 5b: Доработка             │
├─────────────────────────┤     ├───────────────────────────────┤
│                          │     │ - CODE_REVIEW.md              │
│ Фича завершена ✅        │     │ - TEST_REPORT.md              │
│ Переход к следующей     │     │ Содержат задачи для доработки │
└─────────────────────────┘     └───────────────────────────────┘
```

---

## Визуальная схема флоу

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLOW ДЛЯ ОДНОЙ ФИЧИ                          │
└─────────────────────────────────────────────────────────────────┘

    developer-agent          (последовательно)
           │
           ▼
    ┌──────┴──────┐
    │             │
    ▼             ▼
test-engineer  code-reviewer   (ПАРАЛЛЕЛЬНО ⚡)
    │             │
    └──────┬──────┘
           │
           ▼
  feature-verifier             (последовательно, после ОБИХ)
           │
           ▼
      score ≥ 9?
           │
     ┌─────┴─────┐
     │           │
    ДА          НЕТ
     │           │
     ▼           ▼
  Оркестратор делает коммит   developer-agent (повтор с контекстом доработки)
                  │
                  ▼ (цикл повторяется)
```

**Ключевые моменты:**
1. Developer идёт первым (создаёт код)
2. Test + Review идут параллельно (независимые проверки)
3. Verifier идёт последним (консолидирует ОБА результата)
4. При score < 9 — возврат к developer с задачами доработки

---

## Правила выполнения

### Последовательность и параллелизм (КРИТИЧЕСКО)

✅ **ПРАВИЛЬНО** (оптимизировано):
```python
# Шаг 1: Developer (последовательно)
task_1 = Task(subagent_type="developer-agent", prompt="...")
result_1 = TaskOutput(task_id=task_1["id"], block=True, timeout=600000)

# Шаг 2+3: Test Engineer + Code Reviewer (ПАРАЛЛЕЛЬНО в одном сообщении)
task_2 = Task(subagent_type="test-engineer", prompt="...")
task_3 = Task(subagent_type="code-reviewer", prompt="...")
# ОДНО сообщение с двумя Task вызовами = параллельный запуск ⚡

result_2 = TaskOutput(task_id=task_2["id"], block=True, timeout=600000)
result_3 = TaskOutput(task_id=task_3["id"], block=True, timeout=600000)

# Шаг 4: Verifier (последовательно, ПОСЛЕ ОБИХ предыдущих)
task_4 = Task(subagent_type="feature-verifier", prompt="...")
result_4 = TaskOutput(task_id=task_4["id"], block=True, timeout=600000)
```

❌ **НЕПРАВИЛЬНО #1** (все параллельно):
```python
# НЕ ДЕЛАЙ ТАК!
Task(subagent_type="developer-agent", ...)  # ← Не ждём завершения
Task(subagent_type="test-engineer", ...)    # ← Запускается сразу
Task(subagent_type="code-reviewer", ...)    # ← Тоже параллельно
Task(subagent_type="feature-verifier", ...) # ← Хаос! Зависит от предыдущих!
```

❌ **НЕПРАВИЛЬНО #2** (не используем параллелизм):
```python
# Работает, но медленно - test и review могли работать параллельно
Task(subagent_type="developer-agent", ...)
TaskOutput(..., block=True)  # ← ждём

Task(subagent_type="test-engineer", ...)
TaskOutput(..., block=True)  # ← ждём (упущенная возможность!)

Task(subagent_type="code-reviewer", ...)
TaskOutput(..., block=True)  # ← ждём (могли работать параллельно)

Task(subagent_type="feature-verifier", ...)
TaskOutput(..., block=True)
```

⚡ **ОПТИМАЛЬНО** (используем параллелизм там где возможно):
- Developer — последовательно (создаёт код)
- Test + Review — **параллельно** (независимые проверки)
- Verifier — последовательно (консолидирует результаты ОБИХ)

---

### Доработка при score < 9

Если `feature-verifier` вернул `score < 9`:

1. Прочитать `FEATURE_VERIFICATION.md` — список задач на доработку
2. Прочитать `CODE_REVIEW.md` — замечания review
3. Прочитать `TEST_REPORT.md` — проблемы в тестах
4. Перезапустить `developer-agent` с контекстом доработки:

```python
Task(
    subagent_type="developer-agent",
    prompt=f"""
    ПЕРЕРАБОТКА ЗАДАЧИ {task_id}

    Текущая реализация получила score {score}/10.

    ЗАДАЧИ НА ДОРАБОТКУ (из FEATURE_VERIFICATION.md):
    [вставить задачи из верификации]

    ЗАМЕЧАНИЯ CODE REVIEW (из CODE_REVIEW.md):
    [вставить замечания]

    ПРОБЛЕМЫ В ТЕСТАХ (из TEST_REPORT.md):
    [вставить проблемы]

    Выполни доработку согласно этим задачам.
    Обнови IMPLEMENTATION_REPORT_{task_id}.md с описанием изменений.
    """
)
```

5. Повторить весь цикл: test → review → verify
6. Повторять пока `score < 9`

---

### Завершение задачи (score ≥ 9)

После успешной верификации задачи (score ≥ 9):
- Оркестратор делает коммит в ветку `feature/<name>`
- Задача считается завершенной
- Переход к следующей задаче в фиче

---

### Завершение фичи

После успешного завершения **ВСЕХ задач** фичи:
- Оркестратор делает merge `feature/<name>` → `{MAIN_BRANCH}`
- Ветка `feature/<name>` удаляется
- Переход к следующей фиче


---

**Промпт для каждого агента:**

```
Task ID: {task_id}
Task Name: {task_name}
Feature: {feature_name}
Domain: {feature_domain}

Директория для артефактов: docs/develop/{feature}/{task_id}/

Выполни свою роль для этой задачи согласно твоим инструкциям в .md файле.
Все создаваемые артефакты сохраняй в указанную директорию.
```

**ЗАПРЕЩЕНО:**
- ❌ Передавать несколько задач в один агент
- ❌ Группировать задачи по версии для developer/test/review
- ❌ Запускать developer для всех задач разом
- ❌ Использовать feature-verifier для целой фичи
- ❌ Создавать артефакты вне `docs/develop/<FEATURE>/<TASK>/`
- ❌ **КРИТИЧЕСКО: Создавать только IMPLEMENTATION_REPORT и пропускать test/review/verify!**
  - КАЖДАЯ задача ДОЛЖНА иметь ВСЕ 4 артефакта
  - "Усталость" оркестатора — НЕ оправдание
- ❌ **КРИТИЧЕСКО: Запускать feature-verifier ДО test-engineer и code-reviewer!**
  - feature-verifier зависит от ОБИХ: TEST_REPORT.md И CODE_REVIEW.md
- ❌ **КРИТИЧЕСКО: Запускать test-engineer или code-reviewer ДО developer-agent!**
  - Они требуют IMPLEMENTATION_REPORT.md от developer
- ❌ **Запускать все 4 агента в одном сообщении** (developer/test/review/verify)
- ❌ **Разрабатывать более 3 задач параллельно**
- ❌ **Игнорировать зависимости между задачами**
  - Зависимая задача НЕ может разрабатываться до родительской
- ❌ **Разрабатывать несколько фич параллельно**
  - Фичи выполняются ТОЛЬКО ПОСЛЕДОВАТЕЛЬНО

**РАЗРЕШЕНО (оптимизация):**
- ✅ **Запускать test-engineer и code-reviewer параллельно** (в одном сообщении)
  - Они независимы и могут работать одновременно
  - feature-verifier запускается только ПОСЛЕ завершения ОБИХ
- ✅ **Разрабатывать до 3 задач параллельно** (при отсутствии зависимостей)
  - Каждая задача проходит ПОЛНЫЙ цикл разработки
  - Задачи должны быть одного Level (без зависимостей друг от друга)

---

## Пример параллельной разработки 3 задач внутри фичи

```
ФИЧА: Authentication System
Ветка: feature/auth-system

Level 0 (нет зависимостей):
┌─────────────────────────────────────────────────────────────────┐
│ ЗАДАЧИ T-001, T-002, T-003 — ПАРАЛЛЕЛЬНО                      │
└─────────────────────────────────────────────────────────────────┘

[ОДНО сообщение с ТРЁМЯ developer-agent Task]
Task(developer-agent, prompt="...T-001...")  ─┐
Task(developer-agent, prompt="...T-002...")  ─┼─ ПАРАЛЛЕЛЬНО
Task(developer-agent, prompt="...T-003...")  ─┘

⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ВСЕХ ТРЁХ

[ОДНО сообщение с ШЕСТЬЮ Task — test + review для каждой задачи]
Task(test-engineer, "...T-001...")           ─┐
Task(code-reviewer, "...T-001...")          ─┤
Task(test-engineer, "...T-002...")           ─┼─ ПАРАЛЛЕЛЬНО
Task(code-reviewer, "...T-002...")          ─┤
Task(test-engineer, "...T-003...")           ─┤
Task(code-reviewer, "...T-003...")          ─┘

⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ВСЕХ ШЕСТИ

[ТРИ сообщения для verifier — последовательно для каждой задачи]
Task(feature-verifier, "...T-001...")  → score ≥ 9 → КОММИТ
Task(feature-verifier, "...T-002...")  → score ≥ 9 → КОММИТ
Task(feature-verifier, "...T-003...")  → score ≥ 9 → КОММИТ

⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ВСЕХ ТРЁХ

Для каждой задачи с score < 9 — повтор цикла
Для каждой задачи с score ≥ 9 — оркестратор сделал коммит

⚠️ ТОЛЬКО ПОСЛЕ ВСЕХ ЗАДАЧ фичи → merge feature/auth → {MAIN_BRANCH}
```

---

## Проверка зависимостей перед разработкой задачи

**Перед запуском разработки задачи ОБЯЗАТЕЛЬНО:**

1. Прочитать `docs/roadmaps/ROADMAP_TASKS_{feature}.md`
2. Найти секцию **Dependencies** для задачи
3. Проверить что все зависимые задачи:
   - Реализованы (есть IMPLEMENTATION_REPORT)
   - Прошли верификацию (score ≥ 9)
   - Есть коммиты в ветке фичи

**Если зависимости не выполнены:**
- НЕ запускать разработку этой задачи
- Перейти к следующей задаче без зависимостей
- Вернуться к зависимой задаче позже

**Завершение Фазы 6 (после ВСЕХ фич):**

Все фичи разработаны, смержены в {MAIN_BRANCH}. Фаза 6 завершена.

**Правила:**
- Фичи выполняются ПОСЛЕДОВАТЕЛЬНО (одна за другой)
- Каждая фича = отдельная ветка `feature/<name>`
- Задачи внутри фичи выполняются параллельно (до 3 штук)
- После завершения ВСЕХ задач фичи → merge в {MAIN_BRANCH}
- Только после завершения ВСЕХ фич переходи к Фазе 7

---

### Фаза 7 — Системная верификация

Выполни через Task tool: `system-verifier`

**Агент сам сделает коммит** после создания артефакта.

Выход: `docs/project/SYSTEM_VERIFICATION.md`

Если score < 9:
- возврат к блокирующей стадии

---

### Фаза 8 — Документация

Выполни через Task tool: `documentation-agent`

**Агент сам сделает коммит** после создания артефактов.

Выход: `docs/project/README.md`, `docs/project/ARCHITECTURE.md`, `docs/project/USAGE.md`

---

### Фаза 9 — Релиз

Выполни через Task tool: `release-devops`

**Агент сам сделает финальный коммит** после создания артефактов.

Выход: `docs/project/DEPLOY.md`, `docs/project/RELEASE_NOTES.md`

**🎉 Пайплайн завершён! Продукт создан и задокументирован.**

---

### Фаза 9 — Релиз

Выполни через Task tool: `release-devops`

**Агент сам сделает финальный коммит** после создания артефактов.

Выход: `docs/project/DEPLOY.md`, `docs/project/RELEASE_NOTES.md`

**🎉 Пайплайн завершён! Продукт создан и задокументирован.**
