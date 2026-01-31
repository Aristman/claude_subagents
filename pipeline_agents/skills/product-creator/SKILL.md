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
├── phase-1-project-profile      ┐
├── phase-2-pipeline             │
├── phase-3-analytics            │ 9 веток фаз
├── phase-4-architecture         │ (phase-N → {MAIN_BRANCH})
├── phase-5-tdd-planning         │
├── phase-6-implementation       ┘
│   ├── feature-F001             ┐
│   ├── feature-F002             │ Ветки фич
│   ├── feature-F003             │ (feature-N → phase-6)
│   └── ...                      ┘
├── phase-7-system-verification ┐
├── phase-8-documentation        │ 3 ветки фаз
└── phase-9-release              ┘ (phase-N → {MAIN_BRANCH})
```

### Общий алгоритм работы с ветками

**В начале пайплайна (Фаза 0):**
1. Определить имя основной ветки `{MAIN_BRANCH}`
2. Проверить существование ветки `{MAIN_BRANCH}`
3. Если не существует — создать от текущей ветки
4. Переключиться на `{MAIN_BRANCH}`

**Для каждой фазы 1-5, 7-9:**
1. Создать ветку `phase-N-<name>` от `{MAIN_BRANCH}`
2. Переключиться на ветку фазы
3. Выполнить работу фазы
4. Создать коммит через `Skill(commit)`
5. Создать Pull Request: `phase-N-<name>` → `{MAIN_BRANCH}`
6. Смёржить PR (merge)

**Для фазы 6 (реализация фич):**
1. Создать ветку `phase-6-implementation` от `{MAIN_BRANCH}`
2. Для каждой фичи:
   - Создать ветку `feature-<ID>` от `phase-6-implementation`
   - Выполнить цикл разработки
   - Создать PR: `feature-<ID>` → `phase-6-implementation`
   - Смёржить PR
3. После всех фич создать PR: `phase-6-implementation` → `{MAIN_BRANCH}`
4. Смёржить PR

### Git команды для работы с ветками

**Создание ветки:**
```bash
git checkout -b <branch-name>
```

**Создание Pull Request (через gh CLI):**
```bash
gh pr create --base <base-branch> --head <current-branch> --title "<title>" --body "<body>"
```

**Мёрж PR:**
```bash
gh pr merge <pr-number> --merge --delete-branch
```

**Автоматизация создания PR:**
Используй Bash tool для выполнения команд `gh`.

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

**Git операции:**
```bash
# Создать ветку фазы от {MAIN_BRANCH}
git checkout {MAIN_BRANCH}
git checkout -b phase-1-project-profile
```

Выполни через Task tool:
```
subagent_type: project-profile-generator
```

Выход: `docs/project/PROJECT_PROFILE.md`, `docs/project/PROJECT_PROFILE_HUMAN.md`

ДЕЙСТВИЯ:
- Прочитать файл `docs/project/PROJECT_PROFILE_HUMAN.md` через Read tool
- Включить ПОЛНОЕ содержимое в текстовый ответ между "---"
- Запросить явное подтверждение через AskUserQuestion

При правках — ПОВТОРИТЬ эту фазу.

**После подтверждения:**
1. Создай коммит:
```
Skill(skill="commit", args="docs/project/PROJECT_PROFILE.md docs/project/PROJECT_PROFILE_HUMAN.md")
```
2. Создай Pull Request:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-1-project-profile \
  --title "Phase 1: Project Profile" \
  --body "Формализация проекта: PROJECT_PROFILE.md"
```
3. Смёржь PR:
```bash
gh pr merge --merge --delete-branch
```
4. Переключись обратно на {MAIN_BRANCH}:
```bash
git checkout {MAIN_BRANCH}
```

---

### Фаза 2 — Определение пайплайна

**Git операции:**
```bash
git checkout {MAIN_BRANCH}
git checkout -b phase-2-pipeline
```

Выполни через Task tool:
```
subagent_type: pipeline-prompt-generator
```

Выход: `docs/project/PIPELINE_PROMPT.md`

ДЕЙСТВИЯ:
- Прочитать файл `docs/project/PIPELINE_PROMPT.md` через Read tool
- Включить ПОЛНОЕ содержимое в текстовый ответ между "---"
- Запросить явное подтверждение через AskUserQuestion

При правках — ПОВТОРИТЬ Фазу 1 и Фазу 2.

