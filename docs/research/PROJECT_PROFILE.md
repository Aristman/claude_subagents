# PROJECT_PROFILE.md

## Назначение документа

`PROJECT_PROFILE.md` описывает **профиль проекта**, который определяет,
**как именно** агентный пайплайн должен выполнять работу над данным проектом.

Документ является:
- обязательным входным артефактом пайплайна
- источником истины для execution-агентов
- контрактом между человеком, оркестратором и агентами

Без корректного `PROJECT_PROFILE.md` запуск execution-агентов **запрещён**.

---

## Роль PROJECT_PROFILE.md в пайплайне

PROJECT_PROFILE.md:
- связывает **универсальные роли агентов** с **конкретными профилями**
- задаёт технологический и платформенный контекст
- исключает неявные допущения при реализации

Документ **не описывает требования к продукту**  
(они находятся в `TECH_REQUIREMENTS.md`).

---

## Обязательная структура документа

Документ состоит из **строго определённых разделов**.
Отсутствие любого обязательного раздела делает профиль невалидным.

---

## 1. Общая информация о проекте

```yaml
project:
  name: "<project_name>"
  description: "<short_description>"
  type: "<project_type>"
````

### Поля

* `name` — человекочитаемое имя проекта
* `description` — краткое описание назначения проекта
* `type` — тип проекта (backend, web, mobile, multiplatform, cli)

---

## 2. Профили для ролей агентов

```yaml
profiles:
  developer: "<profile_id>"
  tester: "<profile_id>"
  reviewer: "<profile_id>"
  verifier: "<profile_id>"
  release: "<profile_id>"
```

### Правила

* каждый execution-агент **обязан иметь профиль**
* профиль должен существовать в `AGENT_PROFILE_<profile>.md`
* разные роли **могут** использовать разные профили
* если профиль не поддерживается агентом — агент обязан отказать в работе

---

## 3. Технологический стек

```yaml
stack:
  language: "<language>"
  framework: "<framework>"
  runtime: "<runtime_optional>"
  database: "<database_optional>"
  additional:
    - "<tool_or_library>"
```

### Назначение

* задаёт контекст реализации
* используется developer / reviewer / release агентами
* не заменяет архитектурные решения, а ограничивает их

---

## 4. Платформенные ограничения

```yaml
platform_constraints:
  os:
    - "<os_name>"
  architecture:
    - "<arch>"
  environment:
    - "<env_type>"
```

### Примеры

* mobile → iOS / Android версии
* backend → Linux / containerized
* web → browser support matrix

---

## 5. Нефункциональные приоритеты

```yaml
non_functional_priorities:
  performance: high | medium | low
  security: high | medium | low
  reliability: high | medium | low
  scalability: high | medium | low
  maintainability: high | medium | low
```

### Используется для

* архитектурных решений
* code review
* stage-gate проверок

---

## 6. Ограничения и допущения

```yaml
constraints:
  - "<hard_constraint>"

assumptions:
  - "<assumption>"
```

### Примечание

* `constraints` — жёсткие ограничения (нарушать нельзя)
* `assumptions` — допущения, которые можно пересматривать

---

## 7. Поддерживаемые окружения

```yaml
environments:
  development: true
  staging: false
  production: true
```

Используется:

* test-engineer агентом
* release агентом

---

## 8. Валидация профиля проекта

PROJECT_PROFILE.md считается **валидным**, если:

* указан `project.type`
* все execution-роли имеют профиль
* все профили существуют
* стек определён
* документ синтаксически корректен

Невалидный профиль **блокирует пайплайн**.

---

## 9. Связанные документы

* AGENTS_INDEX.md
* ARTIFACTS_INDEX.md
* AGENT_PROFILE_<profile>.md
* TECH_REQUIREMENTS.md
* PIPELINE_PROMPT.md

---

## Статус документа

Статус: Production-ready
Версия: 1.0
Назначение: Project-level profile specification
