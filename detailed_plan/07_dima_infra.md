# Дима — Infra

**Состав:** 3 человека (Дм-1, Дм-2, Дм-3)
**Scope:** Хранение данных, OS reward serving, LoRA training pipeline, детерминированные checks, reward aggregation, monitoring.
[← Обзорный план](00_overview.md)

---

## Чек-лист руководителя

| W | Задача команды | DoD | Результат недели | Status |
|---|---------------|-----|-----------------|--------|
| [W1](#w1) | Storage (Дм-1), OS rewards x5 + **verify** (Дм-2), Determ checks (Дм-3) | Storage works; >= 3 OS online + verification; API P >= 95% | **Infra + OS verification** | - [ ] |
| [W2](#w2) | Ingest all (Дм-1), **OS matrix** (Дм-2), LoRA pipeline (Дм-3) | All in storage; 5x5+ matrix; pipeline works | **OS reward matrix** | - [ ] |
| [W3](#w3) | Determ→RL (Дм-1), LoRA training (Дм-2), rw_config auto (Дм-3) | Determ in RL; LoRAs training | LoRA launched | - [ ] |
| [W4](#w4) | **Aggregation** (Дм-1), Multi-rw pipe (Дм-2), Canary infra (Дм-3) | Aggregation works; canary ready | **EXP-AGG results** | - [ ] |
| [W5](#w5) | Prod pipeline (Дм-1), Monitoring (Дм-2), ODIN (Дм-3) | Prod-grade; dashboard live | **Monitoring live** | - [ ] |
| [W6](#w6) | PAR (Дм-1), Gradient reg (Дм-2), Scaling 200+ (Дм-3) | Hack mitigation results | RL 200+ stable | - [ ] |
| [W7](#w7) | Prod deploy (Дм-1), Eval sweep (Дм-2), **Competitor bench** (Дм-3) | Serving ready; **competitor table** | **Us vs competitors** | - [ ] |
| [W8](#w8) | Canary 48h (Дм-1), Monitoring (Дм-2), Rollback docs (Дм-3) | Canary stable | **Canary: PASS/FAIL** | - [ ] |

**Ключевые риски:**
- GPU capacity для 5 OS rewards → приоритет: Skywork-0.6B + ArmoRM-8B + InternLM2-7B
- LoRA pipeline нестабилен → тест 100 записей перед production

---

## Детальный план по людям

<a id="w1"></a>

### W1 (Mar 17-21)

<a id="дм1-w1"></a>

**Дима-1: Хранилище данных + pipeline ингеста**
- Вход: Данные разбросаны по `/home/valenchik/`, train_catalog, notebooks.
- Задача: (1) S3/MinIO. (2) Бакеты: rollout-queries, reward-train, reward-test, stage15, sft. (3) CLI. (4) Per-sample reward_config. (5) Тест 100 записей.
- DoD: Endpoint, round-trip 100, README.
- Day 1: выбор. Day 2-3: CLI + формат. Day 4: данные. Day 5: README + тест.

<a id="дм2-w1"></a>

**Дима-2: OS reward модели (5 штук) + verification**
- Вход: Skywork-8B, Skywork-0.6B, ArmoRM-8B, InternLM2-7B, ToolRM-1.7B.
- Задача: Deploy (vLLM) → API → latency. **Verification: 100 labeled примеров → P/R/F1 → work/don't work.**
- DoD: 5 online. Table: model|latency|GPU|P/R/F1. Golden test.
- Риск: GPU → приоритет 3 must-have.

<a id="дм3-w1"></a>

**Дима-3: Детерминированные чеки v1**
- Задача: FastAPI: loop_detected, repetition_ratio, json_valid, latex_valid. Тест 50.
- DoD: API works, P >= 95%, latency < 50ms.

---

<a id="w2"></a>

### W2 (Mar 24-28)

<a id="дм1-w2"></a>
**Дм-1:** Залить ВСЕ datasets → inventory table.
<a id="дм2-w2"></a>
**Дм-2:** 5 OS rewards на всех тестах → **matrix P/R/F1 (25+ ячеек)**.
<a id="дм3-w2"></a>
**Дм-3:** LoRA training pipeline → end-to-end test.

---

<a id="w3"></a>
<a id="дм1-w3"></a>
<a id="дм2-w3"></a>
<a id="дм3-w3"></a>

### W3-W8

**W3:** Determ→RL pipeline (Дм-1), MD+Русификатор LoRA training (Дм-2), reward_config auto-assign (Дм-3).
<a id="дм1-w4"></a>
<a id="дм2-w4"></a>
<a id="дм3-w4"></a>
**W4:** Hierarchical aggregation safety+GDPO (Дм-1), multi-reward pipeline (Дм-2), canary infra (Дм-3).
<a id="дм1-w5"></a>
<a id="дм2-w5"></a>
<a id="дм3-w5"></a>
**W5:** Prod reward pipeline (Дм-1), monitoring dashboard (Дм-2), ODIN EXP-HACK-2 (Дм-3).
<a id="дм1-w6"></a>
<a id="дм2-w6"></a>
<a id="дм3-w6"></a>
**W6:** PAR EXP-HACK-3 (Дм-1), gradient reg EXP-HACK-4 (Дм-2), scaling 200+ (Дм-3).
**W7:** Prod deploy (Дм-1), eval sweep automation (Дм-2), **competitor bench Qwen3.5-2B/4B + GLM-5 + DS** (Дм-3).
**W8:** Canary 48h (Дм-1), monitoring (Дм-2), rollback docs (Дм-3).

---

## Эксперименты команды

| ID | Неделя | Owner | Описание |
|----|--------|-------|----------|
| EXP-RWVER-1 | W1 | Дм-2 + МЛ-6+7 | 5 rewards x 100 examples → P/R/F1 |
| EXP-AGG-1 | W4 | МЛ-4 | Weighted sum baseline |
| EXP-AGG-2 | W4 | МЛ-5 | GDPO decoupled |
| EXP-AGG-5 | W4 | Дм-1 | Hierarchical (safety+GDPO) |
| EXP-HACK-2 | W5 | Дм-3 | ODIN length debiasing |
| EXP-HACK-3 | W6 | Дм-1 | PAR reward shaping |
| EXP-HACK-4 | W6 | Дм-2 | Gradient reg vs KL |
