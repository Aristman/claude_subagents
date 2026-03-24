---
name: product-creator
description: Запускает полный мультиагентный пайплайн создания нового продукта от идеи до релиза
---

# PRODUCT_CREATOR — Создание нового продукта

## Версия
- version: 4.0.0
- standalone: true
- purpose: product_creation

## Изменения v4.0.0
- CONTEXT.md кэширование: однократный сбор контекста, повторное использование агентами
- test-reviewer вместо test-engineer + code-reviewer (3 агента на задачу вместо 4)
- Компактный формат TDD roadmaps
- Умный retry с сжатием промпта (full → error → minimal)
- Глобальный лимит агентов 5 → 3
- Удаление дублирований, сжатие SKILL.md (~60%)
- Обновление артефактов: TEST_AND_REVIEW вместо TEST_REPORT + CODE_REVIEW

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
│   ├── FEATURES_INDEX.md
│   ├── SYSTEM_VERIFICATION.md
│   ├── ARCHITECTURE.md
│   ├── USAGE.md
│   ├── DEPLOY.md
│   └── RELEASE_NOTES.md
│
├── roadmaps/             # TDD roadmaps для задач фич (Фаза 5)
│   └── ROADMAP_TASKS_<feature>.md
│
└── develop/              # Артефакты разработки задач (Фаза 6)
    ├── CONTEXT.md        # Кэш контекста проекта (создаётся один раз)
    └── <FEATURE>/        # Артефакты фичи
        └── <TASK>/       # Артефакты задачи
            ├── IMPLEMENTATION_REPORT_<TASK>.md
            ├── TEST_AND_REVIEW_<TASK>.md
            └── FEATURE_VERIFICATION_<TASK>.md

README.md                 # Главный файл проекта (в корне!)
```

**Правила:**
- **README.md → в корне проекта** (главный файл документации)
- Все проектные артефакты → `docs/project/`
- Roadmaps задач → `docs/roadmaps/ROADMAP_TASKS_<feature>.md`
- CONTEXT.md → `docs/develop/CONTEXT.md` (один на весь проект)
- Артефакты разработки задачи → `docs/develop/<FEATURE>/<TASK>/`

---

## GIT WORKFLOW (ОБЯЗАТЕЛЬНЫЙ)

**Основная ветка разработки:** `{MAIN_BRANCH}` — определяется в Фазе 0, формат `<PROJECT>-DEV`

### Структура веток (ФИЧИ ПОСЛЕДОВАТЕЛЬНО, ЗАДАЧИ ПАРАЛЛЕЛЬНО)

```
{MAIN_BRANCH} (основная ветка)
│
├── Фаза 1-5: коммиты на {MAIN_BRANCH}
│
├──────────────────────────────────────────────────┐
│ ФАЗА 6: Реализация фич (ПОСЛЕДОВАТЕЛЬНО)         │
│                                                  │
│  ФИЧА F-001 → feature/<name>                     │
│  ├─ Задачи параллельно (до 3)                    │
│  ├─ Коммиты задач → feature/<name>               │
│  └─ Merge → {MAIN_BRANCH}                        │
│                                                  │
│  ФИЧА F-002 → feature/<name>                     │
│  ├─ ...                                          │
│  └─ Merge → {MAIN_BRANCH}                        │
│                                                  │
│  ФИЧА F-003 → ...                                │
└──────────────────────────────────────────────────┘
    │
    ├── Фаза 7: System Verification
    ├── Фаза 8: Documentation
    └── Фаза 9: Release
```

### Ответственность за коммиты

| Кто | Когда делает коммит | Что коммитит |
|-----|---------------------|--------------|
| **Агенты фаз 1-5, 7-9** | После завершения работы фазы | Созданные артефакты |
| **Оркестратор** | После каждой задачи (score ≥ 9) | Код + артефакты задачи |
| **Оркестратор** | Merge фичи в {MAIN_BRANCH} | Вся фича |

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
   - Формат: `<PROJECT>-DEV`

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

4. Запомни имя ветки как `{MAIN_BRANCH}`

---

### Фаза 1 — Формализация проекта

Выполни через Task tool:
```
subagent_type: project-profile-generator
```

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

    agent_result = Task(subagent_type="project-profile-generator", prompt="""
    ПЕРЕЗАПУСК С ОТВЕТАМИ:

    Файл docs/project/USER_ANSWERS.md содержит ответы пользователя.
    Используй эти ответы для создания финальных PROJECT_PROFILE.md.
    НЕ задавай те же вопросы повторно.
    """)

    remove("docs/project/USER_ANSWERS.md")
```

