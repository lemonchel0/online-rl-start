# Серёжа — Code: reasoning + reward

**Состав:** 2 человека (С-1, С-2)
**Scope:** Code reasoning, code reward, LCB, SWE.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | LCB v6 + sandbox (С-1), code queries 10k+ (С-2) | LCB pass@1; sandbox works; 10k queries | **LCB vs Qwen3.5-2B/4B** | - [ ] |
| [W2](#w2) | Code reward pipeline (С-1), queries labels (С-2) | Pipeline P/R; 10k+ labelled | Code reward v0 metrics | - [ ] |
| [W3](#w3) | Code reward v1 exec+judge (С-1), difficulty filter (С-2) | Improved P/R; difficulty labels | Code RL set 10k+ | - [ ] |
| [W4](#w4) | Code RL set final (С-1), canary 20-30 steps (С-2) | Set ready; canary metrics | **First code RL canary** | - [ ] |
| [W5](#w5) | Code RL 50+ (С-1), VeRPO test (С-2) | 50 step metrics | Code RL: LCB trend | - [ ] |
| [W6](#w6) | Code RL 100+ (С-1), code+math joint (С-2) | 100+ metrics; no regression | **LCB at 100 steps** | - [ ] |
| [W7](#w7) | LCB gate (С-1), SWE gate (С-2) | LCB >= target; no regression | **Code gate: PASS/FAIL** | - [ ] |
| [W8](#w8) | Final code sign-off | All GREEN | Sign-off | - [ ] |

**Ключевые риски:**
- Sandbox security → gVisor/Firecracker
- Code reward FP → execution-first, judge-fallback

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17-21)

<a id="с1-w1"></a>

**Серёжа-1: LCB v6 + sandbox**
- DoD: Sandbox works; LCB pass@1; **comparison table vs Qwen3.5-2B/4B.**

<a id="с2-w1"></a>

**Серёжа-2: Code queries 10k+**
- DoD: 10k+ с domain + difficulty labels, в хранилище.

---

<a id="w2"></a>

### W2 (Mar 24-28)

<a id="с1-w2"></a>

**Серёжа-1:** Code reward pipeline: sandbox + LLM-judge → P/R.

<a id="с2-w2"></a>

**Серёжа-2:** Queries с domain labels → 10k+.

---

<a id="w3"></a>
<a id="с1-w3"></a>
<a id="с2-w3"></a>

### W3-W8

**W3:** Code reward v1 exec+judge (С-1), difficulty filtering P(mixed) (С-2).
**W4:** Code RL set final (С-1), canary 20-30 steps (С-2).
**W5:** Code RL 50+ (С-1), VeRPO test (С-2).
**W6:** Code RL 100+ (С-1), code+math joint (С-2).
**W7:** LCB gate (С-1), SWE gate + regression (С-2).
**W8:** Final sign-off.
