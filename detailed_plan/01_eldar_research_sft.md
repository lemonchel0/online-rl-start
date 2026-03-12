# Эльдар — Research: SFT + Distill Pipeline + PRM + Annotation

**Состав:** 3 человека (Э-1, Э-2, Э-3)
**Scope:** SFT reasoning + multi-teacher distill pipeline, Process Reward Model, SFT annotation по reward directions.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | **Distill pipeline** (Э-1), SFT 20k + annotation v0 (Э-2), PRM: ProcessBench + start protocol 3c (Э-3) | Pipeline v0 на 100 задачах; >= 20k SFT annotated; ProcessBench accuracy | Pipeline demo; SFT table; ProcessBench vs competitors | - [ ] |
| [W2](#w2) | Pipeline → production (Э-1), SFT +30k annotated (Э-2), PRM prompt eng + test set (Э-3) | Pipeline beats single-teacher; >= 50k annotated; PRM test set | **Quality audit: pipeline vs single-teacher** | - [ ] |
| [W3](#w3) | SFT masking ablation 4 стратегии (Э-1+2), PRM cross-val + train gen (Э-3) | Best masking; >= 80k annotated; PRM cross-val | **Masking: strategy x benchmark table** | - [ ] |
| [W4](#w4) | SFT final 100-200k annotated (Э-1), Distill comparison (Э-2), PRM 0.6B LoRA (Э-3) | **GATE: SFT beats distill**; PRM LoRA trained | **SFT vs distill gate: PASS/FAIL** | - [ ] |
| [W5](#w5) | PRM ablation: outcome vs process (Э-1+2), SFT promotion check (Э-3) | EXP-PRM-3 results; SFT sign-off | PRM impact на AIME | - [ ] |
| [W6](#w6) | PRM strategy: partial credit vs veto (Э-1), regression analysis (Э-2+3) | PRM best strategy; regression clean | **PRM final + regression** | - [ ] |
| [W7](#w7) | ProcessBench/PRMBench final (Э-1), math gate (Э-2), cross-domain (Э-3) | math_500 + AIME targets | **Math gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final math gate sign-off | Все math метрики GREEN | Sign-off | - [ ] |

**Ключевые риски:**
- Distill pipeline не готов к W2 → SFT W2 на single-teacher data, re-generate позже
- SFT не бьёт distill → STOP, добавить данные, пересмотреть masking
- PRM не улучшает AIME → outcome-only, не блокирует release

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17-21)

<a id="э1-w1"></a>

**Эльдар-1: Multi-teacher distill pipeline (BLOCKING)**
- Вход: API Qwen-3, Gemini-Pro-3.5-preview, DeepSeek-V3.2.
- Задача: Pipeline: (1) генерация из 3+ учителей, (2) cross-verification (keep если >= 2 согласны), (3) Best-of-N (скоринг OS rewards), (4) rejection sampling (execution/format), (5) quality ranking (pair-wise).
- DoD: Pipeline script ready. На 100 задачах measurable quality gain vs single-teacher.
- Day 1-2: API wrappers, generation module. Day 3: cross-verification + rejection. Day 4: Best-of-N + ranking. Day 5: end-to-end test, quality audit.
- Риск: API latency → batch, cache, 2 учителя если 3-й недоступен.

<a id="э2-w1"></a>

**Эльдар-2: SFT data collection из opensource + annotation v0**
- Вход: MATH, GSM8K, OpenCodeInstruct, ODA-Mixture, IFEval seeds, WildChat RU.
- Задача: >= 20k SFT (10k math+code, 5k IF, 5k B2C/STEM). Annotation v0: (1) MD check (determ), (2) logic/correctness (execution/answer match), (3) Arena preference (LLM-as-judge). RU quality — с W2+.
- DoD: >= 20k annotated, distribution table source x domain x count.
- Риск: LLM-as-judge дорогой → batch 1k/день.

<a id="э3-w1"></a>

**Эльдар-3: PRM — ProcessBench baseline + annotation start (protocol 3c step 1-2)**
- Вход: ProcessBench (3400 задач), 10b_toolmind.
- Задача: (1) ProcessBench accuracy. (2) Comparison table vs competitors. (3) PRM annotation: промпт → пилот 200 → IAA.
- DoD: Accuracy зафиксирована, comparison table, PRM prompt draft, пилот 200.
- Риск: IAA < 0.8 → упростить промпт, binary step labels.

---

<a id="w2"></a>

### W2 (Mar 24-28)

<a id="э1-w2"></a>

**Эльдар-1: Distill pipeline → production quality**
- Задача: Оптимизировать pipeline (throughput, cost). Generate first batch 5k+ через pipeline.
- DoD: Pipeline production-ready. Quality audit: pipeline vs single-teacher на 100 samples.

<a id="э2-w2"></a>

**Эльдар-2: SFT +30k с full annotation**
- Задача: +30k SFT (expanding domains). Full annotation: MD + logic + Arena + RU (when Русификатор ready).
- DoD: >= 50k annotated total. EXP-SFT-1 (baseline distill Qwen-3) launched.

<a id="э3-w2"></a>

**Эльдар-3: PRM prompt engineering + test set (protocol 3c step 2-3)**
- Задача: Итерация промпта на pilot данных. Test set >= 500 примеров.
- DoD: PRM test set 500+ ready, prompt finalized.

---

<a id="w3"></a>

### W3 (Mar 31 - Apr 4)

<a id="э1-w3"></a>

**Эльдар-1+2: SFT masking ablation (4 strategies)**
- Задача: EXP-SFT-5. Тренировка 4 вариантов: full masking, negative masking, contrast pairs, no masking. Benchmark каждый.
- DoD: Лучшая стратегия выбрана. SFT >= 80k annotated.

<a id="э2-w3"></a>

*(Э-2 работает в паре с Э-1 на ablation)*

<a id="э3-w3"></a>

**Эльдар-3: PRM cross-validation + train generation (protocol 3c step 3-4)**
- Задача: EXP-PRM-1. Cross-val PRM annotations. Start generating train data.
- DoD: Cross-val done, train generation started.

---

<a id="w4"></a>

### W4 (Apr 7-11)

<a id="э1-w4"></a>

**Эльдар-1: SFT final 100-200k annotated**
- Задача: Финализировать SFT dataset 100-200k, все annotated (MD, RU, logic, Arena).
- DoD: Dataset ready, all annotated. EXP-SFT-4 (наш подход masking).

<a id="э2-w4"></a>

**Эльдар-2: EXP-SFT-2/3/4 distill comparison**
- Задача: Сравнить наш SFT vs distill Qwen-3/3.5/GLM-5.
- DoD: **GATE: SFT бьёт distill. PASS/FAIL.**

<a id="э3-w4"></a>

**Эльдар-3: PRM 0.6B LoRA training (protocol 3c step 5-6a)**
- Задача: Train PRM LoRA на собранных данных. Test measurement.
- DoD: PRM LoRA trained, accuracy on test documented.

---

<a id="w5"></a>

### W5 (Apr 14-18)

<a id="э1-w5"></a>

**Эльдар-1+2: PRM ablation EXP-PRM-3**
- Задача: Outcome-only vs outcome+process reward. Measure AIME delta.
- DoD: EXP-PRM-3 results documented.

<a id="э3-w5"></a>

**Эльдар-3: SFT promotion check**
- Задача: Final SFT sign-off, verify no regressions.
- DoD: SFT sign-off document.

---

<a id="w6"></a>

### W6 (Apr 21-25)

<a id="э1-w6"></a>

**Эльдар-1: PRM strategy EXP-PRM-4**
- Задача: Partial credit vs veto vs decouple. Choose best.
- DoD: PRM best strategy chosen.

**Эльдар-2+3: Cross-domain regression analysis**
- Задача: Verify no cross-domain regressions from PRM/RL.
- DoD: Regression matrix clean.

---

<a id="w7"></a>

### W7 (Apr 28 - May 2)

**Эльдар-1: ProcessBench/PRMBench final numbers**
**Эльдар-2: Math gate check (math_500, AIME)**
**Эльдар-3: Cross-domain final check**
- DoD: **Math gate: PASS/FAIL.**

---

<a id="w8"></a>

### W8 (May 5-9)

**Final math gate sign-off, release readiness.**
- DoD: Все math метрики GREEN. Sign-off document.

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-DISTILL-1 | W1-2 | Э-1 | Multi-teacher pipeline quality vs single-teacher |
| EXP-SFT-1 | W2 | Э-2 | Baseline distill Qwen-3 |
| EXP-SFT-2 | W4 | Э-2 | Distill Qwen-3.5 |
| EXP-SFT-3 | W4 | Э-2 | Distill GLM-5 |
| EXP-SFT-4 | W4 | Э-1 | Наш подход (masking) |
| EXP-SFT-5 | W3 | Э-1+2 | Masking ablation a/b/c/d |
| EXP-PRM-1 | W3 | Э-3 | Baseline step-error detection + cross-val |
| EXP-PRM-2 | W4 | Э-3 | Train PRM 0.6B LoRA |
| EXP-PRM-3 | W5 | Э-1+2 | Outcome vs outcome+process |
| EXP-PRM-4 | W6 | Э-1 | Partial credit vs veto vs decouple |