**После подтверждения:**
1. Создай коммит:
```
Skill(skill="commit", args="docs/project/PIPELINE_PROMPT.md")
```
2. Создай и смёржь PR:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-2-pipeline \
  --title "Phase 2: Pipeline Definition" \
  --body "Определение пайплайна: PIPELINE_PROMPT.md"
gh pr merge --merge --delete-branch
git checkout {MAIN_BRANCH}
```

---

### Фаза 3 — Аналитика

**Git операции:**
```bash
git checkout {MAIN_BRANCH}
git checkout -b phase-3-analytics
```

Выполни через Task tool (последовательно):
- `research-agent` → `docs/project/ANALYSIS.md`
- `system-analyst` → `docs/project/TECH_REQUIREMENTS.md`, `docs/project/SCOPE.md`

**После завершения:**
1. Коммит:
```
Skill(skill="commit", args="docs/project/ANALYSIS.md docs/project/TECH_REQUIREMENTS.md docs/project/SCOPE.md")
```
2. PR и мёрж:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-3-analytics --title "Phase 3: Analytics" --body "Аналитика: ANALYSIS.md, TECH_REQUIREMENTS.md, SCOPE.md"
gh pr merge --merge --delete-branch
git checkout {MAIN_BRANCH}
```

---

### Фаза 4 — Архитектура и декомпозиция

**Git операции:**
```bash
git checkout {MAIN_BRANCH}
git checkout -b phase-4-architecture
```

Выполни через Task tool (последовательно):
- `solution-architect` → `docs/project/ARCHITECTURE_OVERVIEW.md`
- `feature-decomposer` → `docs/project/WORK_BREAKDOWN.md`, `docs/project/FEATURES_INDEX.md`

Каждая фича ОБЯЗАНА иметь поле Domain.

**После завершения:**
1. Коммит:
```
Skill(skill="commit", args="docs/project/ARCHITECTURE_OVERVIEW.md docs/project/WORK_BREAKDOWN.md docs/project/FEATURES_INDEX.md")
```
2. PR и мёрж:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-4-architecture --title "Phase 4: Architecture" --body "Архитектура: ARCHITECTURE_OVERVIEW.md, WORK_BREAKDOWN.md, FEATURES_INDEX.md"
gh pr merge --merge --delete-branch
git checkout {MAIN_BRANCH}
```

---

### Фаза 5 — TDD планирование

**Git операции:**
```bash
git checkout {MAIN_BRANCH}
git checkout -b phase-5-tdd-planning
```

Выполни через Task tool с пакетной обработкой.

**Путь назначения:** `docs/roadmaps/`

**Формат имени файла:** `ROADMAP_<FEATURE>_<NUMBER>.md`

где:
- `<FEATURE>` — ID фичи (например, F001, AUTH-001)
- `<NUMBER>` — номер версии roadmap (01, 02, ...)

**Алгоритм:**

1. Прочитай `docs/project/FEATURES_INDEX.md` и получи список всех фич
2. Разбей фичи на пакеты по **максимум 5 штук**
3. Для каждого пакета запусти `tdd-planner` **параллельно** в одном сообщении:
   - Используй отдельный Task вызов для каждой фичи
   - Максимум 5 параллельных Task вызовов в одном сообщении
4. Дождись завершения всех Task в пакете
5. Повторяй для следующего пакета, пока все фичи не будут обработаны

**Пример запуска пакета:**
```
// Одно сообщение с несколькими Task вызовами:
Task(subagent_type="tdd-planner", prompt="... Feature F-001 ...")
Task(subagent_type="tdd-planner", prompt="... Feature F-002 ...")
Task(subagent_type="tdd-planner", prompt="... Feature F-003 ...")
Task(subagent_type="tdd-planner", prompt="... Feature F-004 ...")
Task(subagent_type="tdd-planner", prompt="... Feature F-005 ...")
```

**Промпт для каждого tdd-planner:**
```
Создай TDD roadmap для фичи:

Feature ID: {feature_id}
Feature Name: {feature_name}
Domain: {feature_domain}

Входные артефакты:
- docs/project/FEATURES_INDEX.md
- docs/project/WORK_BREAKDOWN.md
- docs/project/ARCHITECTURE_OVERVIEW.md
- docs/project/PROJECT_PROFILE.md

