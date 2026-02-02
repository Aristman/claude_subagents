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
│   ├── WORK_BREAKDOWN.md
│   ├── FEATURES_INDEX.md
│   ├── SYSTEM_VERIFICATION.md
│   ├── README.md
│   ├── ARCHITECTURE.md
│   ├── USAGE.md
│   ├── DEPLOY.md
│   └── RELEASE_NOTES.md
│
├── roadmaps/             # TDD roadmaps для каждой фичи (Фаза 5)
│   ├── ROADMAP_<FEATURE>_01.md
│   ├── ROADMAP_<FEATURE>_02.md
│   └── ...
│
└── develop/              # Артефакты разработки каждой фичи (Фаза 6)
    ├── <FEATURE_01>/
    │   ├── IMPLEMENTATION_REPORT_<FEATURE_01>.md
    │   ├── TEST_REPORT_<FEATURE_01>.md
    │   ├── CODE_REVIEW_<FEATURE_01>.md
    │   └── FEATURE_VERIFICATION_<FEATURE_01>.md
    ├── <FEATURE_02>/
    │   └── ...
    └── ...
```

**Правила:**
- Все проектные артефакты → `docs/project/`
- Все roadmaps → `docs/roadmaps/ROADMAP_<FEATURE>_<NUMBER>.md`
- Артефакты разработки фичи → `docs/develop/<FEATURE>/`

---

## GIT WORKFLOW (ОБЯЗАТЕЛЬНЫЙ)

**Основная ветка разработки:** `{MAIN_BRANCH}` — определяется в Фазе 0, формат `<PROJECT>-DEV`

### Структура веток

```
{MAIN_BRANCH} (основная ветка, например SW-DEV)
│
└── (коммиты на каждой стадии)
    ├── Фаза 1: Project Profile
    ├── Фаза 2: Pipeline Definition
    ├── Фаза 3: Analytics
    ├── Фаза 4: Architecture
    ├── Фаза 5: TDD Planning
    ├── Фаза 6: Implementation (множество коммитов от агентов)
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

**Для всех фаз 1-9:**
1. Всё работает в ветке `{MAIN_BRANCH}`
2. **Агенты сами делают коммиты** при завершении своей работы
3. Оркестратор НЕ делает коммиты
4. Pull Request НЕ используются (только коммиты в основную ветку)

### Ответственность за коммиты

| Кто | Когда делает коммит | Что коммитит |
|-----|---------------------|--------------|
| **Агенты фаз 1-5, 7-9** | После завершения работы фазы | Созданные артефакты |
| **developer-agent** | После реализации фичи | Код + IMPLEMENTATION_REPORT |
| **test-engineer** | После тестирования | TEST_REPORT |
| **code-reviewer** | После ревью | CODE_REVIEW |
| **feature-verifier** | После верификации | FEATURE_VERIFICATION |
| **Оркестратор** | **НЕ делает коммиты** | — |

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
### Фаза 4 — Архитектура и декомпозиция

Выполни через Task tool (последовательно):

1. **solution-architect** → `docs/project/ARCHITECTURE_OVERVIEW.md`
   ```
   Task(subagent_type="solution-architect", prompt="...")
   ```

