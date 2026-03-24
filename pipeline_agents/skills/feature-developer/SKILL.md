---
name: feature-developer
description: Запускает разработку фичи в существующем проекте через декомпозицию на задачи и TDD roadmaps
---

# FEATURE_DEVELOPER

## Назначение
Оркестратор разработки одной фичи в существующем проекте:
- Декомпозиция фичи на задачи (2-4 часа каждая)
- TDD планирование для каждой задачи
- Реализация задач через мультиагентный пайплайн
- Координация и коммиты

**Глобальный лимит:** Максимум 3 агентов одновременно.

---

## ЭТАПЫ РАЗРАБОТКИ

### Этап 0 — Приём намерения + CONTEXT.md

1. Получить описание фичи от пользователя
2. Запросить фичевый путь через AskUserQuestion:
   - Пустой → `docs/project/`, `docs/roadmaps/`, `docs/develop/`
   - `features/auth/`, `modules/user/` и т.д.
3. Определить границы фичи и текущую ветку
4. **Создать CONTEXT.md** (кэширование контекста):

```python
project_context = Task(
    subagent_type="Explore",
    prompt="""
Собери краткий контекст проекта:
1. Tech stack
2. Architecture summary
3. Coding conventions
4. Build/run/test команды

Формат: КРАТКО, только факты.
"""
)

create_file(f"docs/develop/{FEATURE_PATH}CONTEXT.md", f"""
# Project Context

## Tech Stack
{project_context.tech_stack}

## Architecture
{project_context.architecture}

## Conventions
{project_context.conventions}

## Commands
- Build: {project_context.build_command}
- Run: {project_context.run_command}
- Test: {project_context.test_command}

## Feature
{FEATURE_DESCRIPTION}
""")
```

**Выход:**
- `{FEATURE_PATH}` — фичевый путь (с trailing slash или пустой)
- `docs/develop/{FEATURE_PATH}CONTEXT.md`

---

### Этап 1 — Декомпозиция на задачи

```python
Task(
    subagent_type="feature-decomposer",
    prompt=f"""
Выполни декомпозицию ОДНОЙ фичи на задачи.

Фича: {FEATURE_DESCRIPTION}

Требования:
1. Разбей фичу на 3-10 задач
2. Каждая задача = 2-4 часа работы
3. Задачи независимые (минимум зависимостей)
4. Создай TASKS_INDEX.md в docs/project/{FEATURE_PATH}

Формат TASKS_INDEX.md:
# Tasks Index: {{feature_name}}

## Feature Info
- Name: ...
- Description: ...
- Path: {FEATURE_PATH}

## Tasks

### Task T-001: <Name>
- Description: ...
- Estimated Time: 2-4 hours
- Dependencies: None
- Domain: ...

## Dependency Graph
[Граф]
"""
)
```

**Обработка CLARIFICATION_NEEDED.md:**
```python
if exists(f"docs/project/{FEATURE_PATH}CLARIFICATION_NEEDED.md"):
    questions = parse_clarification_needed(...)
    answers = AskUserQuestion(questions=questions)
    create_file(f"docs/project/{FEATURE_PATH}USER_ANSWERS.md", answers)
    remove(f"docs/project/{FEATURE_PATH}CLARIFICATION_NEEDED.md")
    Task(subagent_type="feature-decomposer", prompt="ПЕРЕЗАПУСК С ОТВЕТАМИ...")
    remove(f"docs/project/{FEATURE_PATH}USER_ANSWERS.md")
```

**Выход:** `docs/project/{FEATURE_PATH}TASKS_INDEX.md`

---

### Этап 2 — TDD планирование (компактный формат)

**Лимит:** Максимум 3 агентов одновременно.

```python
tasks = parse_tasks(f"docs/project/{FEATURE_PATH}TASKS_INDEX.md")
BATCH_SIZE = 3

for batch_start in range(0, len(tasks), BATCH_SIZE):
    batch = tasks[batch_start:batch_start + BATCH_SIZE]

    for task in batch:
        safe_launch_agent(
            subagent_type="tdd-planner",
            prompt=f"""
Создай КОМПАКТНЫЙ TDD roadmap для задачи:

Task ID: {task['id']}
Task Name: {task['name']}

ФОРМАТ:
## Task {task['id']}: {task['name']}
**Domain:** {task['domain']} | **Dependencies:** {task['dependencies']}

### Checklist
- [ ] CODE: <файлы>
- [ ] TEST: <тесты>
- [ ] BUILD: <команда>

### Acceptance
- <критерии>

Создай docs/roadmaps/{FEATURE_PATH}ROADMAP_TASKS_{task['id']}.md
"""
        )

# Коммит всех roadmaps после создания ВСЕХ
bash(f"git add docs/roadmaps/{FEATURE_PATH} && git commit -m 'docs: add TDD roadmaps for all tasks'")
```

