# Online RL: План работ на 8 недель (Mar 17 — May 9, 2026)

> Последнее обновление: 2026-03-11
> Цель: от текущего состояния к production-ready модели за 8 недель.
> Связанные документы: [gap analysis](../online_rl_ml_data_gap_analysis.md), [команды](../online_rl_teams.md), [индекс](../online_rl_index.md).

---

## Файлы команд

| Файл | Команда | Лид | Человек |
|------|---------|-----|---------|
| [01_eldar_research_sft.md](01_eldar_research_sft.md) | Research: SFT + Distill + PRM | Эльдар | 3 |
| [02_ivan_research_stage15.md](02_ivan_research_stage15.md) | Research: Stage 1.5 + IFEval | Иван | 3 |
| [03_vova_functions.md](03_vova_functions.md) | Functions: Tools + Agents + SWE | Вова | 2 |
| [04_sereja_code.md](04_sereja_code.md) | Code: reasoning + reward | Серёжа | 2 |
| [05_polya_product.md](05_polya_product.md) | Product: B2C reward + Safety | Поля | 6 |
| [06_dasha_platform.md](06_dasha_platform.md) | Platform: B2C функции + reward | Даша | 4 |
| [07_dima_infra.md](07_dima_infra.md) | Infra: хранение, LoRA, checks | Дима | 3 |
| [08_ml.md](08_ml.md) | ML: RL pipeline, эксперименты, 120B | ML | 7+ |
| [09_reward_collection_guide.md](09_reward_collection_guide.md) | Гайд: протокол сбора собственного reward | — | Справочник |

**Всего:** 23 data + 7+ ML = 30+ человек.

---

## 0. Путь модели: milestone-ы

### Pipeline

```
Stage 1.5 (multi-teacher distill) -> SFT (masking + annotation) -> Online RL (rewards)
```

> DPO отложен на пост-8-недельный план. См. [gap analysis 0c+](../online_rl_ml_data_gap_analysis.md).

### Состояния модели по неделям

| # | Состояние | Неделя | Что сделано | GATE для перехода |
|---|-----------|--------|------------|-------------------|
| 0 | **Base** | W1 | Текущая 10b_toolmind. Замер baselines. OS reward verification. Distill pipeline. Leak check. | — |
| 1 | **Cold Start v0** | W2-3 | Stage 1.5: 2M записей (multi-teacher distill). OLMO RL baseline. OS rewards verified. | Все домены покрыты, quality audit >= 95%, OLMO RL works |
| 2 | **Cold Start v1** | W3-4 | SFT: 100-200k validated + annotated (MD, RU, logic, Arena). Лучшая маскировка. First real RL с verified rewards. | **Обыгрывает distill Qwen-3/3.5/GLM-5** |
| 3 | **RL v0** | W4-5 | Online RL: OS rewards + determ checks + наши данные + OLMO mix. Multi-reward. | math/code растут, нет cross-domain регрессии |
| 4 | **RL v1** | W5-7 | Online RL: own reward LoRA + full stack. 120B checkpoint eval. | Все треки растут vs RL v0 |
| 5 | **RL v2 / Prod** | W7-8 | Оптимизация, canary 48h, release | Все promotion thresholds пройдены |

### Целевые метрики по направлениям

| Направление | Метрика | Base (W1) | CS v0 | CS v1 | RL v0 | RL v1 | Prod target |
|-------------|---------|-----------|-------|-------|-------|-------|-------------|
| **Math** | math_500 | ~0.87 | 0.89+ | 0.90+ | 0.92+ | 0.93+ | TBD после W1 |
| **Math** | AIME | ~0.31 | 0.35+ | 0.40+ | 0.45+ | 0.50+ | TBD после W1 |
| **Code** | LCB v6 | ? | +5% | +8% | +12% | +15% | TBD после W1 |
| **Code** | SWE-bench | 1-4/500 | x3 | x5 | x8 | x10 | TBD после W1 |
| **IF** | IFEval strict | ? | +5% | +10% | +12% | +15% | TBD после W1 |
| **Tools** | BFCL v4 | ? | = | = | +5% | +10% | TBD после W1 |
| **Arena** | Arena-Hard LC | ? | +3% | +5% | +10% | +15% | TBD после W1 |
| **RU** | Русификатор + Ты/Вы + Смена | ? | +5% | +8% | +10% | +12% | Internal **#1** |
| **Safety** | refusal rate | ? | = | = | 95%+ | 98%+ | **99%+** |

