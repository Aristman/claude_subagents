---
name: feature-developer
description: Запускает разработку фичи в существующем проекте через декомпозицию на задачи и TDD roadmaps
---

# FEATURE_DEVELOPER — Разработка фичи в существующем проекте

## Версия
- version: 4.0.0
- standalone: true
- purpose: feature_development

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
2. Понять границы фичи (что входит, что нет)
3. Определить текущую ветку разработки

**Выход:** Описание фичи в формате естественного языка

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
4. Создай TASKS_INDEX.md в docs/project/

Формат TASKS_INDEX.md:
```markdown
# Tasks Index for Feature: {feature_name}

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
Если agent создал `docs/project/CLARIFICATION_NEEDED.md`:
```python
if exists("docs/project/CLARIFICATION_NEEDED.md"):
    questions = parse_clarification_needed("docs/project/CLARIFICATION_NEEDED.md")
    user_answers = AskUserQuestion(questions=questions["Вопросы"], ...)
    create_file("docs/project/USER_ANSWERS.md", user_answers)
    remove("docs/project/CLARIFICATION_NEEDED.md")

    # Перезапускаем агента с ответами
    Task(subagent_type="feature-decomposer", prompt="ПЕРЕЗАПУСК С ОТВЕТАМИ...")
    remove("docs/project/USER_ANSWERS.md")
```

**Выход:** `docs/project/TASKS_INDEX.md`

---

### Этап 2 — TDD планирование для ВСЕХ задач

**Задача:** Создать roadmap для каждой задачи

**Выполни последовательно для КАЖДОЙ задачи из TASKS_INDEX.md:**

```python
# Читаем TASKS_INDEX.md
tasks_index = read_file("docs/project/TASKS_INDEX.md")
tasks = parse_tasks(tasks_index)

# Для каждой задачи создаём roadmap
for task in tasks:
    Task(
        subagent_type="tdd-planner",
        prompt=f"""
Создай TDD roadmap для задачи:

Task ID: {task['id']}
Task Name: {task['name']}
Task Description: {task['description']}
Domain: {task['domain']}
Dependencies: {task['dependencies']}

Создай ROADMAP_TASKS_{feature}_{task_id}.md в docs/roadmaps/

После создания — git commit.
"""
    )
    # Ждём завершения перед следующей задачей
```

**Выход:** `docs/roadmaps/ROADMAP_TASKS_<feature>_<task>.md` для каждой задачи

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
- **Параллельно** (до 3 задач) — если нет зависимостей

**Алгоритм:**
```python
# Группируем задачи по зависимостям
task_groups = group_by_dependencies(tasks)

# Для каждой группы
for group in task_groups:
    if len(group) == 1:
        # Одна задача — последовательно
        implement_task(group[0])
    elif len(group) <= 3:
        # До 3 задач без зависимостей — ПАРАЛЛЕЛЬНО
        implement_tasks_parallel(group)
    else:
        # Больше 3 задач — последовательно
        for task in group:
            implement_task(task)
```

**Параллельная реализация (до 3 задач):**
```python
# Запускаем developer для всех задач параллельно
dev_tasks = []
for task in group:
    dev_tasks.append(Task(
        subagent_type="developer-agent",
        prompt=f"Реализуй задачу {task['id']}: {task['name']}"
    ))

# Ждём завершения ВСЕХ
for dev_task in dev_tasks:
    TaskOutput(task_id=dev_task["id"], block=True, timeout=600000)

# Запускаем test + review параллельно
test_review_tasks = []
for task in group:
    test_review_tasks.append(Task(subagent_type="test-engineer", ...))
    test_review_tasks.append(Task(subagent_type="code-reviewer", ...))

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

### Параллельная разработка задач (до 3 одновременно)

✅ **МОЖНО разрабатывать до 3 задач параллельно:**
- У задач **нет зависимостей** друг от друга
- Каждая задача проходит **ПОЛНЫЙ цикл** разработки
- Test + Review для всех задач запускаются параллельно
- Verifier — последовательно для каждой задачи

❌ **ЗАПРЕЩЕНО:**
- Разрабатывать более 3 задач параллельно
- Игнорировать зависимости между задачами

---

## ФАЙЛОВАЯ СТРУКТУРА АРТЕФАКТОВ

