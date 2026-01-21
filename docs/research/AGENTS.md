## 3. Рекомендуемый состав субагентов

### 🧠 Уровень 0 — Оркестрация

#### **Pipeline Orchestrator**

**Роль:** дирижер всего процесса
**Ответственность:**

* запускает этапы
* передает артефакты между агентами
* останавливает пайплайн для human review

---

### 🔍 Этап 1 — Аналитика

#### **Research Agent**

* сбор информации
* анализ аналогов
* выявление рисков

#### **System Analyst Agent**

* формирует ТЗ
* описывает ограничения
* формирует acceptance criteria

📄 Результат:

* `ANALYSIS.md`
* `TECH_REQUIREMENTS.md`

---

### 🧩 Этап 2 — Декомпозиция

#### **Solution Architect Agent**

* проектирует архитектуру
* разбивает систему на стадии

#### **Feature Decomposition Agent**

* выделяет фичи
* описывает зависимости

📄 Результат:

* `WORK_BREAKDOWN.md`
* `FEATURES_INDEX.md`

---

### 🗺 Этап 3 — Роадмапы (TDD)

#### **TDD Planner Agent**

* создает роадмапы в формате:

    * tests first
    * затем реализация
    * затем acceptance

📄 Результат:

* `ROADMAP_feature_X.md`

---

### 🧑‍💻 Этап 4 — Разработка (циклическая)

#### **Developer Agent (per feature)**

* реализует фичу строго по роадмапу

#### **Test Engineer Agent**

* проверяет TDD
* полноту тестов
* негативные кейсы

#### **Code Reviewer Agent**

* архитектура
* стиль
* best practices

#### **Feature Verifier Agent**

* соответствие ТЗ
* UI / UX (если есть)
* выставляет оценку 0–10

🔁 Цикл:

```
Develop → Review → Verify → Fix → Repeat (до ≥9)
```

📄 Результат:

* `IMPLEMENTATION_REPORT_feature_X.md`

---

### 🧬 Этап 5 — Интеграция и целостность

#### **System Verifier Agent**

* проверяет, что все фичи:

    * работают вместе
    * не конфликтуют
    * соответствуют исходному ТЗ

#### **Documentation Agent**

* собирает:

    * README
    * architecture overview
    * usage docs

📄 Результат:

* `README.md`
* `SYSTEM_VERIFICATION.md`

---

### 🚀 Этап 6 — Деплой (опционально)

#### **DevOps / Release Agent**

* инструкции по деплою
* тестовый прогон
* rollback сценарии

📄 Результат:

* `DEPLOY.md`
* `RELEASE_NOTES.md`

