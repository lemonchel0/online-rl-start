# Поля — Product: B2C reward + Safety

**Состав:** 6 человек (П-1, П-2, П-3, П-4, П-5, П-6)
**Scope:** B2C reward (MD, Русификатор, Эквивалентность, Vikacheck, Ты/Вы, Смена языков, Arena), Safety reward, Classifier.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Кто | Задача | DoD | Status |
|---|-----|--------|-----|--------|
| **[W1](#w1)** | П-1 | MD-check тест >= 800 → ОБМ | Файл в ОБМ, distribution | - [ ] |
| | П-2 | Смена языков: трейн + тест 500+ | Train >= 1.5k, test >= 500 | - [ ] |
| | П-3 | Safety: инструкция + пилот 200, IAA | IAA > 0.8 | - [ ] |
| | П-4 | Arena 28k → разметку, тест 2k | Разметка запущена | - [ ] |
| | П-5 | Русификатор → reward >= 5k | 5k в формате | - [ ] |
| | П-6 | Vikacheck-assistant → reward | Train >= 800, test >= 350 | - [ ] |
| **[W2](#w2)** | П-1 | MD train → reward prep >= 3k | >= 3k reward train MD | - [ ] |
| | П-2 | Русификатор тест → 500+ | Test >= 500 | - [ ] |
| | П-3 | Safety → 1000+ | >= 1000 с labels | - [ ] |
| | П-4 | Arena 30%+ | 30% done | - [ ] |
| | П-5 | **Эквивалентность LoRA** | **P/R/F1 vs OS** | - [ ] |
| | П-6 | Vikacheck-reasoning SFT + тест | >= 500 | - [ ] |
| **[W3](#w3)** | П-1 | **MD LoRA** training + тест | **acc ~0.94** | - [ ] |
| | П-2 | Русификатор LoRA training | LoRA metrics | - [ ] |
| | П-3 | Safety train 1000+ + test 400+ | Train >= 1000, test >= 400 | - [ ] |
| | П-4 | Arena 60%+, classifier start | 60% done; classifier v0 | - [ ] |
| | П-5 | Ты/Вы: train + тест 500+ | Train >= 2k, test >= 500 | - [ ] |
| | П-6 | Classifier reward routing | Classifier v0 | - [ ] |
| **[W4](#w4)** | П-1 | Safety LoRA | **P >= 99%** | - [ ] |
| | П-2 | Arena тест 2k | Test стратифицирован | - [ ] |
| | П-3 | Смена LoRA | LoRA metrics | - [ ] |
| | П-4 | Ты/Вы LoRA | LoRA metrics | - [ ] |
| | П-5 | **All LoRA validation (6a)** | **LoRA x test → P/R/F1** | - [ ] |
| | П-6 | Routing classifier v1 | Accuracy на тесте | - [ ] |
| **[W5](#w5)** | П-1 | Arena LoRA | Arena LoRA metrics | - [ ] |
| | П-2 | Safety P >= 99% | **Confirmed** | - [ ] |
| | П-3 | All tests >= 500 | Checklist | - [ ] |
| | П-4 | Routing deployed | В pipeline | - [ ] |
| | П-5 | B2C integration test | Smoke test pass | - [ ] |
| | П-6 | NLI factuality | EXP-FACT-2 | - [ ] |
| **[W6](#w6)** | П-1 | Fix regressions | Fixed | - [ ] |
| | П-2 | Expand worst rewards | Improved | - [ ] |
| | П-3 | **Safety per-domain** 7 | **Per-domain table** | - [ ] |
| | П-4 | Human eval prep | Started | - [ ] |
| | П-5 | Reward ensemble validation | own vs own+OS delta | - [ ] |
| | П-6 | Reward hacking monitoring | Correlation per 20 steps | - [ ] |
| **[W7](#w7)** | П-1 | Final reward audit | Updated P/R/F1 | - [ ] |
| | П-2 | Arena swap canary | Canary result | - [ ] |
| | П-3 | Safety swap canary | Canary result | - [ ] |
| | П-4 | Human eval results | Score | - [ ] |
| | П-5 | Factuality gate | SimpleQA >= threshold | - [ ] |
| | П-6 | B2C metrics assembly | All numbers | - [ ] |
| **[W8](#w8)** | П-1..3 | **Safety gate final** | **PASS/FAIL** | - [ ] |
| | П-4..6 | Factuality + human eval | All GREEN | - [ ] |

**Ключевые риски:**
- Safety с нуля (0 данных) → ЦУР fallback + InternLM2 OS
- Arena 28k медленная → fast batch 5k
- Русификатор reasoning некачественная → кросс-валидация 2 моделей (протокол 3c)

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17-21)

<a id="п1-w1"></a>

**Поля-1: MD-check тест → ОБМ**
- Вход: ~4k размечено, ~15-20k прогонов.
- Задача: Выгрузить → фильтрация → стратификация → тест >= 800 (50/50) → ОБМ.
- DoD: >= 800 в ОБМ + distribution table.

<a id="п2-w1"></a>

**Поля-2: Смена языков трейн + тест 500+**
- Вход: Thinking 1086, Answer 804, тест ~300.
- Задача: Забрать answer → объединить → формат reward → +200 тест.
- DoD: Train >= 1.5k, test >= 500.

<a id="п3-w1"></a>

**Поля-3: Safety инструкция + пилот**
- Вход: НИЧЕГО (P0 gap).
- Задача: Инструкция (бинарная, 7 доменов) → seed 200 → пилот, перекрытие 5.
- DoD: Инструкция утверждена, IAA > 0.8.

<a id="п4-w1"></a>

**Поля-4: Arena 28k → разметка**
- Вход: 28k в train_catalog (MR 317).
- Задача: Pair-wise → стратификация → разметка, перекрытие 3 → тест 2k.
- DoD: Разметка запущена, 2k тест отложен, ETA.

<a id="п5-w1"></a>

**Поля-5: Русификатор → reward формат**
- Вход: ~7.6k в разметке, LoRA train 3239.
- Задача: reward формат → reasoning (протокол 3c) → >= 5k reward train.
- DoD: >= 5k, correlation >= 0.9.

<a id="п6-w1"></a>

**Поля-6: Vikacheck-assistant → reward**
- Вход: 1150 тест.
- Задача: Формат reward → 2 реварда → 800 train + 350 test.
- DoD: Train >= 800 x2, test >= 350 x2.

---

<a id="w2"></a>

### W2 (Mar 24-28)

<a id="п1-w2"></a>
**П-1:** MD train → reward training prep >= 3k.
<a id="п2-w2"></a>
**П-2:** Русификатор тест → 500+.
<a id="п3-w2"></a>
**П-3:** Safety → 1000+ (если IAA ok).
<a id="п4-w2"></a>
**П-4:** Arena 30%+.
<a id="п5-w2"></a>
**П-5:** **Эквивалентность reward LoRA** (2.1k train ready!) → P/R/F1.
<a id="п6-w2"></a>
**П-6:** Vikacheck-reasoning SFT + тест markup.

---

<a id="w3"></a>

### W3 (Mar 31 - Apr 4)

<a id="п1-w3"></a>
**П-1:** **MD-check LoRA** training + тест → acc ~0.94.
<a id="п2-w3"></a>
**П-2:** Русификатор LoRA training.
<a id="п3-w3"></a>
**П-3:** Safety train 1000+ + test 400+.
<a id="п4-w3"></a>
**П-4:** Arena 60%+, routing classifier start.
<a id="п5-w3"></a>
**П-5:** Ты/Вы train + test 500+.
<a id="п6-w3"></a>
**П-6:** Classifier reward routing (с командой Ивана).

---

<a id="w4"></a>
<a id="п1-w4"></a>
<a id="п2-w4"></a>
<a id="п3-w4"></a>
<a id="п4-w4"></a>
<a id="п5-w4"></a>
<a id="п6-w4"></a>

### W4 (Apr 7-11)

**П-1:** Safety LoRA (P >= 99%). **П-2:** Arena тест 2k. **П-3:** Смена LoRA. **П-4:** Ты/Вы LoRA. **П-5:** **All LoRA validation** (таблица). **П-6:** Classifier v1.

---

<a id="w5"></a>
<a id="п1-w5"></a>
<a id="п2-w5"></a>
<a id="п3-w5"></a>
<a id="п4-w5"></a>
<a id="п5-w5"></a>
<a id="п6-w5"></a>

### W5 (Apr 14-18)

**П-1:** Arena LoRA. **П-2:** Safety P >= 99%. **П-3:** All tests >= 500. **П-4:** Routing deployed. **П-5:** B2C integration test. **П-6:** NLI factuality EXP-FACT-2.

---

<a id="w6"></a>
<a id="п1-w6"></a>
<a id="п2-w6"></a>
<a id="п3-w6"></a>
<a id="п4-w6"></a>
<a id="п5-w6"></a>
<a id="п6-w6"></a>

### W6 (Apr 21-25)

**П-1:** Fix regressions. **П-2:** Expand worst rewards. **П-3:** **Safety per-domain** (7 domains). **П-4:** Human eval prep. **П-5:** Reward ensemble validation. **П-6:** Reward hacking monitoring EXP-HACK-1.

---

<a id="w7"></a>

### W7 (Apr 28 - May 2)

Final reward audit (П-1), Arena swap canary (П-2), Safety swap canary (П-3), Human eval (П-4), Factuality (П-5), B2C metrics (П-6).

<a id="w8"></a>

### W8 (May 5-9)

**Safety gate final** (П-1..3). Factuality + human eval + sign-off (П-4..6).

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-HACK-1 | W5-6 | П-6 | Correlation tracking per 20 steps |
| EXP-FACT-2 | W5-6 | П-6 | NLI factuality reward in RL |
