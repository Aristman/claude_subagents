---
name: feature-developer
description: Запускает разработку фичи в существующем проекте через декомпозицию на задачи и TDD roadmaps
---

# FEATURE_DEVELOPER — Разработка фичи в существующем проекте

## Версия
- version: 5.0.0
- standalone: true
- purpose: feature_development

## Изменения v5.0.0
- Добавлена обработка ошибок при запуске агентов (safe_launch_agent)
- Добавлен retry с exponential backoff для failed агентов
- Установлен лимит параллельных агентов = 5
- Добавлена функция log_failed_task для регистрации неудачных задач
- Добавлена функция verify_tdd_planning_complete для проверки создания roadmap

---

## Назначение

Этот скилл превращает ТЕБЯ (встроенный оркестратор Claude Code) в **оркестратор разработки одной фичи** в существующем проекте.

Ты отвечаешь за:
- **Декомпозицию фичи на задачи** (2-4 часа каждая)
- **TDD планирование для каждой задачи**
- **Реализацию задач** через мультиагентный пайплайн
- **Координацию и коммиты**

Ты работаешь как **мини-product-creator для одной фичи**, используя те же агенты и подходы.

---

## ⚠️ КЛЮЧЕВОЕ ОТЛИЧИЕ ОТ product-creator

| Аспект | product-creator | feature-developer |
|--------|-----------------|-------------------|
| **Масштаб** | Весь продукт с нуля | Одна фича в существующем проекте |
| **Фазы** | 9 фаз (0-9) | 4 этапа для фичи |
| **Декомпозиция** | FEATURES_INDEX (фичи проекта) | TASKS_INDEX (задачи одной фичи) |
| **Roadmaps** | ROADMAP_TASKS_<feature>.md | ROADMAP_TASKS_<feature>_<task>.md |
| **Ветка** | feature/<name> → {MAIN_BRANCH} | feature/<feature-name> → {MAIN_BRANCH} |

---

## ЭТАПЫ РАЗРАБОТКИ ФИЧИ

### Этап 0 — Приём намерения

**Действия:**
1. Получить описание фичи от пользователя
2. **Запросить фичевый путь для артефактов** через AskUserQuestion
   - Может быть пустым (артефакты в `docs/project/`, `docs/roadmaps/`, `docs/develop/`)
   - Может быть путём вида `features/auth/`, `modules/user/`, `episodes/season1/` и т.д.
   - Путь добавляется внутри `project/{FEATURE_PATH}`, `roadmaps/{FEATURE_PATH}`, `develop/{FEATURE_PATH}`
3. Понять границы фичи (что входит, что нет)
4. Определить текущую ветку разработки

**Пример запроса пути:**
```python
AskUserQuestion(
    questions=[{
        "question": "Укажи фичевый путь для сохранения артефактов (например: features/auth/, modules/user/). Оставь пустым для сохранения в docs/project/, docs/roadmaps/ без подпути:",
        "header": "Feature Path",
        "options": [
            {"label": "Пустой (docs/project/, docs/roadmaps/)", "description": "Артефакты в корне project/, roadmaps/"},
            {"label": "features/<name>/", "description": "Артефакты в project/features/<name>/, roadmaps/features/<name>/"},
            {"label": "modules/<name>/", "description": "Артефакты в project/modules/<name>/, roadmaps/modules/<name>/"}
        ],
        "multiSelect": False
    }]
)
```

**Выход:**
- Описание фичи в формате естественного языка
- `{FEATURE_PATH}` — фичевый путь (с trailing slash или пустой)

---

### Этап 1 — Декомпозиция фичи на задачи

**Задача:** Разбить фичу на мелкие задачи (2-4 часа каждая)

**Выполни через Task tool:**
```python
Task(
    subagent_type="feature-decomposer",
    prompt="""
Выполни декомпозицию ОДНОЙ фичи на задачи.

Фича:
{FEATURE_DESCRIPTION}

Требования:
1. Разбей фичу на 3-10 задач
2. Каждая задача = 2-4 часа работы
3. Задачи должны быть независимыми (минимум зависимостей)
4. Создай TASKS_INDEX.md в docs/project/{FEATURE_PATH}

**Формат TASKS_INDEX.md:**
```markdown
# Tasks Index for Feature: {feature_name}

