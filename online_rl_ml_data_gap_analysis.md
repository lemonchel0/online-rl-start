# Online RL: Gap Analysis и Roadmap до Production-Ready Checkpoint

> Последнее обновление: 2026-03-10
> Контекст: подробный аналитический документ, разбирающий текущее состояние проекта online RL, выявленные пробелы, SOTA-практики, стратегию reward и бенчмарков, и сквозной roadmap до продового чекпоинта.

---

## 0a. Cold Start: Stage 1.5

### Цель

Залить большой объём данных по всем доменам для обучения прямому reasoning. Модель учится планировать и размышлять с понятной фактологией — без переписываний и передумываний (no "Wait", no backtracking à la DeepSeek-R1).

### Источник дистилляции

**Qwen-3** — самый прямой reasoning без Wait и передумываний. Подходит как teacher для stage 1.5.

### Стратегия расширения данных

**Solve First** — pipeline уже есть. Модель-teacher решает задачу, затем трейс используется как обучающий пример.

Три стратегии расширения по стадиям:

| Стадия | Стратегия | Описание |
|--------|-----------|----------|
| **Stage 1.5** | Solve First | Qwen-3 решает задачу, берём полный трейс с прямым reasoning |
| **SFT** | Same Steps | Генерация с теми же шагами, но с валидацией и переписыванием |
| **Online RL** | Усложнение Solve First | Задачи разной сложности, фильтр по solve rate ~50% (см. секцию 3a) |

### Целевой объём

**>= 2M записей** через pipeline расширения по всем доменам.

### Распределение по доменам

| Домен | Источники | Приоритет |
|-------|-----------|-----------|
| Математика | MATH, GSM8K, AIME-style, competition problems + synthetic | Высокий |
| Код | LiveCodeBench-style, HumanEval/MBPP seeds, competitive programming + synthetic | Высокий |
| STEM + финансы/медицина | GPQA, MMLU-Pro domains, specialized QA + synthetic | Средний |
| Форматы + IF | IFEval-style, structured output tasks + synthetic | Средний |
| LMArena / B2C | WildChat, LMArena 55k, prod logs + synthetic | Высокий |
| Функции (базовые + среды + SWE) | BFCL-style, SWE-bench seeds, tool-use scenarios + synthetic | Средний |
| Русский язык | RU-specific QA, MERA-style, translation pairs + synthetic | Высокий |
| Safety | Safety scenarios, jailbreak defense + synthetic | Средний |

Основа: расширения под все бенчи + лучший opensource (секция 3).

### Эксперименты (5 штук)

Эксперименты нечастые — модель постоянно пополняется новыми данными, эксперименты фиксируют контрольные точки.

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-S15-1 | Baseline — текущая модель после stage 1.5 | Все бенчи: AIME, LCB, IFEval, BFCL, MERA, Safety |
| EXP-S15-2 | + math/code expansion (solve first pipeline, +500k) | Delta по AIME, LCB, GSM8K |
| EXP-S15-3 | + IF/Arena/Safety expansion (+300k) | Delta по IFEval, Arena-Hard, Safety refusal |
| EXP-S15-4 | + RU expansion (+200k) | Delta по MERA, Русификатор, Смена языков |
| EXP-S15-5 | Full 2M mix — финальный замер | Все треки vs EXP-S15-1, cross-domain regression matrix |

---

## 0b. Cold Start: SFT

### Цель

100-200k **очень чистых**, полностью провалидированных и размеченных записей. Переписывание допускается. Качество > количество.

### Стратегия

**Same Steps** — pipeline уже есть. Генерация с теми же шагами решения, но каждый трейс проходит валидацию и при необходимости переписывается.

### Маскирование ошибочных кусков

Ключевая идея: модель переписывает часть ответа. Маскируем исходные ошибочные куски, чтобы модель училась **исправлять** ошибки, но не **совершать** их. Без маскирования модель видит ошибку как допустимый паттерн.

**Стратегии маскирования:**

| Стратегия | Описание | Ожидание |
|-----------|----------|----------|
| **(a) Полное маскирование** | loss = 0 на ошибочных chunks, модель учится только на правильных | Conservative, no negative signal |
| **(b) Negative masking** | Ошибочные chunks с negative label (учим "так не надо") | Агрессивнее, риск instability |
| **(c) Contrast pairs** | Ошибочный chunk + правильный chunk рядом (DPO-style) | Best of both worlds, но дороже |
| **(d) No masking** | Стандартный SFT, baseline | Control group |

### Обязательное условие: обыграть дефолтный дистил

Эксперименты на **едином наборе промптов**, но с разными трейсами и стратегиями маскирования:

| ID | Описание | Gate |
|----|----------|------|
| EXP-SFT-1 | Baseline — дистил Qwen-3 → наша модель (same steps, no masking) | Baseline |
| EXP-SFT-2 | Дистил Qwen-3.5 → наша модель | Должны обыграть |
| EXP-SFT-3 | Дистил GLM-5 → наша модель | Должны обыграть |
| EXP-SFT-4 | Наш подход (same steps + masking strategy a) vs EXP-SFT-1/2/3 | **Обязан быть лучше всех дистилов** |
| EXP-SFT-5 | Ablation — стратегии маскирования a/b/c/d на одних промптах | Выбор лучшей стратегии |

### Распределение по доменам

Аналогично stage 1.5 (покрытие всех бенчей), но с упором на качество данных:
- Opensource: лучшие датасеты из shortlist (секция 3), отфильтрованные и провалидированные.
- Расширение через pipeline (same steps strategy).
- Каждый сэмпл прошёл валидацию (автоматическую + выборочную ручную).

---

## 0c+. DPO: Preference Alignment перед Online RL

> **⏸ DEFERRED**: DPO отложен на пост-8-недельный план. Секция ниже сохранена для контекста. Приоритет сейчас — OS reward verification → OLMO RL baseline → Online RL с verified rewards. DPO потребует отдельного плана по сбору preference data.

### Зачем

DPO (Direct Preference Optimization) — стандартная стадия между SFT и online RL. Выравнивает модель по человеческим предпочтениям дёшево (без reward model inference, без rollouts) и стабильно.

### Наши данные: DPO помогает

- Инициализация с `dpo` жила **270 степов** в RL vs **200** для `toolmind` (Round 1-2, Setups doc).
- DPO даёт лучшую стартовую точку для online RL — модель уже выровнена по предпочтениям, RL дотюнивает по verifiable metrics.

### Индустрия