ДЕЙСТВИЯ:
- Прочитать `docs/project/PROJECT_PROFILE_HUMAN.md` через Read tool
- Включить ПОЛНОЕ содержимое в текстовый ответ между "---"
- Запросить явное подтверждение через AskUserQuestion

При правках — ПОВТОРИТЬ эту фазу.

---

### Фаза 2 — Определение пайплайна

Выполни через Task tool:
```
subagent_type: pipeline-prompt-generator
```

**Агент сам сделает коммит** после создания артефактов.

Выход: `docs/project/PIPELINE_PROMPT.md`

ДЕЙСТВИЯ:
- Прочитать `docs/project/PIPELINE_PROMPT.md` через Read tool
- Включить ПОЛНОЕ содержимое в текстовый ответ между "---"
- Запросить явное подтверждение через AskUserQuestion

При правках — ПОВТОРИТЬ Фазу 1 и Фазу 2.

---

### Фаза 3 — Аналитика

Выполни через Task tool **с обработкой вопросов**:

1. **research-agent** → `docs/project/ANALYSIS.md`

2. **system-analyst** — ЦИКЛ с обработкой вопросов:

   ```
   ЦИКЛ while (true):
     Task(subagent_type="system-analyst", prompt="Проанализируй docs/project/ANALYSIS.md и создай системные требования.")

     ПРОВЕРКА: существует ли файл docs/project/CLARIFICATION_NEEDED.md?

     Если ДА:
       → Читаешь CLARIFICATION_NEEDED.md
       → Задаю вопросы пользователю через AskUserQuestion
       → Создаю docs/project/USER_ANSWERS.md
       → Удаляю CLARIFICATION_NEEDED.md
       → Повторяю цикл

     Если НЕТ:
       → TECH_REQUIREMENTS.md и SCOPE.md созданы
       → Выход из цикла
   ```

**Агенты сами сделают коммит** после создания артефактов.

---

### Фаза 4 — Архитектура и декомпозиция на ФИЧИ

Выполни через Task tool (последовательно):

1. **solution-architect** → `docs/project/ARCHITECTURE_OVERVIEW.md`

2. **feature-decomposer** → `docs/project/FEATURES_INDEX.md` (ЦИКЛ с обработкой вопросов):

   ```
   ЦИКЛ while (true):
     Task(subagent_type="feature-decomposer", prompt="Выполни декомпозицию архитектуры на ФИЧИ.")

     Если CLARIFICATION_NEEDED.md существует:
       → Обработка вопросов через AskUserQuestion
       → Перезапуск с ответами

     Если НЕТ:
       → FEATURES_INDEX.md создан
       → Выход из цикла
   ```

Каждая фича ОБЯЗАНА иметь поле Domain.

**Агенты сами сделают коммит** после создания артефактов.

---

### Фаза 5 — CONTEXT.md + TDD планирование

#### Шаг 5.1: Создание CONTEXT.md (кэш контекста)

Перед TDD планированием оркестратор создаёт `docs/develop/CONTEXT.md` **один раз**.
Все агенты Фазы 6 читают его вместо получения полного контекста через промпт.

```python
project_context = Task(
    subagent_type="Explore",
    prompt="""
Собри краткий контекст проекта:
1. Tech stack
2. Architecture summary
3. Coding conventions
4. Build/run/test команды

Формат: КРАТКО, только факты.
"""
)

create_file("docs/develop/CONTEXT.md", f"""
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
""")
```

**Выход:** `docs/develop/CONTEXT.md`

#### Шаг 5.2: TDD планирование (компактный формат)

Выполни через Task tool последовательно для **КАЖДОЙ фичи**.

**Путь назначения:** `docs/roadmaps/ROADMAP_TASKS_<feature>.md`

**Алгоритм:**

1. Прочитай `docs/project/FEATURES_INDEX.md` — список всех фич
2. Для каждой фичи запусти `tdd-planner` последовательно
3. Дождись завершения, проверь создание файла
4. Перейди к следующей фиче

**Промпт для tdd-planner (компактный формат):**