```
docs/
├── project/              # Артефакты уровня фичи
│   └── TASKS_INDEX.md          # Декомпозиция фичи на задачи
│
├── roadmaps/             # TDD roadmaps для задач фичи
│   ├── ROADMAP_TASKS_<feature>_<task1>.md
│   ├── ROADMAP_TASKS_<feature>_<task2>.md
│   └── ...
│
└── develop/              # Артефакты разработки задач
    └── <FEATURE>/        # Артефакты фичи
        └── <TASK>/       # Артефакты задачи
            ├── IMPLEMENTATION_REPORT.md
            ├── TEST_REPORT.md
            ├── CODE_REVIEW.md
            └── FEATURE_VERIFICATION.md
```

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
questions = parse_clarification_needed("docs/project/CLARIFICATION_NEEDED.md")

# Задаем пользователю
user_answers = AskUserQuestion(questions=questions["Вопросы"], ...)

# Создаём файл с ответами
create_file("docs/project/USER_ANSWERS.md", user_answers)

# Удаляем CLARIFICATION_NEEDED.md
remove("docs/project/CLARIFICATION_NEEDED.md")

# Перезапускаем агента с ответами
Task(subagent_type="...", prompt="ПЕРЕЗАПУСК С ОТВЕТАМИ...")

# Удаляем USER_ANSWERS.md
remove("docs/project/USER_ANSWERS.md")
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

Roadmap: docs/roadmaps/ROADMAP_TASKS_{feature}_{task_id}.md

Выполни:
1. Изучи существующий код в проекте
2. Следуй паттернам и конвенциям проекта
3. Реализуй задачу согласно roadmap
4. Создай IMPLEMENTATION_REPORT.md
```

**Выход:** `docs/develop/<FEATURE>/<TASK>/IMPLEMENTATION_REPORT.md` + исходный код

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

**Выход:** `docs/develop/<FEATURE>/<TASK>/FEATURE_VERIFICATION.md`

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
        git add docs/develop/{FEATURE}/{TASK_ID}/
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

❌ **НЕПРАВИЛЬНО #1** (все агенты параллельно):
```python
Task(developer-agent, ...)
Task(test-engineer, ...)
Task(code-reviewer, ...)
Task(feature-verifier, ...)
```

❌ **НЕПРАВИЛЬНО #2** (только developer):
```python
Task(developer-agent, ...)
# Создал только IMPLEMENTATION_REPORT и "устал"
```

---

## ✅ ПРАВИЛЬНЫЕ ПАТТЕРНЫ

### Правильный запуск (одна задача)

```python
# Developer (последовательно)
task_dev = Task(subagent_type="developer-agent", prompt="...")
result_dev = TaskOutput(task_id=task_dev["id"], block=True, timeout=600000)

# Test + Review (ПАРАЛЛЕЛЬНО)
task_test = Task(subagent_type="test-engineer", prompt="...")
task_review = Task(subagent_type="code-reviewer", prompt="...")

result_test = TaskOutput(task_id=task_test["id"], block=True, timeout=600000)
result_review = TaskOutput(task_id=task_review["id"], block=True, timeout=600000)

# Verifier (последовательно)
task_verify = Task(subagent_type="feature-verifier", prompt="...")
result_verify = TaskOutput(task_id=task_verify["id"], block=True, timeout=600000)
```

### Правильный запуск (3 задачи параллельно)

```python
# Developer для всех (параллельно)
dev_1 = Task(developer-agent, "...T-001...")
dev_2 = Task(developer-agent, "...T-002...")
dev_3 = Task(developer-agent, "...T-003...")

TaskOutput(task_id=dev_1["id"], block=True)
TaskOutput(task_id=dev_2["id"], block=True)
TaskOutput(task_id=dev_3["id"], block=True)

# Test + Review для всех (параллельно)
test_1 = Task(test-engineer, "...T-001...")
review_1 = Task(code-reviewer, "...T-001...")
test_2 = Task(test-engineer, "...T-002...")
review_2 = Task(code-reviewer, "...T-002...")
# ...

TaskOutput(task_id=test_1["id"], block=True)
# ... все 6 TaskOutput

# Verifier (последовательно)
verify_1 = Task(feature-verifier, "...T-001...")
TaskOutput(task_id=verify_1["id"], block=True)
verify_2 = Task(feature-verifier, "...T-002...")
# ...
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
2. **Этап 1:** Декомпозиция на задачи (feature-decomposer)
3. **Этап 2:** TDD планирование (tdd-planner для каждой задачи)
4. **Этап 3:** Создание ветки `feature/<name>`
5. **Этап 4:** Реализация задач (последовательно или параллельно)
6. **Merge** в основную ветку

---