2. **feature-decomposer** — ЦИКЛ с обработкой вопросов:

   ```
   ЦИКЛ while (true):
     Task(subagent_type="feature-decomposer", prompt="Выполни декомпозицию архитектуры на фичи.")

     ПРОВЕРКА: существует ли файл docs/project/CLARIFICATION_NEEDED.md?

     Если ДА:
       → Читаешь CLARIFICATION_NEEDED.md
       → Задаю вопросы пользователю через AskUserQuestion
       → Получаю ответы
       → Создаю docs/project/USER_ANSWERS.md с ответами
       → Повторяю цикл (агент перезапустится с ответами)

     Если НЕТ:
       → Артефакты WORK_BREAKDOWN.md и FEATURES_INDEX.md созданы
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

    Используй эти ответы для создания финальных WORK_BREAKDOWN.md и FEATURES_INDEX.md.
    НЕ задавай повторно те же вопросы.
    """)

    # После завершения агента - УДАЛЯЮ USER_ANSWERS.md
    remove("docs/project/USER_ANSWERS.md")
\`\`\`

Каждая фича ОБЯЗАНА иметь поле Domain.

**Агенты сами сделают коммит** после создания артефактов.
### Фаза 5 — TDD планирование

Выполни через Task tool с пакетной обработкой.

**Путь назначения:** `docs/roadmaps/`

**Формат имени файла:** `ROADMAP_<FEATURE>_<NUMBER>.md`

где:
- `<FEATURE>` — ID фичи (например, F001, AUTH-001)
- `<NUMBER>` — номер версии roadmap (01, 02, ...)

**Алгоритм:**

1. Прочитай `docs/project/FEATURES_INDEX.md` и получи список всех фич
2. Для каждой фичи определи зависимости (поле `Dependencies` в FEATURES_INDEX.md)
3. Разбей фичи на пакеты по **максимум 5 штук**
4. Для каждого пакета запусти `tdd-planner` **параллельно** в одном сообщении:
   - Используй отдельный Task вызов для каждой фичи
   - Максимум 5 параллельных Task вызовов в одном сообщении
5. Дождись завершения всех Task в пакете
6. Повторяй для следующего пакета, пока все фичи не будут обработаны

**Пример запуска пакета:**
\`\`\`
// Одно сообщение с несколькими Task вызовами:
Task(subagent_type="tdd-planner", prompt="... Feature F-001 ...")
Task(subagent_type="tdd-planner", prompt="... Feature F-002 ...")
Task(subagent_type="tdd-planner", prompt="... Feature F-003 ...")
Task(subagent_type="tdd-planner", prompt="... Feature F-004 ...")
Task(subagent_type="tdd-planner", prompt="... Feature F-005 ...")
\`\`\`

**Промпт для каждого tdd-planner:**
\`\`\`
Создай TDD roadmap для фичи:

Feature ID: {feature_id}
Feature Name: {feature_name}
Domain: {feature_domain}
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
### Фаза 6 — Реализация и верификация фич (ДЕТАЛИЗИРОВАНО)

**Все работы в ветке:** `{MAIN_BRANCH}`

**Путь назначения артефактов разработки:** `docs/develop/<FEATURE>/`


**Путь назначения артефактов разработки:** `docs/develop/<FEATURE>/`

**Формат:** для фичи `{feature_id}` все артефакты сохраняются в `docs/develop/{feature_id}/`

---

## ⚠️ КРИТИЧЕСКИ ВАЖНЫЕ ТРЕБОВАНИЯ

### 1. Полный цикл для КАЖДОЙ фичи (НЕПРИЕМЛЕМО ПРОПУСКАТЬ ШАГИ)

**КАЖДАЯ фича ДОЛЖНА иметь ВСЕ артефакты:**
- ✅ `IMPLEMENTATION_REPORT_<feature>.md` — от developer-agent
- ✅ `TEST_REPORT_<feature>.md` — от test-engineer
- ✅ `CODE_REVIEW_<feature>.md` — от code-reviewer
- ✅ `FEATURE_VERIFICATION_<feature>.md` — от feature-verifier (с score ≥ 9)

❌ **ЗАПРЕЩЕНО:**
- Создавать только IMPLEMENTATION_REPORT и пропускать остальные
- Группировать фичи для пакетной обработки
- "Устать" и делать фичи в ускоренном режиме

### 2. Параллельная разработка фич (до 3 одновременно)

✅ **МОЖНО разрабатывать до 3 фич параллельно** при соблюдении условий:
- У фич **нет зависимостей** друг от друга (проверь Dependencies в FEATURES_INDEX.md)
- Каждая фича проходит **ПОЛНЫЙ цикл** разработки
- Все фичи разрабатываются в ветке {MAIN_BRANCH}, каждая фича = отдельный коммит

❌ **ЗАПРЕЩЕНО:**
- Разрабатывать более 3 фич параллельно
- Игнорировать зависимости между фичами
- Запускать разработку зависимой фичи до завершения родительской

### 3. Учёт зависимостей фич

**Порядок разработки:**
1. Сначала фичи без зависимостей (Level 0)
2. Затем фичи, зависящие от Level 0 (Level 1)
3. И так далее по дереву зависимостей

**Пример:**
```
F-001 (база данных)      → нет зависимостей → Level 0
F-002 (API сервис)       → зависит от F-001 → Level 1
F-003 (авторизация)      → зависит от F-001 → Level 1
F-004 (dashboard)        → зависит от F-002 → Level 2

