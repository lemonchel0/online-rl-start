# Вова — Functions: BFCL v4 baseline + ToolRM deploy

**Состав:** 2 человека (В-1, В-2)
**Scope:** BFCL v4 baseline на всех моделях, ToolRM-1.7B deploy, первый error analysis.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | **BFCL v4 baseline** + ToolRM online (В-1), tool specs verify (В-2) | BFCL score 10b; ToolRM online + latency | **BFCL: нас vs Qwen3.5-2B** | - [ ] |
| [W2](#w2) | **BFCL vs competitors table** (В-1), ToolRM error analysis (В-2) | Все 5 моделей на BFCL; top-5 error types | **Competitors BFCL table** | - [ ] |

**Ключевые риски:**
- ToolRM-1.7B недостаточно точный на нашем распределении → задокументировать + fallback AST-based
- Веса конкурентов для BFCL недоступны → API-режим, если есть; скачать заранее Day 1
- BFCL v4 infra → проверить W1 Day 1 (SWE-bench infra — тот же риск с Вадимом)

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17–21)

<a id="в1-w1"></a>

**Вова-1: BFCL v4 baseline + ToolRM-1.7B deploy**
- Вход: BFCL v4 eval framework, 10b_toolmind, ToolRM-1.7B.
- Задача:
  - Day 1: Проверить BFCL v4 infra. Если не работает → эскалировать сразу.
  - Day 1-2: Развернуть ToolRM-1.7B (vLLM), проверить latency.
  - Day 2-3: Запустить BFCL v4 на 10b_toolmind → baseline score.
  - Day 3-4: Запустить BFCL v4 на Qwen3.5-2B → comparison.
  - Day 5: Оформить таблицу, задокументировать.
- DoD:
  - BFCL v4 overall + multi-turn score для 10b_toolmind.
  - BFCL v4 score для Qwen3.5-2B.
  - ToolRM-1.7B online, latency < 500ms p95.
  - Comparison table: 10b vs Qwen3.5-2B (BFCL).
- Риск: BFCL infra → Day 1 check, если сломано — 2ч на debug, иначе эскалировать.

**Метрики BFCL v4 для фиксации:**

| Метрика | 10b_toolmind | Qwen3.5-2B |
|---------|-------------|------------|
| Overall accuracy | ? | ? |
| Simple function | ? | ? |
| Multiple function | ? | ? |
| Parallel function | ? | ? |
| Nested function | ? | ? |
| Multi-turn | ? | ? |
| Irrelevance detection | ? | ? |

<a id="в2-w1"></a>

**Вова-2: Tool specs verify + SWE baseline check**
- Задача:
  - Проверить наличие 50+ tool specs в JSON формате.
  - Запустить SWE-bench baseline на 10b_toolmind (если infra доступна).
- DoD: 50 specs в формате. SWE mean@1 с 10b_toolmind.
- Риск: SWE-bench infra (Вадим) → проверить Day 1; если нет — перенести на W2.

---

<a id="w2"></a>

### W2 (Mar 24–28)

<a id="в1-w2"></a>

**Вова-1: BFCL vs competitors финальная таблица**
- Задача:
  - Запустить BFCL v4 на всех 5 моделях: 10b_toolmind, Qwen3.5-2B, Qwen3.5-4B, GLM-5, DeepSeek V3.2.
  - Сформировать полную comparison table.
  - Анализ: где мы проигрываем? Топ-5 error типов по категориям.
- DoD: **Полная таблица BFCL v4 по всем 5 моделям.** Top-5 error types.

**Финальная таблица (шаблон):**

| Метрика | 10b_toolmind | Qwen3.5-2B | Qwen3.5-4B | GLM-5 | DeepSeek V3.2 |
|---------|-------------|------------|------------|-------|---------------|
| Overall | ? | ? | ? | ? | ? |
| Multi-turn | ? | ? | ? | ? | ? |
| Parallel | ? | ? | ? | ? | ? |
| Irrelevance | ? | ? | ? | ? | ? |

<a id="в2-w2"></a>

**Вова-2: ToolRM error analysis**
- Задача: Запустить ToolRM-1.7B на BFCL v4 → error distribution. Топ-5 категорий ошибок.
- DoD: Error distribution таблица. Рекомендация: ToolRM работает / нужен fallback AST-based.

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-BFCL-BASE | W1-2 | В-1 | BFCL v4: 10b_toolmind vs Qwen3.5-2B/4B/GLM-5/DeepSeek |
| EXP-TOOLRM-VERIFY | W2 | В-2 | ToolRM-1.7B on BFCL: P/R + error analysis |
