# Вова — Functions: Tools + Agents + SWE

**Состав:** 2 человека (В-1, В-2)
**Scope:** Function calling, agents, SWE-bench, ToolRM.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | BFCL v4 + ToolRM-1.7B (В-1), tool specs 50+ (В-2) | BFCL score; ToolRM online; 50 specs | BFCL + ToolRM latency | - [ ] |
| [W2](#w2) | ToolRM BFCL + error analysis (В-1), SWE-bench 10b (В-2) | ToolRM distribution; SWE mean@1 | **Top-5 error types** | - [ ] |
| [W3](#w3) | Tool reward v1: AST + exec (В-1), SWE-smith setup (В-2) | Tool reward P/R; SWE-smith running | Tool reward v1 metrics | - [ ] |
| [W4](#w4) | EXP-TOOL-EFF-1 (В-1), agent synthesis start (В-2) | Call distribution + redundancy | EXP-TOOL-EFF-1 report | - [ ] |
| [W5](#w5) | OTC-PO EXP-TOOL-EFF-2/3 (В-1), trajectories 100+ (В-2) | Efficiency results; 100+ trajectories | **-X% calls, accuracy =** | - [ ] |
| [W6](#w6) | Loop penalty EXP-LOOP-2/3 (В-1), SWE-smith 100+ (В-2) | Optimal penalty | Hard vs soft results | - [ ] |
| [W7](#w7) | BFCL gate (В-1), agent gate: SWE + BrowseComp (В-2) | BFCL >= target; suppression <= 5% | **Tools/Agent gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final tools/agent sign-off | All GREEN | Sign-off | - [ ] |

**Ключевые риски:**
- SWE-bench infra (Вадим) → проверить W1 Day 1
- ToolRM-1.7B недостаточно → fallback AST-based

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17-21)

<a id="в1-w1"></a>

**Вова-1: BFCL v4 + ToolRM-1.7B**
- DoD: BFCL overall + multi-turn; ToolRM online + latency.

<a id="в2-w1"></a>

**Вова-2: Tool specs 50+ + SWE baseline**
- DoD: 50 specs JSON; SWE mean@1 с 10b.

---

<a id="w2"></a>

### W2 (Mar 24-28)

<a id="в1-w2"></a>

**Вова-1:** ToolRM на BFCL + error analysis → top-5 error types.

<a id="в2-w2"></a>

**Вова-2:** SWE-bench 10b → mean@1.

---

<a id="w3"></a>

### W3-W8

<a id="в1-w3"></a>
<a id="в2-w3"></a>

**W3:** Tool reward v1 AST+exec (В-1), SWE-smith setup (В-2).

<a id="в1-w4"></a>
<a id="в2-w4"></a>

**W4:** EXP-TOOL-EFF-1 (В-1), agent synthesis (В-2).

<a id="в1-w5"></a>
<a id="в2-w5"></a>

**W5:** OTC-PO EXP-TOOL-EFF-2/3 (В-1), agent trajectories 100+ (В-2).

<a id="в1-w6"></a>
<a id="в2-w6"></a>

**W6:** Loop penalty EXP-LOOP-2/3 (В-1), SWE-smith 100+ (В-2).
**W7:** BFCL gate (В-1), agent gate SWE+BrowseComp (В-2).
**W8:** Final sign-off.

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-TOOL-EFF-1 | W4 | В-1 | Call distribution, redundancy |
| EXP-TOOL-EFF-2 | W5 | В-1 | OTC-PO efficiency |
| EXP-TOOL-EFF-3 | W5 | В-1 | ToolRLA multiplicative |
| EXP-LOOP-2 | W6 | В-1 | Hard penalty (-1.0) |
| EXP-LOOP-3 | W6 | В-1 | Soft penalty (graduated) |