**Выход:** `docs/roadmaps/{FEATURE_PATH}ROADMAP_TASKS_*.md`

---

### Этап 3 — Создание ветки

```bash
git checkout -b feature/<feature-name>  # выполняется пользователем
```

---

### Этап 4 — Реализация задач

**Правила:**
- Задачи без зависимостей → параллельно (до 3)
- Задачи с зависимостями → последовательно

```python
MAX_PARALLEL = 3
task_groups = group_by_dependencies(tasks)

for group in task_groups:
    if len(group) <= MAX_PARALLEL:
        implement_tasks_parallel(group)
    else:
        for batch in chunks(group, MAX_PARALLEL):
            implement_tasks_parallel(batch)
```

---

## ЦИКЛ РЕАЛИЗАЦИИ ЗАДАЧИ

```
1. developer-agent (последовательно)
   ↓
2. test-reviewer (ОДИН агент вместо двух)
   ↓
3. verifier (ВСЕГДА, учитывает HAS_ISSUES из test-reviewer)
   ↓
4. Проверка score
   ┌────────┴────────┐
   │                 │
score ≥ 9        score < 9
   │                 │
   ↓                 ↓
Коммит задачи   Доработка
```

---

## ШАГИ РЕАЛИЗАЦИИ

### Шаг 1: Developer Agent

```python
result_dev = safe_launch_agent(
    subagent_type="developer-agent",
    prompt=f"""
См. CONTEXT.md в docs/develop/{FEATURE_PATH}/

Задача: {TASK_ID} — {TASK_NAME}
Roadmap: docs/roadmaps/{FEATURE_PATH}ROADMAP_TASKS_{TASK_ID}.md

Выполни:
1. Изучи CONTEXT.md
2. Изучи roadmap
3. Реализуй задачу
4. Создай IMPLEMENTATION_REPORT_{TASK_ID}.md (кратко)
"""
)
```

**Выход:** `docs/develop/{FEATURE_PATH}/IMPLEMENTATION_REPORT_{TASK_ID}.md`

---

### Шаг 2: Test-Reviewer

```python
result_test_review = safe_launch_agent(
    subagent_type="test-reviewer",
    prompt=f"""
См. CONTEXT.md

Задача: {TASK_ID} — {TASK_NAME}
Roadmap: docs/roadmaps/{FEATURE_PATH}ROADMAP_TASKS_{TASK_ID}.md
Code: docs/develop/{FEATURE_PATH}/

Выполни:
1. BUILD: команда из CONTEXT.md
2. RUN: проверь запуск
3. TESTS: по roadmap
4. REVIEW: проверка кода

Создай TEST_AND_REVIEW.md

Формат:
## Test & Review: {{TASK_ID}}

### Build/Run
- Build: ✅/❌
- Run: ✅/❌

### Tests
- Written: <список>
- Coverage: <%>

### Review Issues
- <замечания или "нет">

### Verdict
- HAS_ISSUES: true/false
"""
)
```

**Выход:** `docs/develop/{FEATURE_PATH}/TEST_AND_REVIEW_{TASK_ID}.md`

---

### Шаг 3: Верификация (всегда)

```python
test_review = read_file(f"docs/develop/{FEATURE_PATH}/TEST_AND_REVIEW_{TASK_ID}.md")
has_issues = "HAS_ISSUES: true" in test_review
is_critical = is_critical_task(TASK_ID)  # security, payments, auth

# Verifier запускается ВСЕГДА, но учитывает HAS_ISSUES из test-reviewer
result_verify = safe_launch_agent(
    subagent_type="feature-verifier",
    prompt=f"""
См. CONTEXT.md в docs/develop/{FEATURE_PATH}/

Задача: {TASK_ID}
Roadmap: docs/roadmaps/{FEATURE_PATH}ROADMAP_TASKS_{TASK_ID}.md
TEST_AND_REVIEW: docs/develop/{FEATURE_PATH}/TEST_AND_REVIEW_{TASK_ID}.md
HAS_ISSUES из test-reviewer: {has_issues}
Критичная задача: {is_critical}

Выполни ПОЛНУЮ верификацию:
1. Проверь все артефакты (IMPLEMENTATION_REPORT, TEST_AND_REVIEW)
2. Учти уже выявленные проблемы из TEST_AND_REVIEW
3. Проверь build/run/tests
4. Оцени качество кода

Создай FEATURE_VERIFICATION_{TASK_ID}.md с итоговым score (0-10).
"""
)
score = extract_score(result_verify)
verification_status = "MANUAL_VERIFIED"
```

---

### Шаг 4: Завершение задачи + Коммит

