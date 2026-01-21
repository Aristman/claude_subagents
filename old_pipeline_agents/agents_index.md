# AGENTS INDEX

Этот документ является **единым реестром агентов**, используемых в feature-driven системе разработки на базе Claude CLI.

Он предназначен **для людей**, а не для оркестрации, и служит:
- навигацией по агентам
- описанием их ролей и границ ответственности
- фиксацией места каждого агента в общем процессе

---

## 🧭 Общая схема процесса

```
Идея / Запрос
   ↓
product-analyst
   ↓
feature-analyst
   ↓
feature-splitter (optional)
   ↓
reviewer (feature scope)
   ↓
system-architect
   ↓
backend-architect
   ↓
mobile-architect
   ↓
tech-lead
   ↓
FOR EACH FEATURE:
   ├─ backend-dev
   ├─ mobile-dev
   ├─ test-engineer
   ├─ feature-verifier
   │     └─ feedback loop if needed
   ↓
system-verifier
   ↓
quality-gate
```

---

## 🧩 Product & Feature Layer

### product-analyst
**Роль:** Продуктовая аналитика и формирование PRD  
**Отвечает за:** бизнес-цели, требования, ограничения  
**НЕ делает:** архитектуру, код, фич-декомпозицию  
**Вход:** идея, бизнес-запрос  
**Выход:** PRD

---

### feature-analyst
**Роль:** Формирование и нормализация фич  
**Отвечает за:** immutable Feature Set  
**НЕ делает:** архитектуру, реализацию  
**Вход:** PRD  
**Выход:** Feature Definitions

---

### feature-splitter (optional)
**Роль:** Дробление oversized / complex фич  
**Отвечает за:** атомарность и тестируемость фич  
**Вход:** Feature Definitions (OVERSIZED)  
**Выход:** Refined Feature Set

---

## 🏗 Architecture Layer

### system-architect
**Роль:** Системная архитектура под фичи  
**Отвечает за:** контейнеры, границы, ADR  
**НЕ делает:** изменение фич  
**Вход:** PRD, Feature Set  
**Выход:** System Architecture

---

### backend-architect
**Роль:** Backend-архитектура под фичи  
**Отвечает за:** сервисы, API, data models  
**НЕ делает:** код, изменение фич  
**Вход:** System Architecture, Feature Set  
**Выход:** Backend Architecture

---

### mobile-architect
**Роль:** Mobile-архитектура под фичи  
**Отвечает за:** навигацию, state, UI flows  
**НЕ делает:** код, изменение фич  
**Вход:** Backend Architecture, Feature Set  
**Выход:** Mobile Architecture

---

## ⚙️ Technical Governance Layer

### tech-lead
**Роль:** Техническое руководство и планирование  
**Отвечает за:** стек, стандарты, DoD, план по фичам  
**НЕ делает:** архитектуру, код  
**Вход:** Architecture Docs, Feature Set  
**Выход:** Feature-based Implementation Plan

---

## 💻 Implementation Layer

### backend-dev
**Роль:** Реализация backend-кода по фичам  
**Отвечает за:** код backend для одной фичи  
**НЕ делает:** архитектуру, тест-планирование  
**Вход:** Backend Architecture, Feature Tasks  
**Выход:** Backend Code

---

### mobile-dev
**Роль:** Реализация mobile-кода по фичам  
**Отвечает за:** UI и client-logic одной фичи  
**НЕ делает:** архитектуру, тест-планирование  
**Вход:** Mobile Architecture, Feature Tasks  
**Выход:** Mobile Code

---

### test-engineer
**Роль:** Автоматизированные тесты по фичам  
**Отвечает за:** тесты, покрытие, регрессии  
**НЕ делает:** бизнес-логику  
**Вход:** Feature Definitions, Code  
**Выход:** Test Suite

---

## 🔍 Feature Quality Layer

### feature-verifier
**Роль:** Глубокая верификация одной фичи  
**Отвечает за:** качество, работоспособность, соответствие архитектуре  
**Формат:** отчёт с оценками  
**Вход:** Feature, Code, Tests  
**Выход:** Feature Verification Report

---

## 🧪 System Quality Layer

### system-verifier
**Роль:** Системная верификация после всех фич  
**Отвечает за:** интеграцию, архитектурную целостность, готовность  
**Формат:** системный отчёт  
**Вход:** Все фичи + отчёты верификации  
**Выход:** System Verification Report

---

## 🚦 Final Control Layer

### reviewer
**Роль:** Независимое ревью артефактов  
**Отвечает за:** соответствие правилам и контрактам  
**НЕ делает:** правки  
**Вход:** любые артефакты  
**Выход:** Review Notes

---

### feedback-synthesizer
**Роль:** Агрегация обратной связи  
**Отвечает за:** actionable feedback  
**Вход:** review / verifier reports  
**Выход:** Consolidated Feedback

---

### quality-gate
**Роль:** Финальное решение  
**Отвечает за:** APPROVE / REJECT  
**Вход:** все отчёты  
**Выход:** Release Decision

---

## 📌 Примечания

- Фичи считаются **immutable** после feature-analyst / feature-splitter
- Архитектура всегда подчинена фичам
- Каждая фича имеет собственный цикл качества
- Система проверяется только после проверки всех фич

---

**AGENTS_INDEX.md — это точка истины для всей агентной системы.**