Создай ROADMAP_{feature_id}_01.md в директории docs/roadmaps/
в соответствии с твоим process workflow.
```

**После завершения всех пакетов:**
1. Создай коммит:
```
Skill(skill="commit", args="docs/roadmaps/ROADMAP_*.md")
```
2. Создай и смёржь PR:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-5-tdd-planning --title "Phase 5: TDD Planning" --body "TDD roadmaps для всех фич"
gh pr merge --merge --delete-branch
git checkout {MAIN_BRANCH}
```

Выбор профиля ОБЯЗАТЕЛЕН и доменно-ориентирован.

---

### Фаза 6 — Реализация и верификация фич (ДЕТАЛИЗИРОВАНО)

**Git операции (в начале фазы):**
```bash
git checkout {MAIN_BRANCH}
git checkout -b phase-6-implementation
```

**Путь назначения артефактов разработки:** `docs/develop/<FEATURE>/`

**Формат:** для фичи `{feature_id}` все артефакты сохраняются в `docs/develop/{feature_id}/`

**КРИТИЧЕСКИ ВАЖНО:** Каждая фича обрабатывается **ПОЛНОСТЬЮ независимо**
в цикле developer → test → review → verify. НЕ группируй фичи!

**Алгоритм:**

1. Прочитай `docs/project/WORK_BREAKDOWN.md` и `docs/project/FEATURES_INDEX.md`
2. Получи список всех фич, сгруппированных по Stage (версиям)
3. Для КАЖДОЙ фичи (строго по одной) выполни следующий цикл:

```
ЦИКЛ для фичи {feature_id} (повторять пока score < 9):
┌─────────────────────────────────────────────────────────────┐
│ 0. Git: создать ветку feature-{feature_id}                  │
│    git checkout phase-6-implementation                      │
│    git checkout -b feature-{feature_id}                     │
├─────────────────────────────────────────────────────────────┤
│ 1. developer-agent                                           │
│    - subagent_type: "developer-agent"                       │
│    - prompt: "Реализуй фичу {feature_id} согласно           │
│               docs/roadmaps/ROADMAP_{feature_id}_*.md"      │
│    - Выход: docs/develop/{feature_id}/                      │
│            IMPLEMENTATION_REPORT_{feature_id}.md            │
├─────────────────────────────────────────────────────────────┤
│ 2. test-engineer                                             │
│    - subagent_type: "test-engineer"                         │
│    - prompt: "Протестируй реализованную фичу {feature_id}"  │
│    - Выход: docs/develop/{feature_id}/                      │
│            TEST_REPORT_{feature_id}.md                      │
├─────────────────────────────────────────────────────────────┤
│ 3. code-reviewer                                             │
│    - subagent_type: "code-reviewer"                         │
│    - prompt: "Выполни code review для фичи {feature_id}"    │
│    - Выход: docs/develop/{feature_id}/                      │
│            CODE_REVIEW_{feature_id}.md                      │
├─────────────────────────────────────────────────────────────┤
│ 4. feature-verifier                                          │
│    - subagent_type: "feature-verifier"                      │
│    - prompt: "Выполни финальную верификацию фичи {feature_id}" │
│    - Выход: docs/develop/{feature_id}/                      │
│            FEATURE_VERIFICATION_{feature_id}.md             │
│    - Извлеки score из результата                            │
├─────────────────────────────────────────────────────────────┤
│ 5. Коммит и PR (если score ≥ 9):                            │
│    - Skill(skill="commit", args="docs/develop/{feature_id}")│
│    - gh pr create --base phase-6-implementation              │
│        --head feature-{feature_id}                           │
│        --title "Feature {feature_id}"                        │
│    - gh pr merge --merge --delete-branch                    │
│    - git checkout phase-6-implementation                    │
├─────────────────────────────────────────────────────────────┤
│ ПРОВЕРКА:                                                    │
│ - Если score ≥ 9 → фича завершена, переход к следующей      │
│ - Если score < 9 → возврат к п.1 (повтор цикла)             │
└─────────────────────────────────────────────────────────────┘
```

4. После завершения текущей фичи (score ≥ 9) переходи к следующей
5. После завершения ВСЕХ фич текущего Stage переходи к следующему Stage

**Промпт для каждого агента:**

Минимальный контекст — агенты знают свою работу по своим определениям:
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