> **Сравниваем с Qwen3.5-2B** (2B dense, ближайший по active params к нашим 1.8B) и **Qwen3.5-4B** (stretch target).

---

## 1. Gate checklists

### Gate 0: Cold Start Readiness

- [ ] Stage 1.5 данные >= 2M, покрытие всех доменов
- [ ] Multi-teacher distill pipeline operational (beats single-teacher on 100-sample audit)
- [ ] SFT данные 100-200k, провалидированы (quality audit)
- [ ] Все SFT данные annotated по reward directions (MD, RU, logic, Arena)
- [ ] Стратегия маскирования выбрана (EXP-SFT-5 ablation done)
- [ ] Наш SFT обыгрывает distill Qwen-3/3.5/GLM-5 (EXP-SFT-1..4)
- [ ] Developer system prompt <= 1k tokens, edge cases ok

### Gate 1: Data Readiness

- [ ] Rollout queries dedup/leak checked, zero overlap с eval
- [ ] Все reward train sets >= 2k (per checker)
- [ ] Все reward test sets >= 500 (per checker)
- [ ] Хранилище определено, pipeline ингеста работает
- [ ] Доменная смесь валидирована (минимум 1 ablation)
- [ ] Per-sample reward_config формат работает

### Gate 2: Reward Readiness

- [ ] Все OS rewards deployed, verified на OLMO + нашем сете
- [ ] Non-working OS rewards identified and excluded
- [ ] Свои rewards замерены (таблица own vs OS P/R/F1)
- [ ] Safety reward operational (OS или свой, precision >= 99%)
- [ ] Reward routing v1 operational (ArmoRM + rule overrides)
- [ ] Формула агрегации определена и протестирована (EXP-AGG)
- [ ] Детерминированные rewards deployed (циклы, формат, LaTeX)

### Gate 3: Training Readiness

- [ ] Round 3 завершён (clip range + dr.grpo)
- [ ] OLMO RL baseline done (positive learning signal confirmed)
- [ ] pass@k fix deployed, корректность проверена
- [ ] Eval sweep pipeline автоматизирован (все треки per checkpoint)
- [ ] Collapse mitigation validated (>= 300 степов без collapse)
- [ ] Process reward для math протестирован (EXP-PRM)

### Gate 4: Eval Readiness

- [ ] Все capability-треки замерены на baseline
- [ ] **Qwen-3.5-2B** + Qwen-3.5-4B + GLM-5 + DeepSeek V3.2 замерены на тех же бенчах
- [ ] Promotion thresholds зафиксированы по каждому треку
- [ ] OBM format validation >= 90% (50 задач)
- [ ] Safety per-domain breakdown готов (ЦУР + political + all)
- [ ] **120B checkpoint eval** results documented

### Gate 5: Release Readiness

- [ ] Все primary metrics >= promotion threshold
- [ ] Ни один трек не регрессировал > allowed delta
- [ ] Safety gate: refusal >= 99%, false refusal <= target, per-domain ok
- [ ] Factuality gate: SimpleQA >= threshold
- [ ] Canary 48h в prod-like env стабилен
- [ ] 120B eval: no regressions vs 10B proportional targets

---

## 2. Понедельный план (верхний уровень)

### W1 (Mar 17-21): Foundation — OS reward verification, distill pipeline

**Цель:** Развернуть инфру. Baselines. Verify OS rewards. Distill pipeline. Leak check. PRM start.
**Модель:** Base (без изменений).