```
Создай КОМПАКТНЫЙ TDD roadmap для фичи:

Feature ID: {feature_id}
Feature Name: {feature_name}
Feature Description: {feature_description}
Domain: {feature_domain}
Dependencies: {feature_dependencies}

Входные артефакты:
- docs/project/FEATURES_INDEX.md
- docs/project/ARCHITECTURE_OVERVIEW.md
- docs/project/PROJECT_PROFILE.md

ФОРМАТ ROADMAP:

## Feature {feature_id}: {feature_name}
**Domain:** {feature_domain} | **Dependencies:** {feature_dependencies}

### Task T-001: <Task Name>
**Domain:** {feature_domain} | **Dependencies:** None

#### Checklist
- [ ] CODE: <файлы для создания/изменения>
- [ ] TEST: <тесты>
- [ ] BUILD: <команда сборки>

#### Acceptance
- <критерии приёмки>

[повторить для каждой задачи, 3-10 задач на фичу, 2-4 часа каждая]

⚠️ ОБЯЗАТЕЛЬНО:
1. Создай файл docs/roadmaps/ROADMAP_TASKS_{feature_id}.md
2. Сделай git commit
```

**Агенты сами сделают коммит** после создания roadmaps.

---

## ОБРАБОТКА ОШИБОК ПРИ ЗАПУСКЕ АГЕНТОВ

### Типичные ошибки

| Ошибка | Признак | Решение |
|--------|---------|---------|
| **Empty result** | Агент вернул пустую строку | Retry с сжатием промпта |
| **Rate limit** | 429, timeout | Подождать 60 сек, retry |
| **Agent crash** | TaskOutput вернул ошибку | Retry до 3 раз |
| **Invalid output** | Артефакт не создан | Retry с уточнением |

### safe_launch_agent (с сжатием промпта при retry)

```python
def safe_launch_agent(subagent_type, prompt, max_retries=3, base_delay=30):
    """
    Безопасный запуск агента с обработкой ошибок.
    Retry с сжатием промпта: full → error → minimal
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

    log_failed_task(extract_task_id(prompt), subagent_type, "failed after retries")
    return None
```

### log_failed_task

```python
def log_failed_task(task_id, agent_type, error_reason):
    append_to_file("docs/project/FAILED_TASKS.md", f"""
## Failed Task: {task_id}
- Agent: {agent_type}
- Error: {error_reason}
- Timestamp: {datetime.now()}
- Action: MANUAL REVIEW REQUIRED
""")
```

### verify_tdd_planning_complete

```python
def verify_tdd_planning_complete(features):
    """Проверить что все roadmap созданы."""
    failed = []
    for feature in features:
        path = f"docs/roadmaps/ROADMAP_TASKS_{feature['id']}.md"
        if not file_exists(path) or file_size(path) < 500:
            failed.append(feature['id'])
    if failed:
        print(f"⚠️ Roadmap не созданы для: {', '.join(failed)}")
        return False
    return True
```

---

### Фаза 6 — Реализация ФИЧ (ПОСЛЕДОВАТЕЛЬНО) и ЗАДАЧ (ПАРАЛЛЕЛЬНО)

**Критические правила:**
- **ФИЧИ — ПОСЛЕДОВАТЕЛЬНО** (одна за другой)
- **ЗАДАЧИ внутри фичи — ПАРАЛЛЕЛЬНО** (до 3 штук)
- Каждая фича = отдельная ветка `feature/<name>`
- **ГЛОБАЛЬНЫЙ ЛИМИТ: Максимум 3 агента одновременно**
- После завершения всех задач фичи → merge в `{MAIN_BRANCH}`

---

## ЦИКЛ РАЗРАБОТКИ ОДНОЙ ЗАДАЧИ

```
1. developer-agent (последовательно)
   ↓
2. test-reviewer (ОДИН агент: тесты + ревью)
   ↓
3. feature-verifier (ВСЕГДА, после test-reviewer)
   ↓
4. Проверка score
   ┌────────┴────────┐
   │                 │
score ≥ 9        score < 9
   │                 │
   ↓                 ↓
Коммит задачи   Доработка (повтор цикла)
```

**Ключевые моменты:**
1. Developer идёт первым (создаёт код)
2. test-reviewer выполняет тесты И ревью в одном агенте
3. Verifier запускается ВСЕГДА после test-reviewer
4. При score < 9 — возврат к developer с контекстом доработки

---

## Промпты для агентов Фазы 6

