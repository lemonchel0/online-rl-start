# Online RL: Оглавление проекта

## Цель программы

Обучить production-ready модель с помощью online reinforcement learning, которая:
1. Станет лучшей моделью на русском языке (внутренние метрики: Русификатор, Ты/Вы, Смена языков — #1).
2. Обгонит **Qwen-3.5-2B** (ближайший по active params) и будет сравнима с Qwen-3.5-4B, GLM-5 по ключевым глобальным бенчмаркам (код, math, tool-use, агентность, arena).
3. Даст реальному пользователю ощущение «умной модели на кончиках пальцев» (B2C quality).

## Команды и зоны ответственности

| Команда | Зона | Ключевые задачи |
|---------|------|-----------------|
| **ML** | Архитектура, обучение, эксперименты | RL pipeline (verl), сетапы, гиперпараметры, стабильность обучения, eval, promotion gates |
| **Data** | Данные, разметка, reward-модели | Coldstart, reward train/test, rollout queries, reward LoRA, чекалки, leakage контроль |

## Конкуренты (актуальные, 2025-2026)

| Модель | Архитектура | Ключевые метрики | RL-подход |
|--------|-------------|-------------------|-----------|
| **DeepSeek V3.2** | 685B MoE, 37B active | AIME25 88-89%, LiveCodeBench 74-75%, IMO Gold | GRPO variant + verifiable rewards, 85K agentic prompts / 1800 environments |
| **Qwen-3.5** | 397B MoE, 17B active | LiveCodeBench 83.6, SWE-bench 76.4, BrowseComp 78.6, MMLU-Pro 87.8 | GRPO (10x vs PPO), million-agent RL, multi-stage post-training |
| **GLM-5** | 744B MoE, 40B active | SWE-bench 77.8, BrowseComp 89.7, MCP-Atlas 80.9, HLE 50.4 | Slime async RL (Megatron + SGLang), decoupled generation/training |

## Карта документов

### ML (обучение и эксперименты)

| Документ | Что внутри | Статус |
|----------|-----------|--------|
| [RL/RnD/Setups (+baseline).md](start_pack/RL/RnD/Setups%20(+baseline).md) | Сводка open-source сетапов + внутренние эксперименты (8 baseline runs) | Confirmed |
| [RL/RnD/Setups (+baseline)/Round 1.md](start_pack/RL/RnD/Setups%20(+baseline)/Round%201.md) | Batch/minibatch sweep -> выбран 128/64 | Done |
| [RL/RnD/Setups (+baseline)/Round 2.md](start_pack/RL/RnD/Setups%20(+baseline)/Round%202.md) | Context length (8k/16k), masks, LR scheduler -> cosine recommended, masks not recommended | Done |
| [RL/RnD/Setups (+baseline)/Round 3.md](start_pack/RL/RnD/Setups%20(+baseline)/Round%203.md) | Clip range, dr.grpo, adv_normalize | Planned |
| [RL/RnD/DPPO.md](start_pack/RL/RnD/DPPO.md) | Trust-region маска в REINFORCE-like loss | In progress (porting) |
| [RL/RnD/MinPRO.md](start_pack/RL/RnD/MinPRO.md) | MinPRO hypothesis | Placeholder |
| [RL/RnD/Rollout-IS-correction with sampler.md](start_pack/RL/RnD/Rollout-IS-correction%20with%20sampler.md) | IS-correction + advantage sampler | In progress |
| [RL/RnD/Rollout-IS-correction for multi domain.md](start_pack/RL/RnD/Rollout-IS-correction%20for%20multi%20domain.md) | DeepSeek-style masks for math+code | Planned |
| [RL/RnD/Rollout replay.md](start_pack/RL/RnD/Rollout%20replay.md) | RLEP-style replay buffer | Planned |
| [RL/RnD/Process reward.md](start_pack/RL/RnD/Process%20reward.md) | Process reward shaping | TODO |
| [RL/General/Scheduler.md](start_pack/RL/General/Scheduler.md) | Scheduler overview | Stub |
| [RL/General/Scheduler/Advantage sampler.md](start_pack/RL/General/Scheduler/Advantage%20sampler.md) | EMA-based advantage sampler | Documented |
| [RL/General/Scheduler/Difficulty (Kimi-like) sampler.md](start_pack/RL/General/Scheduler/Difficulty%20(Kimi-like)%20sampler.md) | Success-rate difficulty sampler | Documented |
| [RL/General/Scheduler/Mixed sampler.md](start_pack/RL/General/Scheduler/Mixed%20sampler.md) | Advantage + difficulty combo | Documented |
| [RL/General/Benchmarks.md](start_pack/RL/General/Benchmarks.md) | Список бенчей (IFEval, IFBench, GPQA, MMLU-Pro, MERA) | Catalog |
| [RL/Backlog VERL.md](start_pack/RL/Backlog%20VERL.md) | Pipeline tech debt (lr warmup, pass@k fix, docker, etc.) | Active |

### Data (датасеты, reward, разметка)

| Документ | Что внутри | Статус |
|----------|-----------|--------|
| [online-rl.md](start_pack/online-rl.md) | Три типа датасетов (rollout queries, reward train, reward test), формат, хранение | Active |
| [datasets.md](start_pack/datasets.md) | Per-checker статусы: MD, Русификатор, Эквивалентность, Vikacheck, Ты/Вы, Безопасность, Смена языков, Арена + LoRA | Active |
| [reward-lora.md](start_pack/reward-lora.md) | Форматы reward/LoRA, статусы всех чекалок, задачи по дообору данных | Active |
| [lora-tests.md](start_pack/lora-tests.md) | Результаты тестов LoRA (Русификатор, MD, Смена языков, Ты/Вы) | Partial |
| [how-to-shoot.md](start_pack/how-to-shoot.md) | Инструкция по запуску LoRA через API | Reference |

### Tools & Environments

| Документ | Что внутри | Статус |
|----------|-----------|--------|
| [RL/Tools&Env's/(all) ToolCurrentPlan.md](start_pack/RL/Tools&Env's/(all)%20ToolCurrentPlan.md) | Текущий план по tool-экспам (SFT, файлы, BrowseComp, tau2, BFCL, SWE) | Active |
| [RL/Tools&Env's/(lproz + mmpavlov) ToolCalling in verl 0.7.0.md](start_pack/RL/Tools&Env's/(lproz%20+%20mmpavlov)%20ToolCalling%20in%20verl%200.7.0.md) | Unified Agent Loop, SWE-bench integration, perf profiling | Active |
| [RL/Tools&Env's/(v_min) Tool Calling Rewards.md](start_pack/RL/Tools&Env's/(v_min)%20Tool%20Calling%20Rewards.md) | Tool-call reward penalties, LM-judge integration | In progress |
| [RL/Tools&Env's/(v_min) LM-judge for Code Exec.md](start_pack/RL/Tools&Env's/(v_min)%20LM-judge%20for%20Code%20Exec.md) | LM-judge to reduce FPR from naive tool rewards | In progress |
| [RL/Tools&Env's/(lproz) Browse comp.md](start_pack/RL/Tools&Env's/(lproz)%20Browse%20comp.md) | BrowseComp: low yield, duplicate loops, fabricated URLs | In progress |
| [RL/Tools&Env's/(lproz) Websearch LLM as Judge.md](start_pack/RL/Tools&Env's/(lproz)%20Websearch%20LLM%20as%20Judge.md) | LLM-judge vs deterministic checker for websearch | In progress |
| [RL/Tools&Env's/(mmpavlov) Tool&Agent Benches.md](start_pack/RL/Tools&Env's/(mmpavlov)%20Tool&Agent%20Benches.md) | Synthetic Tool Universe vision, bench targets | Planning |
| [RL/Tools&Env's/(mmpavlov) GigaGym updates.md](start_pack/RL/Tools&Env's/(mmpavlov)%20GigaGym%20updates.md) | GigaGym API, Docker images, SWE infra | Active |

## Как пользоваться этими документами

1. **Новый в проекте?** Начни с этого файла -> перейди к `online_rl_ml_data_gap_analysis.md` для понимания текущего состояния и приоритетов.
2. **ML-инженер?** Смотри `RL/RnD/Setups (+baseline).md` и Round 1-3 для текущих сетапов; `RL/Backlog VERL.md` для tech debt; `RL/General/Scheduler/` для сэмплеров.
3. **Data-инженер?** Смотри `datasets.md` и `reward-lora.md` для статусов по каждой чекалке; `online-rl.md` для формата данных.
4. **Tool/Agent?** Смотри `RL/Tools&Env's/` — каждый файл по отдельному направлению (BrowseComp, BFCL, SWE, code exec).

## Связанные документы

- [online_rl_workplan_8w.md](online_rl_workplan_8w.md) — 8-недельный план работ до production (монолитный файл, архив).
- [detailed_plan/](detailed_plan/00_overview.md) — **разбитая версия:** обзор + файлы по каждой команде с перекрёстными ссылками на конкретных людей.
- [online_rl_teams.md](online_rl_teams.md) — состав команд, зоны ответственности, лиды.
- [online_rl_ml_data_gap_analysis.md](online_rl_ml_data_gap_analysis.md) — подробный аналитический документ: что есть, чего нет, что делать, SOTA practices, benchmark strategy, roadmap до prod-ready checkpoint.

### Структура Gap Analysis (карта секций)

| Секция | Содержание |
|--------|-----------|
| **0a. Cold Start: Stage 1.5** | Qwen-3 дистилляция, solve first, 2M+ target, распределение по доменам, 5 экспериментов |
| **0b. Cold Start: SFT** | 100-200k чистых данных, same steps, маскирование ошибочных кусков (4 стратегии), gate: обыграть дистил Qwen-3/3.5/GLM-5 |
| **0c+. DPO** ⏸ | DPO перед online RL: **DEFERRED** на пост-8-недельный план. Секция сохранена для контекста |
| **0c. Developer System Prompt** | Проблемы длинного prompt, план сокращения до 1k токенов, что оставить / что убрать, плейсхолдеры |
| 1. ML: текущее состояние | Pipeline, эксперименты Round 1-3, пробелы, must-do roadmap |
| 2. Data: текущее состояние | Статус чекалок, пробелы, must-do roadmap |
| **2.4 Хранение + reward mapping** | Формат хранения online-RL сета, per-sample reward_config, classification pipeline |
| 3. SOTA датасеты | ArenaLift + B2C критерии, shortlist, план внедрения |
| **3a. Отбор задач для RL** | 10k+ per domain, 50% solve rate критерий, P(mixed), динамическое сэмплирование, классификация запросов |
| **3b. Reward format pipeline** | 4-step pipeline: Sky-reward → point/pair/n-rank → LoRA → main model |
| **3c. Протокол создания reward** | 7-step protocol для команды: промпт → тест → кросс-валидация → трейн → LoRA → разметка → улучшение |
| 4. Reward Strategy | OS-first, маппинг замены, протокол, ресурсные ограничения |
| **4.5 Детерминированные reward** | Циклы, повторы, JSON format, LaTeX — rule-based проверки |
| **4.6 Каталог reward + бенчмарки** | Полный список reward с бенчмарками для каждого |
| 5. Reward routing | Подходы к routing reward в random chat, рекомендация v1 |
| **5a. Формула агрегации reward** | Weighted sum, GDPO, Multiplicative, Cascade, Hierarchical — эксперименты |
| 6. Метрики reward | Precision vs recall, safe operating points, guardrails |
| **6.5 Митигация reward hacking** | ARA, PAR, Gradient Regularization, ODIN — эксперименты |
| 7. Benchmark strategy | По capability-трекам: Math, Code, IF, Tools, Agents, Arena, RU, Safety, Factuality |
| **7.1 Math — Process Reward** | ProcessBench, PRMBench, Socratic-PRMBench, roadmap "correct answer wrong reasoning" |
| **7.4 Tools — Efficiency** | OTC-PO, ToolRLA, redundancy detection, tool productivity |
| **7.4b B2C Functions** | Code interpreter, web search, image generation — reward, format compliance, efficiency limits, 5 экспериментов |
| **7.5 Agents — Loops + Data** | SpecRA loop penalty, agent environment synthesis (Kimi K2, SWE-smith, DeepSeek V3.2) |
| **7.9 Factuality** | FACTS Benchmark, NLI reward, search-grounded eval, Meta approach |
| 8. Roadmap | 5 stage-gates до production-ready checkpoint |
