# Mixed sampler

## Mixed-семплер (advantage + difficulty)

### Обозначения

* Pool size: $N$
* $u_i = \frac{1}{N}$

Softmax:

$$
\operatorname{softmax}(x)_i =
\frac{\exp(x_i - \max_j x_j)}{\sum_k \exp(x_k - \max_j x_j)}
$$


---

### Хранимые статистики

* $S^{adv}[s]$ — EMA advantage
* $\mu[s]$ — EMA success rate
* $d[s] = 1 - \mu[s]$ — difficulty


---

### Advantage-ветка

Метрика строки батча:

$$
a_j =
\frac{\sum_t |A_{j,t}| M_{j,t}}{\sum_t M_{j,t}}
$$

Агрегация:

$$
m^{adv}_t(s) = \frac{1}{|J_t(s)|} \sum_{j \in J_t(s)} a_j
$$

EMA:

$$
S^{adv}_t(s) =
(1 - \alpha_{adv}) S^{adv}_{t-1}(s)
+ \alpha_{adv} m^{adv}_t(s)
$$


---

### Difficulty-ветка

$$
y_j = \mathbb{I}\{ r_j > \tau \}
$$

$$
\tilde{y}_t(s) = \frac{1}{|J_t(s)|} \sum_{j \in J_t(s)} y_j
$$

$$
\mu_t(s) = (1 - \lambda)\mu_{t-1}(s) + \lambda \tilde{y}_t(s)
$$

$$
d_t(s) = 1 - \mu_t(s)
$$


---

### Итоговое распределение

$$
p^{adv}_i = \operatorname{softmax}(S^{adv}(s_i))
$$

$$
p^{diff}_i = \operatorname{softmax}(\gamma d(s_i))
$$

$$
w_i =
\lambda_{adv} p^{adv}_i
+ \lambda_{diff} p^{diff}_i
+ \lambda_{uni} u_i
$$


---

### Гиперпараметры

* `warmup_updates`
* $\alpha_{adv}$ — EMA для advantage
* $\lambda$ — EMA для success rate
* $\gamma$ — температура difficulty softmax
* $\lambda_{adv}, \lambda_{diff}, \lambda_{uni} \ge 0$

$$
\lambda_{adv} + \lambda_{diff} + \lambda_{uni} = 1
$$


### Пример конфига

```javascript
seed: 42
use_adaptive_batch_fractions: true

buckets:
  math:
    batch_fraction: 1.0
    data_sources: [
      "math-deepscaler",
      "math-numinamath1.5_olympiads",
      "math-numinamath1.5_aops_forum",
      "math-numinamath1.5_cn_contest",
      "math-still3",
      "math-numinamath1.5_inequalities",
      "math-numinamath1.5_olympiads_ref",
      "math-omnimath",
      "math-numinamath1.5_number_theory"
    ]

    retriever:
      strategy: all_data

    sampler:
      strategy: mixed

      seed: 42   
      warmup_updates: 100
      
      # EMA coefficients
      adv_ema_alpha: 0.15
      success_ema_alpha: 0.1
      
      # advantage strategy
      prioritize_by: "abs advantage mean" # or "score std"
      
      # difficulty strategy
      difficulty_gamma: 1.2
      difficulty_mode: "hard_first" # "hard_first" | "easy_first"
      
      # mixture coefficients
      mix_adv: 0.5
      mix_diff: 0.3
      mix_uni: 0.2
```