```python
if score >= 9:
    # Отметка в roadmap
    roadmap_path = find_roadmap_for_task(TASK_ID)
    if roadmap_path:
        roadmap = read_file(roadmap_path)
        updated = mark_task_completed(roadmap, TASK_ID, score, verification_status)
        write_file(roadmap_path, updated)

    # Коммит артефактов задачи
    bash(f"""
    git add docs/develop/{FEATURE_PATH}/IMPLEMENTATION_REPORT_{TASK_ID}.md
    git add docs/develop/{FEATURE_PATH}/TEST_AND_REVIEW_{TASK_ID}.md
    git add docs/develop/{FEATURE_PATH}/FEATURE_VERIFICATION_{TASK_ID}.md
    git commit -m "feat: {TASK_NAME}

    Task: {TASK_ID}
    Implementation: developer-agent
    Test & Review: test-reviewer
    Verification: {verification_status}
    Score: {score}/10
    """
    )
else:
    # Доработка (повтор цикла)
    ...
```

**Коммит выполняется для КАЖДОЙ задачи при score >= 9.**

---

## ОБРАБОТКА ОШИБОК

### safe_launch_agent

```python
def safe_launch_agent(subagent_type, prompt, max_retries=3, base_delay=30):
    """
    Запуск агента с умными retry.
    - 1-я попытка: полный промпт
    - 2-я попытка: сжатый (только ошибка)
    - 3-я попытка: минимальный
    """
    import time
    current_prompt = prompt

    for attempt in range(max_retries):
        try:
            task = Task(subagent_type=subagent_type, prompt=current_prompt)
            result = TaskOutput(task_id=task["id"], block=True, timeout=600000)

            if result is None or result.strip() == "" or len(result) < 100:
                if attempt == 0:
                    current_prompt = f"ERROR: Empty result. Fix: {extract_instruction(prompt)}"
                elif attempt == 1:
                    current_prompt = f"TASK: {extract_task_id(prompt)}. ACTION: create file."
                delay = base_delay * (2 ** attempt)
                time.sleep(delay)
                continue

            if "ROADMAP" in prompt or "REPORT" in prompt:
                expected_files = extract_expected_files(prompt)
                for filepath in expected_files:
                    if not file_exists(filepath):
                        if attempt == 0:
                            current_prompt = f"ERROR: File not created: {filepath}. Create now."
                        elif attempt == 1:
                            current_prompt = f"CREATE: {filepath}"
                        delay = base_delay * (2 ** attempt)
                        time.sleep(delay)
                        continue

            return result

        except Exception as e:
            error_msg = str(e).lower()
            if "rate limit" in error_msg or "429" in error_msg:
                time.sleep(60)
                continue
            if "timeout" in error_msg:
                if attempt == 0:
                    current_prompt = f"RETRY after timeout. Task: {extract_task_id(prompt)}"
                time.sleep(30)
                continue

            if attempt == 0:
                current_prompt = f"ERROR: {str(e)[:100]}. Fix: {extract_instruction(prompt)}"
            elif attempt == 1:
                current_prompt = f"SIMPLE TASK: {extract_task_id(prompt)}"
            delay = base_delay * (2 ** attempt)
            time.sleep(delay)

    log_failed_task(TASK_ID, subagent_type, "failed after retries")
    return None
```

### Логирование неудач

```python
def log_failed_task(task_id, agent_type, error_reason):
    append_to_file(f"docs/project/{FEATURE_PATH}FAILED_TASKS.md", f"""
## Failed Task: {task_id}
- Agent: {agent_type}
- Error: {error_reason}
- Timestamp: {datetime.now()}
- Action: MANUAL REVIEW REQUIRED
""")
```

---

### Этап 5 — Финализация и документация

После завершения ВСЕХ задач:

```python
# Проверка завершения всех задач
tasks_index = read_file(f"docs/project/{FEATURE_PATH}TASKS_INDEX.md")
all_completed = check_all_tasks_completed(tasks_index)

if all_completed:
    # Запуск documentation-agent для создания документации фичи
    result_docs = safe_launch_agent(
        subagent_type="documentation-agent",
        prompt=f"""
        Создай документацию для завершённой фичи.

        Фича: {FEATURE_NAME}
        Path: {FEATURE_PATH}

        Источники:
        - CONTEXT.md: docs/develop/{FEATURE_PATH}/CONTEXT.md
        - TASKS_INDEX.md: docs/project/{FEATURE_PATH}/TASKS_INDEX.md
        - Roadmaps: docs/roadmaps/{FEATURE_PATH}/ROADMAP_TASKS_*.md
        - Implementation Reports: docs/develop/{FEATURE_PATH}/IMPLEMENTATION_REPORT_*.md

        Выполни:
        1. Собери информацию из всех артефактов
        2. Создай FEATURE_DOCS.md в docs/develop/{FEATURE_PATH}/

        Формат FEATURE_DOCS.md:
        # Документация: {FEATURE_NAME}

        ## Обзор
        - Описание фичи
        - Цели и задачи

        ## Архитектура
        - Структура модулей
        - Ключевые компоненты
        - Зависимости

        ## API / Интерфейсы
        - Публичные методы
        - Параметры и возвращаемые значения

        ## Использование
        - Примеры использования
        - Конфигурация (если есть)

        ## Тестирование
        - Покрытие тестами
        - Как запускать тесты

        ## Реализованные задачи
        - Список задач со статусами

        ## Известные ограничения
        - Если есть
        """
    )

    # Финальный коммит документации
    bash(f"""
    git add docs/develop/{FEATURE_PATH}/FEATURE_DOCS.md
    git commit -m "docs: add feature documentation for {FEATURE_NAME}

    Feature: {FEATURE_NAME}
    Path: {FEATURE_PATH}
    Tasks completed: {tasks_count}"
    """)
```