| Команда | Ключевые задачи | Детали |
|---------|-----------------|--------|
| Эльдар | Distill pipeline ([Э-1](01_eldar_research_sft.md#э1-w1)), SFT 20k + annotation ([Э-2](01_eldar_research_sft.md#э2-w1)), PRM start ([Э-3](01_eldar_research_sft.md#э3-w1)) | [подробнее](01_eldar_research_sft.md#w1) |
| Иван | Math 200k ([И-1](02_ivan_research_stage15.md#и1-w1)), Code 100k ([И-2](02_ivan_research_stage15.md#и2-w1)), Leak check ([И-3](02_ivan_research_stage15.md#и3-w1)) | [подробнее](02_ivan_research_stage15.md#w1) |
| Вова | BFCL baseline ([В-1](03_vova_functions.md#в1-w1)), Tool specs + SWE ([В-2](03_vova_functions.md#в2-w1)) | [подробнее](03_vova_functions.md#w1) |
| Серёжа | LCB + sandbox ([С-1](04_sereja_code.md#с1-w1)), Code queries ([С-2](04_sereja_code.md#с2-w1)) | [подробнее](04_sereja_code.md#w1) |
| Поля | MD-check ([П-1](05_polya_product.md#п1-w1)), Смена ([П-2](05_polya_product.md#п2-w1)), Safety ([П-3](05_polya_product.md#п3-w1)), Arena ([П-4](05_polya_product.md#п4-w1)), Русификатор ([П-5](05_polya_product.md#п5-w1)), Vikacheck ([П-6](05_polya_product.md#п6-w1)) | [подробнее](05_polya_product.md#w1) |
| Даша | Interpreter ([Д-1](06_dasha_platform.md#д1-w1)), Search ([Д-2](06_dasha_platform.md#д2-w1)), T2I ([Д-3](06_dasha_platform.md#д3-w1)), B2C report ([Д-4](06_dasha_platform.md#д4-w1)) | [подробнее](06_dasha_platform.md#w1) |
| Дима | Storage ([Дм-1](07_dima_infra.md#дм1-w1)), OS rewards + verify ([Дм-2](07_dima_infra.md#дм2-w1)), Determ checks ([Дм-3](07_dima_infra.md#дм3-w1)) | [подробнее](07_dima_infra.md#w1) |
| ML | Round 3 ([МЛ-1..4](08_ml.md#мл1-w1)), pass@k ([МЛ-5](08_ml.md#мл5-w1)), OS reward verification ([МЛ-6+7](08_ml.md#мл6-w1)) | [подробнее](08_ml.md#w1) |

**Friday Checkpoint:**
- [ ] Storage + OS rewards online + verification P/R/F1
- [ ] Distill pipeline v0 demo
- [ ] Leak check report
- [ ] SFT >= 20k annotated, ProcessBench accuracy
- [ ] Stage 1.5: math >= 200k, code >= 100k
- [ ] BFCL + LCB baseline vs Qwen3.5-2B/4B
- [ ] Round 3 launched (50+ steps)

---

### W2 (Mar 24-28): OS Reward Matrix + OLMO RL Baseline

**Цель:** OS matrix. OLMO RL baseline (4 configs). Distill pipeline → production. First own reward. Stage 1.5 >= 600k.
**Модель:** Base. OLMO RL signal confirmed/denied.

| Команда | Ключевые задачи | Детали |
|---------|-----------------|--------|
| Эльдар | Pipeline → prod ([Э-1](01_eldar_research_sft.md#э1-w2)), SFT +30k annotated ([Э-2](01_eldar_research_sft.md#э2-w2)), PRM test set ([Э-3](01_eldar_research_sft.md#э3-w2)) | [подробнее](01_eldar_research_sft.md#w2) |
| Иван | Stage 1.5 → 600k ([И-1+2](02_ivan_research_stage15.md#и1-w2)), Contamination fix ([И-3](02_ivan_research_stage15.md#и3-w2)) | [подробнее](02_ivan_research_stage15.md#w2) |
| Вова | ToolRM BFCL ([В-1](03_vova_functions.md#в1-w2)), SWE 10b ([В-2](03_vova_functions.md#в2-w2)) | [подробнее](03_vova_functions.md#w2) |
| Серёжа | Code reward pipe ([С-1](04_sereja_code.md#с1-w2)), Queries labels ([С-2](04_sereja_code.md#с2-w2)) | [подробнее](04_sereja_code.md#w2) |
| Поля | MD train ([П-1](05_polya_product.md#п1-w2)), Русификатор 500+ ([П-2](05_polya_product.md#п2-w2)), Safety 1k ([П-3](05_polya_product.md#п3-w2)), Arena 30% ([П-4](05_polya_product.md#п4-w2)), **Эквивалентность LoRA** ([П-5](05_polya_product.md#п5-w2)), Vikacheck ([П-6](05_polya_product.md#п6-w2)) | [подробнее](05_polya_product.md#w2) |
| Даша | EXP-B2C-1 ([Д-1+2](06_dasha_platform.md#д1-w2)), Reward data ([Д-3+4](06_dasha_platform.md#д3-w2)) | [подробнее](06_dasha_platform.md#w2) |
| Дима | Ingest all ([Дм-1](07_dima_infra.md#дм1-w2)), **OS matrix** ([Дм-2](07_dima_infra.md#дм2-w2)), LoRA pipeline ([Дм-3](07_dima_infra.md#дм3-w2)) | [подробнее](07_dima_infra.md#w2) |
| ML | Round 3 analysis ([МЛ-1..4](08_ml.md#мл1-w2)), **OLMO RL baseline** ([МЛ-5+6](08_ml.md#мл5-w2)), Dev prompt ([МЛ-7](08_ml.md#мл7-w2)) | [подробнее](08_ml.md#w2) |

**Friday Checkpoint:**
- [ ] OS reward matrix documented
- [ ] Эквивалентность LoRA P/R/F1
- [ ] Stage 1.5 >= 600k; SFT >= 50k annotated
- [ ] **OLMO RL: 4 configs compared, learning signal confirmed/denied**
- [ ] Round 3: best clip + dr.grpo

---

### W3 (Mar 31 - Apr 4): Reward Training v1 + First Real RL

**Цель:** MD + Русификатор LoRA. SFT masking ablation. Stage 1.5 >= 1M. **First real RL canary.**
**Модель:** Cold Start v0 training started. RL canary: math/code delta.

| Команда | Ключевые задачи | Детали |
|---------|-----------------|--------|
| Эльдар | **SFT masking ablation** ([Э-1+2](01_eldar_research_sft.md#э1-w3)), PRM cross-val ([Э-3](01_eldar_research_sft.md#э3-w3)) | [подробнее](01_eldar_research_sft.md#w3) |
| Иван | Stage 1.5 → 1M ([И-1+2](02_ivan_research_stage15.md#и1-w3)), IFEval reward ([И-3](02_ivan_research_stage15.md#и3-w3)) | [подробнее](02_ivan_research_stage15.md#w3) |
| Поля | **MD LoRA** ([П-1](05_polya_product.md#п1-w3)), Русификатор LoRA ([П-2](05_polya_product.md#п2-w3)), Safety train ([П-3](05_polya_product.md#п3-w3)), Arena 60% ([П-4](05_polya_product.md#п4-w3)), Ты/Вы ([П-5](05_polya_product.md#п5-w3)), Classifier ([П-6](05_polya_product.md#п6-w3)) | [подробнее](05_polya_product.md#w3) |
| Дима | Determ→RL ([Дм-1](07_dima_infra.md#дм1-w3)), LoRA training ([Дм-2](07_dima_infra.md#дм2-w3)), rw_config ([Дм-3](07_dima_infra.md#дм3-w3)) | [подробнее](07_dima_infra.md#w3) |
| ML | DPPO ([МЛ-1+2](08_ml.md#мл1-w3)), Sampler ([МЛ-3+4](08_ml.md#мл3-w3)), **First real RL canary** ([МЛ-5+6](08_ml.md#мл5-w3)), CS v0 ([МЛ-7](08_ml.md#мл7-w3)) | [подробнее](08_ml.md#w3) |

**Friday Checkpoint:**
- [ ] **MD LoRA acc ~0.94**; Русификатор LoRA metrics
- [ ] **SFT masking: 4 strategies compared, best selected**
- [ ] Stage 1.5 >= 1M; SFT >= 80k annotated
- [ ] **First real RL canary: math+code delta**

---

### W4 (Apr 7-11): SFT Gate + RL v0 — multi-reward

**Цель:** **SFT GATE: beats distill.** All LoRA validated. Multi-reward RL.
**Модель:** **Cold Start v1** (SFT). RL v0 started.

| Команда | Ключевые задачи | Детали |
|---------|-----------------|--------|
| Эльдар | **SFT final** ([Э-1](01_eldar_research_sft.md#э1-w4)), **SFT vs distill** ([Э-2](01_eldar_research_sft.md#э2-w4)), PRM LoRA ([Э-3](01_eldar_research_sft.md#э3-w4)) | [подробнее](01_eldar_research_sft.md#w4) |
| Иван | **Stage 1.5 → 2M GATE** ([И-1+2](02_ivan_research_stage15.md#и1-w4)), Dynamic sampling ([И-3](02_ivan_research_stage15.md#и3-w4)) | [подробнее](02_ivan_research_stage15.md#w4) |
| Поля | Safety LoRA ([П-1](05_polya_product.md#п1-w4)), Arena 2k ([П-2](05_polya_product.md#п2-w4)), Смена LoRA ([П-3](05_polya_product.md#п3-w4)), Ты/Вы LoRA ([П-4](05_polya_product.md#п4-w4)), **All LoRA validation** ([П-5](05_polya_product.md#п5-w4)), Classifier v1 ([П-6](05_polya_product.md#п6-w4)) | [подробнее](05_polya_product.md#w4) |
| Дима | **Aggregation** ([Дм-1](07_dima_infra.md#дм1-w4)), Multi-rw pipe ([Дм-2](07_dima_infra.md#дм2-w4)), Canary infra ([Дм-3](07_dima_infra.md#дм3-w4)) | [подробнее](07_dima_infra.md#w4) |
| ML | Multi-reward RL + **EXP-AGG** + SFT training | [подробнее](08_ml.md#w4) |

**Friday Checkpoint:**
- [ ] **SFT GATE: beats distill → PASS/FAIL**
- [ ] **Stage 1.5: 2M DONE**
- [ ] All LoRA P/R/F1 table
- [ ] Multi-reward RL metrics
- [ ] **CRITICAL: если SFT не бьёт distill → STOP**

---

### W5 (Apr 14-18): RL v0 Extended + 120B Prep

**Цель:** RL v0 100+ steps. Full reward stack. **120B checkpoint selection.**
**Модель:** **RL v0** at scale. 120B candidate selected.

| Команда | Ключевые задачи | Детали |
|---------|-----------------|--------|
| Эльдар | PRM ablation ([Э-1+2](01_eldar_research_sft.md#э1-w5)), SFT check ([Э-3](01_eldar_research_sft.md#э3-w5)) | [подробнее](01_eldar_research_sft.md#w5) |
| Поля | Arena LoRA, Safety P99, routing, B2C test, NLI | [подробнее](05_polya_product.md#w5) |
| ML | RL v0 100+ steps ([МЛ-1..4](08_ml.md#мл1-w5)), RLEP ([МЛ-5+6](08_ml.md#мл5-w5)), **120B selection** ([МЛ-7](08_ml.md#мл7-w5)) | [подробнее](08_ml.md#w5) |

**Friday Checkpoint:**
- [ ] RL v0 100+ steps, all tracks measured
- [ ] Full reward stack operational
- [ ] **120B checkpoint selected**

---

### W6 (Apr 21-25): RL v1 + 120B Eval

**Цель:** RL v1 300+ steps. Safety per-domain. **120B eval sweep.**
**Модель:** **RL v1**. **120B eval results.**

**Friday Checkpoint:**
- [ ] **RL v1: 300+ steps without collapse**
- [ ] Safety per-domain (7 domains)
- [ ] **120B eval sweep: results on all tracks**

---

### W7 (Apr 28 - May 2): Pre-release — eval sweep, canary

**Цель:** Full eval sweep vs **Qwen3.5-2B/4B**, GLM-5, DeepSeek V3.2. Canary 48h start.
**Модель:** **RL v2 candidate**.

**Friday Checkpoint:**
- [ ] **Full eval sweep: us vs competitors (all tracks)**
- [ ] Per-track gate: GREEN/RED
- [ ] Canary 48h started; 120B results

---

### W8 (May 5-9): Release gate

**Цель:** Решение о релизе.
**Модель:** **Prod release** или **blocker list**.

**Friday Checkpoint:**
- [ ] **Gate 5: ALL PASS/FAIL**
- [ ] Canary 48h stable
- [ ] **DECISION: RELEASE / BLOCKED**

---

## 3. Загрузка по людям

| Человек | W1 | W2 | W3 | W4 | W5 | W6 | W7 | W8 |
|---------|----|----|----|----|----|----|----|----|
| Эльдар-1 | **distill pipeline** | pipeline → prod | **masking ablation** | SFT final | PRM ablation | PRM strategy | ProcessBench | sign-off |
| Эльдар-2 | SFT 20k + annotation | SFT +30k annotated | masking ablation | **SFT vs distill** | SFT check | regression | math gate | sign-off |
| Эльдар-3 | ProcessBench + PRM start | PRM prompt + test set | PRM cross-val | PRM 0.6B LoRA | PRM check | PRM expand | PRMBench | sign-off |
| Иван-1 | math 200k | code+IF 200k | STEM/B2C 200k | **Stage 1.5 2M** | domain mix | mixed sampler | IFEval gate | sign-off |
| Иван-2 | code 100k | code+IF 100k | STEM/RU 100k | Stage 1.5 2M | IFEval ablation | dynamic sampling | Vikacheck gate | sign-off |
| Иван-3 | dedup + **leak check** | contamination fix | IFEval reward | dynamic proto | difficulty sampler | OBM format | thresholds | sign-off |
| Вова-1 | BFCL baseline | ToolRM BFCL | tool reward v1 | EFF-1 | OTC-PO | loop penalty | BFCL gate | sign-off |
| Вова-2 | tool specs+SWE | SWE 10b | SWE-smith | agent synth | trajectories | SWE expand | agent gate | sign-off |
| Серёжа-1 | LCB+sandbox | code reward pipe | code reward v1 | code set final | code RL 50 | code RL 100 | LCB gate | sign-off |
| Серёжа-2 | code queries | queries labels | difficulty filter | canary 20 | VeRPO | code+math | SWE gate | sign-off |
| Поля-1 | MD test→ОБМ | MD train | **MD LoRA** | Safety LoRA | Arena LoRA | fix regress | reward audit | safety gate |
| Поля-2 | Смена train | Русификатор 500+ | Русификатор LoRA | Arena test 2k | Safety P99 | expand weak | Arena swap | factuality |
| Поля-3 | Safety pilot | Safety 1000+ | Safety train+test | Смена LoRA | all tests 500 | **safety domains** | Safety swap | safety gate |
| Поля-4 | Arena markup | Arena 30% | Arena 60% | Ты/Вы LoRA | routing deploy | human eval | human eval | sign-off |
| Поля-5 | Русификатор→rw | Эквивалентность LoRA | Ты/Вы expand | **all LoRA valid** | B2C test | rw ensemble | factuality | sign-off |
| Поля-6 | Vikacheck→rw | Vikacheck-reasoning | classifier start | classifier v1 | NLI factuality | hack monitor | B2C metrics | sign-off |
| Даша-1 | interp reward | B2C baseline | EXP-B2C-2 | EXP-B2C-4 | NLI fact-check | B2C RL 100 | B2C gate | sign-off |
| Даша-2 | search reward | B2C baseline | EXP-B2C-3 | EXP-B2C-5 | search ground | B2C RL 100 | B2C gate | sign-off |
| Даша-3 | t2i trigger | reward data | t2i reward | websearch NLI | B2C canary | URL track | latency | sign-off |
| Даша-4 | B2C report | reward data | B2C RL prep | B2C integration | B2C canary | max calls | latency | sign-off |
| Дима-1 | storage | ingest all | determ→RL | **aggregation** | prod pipeline | PAR hack | prod deploy | canary 48h |
| Дима-2 | OS rewards + **verify** | **OS matrix** | LoRA training | multi-rw pipe | monitoring | gradient reg | eval sweep | monitoring |
| Дима-3 | determ checks | LoRA pipeline | rw_config auto | canary infra | ODIN | scaling 200+ | **competitor bench** | rollback |

---

## 4. Карта экспериментов

| ID | Неделя | Owner | Зависимость | Описание |
|----|--------|-------|-------------|----------|
| **Distill Pipeline** | | | | |
| EXP-DISTILL-1 | W1-2 | [Э-1](01_eldar_research_sft.md#э1-w1) | API access | Multi-teacher pipeline: quality vs single-teacher |
| **Cold Start** | | | | |
| EXP-SFT-1 | W2 | [Э-2](01_eldar_research_sft.md#э2-w2) | SFT >= 50k | Baseline distill Qwen-3 |
| EXP-SFT-2 | W4 | [Э-2](01_eldar_research_sft.md#э2-w4) | SFT-1 done | Distill Qwen-3.5 |
| EXP-SFT-3 | W4 | [Э-2](01_eldar_research_sft.md#э2-w4) | SFT-1 done | Distill GLM-5 |
| EXP-SFT-4 | W4 | [Э-1](01_eldar_research_sft.md#э1-w4) | SFT-5 done | Наш подход (masking) |
| EXP-SFT-5 | W3 | [Э-1+2](01_eldar_research_sft.md#э1-w3) | SFT >= 80k | Masking ablation a/b/c/d |
| **OS Reward Verification** | | | | |
| EXP-RWVER-1 | W1 | [Дм-2](07_dima_infra.md#дм2-w1) + [МЛ-6+7](08_ml.md#мл6-w1) | OS rewards deployed | 5 rewards x 100 examples → P/R/F1 |
| EXP-LEAK-1 | W1 | [И-3](02_ivan_research_stage15.md#и3-w1) | datasets loaded | Eval overlap check |
| **OLMO RL Baseline** | | | | |
| EXP-OLMO-1 | W2 | [МЛ-5+6](08_ml.md#мл5-w2) | rewards verified | OLMO only, no rewards |
| EXP-OLMO-2 | W2 | [МЛ-5+6](08_ml.md#мл5-w2) | rewards verified | OLMO + OS rewards |
| EXP-OLMO-3 | W2 | [МЛ-5+6](08_ml.md#мл5-w2) | rewards verified | Math + OS rewards |
| EXP-OLMO-4 | W2 | [МЛ-5+6](08_ml.md#мл5-w2) | rewards verified | Mix (math + OLMO) + OS rewards |
| **Reward aggregation** | | | | |
| EXP-AGG-1 | W4 | [МЛ-4](08_ml.md#мл1-w4) | OS+determ | Weighted sum baseline |
| EXP-AGG-2 | W4 | [МЛ-5](08_ml.md#мл5-w4) | OS+determ | GDPO decoupled |
| EXP-AGG-5 | W4 | [Дм-1](07_dima_infra.md#дм1-w4) | safety veto | Hierarchical (safety+GDPO) |
| **Process reward** | | | | |
| EXP-PRM-1 | W3 | [Э-3](01_eldar_research_sft.md#э3-w3) | PRM annotations | Baseline step-error + cross-val |
| EXP-PRM-2 | W4 | [Э-3](01_eldar_research_sft.md#э3-w4) | PRM train data | Train PRM 0.6B LoRA |
| EXP-PRM-3 | W5 | [Э-1+2](01_eldar_research_sft.md#э1-w5) | PRM-2 done | Outcome vs outcome+process |
| EXP-PRM-4 | W6 | [Э-1](01_eldar_research_sft.md#э1-w6) | PRM-3 done | Partial credit vs veto vs decouple |
| **120B** | | | | |
| EXP-120B-1 | W6 | [МЛ-7](08_ml.md#мл7-w6) | best 10B ckpt | Scale to 120B, full eval |
| **Reward hacking** | | | | |
| EXP-HACK-1 | W5-6 | [П-6](05_polya_product.md#п6-w6) | dashboard | Correlation per 20 steps |
| EXP-HACK-2 | W5 | [Дм-3](07_dima_infra.md#дм3-w5) | Arena reward | ODIN debiasing |
| EXP-HACK-3 | W6 | [Дм-1](07_dima_infra.md#дм1-w6) | RL v1 | PAR reward shaping |
| EXP-HACK-4 | W6 | [Дм-2](07_dima_infra.md#дм2-w6) | RL v1 | Gradient reg vs KL |
| **B2C functions** | | | | |
| EXP-B2C-1 | W2 | [Д-1+2](06_dasha_platform.md#д1-w2) | tool gen | Baseline: call dist, loop |
| EXP-B2C-2 | W3 | [Д-1](06_dasha_platform.md#д1-w3) | interp rw | Interpreter reward v1 |
| EXP-B2C-3 | W3 | [Д-2](06_dasha_platform.md#д2-w3) | search rw | Websearch decomposition v1 |
| EXP-B2C-4 | W4 | [Д-1](06_dasha_platform.md#д1-w4) | B2C-2 | Max tool calls |
| EXP-B2C-5 | W4 | [Д-2](06_dasha_platform.md#д2-w4) | B2C-3 | Cross-tool |
| **Tool efficiency** | | | | |
| EXP-TOOL-EFF-1 | W4 | [В-1](03_vova_functions.md#в1-w4) | tool rw v1 | Call distribution |
| EXP-TOOL-EFF-2 | W5 | [В-1](03_vova_functions.md#в1-w5) | EFF-1 | OTC-PO |
| EXP-TOOL-EFF-3 | W5 | [В-1](03_vova_functions.md#в1-w5) | EFF-1 | ToolRLA multiplicative |
| **Loops** | | | | |
| EXP-LOOP-2 | W6 | [В-1](03_vova_functions.md#в1-w6) | SpecRA | Hard penalty |
| EXP-LOOP-3 | W6 | [В-1](03_vova_functions.md#в1-w6) | loop-2 | Soft penalty |
| **Factuality** | | | | |
| EXP-FACT-2 | W5-6 | [П-6](05_polya_product.md#п6-w5) | NLI | NLI factuality reward |
| EXP-FACT-3 | W5 | [Д-2](06_dasha_platform.md#д2-w5) | search rw | Search-grounded eval |
| **ML training** | | | | |
| Round 3 clip | W1-2 | [МЛ-1+2](08_ml.md#мл1-w1) | verl | 3 clip configs |
| Round 3 dr.grpo | W1-2 | [МЛ-3+4](08_ml.md#мл3-w1) | verl | DR-GRPO |
| DPPO | W3-4 | [МЛ-1+2](08_ml.md#мл1-w3) | OOM fix | Trust-region mask |
| IS-correction | W3-4 | [МЛ-5+6](08_ml.md#мл5-w3) | running | Collapse zone |
| Advantage sampler | W3-4 | [МЛ-3+4](08_ml.md#мл3-w3) | plateau | EMA-based sampler |
| First real RL | W3 | [МЛ-5+6](08_ml.md#мл5-w3) | rewards verified | Math + verified rewards |

---

## 5. Топ-10 рисков

| # | Риск | P | Impact | Митигация | Owner |
|---|------|---|--------|-----------|-------|
| 1 | Collapse не решён Round 3 | Medium | Critical | DPPO + sampler + IS-correction | ML |
| 2 | Safety данных 0 → не успеем к W5 | High | Critical | ЦУР fallback + InternLM2 OS | Поля-3 |
| 3 | SFT не бьёт distill (W4 gate) | Medium | Critical | Больше данных, masking, compute | Эльдар |
| 4 | Distill pipeline не готов к W2 | Medium | High | Single-teacher fallback, catch-up W3 | Эльдар-1 |
| 5 | OLMO baseline не даёт signal | Low | High | Determ-only fallback, math-only | ML |
| 6 | Arena 28k разметка медленная | Medium | High | Fast batch 5k; crowd | Поля-4 |
| 7 | GPU capacity для OS rewards | Medium | Medium | Приоритет 3 из 5 | Дима-2 |
| 8 | SWE-bench infra недоступна | Medium | Medium | Synthetic tasks | Вова-2 |
| 9 | Reward hacking multi-reward | Medium | High | Monitoring + ODIN + PAR | Дима+ML |
| 10 | 120B infra недоступна | Medium | Medium | Отложить eval на W8 | МЛ-7 |

---

## 6. Критические решения по неделям

| Неделя | Решение | Кто решает | Данные для решения |
|--------|---------|------------|-------------------|
| W1 | **Какие OS rewards работают?** | Дима + ML lead | EXP-RWVER-1 |
| W2 | Лучший clip + dr.grpo конфиг | ML lead | Round 3 metrics |
| W2 | **OLMO config: лучший signal?** | ML lead | EXP-OLMO-1..4 |
| W3 | Стратегия маскирования SFT | Эльдар | EXP-SFT-5 |
| W3 | **OLMO mix в real RL?** | ML lead | OLMO vs math-only |
| W4 | **SFT GATE: go/no-go** | Эльдар + ML lead | EXP-SFT-1..4 |
| W4 | Формула агрегации reward | Дима + ML lead | EXP-AGG-1/2/5 |
| W5 | **120B checkpoint selection** | ML lead | RL v0 metrics |
| W5 | Domain mix proportions | Иван + ML lead | Ablation |
| W6 | PRM strategy | Эльдар | EXP-PRM-4 |
| W7 | **Promotion thresholds** | All leads | Eval sweep + 120B |
| W8 | **RELEASE DECISION** | All leads + руководитель | Gate 5 |