Порядок: (F-001) → (F-002, F-003 параллельно) → (F-004)
```

---

## Алгоритм выполнения

---

## Git Workflow для Фазы 6 (Оркестратор делает коммиты)

### Ответственность за коммиты

**Оркестратор делает коммит ПОСЛЕ успешной верификации каждой фичи:**

| Агент | Действие | Коммит? |
|-------|----------|---------|
| developer-agent | Реализует фичу | ❌ Нет |
| test-engineer | Тестирует фичу | ❌ Нет |
| code-reviewer | Делает ревью | ❌ Нет |
| feature-verifier | Верифицирует | ❌ Нет |
| **Оркестратор** | **Делает коммит после score ≥ 9** | ✅ **Да** |

### Когда делать коммит

**Секвенциальная разработка одной фичи:**
```
developer-agent → test-engineer → code-reviewer → feature-verifier
                                                                  ↓
                                                           score ≥ 9?
                                                                ✅ Да
                                                        ┌───────────────┐
                                                        │ ОРКЕСТРАТОР   │
                                                        │ делает коммит │
                                                        └───────────────┘
```

**Параллельная разработка 3 фич:**
```
Фича F-001: developer → test → review → verifier → score ≥ 9 → КОММИТ F-001
Фича F-002: developer → test → review → verifier → score ≥ 9 → КОММИТ F-002
Фича F-003: developer → test → review → verifier → score ≥ 9 → КОММИТ F-003
```

### Что коммитить

**Для фичи `{feature_id}` коммитить:**
```
docs/develop/{feature_id}/IMPLEMENTATION_REPORT_{feature_id}.md
docs/develop/{feature_id}/TEST_REPORT_{feature_id}.md
docs/develop/{feature_id}/CODE_REVIEW_{feature_id}.md
docs/develop/{feature_id}/FEATURE_VERIFICATION_{feature_id}.md
```

### Команды для коммита (выполняет оркестратор)

```bash
# После успешной верификации фичи (score ≥ 9)
git add docs/develop/{feature_id}/
git commit -m "feat: {feature_name} ({feature_id})

- Implementation: developer-agent
- Test: test-engineer  
- Review: code-reviewer
- Verification: feature-verifier (score ≥ 9)
"
```

### Псевдокод для параллельной разработки

```python
# После завершения feature-verifier для всех фич в батче
for feature_id in batch:
    verification = read_file(f"docs/develop/{feature_id}/FEATURE_VERIFICATION_{feature_id}.md")
    score = extract_score(verification)
    
    if score >= 9:
        # ОРКЕСТРАТОР делает коммит
        bash_command(f"""
            git add docs/develop/{feature_id}/
            git commit -m "feat: {feature_name} ({feature_id})"
        """)
        print(f"✅ {feature_id}: коммит создан (score={score})")
    else:
        print(f"❌ {feature_id}: score={score} < 9, нужен redo")
```

### Критические правила

1. **Коммит ТОЛЬКО после score ≥ 9** от feature-verifier
2. **Агенты НЕ делают коммиты** — только оркестратор
3. **Каждая фича = отдельный коммит** (даже при параллельной разработке)
4. **НЕ использовать Skill(commit)** — оркестратор делает напрямую через `git commit`


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

┌─────────────────────────────────────────────────────────────────┐
│ ГРУППА ФИЧ Level N (до 3 фич параллельно)                      │
└─────────────────────────────────────────────────────────────────┘

**Для каждой фичи в группе запускаем ПОЛНЫЙ цикл:**

┌─────────────────────────────────────────────────────────────────┐
│ ФИЧА {feature_id_1}                                            │
├─────────────────────────────────────────────────────────────────┤
│ 1. developer-agent → IMPLEMENTATION_REPORT                     │
│ 2. test-engineer + code-reviewer (параллельно)                 │
│ 3. feature-verifier → FEATURE_VERIFICATION (score)             │
│ 4. Если score < 9 → доработка (повтор 1-3)                     │
│ 5. Если score ≥ 9 → оркестратор делает коммит                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ ФИЧА {feature_id_2} (ПАРАЛЛЕЛЬНО с {feature_id_1})             │
├─────────────────────────────────────────────────────────────────┤
│ 1. developer-agent → IMPLEMENTATION_REPORT                     │
│ 2. test-engineer + code-reviewer (параллельно)                 │
│ 3. feature-verifier → FEATURE_VERIFICATION (score)             │
│ 4. Если score < 9 → доработка (повтор 1-3)                     │
│ 5. Если score ≥ 9 → оркестратор делает коммит                   │
└─────────────────────────────────────────────────────────────────┘

⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ВСЕХ ФИЧ В ГРУППЕ перед переходом к следующему Level
```

**⚠️ ЯВНЫЙ ПСЕВДОКОД полного цикла для параллельной разработки:**