## Feature Info
- Feature Name: ...
- Feature Description: ...
- Feature Path: {FEATURE_PATH}  <!-- Фичевый путь для артефактов -->

## Feature Info
- Feature Name: ...
- Feature Description: ...

## Task Breakdown

### Task T-001: <Task Name>
- Description: ...
- Estimated Time: 2-4 hours
- Dependencies: None
- Domain: ...

### Task T-002: <Task Name>
...

## Dependency Graph
[Граф зависимостей задач]
```
"""
)
```

**Обработка вопросов:**
Если agent создал `docs/project/{FEATURE_PATH}CLARIFICATION_NEEDED.md`:
```python
if exists(f"docs/project/{FEATURE_PATH}CLARIFICATION_NEEDED.md"):
    questions = parse_clarification_needed(f"docs/project/{FEATURE_PATH}CLARIFICATION_NEEDED.md")
    user_answers = AskUserQuestion(questions=questions["Вопросы"], ...)
    create_file(f"docs/project/{FEATURE_PATH}USER_ANSWERS.md", user_answers)
    remove(f"docs/project/{FEATURE_PATH}CLARIFICATION_NEEDED.md")

    # Перезапускаем агента с ответами
    Task(subagent_type="feature-decomposer", prompt="ПЕРЕЗАПУСК С ОТВЕТАМИ...")
    remove(f"docs/project/{FEATURE_PATH}USER_ANSWERS.md")
```

**Выход:** `docs/project/{FEATURE_PATH}TASKS_INDEX.md`

---

### Этап 2 — TDD планирование для ВСЕХ задач

**Задача:** Создать roadmap для каждой задачи

**⚠️ ОГРАНИЧЕНИЕ ПАРАЛЛЕЛИЗМА:** Максимум 5 агентов одновременно.

**Выполни батчами по 5 задач:**

```python
# Читаем TASKS_INDEX.md
tasks_index = read_file(f"docs/project/{FEATURE_PATH}TASKS_INDEX.md")
tasks = parse_tasks(tasks_index)

BATCH_SIZE = 5  # Максимум параллельных агентов

# Разбиваем на батчи
for batch_start in range(0, len(tasks), BATCH_SIZE):
    batch = tasks[batch_start:batch_start + BATCH_SIZE]

    # Запускаем агентов в батче (до 5 параллельно)
    for task in batch:
        result = safe_launch_agent(
            subagent_type="tdd-planner",
            prompt=f"""
Создай TDD roadmap для задачи:

Task ID: {task['id']}
Task Name: {task['name']}
Task Description: {task['description']}
Domain: {task['domain']}
Dependencies: {task['dependencies']}
Feature Path: {FEATURE_PATH}

Создай ROADMAP_TASKS_{feature}_{task_id}.md в docs/roadmaps/{FEATURE_PATH}

После создания — git commit.
"""
        )
        # Проверяем результат
        if result is None or result.strip() == "":
            log_error(f"Task {task['id']}: agent returned empty result")
            # Добавляем в список для повторного запуска
```

**Выход:** `docs/roadmaps/{FEATURE_PATH}ROADMAP_TASKS_<feature>_<task>.md` для каждой задачи

---

### Этап 3 — Создание ветки фичи

**Действия:**
```bash
# Создаём ветку для фичи
git checkout -b feature/<feature-name>
```

---

### Этап 4 — Реализация задач (ПОСЛЕДОВАТЕЛЬНО или ПАРАЛЛЕЛЬНО)

**⚠️ КРИТИЧЕСКИ ВАЖНО:**

**Задачи выполняются:**
- **Последовательно** — если есть зависимости
- **Параллельно** (до 5 задач/агентов) — если нет зависимостей

**⚠️ ГЛОБАЛЬНЫЙ ЛИМИТ: Максимум 5 агентов одновременно во всём пайплайне.**

**Алгоритм:**
```python
MAX_PARALLEL_AGENTS = 5

# Группируем задачи по зависимостям
task_groups = group_by_dependencies(tasks)

# Для каждой группы
for group in task_groups:
    if len(group) == 1:
        # Одна задача — последовательно
        implement_task(group[0])
    elif len(group) <= MAX_PARALLEL_AGENTS:
        # До 5 задач без зависимостей — ПАРАЛЛЕЛЬНО
        implement_tasks_parallel(group)
    else:
        # Больше 5 задач — батчами по 5
        for batch in chunks(group, MAX_PARALLEL_AGENTS):
            implement_tasks_parallel(batch)
```

