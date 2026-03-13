# Дима — Infra: Storage + OS rewards deploy + Determ checks + OS reward matrix

**Состав:** 3 человека (Дм-1, Дм-2, Дм-3)
**Scope:** Хранилище данных, deploy 5 OS reward моделей, детерминированные чеки, OS reward matrix 5x7.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | **Storage** (Дм-1), **OS rewards x5 + verify** (Дм-2), **Determ checks** (Дм-3) | Storage works; >= 3 OS rewards online + P/R/F1; determ API P >= 95% | **Infra foundation + первые reward числа** | - [ ] |
| [W2](#w2) | Inventory + ingest (Дм-1), **OS reward matrix 5x7** (Дм-2), LoRA pipeline test (Дм-3) | All datasets in storage; **matrix P/R/F1 25+ cells**; pipeline works | **OS reward matrix (work/don't work)** | - [ ] |

**Ключевые риски:**
- GPU capacity для 5 OS rewards одновременно → приоритет: Skywork-0.6B + ArmoRM-8B + InternLM2-7B (must-have), Skywork-8B + ToolRM-1.7B (nice-to-have)
- LoRA pipeline нестабилен → тест 100 записей перед production
- OS rewards могут не работать на RU-доменах → документировать как DON'T WORK, искать альтернативы

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17–21)

<a id="дм1-w1"></a>

**Дима-1: Хранилище данных + pipeline ингеста**
- Вход: Данные разбросаны по `/home/valenchik/`, train_catalog, notebooks.
- Задача:
  - (1) S3/MinIO setup: endpoint, credentials, botocore или mc CLI.
  - (2) Бакеты:
    - `rollout-queries` — промпты для RL
    - `reward-train` — данные для обучения reward моделей
    - `reward-test` — тест-сеты по reward доменам
    - `stage15` — Stage 1.5 данные
    - `sft` — SFT данные
  - (3) CLI команды: upload, download, list, delete.
  - (4) Per-sample reward_config формат (JSON schema).
  - (5) Round-trip тест: upload 100 записей → download → verify.
- DoD: Endpoint работает. Round-trip 100 OK. README с примерами.
- Day 1: выбор S3/MinIO. Day 2-3: CLI + bucket setup. Day 4: данные. Day 5: README + тест.

**Per-sample format (обязательные поля):**
```json
{
  "id": "uuid",
  "prompt": "...",
  "response": "...",
  "domain": "math|code|if|ru|stem|arena|b2c|safety",
  "difficulty": "easy|medium|hard",
  "context_length": "short|medium|long",
  "source": "pipeline|human|opensource",
  "teacher": "qwen3|deepseek|gemini|human",
  "reward_config": {
    "md_check": true,
    "ru_check": false,
    "safety_check": false,
    "code_exec": false,
    "arena_pref": true
  }
}
```

<a id="дм2-w1"></a>

**Дима-2: OS reward модели x5 + verification**
- Вход: Skywork-8B, Skywork-0.6B, ArmoRM-8B, InternLM2-7B, ToolRM-1.7B.
- Задача:
  - Deploy каждой модели (vLLM): endpoint, health check, latency тест.
  - **Verification (BLOCKING):** 100 labeled примеров (50 pos + 50 neg) → P/R/F1 → work/don't work.
  - Первые числа к W1 Friday: хотя бы для 3 must-have моделей.
- DoD: >= 3 моделей online. Таблица: model | latency p50/p95 | GPU | P/R/F1. Golden test set.
- Приоритет deploy: (1) Skywork-0.6B, (2) ArmoRM-8B, (3) InternLM2-7B, (4) Skywork-8B, (5) ToolRM-1.7B.
- Риск: GPU memory → использовать разные GPU nodes, не один сервер.

**Таблица deploy (шаблон):**

