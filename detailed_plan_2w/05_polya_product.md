# Поля — Product: Reward test sets + OS reward verification

**Состав:** 6 человек (П-1, П-2, П-3, П-4, П-5, П-6)
**Scope:** Подготовка reward test sets в ОБМ, OS reward verification по всем доменам, таблица work/don't work.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Кто | Задача | DoD | Status |
|---|-----|--------|-----|--------|
| **[W1](#w1)** | П-1 | MD test >= 800 → ОБМ | Файл в ОБМ, distribution table | - [ ] |
| | П-2 | Русификатор test >= 300 + Смена test >= 300 | Оба файла готовы | - [ ] |
| | П-3 | Safety pilot: инструкция + 200 примеров + IAA | IAA > 0.8 | - [ ] |
| | П-4 | Arena: разметка запущена, тест 2k отложен | Разметка started | - [ ] |
| | П-5 | Эквивалентность test >= 1.1k + Ты/Вы test >= 288 | Оба файла готовы | - [ ] |
| | П-6 | Vikacheck reward: train >= 800, test >= 350 | Файлы готовы | - [ ] |
| **[W2](#w2)** | П-1 | **OS reward verification: MD test** | P/R/F1 по каждому OS reward на MD test | - [ ] |
| | П-2 | **OS reward verification: Русификатор + Смена** | P/R/F1 по каждому OS reward | - [ ] |
| | П-3 | **OS reward verification: Safety subset** | P/R/F1 по каждому OS reward | - [ ] |
| | П-4 | **OS reward verification: Arena subset** | P/R/F1 по каждому OS reward | - [ ] |
| | П-5 | **OS reward verification: Эквивалентность + Ты/Вы** | P/R/F1; Эквивалентность LoRA start | - [ ] |
| | П-6 | **Reward verification report: work/don't work** | Итоговая таблица + рекомендации | - [ ] |

**Ключевые риски:**
- Safety данных нет (P0 gap) → ЦУР fallback + InternLM2 OS reward для safety в W1
- Arena разметка медленная → не блокирует: Arena subset 200 примеров из уже готовых достаточно для W2 verification
- OS rewards не работают на RU-доменах → документировать, искать RU-specific альтернативы

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17–21)

<a id="п1-w1"></a>

**Поля-1: MD-check тест >= 800 → ОБМ**
- Вход: ~4k размечено, ~15-20k прогонов с md_check деtect.
- Задача: Выгрузить → стратификация (50% pos, 50% neg) → тест >= 800 → загрузить в ОБМ.
- DoD: >= 800 примеров в ОБМ + distribution table (source, type, balance).
- Примечание: Этот тест используется в OS reward verification W2. Нужен к W2 Day 1.

<a id="п2-w1"></a>

**Поля-2: Русификатор test >= 300 + Смена языков test >= 300**
- Вход: Русификатор: ~7.6k в разметке. Смена: Thinking 1086 + Answer 804 + тест ~300.
- Задача:
  - Русификатор: reward format, reasoning (протокол 3c) → test >= 300 (стратифицировано: RU correct / RU violation / non-RU).
  - Смена: дополнить до >= 300 тест-примеров (pos/neg).
- DoD: Русификатор test >= 300, Смена test >= 300 — оба файла готовы.

<a id="п3-w1"></a>

**Поля-3: Safety pilot — инструкция + 200 примеров + IAA**
- Вход: НИЧЕГО (P0 gap). Fallback: ЦУР + InternLM2-7B OS reward.
- Задача:
  - Написать инструкцию: бинарный (safe/unsafe), 7 доменов (ЦУР, политика, ...).
  - Seed 200 примеров → пилот разметки, перекрытие 5 исполнителей.
  - IAA (Cohen's kappa или Fleiss).
- DoD: Инструкция утверждена, IAA > 0.8, 200 примеров размечено.
- Риск: IAA < 0.8 → упростить до бинарной без доменов, перепилотировать Day 4-5.
- Fallback для W2: использовать InternLM2-7B как proxy safety reward пока нет своих данных.

<a id="п4-w1"></a>

**Поля-4: Arena 28k → разметка start**
- Вход: 28k в train_catalog (MR 317).
- Задача: Pair-wise разметка → стратификация → запустить разметку.
  - Тест 2k: отложить (нужен к W4 по основному плану).
  - Для W2 OS verification: выделить 200-примерный subset из имеющихся готовых.
- DoD: Разметка запущена. 200-примерный Arena subset для OS verification готов к W2.

<a id="п5-w1"></a>

**Поля-5: Эквивалентность test >= 1.1k + Ты/Вы test >= 288**
- Вход: Эквивалентность: 2.1k train + тест нужен. Ты/Вы: ~288 тест примеров.
- Задача:
  - Эквивалентность: разбить данные → тест >= 1.1k (50% pos, 50% neg, stratified).
  - Ты/Вы: проверить формат, загрузить 288 примеров в нужный формат.
- DoD: Оба test файла готовы для OS verification.

<a id="п6-w1"></a>

**Поля-6: Vikacheck reward format + train + test**
- Вход: 1150 тест примеров Vikacheck-assistant.
- Задача: reward format → два чека (vikacheck_assistant + vikacheck_reasoning) → train >= 800 + test >= 350.
- DoD: Train >= 800 x2 rewarda, test >= 350 x2 reward.

---

<a id="w2"></a>

### W2 (Mar 24–28)

**Общая задача W2:** OS reward verification — запустить каждый из 5 OS rewards на каждом тест-домене, получить P/R/F1, таблица work/don't work.

**Формат verification:**
- Для каждого OS reward: скормить 100-200 примеров из test domain.
- Сравнить с ground truth labels.
- Посчитать P/R/F1, threshold sweep (0.3, 0.5, 0.7).
- Вердикт: WORK (F1 >= 0.75) / MARGINAL (F1 0.5-0.75) / DON'T WORK (F1 < 0.5).

<a id="п1-w2"></a>

**П-1: OS reward verification — MD test**
- Задача: Прогнать 5 OS rewards (Skywork-8B, Skywork-0.6B, ArmoRM-8B, InternLM2-7B, ToolRM-1.7B) на MD test 800 примерах.
- DoD: Таблица reward x P/R/F1 на MD test.

<a id="п2-w2"></a>

**П-2: OS reward verification — Русификатор + Смена**
- Задача: 5 OS rewards на Русификатор test 300 + Смена test 300.
- DoD: Таблица reward x P/R/F1 на Русификатор и Смена.

<a id="п3-w2"></a>

**П-3: OS reward verification — Safety subset**
- Задача: 5 OS rewards на Safety pilot 200 примерах.
- DoD: Таблица reward x P/R/F1 на Safety. Особое внимание: precision >= 99% для safety.

<a id="п4-w2"></a>

**П-4: OS reward verification — Arena subset**
- Задача: 5 OS rewards на Arena 200-примерном subset.
- DoD: Таблица reward x P/R/F1 на Arena.

<a id="п5-w2"></a>

**П-5: OS reward verification — Эквивалентность + Ты/Вы**
- Задача: 5 OS rewards на Эквивалентность 1.1k + Ты/Вы 288.
- DoD: Таблица reward x P/R/F1 по обоим.
- Дополнительно: Эквивалентность LoRA — начать подготовку (2.1k train ready).

<a id="п6-w2"></a>

**П-6: Итоговый reward verification report**
- Задача: Собрать все таблицы от П-1..П-5 + determ checks (от Димы) + B2C checkers (от Даши) в единый отчёт.
- DoD: **Reward matrix: каждый reward x каждый домен → P/R/F1 → WORK/MARGINAL/DON'T WORK.** Рекомендации для RL v0.

**Итоговая reward matrix (шаблон):**

| Reward | MD | Русификатор | Смена | Эквив. | Ты/Вы | Arena | Safety | Вердикт |
|--------|----|-----------|----|-----|-----|-------|--------|---------|
| Skywork-8B | ? | ? | ? | ? | ? | ? | ? | ? |
| Skywork-0.6B | ? | ? | ? | ? | ? | ? | ? | ? |
| ArmoRM-8B | ? | ? | ? | ? | ? | ? | ? | ? |
| InternLM2-7B | ? | ? | ? | ? | ? | ? | ? | ? |
| ToolRM-1.7B | ? | ? | ? | ? | ? | ? | ? | ? |

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-OSVER-MD | W2 | П-1 | 5 OS rewards x MD test → P/R/F1 |
| EXP-OSVER-RU | W2 | П-2 | 5 OS rewards x Русификатор/Смена test |
| EXP-OSVER-SAFETY | W2 | П-3 | 5 OS rewards x Safety subset |
| EXP-OSVER-ARENA | W2 | П-4 | 5 OS rewards x Arena subset |
| EXP-OSVER-EQUIV | W2 | П-5 | 5 OS rewards x Эквивалентность/Ты/Вы |