**Параллельная реализация (до 5 агентов):**
```python
# Запускаем developer для всех задач параллельно (до 5)
dev_tasks = []
for task in group[:5]:  # Максимум 5
    dev_tasks.append(Task(
        subagent_type="developer-agent",
        prompt=f"Реализуй задачу {task['id']}: {task['name']}"
    ))

# Ждём завершения ВСЕХ с обработкой ошибок
for dev_task in dev_tasks:
    try:
        TaskOutput(task_id=dev_task["id"], block=True, timeout=600000)
    except Exception as e:
        log_agent_error(dev_task["id"], "developer-agent", e)

# Запускаем test + review параллельно (до 10 агентов = 5*2)
# НО лимит 5 — значит батчами
for task in group:
    # Последовательно для каждой задачи, но test+review параллельно (2 агента)
    test_task = Task(subagent_type="test-engineer", ...)
    review_task = Task(subagent_type="code-reviewer", ...)

# Ждём завершения ВСЕХ
for tr_task in test_review_tasks:
    TaskOutput(task_id=tr_task["id"], block=True, timeout=600000)

# Verifier — последовательно для каждой задачи
for task in group:
    verify = Task(subagent_type="feature-verifier", ...)
    TaskOutput(task_id=verify["id"], block=True, timeout=600000)

    # Если score >= 9 — оркестратор делает коммит
    # Если score < 9 — доработка
```

---

## ⚠️ ПОЛНЫЙ ЦИКЛ РЕАЛИЗАЦИИ ЗАДАЧИ

```
┌─────────────────────────────────────────────────────────────────┐
│ ЦИКЛ реализации задачи (повторять пока score < 9)             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 1. developer-agent (ПОСЛЕДОВАТЕЛЬНО)                           │
├─────────────────────────────────────────────────────────────────┤
│ Task(subagent_type="developer-agent", ...)                     │
│ ЖДЁМ ЗАВЕРШЕНИЯ (block=true, timeout=600000)                  │
│ Выход: IMPLEMENTATION_REPORT.md + исходный код                 │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2+3. test-engineer + code-reviewer (ПАРАЛЛЕЛЬНО ⚡)            │
├─────────────────────────────────────────────────────────────────┤
│ [ОДНО сообщение с ДВУМЯ Task вызовами]                         │
│ Task(subagent_type="test-engineer", ...)                       │
│ Task(subagent_type="code-reviewer", ...)                       │
│ ЖДЁМ ЗАВЕРШЕНИЯ ОБИХ (block=true)                             │
│ Выход: TEST_REPORT.md + CODE_REVIEW.md                         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 4. feature-verifier (ПОСЛЕДОВАТЕЛЬНО после ОБИХ)              │
├─────────────────────────────────────────────────────────────────┤
│ Task(subagent_type="feature-verifier", ...)                    │
│ ЖДЁМ ЗАВЕРШЕНИЯ (block=true, timeout=600000)                  │
│ Выход: FEATURE_VERIFICATION.md (score)                         │
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
      ОРКЕСТРАТОР делает коммит                  Повтор цикла (доработка)
```

---

## ⚠️ КРИТИЧЕСКИ ВАЖНЫЕ ТРЕБОВАНИЯ

### Полный цикл ОБЯЗАТЕЛЕН для КАЖДОЙ задачи

**КАЖДАЯ задача ДОЛЖНА иметь ВСЕ артефакты:**
- ✅ `IMPLEMENTATION_REPORT.md` — от developer-agent
- ✅ `TEST_REPORT.md` — от test-engineer
- ✅ `CODE_REVIEW.md` — от code-reviewer
- ✅ `FEATURE_VERIFICATION.md` — от feature-verifier (с score ≥ 9)

❌ **ЗАПРЕЩЕНО:**
- Создавать только IMPLEMENTATION_REPORT и пропускать остальные
- "Устать" и делать задачи в ускоренном режиме
- Создавать задачи больше 4-6 часов работы

### Параллельная разработка задач (до 5 одновременно)