**Завершение Фазы 6 (после ВСЕХ фич):**
1. Создай финальный коммит для фазы:
```bash
git checkout phase-6-implementation
Skill(skill="commit", args="docs/develop")
```
2. Создай и смёржь PR в основную ветку:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-6-implementation \
  --title "Phase 6: Implementation Complete" \
  --body "Реализация всех фич завершена"
gh pr merge --merge --delete-branch
git checkout {MAIN_BRANCH}
```

**Правила:**
- После успешного завершения фичи (score ≥ 9) создай git commit через Skill tool
- Каждая фича создаёт PR в `phase-6-implementation`
- После всех фич — PR `phase-6-implementation` → `{MAIN_BRANCH}`
- Только после завершения ВСЕХ фич переходи к Фазе 7

---

### Фаза 7 — Системная верификация

**Git операции:**
```bash
git checkout {MAIN_BRANCH}
git checkout -b phase-7-system-verification
```

Выполни через Task tool: `system-verifier`

Выход: `docs/project/SYSTEM_VERIFICATION.md`

Если score < 9:
- возврат к блокирующей стадии

**После успешной верификации (score ≥ 9):**
1. Коммит:
```
Skill(skill="commit", args="docs/project/SYSTEM_VERIFICATION.md")
```
2. PR и мёрж:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-7-system-verification --title "Phase 7: System Verification" --body "Системная верификация пройдена"
gh pr merge --merge --delete-branch
git checkout {MAIN_BRANCH}
```

---

### Фаза 8 — Документация

**Git операции:**
```bash
git checkout {MAIN_BRANCH}
git checkout -b phase-8-documentation
```

Выполни через Task tool: `documentation-agent`

Выход: `docs/project/README.md`, `docs/project/ARCHITECTURE.md`, `docs/project/USAGE.md`

**После завершения:**
1. Коммит:
```
Skill(skill="commit", args="docs/project/README.md docs/project/ARCHITECTURE.md docs/project/USAGE.md")
```
2. PR и мёрж:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-8-documentation --title "Phase 8: Documentation" --body "Финальная документация"
gh pr merge --merge --delete-branch
git checkout {MAIN_BRANCH}
```

---

### Фаза 9 — Релиз

**Git операции:**
```bash
git checkout {MAIN_BRANCH}
git checkout -b phase-9-release
```

Выполни через Task tool: `release-devops`

Выход: `docs/project/DEPLOY.md`, `docs/project/RELEASE_NOTES.md`

**После завершения:**
1. Финальный коммит:
```
Skill(skill="commit", args="docs/project/DEPLOY.md docs/project/RELEASE_NOTES.md")
```
2. Финальный PR:
```bash
gh pr create --base {MAIN_BRANCH} --head phase-9-release --title "Phase 9: Release" --body "Продукт готов к релизу"
gh pr merge --merge --delete-branch
git checkout {MAIN_BRANCH}
```

**🎉 Пайплайн завершён! Продукт создан и задокументирован.**
```
Skill(skill="commit", args="docs/project/DEPLOY.md docs/project/RELEASE_NOTES.md")
```

---

## КАК ЗАПУСКАТЬ АГЕНТОВ

ТЫ запускаешь агентов через Task tool. Паттерн:

```
Task(
    subagent_type="<agent-id>",
    prompt="""
    [ИНСТРУКЦИЯ: сначала прочитай ~/.claude/agents/<agent-id>.md через Read tool]
    [ВКЛЮЧИ ПОЛНОЕ СОДЕРЖИНИЕ ЭТОГО ФАЙЛА В ПРОМПТ]

    ---
    КОНТЕКСТ ДЛЯ ЭТОГО ЗАПУСКА:
    - Human Intent: {...}
    - Previous Artifacts: {...}
    - Expected Output: {...}

    Execute your role according to your instructions above.
    """
)
```

### Соответствие agent-id → файл определения

| Agent ID | Файл определения |
|----------|------------------|
| project-profile-generator | ~/.claude/agents/project-profile-generator.md |
| pipeline-prompt-generator | ~/.claude/agents/pipeline-prompt-generator.md |
| research-agent | ~/.claude/agents/research-agent.md |
| system-analyst | ~/.claude/agents/system-analyst.md |
| solution-architect | ~/.claude/agents/solution-architect.md |
| feature-decomposer | ~/.claude/agents/feature-decomposer.md |
| tdd-planner | ~/.claude/agents/tdd-planner.md |
| developer-agent | ~/.claude/agents/developer-agent.md |
| test-engineer | ~/.claude/agents/test-engineer.md |
| code-reviewer | ~/.claude/agents/code-reviewer.md |
| feature-verifier | ~/.claude/agents/feature-verifier.md |
| system-verifier | ~/.claude/agents/system-verifier.md |
| documentation-agent | ~/.claude/agents/documentation-agent.md |
| release-devops | ~/.claude/agents/release-devops.md |

### Использование Skill tool для git commit

Для создания git commit в ключевых точках пайплайна используй Skill tool:

```
Skill(skill="commit", args="<файлы_для_коммита>")
```

**Где создавать коммиты и PR (после каждой фазы):**

| Фаза | Ветка | Когда | PR в |
|------|-------|-------|------|
| **Фаза 1** | `phase-1-project-profile` | После подтверждения | `{MAIN_BRANCH}` |
| **Фаза 2** | `phase-2-pipeline` | После подтверждения | `{MAIN_BRANCH}` |
| **Фаза 3** | `phase-3-analytics` | После завершения | `{MAIN_BRANCH}` |
| **Фаза 4** | `phase-4-architecture` | После завершения | `{MAIN_BRANCH}` |
| **Фаза 5** | `phase-5-tdd-planning` | После roadmaps | `{MAIN_BRANCH}` |
| **Фаза 6** | `phase-6-implementation` | После всех фич | `{MAIN_BRANCH}` |
| └─ Feature | `feature-{id}` | После score ≥ 9 | `phase-6-implementation` |
| **Фаза 7** | `phase-7-system-verification` | После score ≥ 9 | `{MAIN_BRANCH}` |
| **Фаза 8** | `phase-8-documentation` | После завершения | `{MAIN_BRANCH}` |
| **Фаза 9** | `phase-9-release` | Финальный коммит | `{MAIN_BRANCH}` |

**Примеры команд:**
```bash
# Создание PR
gh pr create --base {MAIN_BRANCH} --head phase-1-project-profile --title "Phase 1: Project Profile"