```python
# ============================================================
# ПОЛНЫЙ ЦИКЛ ПАРАЛЛЕЛЬНОЙ РАЗРАБОТКИ ГРУППЫ ФИЧ
# ============================================================

batch = ["F-005", "F-006", "F-007"]  # Level 2

# ─────────────────────────────────────────────────────────────────
# ЭТАП 1: developer-agent (ПАРАЛЛЕЛЬНО)
# ─────────────────────────────────────────────────────────────────
dev_tasks = []
for feature_id in batch:
    task = Task(subagent_type="developer-agent", prompt=f"Реализуй {feature_id}")
    dev_tasks.append((feature_id, task))

# ⚠️ КРИТИЧЕСКО: Ждём завершения ВСЕХ
for feature_id, task in dev_tasks:
    result = TaskOutput(task_id=task["id"], block=True, timeout=600000)

# ─────────────────────────────────────────────────────────────────
# ЭТАП 2: test-engineer + code-reviewer (ПАРАЛЛЕЛЬНО для каждой фичи)
# ─────────────────────────────────────────────────────────────────
test_review_tasks = []
for feature_id in batch:
    task_test = Task(subagent_type="test-engineer", prompt=f"Тестируй {feature_id}")
    task_review = Task(subagent_type="code-reviewer", prompt=f"Ревью {feature_id}")
    test_review_tasks.append((feature_id, task_test, task_review))

# ⚠️ КРИТИЧЕСКО: Ждём завершения ВСЕХ
for feature_id, task_test, task_review in test_review_tasks:
    TaskOutput(task_id=task_test["id"], block=True, timeout=600000)
    TaskOutput(task_id=task_review["id"], block=True, timeout=600000)

# ─────────────────────────────────────────────────────────────────
# ЭТАП 3: feature-verifier (ПОСЛЕДОВАТЕЛЬНО для каждой фичи)
# ─────────────────────────────────────────────────────────────────
for feature_id in batch:
    task_verify = Task(subagent_type="feature-verifier", prompt=f"Верифицируй {feature_id}")
    result = TaskOutput(task_id=task_verify["id"], block=True, timeout=600000)

    # Проверяем score из FEATURE_VERIFICATION.md
    score = extract_score(result)

    if score >= 9:
        # Оркестратор делает коммит
        git add docs/develop/{feature_id}/
        git commit -m "feat: {feature_id}"
    else:
        # ⚠️ КРИТИЧЕСКО: score < 9 — ПЕРЕЗАПУСК developer-agent с доработкой
        # Читаем задачи из FEATURE_VERIFICATION.md, CODE_REVIEW.md, TEST_REPORT.md
        verification = read_file(f"docs/develop/{feature_id}/FEATURE_VERIFICATION_{feature_id}.md")
        review = read_file(f"docs/develop/{feature_id}/CODE_REVIEW_{feature_id}.md")
        tests = read_file(f"docs/develop/{feature_id}/TEST_REPORT_{feature_id}.md")

        # Перезапускаем developer-agent с контекстом доработки
        Task(
            subagent_type="developer-agent",
            prompt=f"""
            ПЕРЕРАБОТКА ФИЧИ {feature_id} (score был {score}/10)

            ЗАДАЧИ ИЗ ВЕРИФИКАЦИИ:
            {verification}

            ЗАМЕЧАНИЯ ИЗ REVIEW:
            {review}

            ПРОБЛЕМЫ ИЗ ТЕСТОВ:
            {tests}

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
    ПЕРЕРАБОТКА ФИЧИ {feature_id}

    Текущая реализация получила score {score}/10.

    ЗАДАЧИ НА ДОРАБОТКУ (из FEATURE_VERIFICATION.md):
    [вставить задачи из верификации]

    ЗАМЕЧАНИЯ CODE REVIEW (из CODE_REVIEW.md):
    [вставить замечания]

    ПРОБЛЕМЫ В ТЕСТАХ (из TEST_REPORT.md):
    [вставить проблемы]

    Выполни доработку согласно этим задачам.
    Обнови IMPLEMENTATION_REPORT_{feature_id}.md с описанием изменений.
    """
)
```

5. Повторить весь цикл: test → review → verify
6. Повторять пока `score < 9`

---

### Завершение фичи (score ≥ 9)


---

**Промпт для каждого агента:**

```
Feature ID: {feature_id}
Feature Name: {feature_name}
Domain: {feature_domain}

Директория для артефактов: docs/develop/{feature_id}/

Выполни свою роль для этой фичи согласно твоим инструкциям в .md файле.
Все создаваемые артефакты сохраняй в указанную директорию.
```