✅ **МОЖНО запускать до 5 агентов параллельно:**
- У задач **нет зависимостей** друг от друга
- Каждая задача проходит **ПОЛНЫЙ цикл** разработки
- Test + Review для всех задач запускаются параллельно
- Verifier — последовательно для каждой задачи

❌ **ЗАПРЕЩЕНО:**
- Запускать более 5 агентов одновременно
- Игнорировать зависимости между задачами

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
def run_tdd_planner_with_retry(task, feature_path, max_retries=3):
    """Запуск TDD Planner с обработкой ошибок."""

    for attempt in range(max_retries):
        result = safe_launch_agent(
            subagent_type="tdd-planner",
            prompt=f"""
Создай TDD roadmap для задачи:

Task ID: {task['id']}
Task Name: {task['name']}
Task Description: {task['description']}
Domain: {task['domain']}
Dependencies: {task['dependencies']}
Feature Path: {feature_path}

⚠️ ОБЯЗАТЕЛЬНО:
1. Создай файл docs/roadmaps/{feature_path}ROADMAP_TASKS_{task['id']}.md
2. Убедись что файл не пустой
3. Сделай git commit

После создания — подтверди полный путь к созданному файлу.
""",
            max_retries=1  # Однократный retry внутри safe_launch
        )

        if result is None or result.strip() == "":
            print(f"⚠️ TDD Planner для {task['id']}: пустой результат (попытка {attempt + 1})")
            time.sleep(30 * (2 ** attempt))
            continue

        # Проверяем что roadmap создан
        roadmap_path = f"docs/roadmaps/{feature_path}ROADMAP_TASKS_{task['id']}.md"
        if file_exists(roadmap_path) and file_size(roadmap_path) > 500:
            print(f"✅ Roadmap создан: {roadmap_path}")
            return True
        else:
            print(f"⚠️ Roadmap не найден или пуст: {roadmap_path}")
            time.sleep(30)

    # Все попытки исчерпаны
    print(f"❌ TDD Planner не смог создать roadmap для {task['id']}")
    log_failed_task(task['id'], "tdd-planner", "empty result after retries")
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

    append_to_file(f"docs/project/{FEATURE_PATH}FAILED_TASKS.md", log_entry)
```

### Проверка после этапа TDD планирования

```python
def verify_tdd_planning_complete(tasks, feature_path):
    """Проверить что все roadmap созданы."""

    failed_tasks = []

    for task in tasks:
        roadmap_path = f"docs/roadmaps/{feature_path}ROADMAP_TASKS_{task['id']}.md"

        if not file_exists(roadmap_path):
            failed_tasks.append(task['id'])
        elif file_size(roadmap_path) < 500:
            failed_tasks.append(f"{task['id']} (empty/small)")

    if failed_tasks:
        print(f"⚠️ Roadmap не созданы для задач: {', '.join(failed_tasks)}")
        return False

    return True
```

---

## ФАЙЛОВАЯ СТРУКТУРА АРТЕФАКТОВ

```
docs/
├── project/              # Артефакты уровня фичи
│   └── {FEATURE_PATH}/   # Фичевый путь (может быть пустым)
│       └── TASKS_INDEX.md          # Декомпозиция фичи на задачи
│
├── roadmaps/             # TDD roadmaps для задач фичи
│   └── {FEATURE_PATH}/   # Фичевый путь (может быть пустым)
│       ├── ROADMAP_TASKS_<feature>_<task1>.md
│       ├── ROADMAP_TASKS_<feature>_<task2>.md
│       └── ...
│
└── develop/              # Артефакты разработки задач
    └── {FEATURE_PATH}/   # Фичевый путь (может быть пустым)
        └── <FEATURE>/        # Артефакты фичи
            └── <TASK>/       # Артефакты задачи
                ├── IMPLEMENTATION_REPORT.md
                ├── TEST_REPORT.md
                ├── CODE_REVIEW.md
                └── FEATURE_VERIFICATION.md
```

**Примеры:**
- `FEATURE_PATH = ""` → `docs/project/TASKS_INDEX.md`, `docs/roadmaps/...`, `docs/develop/...`
- `FEATURE_PATH = "features/auth/"` → `docs/project/features/auth/TASKS_INDEX.md`
- `FEATURE_PATH = "modules/user/"` → `docs/roadmaps/modules/user/...`, `docs/develop/modules/user/...`

---

## GIT WORKFLOW (ОБЯЗАТЕЛЬНЫЙ)

**Структура веток:**

```
{MAIN_BRANCH} (например, main, develop)
    ↑
    │ Merge после завершения ВСЕХ задач фичи
    │
