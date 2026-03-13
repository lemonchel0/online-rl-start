# 2-недельный baseline sprint (Mar 17–28, 2026)

> Последнее обновление: 2026-03-13
> Цель: супер крутой baseline на опенсорсных моделях + baseline по каждому reward.
> Связанные документы: [gap analysis](../online_rl_ml_data_gap_analysis.md), [8-недельный план](../detailed_plan/00_overview.md).

---

## Файлы команд

| Файл | Команда | Лид | Человек |
|------|---------|-----|---------|
| [01_eldar_research_sft.md](01_eldar_research_sft.md) | Research: SFT + Distill + PRM baseline | Эльдар | 3 |
| [02_ivan_research_stage15.md](02_ivan_research_stage15.md) | Research: Stage 1.5 данные, Leak check, OBM format | Иван | 3 |
| [03_vova_functions.md](03_vova_functions.md) | Functions: BFCL v4 baseline, ToolRM deploy | Вова | 2 |
| [04_sereja_code.md](04_sereja_code.md) | Code: LCB v6 baseline, sandbox, code reward P/R | Серёжа | 2 |
| [05_polya_product.md](05_polya_product.md) | Product: Reward test sets в ОБМ, OS reward verification | Поля | 6 |
| [06_dasha_platform.md](06_dasha_platform.md) | Platform: B2C reward baselines (interpreter, search, t2i) | Даша | 4 |
| [07_dima_infra.md](07_dima_infra.md) | Infra: Storage, OS rewards x5, determ checks, OS reward matrix | Дима | 3 |
| [08_ml.md](08_ml.md) | ML: Eval sweep pipeline, baseline по всем трекам, OS reward verification | ML | 7+ |

---

## Цели спринта

### Цель 1: ОБМ baseline table

Замерить все capability-треки на opesource-моделях и нашей текущей 10b_toolmind.

**Модели для сравнения:**
- 10b_toolmind (наша текущая)
- Qwen3.5-2B (ближайший по active params конкурент)
- Qwen3.5-4B (stretch target)
- GLM-5
- DeepSeek V3.2

**Треки:**

| Трек | Бенчи | Owner |
|------|-------|-------|
| Math | MATH-500, AIME, GSM8K | ML |
| Code | LCB v6, SWE-bench | Серёжа |
| IF | IFEval strict/loose, Vikacheck | ML + Иван |
| Tools | BFCL v4 | Вова |
| Arena | Arena-Hard LC | ML |
| RU | MERA, Русификатор test, Ты/Вы test, Смена test | Поля |
| Safety | Internal safety test, refusal rate | Поля |
| OBM format | 50 задач, retention >= 90% | Иван |

### Цель 2: Reward baseline table

Замерить каждый reward по P/R/F1 — таблица work/don't work.

**OS rewards (5 штук):**
- Skywork-8B, Skywork-0.6B, ArmoRM-8B, InternLM2-7B, ToolRM-1.7B

**Determ checks (4 штуки):**
- loop_detected, repetition_ratio, json_valid, latex_valid → P >= 95%

**B2C checkers (3 штуки):**
- interpreter, websearch, text2image → P/R на 100 примерах

**Verification test domains:**
- MD test (800+ примеров в ОБМ)
- Русификатор test (300+ примеров)
- Эквивалентность test (1.1k примеров)
- Ты/Вы test (288 примеров)
- Смена языков test (300+ примеров)
- Arena subset
- Safety subset

---

## Карта экспериментов (2 недели)

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-LEAK-1 | W1 | Иван-3 | Eval overlap check: rollout vs eval vs reward |
| EXP-RWVER-1 | W1-2 | Дм-2 + МЛ | 5 rewards x test domains → P/R/F1 |
| EXP-DISTILL-START | W1 | Эльдар-1 | Multi-teacher pipeline запуск |
| EXP-BFCL-BASE | W1 | Вова-1 | BFCL v4: 10b vs Qwen3.5-2B/4B |
| EXP-LCB-BASE | W1 | Серёжа-1 | LCB v6: 10b vs Qwen3.5-2B/4B |
| EXP-B2C-CHECK | W1 | Даша-1..3 | interpreter/search/t2i checker P/R |
| EXP-PROCESSBENCH | W1 | Эльдар-3 | ProcessBench: 10b vs конкуренты |
| EXP-EVAL-SWEEP | W2 | ML | Full eval sweep: все треки x 5 моделей |
| EXP-OSMATRIX | W2 | Дм-2 | OS reward matrix: 5 rewards x все тесты |
| EXP-OBM-FORMAT | W2 | Иван-3 | OBM format: 50 задач, retention |

