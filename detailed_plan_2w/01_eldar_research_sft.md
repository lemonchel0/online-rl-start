# Эльдар — Research: SFT + Distill Pipeline + PRM baseline

**Состав:** 3 человека (Э-1, Э-2, Э-3)
**Scope:** Запуск multi-teacher distill pipeline, SFT annotation старт, ProcessBench baseline vs конкуренты.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Distill pipeline start (Э-1), SFT 20k annotation v0 (Э-2), **ProcessBench baseline** (Э-3) | Pipeline script ready на 100 задачах; >= 20k annotated; ProcessBench accuracy зафиксирована | Pipeline demo; annotation table; **ProcessBench vs competitors** | - [ ] |
| [W2](#w2) | Pipeline → first 5k batch (Э-1), SFT +20k annotated (Э-2), **ProcessBench vs competitors table** (Э-3) | 5k+ через pipeline; >= 40k total annotated; comparison table готова | **Quality pipeline vs single-teacher; comparison table** | - [ ] |

**Ключевые риски:**
- API latency для 3 учителей → batch + cache, 2 учителя если третий недоступен
- ProcessBench конкуренты (Qwen3.5-2B) — нужны веса заранее
- SFT annotation LLM-judge дорого → batch 1k/день

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17–21)

<a id="э1-w1"></a>

**Эльдар-1: Multi-teacher distill pipeline — запуск**
- Вход: API доступ к Qwen-3, DeepSeek-V3.2, (опционально) Gemini-Pro.
- Задача:
  - (1) API wrappers для 3 учителей.
  - (2) Generation module: промпт → ответы от 3 учителей.
  - (3) Cross-verification: keep если >= 2 учителя согласны.
  - (4) Rejection sampling: execution/format filter.
  - (5) Best-of-N черновик: скоринг OS rewards (готовится параллельно Димой).
- DoD: Pipeline script ready. Тест на 100 задачах с измеримым quality vs single-teacher.
- Day 1-2: API wrappers + generation. Day 3: cross-verification + rejection. Day 4: BoN draft. Day 5: end-to-end тест.
- Риск: BoN требует OS rewards → использовать length + execution как proxy в W1.

**Ключевое требование к reasoning:** Qwen-3 дистилл выбран как основной учитель — у него наиболее прямой reasoning без Wait и передумываний. Трейсы должны быть: план → вычисление → ответ, без фраз "подождите", "я ошибся", "давайте переосмыслим". При cross-verification отфильтровывать трейсы с рефлексией.

<a id="э2-w1"></a>

**Эльдар-2: SFT data collection + annotation v0**
- Вход: MATH 12k, GSM8K 8k, HumanEval/MBPP, IFEval seeds, WildChat RU.
- Задача: >= 20k SFT из opensource (10k math+code, 5k IF, 5k B2C/STEM).
  - Annotation v0: (1) MD check (determ), (2) logic/correctness (execution/answer match), (3) Arena preference (LLM-as-judge).
  - RU quality annotation — с W2+.
- DoD: >= 20k annotated, distribution table source x domain x count.
- Риск: judge дорого → batch 1k/день, кэшировать.

**Примечание:** Переписывание в SFT допускается. Основная цель — качественные трейсы, где модель учится исправлять ошибки через маскирование, а не воспроизводить их. Annotation должна помечать какие части трейса корректны, какие — нет.

<a id="э3-w1"></a>

**Эльдар-3: ProcessBench baseline + annotation start**
- Вход: ProcessBench (3400 задач), 10b_toolmind, веса Qwen3.5-2B.
- Задача:
  - (1) Запустить ProcessBench на 10b_toolmind → accuracy.
  - (2) Запустить на Qwen3.5-2B (если доступны веса) → comparison.
  - (3) PRM annotation: промпт-инструкция → пилот 200 примеров → IAA.
- DoD: ProcessBench accuracy 10b_toolmind зафиксирована. Comparison table черновик. PRM prompt draft.
- Day 1: Setup ProcessBench eval. Day 2-3: Run на моделях. Day 4: Comparison table. Day 5: PRM annotation pilot.
- Риск: Qwen3.5-2B веса недоступны → использовать API для W1, запустить локально W2.