feature/<feature-name>
    ↑
    │ Коммиты от оркестратора после каждой задачи (score ≥ 9)
```

### Алгоритм работы

**В начале разработки фичи:**
1. Определить `{MAIN_BRANCH}` — текущая ветка или спросить
2. Создать ветку фичи: `git checkout -b feature/<feature-name>`

**После завершения ВСЕХ задач фичи:**
1. Merge в `{MAIN_BRANCH}`: `git merge feature/<feature-name>`
2. Удалить ветку: `git branch -d feature/<feature-name>`

---

## ⚠️ ОБРАБОТКА ВОПРОСОВ ОТ АГЕНТОВ

Если агент создал `CLARIFICATION_NEEDED.md`:

```python
# Читаем вопросы
questions = parse_clarification_needed(f"docs/project/{FEATURE_PATH}CLARIFICATION_NEEDED.md")

# Задаем пользователю
user_answers = AskUserQuestion(questions=questions["Вопросы"], ...)

# Создаём файл с ответами
create_file(f"docs/project/{FEATURE_PATH}USER_ANSWERS.md", user_answers)

# Удаляем CLARIFICATION_NEEDED.md
remove(f"docs/project/{FEATURE_PATH}CLARIFICATION_NEEDED.md")

# Перезапускаем агента с ответами
Task(subagent_type="...", prompt="ПЕРЕЗАПУСК С ОТВЕТАМИ...")

# Удаляем USER_ANSWERS.md
remove(f"docs/project/{FEATURE_PATH}USER_ANSWERS.md")
```

---

## Детальные шаги цикла реализации задачи

### Шаг 1: Developer Agent (последовательно)

**Промпт:**
```
Реализуй задачу: {TASK_DESCRIPTION}

Контекст проекта:
- Основная технология: {tech_stack}
- Архитектура: {architecture_summary}
- Coding conventions: {conventions}

Roadmap: docs/roadmaps/{FEATURE_PATH}ROADMAP_TASKS_{feature}_{task_id}.md

Выполни:
1. Изучи существующий код в проекте
2. Следуй паттернам и конвенциям проекта
3. Реализуй задачу согласно roadmap
4. Создай IMPLEMENTATION_REPORT.md
```

**Выход:** `docs/develop/{FEATURE_PATH}<FEATURE>/<TASK>/IMPLEMENTATION_REPORT.md` + исходный код

---

### Шаг 2+3: Test Engineer + Code Reviewer (параллельно)

**Промпт для test-engineer:**
```
Протестируй задачу: {TASK_DESCRIPTION}

⚠️ ОБЯЗАТЕЛЬНО:
1. Build Verification — выполни команду сборки
2. Run Verification — выполни команду запуска
3. Напиши тесты согласно roadmap
4. Проверь покрытие
5. Создай TEST_REPORT.md

Команды: {build_run_commands}
```

**Промпт для code-reviewer:**
```
Выполни code review для задачи: {TASK_DESCRIPTION}

Создай CODE_REVIEW.md с замечаниями.
```

**Выход:** `TEST_REPORT.md` + `CODE_REVIEW.md`

---

### Шаг 4: Feature Verifier Agent (последовательно)

**Промпт:**
```
Выполни финальную верификацию задачи: {TASK_DESCRIPTION}

⚠️ КРИТИЧЕСКО: Автоматически отклоняй (score < 9) если:
- Build verification = FAIL
- Run verification = FAIL

Создай FEATURE_VERIFICATION.md с итоговым score.
```

**Выход:** `docs/develop/{FEATURE_PATH}<FEATURE>/<TASK>/FEATURE_VERIFICATION.md`

---

### Проверка score и решение

```python
score = extract_score(verification)