### Промпт developer-agent

```
См. CONTEXT.md в docs/develop/

Задача: {task_id} — {task_name}
Feature: {feature_name}
Domain: {feature_domain}

Roadmap: docs/roadmaps/ROADMAP_TASKS_{feature_id}.md

Выполни:
1. Изучи CONTEXT.md
2. Изучи roadmap
3. Реализуй задачу
4. Создай IMPLEMENTATION_REPORT_{task_id}.md (кратко)
```

### Промпт test-reviewer

```
См. CONTEXT.md в docs/develop/

Задача: {task_id} — {task_name}
Feature: {feature_name}

Roadmap: docs/roadmaps/ROADMAP_TASKS_{feature_id}.md
Implementation: docs/develop/{feature_id}/{task_id}/IMPLEMENTATION_REPORT_{task_id}.md

Выполни:
1. BUILD: команда из CONTEXT.md
2. RUN: проверь запуск
3. TESTS: по roadmap
4. REVIEW: проверка кода

Создай TEST_AND_REVIEW_{task_id}.md

Формат:
## Test & Review: {task_id}

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
```

### Промпт feature-verifier

```
См. CONTEXT.md в docs/develop/

Задача: {task_id}
Feature: {feature_name}

Roadmap: docs/roadmaps/ROADMAP_TASKS_{feature_id}.md
TEST_AND_REVIEW: docs/develop/{feature_id}/{task_id}/TEST_AND_REVIEW_{task_id}.md
HAS_ISSUES из test-reviewer: {has_issues}
Критичная задача: {is_critical}

Выполни ПОЛНУЮ верификацию:
1. Проверь все артефакты
2. Учти выявленные проблемы из TEST_AND_REVIEW
3. Проверь build/run/tests
4. Оцени качество кода

Создай FEATURE_VERIFICATION_{task_id}.md с итоговым score (0-10).
```

---

## Псевдокод Фазы 6

```python
features = parse_features("docs/project/FEATURES_INDEX.md")

for feature in features:
    feature_id = feature["id"]

    # Создать ветку фичи
    bash_command(f"git checkout -b feature/{feature_id}")

    tasks = parse_tasks(f"docs/roadmaps/ROADMAP_TASKS_{feature_id}.md")

    # Группируем задачи по зависимостям (Level 0, 1, ...)
    for level_tasks in group_by_level(tasks):
        # ========================================
        # ШАГ 1: developer-agent (ПАРАЛЛЕЛЬНО)
        # ========================================
        dev_tasks = []
        for task in level_tasks:
            task_dev = Task(subagent_type="developer-agent", prompt=...)
            dev_tasks.append((task, task_dev))

        for task, task_dev in dev_tasks:
            TaskOutput(task_id=task_dev["id"], block=True, timeout=600000)

        # ========================================
        # ШАГ 2: test-reviewer (ПАРАЛЛЕЛЬНО)
        # ========================================
        review_tasks = []
        for task in level_tasks:
            task_review = Task(subagent_type="test-reviewer", prompt=...)
            review_tasks.append((task, task_review))

        for task, task_review in review_tasks:
            TaskOutput(task_id=task_review["id"], block=True, timeout=600000)

        # ========================================
        # ШАГ 3: feature-verifier (ПОСЛЕДОВАТЕЛЬНО)
        # ========================================
        for task in level_tasks:
            task_verify = Task(subagent_type="feature-verifier", prompt=...)
            result = TaskOutput(task_id=task_verify["id"], block=True, timeout=600000)

            score = extract_score(result)

            if score >= 9:
                # Отметить в roadmap
                roadmap_path = f"docs/roadmaps/ROADMAP_TASKS_{feature_id}.md"
                roadmap = read_file(roadmap_path)
                updated_roadmap = mark_task_completed(roadmap, task['id'], score)
                write_file(roadmap_path, updated_roadmap)

                # Коммит roadmap + артефакты задачи
                bash_command(f"""
                    git add {roadmap_path}
                    git commit -m "docs: mark task {task['id']} as completed (score {score}/10)"
                """)
                bash_command(f"""
                    git add docs/develop/{feature_id}/{task['id']}/
                    git commit -m "feat: {task['name']} ({task['id']})"
                """)
            else:
                # Доработка: повторить цикл для этой задачи
                # developer-agent получает контекст из FEATURE_VERIFICATION + TEST_AND_REVIEW
                pass

    # ========================================
    # ВСЕ задачи фичи завершены → Merge
    # ========================================
    bash_command(f"""
        git checkout {MAIN_BRANCH}
        git merge feature/{feature_id}
        git branch -d feature/{feature_id}
    """)

    # Отметить фичу в FEATURES_INDEX.md
    features_index = read_file("docs/project/FEATURES_INDEX.md")
    updated_index = mark_feature_completed(features_index, feature_id)
    write_file("docs/project/FEATURES_INDEX.md", updated_index)
    bash_command(f"""
        git add docs/project/FEATURES_INDEX.md
        git commit -m "docs: mark feature {feature_id} as completed"
    """)
```

