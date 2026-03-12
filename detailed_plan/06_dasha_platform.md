# Даша — Platform: B2C функции + reward

**Состав:** 4 человека (Д-1, Д-2, Д-3, Д-4)
**Scope:** Code interpreter, websearch, text2image, B2C function rewards.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Interpreter (Д-1), Search (Д-2), T2I (Д-3), B2C report (Д-4) | P/R/F1 per checker; report | **3 checker P/R + report** | - [ ] |
| [W2](#w2) | EXP-B2C-1 (Д-1+2), reward data 500+ per func (Д-3+4) | Baseline report; >= 500/func | EXP-B2C-1 | - [ ] |
| [W3](#w3) | Interpreter v1 (Д-1), Search v1 (Д-2), T2I reward (Д-3), RL data (Д-4) | EXP-B2C-2/3; t2i P/R | **Interpreter reward delta** | - [ ] |
| [W4](#w4) | Max calls (Д-1), Cross-tool (Д-2), NLI (Д-3), Integration (Д-4) | EXP-B2C-4/5; NLI accuracy | Tool productivity | - [ ] |
| [W5](#w5) | NLI fact-check (Д-1), Search grounding (Д-2), B2C canary (Д-3+4) | EXP-FACT-3; canary metrics | **B2C canary** | - [ ] |
| [W6](#w6) | B2C RL 100+ (Д-1+2), URL track + max calls (Д-3+4) | 100+ metrics | B2C RL at 100 | - [ ] |
| [W7](#w7) | B2C gate: interp, search, t2i (Д-1+2), latency (Д-3+4) | Rates >= target; latency ok | **B2C gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final B2C sign-off | All GREEN | Sign-off | - [ ] |

**Ключевые риски:**
- Websearch NLI дорогой → keyword matching v0, NLI offline
- Fabricated URLs → hard penalty -1.0

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17-21)

<a id="д1-w1"></a>

**Даша-1: Interpreter reward checker**
- DoD: execution_success + answer_match + penalties. P/R на 100.

<a id="д2-w1"></a>

**Даша-2: Websearch reward checker**
- 4-component: query_relevance, citation_accuracy, answer_grounding, format. Fabricated URL → -1.0.
- DoD: Per-component P/R на 100. Timeout 3s.

<a id="д3-w1"></a>

**Даша-3: Text2image trigger checker**
- DoD: Trigger P/R на 100 (50 pos + 50 neg).

<a id="д4-w1"></a>

**Даша-4: B2C baseline report**
- DoD: Call distribution, loop rate, format compliance, redundancy.

---

<a id="w2"></a>

### W2 (Mar 24-28)

<a id="д1-w2"></a>
<a id="д2-w2"></a>

**Д-1+2:** EXP-B2C-1 full baseline report.

<a id="д3-w2"></a>
<a id="д4-w2"></a>

**Д-3+4:** B2C reward train data >= 500 per function.

---

<a id="w3"></a>
<a id="д1-w3"></a>
<a id="д2-w3"></a>

### W3-W8

**W3:** EXP-B2C-2 interpreter (Д-1), EXP-B2C-3 search (Д-2), T2I reward (Д-3), RL data (Д-4).
<a id="д1-w4"></a>
<a id="д2-w4"></a>
**W4:** EXP-B2C-4 max calls (Д-1), EXP-B2C-5 cross-tool (Д-2), NLI (Д-3), Integration (Д-4).
<a id="д1-w5"></a>
<a id="д2-w5"></a>
**W5:** NLI (Д-1), EXP-FACT-3 search grounding (Д-2), B2C canary (Д-3+4).
**W6:** B2C RL 100+ (Д-1+2), URL tracking + max calls (Д-3+4).
**W7:** B2C gates (Д-1+2), latency (Д-3+4).
**W8:** Final sign-off.

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-B2C-1 | W2 | Д-1+2 | Baseline: call dist, loop, format |
| EXP-B2C-2 | W3 | Д-1 | Interpreter reward v1 |
| EXP-B2C-3 | W3 | Д-2 | Websearch decomposition v1 |
| EXP-B2C-4 | W4 | Д-1 | Max tool calls limit |
| EXP-B2C-5 | W4 | Д-2 | Cross-tool: interp + search |
| EXP-FACT-3 | W5 | Д-2 | Search-grounded eval offline |