if score >= 9:
    # ─────────────────────────────────────────────────────────────────
    # ШАГ 1: Отметить задачу в roadmap как выполненную
    # ─────────────────────────────────────────────────────────────────
    roadmap_path = find_roadmap_for_task(TASK_ID)

    if roadmap_path:
        roadmap = read_file(roadmap_path)
        updated_roadmap = mark_task_completed(roadmap, TASK_ID, score)
        write_file(roadmap_path, updated_roadmap)

        bash_command(f"""
            git add {roadmap_path}
            git commit -m "docs: mark task {TASK_ID} as completed (score {score}/10)"
        """)

    # ─────────────────────────────────────────────────────────────────
    # ШАГ 2: Коммит артефактов задачи
    # ─────────────────────────────────────────────────────────────────
    bash_command(f"""
        git add docs/develop/{FEATURE_PATH}{FEATURE}/{TASK_ID}/
        git commit -m "feat: {TASK_NAME}

        - Implementation: developer-agent
        - Test: test-engineer
        - Review: code-reviewer
        - Verification: feature-verifier (score ≥ 9)
        "
    """)
else:
    # score < 9 — доработка
    # ...
```

---

## 🚫 ЗАПРЕЩЕНО

❌ **НЕПРАВИЛЬНО #1** (более 5 агентов параллельно):
```python
# 10 агентов одновременно — ПРЕВЫШЕНИЕ ЛИМИТА
dev_1 = Task(developer-agent, ...)
dev_2 = Task(developer-agent, ...)
...
dev_6 = Task(developer-agent, ...)  # ❌ Лимит 5!
```

❌ **НЕПРАВИЛЬНО #2** (все агенты параллельно без порядка):
```python
Task(developer-agent, ...)
Task(test-engineer, ...)      # ❌ Test до завершения developer
Task(code-reviewer, ...)     # ❌ Review до завершения developer
Task(feature-verifier, ...)  # ❌ Verifier до завершения test+review
```

❌ **НЕПРАВИЛЬНО #3** (только developer):
```python
Task(developer-agent, ...)
# Создал только IMPLEMENTATION_REPORT и "устал"
# ❌ Пропущены test, review, verifier
```

❌ **НЕПРАВИЛЬНО #4** (игнорирование пустых результатов):
```python
result = Task(subagent_type="tdd-planner", ...)
# result = None или пустая строка
# ❌ Не проверили результат, продолжаем как будто всё ок
```

❌ **НЕПРАВИЛЬНО #5** (нет retry при ошибках):
```python
try:
    TaskOutput(task_id=task["id"], block=True)
except:
    pass  # ❌ Просто игнорируем ошибку, нет retry или логирования
```

---

## ✅ ПРАВИЛЬНЫЕ ПАТТЕРНЫ

### Правильный запуск (одна задача)

```python
# Developer (последовательно) с обработкой ошибок
result_dev = safe_launch_agent(
    subagent_type="developer-agent",
    prompt="..."
)
if result_dev is None:
    log_failed_task(TASK_ID, "developer-agent", "empty result")
    # Retry или skip

# Test + Review (ПАРАЛЛЕЛЬНО, 2 агента < 5)
task_test = Task(subagent_type="test-engineer", prompt="...")
task_review = Task(subagent_type="code-reviewer", prompt="...")

result_test = TaskOutput(task_id=task_test["id"], block=True, timeout=600000)
result_review = TaskOutput(task_id=task_review["id"], block=True, timeout=600000)

# Проверка результатов
if result_test is None or result_test.strip() == "":
    log_failed_task(TASK_ID, "test-engineer", "empty result")

# Verifier (последовательно)
result_verify = safe_launch_agent(
    subagent_type="feature-verifier",
    prompt="..."
)
```

### Правильный запуск (до 5 задач параллельно)

```python
MAX_PARALLEL = 5
tasks = [T_001, T_002, T_003, T_004, T_005]  # До 5 задач

# Developer для всех (параллельно, до 5 агентов)
dev_tasks = []
for task in tasks[:MAX_PARALLEL]:
    dev_tasks.append({
        "task_id": task["id"],
        "agent": Task(subagent_type="developer-agent", prompt=f"...{task['id']}...")
    })

# Ждём завершения с обработкой ошибок
failed_dev_tasks = []
for dev_task in dev_tasks:
    try:
        result = TaskOutput(task_id=dev_task["agent"]["id"], block=True, timeout=600000)
        if result is None or result.strip() == "":
            failed_dev_tasks.append(dev_task["task_id"])
    except Exception as e:
        log_failed_task(dev_task["task_id"], "developer-agent", str(e))
        failed_dev_tasks.append(dev_task["task_id"])