---

## Недельная разбивка

### W1 (Mar 17–21): Foundation + OS rewards deploy

**Цель:** Поднять инфру. OS rewards online. Baseline запуски. Reward test sets готовы. Leak check.

| Команда | Ключевые задачи | Детали |
|---------|-----------------|--------|
| Эльдар | ProcessBench baseline ([Э-3](01_eldar_research_sft.md#э3-w1)), Distill pipeline start ([Э-1](01_eldar_research_sft.md#э1-w1)), SFT annotation start ([Э-2](01_eldar_research_sft.md#э2-w1)) | [подробнее](01_eldar_research_sft.md#w1) |
| Иван | Leak check ([И-3](02_ivan_research_stage15.md#и3-w1)), данные для baseline ([И-1+2](02_ivan_research_stage15.md#и1-w1)) | [подробнее](02_ivan_research_stage15.md#w1) |
| Вова | BFCL v4 baseline ([В-1](03_vova_functions.md#в1-w1)), ToolRM online ([В-2](03_vova_functions.md#в2-w1)) | [подробнее](03_vova_functions.md#w1) |
| Серёжа | LCB v6 baseline + sandbox ([С-1](04_sereja_code.md#с1-w1)), code queries 10k+ ([С-2](04_sereja_code.md#с2-w1)) | [подробнее](04_sereja_code.md#w1) |
| Поля | MD test 800+ → ОБМ ([П-1](05_polya_product.md#п1-w1)), Reward test sets: Русификатор, Эквивалентность, Ты/Вы, Смена, Safety pilot ([П-2..6](05_polya_product.md#п2-w1)) | [подробнее](05_polya_product.md#w1) |
| Даша | Interpreter P/R ([Д-1](06_dasha_platform.md#д1-w1)), Search P/R ([Д-2](06_dasha_platform.md#д2-w1)), T2I P/R ([Д-3](06_dasha_platform.md#д3-w1)), B2C report ([Д-4](06_dasha_platform.md#д4-w1)) | [подробнее](06_dasha_platform.md#w1) |
| Дима | Storage ([Дм-1](07_dima_infra.md#дм1-w1)), OS rewards x5 + verify ([Дм-2](07_dima_infra.md#дм2-w1)), Determ checks ([Дм-3](07_dima_infra.md#дм3-w1)) | [подробнее](07_dima_infra.md#w1) |
| ML | OS reward verification design ([МЛ-1](08_ml.md#мл1-w1)), Eval sweep pipeline ([МЛ-2](08_ml.md#мл2-w1)), Math baseline ([МЛ-3](08_ml.md#мл3-w1)) | [подробнее](08_ml.md#w1) |

**Friday W1 Checkpoint:**
- [ ] Storage + endpoint works. >= 3 OS rewards online + first P/R/F1 numbers.
- [ ] Determ checks API (loop, repetition, json_valid, latex_valid), P >= 95%.
- [ ] Leak report: overlap % rollout vs eval vs reward.
- [ ] BFCL v4 baseline: 10b_toolmind vs Qwen3.5-2B — table.
- [ ] LCB v6 baseline: 10b_toolmind vs Qwen3.5-2B — table.
- [ ] Reward test sets готовы: MD 800+, Русификатор 300+, Эквивалентность 1.1k, Ты/Вы 288, Смена 300+.
- [ ] ProcessBench accuracy: 10b_toolmind vs competitors.
- [ ] B2C checkers: interpreter P/R, search P/R, t2i P/R на 100 примерах.

---

### W2 (Mar 24–28): Full baseline + reward matrix

**Цель:** Полная таблица "мы vs конкуренты" по всем трекам. Reward matrix. OBM format gate.

| Команда | Ключевые задачи | Детали |
|---------|-----------------|--------|
| Эльдар | ProcessBench vs competitors table ([Э-3](01_eldar_research_sft.md#э3-w2)), Distill pipeline → first 5k ([Э-1](01_eldar_research_sft.md#э1-w2)), SFT +20k annotated ([Э-2](01_eldar_research_sft.md#э2-w2)) | [подробнее](01_eldar_research_sft.md#w2) |
| Иван | OBM format 50 задач ([И-3](02_ivan_research_stage15.md#и3-w2)), данные доменное распределение ([И-1+2](02_ivan_research_stage15.md#и1-w2)) | [подробнее](02_ivan_research_stage15.md#w2) |
| Вова | BFCL vs competitors table ([В-1](03_vova_functions.md#в1-w2)), ToolRM error analysis ([В-2](03_vova_functions.md#в2-w2)) | [подробнее](03_vova_functions.md#w2) |
| Серёжа | LCB vs competitors table ([С-1](04_sereja_code.md#с1-w2)), code reward P/R ([С-2](04_sereja_code.md#с2-w2)) | [подробнее](04_sereja_code.md#w2) |
| Поля | OS reward verification: MD/Русификатор/Эквивалентность/Ты/Вы/Смена/Arena/Safety ([П-1..6](05_polya_product.md#п1-w2)) | [подробнее](05_polya_product.md#w2) |
| Даша | EXP-B2C-1 baseline report ([Д-1+2](06_dasha_platform.md#д1-w2)) | [подробнее](06_dasha_platform.md#w2) |
| Дима | OS reward matrix: 5 rewards x все тесты → P/R/F1 ([Дм-2](07_dima_infra.md#дм2-w2)), inventory (Дм-1), LoRA pipeline test ([Дм-3](07_dima_infra.md#дм3-w2)) | [подробнее](07_dima_infra.md#w2) |
| ML | Full eval sweep: все треки x 5 моделей ([МЛ-1..3](08_ml.md#мл1-w2)), reward matrix analysis ([МЛ-4](08_ml.md#мл4-w2)) | [подробнее](08_ml.md#w2) |

**Friday W2 Checkpoint:**
- [ ] **Baseline table: 10b_toolmind vs Qwen3.5-2B vs Qwen3.5-4B vs GLM-5 vs DeepSeek V3.2 — все треки.**
- [ ] **Reward matrix: 5 OS rewards x test domains → P/R/F1 → work/don't work.**
- [ ] Determ checks: все 4 в статусе work/don't work.
- [ ] **OBM format 50 задач: retention >= 90%.**
- [ ] ProcessBench: comparison table vs Qwen3.5/GLM-5/DeepSeek.
- [ ] EXP-B2C-1: baseline report B2C (call distribution, loop rate, format).
- [ ] Code reward pipeline P/R.

---

## Ключевые артефакты (DoD за 2 недели)

### Baseline table

| Трек | Метрика | 10b_toolmind | Qwen3.5-2B | Qwen3.5-4B | GLM-5 | DeepSeek V3.2 |
|------|---------|-------------|------------|------------|-------|---------------|
| Math | MATH-500 | ? | ? | ? | ? | ? |
| Math | AIME | ? | ? | ? | ? | ? |
| Math | GSM8K | ? | ? | ? | ? | ? |
| Code | LCB v6 | ? | ? | ? | ? | ? |
| Code | SWE-bench | ? | ? | ? | ? | ? |
| IF | IFEval strict | ? | ? | ? | ? | ? |
| IF | Vikacheck | ? | ? | ? | ? | ? |
| Tools | BFCL v4 | ? | ? | ? | ? | ? |
| Arena | Arena-Hard LC | ? | ? | ? | ? | ? |
| RU | Русификатор test | ? | ? | ? | ? | ? |
| RU | Ты/Вы test | ? | ? | ? | ? | ? |
| RU | Смена test | ? | ? | ? | ? | ? |
| Safety | refusal rate | ? | ? | ? | ? | ? |
| OBM | retention 50 задач | ? | — | — | — | — |

### Reward matrix

| Reward | MD test | Русификатор | Эквивалентность | Ты/Вы | Смена | Arena | Safety | Вердикт |
|--------|---------|-------------|-----------------|-------|-------|-------|--------|---------|
| Skywork-8B | ? | ? | ? | ? | ? | ? | ? | ? |
| Skywork-0.6B | ? | ? | ? | ? | ? | ? | ? | ? |
| ArmoRM-8B | ? | ? | ? | ? | ? | ? | ? | ? |
| InternLM2-7B | ? | ? | ? | ? | ? | ? | ? | ? |
| ToolRM-1.7B | ? | ? | ? | ? | ? | ? | ? | ? |
| loop_detected | — | — | — | — | — | — | — | P>=95%? |
| repetition_ratio | — | — | — | — | — | — | — | P>=95%? |
| json_valid | — | — | — | — | — | — | — | P>=95%? |
| latex_valid | — | — | — | — | — | — | — | P>=95%? |
| interpreter | — | — | — | — | — | — | — | P/R на 100 |
| websearch | — | — | — | — | — | — | — | P/R на 100 |
| text2image | — | — | — | — | — | — | — | P/R на 100 |

---

## Загрузка по людям (2 недели)

| Человек | W1 | W2 |
|---------|----|----|
| Эльдар-1 | distill pipeline start | pipeline → 5k batch |
| Эльдар-2 | SFT 20k annotation start | SFT +20k annotated |
| Эльдар-3 | **ProcessBench baseline** | **ProcessBench vs competitors table** |
| Иван-1 | math/code данные распределение | domain coverage audit |
| Иван-2 | данные + labels | данные + domain coverage |
| Иван-3 | **leak check** | **OBM format 50 задач** |
| Вова-1 | **BFCL v4 baseline** + ToolRM online | **BFCL vs competitors table** |
| Вова-2 | tool specs verify | ToolRM error analysis |
| Серёжа-1 | **LCB v6 + sandbox** | **LCB vs competitors table** |
| Серёжа-2 | code queries 10k+ | code reward P/R |
| Поля-1 | MD test 800+ → ОБМ | OS reward verification MD |
| Поля-2 | Смена/Русификатор test sets | OS reward verification RU |
| Поля-3 | Safety pilot + IAA | OS reward verification Safety |
| Поля-4 | Arena разметка | OS reward verification Arena |
| Поля-5 | Эквивалентность / Ты/Вы test sets | Эквивалентность LoRA start |
| Поля-6 | Vikacheck reward format | reward verification report |
| Даша-1 | interpreter P/R на 100 | EXP-B2C-1 report |
| Даша-2 | search P/R на 100 | EXP-B2C-1 report |
| Даша-3 | t2i trigger P/R | B2C reward data |
| Даша-4 | B2C baseline report | B2C reward data |
| Дима-1 | **Storage + S3 + CLI** | inventory table |
| Дима-2 | **OS rewards x5 + verification** | **OS reward matrix 5x7** |
| Дима-3 | **determ checks x4** | LoRA pipeline test |
| ML-1..3 | OS reward verification design + math baseline | **Full eval sweep all tracks** |
| ML-4..7 | Eval sweep pipeline setup | **Reward matrix analysis** |

---

## Топ-5 рисков

| # | Риск | P | Impact | Митигация |
|---|------|---|--------|-----------|
| 1 | GPU capacity для 5 OS rewards | Medium | High | Приоритет: Skywork-0.6B + ArmoRM-8B + InternLM2-7B |
| 2 | BFCL/LCB infra для конкурентов недоступна | Medium | High | Скачать веса заранее W1 Day 1 |
| 3 | Reward test sets не готовы к W2 | Medium | High | Параллельно W1 Days 1-3 |
| 4 | Leak check показывает overlap > 5% | Medium | Critical | Немедленная фильтрация + re-split |
| 5 | OBM format retention < 90% | Medium | High | Анализ W2 Days 1-2, патч промпта |

---

## Критические решения

| День | Решение | Кто решает | Данные |
|------|---------|------------|--------|
| W1 Fri | Какие OS rewards работают (P/R достаточно)? | Дима + ML lead | EXP-RWVER-1 первые числа |
| W2 Mon | Приоритет eval sweep конфигурации | ML lead | W1 baseline numbers |
| W2 Thu | **Какие rewards включаем в RL v0?** | Дима + ML lead | OS reward matrix final |
| W2 Fri | **Go/no-go на Cold Start v0** | All leads | Все gate данные |
