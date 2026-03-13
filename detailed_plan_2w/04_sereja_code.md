# Серёжа — Code: LCB v6 baseline + sandbox + code reward P/R

**Состав:** 2 человека (С-1, С-2)
**Scope:** LCB v6 baseline на всех моделях, sandbox для code execution, code reward pipeline P/R.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | **LCB v6 + sandbox** (С-1), code queries 10k+ (С-2) | Sandbox works; LCB pass@1 10b; 10k queries | **LCB: нас vs Qwen3.5-2B** | - [ ] |
| [W2](#w2) | **LCB vs competitors table** (С-1), code reward P/R (С-2) | Все 5 моделей на LCB; **reward P/R** | **LCB competitors + code reward baseline** | - [ ] |

**Ключевые риски:**
- Sandbox security → gVisor/Firecracker, не docker без isolation
- Код конкурентов: LCB timeout → увеличить timeout для 120B-type моделей
- Code reward FP rate высокий → execution-first, judge-fallback только при timeout

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17–21)

<a id="с1-w1"></a>

**Серёжа-1: LCB v6 + sandbox setup + baseline**
- Вход: LCB v6 evaluation framework, 10b_toolmind, sandbox config.
- Задача:
  - Day 1: Sandbox (gVisor/Firecracker) — проверить изоляцию, latency < 5s p95.
  - Day 2-3: LCB v6 на 10b_toolmind → pass@1.
  - Day 3-4: LCB v6 на Qwen3.5-2B → comparison.
  - Day 5: Таблица + документация.
- DoD:
  - Sandbox works + изоляция подтверждена.
  - LCB v6 pass@1 для 10b_toolmind.
  - LCB v6 pass@1 для Qwen3.5-2B.
  - Comparison table: 10b vs Qwen3.5-2B.
- Риск: sandbox latency > 10s → упростить eval на W1, production sandbox W2.

**Метрики LCB v6 для фиксации:**

| Метрика | 10b_toolmind | Qwen3.5-2B |
|---------|-------------|------------|
| pass@1 overall | ? | ? |
| Easy | ? | ? |
| Medium | ? | ? |
| Hard | ? | ? |
| Contest | ? | ? |

<a id="с2-w1"></a>

**Серёжа-2: Code queries 10k+ с метками**
- Вход: HumanEval 164 + MBPP 500 + LCB seeds + competitive programming.
- Задача: Собрать 10k+ code queries. Разметить: domain + difficulty + context_length.
  - difficulty: easy (HumanEval level), medium (MBPP / mid-LCB), hard (contest / hard-LCB).
  - context_length: short (<512 tokens), medium (512-2k), long (2k+).
- DoD: 10k+ queries с метками domain/difficulty/context_length в хранилище.
- Примечание: Метки нужны для online RL сэмплирования (требование из разговора).

---

<a id="w2"></a>

### W2 (Mar 24–28)

<a id="с1-w2"></a>

**Серёжа-1: LCB v6 vs competitors финальная таблица**
- Задача:
  - Запустить LCB v6 на всех 5 моделях: 10b_toolmind, Qwen3.5-2B, Qwen3.5-4B, GLM-5, DeepSeek V3.2.
  - Сформировать полную comparison table.
  - Анализ: где проигрываем, top error categories.
- DoD: **Полная таблица LCB v6 по всем 5 моделям.**

**Финальная таблица (шаблон):**

| Метрика | 10b_toolmind | Qwen3.5-2B | Qwen3.5-4B | GLM-5 | DeepSeek V3.2 |
|---------|-------------|------------|------------|-------|---------------|
| pass@1 overall | ? | ? | ? | ? | ? |
| Easy | ? | ? | ? | ? | ? |
| Medium | ? | ? | ? | ? | ? |
| Hard | ? | ? | ? | ? | ? |

<a id="с2-w2"></a>

**Серёжа-2: Code reward pipeline P/R**
- Задача:
  - Построить code reward pipeline: sandbox execution + LLM-judge fallback.
  - Замерить P/R на labeled set из 200+ примеров.
- DoD: Code reward P/R таблица. Рекомендация: использовать в RL / нужна доработка.

**Code reward pipeline:**
```
input: (prompt, response)
  ├─ step 1: extract code block
  ├─ step 2: sandbox execution → passed/failed/timeout
  │   ├─ passed  → reward = 1.0
  │   ├─ failed  → reward = 0.0
  │   └─ timeout → step 3: LLM-judge
  └─ step 3: LLM-judge → reward = 0.0..1.0
output: reward score + reason
```

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-LCB-BASE | W1-2 | С-1 | LCB v6: 10b_toolmind vs Qwen3.5-2B/4B/GLM-5/DeepSeek |
| EXP-CODE-REWARD | W2 | С-2 | Code reward pipeline P/R на labeled set |
