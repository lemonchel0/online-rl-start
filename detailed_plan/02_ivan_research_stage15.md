# Иван — Research: Stage 1.5 + IFEval

**Состав:** 3 человека (И-1, И-2, И-3)
**Scope:** Stage 1.5 reasoning (Solve First), IFEval, всё instruction following.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Solve First: math 200k (И-1), code 100k (И-2), dedup + **leak check** (И-3) | >= 300k; dedup отчёт; **leak report** | Overlap %, clean dataset, leak status | - [ ] |
| [W2](#w2) | Code +200k, IF/Arena +100k (И-1+2), contamination fixes (И-3) | >= 600k; zero overlap | **Stage 1.5: 600k** | - [ ] |
| [W3](#w3) | STEM/B2C/RU +400k (И-1+2), IFEval reward setup (И-3) | >= 1M; IFEval P/R | **Stage 1.5: 1M** | - [ ] |
| [W4](#w4) | Finalization 2M (И-1+2), IFEval+Vikacheck, dynamic sampling (И-3) | **2M target** | **GATE: 2M done** | - [ ] |
| [W5](#w5) | Domain mix ablation (И-1), IFEval ablation (И-2), difficulty sampler (И-3) | Optimal mix; IFEval delta | **Domain mix: best proportions** | - [ ] |
| [W6](#w6) | Mixed sampler (И-1), dynamic sampling (И-2), OBM format 50 tasks (И-3) | OBM >= 90% | OBM gate check | - [ ] |
| [W7](#w7) | IFEval gate (И-1), Vikacheck gate (И-2), thresholds (И-3) | IFEval >= target | **IF gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final IF gate sign-off | Все IF метрики GREEN | Sign-off | - [ ] |

**Ключевые риски:**
- Solve First throughput < 100k/день → параллелить GPU, synthetic fallback
- Stage 1.5 quality drift → weekly audit 100 random per domain

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17-21)

<a id="и1-w1"></a>

**Иван-1: Solve First — math 200k**
- Вход: Solve First pipeline, teacher Qwen-3, seeds: MATH 12k + GSM8K 8k + AIME + Olympiad.
- DoD: >= 200k math, quality audit 100 random (прямой reasoning, нет Wait), файл в хранилище.
- Риск: throughput → параллелить GPU.

<a id="и2-w1"></a>

**Иван-2: Solve First — code 100k**
- Вход: HumanEval 164 + MBPP 500 + LCB seeds + competitive programming.
- DoD: >= 100k code, execution validation 50 random.
- Риск: solve rate ниже (30-40%) → easier tasks first.

<a id="и3-w1"></a>

**Иван-3: Dedup/contamination + LEAK CHECK**
- Вход: Все rollout queries + eval benchmarks + reward sets.
- Задача: Exact + fuzzy dedup (MinHash + LSH, threshold 0.85). **Leak check: eval data в train.**
- DoD: Отчёт overlap %, leak status, sample IDs, скрипт.

---

<a id="w2"></a>

### W2 (Mar 24-28)

<a id="и1-w2"></a>

**Иван-1+2: Solve First code +200k, IF/Arena/Safety +100k**
- DoD: >= 600k total; quality audit.

<a id="и3-w2"></a>

**Иван-3: Contamination fixes**
- DoD: Zero overlap с eval verified.

---

<a id="w3"></a>

### W3 (Mar 31 - Apr 4)

<a id="и1-w3"></a>

**Иван-1+2: STEM/B2C/RU expansion +400k**
- DoD: >= 1M total.

<a id="и3-w3"></a>

**Иван-3: IFEval reward setup**
- DoD: IFEval checker P/R measured.

---

<a id="w4"></a>

### W4 (Apr 7-11)

<a id="и1-w4"></a>

**Иван-1+2: Stage 1.5 finalization 2M**
- DoD: **2M target GATE.**

<a id="и3-w4"></a>

**Иван-3: IFEval+Vikacheck integration, dynamic sampling proto**
- DoD: Dynamic sampling works.

---

<a id="w5"></a>

### W5-W8

**W5:** Domain mix ablation (И-1), IFEval + IF reward ablation (И-2), difficulty sampler (И-3).
**W6:** Mixed sampler (И-1), dynamic sampling (И-2), OBM format 50 tasks (И-3).
**W7:** IFEval gate (И-1), Vikacheck gate (И-2), promotion thresholds (И-3).
**W8:** Final IF gate sign-off.

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-LEAK-1 | W1 | И-3 | Eval overlap check in train data |