| Модель | GPU | RAM | Latency p50 | Latency p95 | P на 100 | R на 100 | F1 | Вердикт |
|--------|-----|-----|------------|------------|---------|---------|-----|---------|
| Skywork-0.6B | ? | ? | ? | ? | ? | ? | ? | ? |
| ArmoRM-8B | ? | ? | ? | ? | ? | ? | ? | ? |
| InternLM2-7B | ? | ? | ? | ? | ? | ? | ? | ? |
| Skywork-8B | ? | ? | ? | ? | ? | ? | ? | ? |
| ToolRM-1.7B | ? | ? | ? | ? | ? | ? | ? | ? |

<a id="дм3-w1"></a>

**Дима-3: Детерминированные чеки v1**
- Задача: FastAPI service с 4 endpoints:
  - `/loop_detected` — обнаружение цикличных tool calls / повторяющихся блоков.
  - `/repetition_ratio` — доля повторяющихся n-gram в response.
  - `/json_valid` — валидация JSON в response.
  - `/latex_valid` — валидация LaTeX формул.
- DoD: API works. P >= 95% на тест-сете 50 примеров каждый. Latency < 50ms p95.
- Day 1-2: FastAPI skeleton + loop_detected. Day 3: repetition_ratio. Day 4: json_valid + latex_valid. Day 5: тест 50 примеров каждый.

---

<a id="w2"></a>

### W2 (Mar 24–28)

<a id="дм1-w2"></a>

**Дм-1: Ingest ALL datasets → inventory table**
- Задача: Залить все datasets (Stage 1.5 seeds, SFT данные, reward train/test) в хранилище.
- DoD: Inventory table: dataset | bucket | size | record_count | last_updated.

<a id="дм2-w2"></a>

**Дм-2: OS reward matrix 5x7 (BLOCKING)**
- Задача: Запустить каждый OS reward на каждом тест-домене.
  - Домены: MD, Русификатор, Смена, Эквивалентность, Ты/Вы, Arena, Safety.
  - Для каждой пары (reward, domain): P/R/F1 на 100-200 примерах.
  - Вердикт: WORK (F1 >= 0.75) / MARGINAL (0.5-0.75) / DON'T WORK (< 0.5).
- DoD: **Matrix 5x7 P/R/F1 (35+ ячеек). Вердикт для каждой ячейки. Рекомендации для RL v0.**
- Day 6-7: Запуск прогонов (параллельно все домены). Day 8-9: Анализ. Day 10: Matrix + рекомендации.

**OS reward matrix (шаблон вывода):**

| Reward | MD | Русификатор | Смена | Эквив. | Ты/Вы | Arena | Safety | Best use |
|--------|----|-----------|----|-----|-----|-------|--------|----------|
| Skywork-0.6B | P/R/F1 | ... | ... | ... | ... | ... | ... | ? |
| ArmoRM-8B | P/R/F1 | ... | ... | ... | ... | ... | ... | ? |
| InternLM2-7B | P/R/F1 | ... | ... | ... | ... | ... | ... | ? |
| Skywork-8B | P/R/F1 | ... | ... | ... | ... | ... | ... | ? |
| ToolRM-1.7B | P/R/F1 | ... | ... | ... | ... | ... | ... | ? |

<a id="дм3-w2"></a>

**Дм-3: LoRA training pipeline — end-to-end test**
- Задача: Подготовить LoRA training pipeline (для Поли/SFT):
  - Input format: reward train data → LoRA training job.
  - End-to-end тест: 100 записей → train 10 steps → checkpoint → eval.
- DoD: Pipeline works на 100 записях. Checkpoint сохраняется. Eval запускается.

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-RWVER-1 | W1-2 | Дм-2 | 5 OS rewards x 100 примеров → P/R/F1 |
| EXP-OSMATRIX | W2 | Дм-2 | OS reward matrix 5x7 → work/don't work |
| EXP-DETERM | W1 | Дм-3 | Determ checks API: P >= 95%, latency < 50ms |