| Модель | Pipeline | Источник |
|--------|----------|----------|
| **DeepSeek R1** | SFT → RL (GRPO) → Rejection Sampling + SFT → RL (GRPO) | [arxiv](https://arxiv.org/abs/2501.12948) |
| **Qwen-3** | SFT → DPO/RLHF → GRPO | [github](https://github.com/QwenLM/Qwen3) |
| **Standard** | SFT → DPO → Online RL | [HuggingFace blog](https://huggingface.co/blog/karina-zadorozhny/guide-to-llm-post-training-algorithms) |

### Данные для DPO

| Источник | Размер | Тип | Статус |
|----------|--------|-----|--------|
| **LMArena Human Preference 55k** | 55k pairs | Real human preferences, 70+ моделей | Opensource, ready |
| **WildFeedback 20k** | 20k pairs | Natural feedback signals | Opensource, ready |
| **Arena 28k** | 28k | Наши, pair-wise разметка | В разметке (ETA ~W4) |
| **Rejection sampling** | Generated | От Cold Start v1 модели | Генерируем в процессе |

### Эксперименты

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-DPO-1 | Offline DPO на LMArena 55k + WildFeedback 20k | Arena-Hard delta, math_500, AIME, LCB |
| EXP-DPO-2 | Online DPO (генерация пар текущей моделью, скоринг Skywork-8B) | Сравнение с offline DPO |
| EXP-DPO-3 | Beta sweep (0.05, 0.1, 0.2) | Preference sharpness vs diversity |

### Gate

- Arena-Hard LC >= +3% vs Cold Start v1
- math_500 не падает > 1%
- AIME не падает > 2%
- LCB не падает > 1%

---

## 0c. Developer System Prompt

### Проблема

Текущий developer system prompt слишком длинный и сложный. Попытки сделать его полностью инструктивным создают проблемы:
- Модель теряется в большом объёме инструкций.
- Возникают конфликты между developer prompt и user instructions.
- Деградация на некоторых доменах (модель "забывает" часть инструкций при длинном контексте).
- Увеличенный TTFT из-за длинного system prompt.

### Решение: сокращение до <= 1k токенов

Оставляем только скелет + необходимые плейсхолдеры.

**Что оставляем:**

| Элемент | Обоснование |
|---------|-------------|
| Дата (timestamp) | Модель должна знать текущую дату |
| Инструкция безопасности | Hard constraint, не подлежит удалению |
| Дефолтное поведение | Тон, формат по умолчанию, язык ответа |
| `{tools_available}` | Список доступных инструментов (interpreter, search, text2image, etc.) |
| `{custom_instructions}` | Плейсхолдер для per-deployment overrides |
| `{persona}` | Если нужно менять persona |
| `{language}` | Предпочтительный язык |

**Что убираем:**

| Элемент | Почему |
|---------|--------|
| Длинные примеры и few-shot | Перенести в per-domain prompts или RAG |
| Детальные инструкции по форматированию | Вынести в per-domain system prompts |
| Redundant safety disclaimers | Оставить одну чёткую инструкцию |
| Перечисление capabilities | Модель и так знает свои возможности |
| Развёрнутые описания tool usage | Перенести в tool descriptions |

### Эксперименты

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-DEVPROMPT-1 | Baseline — текущий длинный prompt | Все бенчи, latency (TTFT) |
| EXP-DEVPROMPT-2 | Сокращённый 1k prompt | Delta по всем бенчам, TTFT improvement |
| EXP-DEVPROMPT-3 | Edge cases — safety compliance, tool triggering, format compliance | Per-domain regression check |

---

## 1. ML: текущее состояние

### 1.1 Что уже сделано (confirmed)

**Базовый RL pipeline:**
- Фреймворк: verl v0.7.0 (Megatron + SGLang/vLLM), submit-only jobs.
- Базовая модель: `10b_toolmind` (GigaChat 10B-A1B), проверены также `stage 1.5` и `dpo` инициализации.
- Основной loss: `cispo_prod` (стабильнее vanilla и cispo_masked).
- Текущий baseline сетап: `batch=128, minibatch=64, lr=1e-6, n_resp_per_prompt=16, clip_low=0.2, clip_high=0.28, weight_decay=0.1, sampler=random`.
- Доменная смесь: `code 0.285 / math 0.285 / structured_output 0.01 / mcqa 0.09 / nemotron 0.33`.

**Завершенные эксперименты (Round 1-2):**

| Параметр | Findings | Источник |
|----------|----------|----------|
| Batch/minibatch sweep | 512 — слишком медленно; 256/128 и 256/64 — нестабильны; **128/64 — лучший баланс** | Round 1 |
| Context length 8k vs 16k | Разница минимальна; staged 8k->16k->24k->32k чуть лучше для AIME (0.374) | Round 2 |
| Masks (cispo_masked 0.5/1.0/1.5) | Все варианты коллапсировали раньше baseline; **маски не рекомендованы** | Round 2 |
| LR scheduler | Cosine scheduler повышает стабильность, обучение живет до ~400 шагов; **рекомендован** | Round 2 |
| Базовые модели | toolmind: AIME ~0.31 (200 steps), dpo: дольше всего жила (~270 steps) | Setups |

**Текущие пиковые метрики:**
- `math_500`: ~0.87
- `AIME`: ~0.31-0.374 (лучшие раны)
- Типичный collapse: ~150-200 степов (без scheduler), ~360-400 (со scheduler).

### 1.2 Что запланировано, но не сделано

| Направление | Статус | Источник |
|------------|--------|----------|
| Clip range sweep (1.0/0.28, 0.2/4.0, 1.0/4.0) | Planned, не запущено | Round 3, Clip-range.md |
| DR-GRPO (adv_normalize=False, loss_agg_mode=dr_grpo) | Planned, нужна проверка verl config | Round 3, dr.grpo like.md |
| DPPO (trust-region маска) | In progress — porting из Stable-RL, OOM на Megatron+SGLang | DPPO.md |
| IS-correction + sampler | In progress, ещё не дошли до зоны collapse | Rollout-IS-correction with sampler.md |
| IS-correction multi-domain | Planned, ждет код данных от Vadim | Rollout-IS-correction for multi domain.md |
| Rollout replay (RLEP) | Planned, код есть, не запускали | Rollout replay.md |
| MinPRO | Placeholder, нет сетапа | MinPRO.md |
| Process reward | TODO, нет данных | Process reward.md |
| n_resp_per_prompt tuning | Empty placeholder | n_response_per_prompt.md |
| entropy_coeff, use_kl, weight_decay tuning | Deferred (resource-intensive) | Round 3 |

### 1.3 Критичные пробелы ML (P0)

1. **Нет promotion criteria.** Непонятно, при каких метриках checkpoint считается production-ready. Нужен формальный gate.
2. **Нет anti-regression test suite.** При росте math может падать IF/safety/RU — нет автоматического контроля.
3. **pass@k считается неправильно.** Отмечено в Backlog VERL: «перепроверить как pass@k считаем и как с несмещенными оценками».
4. **Collapse не решен.** Все раны деградируют на 150-400 шагах. Cosine scheduler отодвигает, но не устраняет.
5. **Нет eval sweep pipeline.** Нет автоматического замера ВСЕХ capability-треков после каждого чекпоинта.

### 1.4 ML must-do roadmap (приоритезированный)

| P | Действие | Зависимости |
|---|----------|-------------|
| P0 | Зафиксировать promotion criteria по всем capability-трекам (см. секцию 6) | Benchmark strategy |
| P0 | Запустить clip range + dr.grpo (Round 3) — потенциально решают collapse | Карты |
| P0 | Починить pass@k | Backlog VERL |
| P0 | Собрать автоматический eval sweep (все треки после каждого значимого run) | Infra |
| P1 | Завершить DPPO porting, запустить ablation | Stable-RL code |
| P1 | Advantage sampler на plateau (после cosine scheduler) | Sampler code ready |
| P1 | IS-correction + sampler: дождаться результата в зоне collapse | Running |
| P2 | Rollout replay, process reward | Data readiness |
| P2 | entropy_coeff / use_kl / weight_decay sweep | Resources |

---

## 2. Data: текущее состояние

### 2.1 Что уже сделано (confirmed)

**Форматы зафиксированы:**
- Reward: `<critic>reasoning</critic>$\boxed{True/False}$` (point-wise), pair-wise, n-rank.
- LoRA: `{"has_grammar_spelling_or_punctuation_errors": false, "text_with_objective_corrections": null}`.
- Rollout queries: API-формат (messages с system/user).

**Статус по чекалкам:**

| Чекалка | Reward train | Reward test | Rollout queries | LoRA | Статус |
|---------|-------------|-------------|----------------|------|--------|
| **MD-check** | ~4k (в разметке) | ~800 (формируется) | — | В разметке | Разметка пробежала |
| **Русификатор** | ~7.6k (в разметке) | 300 | 13.5k (готово) | 3.2k train, 300 test; заваривается | Стоит разметка |
| **Эквивалентность ответа** | 2.1k (done, залит) | 1.1k (done, в разметке) | 13.8k (готово) | — | Заваривается модель |
| **Vikacheck-assistant** | — (собираем) | 1.1k | — | — | Разметка пробежала |
| **Vikacheck-reasoning** | — (собираем) | — | — | — | Собираем данные |
| **Ты/Вы** | 2k (в разметке) | 288 | — | — | Стоит разметка |
| **Безопасность** | — (нет) | — (нет) | — | — | Пишем инструкцию |
| **Смена языков** | ~1.9k (thinking done + answer in markup) | ~300 | — | — | Собираем train |
| **Арена** | 28k (не размечено) | 2k (надо собрать) | — | — | Надо поставить в разметку |

**Тесты LoRA (lora-tests.md):**
- Русификатор: accuracy 0.72-0.78 (улучшение от GigaChat-2-Pro 0.49).
- MD: accuracy 0.94, precision 0.99 (LoRA значительно лучше базы).
- Смена языков: accuracy 0.71 на GigaChat-2-Pro (LoRA ещё не замерена).
- Ты/Вы: accuracy 0.69 на GigaChat-2-Pro (precision 0.9, recall 0.23 — сильный bias к negative class).

### 2.2 Критичные пробелы Data (P0)

1. **Нет проверки на leakage/contamination.** Rollout queries «взяты из opensource, но не проверены на лики» (ваши слова). Overlap между rollout/reward train/reward test — policy, но нет enforcement pipeline.
2. **Хранение не определено.** Многократно отмечено: «ответственный: Дима, должен придумать куда грузить». Ингест может застопориться.
3. **Безопасность reward отсутствует.** Нет ни train, ни test, ни инструкции для разметки.
4. **Арена reward не размечена.** 28k train не в разметке, 2k test не собран.
5. **Доменная смесь не валидирована.** Текущая `code 0.285 / math 0.285 / structured_output 0.01 / mcqa 0.09 / nemotron 0.33` — эмпирическая, нет данных о её оптимальности.
6. **Тестовые сеты слишком маленькие.** Русификатор test: 300, Ты/Вы test: 288, Смена языков test: ~300 — для надежной оценки нужно 500+.

### 2.3 Data must-do roadmap

| P | Действие | Owner |
|---|----------|-------|
| P0 | Запустить dedup/contamination check для rollout queries vs reward train/test vs eval benchmarks | Data |
| P0 | Определить хранилище, pipeline ингеста | Дима/Сергей |
| P0 | Собрать Safety reward: инструкция -> разметка -> train -> test | Максим Гольцов |
| P0 | Поставить Арена 28k в разметку, собрать 2k тест | Никита Лобов |
| P1 | Расширить тестовые сеты до 500+ (Русификатор, Ты/Вы, Смена языков) | Data |
| P1 | Валидировать доменную смесь: ablation по пропорциям | ML + Data |
| P2 | Добрать данные для Vikacheck-reasoning, MD, Безопасность | Data |

### 2.4 Хранение сета для online-RL + маппинг reward на сэмпл

**Формат хранения query:** JSON с метаданными и привязкой к reward-ам:

```json
{
  "id": "sample_001",
  "dataset_id": "online_rl_v1",
  "messages": [
    {"role": "system", "content": ""},
    {"role": "user", "content": "Реши уравнение x^2 - 5x + 6 = 0"}
  ],
  "domain": "math",
  "difficulty": "medium",
  "reward_config": [
    {"reward_id": "rule_boxed", "weight": 1.0, "type": "deterministic"},
    {"reward_id": "prm_check", "weight": 0.5, "type": "model", "model": "prm-0.6b"},
    {"reward_id": "ru_quality", "weight": 0.3, "type": "model", "model": "russificator-lora"}
  ]
}
```

**Обязательные метки на каждый сэмпл (помимо reward_config):**

| Метка | Значения | Зачем |
|-------|----------|-------|
| `domain` | math / code / stem / if / functions / arena / safety / ru / b2c / ... | Балансировка batch, стратифицированная eval |
| `difficulty` | easy / medium / hard (по solve rate из секции 3a) | Curriculum learning, динамическое сэмплирование |
| `context_length` | short (<2k) / medium (2-8k) / long (8k+) | Staged context training, latency budgeting |

Метки проставляются через **query classification pipeline**: автоклассификация по домену + сложности + длине → auto-assign `reward_config` + метки. Classifier может быть rule-based (regex/keyword) на первом этапе, затем обученный lightweight model.

**Хранение:**
- Единое хранилище (S3/MinIO/GCS) с версионированием. DVC или аналог для трекинга датасетов.
- Три строго непересекающихся подмножества: rollout queries, reward train, reward test (см. [online-rl.md](start_pack/online-rl.md)).
- Reward test хранится в ОБМ.
- Train Catalog — от ветки `main`.

---

## 3. SOTA датасеты для переиспользования (ArenaLift + B2C)

### Двойной критерий отбора

Каждый кандидат оценивается по:
- **ArenaLift**: ожидаемое повышение head-to-head preference в arena-подобных сценариях.
- **B2CPerceivedIntelligence**: улучшение «ощущения ума» для реального пользователя (ясность, точность, полезность, уверенность без галлюцинаций).

### Shortlist

| Датасет | Домен | Размер | ArenaLift | B2C | Contamination risk | Лицензия | Verdict |
|---------|-------|--------|-----------|-----|-------------------|----------|---------|
| **LMArena Human Preference 55k** | General preference | 55k pairs, 70+ моделей | Высокий (реальные arena preferences) | Высокий (diverse real prompts) | Низкий (recent, timestamped) | Open | **core-mix** |
| **ODA-Mixture-500k** | Math/Code/Reasoning/General | 500k | Высокий (optimized for ODA bench) | Средний (academic tasks) | Средний (проверить overlap) | Open | **core-mix (после dedup)** |
| **OpenCodeInstruct** | Code | 5M (filtered subsets) | Средний (code-only) | Высокий (execution feedback, real tasks) | Средний (check vs LiveCodeBench) | CC-BY-4.0 | **core-mix (code track)** |
| **WildChat 650k** | Multilingual dialog | 650k conversations | Высокий (real user conversations, 66 langs) | Очень высокий (authentic B2C patterns) | Низкий (anonymized, timestamped) | ODC-BY | **core-mix (RU subset)** |
| **WildFeedback 20k** | Preference from real users | 20k pairs | Высокий (natural feedback signals) | Высокий (authentic preferences) | Низкий | Open | **core-mix** |
| **UltraFeedback 64k** | General preference | 64k prompts x 4 completions | Средний (GPT-4 scored, not human) | Средний | Средний (widely used, check overlap) | MIT | **diag-only** |
| **Nectar 183k** | Preference | 183k prompts x 7 responses | Средний | Средний | Высокий (older, widely distributed) | Open | **diag-only** |
| **OpenMathInstruct-2** | Math | 14M pairs | Низкий (math-only) | Низкий (academic) | Средний (check vs MATH/GSM8K test) | CC-BY-4.0 | **diag-only** |

### План внедрения

1. Начать с `LMArena 55k` + `WildChat RU subset` + `WildFeedback` — максимальный ArenaLift + B2C эффект.
2. Добавить `ODA-Mixture` и `OpenCodeInstruct` после dedup vs наши eval benchmarks.
3. Для каждого: canary run (20-30 шагов RL), замер cross-domain regression.

---

## 3a. Отбор задач для RL

### Размер и покрытие

- **Минимум 10K сэмплов на домен** для статистически значимого learning signal.
- **Обязательные направления**: Математика, Код, остальной STEM + финансы/медицина, Форматы + IFEval, LMArena (lmsys + прод логи), Функции (базовые + среды + SWE).

### Базовый отбор: критерий 50% solve rate

Лучше отбирать задачи, которые решаются в ~50% случаев — максимальный learning signal (ни слишком легко, ни невозможно).

**Процесс:**
1. Прогнать 4/8/16 генераций на каждую задачу.
2. Посчитать success rate `p` (scores: 0, 0.25, 0.5, 0.75, 1).
3. Отфильтровать по `P(mixed) = 1 − p^N − (1−p)^N` — вероятность получить mixed batch ([source](https://openreview.net/pdf?id=QXrZ0Y3yGJ)).
4. Предпочтение задачам с `p ∈ [0.3, 0.7]`.

### Динамическое сэмплирование

По мере обучения задачи, которые стали слишком лёгкими (p > 0.9) или слишком тяжёлыми (p < 0.1), заменяются новыми. Referece: [Dynamic Sampling for RL](https://openreview.net/pdf/b3c7dc1539020266bcb75760d12e6850036356c8.pdf).

### Классификация запросов

Каждый query классифицируется по:
- **Теме**: математика, код, b2c, инструкции, функции, safety и пр.
- **Сложности**: easy / medium / hard (по solve rate или эвристике).
- Классификация используется для: (a) балансировки batch, (b) auto-assign reward_config, (c) стратифицированной eval.

---

## 3b. Reward format и pipeline обучения reward

### 4-step reward pipeline

| Шаг | Описание | Формат |
|-----|----------|--------|
| **Шаг 0** | Sky-reward (OS baseline) | Score от OS-модели |
| **Шаг 1** | Point-wise / pair-wise / n-rank reward | `<critic>reasoning</critic>$\boxed{True/False}$` (point-wise), `$\boxed{A>>B\|A>B\|A==B\|A<B\|A<<B}$` (pair-wise), `$\boxed{1\|2\|3\|4\|5}$` (n-rank) |
| **Шаг 2** | Переписывание LoRA-ми | JSON: `{"has_grammar_spelling_or_punctuation_errors": bool, "text_with_objective_corrections": str\|null}` |
| **Шаг 3** | Основная модель обучается на reward + переписывание | RL training loop |

Формат зафиксирован для всех reward-ов — reasoning обязателен в `<critic>` тегах, финальная метка в `\boxed{}`.

---

## 3c. Протокол создания reward (train + test)

> Этот блок — стандарт для всей команды. Каждый новый reward проходит эти шаги.

### Шаги

**Шаг 1. Промпт-инженерия.** Собираем system prompt, который на топ-тир модели даёт 80%+ качества. Reasoning в ответе + boxed финальный скор (формат из 3b). Две разные топ-тир модели должны давать корреляцию > 0.9 на финальную метку.

**Шаг 2. Тестовый сет.** Собираем простой проект разметки — минимум инструкций, однозначные критерии, бинарная или ordinal шкала. Проверяется финальная метка. Размечаем тестовый сет с высоким перекрытием 5+ (400+ штук разнообразных задач 50/50 по классам).

**Шаг 2a. Gate.** Если метрика на тесте ниже порога X — итерируем промпт. Не переходить к шагу 3, пока порог не пройден.

**Шаг 3. Кросс-валидация.** Прогоняем 2 топ-тир модели, берём majority метку, считаем метрику относительно размеченного тестового сета. Метрика должна быть >= X. Если ответы моделей расходятся — метка = false.

**Шаг 4. Генерация трейна.** Берём релевантный реварду датасет и прогоняем с этим промптом 2 топ-тир модели (именно те, на которых делали замер Шага 1). Дополнительно: берём сэмпл из каждого source трейна (для разнообразия обучающей выборки) и тоже прогоняем обе модели.

**Шаг 5. Сборка трейна.** Из совпавших трейсов (можно брать оба) собираем train set. Запускаем обучение LoRA.

**Шаг 5a. Contamination check.** Перед обучением — dedup train vs test (exact + fuzzy match). Тестовый сет не должен утечь в train.

**Шаг 6. Разметка расхождений.** Пока LoRA обучается — ставим разъехавшиеся метки на ручную разметку (можно обе). Если разметка тоже расходится — пока выкидываем.

**Шаг 6a. Валидация LoRA.** После обучения — прогнать LoRA на тестовом сете. Зафиксировать P/R/F1/accuracy. Если ниже порога — не использовать в RL, вернуться к шагу 5 или 1.

**Шаг 7. Итеративное улучшение.** Улучшаем разметкой train set, расширяем test, закрываем баги разметкой. Фиксируем: сколько сэмплов добавлено, из каких источников, разнообразие по доменам/сложности.

### Длительная разметка

После пилота (шаг 2) проект выставляется на длительную разметку большого сета. За этим процессом обязательно присматривать: назначить AI-тренера, который отвечает на вопросы разметчиков, контролирует качество (споты, метрики согласованности), менеджерит проект.

Простые проекты (бинарная метка, однозначные критерии) можно выставлять на **crowd**-разметку. Сложные (reasoning quality, arena preference) — только in-house или экспертная разметка.

### Обязательные правила

- Нет смысла в заварках без тестового сета. Тестовый сет может быть opensource, если такой есть — это ок.
- Тестовый сет должен быть размечен за исключением очень редких краевых случаев.
- Для арен допускается в качестве теста корреляция с очень мощным оценщиком (GPT-5.4 или аналог последней версии).
- Пороги X по типу reward: precision >= 95% (safety), accuracy >= 85% (general), correlation >= 0.8 (arena/preference).
- Логирование: в метаданных reward-датасета обязательно фиксировать: (a) какие модели использовались для генерации меток, (b) версии моделей, (c) дата прогона, (d) промпт version. Хранить рядом с train/test в том же хранилище.
- Проект разметки должен быть **простым**: минимум инструкций, однозначные критерии, бинарная или ordinal шкала. Чем проще проект — тем выше inter-annotator agreement и ниже стоимость.

---

## 4. Reward Strategy: opensource-first, замена на свои

### 4.1 Стартовые OS-reward модели

| Назначение | OS-модель | Размер | Обоснование | Источник |
|---|---|---|---|---|
| General preference / Arena | Skywork-Reward-V2-Llama-3.1-8B | 8B | #1 на RewardBench v1/v2, PPE, RM-Bench, JudgeBench. 26M pairs | [HF](https://huggingface.co/Skywork/Skywork-Reward-V2-Llama-3.1-8B) |
| General (budget/daily) | Skywork-Reward-V2-Qwen3-0.6B | 0.6B | Почти = Gemma-2-27B при 45x меньше | [HF](https://huggingface.co/Skywork/Skywork-Reward-V2-Qwen3-0.6B) |
| Multi-objective | ArmoRM-Llama3-8B-v0.1 | 8B | 19 objectives + MoE gating. RewardBench 89.0, reasoning 97.3, safety 92.2 | [HF](https://huggingface.co/RLHFlow/ArmoRM-Llama3-8B-v0.1) |
| Tool-calling | ToolRM (1.7B-8B) | 1.7-8B | Единственный специализированный для FC. +25% Best-of-N | [arxiv](https://arxiv.org/abs/2509.11963) |
| Math | Rule-based `\boxed{}` checker + LLM-judge fallback | 0B + 0.6B | Precision ~100% для формальной проверки | — |
| Code | Execution sandbox + LLM-judge fallback | 0B + 0.6B | Самый честный reward для кода | — |
| Safety | InternLM2-7B-Reward | 7B | Bilingual, 2.4M pairs, safety dim. RewardBench 86.6 | [HF](https://huggingface.co/internlm/internlm2-7b-reward) |
| Safety (budget) | InternLM2-1.8B-Reward | 1.8B | RewardBench 80.6 | [HF](https://huggingface.co/internlm/internlm2-1.8b-reward) |

### 4.2 Маппинг замены: OS -> свой reward

| Capability | OS-start | Наш reward (target) | Критерий замены | Когда |
|---|---|---|---|---|
| MD-check | ArmoRM-8B (helpfulness dim) | Свой MD-reward LoRA (~4k train) | Свой P/R на тесте выше ArmoRM на MD-домене | После заварки |
| Русификатор | Skywork-0.6B | Свой Русификатор LoRA (7.6k train) | Accuracy на тесте выше + стилистика по LLM-judge | После расширения теста до 500+ |
| Эквивалентность | Skywork-0.6B + rule-based boxed | Свой Equivalence reward (2.1k train, 1.1k test) | Recall выше при равном precision | Почти готов |
| Ты/Вы | Rule-based heuristic | Свой Ты/Вы reward (2k train, 288 test) | Precision >= 98%, FP < 1% | После разметки |
| Safety | InternLM2-7B-Reward | Свой Safety reward (нет данных) | Precision >= 99% на RU-specific harm | Когда инструкция + разметка |
| Смена языков | Heuristic + Skywork-0.6B | Свой Lang-switch reward (~1.9k train) | Recall выше при precision >= 95% | После разметки |
| Tool-calling | ToolRM-1.7B | Свой checker (v_min) | FPR ниже при равном recall | Когда BFCL-checker стабилизирован |
| Arena/preference | Skywork-8B | Свой Arena reward (28k train, 2k test) | Correlation с human preference выше | После замера |

### 4.3 Протокол замены

1. **Baseline**: замерить OS-reward на внутреннем тесте (P, R, F1, correlation).
2. **Train own**: обучить свой reward на размеченных данных.
3. **Compare**: замерить свой reward на том же тесте. Критерий: primary metric строго выше, secondary не хуже.
4. **Canary run**: 20-30 степов RL с новым reward, проверить convergence + кросс-домены.
5. **Swap**: если canary ok — заменить. OS-reward остается как fallback/sanity.
6. **Monitor**: regression watch первые 100 степов.

### 4.4 Ресурсные ограничения

- Preference: <= 8B. 20B+ только если <= 8B не дает acceptable quality.
- Daily monitoring: 0.6B-1.8B.
- Gate decisions: 7B-8B.
- Rule-based > LLM-judge при равном качестве (дешевле, precision выше).

### 4.5 Детерминированные reward

Детерминированные reward — rule-based проверки, не требующие inference модели. Precision ~100%, latency ~0.

| Reward | Как работает | Метрика | Penalty |
|--------|-------------|---------|---------|
| **Циклы / loops** | SpecRA (FFT-based spectral detection, [ICLR 2026](https://openreview.net/forum?id=xVO4BqmzVD)) или n-gram match >= 3 повторов | `loop_detected: bool` | Hard: reward = -1.0 |
| **Повторы** | N-gram repetition detection на уровне предложений | `repetition_ratio` | Graduated: penalty пропорционален degree |
| **Следование формату (JSON etc.)** | Rule-based JSON/XML/YAML parser | `parse_failed_rate` | Binary: 0 если parse failed |
| **Сломанный LaTeX** | Regex + LaTeX compiler check | `latex_valid_rate` | Binary: penalty если invalid |

Привязка к OBM: 50 задач с форматом 1-в-1 для валидации (см. секцию 7.3).

### 4.6 Полный каталог reward и их бенчмарки

| Reward | Тип | Бенчмарк для оценки reward |
|--------|-----|---------------------------|
| **Оценка рассуждений (PRM)** | Model | ProcessBench (3,400 задач, ACL 2025, [GitHub](https://github.com/QwenLM/ProcessBench)), PRMBench (6,216 проблем, 83K step-labels, ACL 2025, [paper](https://arxiv.org/abs/2501.03124)), Socratic-PRMBench (2,995 paths, 20 error types, [paper](https://arxiv.org/abs/2505.23474)) |
| **Человеческие предпочтения (Arena)** | Model | RewardBench, AlpacaEval 2.0 LC, Arena-Hard, корреляция с human preference |
| **Безопасность** | Model | Internal safety test, ruHateSpeech, ruEthics, jailbreak test, сеты ЦУР |
| **Смена языков** | Model | Internal lang-switch test, WER на RU-only subset |
| **Проверка функций (Function call)** | Model + Rule | FC-RewardBench, BFCL v4, [arxiv 2501.12948](https://arxiv.org/abs/2501.12948) |
| **Проверка MD** | Model | [arxiv 2501.15000](https://arxiv.org/abs/2501.15000), internal MD test |
| **Сравнение golden vs model answer** | Model | Accuracy на внутреннем тесте (Эквивалентность ответа) |
| **Русификатор** | Model | Русификатор test (расширить до 500+), P/R/F1 |
| **Следование system/инструкции** | Model | IFEval, IFBench, Vikacheck |
| **Фактология** | Model + Search | FACTS Benchmark (DeepMind), SimpleQA, TriviaQA. **NB: search-based reward имеет latency 5-15 сек, несовместим с online RL at scale — использовать для gate eval** |
| **Оценка LaTeX quality** | Rule | LaTeX compiler check, regex validation |
| **Циклы, повторы, JSON, LaTeX** | Rule | См. 4.5 Детерминированные reward |

---

## 5. Reward routing: как прикручивать reward к случайному чату

### 5.1 Проблема

В random chat (типа LM-sys arena) нет заранее известного домена. Нужно решить: какой reward применять к данному контексту/ответу.

### 5.2 Сравнение подходов

| Подход | Как работает | Плюсы | Минусы | Источник |
|--------|-------------|-------|--------|----------|
| **Classifier-gated routing** (ваша идея) | Classifier (контекст + ответ + инструкция reward) -> применять/не применять | Гибкий, обучаемый | Нужны данные для classifier, risk error propagation | — |
| **BayesianRouter** | Offline multi-task router + online Thompson sampling per query | O(1) RM calls, адаптивный, outperforms ensembles | Сложность реализации | [arxiv 2510.02850](https://arxiv.org/abs/2510.02850) |
| **Rule + heuristic routing** | Regex/keyword -> domain -> reward | Простой, предсказуемый | Не покрывает edge cases, brittle | — |
| **Unified reward mixture** | ArmoRM-style multi-objective с динамическим gating | Один вызов, 19 dimensions | Может быть noisy для специфичных доменов | ArmoRM paper |

### 5.3 Рекомендация v1

**Начать с ArmoRM-8B** как unified reward (покрывает 19 dimensions) + **rule-based overrides** для специфичных случаев:
- Math: если есть `\boxed{}` -> формальный checker (precision ~100%).
- Code: если execution sandbox доступен -> execution reward.
- Safety: всегда InternLM2-7B-Reward параллельно (hard veto).

Далее, когда будет достаточно данных по доменам:
- Обучить lightweight classifier (на базе Skywork-0.6B) для routing между domain-specific rewards.
- Или перейти на BayesianRouter если ArmoRM не дает достаточной гранулярности.

---

## 5a. Формула агрегации reward

### Проблема

Несколько чекалок на один сэмпл. Одна прошла, другая нет. Как формировать итоговый reward?

### Подходы

| Подход | Формула | Плюсы | Минусы | Источник |
|--------|---------|-------|--------|----------|
| **Weighted sum** | `R = Σ(w_i × r_i)` | Простой | Reward collapse при GRPO-нормализации | — |
| **GDPO** | Decoupled normalization per reward signal | Preserves distinct signals, drop-in для GRPO | Чуть сложнее имплементация | [arxiv 2601.05242](https://arxiv.org/abs/2601.05242) |
| **Multiplicative (ToolRLA)** | `R = Π(r_i)` | Если любой = 0, итог = 0 (hard veto) | Слишком агрессивно для noisy rewards | [arxiv 2603.01620](https://arxiv.org/abs/2603.01620) |
| **Cascade RL** | Sequential domain-wise RL: RLHF → IF → Math → Code → SWE | Избегает cross-domain interference | Дольше, сложнее rollback | [Nemotron-Cascade](https://arxiv.org/abs/2512.13607) |
| **Hierarchical** | Hard constraints (veto) → soft combination (weighted) | Баланс safety + quality | Нужна настройка thresholds | — |

### Рекомендация v1: Hierarchical

1. **Safety veto**: если `safety_reward < threshold` → `R = -1.0` (hard override).
2. **Loop/cycle veto**: если цикл детектирован → `R = -1.0`.
3. **Остальные**: GDPO-style decoupled normalization — каждый reward нормализуется независимо, advantage считается per-reward.

### Эксперименты

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-AGG-1 | Weighted sum baseline | Cross-domain metrics, convergence speed |
| EXP-AGG-2 | GDPO decoupled normalization | Сравнение advantage distribution с AGG-1 |
| EXP-AGG-3 | Multiplicative (ToolRLA-style) | Impact на tool-calling tasks |
| EXP-AGG-4 | Cascade RL (sequential domain training) | Cross-domain regression после каждого stage |
| EXP-AGG-5 | Hierarchical (safety veto + GDPO) | Safety compliance + quality metrics |

---

## 6. Метрики reward: precision vs recall и сходимость

### 6.1 Матрица ошибок reward

| Тип ошибки | Описание | Влияние на RL |
|-----------|----------|---------------|
| **FP (False Positive)** | Награда за плохой ответ | **Критично.** Учит модель генерировать плохое. Reward hacking. Может привести к emergent misalignment ([arxiv 2511.18397](https://arxiv.org/abs/2511.18397)). |
| **FN (False Negative)** | Пропуск хорошего ответа | **Умеренно.** Снижает learning signal, замедляет обучение. Модель не получает reward за правильное поведение. |
| **Miscalibration** | Неправильная шкала (все награды ~одинаковые) | **Опасно.** Advantage -> 0, нет gradient signal. Обучение стагнирует. |
| **Ranking noise** | Правильный rank, но шумный score | **Умеренно.** GRPO/CISPO robust к шуму в абсолютных значениях, но чувствителен к инверсии рангов. |

### 6.2 Куда целиться по доменам

| Домен | Приоритет | Обоснование |
|-------|-----------|-------------|
| **Math** | Precision >> Recall | FP учит модель ложным решениям. Rule-based checker дает ~100% precision. |
| **Code** | Precision >> Recall | FP учит невалидному коду. Execution sandbox — единственный честный arbiter. |
| **Tool-calling** | Precision >> Recall | FP учит вызывать инструменты ненужно. Tool suppression от FN менее опасна. |
| **Safety** | Precision >>> Recall | FP (награда за unsafe) — катастрофа. Лучше FN (пропустить safe ответ). |
| **Arena/preference** | Recall ~ Precision | Нужен баланс: FP учит «красиво врать», FN теряет learning signal. |
| **IF** | Precision > Recall | FP учит игнорировать инструкции при формальном соответствии. |
| **RU quality** | Precision > Recall | FP учит ошибкам в русском, заметным пользователю. |

### 6.3 Safe operating point

- Для rule-based rewards (math checker, code execution, heuristics): **precision ~100%, recall варьируется** — это ok.
- Для LLM-judge rewards: задать **confidence threshold** (abstain при неуверенности). Лучше `abstain` чем FP.
- Для preference rewards: мониторить **correlation с human preference** на тестовом сете (>= 0.8).
- Обязательный canary run перед каждым swap reward.

### 6.4 Guardrails

| Индикатор | Trigger | Действие |
|-----------|---------|----------|
| Entropy падает >20% за 10 шагов | Early collapse signal | Checkpoint + pause + diagnose |
| Reward mean растет, но eval metrics стагнируют/падают | Reward hacking | Stop, inspect reward FP rate |
| math_500 падает >2% от пика на 3 consecutive checkpoints | Math regression | Rollback to best checkpoint |
| Safety refusal rate < threshold | Safety degradation | Immediate stop |
| Response length растет без роста arena win rate | Length hacking | Check preference reward, add length penalty |

### 6.5 Митигация reward hacking: подходы и эксперименты

| Подход | Суть | Источник |
|--------|------|----------|
| **ARA** (Adversarial Reward Auditing) | Hacker policy ищет уязвимости reward, Auditor детектит exploitation. Auditor-Guided RLHF gates reward signals. Снижает sycophancy, verbosity, code gaming. | [arxiv 2602.01750](https://arxiv.org/abs/2602.01750) |
| **PAR** (Preference As Reward) | Bounded rewards из латентных preferences самой reward model. Rapid initial growth + gradual convergence. +5pp AlpacaEval 2.0, robust после 2 epoch. | [arxiv 2502.18770](https://arxiv.org/abs/2502.18770) |
| **Gradient Regularization** | Bias policy updates к регионам с высокой accuracy reward model. Outperforms KL penalty across diverse settings. | [arxiv 2602.18037](https://arxiv.org/abs/2602.18037) |
| **ODIN** (Disentangled Reward) | Два reward heads — один коррелирует с length, другой decorrelated. Discard length head при RL. Убирает length bias. | [ICML 2024](https://proceedings.mlr.press/v235/chen24bn.html) |

**Эксперименты:**

| ID | Описание | Гипотеза |
|----|----------|----------|
| EXP-HACK-1 | Мониторинг — трекать correlation(reward_mean, eval_metrics) каждые 20 шагов | Baseline: выявить где hacking начинается |
| EXP-HACK-2 | ODIN-style length debiasing на Arena reward | Убирает length hacking без потери quality |
| EXP-HACK-3 | PAR reward shaping вместо raw reward | Bounded reward замедляет hacking |
| EXP-HACK-4 | Gradient regularization vs KL penalty ablation | GR outperforms KL при том же бюджете |
| EXP-HACK-5 | ARA adversarial auditing pipeline | Если ресурсы позволяют — proactive vulnerability detection |

---

## 7. Benchmark strategy: RU + Global по capability-трекам

### 7.0 Общие принципы

- Двухконтурная eval: **RU** (MERA + internal) + **Global** (код/math/tool/agent/arena/factuality).
- Честное сравнение: единый prompt policy, одинаковые ограничения, reproducible eval против Qwen-3.5 / GLM-5 / DeepSeek V3.2.
- Cross-domain regression matrix: после каждого значимого RL run — полный sweep по всем трекам.

### 7.1 Math

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | MATH-500 (daily), AIME/AIME25 (gate), GSM8K (sanity), AMC 2023 |
| **Primary metrics** | math_500 accuracy, AIME pass@1 |
| **Reward** | Rule-based `\boxed{}` (precision ~100%) + LLM-judge fallback для нестандартных форматов |
| **Process validation** | **ProcessBench** (step-level error detection), **PRMBench** (6,216 проблем, 83K step-labels), **Socratic-PRMBench** (2,995 paths, 6 reasoning patterns, 20 error subtypes) |
| **Конкуренты** | DeepSeek V3.2: AIME25 88-89%; Qwen-3.5: GRPO + CoT cold start |
| **Experiments** | EXP-MATH-1: baseline; EXP-MATH-2: candidate; EXP-MATH-3: math-only stress; EXP-MATH-4: FP vs FN ablation |
| **Gate** | math_500 >= target, AIME >= target, GSM8K не ниже baseline |
| **Rollback** | math_500 падает >2% от пика на 3 consecutive checkpoints |

**Process Reward Validation — roadmap:**

Проблема: outcome-based reward не ловит случаи "ответ верный, рассуждение ошибочно". Такие случаи обучают модель ложным стратегиям.

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-PRM-1 | Baseline — прогон текущей модели через ProcessBench | Step-error detection accuracy |
| EXP-PRM-2 | Обучить lightweight PRM (0.6B-8B) на PRM800K или собственных аннотациях | ProcessBench accuracy, PRMBench score |
| EXP-PRM-3 | Ablation: outcome-only reward vs outcome + process reward | AIME/MATH-500 accuracy + % "lucky guesses" eliminated |
| EXP-PRM-4 | Стратегия для "correct answer, wrong reasoning": (a) Partial credit: `reward = 0.5*outcome + 0.5*process`, (b) Veto: если process < threshold → reward = 0, (c) Decouple (PathFinder-PRM): classify error types per step | Какая стратегия даёт лучший AIME при меньшем reward hacking |
| EXP-PRM-5 | Замер impact на AIME/MATH-500 при стратегиях a/b/c | Convergence speed, final accuracy, reward correlation |

### 7.2 Code

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | LiveCodeBench v6 (contamination-free, обязательный), SWE-bench Verified, BigCodeBench, HumanEval/MBPP (sanity) |
| **Primary metrics** | LiveCodeBench pass@1, SWE-bench resolve rate |
| **Reward** | Execution sandbox (primary) + LLM-judge (monitoring). Qwen3-Coder-Next approach: RL на verifiable tasks |
| **Process validation** | ProcessBench для code reasoning (аналогично Math). Валидация что логика решения корректна, а не только финальный тест проходит |
| **Конкуренты** | Qwen-3.5: LCB 83.6, SWE 76.4; GLM-5: SWE 77.8; DeepSeek V3.2: LCB 74-75% |
| **Experiments** | EXP-CODE-1: baseline; EXP-CODE-2: candidate; EXP-CODE-3: code-only stress; EXP-CODE-4: exec vs judge; EXP-CODE-5: + process reward для code reasoning |
| **Gate** | LCB >= target, SWE >= target |
| **Rollback** | LCB падает >3% от пика |

### 7.3 Instruction Following

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | IFEval (strict + loose), IFBench, Vikacheck (internal) |
| **Primary metrics** | IFEval strict accuracy, Vikacheck compliance |
| **Secondary metrics** | **OBM format validation**: 50 задач с форматом 1-в-1 (JSON, markdown, list, table, code block). Прогон через OBM framework для замера удержания формата |
| **Reward** | Программная проверка формата + LLM-judge семантика. Precision >> Recall |
| **Конкуренты** | Qwen-3.5: 20+ task types в general RL; GLM-5: tau2-Bench 67.8 |
| **Experiments** | EXP-IF-1: baseline; EXP-IF-2: candidate; EXP-IF-3: conflict stress; EXP-IF-4: с/без IF reward ablation; EXP-FORMAT-1: baseline format retention на 50 OBM tasks; EXP-FORMAT-2: + format reward (rule-based JSON/MD parser) |
| **Gate** | IFEval strict >= target, Vikacheck >= baseline, OBM format >= 90% |
| **Rollback** | IFEval strict падает >3% |

### 7.4 Basic Functions (tool/function calling)

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | BFCL v4 (обязательный), FC-RewardBench, internal tool test |
| **Primary metrics** | BFCL v4 overall, BFCL multi-turn |
| **Reward** | ToolRM-1.7B (start) -> свой checker. AST-based + execution для реальных API |
| **Конкуренты** | Qwen-3.5: TAU2 86.7; GLM-5: MCP-Atlas 80.9; DeepSeek V3.2: 85K agentic prompts |
| **Experiments** | EXP-FC-1: baseline; EXP-FC-2: + tool reward; EXP-FC-3: 20+ tools stress; EXP-FC-4: ToolRM vs rule-based |
| **Gate** | BFCL >= target |
| **Rollback** | Tool suppression >5% от baseline |

**Tool call efficiency — проблема и эксперименты:**

Модель может вызывать 3 инструмента, когда хватило бы 1. Redundant calls = лишний latency + cost + шум в reward signal.

**Подходы:**
- **OTC-PO** (Optimal Tool Call Policy Optimization, [arxiv 2504.14870](https://arxiv.org/abs/2504.14870)): reward = f(correctness, tool_efficiency). Metric: "tool productivity" = correct_answers / total_tool_calls. Результат: до 73% reduction tool calls при сохранении accuracy.
- **ToolRLA** ([arxiv 2603.01620](https://arxiv.org/abs/2603.01620)): multiplicative reward decomposition: format_validity × tool_selection × param_accuracy × compliance. 63% reduction в tool errors, 47% рост task completion.

**Чекеры для efficiency:**
- Count checker: `n_tool_calls` vs `min_required_calls` (oracle or heuristic).
- Redundancy detector: повторные вызовы с тем же input → penalty.
- Tool productivity metric: `correct / total_calls`.

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-TOOL-EFF-1 | Baseline — замер distribution tool calls per task | Histogram calls, mean, redundancy rate |
| EXP-TOOL-EFF-2 | OTC-PO reward (joint correctness + efficiency) vs plain correctness-only | Tool productivity, accuracy delta |
| EXP-TOOL-EFF-3 | ToolRLA multiplicative decomposition vs additive | Tool error rate, task completion |
| EXP-TOOL-EFF-4 | Ablation — penalty weight за redundant calls | Optimal weight для баланса quality/efficiency |

### 7.4b B2C Functions (interpreter, search, image generation)

B2C-функции вызываются **внутри reasoning** — модель решает, когда вызвать interpreter/search/text2image в процессе рассуждения. Жёсткие гайдлайны на форматы, формирование query, цитирование.

**Code Interpreter:**

| Элемент | Детали |
|---------|--------|
| **Назначение** | Вычисления (math, data analysis), работа с файлами, проверка гипотез |
| **Reward** | Execution success (binary) + результат verifiable: boxed answer match, unit test pass. Аналогично math — outcome verifiable |
| **Penalty** | -0.2 за failed execution, -0.1 за runtime error (уже реализовано в v_min) |
| **Format** | Строгий формат вызова: `code_interpreter` + правильные arguments |
| **Известные проблемы** | Модель использует interpreter для проверки ответа вместо вычислений (SFT thinking); перестаёт вызывать interpreter когда задачи простые; путает code_interpreter <-> no_call для простой математики |

**Web Search:**

| Элемент | Детали |
|---------|--------|
| **Назначение** | Knowledge retrieval внутри reasoning для factual questions |
| **Reward decomposition** | (1) Query quality — релевантность запроса к задаче; (2) Citation accuracy — правильные URL, не fabricated; (3) Answer grounding — ответ обоснован найденными фактами (NLI-check); (4) Format compliance — правильное форматирование цитат |
| **Penalty** | Fabricated URLs → hard -1.0; Цикл поисков → hard -1.0 |
| **Format** | Строгий формат: `websearch` query + `get_url_content` для парсинга |
| **Известные проблемы** | Путает websearch <-> no_call / self_knowledge; циклит; выдумывает ссылки; downspikes у search; на внутреннем поиске реварды сначала хуже |
| **Референсы** | R1-Searcher ([arxiv 2503.05592](https://arxiv.org/abs/2503.05592)): RL для search invocation; Search-o1: agentic search; WebThinker: DPO для search+reasoning |

**Image Generation (text2image):**

| Элемент | Детали |
|---------|--------|
| **Назначение** | Генерация/редактирование изображений по запросу пользователя |
| **Reward** | Correct trigger detection: пользователь просил картинку → вызвали text2image; не просил → не вызвали |
| **Penalty** | Ложный вызов (hallucinated intent) → penalty; Некорректный prompt для генерации → penalty |
| **Format** | Строгий формат query для text2image API |

**Общие ограничения (cross-cutting для всех B2C functions):**

| Ограничение | Реализация |
|-------------|-----------|
| **Нет циклов** | SpecRA detection + hard penalty -1.0 (из секции 7.5) |
| **Контролируемое кол-во вызовов** | `max_tool_calls` per reasoning chain (hard limit) + OTC-PO efficiency reward |
| **Verifiable результат** | Interpreter: execution-based; Search: NLI fact-check; Image: trigger accuracy |
| **Strict format guidelines** | Per-tool format compliance check (детерминированный reward из 4.5) |

**Эксперименты:**

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-B2C-1 | Baseline — распределение вызовов interpreter/search/image, loop rate, format compliance | Current state |
| EXP-B2C-2 | Code interpreter reward (execution success + boxed match) vs no tool reward | code_success_pct, math accuracy delta |
| EXP-B2C-3 | Search reward decomposition (query + citation + grounding) vs binary reward | Citation accuracy, fabricated URL rate, answer quality |
| EXP-B2C-4 | Max tool calls limit + OTC-PO efficiency penalty | Tool productivity, latency, accuracy |
| EXP-B2C-5 | Cross-tool: interpreter + search в одном reasoning chain | Interference check, combined task success |

### 7.5 Agentic Scenarios

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | SWE-bench Verified, BrowseComp, tau2-Bench, Terminal-Bench 2.0, MCP-Atlas |
| **Primary metrics** | SWE-bench resolve, BrowseComp accuracy, tau2 score |
| **Reward** | Outcome-based (start) -> process reward если outcome не converges |
| **Конкуренты** | GLM-5: BrowseComp 89.7, SWE 77.8; Qwen-3.5: BrowseComp 78.6; DeepSeek V3.2: agentic synthesis |
| **Experiments** | EXP-AGENT-1: baseline; EXP-AGENT-2: + agent data; EXP-AGENT-3: 10+ step stress; EXP-AGENT-4: outcome vs process |
| **Gate** | SWE >= target, BrowseComp >= target |
| **Rollback** | Success rate падает при росте loop frequency |
| **Текущие проблемы** | SWE-bench: mean@1 = 1-4/500 (9b); BrowseComp: 5-10 good из 1.2k; 70% duplicate queries |

**Циклы / Loop penalty:**

Агенты часто впадают в бесконечные циклы — повторяют одни и те же действия без прогресса. Это waste of compute и причина task failure.

**Подходы:**
- **SpecRA** (Spectral Repetition Detection, [ICLR 2026](https://openreview.net/forum?id=xVO4BqmzVD)): FFT-based detection повторяющихся паттернов. Lightweight, non-intrusive, tolerant к minor variations.
- **UFO** (Unary Feedback as Observation): RL с self-reflection, снижает repetitive responses на ~14%.
- **DPO на paired data**: loop trajectory vs non-loop trajectory.

**Penalty scheme:** при детекции цикла (SpecRA threshold или n-gram match >= 3 повторов) — hard penalty: `reward = -1.0`.

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-LOOP-1 | Baseline — замер loop frequency в текущих agent rollouts | Loop rate, mean loop length |
| EXP-LOOP-2 | SpecRA-based loop detector + hard penalty (-1.0) | Loop rate reduction, task success delta |
| EXP-LOOP-3 | Soft penalty (graduated: -0.3 за 2 повтора, -0.7 за 3, -1.0 за 4+) | Сравнение с hard penalty |
| EXP-LOOP-4 | UFO self-reflection approach vs penalty-based | Quality of recovery from near-loops |
| EXP-LOOP-5 | DPO на paired data (loop vs non-loop trajectories) | Impact на loop rate + task quality |

**Сбор данных для агентов (environment synthesis):**

Как это делают топовые модели:

| Модель | Подход | Масштаб | Источник |
|--------|--------|---------|----------|
| **Kimi K2** | 3-stage pipeline: (1) repository of tool specs (real + synthetic via MCP), (2) diverse agent+task generation, (3) successful multi-turn trajectory generation в simulated environments | 1000+ tools | [arxiv 2507.20534](https://arxiv.org/abs/2507.20534) |
| **DeepSeek V3.2** | Large-scale agent task synthesis. "Thinking in Tool-Use" — thinking интегрировано в tool-use. | 1800+ environments, 85K prompts | [HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-V3.2) |
| **SWE-smith** | Автоматическая конвертация GitHub repos в gym environments + синтетическая генерация задач | 52K instances | [GitHub](https://github.com/SWE-bench/SWE-smith) |
| **SWE-Playground** | Синтетическая генерация проектов и задач с нуля через LLM, без зависимости от GitHub | Unlimited | [Paper](https://arxiv.org/abs/2512.12216) |

**Наш roadmap по agent data:**
1. Собрать tool specifications (наши внутренние tools + MCP + synthetic à la Kimi K2).
2. SWE-smith pipeline для code agent tasks.
3. Synthetic multi-turn trajectory generation (Kimi K2 style).
4. Валидация через execution (SWE-bench harness, sandbox).

### 7.6 Arena (B2C user preference)

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | Arena-Hard, AlpacaEval 2.0 LC, MT-Bench |
| **Primary metrics** | Arena-Hard LC win rate, AlpacaEval 2.0 LC |
| **Reward** | Skywork-8B (start) -> свой Arena reward (28k train). Hard constraint: factuality check |
| **Конкуренты** | GLM-4.7: Elo ~1445; Qwen-3.5/GLM-5: выше |
| **Experiments** | EXP-ARENA-1: baseline; EXP-ARENA-2: candidate; EXP-ARENA-3: adversarial stress; EXP-ARENA-4: nemotron weight ablation |
| **Gate** | Arena-Hard LC >= target |
| **Rollback** | LC win rate падает >3% или SimpleQA падает >2% |

### 7.7 Russian Language Quality

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | MERA v1.2.0 (обязательный), Русификатор test, Ты/Вы test, Смена языков test |
| **Primary metrics** | Русификатор accuracy, Смена языков compliance, **human eval sample score** |
| **Подход к оценке** | Бенчмарки типа ruMMLU/ruHumanEval оценивают capabilities, а не качество языка. Для оценки качества русского нужны: **ручная разметка** + **классификатор-оценщик** |
| **Ручная разметка** | Регулярная разметка выборки ответов (100-200 per eval round) по критериям: грамматика, стилистика, естественность, code-switching. Перекрытие 3+. |
| **Классификатор-оценщик** | Русификатор LoRA или Skywork-0.6B как автоматический scorer RU-quality на каждом checkpoint |
| **Reward** | Русификатор LoRA + Ты/Вы reward + lang-switch penalty |
| **Конкуренты** | Qwen-3.5: 201 langs; GLM-5: SWE-bench Multilingual 56.2 |
| **Experiments** | EXP-RU-1: baseline; EXP-RU-2: candidate; EXP-RU-3: RU stress; EXP-RU-4: RU mix ablation; EXP-RU-5: Русификатор on/off; **EXP-RU-EVAL-1**: прогнать classifier + ручную разметку на одном сете, замерить correlation |
| **Gate** | MERA >= target (#1 RU), Русификатор >= baseline, Ты/Вы >= 95%, human eval >= threshold |
| **Rollback** | MERA падает >2% или code-switching >3% |

### 7.8 Safety

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | Internal safety test, ruHateSpeech, ruEthics, jailbreak test, **сеты ЦУР** |
| **Primary metrics** | Harmful refusal rate (>= 99%), false refusal rate (<= target), **per-domain refusal rate** |
| **Reward** | InternLM2-7B (start) -> свой Safety reward. Precision >>> Recall |
| **Experiments** | EXP-SAFE-1: baseline; EXP-SAFE-2: candidate; EXP-SAFE-3: jailbreak stress; EXP-SAFE-4: weight ablation; **EXP-SAFE-DOMAIN-1**: per-domain refusal rate baseline; **EXP-SAFE-DOMAIN-2**: domain-weighted safety reward |
| **Gate** | Refusal >= 99%, false refusal <= target, jailbreak resistance >= target, **per-domain compliance >= threshold** |
| **Rollback** | Любое падение refusal ниже порога -> immediate stop |
| **Текущий статус** | Safety reward: нет данных, инструкция в работе |

**Доменная разбивка Safety:**

| Домен | Источник данных | Статус |
|-------|----------------|--------|
| Hate speech | ruHateSpeech + доп. сбор | Есть бенч |
| Political / sensitive (РФ/СНГ) | **Собрать корзинку из потока логов** — политические специфики, актуальные sensitive topics | НЕТ, надо собрать |
| Extremism | ЦУР + internal | Частично |
| Self-harm | ЦУР + internal | Частично |
| Sexual content | Internal test | Есть |
| Fraud / scam | Собрать из прод-логов | НЕТ |
| PII leakage | Internal test | Есть |

**Сеты ЦУР:** интегрировать safety-тесты от ЦУР (отдел оценки рисков) в evaluation pipeline. Использовать как дополнительный gate на release readiness.

### 7.9 Factuality

| Элемент | Детали |
|---------|--------|
| **Бенчмарки** | SimpleQA (primary), TriviaQA (sanity), **FACTS Benchmark Suite** (Google DeepMind: Parametric, Search, Grounding) |
| **Primary metrics** | SimpleQA accuracy, not-attempted rate, **FActScore** (factual precision для long-form) |
| **Alert** | Падение >2% на SimpleQA |

**Reward для фактологии — варианты:**

| Вариант | Как работает | Latency | Качество | Когда использовать |
|---------|-------------|---------|----------|-------------------|
| **NLI-based classifier** (~1B) | Проверяет consistency между claim и evidence | ~100ms | Средне (consistency, не grounding) | **Online RL** (fast enough) |
| **Search-grounded (RAG)** | Retrieve facts → compare с ответом | 5-15 сек | Высокое (grounded) | **Gate evaluation** (offline, slow but accurate) |
| **Self-consistency** | Multiple rollouts → agreement | ~N×inference | Noisy | Fallback if others unavailable |
| **Meta approach** | Joint: factual_precision + detail + relevance | ~200ms | Высокое, addresses hacking | **Online RL v2** после calibration |

Референс: [Meta — Learning to Reason for Factuality](https://arxiv.org/abs/2508.05618) — joint optimization снижает hallucination на 23pp, при этом не деградируя response quality.

**Рекомендация:** NLI-based classifier для online RL (fast), Search-grounded для gate evaluation (slow but accurate).

| ID | Описание | Что замеряем |
|----|----------|-------------|
| EXP-FACT-1 | Baseline SimpleQA + TriviaQA accuracy | Current factuality level |
| EXP-FACT-2 | NLI-based factuality reward в RL loop | SimpleQA delta, response quality delta |
| EXP-FACT-3 | Search-grounded eval на gate checkpoints (offline) | FActScore, FACTS Benchmark scores |
| EXP-FACT-4 | Ablation NLI reward weight | Optimal weight без degradation других треков |

### 7.10 Cross-cutting

| Срез | Бенч | Метрика | Alert |
|------|------|---------|-------|
| **Latency** | B2C query set | Tokens/sec, TTFT, chain length | Chain > max budget |
| **Robustness** | Noisy versions of key benches | Delta accuracy clean vs noisy | Delta > 5% |
| **Multi-domain consistency** | ALL tracks per checkpoint | Cross-domain regression matrix | Regression в любом треке |

---

## 8. Roadmap до Production-Ready Checkpoint

### Stage-Gate Pipeline

```
Cold Start (Stage 1.5 → SFT) -> Dev Prompt -> Data Readiness -> Reward Readiness -> Training Readiness -> Eval Readiness -> Release Readiness
```

### Gate 0: Cold Start Readiness

| Критерий | Требование | Текущий статус |
|----------|-----------|----------------|
| Stage 1.5 данные >= 2M | Per domain coverage documented | NOT DONE |
| SFT данные 100-200k, провалидированы | Quality audit passed | NOT DONE |
| Masking strategy выбрана (EXP-SFT-5) | Ablation done | NOT DONE |
| Наш SFT обыгрывает дистил Qwen-3/3.5/GLM-5 | EXP-SFT-1..4 documented | NOT DONE |
| Developer system prompt <= 1k tokens | Verified, edge cases ok | NOT DONE |

### Gate 1: Data Readiness

| Критерий | Требование | Текущий статус |
|----------|-----------|----------------|
| Rollout queries dedup/leak checked | Done, documented | NOT DONE |
| Все reward train sets >= 2k | Per checker | Partial (Safety: 0, Vikacheck-reasoning: 0) |
| Все reward test sets >= 500 | Per checker | NOT MET (Русификатор 300, Ты/Вы 288) |
| Хранилище определено, pipeline ингеста работает | Operational | NOT DONE |
| Доменная смесь валидирована (хотя бы 1 ablation) | Done | NOT DONE |
| Хранение с per-sample reward_config | Формат определён, pipeline работает | NOT DONE |

### Gate 2: Reward Readiness

| Критерий | Требование | Текущий статус |
|----------|-----------|----------------|
| Все OS-rewards deployed и замерены на internal test | Baseline numbers documented | NOT DONE |
| Свои rewards замерены где готовы (Эквивалентность, MD) | P/R/F1 documented | Partial |
| Safety reward operational (OS or own) | Running | NOT DONE |
| Reward routing v1 operational | ArmoRM + rule overrides | NOT DONE |
| Формула агрегации reward определена и протестирована | EXP-AGG-* done | NOT DONE |
| Детерминированные rewards (loops, format) deployed | Running | NOT DONE |

### Gate 3: Training Readiness

| Критерий | Требование | Текущий статус |
|----------|-----------|----------------|
| Round 3 эксперименты завершены (clip, dr.grpo) | Results documented | NOT DONE |
| pass@k fix deployed | Correct computation verified | NOT DONE |
| Eval sweep pipeline автоматизирован | All tracks per checkpoint | NOT DONE |
| Collapse mitigation identified | At least 1 method validated (scheduler + clip OR DPPO OR sampler) | Partial (scheduler helps) |
| Process reward для math/code протестирован | EXP-PRM-* results documented | NOT DONE |

### Gate 4: Eval Readiness

| Критерий | Требование | Текущий статус |
|----------|-----------|----------------|
| Все capability-треки замерены на baseline | Numbers documented | NOT DONE |
| Qwen-3.5 / GLM-5 замерены на тех же бенчах | For fair comparison | NOT DONE |
| Promotion thresholds зафиксированы по каждому треку | Documented | NOT DONE |
| OBM format validation прошёл | 50 tasks, >= 90% | NOT DONE |
| Safety per-domain breakdown готов | ЦУР + political + all domains | NOT DONE |

### Gate 5: Release Readiness

| Критерий | Требование |
|----------|-----------|
| Все primary metrics >= promotion threshold | Per track |
| Ни один трек не регрессировал > allowed delta | Cross-domain matrix clean |
| Safety gate passed | Refusal >= 99%, false refusal <= target, per-domain ok |
| Factuality gate passed | SimpleQA >= threshold |
| Canary run in prod-like environment | 48h stable |

### Приоритезированный plan of action

| Phase | Что делаем | Длительность (оценка) |
|-------|-----------|----------------------|
| **Phase -1** | Cold Start: Stage 1.5 расширение до 2M, SFT 100-200k + masking ablation, Developer prompt сокращение до 1k | 3-4 недели (параллельно с Phase 0) |
| **Phase 0** | Data: dedup/leak check, хранилище + reward_config format + метки domain/difficulty/context_length, Safety инструкция. ML: pass@k fix, eval sweep pipeline | 1-2 недели |
| **Phase 1** | Deploy OS-rewards, замер baselines. Round 3 эксперименты. Arena разметка. Детерминированные rewards. OBM 50 tasks | 2-3 недели |
| **Phase 2** | Train own rewards (по протоколу 3c). DPPO / sampler experiments. SOTA datasets integration. Process reward эксперименты. Agent data synthesis. Reward aggregation эксперименты | 3-4 недели |
| **Phase 3** | Canary runs с лучшей комбинацией. Eval sweep. Fix regressions. Safety per-domain. Factuality eval | 2-3 недели |
| **Phase 4** | Promotion gate. Prod-like canary 48h. Release | 1 неделя |

---

## 9. Source Links

### Внутренние

- [online-rl.md](start_pack/online-rl.md) — форматы, хранение, 3 типа датасетов
- [datasets.md](start_pack/datasets.md) — статусы per-checker
- [reward-lora.md](start_pack/reward-lora.md) — статусы reward/LoRA, задачи
- [lora-tests.md](start_pack/lora-tests.md) — результаты тестов LoRA
- [RL/RnD/Setups (+baseline).md](start_pack/RL/RnD/Setups%20(+baseline).md) — все эксперименты Round 1-3
- [RL/Backlog VERL.md](start_pack/RL/Backlog%20VERL.md) — tech debt pipeline
- [RL/Tools&Env's/](start_pack/RL/Tools&Env's/) — tool-calling, agents, environments

### Внешние (только top-tier)

**Конкуренты:**
- DeepSeek V3.2: [HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-V3.2), [GitHub](https://github.com/deepseek-ai/DeepSeek-V3.2-Exp)
- Qwen-3.5: [GitHub](https://github.com/QwenLM/Qwen3.5)
- Qwen3-Coder-Next: [arxiv 2603.00729](https://arxiv.org/abs/2603.00729)
- GLM-5: [GitHub](https://github.com/zai-org/GLM-5), Slime: [GitHub](https://github.com/THUDM/slime)
- Kimi K2: [arxiv 2507.20534](https://arxiv.org/abs/2507.20534)

**Бенчмарки:**
- LiveCodeBench: [livecodebench.github.io](https://livecodebench.github.io/)
- SWE-bench Verified: [swebench.com/verified.html](https://www.swebench.com/verified.html)
- BFCL v4: [gorilla.cs.berkeley.edu/leaderboard.html](http://gorilla.cs.berkeley.edu/leaderboard.html)
- MERA v1.2.0: [mera.a-ai.ru](https://mera.a-ai.ru/en/text/about)
- SimpleQA: [github.com/openai/simple-evals](https://github.com/openai/simple-evals)
- RewardBench: [HuggingFace](https://huggingface.co/spaces/allenai/reward-bench)
- ProcessBench: [GitHub](https://github.com/QwenLM/ProcessBench)
- PRMBench: [arxiv 2501.03124](https://arxiv.org/abs/2501.03124)
- Socratic-PRMBench: [arxiv 2505.23474](https://arxiv.org/abs/2505.23474)
- FACTS Benchmark: [DeepMind](https://deepmind.google/blog/facts-grounding-a-new-benchmark-for-evaluating-the-factuality-of-large-language-models/)

**Reward models:**
- Skywork-Reward-V2: [HuggingFace](https://huggingface.co/Skywork/Skywork-Reward-V2-Llama-3.1-8B), [arxiv 2507.01352](https://arxiv.org/abs/2507.01352)
- ArmoRM: [HuggingFace](https://huggingface.co/RLHFlow/ArmoRM-Llama3-8B-v0.1), [arxiv 2406.12845](https://arxiv.org/abs/2406.12845)
- ToolRM: [arxiv 2509.11963](https://arxiv.org/abs/2509.11963)
- InternLM2-Reward: [HuggingFace](https://huggingface.co/collections/internlm/internlm2-reward)

**Datasets:**
- LMArena Human Preference 55k: [HuggingFace](https://huggingface.co/datasets/lmarena-ai/arena-human-preference-55k)
- WildChat: [HuggingFace](https://huggingface.co/datasets/allenai/WildChat)
- WildFeedback: [HuggingFace](https://huggingface.co/datasets/microsoft/WildFeedback)
- ODA-Mixture-500k: [HuggingFace](https://huggingface.co/datasets/OpenDataArena/ODA-Mixture-500k)
- OpenCodeInstruct: [arxiv 2504.04030](https://arxiv.org/abs/2504.04030)
- SWE-smith: [GitHub](https://github.com/SWE-bench/SWE-smith)

**Методология:**
- RLHF Workflow (online iterative): [arxiv 2405.07863](https://arxiv.org/abs/2405.07863)
- Reward Routing (BayesianRouter): [arxiv 2510.02850](https://arxiv.org/abs/2510.02850)
- Reward Hacking / Emergent Misalignment: [arxiv 2511.18397](https://arxiv.org/abs/2511.18397)
- GRAM (Generative Reward): [arxiv 2506.14175](https://arxiv.org/abs/2506.14175)
- GDPO (Multi-reward aggregation): [arxiv 2601.05242](https://arxiv.org/abs/2601.05242)
- Nemotron-Cascade (Cascade RL): [arxiv 2512.13607](https://arxiv.org/abs/2512.13607)
- OTC-PO (Tool efficiency): [arxiv 2504.14870](https://arxiv.org/abs/2504.14870)
- ToolRLA (Multiplicative reward): [arxiv 2603.01620](https://arxiv.org/abs/2603.01620)
- SpecRA (Loop detection): [openreview xVO4BqmzVD](https://openreview.net/forum?id=xVO4BqmzVD)
- ARA (Adversarial Reward Auditing): [arxiv 2602.01750](https://arxiv.org/abs/2602.01750)
- PAR (Preference As Reward): [arxiv 2502.18770](https://arxiv.org/abs/2502.18770)
- Gradient Regularization: [arxiv 2602.18037](https://arxiv.org/abs/2602.18037)
- Meta Factuality RL: [arxiv 2508.05618](https://arxiv.org/abs/2508.05618)
- Task selection (P(mixed)): [openreview QXrZ0Y3yGJ](https://openreview.net/pdf?id=QXrZ0Y3yGJ)
- Dynamic sampling: [openreview](https://openreview.net/pdf/b3c7dc1539020266bcb75760d12e6850036356c8.pdf)
- R1-Searcher (RL for search): [arxiv 2503.05592](https://arxiv.org/abs/2503.05592)
- Search-o1 (Agentic search reasoning): [search-o1.github.io](https://search-o1.github.io/)
- VeRPO (Dense code reward): [arxiv 2601.03525](https://arxiv.org/abs/2601.03525)
- RLEF (Execution feedback RL): [ICML 2025](https://proceedings.mlr.press/v267/gehring25a.html)