# Test + Review для успешных задач (по 2 агента на задачу)
# Но общий лимит 5 — значит обрабатываем по 2 задачи за раз
successful_tasks = [t for t in tasks if t["id"] not in failed_dev_tasks]

for batch in chunks(successful_tasks, 2):  # 2 задачи * 2 агента = 4 < 5
    test_review_agents = []
    for task in batch:
        test_review_agents.append(Task(subagent_type="test-engineer", prompt=f"...{task['id']}..."))
        test_review_agents.append(Task(subagent_type="code-reviewer", prompt=f"...{task['id']}..."))

    for agent in test_review_agents:
        TaskOutput(task_id=agent["id"], block=True, timeout=600000)

# Verifier (последовательно для каждой задачи)
for task in successful_tasks:
    verify = Task(subagent_type="feature-verifier", prompt=f"...{task['id']}...")
    result = TaskOutput(task_id=verify["id"], block=True, timeout=600000)

    if result:
        score = extract_score(result)
        if score >= 9:
            commit_task(task)
```

### Правильный запуск TDD Planner (батчами по 5)

```python
MAX_PARALLEL = 5
tasks = parse_tasks("docs/project/TASKS_INDEX.md")

# Обрабатываем батчами
for batch_start in range(0, len(tasks), MAX_PARALLEL):
    batch = tasks[batch_start:batch_start + MAX_PARALLEL]

    # Запускаем до 5 TDD Planner параллельно
    planner_tasks = []
    for task in batch:
        planner_tasks.append({
            "task_id": task["id"],
            "agent": Task(subagent_type="tdd-planner", prompt=f"""
Создай TDD roadmap для задачи: {task['id']}
...
""")
        })

    # Ждём завершения всех в батче
    for pt in planner_tasks:
        try:
            result = TaskOutput(task_id=pt["agent"]["id"], block=True, timeout=600000)

            # Проверяем что roadmap создан
            roadmap_path = f"docs/roadmaps/ROADMAP_TASKS_{pt['task_id']}.md"
            if not file_exists(roadmap_path):
                log_failed_task(pt["task_id"], "tdd-planner", "roadmap not created")
                # Retry для этой задачи
                retry_tdd_planner(pt["task_id"])
        except Exception as e:
            log_failed_task(pt["task_id"], "tdd-planner", str(e))
```

---

## ⚠️ ОТМЕТКА ВЫПОЛНЕННЫХ ЗАДАЧ В ROADMAP

### Формат отметки задачи

```markdown
### Task T-001: <Task Name>

**Status:** ✅ COMPLETED
**Completed:** YYYY-MM-DD
**Final Score:** 9/10
```

### Функции отметки

```python
def mark_task_completed(roadmap_content, task_id, score):
    from datetime import datetime

    completed_date = datetime.now().strftime("%Y-%m-%d")

    task_section = find_task_section(roadmap_content, task_id)

    if "**Status:**" in task_section:
        updated = task_section.replace(
            "**Status:** ⏳ IN PROGRESS",
            f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Final Score:** {score}/10"
        )
    else:
        # Добавляем статус
        updated = f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Final Score:** {score}/10\n\n" + task_section

    return roadmap_content.replace(task_section, updated)
```

---

## Завершение фичи

После успешного завершения ВСЕХ задач:

```bash
# Merge в основную ветку
git checkout {MAIN_BRANCH}
git merge feature/<feature-name>
git branch -d feature/<feature-name>
```

---

## Использование

Когда пользователь запускает `/feature-developer`:

1. **Получите описание фичи** от пользователя
2. **Запросите фичевый путь** `{FEATURE_PATH}` через AskUserQuestion
3. **Этап 1:** Декомпозиция на задачи (feature-decomposer) → `docs/project/{FEATURE_PATH}TASKS_INDEX.md`
4. **Этап 2:** TDD планирование (tdd-planner для каждой задачи) → `docs/roadmaps/{FEATURE_PATH}ROADMAP_TASKS_*.md`
5. **Этап 3:** Создание ветки `feature/<name>`
6. **Этап 4:** Реализация задач (последовательно или параллельно) → `docs/develop/{FEATURE_PATH}<FEATURE>/`
7. **Merge** в основную ветку

---
