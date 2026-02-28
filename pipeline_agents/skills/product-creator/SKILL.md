---
name: product-creator
description: Запускает полный мультиагентный пайплайн создания нового продукта от идеи до релиза
---

# PRODUCT_CREATOR — Создание нового продукта

## Версия
- version: 3.2.0
- standalone: true
- purpose: product_creation

## Изменения v3.2.0
- Добавлен обязательный агент build-run-verifier для проверки сборки и запуска кода
- Добавлена фаза Build & Run Verification в цикл разработки каждой задачи
- Обновлён flow задачи: developer → build-run-verifier → test + review → feature-verifier
- Добавлена автоматическая адаптация проверок под разные платформы (IntelliJ, Docker, Rust, etc.)
- Build/Run FAIL теперь блокирует переход к test-engineer и code-reviewer

## Изменения v3.1.0
- Добавлена обработка ошибок при запуске агентов (safe_launch_agent)
- Добавлен retry с exponential backoff для failed агентов
- Установлен лимит параллельных агентов = 5 (было 3)
- Добавлена функция log_failed_task для регистрации неудачных задач
- Добавлена функция verify_tdd_planning_complete для проверки создания roadmap
- Добавлена функция run_tdd_planner_with_retry для TDD планирования с обработкой ошибок

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

### 5. Build & Run Verification Gate (с v3.2.0)
- **ОБЯЗАТЕЛЬНЫЙ шаг** после developer-agent для КАЖДОЙ задачи
- Проверяет что код успешно собирается и запускается
- **Если Build/Run FAIL → возврат к developer-agent**
- **НЕ переходить к test-engineer и code-reviewer** пока Build/Run не пройден
- Адаптируется под платформу проекта (IntelliJ Plugin, Docker, Rust, Node.js, etc.)

**Поддерживаемые платформы:**

| Platform | Build Command | Run Command |
|----------|---------------|-------------|
| IntelliJ Plugin | `./gradlew buildPlugin` | `./gradlew runIde` |
| Kotlin/Spring Boot | `./gradlew build` | `./gradlew bootRun` |
| Rust | `cargo build` | `cargo run` |
| Node.js | `npm run build` | `npm start` |
| Python | `pip install -r requirements.txt` | `python main.py` |
| Docker | `docker build -t app .` | `docker run app` |
| React/TypeScript | `npm run build` | `npm run dev` |

### 6. Отображение артефактов для Human-in-the-Loop
При показе PROJECT_PROFILE_HUMAN.md или PIPELINE_PROMPT.md:
- Прочитать файл через Read tool
- Включить ПОЛНОЕ содержимое в текстовый ответ между "---"
- Запросить явное подтверждение через AskUserQuestion

### 7. Human-in-the-Loop
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
            ├── BUILD_RUN_VERIFICATION_<FEATURE>_<TASK>.md  # (с v3.2.0)
            ├── TEST_REPORT_<TASK>.md
            ├── CODE_REVIEW_<TASK>.md
            └── FEATURE_VERIFICATION_<TASK>.md