### Коммит артефактов задачи

```bash
# После успешной верификации задачи (score ≥ 9)
git add docs/develop/{feature}/{task_id}/
git commit -m "feat: {task_name} ({task_id})

- Implementation: developer-agent
- Test & Review: test-reviewer
- Verification: feature-verifier (score ≥ 9)
"
```

---

## Функции отметки

```python
def mark_task_completed(roadmap_content, task_id, score):
    """Добавляет статус выполненной задачи в roadmap"""
    from datetime import datetime
    completed_date = datetime.now().strftime("%Y-%m-%d")
    task_section = find_task_section(roadmap_content, task_id)

    if "**Status:**" in task_section:
        updated = task_section.replace(
            "**Status:** ⏳ IN PROGRESS",
            f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Final Score:** {score}/10"
        )
    else:
        updated = task_section.replace(
            f"### Task {task_id}:",
            f"### Task {task_id}:\n**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Final Score:** {score}/10"
        )
    return roadmap_content.replace(task_section, updated)


def mark_feature_completed(features_index_content, feature_id):
    """Добавляет статус выполненной фичи в FEATURES_INDEX.md"""
    from datetime import datetime
    completed_date = datetime.now().strftime("%Y-%m-%d")
    feature_section = find_feature_section(features_index_content, feature_id)

    if "**Status:**" in feature_section:
        updated = feature_section.replace(
            "**Status:** ⏳ IN PROGRESS",
            f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}"
        )
    else:
        updated = feature_section.replace(
            f"**Notes:**",
            f"**Status:** ✅ COMPLETED\n**Completed:** {completed_date}\n**Notes:**"
        )
    return features_index_content.replace(feature_section, updated)
```

---

## Доработка при score < 9

1. Прочитать `FEATURE_VERIFICATION_<task>.md` — задачи на доработку
2. Прочитать `TEST_AND_REVIEW_<task>.md` — замечания и проблемы
3. Перезапустить `developer-agent` с контекстом доработки:

```
ПЕРЕРАБОТКА ЗАДАЧИ {task_id}

Текущий score: {score}/10

ЗАДАЧИ НА ДОРАБОТКУ (из FEATURE_VERIFICATION):
[содержимое]

ЗАМЕЧАНИЯ (из TEST_AND_REVIEW):
[содержимое]

Выполни доработку. Обнови IMPLEMENTATION_REPORT_{task_id}.md.
```

4. Повторить цикл: test-reviewer → feature-verifier

---

## ЗАПРЕЩЕНО

- ❌ Передавать несколько задач в один агент
- ❌ Запускать feature-verifier ДО test-reviewer
- ❌ Запускать test-reviewer ДО developer-agent
- ❌ Создавать артефакты вне `docs/develop/<FEATURE>/<TASK>/`
- ❌ Создавать только IMPLEMENTATION_REPORT и пропускать test/review/verify
- ❌ Запускать более 3 агентов одновременно
- ❌ Разрабатывать несколько фич параллельно
- ❌ Игнорировать зависимости между задачами
- ❌ Передавать полный контекст в промпте (использовать CONTEXT.md)
- ❌ Использовать Skill(commit) — оркестратор делает напрямую через `git commit`
- ❌ Запускать developer для всех задач разом

---

## РАЗРЕШЕНО

- ✅ Разрабатывать до 3 задач **параллельно** (при отсутствии зависимостей)
- ✅ Каждая задача проходит **полный цикл** (dev → test-reviewer → verify)
- ✅ Агенты фаз 1-5, 7-9 делают коммиты **сами**
- ✅ Оркестратор делает коммиты и merge в **Фазе 6**

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

**Пайплайн завершён! Продукт создан и задокументирован.**
