# Даша — Platform: B2C reward baselines

**Состав:** 4 человека (Д-1, Д-2, Д-3, Д-4)
**Scope:** Code interpreter reward checker, websearch reward checker, text2image trigger checker, B2C baseline report.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Interpreter P/R (Д-1), Search P/R (Д-2), T2I P/R (Д-3), B2C report (Д-4) | P/R/F1 по каждому checker на 100 примерах | **3 checker baselines** | - [ ] |
| [W2](#w2) | EXP-B2C-1 full report (Д-1+2), reward data >= 500/func (Д-3+4) | Baseline report; >= 500 per function | **EXP-B2C-1 baseline** | - [ ] |

**Ключевые риски:**
- Websearch NLI дорого → keyword matching v0 сначала, NLI offline
- Fabricated URLs → hard penalty -1.0, детектируется regex
- Interpreter timeout > 5s → async execution с timeout handle

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17–21)

<a id="д1-w1"></a>

**Даша-1: Code interpreter reward checker**
- Задача: Построить checker: execution_success + answer_match + code_quality_penalties.
  - execution_success: code выполнился без ошибок → 0/1.
  - answer_match: output совпадает с ожидаемым → 0/1.
  - penalties: timeout, infinite loop, security violation → -1.0.
- DoD: P/R/F1 на 100 примерах (50 pos + 50 neg). Latency < 5s p95.
- Day 1-2: Sandbox integration. Day 3-4: Checker logic. Day 5: eval на 100 примерах.

**Метрики для фиксации:**

| Метрика | Значение |
|---------|---------|
| Precision | ? |
| Recall | ? |
| F1 | ? |
| Latency p50 | ? |
| Latency p95 | ? |
| Timeout rate | ? |

<a id="д2-w1"></a>

**Даша-2: Websearch reward checker**
- Задача: 4-компонентный checker:
  - query_relevance: запрос релевантен задаче → 0/1.
  - citation_accuracy: цитаты из реального источника → 0/1.
  - answer_grounding: ответ подкреплён результатами поиска → 0/1.
  - format_compliance: формат поиска корректен → 0/1.
  - Fabricated URL: любой выдуманный URL → немедленный -1.0 по всему response.
- V0: keyword matching (быстро). NLI offline — опционально W2.
- DoD: Per-component P/R на 100 примерах. Timeout handle 3s. Fabricated URL detection rate.
- Риск: NLI слишком медленно → keyword-matching v0, NLI в W2.

<a id="д3-w1"></a>

**Даша-3: Text2image trigger checker**
- Задача: Определяет, должен ли assistant вызвать text2image функцию.
  - Бинарный: trigger_required / no_trigger.
  - Ложные срабатывания (precision) критичны — модель не должна галлюцинировать t2i.
- DoD: Trigger P/R на 100 примерах (50 pos: явный запрос картинки / 50 neg: текстовые задачи).

<a id="д4-w1"></a>

**Даша-4: B2C baseline report**
- Задача: Собрать baseline метрики текущего поведения 10b_toolmind на B2C задачах:
  - Call distribution: сколько tool calls в среднем на запрос.
  - Loop rate: % ответов с looping tool calls.
  - Format compliance: % ответов с корректным tool call форматом.
  - Redundancy rate: % ненужных tool calls.
- DoD: B2C baseline report с числами.

---

<a id="w2"></a>

### W2 (Mar 24–28)

<a id="д1-w2"></a>
<a id="д2-w2"></a>

**Д-1+2: EXP-B2C-1 — полный baseline report**
- Задача:
  - Запустить 10b_toolmind на 200+ B2C задачах (interpreter + search + t2i).
  - Сравнить с Qwen3.5-2B если есть tool-use capability.
  - Зафиксировать: call distribution, loop rate, format compliance, redundancy.
- DoD: EXP-B2C-1 отчёт с числами. Таблица baseline 10b vs Qwen3.5.

<a id="д3-w2"></a>
<a id="д4-w2"></a>

**Д-3+4: B2C reward train data >= 500 per function**
- Задача: Собрать labeled примеры для reward training:
  - interpreter: >= 500 (250 pos + 250 neg).
  - search: >= 500 (250 pos + 250 neg).
  - text2image: >= 200 (100 pos + 100 neg, учитывая редкость).
- DoD: >= 500/function в хранилище с labels.

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-INTERP-CHECK | W1 | Д-1 | Interpreter checker P/R/F1 на 100 |
| EXP-SEARCH-CHECK | W1 | Д-2 | Search checker P/R/F1 на 100 |
| EXP-T2I-CHECK | W1 | Д-3 | T2I trigger checker P/R на 100 |
| EXP-B2C-1 | W2 | Д-1+2 | B2C baseline: call dist, loop, format, redundancy |