README.md                 # Главный файл проекта (в корне!)
```

**Правила:**
- **README.md → в корне проекта** (главный файл документации)
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

---

## ⚠️ ОБРАБОТКА ОШИБОК ПРИ ЗАПУСКЕ АГЕНТОВ

### Типичные ошибки

| Ошибка | Признак | Решение |
|--------|---------|---------|
| **Empty result** | Агент вернул только ID или пустую строку | Retry с exponential backoff |
| **Rate limit** | Ошибка API, timeout | Подождать 30-60 сек, retry |
| **Agent crash** | TaskOutput вернул ошибку | Retry до 3 раз |
| **Invalid output** | Артефакт не создан или пустой | Retry с уточнением промпта |

### Функция безопасного запуска агента

```python
def safe_launch_agent(subagent_type, prompt, max_retries=3, base_delay=30):
    """
    Безопасный запуск агента с обработкой ошибок и retry.

    Returns:
        str: Результат работы агента или None после всех попыток
    """
    import time

    for attempt in range(max_retries):
        try:
            # Запускаем агент
            task = Task(
                subagent_type=subagent_type,
                prompt=prompt
            )

            # Ждём результат с timeout
            result = TaskOutput(
                task_id=task["id"],
                block=True,
                timeout=600000  # 10 минут
            )

            # Проверяем валидность результата
            if result is None or result.strip() == "" or len(result) < 100:
                print(f"⚠️ Попытка {attempt + 1}: агент вернул пустой или короткий результат")

                # Exponential backoff
                delay = base_delay * (2 ** attempt)
                print(f"   Ожидание {delay} сек перед retry...")
                time.sleep(delay)
                continue

            # Проверяем что артефакт создан (если применимо)
            if "ROADMAP" in prompt or "REPORT" in prompt:
                # Проверяем создание файла
                expected_files = extract_expected_files(prompt)
                for filepath in expected_files:
                    if not file_exists(filepath):
                        print(f"⚠️ Файл не создан: {filepath}")
                        delay = base_delay * (2 ** attempt)
                        time.sleep(delay)
                        continue

            return result

        except Exception as e:
            error_msg = str(e).lower()

            if "rate limit" in error_msg or "429" in error_msg:
                print(f"⚠️ Rate limit, ожидание 60 сек...")
                time.sleep(60)
                continue

            if "timeout" in error_msg:
                print(f"⚠️ Timeout, retry...")
                time.sleep(30)
                continue

            print(f"❌ Ошибка агента: {e}")
            delay = base_delay * (2 ** attempt)
            time.sleep(delay)

    print(f"❌ Агент не смог выполнить задачу после {max_retries} попыток")
    return None
```

### Обработка для TDD Planner

```python
def run_tdd_planner_with_retry(feature, max_retries=3):
    """Запуск TDD Planner с обработкой ошибок."""

    for attempt in range(max_retries):
        result = safe_launch_agent(
            subagent_type="tdd-planner",
            prompt=f"""
Создай TDD roadmap для фичи:

Feature ID: {feature['id']}
Feature Name: {feature['name']}
Feature Description: {feature['description']}
Domain: {feature['domain']}
Dependencies: {feature['dependencies']}

⚠️ ОБЯЗАТЕЛЬНО:
1. Создай файл docs/roadmaps/ROADMAP_TASKS_{feature['id']}.md
2. Убедись что файл не пустой
3. Сделай git commit

После создания — подтверди полный путь к созданному файлу.
""",
            max_retries=1  # Однократный retry внутри safe_launch
        )

        if result is None or result.strip() == "":
            print(f"⚠️ TDD Planner для {feature['id']}: пустой результат (попытка {attempt + 1})")
            time.sleep(30 * (2 ** attempt))
            continue

        # Проверяем что roadmap создан
        roadmap_path = f"docs/roadmaps/ROADMAP_TASKS_{feature['id']}.md"
        if file_exists(roadmap_path) and file_size(roadmap_path) > 500:
            print(f"✅ Roadmap создан: {roadmap_path}")
            return True
        else:
            print(f"⚠️ Roadmap не найден или пуст: {roadmap_path}")
            time.sleep(30)

    # Все попытки исчерпаны
    print(f"❌ TDD Planner не смог создать roadmap для {feature['id']}")
    log_failed_task(feature['id'], "tdd-planner", "empty result after retries")
    return False
```

### Логирование неудачных задач

```python
def log_failed_task(task_id, agent_type, error_reason):
    """Сохранить информацию о неудачной задаче для ручной обработки."""

    log_entry = f"""
## Failed Task: {task_id}
- Agent: {agent_type}
- Error: {error_reason}
- Timestamp: {datetime.now()}
- Action: MANUAL REVIEW REQUIRED
"""

    append_to_file("docs/project/FAILED_TASKS.md", log_entry)
```

### Проверка после этапа TDD планирования

```python
def verify_tdd_planning_complete(features):
    """Проверить что все roadmap созданы."""

    failed_features = []

    for feature in features:
        roadmap_path = f"docs/roadmaps/ROADMAP_TASKS_{feature['id']}.md"

        if not file_exists(roadmap_path):
            failed_features.append(feature['id'])
        elif file_size(roadmap_path) < 500:
            failed_features.append(f"{feature['id']} (empty/small)")

    if failed_features:
        print(f"⚠️ Roadmap не созданы для фич: {', '.join(failed_features)}")
        return False

    return True