**Выход:** `docs/develop/{FEATURE_PATH}/FEATURE_DOCS.md`

---

## GIT WORKFLOW

```bash
# Начало фичи (Этап 3)
git checkout -b feature/<feature-name>  # выполняется пользователем

# Коммит после Этапа 2 — один коммит для всех roadmaps
git add docs/roadmaps/{FEATURE_PATH}
git commit -m "docs: add TDD roadmaps for all tasks

Feature: {feature_name}
Tasks: {task_count}
Path: {FEATURE_PATH}"

# Коммит после каждой задачи при score >= 9 (выполняется в Шаге 4)
# Формат: "feat: {TASK_NAME}"
# Body:
# Task: {TASK_ID}
# Implementation: developer-agent
# Test & Review: test-reviewer
# Verification: {verification_status}
# Score: {score}/10

# Merge фичи в основную ветку
# Merge фичи в основную ветку
# Merge и checkout выполняются пользователем отдельно


---

## ФАЙЛОВАЯ СТРУКТУРА

```
docs/
├── develop/{FEATURE_PATH}/
│   ├── CONTEXT.md
│   ├── IMPLEMENTATION_REPORT_{TASK_ID}.md
│   ├── TEST_AND_REVIEW_{TASK_ID}.md
│   ├── FEATURE_VERIFICATION_{TASK_ID}.md
│   └── FEATURE_DOCS.md
├── project/{FEATURE_PATH}/
│   └── TASKS_INDEX.md
└── roadmaps/{FEATURE_PATH}/
    └── ROADMAP_TASKS_{TASK_ID}.md
```

---

## ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ

### mark_task_completed

```python
def mark_task_completed(roadmap_content, task_id, score, verification_status="MANUAL_VERIFIED"):
    from datetime import datetime
    completed_date = datetime.now().strftime("%Y-%m-%d")
    task_section = find_task_section(roadmap_content, task_id)

    if "**Status:**" in task_section:
        updated = task_section.replace(
            "**Status:** ⏳ IN PROGRESS",
            f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Score:** {score}/10\n**Verification:** {verification_status}"
        )
    else:
        updated = f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Score:** {score}/10\n**Verification:** {verification_status}\n\n" + task_section

    return roadmap_content.replace(task_section, updated)
```

### is_critical_task

```python
def is_critical_task(task_id):
    """Проверка на критичность (security, payments, auth)."""
    critical_domains = ["security", "payment", "auth", "authorization", "encryption"]
    task = get_task_by_id(task_id)
    return any(domain in task.get("domain", "").lower() for domain in critical_domains)
```

---

## ЗАПРЕЩЕНО

❌ Запускать более 3 агентов одновременно
❌ Передавать полный контекст в промпте (использовать CONTEXT.md)
❌ Запускать test-engineer + code-reviewer раздельно (только test-reviewer)
❌ Пропускать verifier — он запускается ВСЕГДА для каждой задачи
❌ Повторять полный промпт при retry (сжимать: full → error → minimal)
❌ Пропускать артефакты (IMPLEMENTATION_REPORT, TEST_AND_REVIEW, FEATURE_VERIFICATION)
❌ Пропускать documentation-agent — он запускается ВСЕГДА после завершения всех задач

---

## ИСПОЛЬЗОВАНИЕ

```
/feature-developer →
1. Описание фичи + FEATURE_PATH
2. CONTEXT.md → docs/develop/{FEATURE_PATH}/
3. TASKS_INDEX.md → docs/project/{FEATURE_PATH}/
4. ROADMAP_TASKS_*.md → docs/roadmaps/{FEATURE_PATH}/ (коммит всех roadmaps)
5. git checkout -b feature/<name>
6. Реализация: dev → test-reviewer → verifier → коммит задачи (для каждой задачи)
7. documentation-agent → FEATURE_DOCS.md (после завершения всех задач)
8. Merge и checkout выполняются пользователем отдельно
```