---

<a id="w2"></a>

### W2 (Mar 24–28)

<a id="э1-w2"></a>

**Эльдар-1: Distill pipeline → первый batch 5k**
- Задача: Оптимизировать throughput. Запустить первый production batch 5k+ задач.
- DoD: 5k+ примеров через pipeline в хранилище. Quality metric: pipeline vs single-teacher на 100 sample audit.
- Day 6-7: Throughput оптимизация (batching, caching). Day 8: production run 5k. Day 9-10: качество audit + документация.

<a id="э2-w2"></a>

**Эльдар-2: SFT +20k, расширение доменов**
- Задача: Добавить +20k SFT (расширение доменов: RU, STEM, Arena seeds). Full annotation: MD + logic + Arena + RU.
- DoD: >= 40k annotated total. Доменное распределение задокументировано.

<a id="э3-w2"></a>

**Эльдар-3: ProcessBench vs competitors финальная таблица**
- Задача:
  - Завершить сравнение: 10b_toolmind vs Qwen3.5-2B vs Qwen3.5-4B vs GLM-5 vs DeepSeek V3.2.
  - По возможности: PRM test set 200+ примеров из ProcessBench.
- DoD: **Comparison table: ProcessBench accuracy по всем моделям.** PRM test set заготовка.

---

## Распределение по доменам — Stage 1.5 и SFT

### Stage 1.5 (целевое: 2M записей, прямой reasoning без Wait)

| Домен | Целевой % | Источники (opensource лучшее) | Расширение под бенч |
|-------|----------|-------------------------------|---------------------|
| Math reasoning | 25% | MATH, GSM8K, AIME, Olympiad, NuminaMath | MATH-500, AIME |
| Code | 20% | HumanEval, MBPP, LCB seeds, OpenCodeInstruct | LCB v6, SWE |
| Instruction Following | 15% | IFEval, FollowBench, WildChat IF | IFEval strict, Vikacheck |
| RU reasoning | 10% | WildChat RU, MERA seeds, RuBQ | MERA, Русификатор |
| STEM / Science | 10% | MMLU STEM, SciQ, ARC-Challenge | MMLU |
| Arena / dialogue | 10% | WildChat, ShareGPT, OpenHermes | Arena-Hard |
| B2C tasks | 5% | ToolBench, APIBench, GAIA | BFCL |
| Safety | 5% | WildGuard, BeaverTails — безопасная часть | Safety test |

**Требования к трейсам (дистилл из Qwen-3):**
- Прямой plan-and-solve: план → шаги → ответ.
- Запрещено: "Wait", "Actually let me reconsider", "Hmm, I made an error earlier".
- Допустимо: внутренние подзаголовки, нумерация шагов, brief check в конце.
- Фильтр: отдельный скрипт pattern-matching на запрещённые фразы.

### SFT (целевое: 100-200k очень чистых записей)

| Домен | Целевой % | Источник | Аннотация |
|-------|----------|----------|-----------|
| Math + логика | 25% | pipeline distill + opensource | MD, correctness, Arena |
| Code | 20% | pipeline distill + LCB | MD, execution, Arena |
| Instruction Following | 20% | IFEval + distill | MD, IF correctness |
| RU quality | 15% | WildChat RU + distill | MD, RU, Русификатор |
| Arena / helpfulness | 10% | WildChat + distill | Arena preference |
| Safety | 5% | ЦУР + InternLM2 fallback | Safety label |
| B2C | 5% | ToolBench + distill | B2C correctness |

---

## Эксперименты команды (2 недели)

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-DISTILL-START | W1 | Э-1 | Multi-teacher pipeline: запуск + 100 задач quality |
| EXP-PROCESSBENCH | W1-2 | Э-3 | ProcessBench: 10b_toolmind vs Qwen3.5-2B/4B/GLM-5/DeepSeek |
| EXP-ANNOT-V0 | W1 | Э-2 | SFT annotation: MD + logic + Arena на 20k |