```

---

### Фаза 6 — Реализация ФИЧ (ПОСЛЕДОВАТЕЛЬНО) и ЗАДАЧ (ПАРАЛЛЕЛЬНО)

**⚠️ КРИТИЧЕСКИ ВАЖНО:**
- **ФИЧИ выполняются ПОСЛЕДОВАТЕЛЬНО** (одна за другой)
- **ЗАДАЧИ внутри фичи могут выполняться ПАРАЛЛЕЛЬНО** (до 3 штук)
- Каждая фича = отдельная ветка `feature/<name>`
- После завершения всех задач фичи → merge в `{MAIN_BRANCH}`

**⚠️ ГЛОБАЛЬНЫЙ ЛИМИТ: Максимум 5 агентов одновременно во всём пайплайне.**

---

## ⚠️ КРИТИЧЕСКИ ВАЖНЫЕ ТРЕБОВАНИЯ

### 1. Полный цикл для КАЖДОЙ задачи (НЕПРИЕМЛЕМО ПРОПУСКАТЬ ШАГИ)

**КАЖДАЯ задача ДОЛЖНА иметь ВСЕ артефакты:**
- ✅ `IMPLEMENTATION_REPORT_<task>.md` — от developer-agent
- ✅ `BUILD_RUN_VERIFICATION_<feature>_<task>.md` — от build-run-verifier (ОБЯЗАТЕЛЬНО с v3.2.0)
- ✅ `TEST_REPORT_<task>.md` — от test-engineer
- ✅ `CODE_REVIEW_<task>.md` — от code-reviewer
- ✅ `FEATURE_VERIFICATION_<task>.md` — от feature-verifier (с score ≥ 9)

❌ **ЗАПРЕЩЕНО:**
- Создавать только IMPLEMENTATION_REPORT и пропускать остальные
- Пропускать Build & Run Verification — это ОБЯЗАТЕЛЬНЫЙ шаг
- "Устать" и делать задачи в ускоренном режиме
- Создавать задачи больше 4-6 часов работы
- Переходить к test-engineer если build-run-verifier вернул FAIL

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
| build-run-verifier | Проверяет сборку и запуск | ❌ Нет (оркестратор) |
| test-engineer | Тестирует задачу | ❌ Нет (оркестратор) |
| code-reviewer | Делает ревью задачи | ❌ Нет (оркестратор) |
| feature-verifier | Верифицирует задачу | ❌ Нет (оркестратор) |
| **Оркестратор** | **Делает коммит после каждой задачи (score ≥ 9)** | ✅ **Да** |
| **Оркестратор** | **Делает merge фазы после всех задач** | ✅ **Да** |

### Когда делать коммит

**Разработка задачи (в ветке feature/<name>):**
```
developer-agent → build-run-verifier → test-engineer → code-reviewer → feature-verifier
                        ↓ FAIL                          (параллельно)           ↓
                  возврат к developer                                    score ≥ 9?
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
        # ШАГ 3.2: build-run-verifier (ПОСЛЕДОВАТЕЛЬНО для каждой задачи)
        # ⚠️ ОБЯЗАТЕЛЬНЫЙ шаг с v3.2.0
        # ─────────────────────────────────────────────────────────────────
        tasks_passed_build_run = []
        for task in level_tasks:
            task_build_run = Task(
                subagent_type="build-run-verifier",
                prompt=f"Проверь что код задачи {task['id']} собирается и запускается"
            )
            result = TaskOutput(task_id=task_build_run["id"], block=True, timeout=600000)

            # Проверяем Build & Run статус
            if "PASS" in result:
                tasks_passed_build_run.append(task)
            else:
                # Build/Run FAIL — возврат к developer-agent
                print(f"❌ {task['id']}: Build/Run FAIL — возврат на доработку")
                task_dev_retry = Task(
                    subagent_type="developer-agent",
                    prompt=f"""
                    ИСПРАВЛЕНИЕ ОШИБОК СБОРКИ/ЗАПУСКА для {task['id']}

                    {result}

                    Исправь код чтобы он успешно собирался и запускался.
                    """
                )
                # После исправления — повтор build-run-verifier
                # ... (рекурсивно или через цикл)

        # Продолжаем только с задачами которые прошли Build & Run
        level_tasks = tasks_passed_build_run

        # ─────────────────────────────────────────────────────────────────
        # ШАГ 3.3: test-engineer + code-reviewer (ПАРАЛЛЕЛЬНО)
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
        # ШАГ 3.4: feature-verifier (ПОСЛЕДОВАТЕЛЬНО для каждой задачи)
        # ─────────────────────────────────────────────────────────────────
        for task in level_tasks:
            task_verify = Task(subagent_type="feature-verifier", prompt=f"Верифицируй {task['id']}")
            result = TaskOutput(task_id=task_verify["id"], block=True, timeout=600000)

            # Проверяем score
            score = extract_score(result)

            if score >= 9:
                # ─────────────────────────────────────────────────────────────────
                # ШАГ: Отметить задачу в roadmap как выполненную
                # ─────────────────────────────────────────────────────────────────
                roadmap_path = f"docs/roadmaps/ROADMAP_TASKS_{feature_id}.md"
                roadmap = read_file(roadmap_path)
                updated_roadmap = mark_task_completed(roadmap, task['id'], score)
                write_file(roadmap_path, updated_roadmap)

                bash_command(f"""
                    git add {roadmap_path}
                    git commit -m "docs: mark task {task['id']} as completed (score {score}/10)"
                """)

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

    # ─────────────────────────────────────────────────────────────────
    # ШАГ: Отметить фичу в FEATURES_INDEX.md как выполненную
    # ─────────────────────────────────────────────────────────────────
    features_index_path = "docs/project/FEATURES_INDEX.md"
    features_index = read_file(features_index_path)
    updated_index = mark_feature_completed(features_index, feature_id)
    write_file(features_index_path, updated_index)

    bash_command(f"""
        git add {features_index_path}
        git commit -m "docs: mark feature {feature_id} as completed"
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

---

## ⚠️ ОТМЕТКА ВЫПОЛНЕННЫХ ЗАДАЧ И ФИЧ В РОАДМАПАХ (ОБЯЗАТЕЛЬНО)

### Правило отметки выполненных задач

**ПОСЛЕ успешной верификации каждой задачи (score ≥ 9) оркестратор ОБЯЗАН:**

1. **Отметить задачу как выполненную в roadmap:**
   - Прочитать `docs/roadmaps/ROADMAP_TASKS_<feature>.md`
   - Найти секцию задачи `Task T-XXX`
   - Добавить/обновить статус: `**Status:** ✅ COMPLETED`
   - Добавить дату завершения: `**Completed:** YYYY-MM-DD`
   - Добавить финальный score: `**Final Score:** X/10`

2. **Сделать коммит с обновлённым roadmap:**
   ```bash
   git add docs/roadmaps/ROADMAP_TASKS_<feature>.md
   git commit -m "docs: mark task {task_id} as completed (score {score}/10)"
   ```

### Формат отметки задачи в roadmap

```markdown
### Task T-001: <Task Name>

**Description:** [описание]
**Estimated Time:** 2-4 hours
**Dependencies:** None
**Status:** ✅ COMPLETED
**Completed:** 2025-01-15
**Final Score:** 9/10

**Scope:**
- **In scope:** [что входит]
- **Out scope:** [что НЕ входит]

[... остальное содержимое задачи ...]
```

### Правило отметки выполненной фичи

**ПОСЛЕ успешного merge фичи в {MAIN_BRANCH} оркестратор ОБЯЗАН:**

1. **Отметить фичу как выполненную в FEATURES_INDEX.md:**
   - Прочитать `docs/project/FEATURES_INDEX.md`
   - Найти секцию фичи `Feature F-XXX`
   - Добавить/обновить статус: `**Status:** ✅ COMPLETED`
   - Добавить дату завершения: `**Completed:** YYYY-MM-DD`

2. **Сделать финальный коммит:**
   ```bash
   git add docs/project/FEATURES_INDEX.md
   git commit -m "docs: mark feature {feature_id} as completed"
   ```

### Формат отметки фичи в FEATURES_INDEX.md

```markdown
## Feature F-001

- **Name:** <Feature name>
- **Description:** <Brief description>
- **Domain:** <Domain ID>
- **Related Requirements:** <FR-IDs>
- **Dependencies:** <List or None>
- **Status:** ✅ COMPLETED
- **Completed:** 2025-01-15
- **Notes:** <Additional notes>
```

### Псевдокод отметки задачи после верификации

```python
# ─────────────────────────────────────────────────────────────────
# ШАГ 3.3: feature-verifier (ПОСЛЕДОВАТЕЛЬНО для каждой задачи)
# ─────────────────────────────────────────────────────────────────
for task in level_tasks:
    task_verify = Task(subagent_type="feature-verifier", prompt=f"Верифицируй {task['id']}")
    result = TaskOutput(task_id=task_verify["id"], block=True, timeout=600000)

    # Проверяем score
    score = extract_score(result)

    if score >= 9:
        # ─────────────────────────────────────────────────────────────────
        # ШАГ 3.4: Отметить задачу в roadmap как выполненную
        # ─────────────────────────────────────────────────────────────────
        roadmap_path = f"docs/roadmaps/ROADMAP_TASKS_{feature_id}.md"
        roadmap = read_file(roadmap_path)

        # Добавляем статус задачи
        updated_roadmap = mark_task_completed(roadmap, task['id'], score)

        # Записываем обновлённый roadmap
        write_file(roadmap_path, updated_roadmap)

        # Коммит с обновлённым roadmap
        bash_command(f"""
            git add {roadmap_path}
            git commit -m "docs: mark task {task['id']} as completed (score {score}/10)"
        """)

        # Оркестратор делает коммит задачи
        bash_command(f"""
            git add docs/develop/{feature_id}/{task['id']}/
            git commit -m "feat: {task['name']} ({task['id']})"
        """)
        print(f"✅ {task['id']}: коммит создан (score={score})")
```

### Псевдокод отметки фичи после merge

```python
# ─────────────────────────────────────────────────────────────────
# ЭТАП 4: ВСЕ задачи фичи завершены (score ≥ 9)
# ─────────────────────────────────────────────────────────────────

# Merge фичи в {MAIN_BRANCH}
bash_command(f"""
    git checkout {MAIN_BRANCH}
    git merge feature/{feature_id}
    git branch -d feature/{feature_id}
""")

# ─────────────────────────────────────────────────────────────────
# ШАГ 4.1: Отметить фичу в FEATURES_INDEX.md как выполненную
# ─────────────────────────────────────────────────────────────────
features_index_path = "docs/project/FEATURES_INDEX.md"
features_index = read_file(features_index_path)

# Добавляем статус фичи
updated_index = mark_feature_completed(features_index, feature_id)

# Записываем обновлённый индекс
write_file(features_index_path, updated_index)

# Коммит с обновлённым индексом
bash_command(f"""
    git add {features_index_path}
    git commit -m "docs: mark feature {feature_id} as completed"
""")

print(f"✅ Фича {feature_id} завершена и смержена")
```

### Функции отметки (псевдокод)

```python
def mark_task_completed(roadmap_content, task_id, score):
    """Добавляет статус выполненной задачи в roadmap"""
    from datetime import datetime

    completed_date = datetime.now().strftime("%Y-%m-%d")

    # Находим секцию задачи
    task_section = find_task_section(roadmap_content, task_id)

    # Если статус уже есть — обновляем, иначе добавляем
    if "**Status:**" in task_section:
        # Обновляем существующий статус
        updated = task_section.replace(
            "**Status:** ⏳ IN PROGRESS",
            f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Final Score:** {score}/10"
        )
    else:
        # Добавляем новый статус после заголовка задачи
        updated = task_section.replace(
            f"### Task {task_id}:",
            f"### Task {task_id}:\n**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Final Score:** {score}/10"
        )

    # Заменяем в roadmap
    return roadmap_content.replace(task_section, updated)


def mark_feature_completed(features_index_content, feature_id):
    """Добавляет статус выполненной фичи в FEATURES_INDEX.md"""
    from datetime import datetime

    completed_date = datetime.now().strftime("%Y-%m-%d")

    # Находим секцию фичи
    feature_section = find_feature_section(features_index_content, feature_id)

    # Если статус уже есть — обновляем, иначе добавляем
    if "**Status:**" in feature_section:
        # Обновляем существующий статус
        updated = feature_section.replace(
            "**Status:** ⏳ IN PROGRESS",
            f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}"
        )
    else:
        # Добавляем новый статус после Dependencies
        updated = feature_section.replace(
            f"**Notes:**",
            f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Notes:**"
        )

    # Заменяем в FEATURES_INDEX
    return features_index_content.replace(feature_section, updated)
```

---
7. **⚠️ ОБЯЗАТЕЛЬНО: Отмечать выполненные задачи в roadmap после score ≥ 9**
8. **⚠️ ОБЯЗАТЕЛЬНО: Отмечать выполненную фичу в FEATURES_INDEX.md после merge**


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
│                    FLOW ДЛЯ ОДНОЙ ЗАДАЧИ (v3.2.0)               │
└─────────────────────────────────────────────────────────────────┘

    developer-agent          (последовательно)
           │
           ▼
    build-run-verifier        (последовательно, ОБЯЗАТЕЛЬНО)
           │
           ├──── PASS ────┐
           │               │
           ▼               ▼
    ┌──────┴──────┐   FAIL → возврат к developer-agent
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
2. Build-run-verifier идёт вторым (проверяет что код собирается и запускается)
3. Если Build/Run FAIL → возврат к developer, НЕ продолжаем к test/review
4. Test + Review идут параллельно (независимые проверки)
5. Verifier идёт последним (консолидирует ОБА результата)
6. При score < 9 — возврат к developer с задачами доработки

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
│ 2. build-run-verifier (последовательно, ОБЯЗАТЕЛЬНО)           │
│    Task(subagent_type="build-run-verifier", ...)               │
│    ЖДЁМ ЗАВЕРШЕНИЯ (block=true)                                │
│    Выход: BUILD_RUN_VERIFICATION.md                             │
│    Если FAIL → возврат к developer-agent                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓ (только если PASS)
┌─────────────────────────────────────────────────────────────────┐
│ 3+4. test-engineer + code-reviewer (ПАРАЛЛЕЛЬНО)                │
│    [ОДНО сообщение с ДВУМЯ Task]                                │
│    ЖДЁМ ЗАВЕРШЕНИЯ ОБИХ (block=true)                           │
│    Выход: TEST_REPORT.md + CODE_REVIEW.md                       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 5. feature-verifier (последовательно)                           │
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
│ Шаг 2: build-run-verifier (ПОСЛЕДОВАТЕЛЬНО, ОБЯЗАТЕЛЬНО)       │
├─────────────────────────────────────────────────────────────────┤
│ Task(                                                            │
│   subagent_type="build-run-verifier",                           │
│   prompt="Проверь что код фичи {feature_id} собирается          │
│           и запускается..."                                      │
│ )                                                                │
│                                                                  │
│ ⚠️ ЖДЁМ ЗАВЕРШЕНИЯ (block=true, timeout=600000)                │
│                                                                  │
│ Выход: BUILD_RUN_VERIFICATION.md                                │
│ Если FAIL → возврат к developer-agent                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓ (только если PASS)
┌─────────────────────────────────────────────────────────────────┐
│ Шаг 3+4: test-engineer + code-reviewer (ПАРАЛЛЕЛЬНО) ⚡        │
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
│ Шаг 5: feature-verifier (ПОСЛЕДОВАТЕЛЬНО после ОБИХ)           │
├─────────────────────────────────────────────────────────────────┤
│ Task(                                                            │
│   subagent_type="feature-verifier",                             │
│   prompt="Выполни финальную верификацию фичи {feature_id}..."    │
│ )                                                                │
│                                                                  │
│ ⚠️ ЖДЁМ ЗАВЕРШЕНИЯ (block=true, timeout=600000)                │
│                                                                  │
│ Требует: BUILD_RUN_VERIFICATION.md, TEST_REPORT.md,             │
│          CODE_REVIEW.md                                          │
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
│ Шаг 6a: Успех            │     │ Шаг 6b: Доработка             │
├─────────────────────────┤     ├───────────────────────────────┤
│                          │     │ - CODE_REVIEW.md              │
│ Фича завершена ✅        │     │ - TEST_REPORT.md              │
│ Переход к следующей     │     │ - BUILD_RUN_VERIFICATION.md   │
└─────────────────────────┘     │ Содержат задачи для доработки │
                                └───────────────────────────────┘
```

---

## Визуальная схема флоу

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLOW ДЛЯ ОДНОЙ ФИЧИ (v3.2.0)                 │
└─────────────────────────────────────────────────────────────────┘

    developer-agent          (последовательно)
           │
           ▼
    build-run-verifier        (последовательно, ОБЯЗАТЕЛЬНО)
           │
           ├──── PASS ────┐
           │               │
           ▼               ▼
    ┌──────┴──────┐   FAIL → возврат к developer-agent
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
2. Build-run-verifier идёт вторым (проверяет что код собирается и запускается)
3. Если Build/Run FAIL → возврат к developer, НЕ продолжаем к test/review
4. Test + Review идут параллельно (независимые проверки)
5. Verifier идёт последним (консолидирует ОБА результата)
6. При score < 9 — возврат к developer с задачами доработки

---

## Правила выполнения

### Последовательность и параллелизм (КРИТИЧЕСКО)

✅ **ПРАВИЛЬНО** (оптимизировано):
```python
# Шаг 1: Developer (последовательно)
task_1 = Task(subagent_type="developer-agent", prompt="...")
result_1 = TaskOutput(task_id=task_1["id"], block=True, timeout=600000)

# Шаг 2: Build-run-verifier (последовательно, ОБЯЗАТЕЛЬНО)
task_2 = Task(subagent_type="build-run-verifier", prompt="...")
result_2 = TaskOutput(task_id=task_2["id"], block=True, timeout=600000)

# Проверяем что Build/Run прошёл
if "FAIL" in result_2:
    # Возврат к developer-agent
    task_1 = Task(subagent_type="developer-agent", prompt="Доработай код - не проходит сборку/запуск...")
    # ... повтор цикла

# Шаг 3+4: Test Engineer + Code Reviewer (ПАРАЛЛЕЛЬНО в одном сообщении)
task_3 = Task(subagent_type="test-engineer", prompt="...")
task_4 = Task(subagent_type="code-reviewer", prompt="...")
# ОДНО сообщение с двумя Task вызовами = параллельный запуск ⚡

result_3 = TaskOutput(task_id=task_3["id"], block=True, timeout=600000)
result_4 = TaskOutput(task_id=task_4["id"], block=True, timeout=600000)

# Шаг 5: Verifier (последовательно, ПОСЛЕ ОБИХ предыдущих)
task_5 = Task(subagent_type="feature-verifier", prompt="...")
result_5 = TaskOutput(task_id=task_5["id"], block=True, timeout=600000)
```

❌ **НЕПРАВИЛЬНО #1** (все параллельно):
```python
# НЕ ДЕЛАЙ ТАК!
Task(subagent_type="developer-agent", ...)       # ← Не ждём завершения
Task(subagent_type="build-run-verifier", ...)    # ← Запускается сразу - код ещё не готов!
Task(subagent_type="test-engineer", ...)         # ← Тоже параллельно
Task(subagent_type="code-reviewer", ...)         # ← Хаос!
Task(subagent_type="feature-verifier", ...)      # ← Зависит от предыдущих!
```

❌ **НЕПРАВИЛЬНО #2** (пропуск build-run-verifier):
```python
# НЕ ДЕЛАЙ ТАК! Build & Run verification - ОБЯЗАТЕЛЬНЫЙ шаг!
Task(subagent_type="developer-agent", ...)
TaskOutput(..., block=True)

# Пропуск build-run-verifier - НЕДОПУСТИМО!

Task(subagent_type="test-engineer", ...)  # ← Код может не собираться!
```

❌ **НЕПРАВИЛЬНО #3** (не используем параллелизм):
```python
# Работает, но медленно - test и review могли работать параллельно
Task(subagent_type="developer-agent", ...)
TaskOutput(..., block=True)  # ← ждём

Task(subagent_type="build-run-verifier", ...)
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
- Build-run-verifier — последовательно (проверяет сборку/запуск)
- Test + Review — **параллельно** (независимые проверки)
- Verifier — последовательно (консолидирует результаты ОБИХ)

---

### Доработка при score < 9

Если `feature-verifier` вернул `score < 9`:

1. Прочитать `FEATURE_VERIFICATION.md` — список задач на доработку
2. Прочитать `BUILD_RUN_VERIFICATION.md` — проблемы сборки/запуска (если есть)
3. Прочитать `CODE_REVIEW.md` — замечания review
4. Прочитать `TEST_REPORT.md` — проблемы в тестах
5. Перезапустить `developer-agent` с контекстом доработки:

```python
Task(
    subagent_type="developer-agent",
    prompt=f"""
    ПЕРЕРАБОТКА ЗАДАЧИ {task_id}

    Текущая реализация получила score {score}/10.

    ПРОБЛЕМЫ СБОРКИ/ЗАПУСКА (из BUILD_RUN_VERIFICATION.md):
    [вставить проблемы если есть]

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

6. Повторить весь цикл: build-run → test → review → verify
7. Повторять пока `score < 9`

### Доработка при Build/Run FAIL

Если `build-run-verifier` вернул `FAIL`:

1. Прочитать `BUILD_RUN_VERIFICATION.md` — детали ошибки сборки/запуска
2. Перезапустить `developer-agent` с контекстом исправления:

```python
Task(
    subagent_type="developer-agent",
    prompt=f"""
    ИСПРАВЛЕНИЕ ОШИБОК СБОРКИ/ЗАПУСКА для {task_id}

    Build Status: FAIL
    Run Status: FAIL (если применимо)

    ОШИБКИ СБОРКИ (из BUILD_RUN_VERIFICATION.md):
    [вставить ошибки компиляции]

    ОШИБКИ ЗАПУСКА (если есть):
    [вставить runtime ошибки]

    Исправь код чтобы он успешно собирался и запускался.
    Обнови IMPLEMENTATION_REPORT_{task_id}.md с описанием изменений.
    """
)
```

3. Повторить `build-run-verifier` после исправления
4. Только после PASS продолжать к test + review

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
- ❌ **КРИТИЧЕСКО: Пропускать build-run-verifier! (с v3.2.0 — ОБЯЗАТЕЛЬНЫЙ шаг)**
  - Build & Run Verification — обязательный этап после developer-agent
  - НЕ переходить к test-engineer если код не собирается/не запускается
- ❌ **КРИТИЧЕСКО: Создавать только IMPLEMENTATION_REPORT и пропускать verification/test/review!**
  - КАЖДАЯ задача ДОЛЖНА иметь ВСЕ 5 артефактов (с v3.2.0)
  - "Усталость" оркестратора — НЕ оправдание
- ❌ **КРИТИЧЕСКО: Запускать feature-verifier ДО test-engineer и code-reviewer!**
  - feature-verifier зависит от: BUILD_RUN_VERIFICATION.md, TEST_REPORT.md И CODE_REVIEW.md
- ❌ **КРИТИЧЕСКО: Запускать test-engineer или code-reviewer ДО build-run-verifier!**
  - Они требуют что код успешно собирался и запускался
- ❌ **КРИТИЧЕСКО: Запускать build-run-verifier ДО developer-agent!**
  - build-run-verifier требует IMPLEMENTATION_REPORT.md от developer
- ❌ **Запускать все 5 агентов в одном сообщении** (developer/build-run/test/review/verify)
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

[ТРИ сообщения для build-run-verifier — последовательно для каждой задачи]
Task(build-run-verifier, "...T-001...")  → PASS → продолжаем
Task(build-run-verifier, "...T-002...")  → PASS → продолжаем
Task(build-run-verifier, "...T-003...")  → FAIL → возврат к developer

⚠️ Если build-run FAIL → возврат к developer-agent, НЕ продолжаем к test/review

[ОДНО сообщение с ШЕСТЬЮ Task — test + review для каждой задачи (только PASS)]
Task(test-engineer, "...T-001...")           ─┐
Task(code-reviewer, "...T-001...")          ─┤
Task(test-engineer, "...T-002...")           ─┼─ ПАРАЛЛЕЛЬНО
Task(code-reviewer, "...T-002...")          ─┤
(test + review для T-003 — после исправления) ─┘

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
   - Прошли Build & Run Verification (есть BUILD_RUN_VERIFICATION с PASS)
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

Выход: `README.md` (в корне проекта), `docs/project/ARCHITECTURE.md`, `docs/project/USAGE.md`

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
