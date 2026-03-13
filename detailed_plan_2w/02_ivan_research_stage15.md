# Иван — Research: Stage 1.5 данные + Leak Check + OBM Format

**Состав:** 3 человека (И-1, И-2, И-3)
**Scope:** Данные для baseline, Solve First pipeline запуск, Leak check, OBM format validation.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Данные для baseline: math/code seeds (И-1+2), **Leak check** (И-3) | Seeds готовы; **Leak report** | Overlap %, clean dataset, leak status | - [ ] |
| [W2](#w2) | Domain coverage audit (И-1+2), **OBM format 50 задач** (И-3) | Coverage table; **retention >= 90%** | **Leak fix + OBM format gate** | - [ ] |

**Ключевые риски:**
- Leak check находит overlap > 5% → немедленная фильтрация + re-split eval/reward/rollout
- OBM format retention < 90% → анализ W2 Day 1-2, патч системного промпта

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17–21)

<a id="и1-w1"></a>

**Иван-1: Данные для baseline — math + code seeds**
- Вход: Solve First pipeline, Qwen-3 как учитель, seeds: MATH 12k + GSM8K 8k + AIME + Olympiad + HumanEval 164 + MBPP 500 + LCB seeds.
- Задача: Подготовить seeds для Solve First pipeline (math + code). Проверить throughput.
  - Запустить первые прогоны: math 50k+ + code 30k+ для baseline данных.
  - Quality audit 100 random: прямой reasoning без Wait/передумываний.
- DoD: >= 80k записей (math + code), quality audit пройден, файлы в хранилище.
- Day 1: Seeds preparation + pipeline check. Day 2-3: прогон math. Day 4-5: прогон code + audit.
- Риск: throughput < 50k/день → параллелить GPU.

**Фильтр reasoning качества (Stage 1.5):**
```
запрещённые паттерны:
- "Wait", "Hmm, wait", "Actually"
- "Let me reconsider", "I made an error"  
- "подождите", "я ошибся", "давайте переосмыслим"
допустимо:
- "Step 1:", "Let me calculate:", "Checking:"
- нумерованные шаги, финальная проверка
```

<a id="и2-w1"></a>

**Иван-2: Данные для baseline — IF/Arena seeds**
- Вход: IFEval seeds 1k, WildChat RU 5k, Arena seeds.
- Задача: Подготовить IF/Arena/RU seeds для pipeline. Запустить первые прогоны.
  - IF прогон: 20k+ записей (IFEval-style tasks).
  - Метки: domain, difficulty (easy/medium/hard), context_length bucket (< 1k / 1-4k / 4k+).
- DoD: >= 20k IF/Arena/RU записей с labels, в хранилище.
- Примечание: **Все записи Stage 1.5 и SFT должны иметь метки: domain / difficulty / context_length.** Это нужно для online RL sampling.

<a id="и3-w1"></a>

**Иван-3: Dedup + Contamination + LEAK CHECK (BLOCKING)**
- Вход: Все rollout queries + все eval benchmarks + все reward test sets.
- Задача:
  - (1) Exact dedup (sha256 hash) всех query/prompt полей.
  - (2) Fuzzy dedup: MinHash + LSH, threshold 0.85 (Jaccard).
  - (3) **LEAK CHECK**: eval data в train data. Три пересечения:
    - rollout-queries ∩ eval-benchmarks
    - reward-train ∩ eval-benchmarks
    - stage15-data ∩ eval-benchmarks
  - (4) Фиксация: скрипт для мониторинга новых добавлений.
- DoD: Отчёт: overlap %, список sample IDs, leak status (CLEAN/CONTAMINATED). Скрипт в репо.
- Day 1-2: Сбор всех датасетов, hash dedup. Day 3-4: MinHash LSH. Day 5: отчёт.
- CRITICAL: Если overlap > 5% в любом пересечении → немедленно эскалировать, фильтрация до конца W1.

**Формат отчёта:**
```
Leak Check Report (W1)
======================
rollout-queries ∩ MATH-500:   N exact, M fuzzy (threshold 0.85)
rollout-queries ∩ AIME:       N exact, M fuzzy
rollout-queries ∩ LCB-v6:     N exact, M fuzzy
rollout-queries ∩ IFEval:     N exact, M fuzzy
rollout-queries ∩ BFCL-v4:   N exact, M fuzzy
reward-train ∩ all eval:      N exact, M fuzzy
Status: CLEAN / CONTAMINATED
Action items: [список фильтраций]
```

---

<a id="w2"></a>

### W2 (Mar 24–28)

<a id="и1-w2"></a>

**Иван-1+2: Domain coverage audit и расширение**
- Задача:
  - Аудит накопленных данных: distribution по доменам.
  - Заполнить пробелы: STEM/RU/B2C если недостаточно.
  - Проверить balance: нет одного домена > 30%.
- DoD: Distribution table domain x count x %, balance OK.

**Целевое распределение Stage 1.5 (долгосрочное, 2M):**

| Домен | % | Бенч-coverage |
|-------|---|---------------|
| Math reasoning | 25% | MATH-500, AIME |
| Code | 20% | LCB v6 |
| Instruction Following | 15% | IFEval, Vikacheck |
| RU reasoning | 10% | MERA, Русификатор |
| STEM / Science | 10% | MMLU |
| Arena / helpfulness | 10% | Arena-Hard |
| B2C / tools | 5% | BFCL |
| Safety | 5% | Safety test |

<a id="и3-w2"></a>

**Иван-3: OBM format 50 задач validation**
- Вход: 50 задач в формате ОБМ, 10b_toolmind.
- Задача:
  - Прогнать 50 задач через модель.
  - Проверить retention: модель сохраняет формат ОБМ >= 90%?
  - Анализ ошибок: где модель ломает формат?
  - **Если retention < 90%**: подготовить список нарушений → передать ML для патча dev prompt.
- DoD: OBM format retention % на 50 задачах. Таблица нарушений. Go/no-go для developer system prompt.
- Day 6-7: Setup + прогон. Day 8-9: Анализ ошибок. Day 10: Отчёт.

**Если leak найден в W1 → W2 Day 6-7: фикс и re-check.**

---

## Метки для online RL (требование)

**Каждая запись в Stage 1.5, SFT и reward sets должна иметь следующие поля:**

```json
{
  "id": "...",
  "prompt": "...",
  "response": "...",
  "domain": "math|code|if|ru|stem|arena|b2c|safety",
  "difficulty": "easy|medium|hard",
  "context_length": "short|medium|long",
  "source": "...",
  "teacher": "qwen3|deepseek|gemini|human",
  "quality_score": 0.0-1.0,
  "reward_config": {
    "md_check": true,
    "ru_check": false,
    ...
  }
}
```

Метки нужны для:
- Online RL: сэмплирование по домену/сложности/длине контекста
- Мониторинг: per-domain performance
- Debugging: анализ регрессий

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-LEAK-1 | W1 | И-3 | Leak check: rollout ∩ eval ∩ reward |
| EXP-OBM-FORMAT | W2 | И-3 | OBM format retention на 50 задачах |