# Мёрж PR
gh pr merge --merge --delete-branch
```

---

## ПРАВИЛА ПАРАЛЛЕЛИЗАЦИИ

- Фичи МОГУТ разрабатываться параллельно
- Аналитика, архитектура и планирование — строго последовательно
- Общие контракты блокируют параллельность
- Для параллельного запуска отправь ОДНО сообщение с НЕСКОЛЬКИМИ Task вызовами

---

## ЗАПРЕЩЁННЫЕ ДЕЙСТВИЯ

- ❌ Пропуск стадий
- ❌ Обход quality gates
- ❌ Изменение артефактов не своим агентом
- ❌ Использование глобальных / дефолтных профилей
- ❌ Частичная или условная приёмка
- ❌ Симуляция выполнения агента — ВСЕГДА используй Task tool

---

## УСЛОВИЕ СТАРТА

Когда пользователь вызывает `/product-creator`:

1. **Собери контекст:**
   - Цель проекта
   - Краткое описание
   - Платформы
   - Ожидания (качество, TDD, архитектура)
   - Ограничения

2. **Начни Фазу 0** — приём намерения

3. **Следуй пайплайну** — запускай агентов через Task tool в указанном порядке

4. **Запрашивай одобрение** на контрольных точках (PROJECT_PROFILE_HUMAN, PIPELINE_PROMPT)

---

## Выходные артефакты

После успешного выполнения пайплайна будет создана следующая структура:

```
docs/
├── project/                    # Артефакты уровня проекта
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
├── roadmaps/                   # TDD roadmaps для каждой фичи
│   ├── ROADMAP_<FEATURE>_01.md
│   ├── ROADMAP_<FEATURE>_02.md
│   └── ...
│
└── develop/                    # Артефакты разработки каждой фичи
    ├── <FEATURE_01>/
    │   ├── IMPLEMENTATION_REPORT_<FEATURE_01>.md
    │   ├── TEST_REPORT_<FEATURE_01>.md
    │   ├── CODE_REVIEW_<FEATURE_01>.md
    │   └── FEATURE_VERIFICATION_<FEATURE_01>.md
    └── ...
```

**Реализация (код):**
- Исходный код (backend, mobile, web)
- Тесты (unit, integration, e2e)