**ЗАПРЕЩЕНО:**
- ❌ Передавать несколько фич в один агент
- ❌ Группировать фичи по версии для developer/test/review
- ❌ Запускать developer для всех фич разом
- ❌ Использовать feature-verifier для целой версии
- ❌ Создавать артефакты вне `docs/develop/<FEATURE>/`
- ❌ **КРИТИЧЕСКО: Создавать только IMPLEMENTATION_REPORT и пропускать test/review/verify!**
  - КАЖДАЯ фича ДОЛЖНА иметь ВСЕ 4 артефакта
  - "Усталость" оркестатора — НЕ оправдание
- ❌ **КРИТИЧЕСКО: Запускать feature-verifier ДО test-engineer и code-reviewer!**
  - feature-verifier зависит от ОБИХ: TEST_REPORT.md И CODE_REVIEW.md
- ❌ **КРИТИЧЕСКО: Запускать test-engineer или code-reviewer ДО developer-agent!**
  - Они требуют IMPLEMENTATION_REPORT.md от developer
- ❌ **Запускать все 4 агента в одном сообщении** (developer/test/review/verify)
- ❌ **Разрабатывать более 3 фич параллельно**
- ❌ **Игнорировать зависимости между фичами**
  - Зависимая фича НЕ может разрабатываться до родительской

**РАЗРЕШЕНО (оптимизация):**
- ✅ **Запускать test-engineer и code-reviewer параллельно** (в одном сообщении)
  - Они независимы и могут работать одновременно
  - feature-verifier запускается только ПОСЛЕ завершения ОБИХ
- ✅ **Разрабатывать до 3 фич параллельно** (при отсутствии зависимостей)
  - Каждая фича проходит ПОЛНЫЙ цикл разработки
  - Фичи должны быть одного Level (без зависимостей друг от друга)

---

## Пример параллельной разработки 3 фич

```
Level 0 (нет зависимостей):
┌─────────────────────────────────────────────────────────────────┐
│ ФИЧИ F-001, F-002, F-003 — ПАРАЛЛЕЛЬНО                         │
└─────────────────────────────────────────────────────────────────┘

[ОДНО сообщение с ТРЁМЯ developer-agent Task]
Task(developer-agent, prompt="...F-001...")  ─┐
Task(developer-agent, prompt="...F-002...")  ─┼─ ПАРАЛЛЕЛЬНО
Task(developer-agent, prompt="...F-003...")  ─┘

⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ВСЕХ ТРЁХ

[ОДНО сообщение с ШЕСТЬЮ Task — test + review для каждой фичи]
Task(test-engineer, "...F-001...")           ─┐
Task(code-reviewer, "...F-001...")          ─┤
Task(test-engineer, "...F-002...")           ─┼─ ПАРАЛЛЕЛЬНО
Task(code-reviewer, "...F-002...")          ─┤
Task(test-engineer, "...F-003...")           ─┤
Task(code-reviewer, "...F-003...")          ─┘

⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ВСЕХ ШЕСТИ

[ТРИ сообщения для verifier — последовательно для каждой фичи]
Task(feature-verifier, "...F-001...")  → score
Task(feature-verifier, "...F-002...")  → score
Task(feature-verifier, "...F-003...")  → score

⚠️ ЖДЁМ ЗАВЕРШЕНИЯ ВСЕХ ТРЁХ

Для каждой фичи с score < 9 — повтор цикла
Для каждой фичи с score ≥ 9 — агент уже сделал коммит

⚠️ ТОЛЬКО ПОСЛЕ ВСЕХ ФИЧ Level 0 → переход к Level 1
```

---

## Проверка зависимостей перед разработкой

**Перед запуском разработки фичи ОБЯЗАТЕЛЬНО:**

1. Прочитать `docs/roadmaps/ROADMAP_{feature}.md`
2. Найти секцию **Dependencies**
3. Проверить что все зависимые фичи:
   - Реализованы (есть IMPLEMENTATION_REPORT)
   - Прошли верификацию (score ≥ 9)
   - Есть коммиты в основной ветке

**Если зависимости не выполнены:**
- НЕ запускать разработку этой фичи
- Перейти к следующей фиче без зависимостей
- Вернуться к зависимой фиче позже

**Завершение Фазы 6 (после ВСЕХ фич):**

Все агенты сами сделали коммиты. Фаза 6 завершена.

**Правила:**
- Агенты (developer, test-engineer, code-reviewer, feature-verifier) сами делают коммиты после завершения работы
- Каждая фича = несколько коммитов (реализация, тестирование, ревью, верификация)
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
