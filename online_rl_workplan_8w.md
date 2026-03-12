# Online RL: План работ на 8 недель (Mar 17 — May 9, 2026)

> **Разбитая версия по командам:** [detailed_plan/](detailed_plan/00_overview.md) — обзор + файлы по каждой команде с перекрёстными ссылками.

> Последнее обновление: 2026-03-11
> Цель: от текущего состояния к production-ready модели за 8 недель.
> Связанные документы: [gap analysis](online_rl_ml_data_gap_analysis.md), [команды](online_rl_teams.md), [индекс](online_rl_index.md).

---

## Оглавление

- [0. Путь модели: milestone-ы](#section-0)
- [1. Gate checklists](#section-1)
- [2. Команды и нотация](#section-2)
- [3. Чек-листы по руководителям](#section-3)
  - [Эльдар](#lead-eldar) | [Иван](#lead-ivan) | [Вова](#lead-vova) | [Серёжа](#lead-sereja)
  - [Поля](#lead-polya) | [Даша](#lead-dasha) | [Дима](#lead-dima) | [ML](#lead-ml)
- [4. Недельный план (детально)](#section-4)
  - [W1](#w1) | [W2](#w2) | [W3](#w3) | [W4](#w4) | [W5](#w5) | [W6](#w6) | [W7](#w7) | [W8](#w8)
- [5. Загрузка по людям](#section-5)
- [6. Карта экспериментов](#section-6)
- [7. Топ-10 рисков](#section-7)
- [8. Критические решения по неделям](#section-8)

---

<a id="section-0"></a>

## 0. Путь модели: milestone-ы

### Pipeline

```
Stage 1.5 (multi-teacher distill) -> SFT (masking + annotation) -> Online RL (rewards)
```

> DPO отложен на пост-8-недельный план. См. [gap analysis 0c+](online_rl_ml_data_gap_analysis.md).

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

> RL — ключ к math/code/tools/arena. Safety рампится через стадии.
> Знак "=" означает: эта стадия не должна менять метрику.
> Конкретные targets ставим после замера baselines в W1.
> **Сравниваем с Qwen3.5-2B** (2B dense, ближайший по active params к нашим 1.8B) и **Qwen3.5-4B** (stretch target).

---

<a id="section-1"></a>

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

<a id="section-2"></a>

## 2. Команды и нотация

| Команда | Лид | Человек | Нотация | Основной scope |
|---------|-----|---------|---------|---------------|
| Research | Эльдар | 3 | Эльдар-1, -2, -3 | SFT reasoning + distill pipeline, PRM, SFT annotation |
| Research | Иван | 3 | Иван-1, -2, -3 | Stage 1.5 reasoning, IFEval, всё IF |
| Functions | Вова | 2 | Вова-1, -2 | Tools + agents + SWE, ToolRM |
| Code | Серёжа | 2 | Серёжа-1, -2 | Code reasoning, code reward |
| Product | Поля | 6 | Поля-1..6 | B2C reward, safety, classifier |
| Platform | Даша | 4 | Даша-1..4 | B2C функции, reward для них |
| Infra | Дима | 3 | Дима-1, -2, -3 | Хранение, LoRA инфра, determ checks |
| ML | — | 7+ | МЛ-1..7+ | RL pipeline, эксперименты, eval, 120B |

**Всего:** 23 data + 7+ ML = 30+ человек.

---

<a id="section-3"></a>

## 3. Чек-листы по руководителям

Ниже — персональные таблицы для каждого лида. Каждая строка = одна неделя, задачи по команде, DoD, статус.
Ссылки ведут на детальную разбивку в [Секции 4](#section-4).

---

<a id="lead-eldar"></a>

### Эльдар (Research: SFT + Distill Pipeline + PRM + Annotation) — 3 человека

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | **Distill pipeline** multi-teacher + rejection sampling (Э-1), SFT data из OS: 20k math+code+IF (Э-2) + annotation v0, PRM: ProcessBench baseline + annotation start protocol 3c (Э-3) | Pipeline v0 на 100 задачах; >= 20k SFT annotated; ProcessBench accuracy | Pipeline demo; SFT distribution table; ProcessBench vs competitors | - [ ] |
| [W2](#w2) | Distill pipeline → production quality (Э-1), SFT +30k с annotation (MD/RU/logic/Arena) (Э-2), PRM prompt eng + test set protocol 3c (Э-3) | Pipeline beats single-teacher audit; >= 50k annotated; PRM test set | **Quality audit: pipeline vs single-teacher** | - [ ] |
| [W3](#w3) | SFT masking ablation: 4 стратегии (Э-1+2), EXP-PRM-1 cross-val + train gen (Э-3) | Лучшая маскировка; >= 80k annotated; PRM cross-val | **Masking: strategy x benchmark table** | - [ ] |
| [W4](#w4) | SFT final 100-200k annotated (Э-1), EXP-SFT-2/3/4: distill comparison (Э-2), PRM 0.6B LoRA training (Э-3) | **GATE: SFT бьёт distill**; PRM LoRA trained | **SFT vs distill gate: PASS/FAIL** | - [ ] |
| [W5](#w5) | PRM ablation: outcome vs outcome+process (Э-1+2), SFT promotion check (Э-3) | EXP-PRM-3 результаты; SFT sign-off | PRM impact на AIME: delta | - [ ] |
| [W6](#w6) | PRM strategy: partial credit vs veto vs decouple (Э-1), cross-domain regression (Э-2+3) | PRM best strategy; regression matrix clean | **PRM final choice + regression** | - [ ] |
| [W7](#w7) | ProcessBench/PRMBench final (Э-1), math gate check (Э-2), cross-domain final (Э-3) | math_500 + AIME targets; cross-domain clean | **Math gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final math gate sign-off, release readiness | Все math метрики GREEN | Sign-off document | - [ ] |

**Ключевые риски Эльдара:**
- Distill pipeline не готов к W2 → SFT W2 на single-teacher data, потребуется re-generate
- SFT не бьёт distill → STOP, добавить данные, пересмотреть masking
- PRM не улучшает AIME → откатиться к outcome-only, не блокирует release

---

<a id="lead-ivan"></a>

### Иван (Research: Stage 1.5 + IFEval) — 3 человека

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Solve First: math 200k (И-1), code 100k (И-2), dedup/contamination + **leak check** (И-3) | >= 300k Stage 1.5; dedup отчёт; **leak report** | Отчёт: overlap %, clean dataset, leak status | - [ ] |
| [W2](#w2) | Solve First: code +200k, IF/Arena/Safety +100k (И-1+2), contamination fixes (И-3) | >= 600k total; zero overlap с eval | **Stage 1.5: 600k milestone** | - [ ] |
| [W3](#w3) | STEM/B2C/RU expansion +400k (И-1+2), IFEval reward setup (И-3) | >= 1M total; IFEval checker P/R | **Stage 1.5: 1M milestone** | - [ ] |
| [W4](#w4) | Stage 1.5 finalization 2M (И-1+2), IFEval+Vikacheck integration, dynamic sampling proto (И-3) | **2M target**; dynamic sampling works | **GATE: Stage 1.5 2M done** | - [ ] |
| [W5](#w5) | Domain mix ablation (И-1), IFEval strict + IF reward ablation (И-2), difficulty sampler (И-3) | Optimal mix найден; IFEval delta с/без reward | **Domain mix: best proportions** | - [ ] |
| [W6](#w6) | Mixed sampler (И-1), dynamic sampling impl (И-2), OBM format validation 50 tasks (И-3) | OBM format >= 90% | OBM format gate check | - [ ] |
| [W7](#w7) | IFEval gate check (И-1), Vikacheck gate (И-2), promotion thresholds proposal (И-3) | IFEval strict >= target; thresholds documented | **IF gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final IF gate sign-off, release readiness | Все IF метрики GREEN | Sign-off document | - [ ] |

**Ключевые риски Ивана:**
- Solve First throughput < 100k/день → параллелить GPU, synthetic fallback
- Stage 1.5 quality drift → weekly audit 100 random per domain

---

<a id="lead-vova"></a>

### Вова (Functions: Tools + Agents + SWE) — 2 человека

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | BFCL v4 baseline + ToolRM-1.7B (В-1), tool specs catalog 50+ (В-2) | BFCL score; ToolRM online; 50 specs | BFCL score + ToolRM latency table | - [ ] |
| [W2](#w2) | ToolRM на BFCL + error analysis (В-1), SWE-bench baseline 10b (В-2) | ToolRM distribution; SWE mean@1 | **Top-5 tool error types** | - [ ] |
| [W3](#w3) | Tool reward v1: AST + execution (В-1), SWE-smith pipeline setup (В-2) | Tool reward P/R; SWE-smith running | Tool reward v1 metrics | - [ ] |
| [W4](#w4) | Tool efficiency baseline EXP-TOOL-EFF-1 (В-1), agent synthesis start (В-2) | Call distribution + redundancy rate | EXP-TOOL-EFF-1 report | - [ ] |
| [W5](#w5) | OTC-PO efficiency reward (В-1), agent trajectories 100+ multi-turn (В-2) | EXP-TOOL-EFF-2/3 results; 100+ trajectories | **Tool efficiency: -X% calls, accuracy =** | - [ ] |
| [W6](#w6) | Loop penalty tuning EXP-LOOP-2/3 (В-1), SWE-smith 100+ instances (В-2) | Optimal penalty; SWE data expanded | Loop penalty: hard vs soft results | - [ ] |
| [W7](#w7) | BFCL gate (В-1), agent gate: SWE + BrowseComp (В-2) | BFCL >= target; tool suppression <= 5% | **Tools/Agent gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final tools/agent sign-off | All tools метрики GREEN | Sign-off document | - [ ] |

**Ключевые риски Вовы:**
- SWE-bench infra зависит от Вадима → проверить W1 Day 1
- ToolRM-1.7B недостаточно для наших задач → fallback к AST-based checker

---

<a id="lead-sereja"></a>

### Серёжа (Code: reasoning + reward) — 2 человека

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | LCB v6 baseline + sandbox (С-1), code rollout queries 10k+ (С-2) | LCB pass@1; sandbox works; 10k queries | **LCB baseline vs Qwen3.5-2B/4B** | - [ ] |
| [W2](#w2) | Code reward pipeline: sandbox + LLM-judge (С-1), queries с domain labels (С-2) | Pipeline P/R; 10k+ с метками | Code reward v0 metrics | - [ ] |
| [W3](#w3) | Code reward v1: execution + judge (С-1), difficulty filtering P(mixed) (С-2) | Code reward P/R improved; difficulty labels | Code RL set: 10k+ filtered | - [ ] |
| [W4](#w4) | Code RL set final (С-1), first canary RL 20-30 steps (С-2) | Set ready; canary metrics | **First code RL canary: LCB delta** | - [ ] |
| [W5](#w5) | Code RL 50+ steps (С-1), VeRPO dense reward test (С-2) | 50 step metrics; VeRPO comparison | Code RL: LCB trend | - [ ] |
| [W6](#w6) | Code RL scaling 100+ steps (С-1), code+math joint (С-2) | 100+ step metrics; joint regression check | **Code RL scaling: LCB at 100 steps** | - [ ] |
| [W7](#w7) | LCB gate check (С-1), SWE gate + code regression (С-2) | LCB >= target; no regression | **Code gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final code sign-off | All code метрики GREEN | Sign-off document | - [ ] |

**Ключевые риски Серёжи:**
- Sandbox security → gVisor/Firecracker
- Code reward FP (награда за невалидный код) → execution-first, judge-fallback

---

<a id="lead-polya"></a>

### Поля (Product: B2C reward + Safety) — 6 человек

| W | Кто | Задача | DoD | Status |
|---|-----|--------|-----|--------|
| **[W1](#w1)** | П-1 | MD-check тест >= 800 → ОБМ | Файл в ОБМ, distribution по типам | - [ ] |
| | П-2 | Смена языков: завершить трейн + тест 500+ | Train >= 1.5k, test >= 500 | - [ ] |
| | П-3 | Safety: инструкция + пилот 200, IAA | Инструкция утверждена, IAA > 0.8 | - [ ] |
| | П-4 | Arena 28k → разметку, отрезать тест 2k | Разметка запущена, ETA | - [ ] |
| | П-5 | Русификатор → формат reward >= 5k | 5k в формате, reasoning quality ok | - [ ] |
| | П-6 | Vikacheck-assistant → формат reward | Train >= 800, test >= 350 | - [ ] |
| **[W2](#w2)** | П-1 | MD-check train → reward training prep | >= 3k reward train MD | - [ ] |
| | П-2 | Русификатор тест расширить до 500+ | Test >= 500 | - [ ] |
| | П-3 | Safety разметка → 1000+ (если IAA ok) | >= 1000 с domain labels | - [ ] |
| | П-4 | Arena контроль разметки (30%+) | >= 30% done, quality ok | - [ ] |
| | П-5 | **Эквивалентность reward LoRA** (data ready!) | **LoRA обучена, P/R/F1 на тесте** | - [ ] |
| | П-6 | Vikacheck-reasoning: SFT + тест markup | >= 500 данных, разметка запущена | - [ ] |
| **[W3](#w3)** | П-1 | MD-check reward LoRA training + тест | **MD LoRA acc ~0.94 (match prev)** | - [ ] |
| | П-2 | Русификатор reward LoRA training | LoRA обучена, metrics | - [ ] |
| | П-3 | Safety: сборка 1000+ train + 400+ test | Train >= 1000, test >= 400 | - [ ] |
| | П-4 | Arena markup 60%+, routing classifier start | 60% done; classifier prototype | - [ ] |
| | П-5 | Ты/Вы: финализация train + тест 500+ | Train >= 2k, test >= 500 | - [ ] |
| | П-6 | Classifier для reward routing (с командой Ивана) | Classifier v0 на тестовых примерах | - [ ] |
| **[W4](#w4)** | П-1 | Safety reward LoRA training | LoRA обучена, **P >= 99% target** | - [ ] |
| | П-2 | Arena: собрать 2k тест из разметки | Test 2k стратифицирован | - [ ] |
| | П-3 | Смена языков reward LoRA | LoRA обучена, metrics | - [ ] |
| | П-4 | Ты/Вы reward LoRA | LoRA обучена, metrics | - [ ] |
| | П-5 | **Все LoRA validation (протокол 6a)** | **Таблица: LoRA x test → P/R/F1** | - [ ] |
| | П-6 | Routing classifier v1 | Classifier accuracy на тесте | - [ ] |
| **[W5](#w5)** | П-1 | Arena reward LoRA (28k train ready) | Arena LoRA metrics | - [ ] |
| | П-2 | Safety validation: precision >= 99% | **Safety P >= 99% confirmed** | - [ ] |
| | П-3 | Все test sets >= 500 verified | Checklist: all >= 500 | - [ ] |
| | П-4 | Routing classifier deployed | Classifier в pipeline | - [ ] |
| | П-5 | B2C integration test (all checkers on 1 batch) | All checkers pass smoke test | - [ ] |
| | П-6 | NLI factuality reward integration | EXP-FACT-2 metrics | - [ ] |
| **[W6](#w6)** | П-1 | Fix reward regressions from W5 | Regressions fixed | - [ ] |
| | П-2 | Expand worst rewards (add data, retrain) | Worst reward improved | - [ ] |
| | П-3 | **Safety per-domain breakdown** (7 domains) | **Per-domain compliance table** | - [ ] |
| | П-4 | Human eval prep (100-200 RU, overlap 3+) | Human eval started | - [ ] |
| | П-5 | Reward ensemble validation (own+OS vs own-only) | Ablation: own vs own+OS delta | - [ ] |
| | П-6 | Reward hacking monitoring (EXP-HACK-1) | Correlation(reward, eval) per 20 steps | - [ ] |
| **[W7](#w7)** | П-1 | Final reward audit (все LoRA re-measured) | Updated P/R/F1 table | - [ ] |
| | П-2 | Arena swap canary (own vs Skywork-8B) | Canary result | - [ ] |
| | П-3 | Safety swap canary (own vs InternLM2) | Canary result | - [ ] |
| | П-4 | Human eval results analysis | Human eval score | - [ ] |
| | П-5 | Factuality gate (SimpleQA) | SimpleQA >= threshold | - [ ] |
| | П-6 | Final B2C metrics assembly | All B2C numbers | - [ ] |
| **[W8](#w8)** | П-1..3 | **Safety gate final** (refusal, false refusal, jailbreak, per-domain, ЦУР) | **Safety gate: PASS/FAIL** | - [ ] |
| | П-4..6 | Factuality + human eval + sign-off | All product метрики GREEN | - [ ] |

**Ключевые риски Поли:**
- Safety с нуля (0 данных) → ЦУР fallback + InternLM2 OS reward как страховка
- Arena 28k разметка медленная (4+ недель) → приоритизировать 5k fast batch
- Русификатор reasoning генерация некачественная → кросс-валидация 2 моделей (протокол 3c)

---

<a id="lead-dasha"></a>

### Даша (Platform: B2C функции + reward) — 4 человека

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Interpreter reward setup (Д-1), websearch reward setup (Д-2), text2image trigger (Д-3), B2C baseline report (Д-4) | P/R/F1 per checker на 100 примерах; baseline report | **3 checker P/R/F1 + baseline report** | - [ ] |
| [W2](#w2) | B2C baselines EXP-B2C-1 (Д-1+2), reward train data 500+ per function (Д-3+4) | Full baseline report; >= 500 examples per function | EXP-B2C-1 numbers | - [ ] |
| [W3](#w3) | Interpreter reward v1 EXP-B2C-2 (Д-1), websearch decomposition v1 EXP-B2C-3 (Д-2), text2image reward (Д-3), B2C RL data prep (Д-4) | EXP-B2C-2/3 metrics; text2image P/R | **Interpreter reward: code_success_pct delta** | - [ ] |
| [W4](#w4) | Max tool calls EXP-B2C-4 (Д-1), cross-tool EXP-B2C-5 (Д-2), websearch NLI (Д-3), B2C reward integration (Д-4) | EXP-B2C-4/5 results; NLI accuracy | Tool productivity improvement | - [ ] |
| [W5](#w5) | NLI fact-check integration (Д-1), search grounding eval EXP-FACT-3 (Д-2), B2C function canary 20-30 steps (Д-3+4) | EXP-FACT-3 offline; canary metrics | **B2C canary: interpreter + search RL** | - [ ] |
| [W6](#w6) | B2C RL scaling 100+ steps (Д-1+2), URL tracking + max calls opt (Д-3+4) | 100+ step metrics; fabricated URL rate down | B2C RL at 100 steps | - [ ] |
| [W7](#w7) | B2C function gate: interpreter, search, text2image (Д-1+2), latency check (Д-3+4) | Success rates >= target; latency ok | **B2C gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final B2C function sign-off | All function метрики GREEN | Sign-off document | - [ ] |

**Ключевые риски Даши:**
- Websearch NLI дорогой в online RL → keyword matching v0, NLI для gate eval offline
- Fabricated URLs от модели → hard penalty -1.0

---

<a id="lead-dima"></a>

### Дима (Infra) — 3 человека

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Storage + pipeline (Д-1), OS rewards x5 deploy + **verification tests** (Д-2), determ checks v1 (Д-3) | Storage works; >= 3 OS models online + verification on 100 examples; checks API P >= 95% | **Infra operational + OS reward verification** | - [ ] |
| [W2](#w2) | Ingest all datasets (Д-1), OS reward full matrix (Д-2), LoRA training pipeline (Д-3) | All data in storage; 5 models x 5+ tests; LoRA pipeline works | **OS reward baseline matrix** | - [ ] |
| [W3](#w3) | Determ rewards → RL pipeline integration (Д-1), MD+Русификатор LoRA training (Д-2), reward_config auto-assign (Д-3) | Determ in RL; LoRAs training; rule-based classifier | LoRA training launched | - [ ] |
| [W4](#w4) | Reward aggregation: Hierarchical (safety veto + GDPO) (Д-1), multi-reward pipeline (Д-2), canary infra (Д-3) | Aggregation works; canary infra ready | **Aggregation v1: EXP-AGG results** | - [ ] |
| [W5](#w5) | Production reward pipeline (Д-1), monitoring dashboard (Д-2), ODIN debiasing EXP-HACK-2 (Д-3) | Pipeline prod-grade; dashboard live | **Monitoring: reward vs eval correlation** | - [ ] |
| [W6](#w6) | PAR EXP-HACK-3 (Д-1), gradient reg EXP-HACK-4 (Д-2), scaling test 200+ (Д-3) | Reward hacking mitigation results | Scaling: RL 200+ steps stable | - [ ] |
| [W7](#w7) | Prod deploy prep (Д-1), eval sweep automation (Д-2), **competitor benchmark Qwen3.5-2B/4B + GLM-5 + DS** (Д-3) | Model serving ready; auto eval; **competitor table** | **Eval sweep: us vs competitors** | - [ ] |
| [W8](#w8) | Canary 48h (Д-1), monitoring prod-like (Д-2), rollback docs (Д-3) | Canary stable 48h | **Canary 48h: PASS/FAIL** | - [ ] |

**Ключевые риски Димы:**
- GPU capacity для 5 OS reward моделей → приоритет: Skywork-0.6B + ArmoRM-8B + InternLM2-7B
- LoRA pipeline нестабилен → тестировать на 100 записях перед production

---

<a id="lead-ml"></a>

### ML (параллельный трек) — 7+ человек

| W | Задача | DoD | Результат недели | Status |
|---|--------|-----|-----------------|--------|
| [W1](#w1) | Round 3: clip range 3 configs (МЛ-1+2), dr.grpo (МЛ-3+4), pass@k fix (МЛ-5), **OS reward verification** test design + launch (МЛ-6+7) | Round 3 запущен; pass@k PR; reward verification results | Round 3 first 50+ steps; **OS reward: work/don't work per model** | - [ ] |
| [W2](#w2) | Round 3 analysis: best config (МЛ-1..4), **OLMO RL baseline** 4 configs (МЛ-5+6), dev prompt 1k (МЛ-7) | **Best clip + dr.grpo config**; **OLMO RL: 4 configs done** | **Round 3 results + OLMO baseline table** | - [ ] |
| [W3](#w3) | DPPO ablation (МЛ-1+2), advantage sampler (МЛ-3+4), **first real RL canary** math + verified OS rewards 20-30 steps (МЛ-5+6), Cold Start v0 training (МЛ-7) | DPPO results; sampler metrics; **RL canary: math+code delta** | **DPPO + first real RL canary** | - [ ] |
| [W4](#w4) | First multi-reward RL (МЛ-1..3), EXP-AGG-1/2/5 (МЛ-4+5), SFT training (МЛ-6+7) | Multi-reward RL metrics; aggregation comparison; **SFT trained** | **Multi-reward RL + SFT gate** | - [ ] |
| [W5](#w5) | RL v0 extended 100+ steps (МЛ-1..4), rollout replay RLEP (МЛ-5+6), **120B checkpoint selection** (МЛ-7) | RL v0 at 100+; RLEP comparison; **120B candidate chosen** | **RL v0 + 120B prep** | - [ ] |
| [W6](#w6) | RL v1 extended 300+ steps (МЛ-1..4), collapse monitoring + guardrails (МЛ-5+6), **120B eval sweep** (МЛ-7) | **300+ steps no collapse**; **120B eval results** | **RL v1 at 300+ steps + 120B eval** | - [ ] |
| [W7](#w7) | Best checkpoint selection (МЛ-1+2), final config frozen (МЛ-3+4), canary 48h start (МЛ-5..7) | Config frozen; canary started; **120B results documented** | **Canary started** | - [ ] |
| [W8](#w8) | Canary 48h analysis (МЛ-1..3), promotion decision (МЛ-4+5), post-release plan (МЛ-6+7) | **PROMOTION: YES/NO** | **Release candidate** | - [ ] |

**Ключевые риски ML:**
- Collapse не решён Round 3 → DPPO/sampler/IS-correction как backup
- OLMO baseline не даёт learning signal → проверить rewards, determ-only fallback
- Multi-reward нестабилен → убирать rewards по одному, binary search
- 120B infra недоступна → план B: eval только top-2 checkpoint, отложить на W8

---

<a id="section-4"></a>

## 4. Недельный план (детально)

<a id="w1"></a>

### W1 (Mar 17-21): Foundation — OS reward verification, distill pipeline

**Цель:** Развернуть инфраструктуру. Зафиксировать baseline по ВСЕМ трекам. Верифицировать OS rewards. Построить distill pipeline. Запустить leak check. Начать PRM.

**Почему первая:** Без инфры данные на личных машинах. Без baselines нельзя мерить прогресс. Без verified rewards нельзя запускать RL. Distill pipeline — blocking для качества SFT данных. Всё это быстрые задачи без исследовательских рисков.

**Результат недели (модель):** Модель = Base (без изменений). Все baseline числа зафиксированы. OS rewards verified. Distill pipeline v0.

#### Infra/Дима

**Дима-1: Хранилище данных + pipeline ингеста**
- Вход: Требования из [online-rl.md](start_pack/online-rl.md). Данные разбросаны по `/home/valenchik/`, train_catalog, notebooks.
- Задача: (1) Развернуть S3/MinIO. (2) Бакеты: rollout-queries, reward-train, reward-test, stage15, sft. (3) CLI upload/download. (4) Формат per-sample reward_config (JSON из gap analysis 2.4). (5) Тест на 100 записей.
- DoD: endpoint доступен, round-trip на 100 записей, README.
- Day 1: выбор S3/MinIO. Day 2-3: CLI + формат. Day 4: тестовые данные. Day 5: README + тест с командами.
- Риск: выбор → дефолт MinIO (self-hosted, S3-compatible).

**Дима-2: OS reward модели (5 штук) + verification**
- Вход: Skywork-8B, Skywork-0.6B, ArmoRM-8B, InternLM2-7B, ToolRM-1.7B.
- Задача: Для каждой: скачать → serving (vLLM) → API → latency test. ArmoRM: 19 dimensions, нужен handler. **Плюс: прогнать каждый reward на 100 labeled примерах из наших данных (math, code, IF) → P/R/F1 → решение: work/don't work.**
- DoD: Все 5 online. Таблица: model | latency | GPU | P/R/F1 per domain. Golden test. **Verification: какие rewards реально различают good/bad.**
- Риск: GPU capacity → приоритет: Skywork-0.6B + ArmoRM-8B + InternLM2-7B (3 must-have).

**Дима-3: Детерминированные чеки v1**
- Вход: Спеки из gap analysis 4.5.
- Задача: FastAPI: (1) loop_detected (n-gram >= 3), (2) repetition_ratio, (3) json_valid, (4) latex_valid. Тест 50 примеров.
- DoD: API works, P >= 95% на 50 примерах, latency < 50ms.
- Риск: SpecRA сложный → начать с n-gram, SpecRA как v2.

#### Research/Эльдар

**Эльдар-1: Multi-teacher distill pipeline (BLOCKING)**
- Вход: Доступ к API Qwen-3, Gemini-Pro-3.5-preview, DeepSeek-V3.2.
- Задача: Построить pipeline: (1) Для каждой задачи генерировать решения из 3+ учителей. (2) Cross-verification: ответ верифицируется остальными, keep если >= 2 согласны. (3) Best-of-N: N решений, скоринг OS rewards (ArmoRM, Skywork), выбрать лучший. (4) Rejection sampling: фильтр по execution/format checks. (5) Quality ranking: pair-wise между учителями.
- DoD: Pipeline script ready. На 100 тестовых задачах pipeline даёт measurable quality gain vs single-teacher distill.
- Day 1-2: API wrappers для 3 учителей, generation module. Day 3: cross-verification + rejection sampling logic. Day 4: Best-of-N + quality ranking. Day 5: end-to-end test на 100 задачах, quality audit.
- Риск: API latency/cost → batch processing, cache, начать с 2 учителей если 3-й недоступен.

**Эльдар-2: SFT data collection из opensource + annotation v0**
- Вход: MATH, GSM8K, OpenCodeInstruct, ODA-Mixture, IFEval seeds, WildChat RU.
- Задача: Собрать >= 20k SFT (10k math+code, 5k IF, 5k B2C/STEM). Для каждого sample: annotation v0 = (1) MD check (deterministic, Дима-3 API), (2) logic/correctness (execution check для кода, answer match для math), (3) Arena preference (LLM-as-judge: Qwen-3.5 vs DeepSeek-V3.2 pair-wise). RU quality annotation добавим W2+ когда Русификатор будет готов.
- DoD: >= 20k annotated, distribution table source x domain x count.
- Риск: LLM-as-judge дорогой → batch 1k/день, приоритизировать math+code.

**Эльдар-3: PRM — ProcessBench baseline + annotation start (protocol 3c step 1-2)**
- Вход: ProcessBench (3400 задач), 10b_toolmind.
- Задача: (1) Замерить ProcessBench accuracy нашей модели. (2) Comparison table vs competitors. (3) Начать сбор PRM annotations: промпт для step-level reward → пилот 200 примеров → IAA.
- DoD: Accuracy зафиксирована, comparison table, PRM prompt draft, пилот 200.
- Риск: IAA < 0.8 → упростить промпт, binary step labels.

#### Product/Поля

**Поля-1: MD-check тест → ОБМ**
- Вход: ~4k размечено, ~15-20k прогонов.
- Задача: Выгрузить → фильтрация → стратификация → тест >= 800 (50/50) → ОБМ.
- DoD: >= 800 в ОБМ + distribution table.

**Поля-2: Смена языков трейн + тест 500+**
- Вход: Thinking 1086, Answer 804 (в разметке), тест ~300.
- Задача: Забрать answer → объединить → формат reward → расширить тест +200.
- DoD: Train >= 1.5k, test >= 500.

**Поля-3: Safety инструкция + пилот**
- Вход: НИЧЕГО (P0 gap).
- Задача: Инструкция (бинарная: safe/unsafe, 7 доменов) → seed 200 → пилот, перекрытие 5.
- DoD: Инструкция утверждена, IAA > 0.8.

**Поля-4: Arena 28k → разметка**
- Вход: 28k в train_catalog (MR 317).
- Задача: Подготовить pair-wise → стратификация → разметка, перекрытие 3 → тест 2k.
- DoD: Разметка запущена, 2k тест отложен, ETA.

**Поля-5: Русификатор → reward формат**
- Вход: ~7.6k в разметке, LoRA train 3239.
- Задача: Забрать → reward формат → reasoning (протокол 3c) → >= 5k reward train.
- DoD: >= 5k, reasoning quality on 50 samples, correlation >= 0.9.

**Поля-6: Vikacheck-assistant → reward**
- Вход: 1150 тест, разметка пробежала.
- Задача: Формат reward → 2 реварда → 800 train + 350 test.
- DoD: Train >= 800 x2, test >= 350 x2.

#### Research/Иван

**Иван-1: Solve First — math 200k**
- Вход: Solve First pipeline, teacher Qwen-3, seeds: MATH 12k + GSM8K 8k + AIME + Olympiad.
- DoD: >= 200k math, quality audit 100 random.

**Иван-2: Solve First — code 100k**
- Вход: HumanEval 164 + MBPP 500 + LCB seeds + competitive programming.
- DoD: >= 100k code, execution validation 50 random.

**Иван-3: Dedup/contamination + LEAK CHECK**
- Вход: Все rollout queries + eval benchmarks + reward sets.
- Задача: Exact + fuzzy dedup (MinHash + LSH, threshold 0.85). **Плюс: leak check — проверить что текущий тренировочный датасет не содержит eval data.**
- DoD: Отчёт overlap %, leak status, список зараженных sample IDs, скрипт.

#### Functions/Вова

**Вова-1: BFCL v4 + ToolRM-1.7B**
- DoD: BFCL overall + multi-turn; ToolRM online + latency.

**Вова-2: Tool specs 50+ + SWE baseline**
- DoD: 50 specs JSON; SWE mean@1 с 10b.

#### Code/Серёжа

**Серёжа-1: LCB v6 + sandbox**
- DoD: Sandbox works; LCB pass@1; **comparison table vs Qwen3.5-2B/4B.**

**Серёжа-2: Code queries 10k+**
- DoD: 10k+ с domain + difficulty labels, в хранилище.

#### Platform/Даша

**Даша-1: Interpreter reward checker** — DoD: P/R на 100 примеров.
**Даша-2: Websearch reward checker** — DoD: Per-component P/R на 100.
**Даша-3: Text2image trigger checker** — DoD: Trigger P/R на 100.
**Даша-4: B2C baseline report** — DoD: Report: call distribution, loop rate, format compliance.

#### ML

- МЛ-1..4: Round 3 launch (clip range x3 + dr.grpo).
- МЛ-5: pass@k fix PR.
- МЛ-6+7: **OS reward verification**: design test matrix (5 rewards x 3 domains x 100 examples), run, analyze. Какие rewards дают learning signal? Какие нет?

#### W1 Friday Checkpoint

- [ ] Storage endpoint + 100 records round-trip
- [ ] >= 3 OS reward models online + **verification P/R/F1 per domain**
- [ ] Determ checks API works, P >= 95%
- [ ] **Distill pipeline v0: demo on 100 tasks, quality vs single-teacher**
- [ ] MD-check test >= 800 in ОБМ
- [ ] Safety pilot launched, IAA measured
- [ ] Arena 28k in markup
- [ ] Русификатор >= 5k reward format
- [ ] Stage 1.5: math >= 200k, code >= 100k
- [ ] **Leak check report ready**
- [ ] SFT data >= 20k annotated
- [ ] ProcessBench accuracy + PRM pilot 200
- [ ] BFCL v4 baseline; **LCB baseline vs Qwen3.5-2B/4B**
- [ ] B2C checkers P/R on 100
- [ ] Round 3 launched (50+ steps)
- [ ] pass@k fix in review
- [ ] **OS reward verification: which rewards work, which don't**

---

<a id="w2"></a>

### W2 (Mar 24-28): OS Reward Matrix + OLMO RL Baseline + Данные

**Цель:** OS rewards замерены на всех тестах. **OLMO RL baseline запущен (4 конфигурации).** Первый свой reward LoRA обучен. Distill pipeline → production. Stage 1.5 >= 600k.

**Результат недели (модель):** Модель = всё ещё Base. Но: OLMO RL даёт/не даёт learning signal. Distill pipeline production-ready. Первый свой reward.

#### Детали по командам

- **Дима-1:** Залить ВСЕ datasets в хранилище → inventory table.
- **Дима-2:** Прогнать 5 OS rewards на всех тестах → **matrix P/R/F1 (25+ ячеек)**.
- **Дима-3:** LoRA training pipeline → end-to-end test.
- **Поля-1:** MD train → reward training prep (>= 3k).
- **Поля-2:** Русификатор тест → 500+.
- **Поля-3:** Safety → 1000+ (если IAA ok W1).
- **Поля-4:** Arena markup progress >= 30%.
- **Поля-5:** **Эквивалентность reward LoRA** (2.1k train ready!) → **P/R/F1 vs OS baseline**.
- **Поля-6:** Vikacheck-reasoning SFT + тест markup.
- **Иван-1+2:** Solve First code +200k, IF/Arena/Safety +100k → **total >= 600k**.
- **Иван-3:** Contamination fixes applied → zero overlap verified.
- **Эльдар-1:** Distill pipeline → production quality. Generate first batch of 5k+ through pipeline.
- **Эльдар-2:** SFT +30k с full annotation (MD, logic, Arena + RU when ready) → total >= 50k annotated.
- **Эльдар-3:** PRM prompt engineering + test set 500+ (protocol 3c step 2-3).
- **Вова-1:** ToolRM on BFCL + error analysis → **top-5 error types**.
- **Вова-2:** SWE-bench 10b → mean@1.
- **Серёжа-1:** Code reward pipeline → P/R.
- **Серёжа-2:** Queries с domain labels → 10k+.
- **Даша-1+2:** EXP-B2C-1 full baseline report.
- **Даша-3+4:** B2C reward train data >= 500 per function.
- **ML (МЛ-1..4):** Round 3 analysis → **best config table**.
- **ML (МЛ-5+6):** **OLMO RL baseline** — 4 configs x 20-30 steps each: (A) OLMO only, no rewards. (B) OLMO + verified OS rewards. (C) Current math + OS rewards. (D) Mix (math + OLMO) + OS rewards. Для каждого: замер math_500, AIME, LCB после 20-30 steps.
- **ML (МЛ-7):** Dev prompt 1k tokens.

#### W2 Friday Checkpoint

- [ ] OS reward matrix (5 models x 5+ tests) documented
- [ ] Эквивалентность LoRA: P/R/F1 vs OS
- [ ] LoRA pipeline works end-to-end
- [ ] Stage 1.5 >= 600k
- [ ] SFT >= 50k annotated; **distill pipeline production-ready**
- [ ] PRM test set 500+ ready
- [ ] Round 3 results: best clip + dr.grpo
- [ ] **OLMO RL baseline: 4 configs compared, learning signal confirmed/denied**
- [ ] **Week result: "OS matrix + OLMO baseline + first own reward + distill pipeline ready"**

---

<a id="w3"></a>

### W3 (Mar 31 - Apr 4): Reward Training v1 + First Real RL

**Цель:** MD + Русификатор LoRA обучены. SFT masking ablation завершён. Stage 1.5 >= 1M. **First real RL canary с verified rewards на наших данных.**

**Результат недели (модель):** Cold Start v0 training started. **First real RL canary: math/code delta after 20-30 steps.** SFT masking answer found.

- **Дима:** Determ rewards → RL pipeline; MD + Русификатор LoRA training; reward_config auto-assign.
- **Поля:** MD LoRA test (**acc ~0.94 expected**); Русификатор LoRA; Ты/Вы expand; Safety 1k+ train; Arena 60%; routing classifier start.
- **Иван:** Stage 1.5 → 1M; IFEval reward setup; contamination clean.
- **Эльдар (Э-1+2):** **SFT masking ablation (4 strategies)**: full masking, negative masking, contrast pairs, no masking → benchmark each. SFT >= 80k annotated.
- **Эльдар (Э-3):** PRM cross-validation + train generation (protocol 3c step 3-4).
- **Вова:** Tool reward v1 (AST + exec); SWE-smith setup.
- **Серёжа:** Code reward v1; difficulty filtering P(mixed).
- **Даша:** EXP-B2C-2/3 (interpreter + websearch reward v1); text2image.
- **ML (МЛ-1+2):** DPPO ablation.
- **ML (МЛ-3+4):** Advantage sampler.
- **ML (МЛ-5+6):** **First real RL canary**: math data + verified OS rewards, 20-30 steps. Does RL improve math_500/AIME on our data?
- **ML (МЛ-7):** Cold Start v0 training.

#### W3 Friday Checkpoint

- [ ] **MD LoRA: acc ~0.94** (match lora-tests.md)
- [ ] Русификатор LoRA: metrics documented
- [ ] **SFT masking: 4 strategies compared, best selected**
- [ ] Stage 1.5 >= 1M
- [ ] SFT >= 80k annotated
- [ ] PRM cross-validation done, train generation started
- [ ] DPPO results (or blocker documented)
- [ ] **First real RL canary: math+code delta after 20-30 steps**
- [ ] **Week result: "Masking chosen, first 2 own rewards work, RL produces learning signal on our data"**

---

<a id="w4"></a>

### W4 (Apr 7-11): SFT Gate + RL v0 — multi-reward

**Цель:** SFT обученa и бьёт distill (GATE). Все LoRA validated. Первый multi-reward RL.

**Результат недели (модель):** **Cold Start v1** (SFT). GATE CHECK: beats distill. RL v0 started.

- **Дима:** Hierarchical aggregation (safety veto + GDPO); multi-reward pipeline; canary infra.
- **Поля:** Safety LoRA; Arena test 2k; Смена LoRA; Ты/Вы LoRA; **ВСЕ LoRA validation (таблица own vs OS)**; routing classifier v1.
- **Иван:** **Stage 1.5 → 2M (GATE)**; IFEval+Vikacheck; dynamic sampling proto.
- **Эльдар (Э-1):** **SFT final 100-200k annotated**.
- **Эльдар (Э-2):** **EXP-SFT-2/3/4 distill comparison** (наш SFT vs distill Qwen-3/3.5/GLM-5).
- **Эльдар (Э-3):** PRM 0.6B LoRA training (protocol 3c step 5-6a).
- **Вова:** EXP-TOOL-EFF-1 (efficiency baseline); agent synthesis start.
- **Серёжа:** Code RL set final; first canary 20-30 steps.
- **Даша:** EXP-B2C-4/5 (max calls + cross-tool).
- **ML:** First multi-reward RL (OS + determ); **EXP-AGG-1/2/5**; SFT training.

#### W4 Friday Checkpoint

- [ ] **SFT GATE: beats distill Qwen-3/3.5/GLM-5 → PASS/FAIL**
- [ ] **Stage 1.5: 2M DONE**
- [ ] All reward LoRA P/R/F1 table (own vs OS)
- [ ] Reward aggregation: best formula chosen
- [ ] First multi-reward RL: metrics on all tracks
- [ ] PRM LoRA trained, test metrics
- [ ] **Week result: "Cold Start v1 done + beats distills. Multi-reward RL works. PRM trained."**
- [ ] **CRITICAL: если SFT не бьёт distill → STOP, diagnose, replan**

---

<a id="w5"></a>

### W5 (Apr 14-18): RL v0 Extended + 120B Prep

**Цель:** RL v0 extended до 100+ steps с verified rewards. Полный reward стек operational. **120B checkpoint selection.**

**Результат недели (модель):** **RL v0** — running at scale. 120B candidate selected.

- **Дима:** Production reward pipeline; monitoring dashboard (reward vs eval per 20 steps); ODIN EXP-HACK-2.
- **Поля:** Arena LoRA; Safety P >= 99%; all tests >= 500; routing deployed; B2C integration test; NLI factuality.
- **Иван:** Domain mix ablation; IFEval + IF reward ablation; difficulty sampler.
- **Эльдар:** PRM ablation EXP-PRM-3 (outcome vs outcome+process); SFT promotion check.
- **Вова:** OTC-PO EXP-TOOL-EFF-2/3; agent trajectories 100+.
- **Серёжа:** Code RL 50+ steps; VeRPO test.
- **Даша:** NLI fact-check; search grounding EXP-FACT-3; B2C canary 20-30 steps.
- **ML (МЛ-1..4):** RL v0 extended 100+ steps (all tracks measured per 20 steps).
- **ML (МЛ-5+6):** Rollout replay RLEP.
- **ML (МЛ-7):** **120B checkpoint selection** — выбрать лучший 10B checkpoint, подготовить к масштабированию на 120B.

#### W5 Friday Checkpoint

- [ ] RL v0 running 100+ steps, all tracks measured
- [ ] Full reward stack operational (all LoRA + OS + determ)
- [ ] Monitoring dashboard live
- [ ] Domain mix ablation: best proportions
- [ ] PRM: outcome vs process → delta on AIME
- [ ] **120B checkpoint selected, eval prep started**
- [ ] **Week result: "RL v0 at 100+ steps. Full stack live. 120B prep."**

---

<a id="w6"></a>

### W6 (Apr 21-25): RL v1 + 120B Eval + Оптимизация

**Цель:** RL v1 с собственными rewards. 300+ степов без collapse. Safety per-domain. **120B eval sweep.** Cross-domain regression clean.

**Результат недели (модель):** **RL v1** — extended run, all tracks. **120B eval results.**

- **Дима:** PAR EXP-HACK-3; gradient reg EXP-HACK-4; scaling 200+ steps.
- **Поля:** Fix regressions; expand weak rewards; **safety per-domain** (7 domains); human eval prep; reward ensemble validation; reward hacking monitoring.
- **Иван:** Mixed sampler; dynamic sampling; OBM format 50 tasks.
- **Эльдар:** PRM strategy final; cross-domain regression analysis.
- **Вова:** Loop penalty tuning EXP-LOOP-2/3; SWE-smith 100+.
- **Серёжа:** Code RL 100+ steps; code+math joint.
- **Даша:** B2C RL 100+ steps; URL tracking; max calls optimization.
- **ML (МЛ-1..4):** **RL v1: 300+ steps best config**; collapse monitoring + guardrails.
- **ML (МЛ-5+6):** Checkpoint selection.
- **ML (МЛ-7):** **120B eval sweep**: scale best 10B checkpoint, run full eval matrix.

#### W6 Friday Checkpoint

- [ ] **RL v1: 300+ steps without collapse**
- [ ] Cross-domain regression matrix: no track regressed > allowed delta
- [ ] Safety per-domain compliance table (7 domains)
- [ ] Reward hacking check: reward/eval correlation ok
- [ ] OBM format >= 90%
- [ ] **120B eval sweep: results on all tracks**
- [ ] **Week result: "RL v1 at scale. 120B eval done. No regressions. Safety per-domain done."**

---

<a id="w7"></a>

### W7 (Apr 28 - May 2): Pre-release — eval sweep, canary

**Цель:** Полный eval sweep vs конкуренты (**Qwen3.5-2B/4B**, GLM-5, DeepSeek V3.2). Все gate checks. 120B results documented. Canary 48h started.

**Результат недели (модель):** **RL v2 candidate** — best checkpoint selected.

- **Дима:** Prod deploy prep; eval sweep automation; **competitor benchmarking** (Qwen3.5-2B, Qwen3.5-4B, GLM-5, DeepSeek V3.2 на тех же бенчах).
- **Поля:** Final reward audit; Arena swap canary; Safety swap canary; human eval 100-200.
- **Иван:** IFEval gate; Vikacheck gate; OBM format gate; promotion thresholds.
- **Эльдар:** ProcessBench/PRMBench final; math gate; cross-domain final.
- **Вова:** BFCL gate; agent gate (SWE + BrowseComp); tool suppression <= 5%.
- **Серёжа:** LCB gate; SWE gate; code regression.
- **Даша:** B2C function gates; latency (tokens/sec, TTFT, chain length).
- **ML:** Best checkpoint; config frozen; **canary 48h start**; **120B final results documented**.

#### W7 Friday Checkpoint

- [ ] **Full eval sweep: us vs Qwen3.5-2B vs Qwen3.5-4B vs GLM-5 vs DeepSeek (all tracks)**
- [ ] Per-track gate: GREEN/RED matrix
- [ ] Canary 48h started
- [ ] 120B eval: final results
- [ ] Human eval results
- [ ] **Week result: "We know exactly where we stand vs competitors. Canary running."**

---

<a id="w8"></a>

### W8 (May 5-9): Release gate

**Цель:** Решение о релизе. Production candidate или explicit blocker list.

**Результат недели (модель):** **Prod release** или **blocker list + timeline**.

- **Дима:** Canary 48h analysis; monitoring; rollback docs.
- **Поля:** **Safety gate final** (refusal, false refusal, jailbreak, per-domain, ЦУР); factuality gate; human eval analysis.
- **Иван:** Final IF gate sign-off.
- **Эльдар:** Final math gate sign-off.
- **Вова:** Final tools/agent gate sign-off.
- **Серёжа:** Final code gate sign-off.
- **Даша:** Final B2C function gate sign-off.
- **ML:** **PROMOTION DECISION**. Release candidate. Post-release monitoring plan. 120B promotion assessment.

#### W8 Friday Checkpoint

- [ ] **Gate 5: ALL criteria PASS/FAIL**
- [ ] Canary 48h: stable
- [ ] 120B assessment documented
- [ ] All team sign-offs collected
- [ ] **DECISION: RELEASE / BLOCKED (with list + timeline)**

---

<a id="section-5"></a>

## 5. Загрузка по людям (валидация)

Каждый человек имеет 1-2 задачи в неделю. Ниже — heatmap.

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

> Каждая ячейка = 1 задача. Каждый человек загружен каждую неделю. Пустых ячеек нет.

---

<a id="section-6"></a>

## 6. Карта экспериментов

| ID | Неделя | Owner | Зависимость | Описание |
|----|--------|-------|-------------|----------|
| **Distill Pipeline** | | | | |
| EXP-DISTILL-1 | W1-2 | Эльдар-1 | API access | Multi-teacher pipeline: quality vs single-teacher на 100 задачах |
| **Cold Start** | | | | |
| EXP-SFT-1 | W2 | Эльдар-2 | SFT >= 50k | Baseline distill Qwen-3 |
| EXP-SFT-2 | W4 | Эльдар-2 | SFT-1 done | Distill Qwen-3.5 |
| EXP-SFT-3 | W4 | Эльдар-2 | SFT-1 done | Distill GLM-5 |
| EXP-SFT-4 | W4 | Эльдар-1 | SFT-5 done | Наш подход (masking) |
| EXP-SFT-5 | W3 | Эльдар-1+2 | SFT >= 80k | Masking ablation a/b/c/d |
| **OS Reward Verification** | | | | |
| EXP-RWVER-1 | W1 | Дима-2 + МЛ-6+7 | OS rewards deployed | 5 rewards x 100 labeled examples → P/R/F1, work/don't work |
| EXP-LEAK-1 | W1 | Иван-3 | datasets loaded | Eval overlap check in train data |
| **OLMO RL Baseline** | | | | |
| EXP-OLMO-1 | W2 | МЛ-5+6 | rewards verified | OLMO data only, no rewards (sanity baseline) |
| EXP-OLMO-2 | W2 | МЛ-5+6 | rewards verified | OLMO + verified OS rewards |
| EXP-OLMO-3 | W2 | МЛ-5+6 | rewards verified | Current math + verified OS rewards |
| EXP-OLMO-4 | W2 | МЛ-5+6 | rewards verified | Mix (math + OLMO) + OS rewards |
| **Reward aggregation** | | | | |
| EXP-AGG-1 | W4 | ML-4 | OS rewards + determ | Weighted sum baseline |
| EXP-AGG-2 | W4 | ML-5 | OS rewards + determ | GDPO decoupled |
| EXP-AGG-5 | W4 | Дима-1 | safety veto impl | Hierarchical (safety+GDPO) |
| **Process reward** | | | | |
| EXP-PRM-1 | W3 | Эльдар-3 | PRM annotations done | Baseline step-error detection + cross-val |
| EXP-PRM-2 | W4 | Эльдар-3 | PRM train data | Train PRM 0.6B LoRA |
| EXP-PRM-3 | W5 | Эльдар-1+2 | PRM-2 done | Outcome vs outcome+process |
| EXP-PRM-4 | W6 | Эльдар-1 | PRM-3 done | Partial credit vs veto vs decouple |
| **120B Experiments** | | | | |
| EXP-120B-1 | W6 | МЛ-7 | best 10B ckpt | Scale to 120B, full eval sweep |
| **Reward hacking** | | | | |
| EXP-HACK-1 | W5-6 | Поля-6 | monitoring dashboard | Correlation tracking per 20 steps |
| EXP-HACK-2 | W5 | Дима-3 | Arena reward | ODIN length debiasing |
| EXP-HACK-3 | W6 | Дима-1 | RL v1 running | PAR reward shaping |
| EXP-HACK-4 | W6 | Дима-2 | RL v1 running | Gradient reg vs KL |
| **B2C functions** | | | | |
| EXP-B2C-1 | W2 | Даша-1+2 | tool generations | Baseline: call dist, loop, format |
| EXP-B2C-2 | W3 | Даша-1 | interpreter reward | Interpreter reward v1 |
| EXP-B2C-3 | W3 | Даша-2 | websearch reward | Websearch decomposition v1 |
| EXP-B2C-4 | W4 | Даша-1 | B2C-2 done | Max tool calls limit |
| EXP-B2C-5 | W4 | Даша-2 | B2C-3 done | Cross-tool: interp + search |
| **Tool efficiency** | | | | |
| EXP-TOOL-EFF-1 | W4 | Вова-1 | tool reward v1 | Call distribution, redundancy |
| EXP-TOOL-EFF-2 | W5 | Вова-1 | EFF-1 done | OTC-PO efficiency |
| EXP-TOOL-EFF-3 | W5 | Вова-1 | EFF-1 done | ToolRLA multiplicative |
| **Loops** | | | | |
| EXP-LOOP-2 | W6 | Вова-1 | SpecRA impl | Hard penalty (-1.0) |
| EXP-LOOP-3 | W6 | Вова-1 | loop-2 done | Soft penalty (graduated) |
| **Factuality** | | | | |
| EXP-FACT-2 | W5-6 | Поля-6 | NLI model | NLI factuality reward in RL |
| EXP-FACT-3 | W5 | Даша-2 | search reward | Search-grounded eval offline |
| **ML training** | | | | |
| Round 3 clip | W1-2 | МЛ-1+2 | verl ready | 3 clip configs |
| Round 3 dr.grpo | W1-2 | МЛ-3+4 | verl ready | DR-GRPO |
| DPPO | W3-4 | МЛ-1+2 | OOM fix | Trust-region mask |
| IS-correction | W3-4 | МЛ-5+6 | running | Collapse zone check |
| Advantage sampler | W3-4 | МЛ-3+4 | plateau | EMA-based sampler |
| First real RL | W3 | МЛ-5+6 | rewards verified | Math + verified rewards, 20-30 steps |

---

<a id="section-7"></a>

## 7. Топ-10 рисков

| # | Риск | P | Impact | Митигация | Owner |
|---|------|---|--------|-----------|-------|
| 1 | Collapse не решён Round 3 | Medium | Critical | DPPO + sampler + IS-correction как backup | ML |
| 2 | Safety данных 0 → не успеем к W5 | High | Critical | ЦУР fallback + InternLM2 OS reward | Поля-3 |
| 3 | SFT не бьёт distill (W4 gate) | Medium | Critical | Добавить данные, пересмотреть masking, увеличить compute | Эльдар |
| 4 | Distill pipeline не готов к W2 | Medium | High | Fallback: single-teacher data W2, pipeline catch-up W3. Re-generate | Эльдар-1 |
| 5 | OLMO baseline не даёт learning signal | Low | High | Проверить rewards (W1), determ-only fallback, switch to math-only | ML |
| 6 | Arena 28k разметка медленная | Medium | High | Fast batch 5k; crowd остальное | Поля-4 |
| 7 | GPU capacity для OS rewards | Medium | Medium | Приоритет 3 модели из 5 | Дима-2 |
| 8 | SWE-bench infra недоступна | Medium | Medium | Synthetic tasks fallback | Вова-2 |
| 9 | Reward hacking при multi-reward | Medium | High | Monitoring + ODIN + PAR | Дима + ML |
| 10 | 120B infra недоступна | Medium | Medium | Отложить eval на W8, eval top-2 checkpoints only | МЛ-7 |

---

<a id="section-8"></a>

## 8. Критические решения по неделям

| Неделя | Решение | Кто решает | Данные для решения |
|--------|---------|------------|-------------------|
| W1 | **Какие OS rewards работают?** Verification: work/don't work | Дима + ML lead | EXP-RWVER-1: P/R/F1 per reward |
| W2 | Выбор лучшего clip range + dr.grpo конфига | ML lead | Round 3 metrics table |
| W2 | **OLMO config: какой даёт лучший signal?** | ML lead | EXP-OLMO-1..4 results |
| W3 | Выбор стратегии маскирования SFT | Эльдар | EXP-SFT-5 ablation results |
| W3 | **Использовать ли OLMO mix в real RL?** | ML lead | OLMO vs math-only RL results |
| W4 | **SFT GATE: go/no-go** | Эльдар + ML lead | EXP-SFT-1..4 comparison |
| W4 | Формула агрегации reward | Дима + ML lead | EXP-AGG-1/2/5 results |
| W5 | **120B checkpoint selection** | ML lead | RL v0 track-level metrics |
| W5 | Domain mix proportions | Иван + ML lead | Ablation results |
| W6 | PRM strategy (partial credit vs veto vs decouple) | Эльдар | EXP-PRM-4 results |
| W7 | **Promotion thresholds per track** | All leads | Full eval sweep table + 120B eval |
| W8 | **RELEASE DECISION** | All leads + руководитель | Gate 5 checklist |
