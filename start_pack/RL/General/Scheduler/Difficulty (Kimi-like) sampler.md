# Difficulty (Kimi-like) sampler

## Success-rate / Kimi-семплер

### Обозначения

* Pool size: $N$
* $i \in \{1, \dots, N\}$
* $s_i = \text{sample\_id}(\text{pool}[i])$


---

### Warmup

$$
w_i = \frac{1}{N}
$$


---

### После warmup

Индикатор успеха:

$$
y_j = \mathbb{I}\{ r_j > 0 \}
$$

Агрегация по `sample_id`:

$$
\tilde{y}_t(s) = \frac{1}{|J_t(s)|} \sum_{j \in J_t(s)} y_j
$$

EMA успеха:

$$
\mu_t(s) = (1 - \lambda)\mu_{t-1}(s) + \lambda \tilde{y}_t(s)
$$

Difficulty:

$$
d_t(s) = 1 - \mu_t(s)
$$


---

### Вероятности семплирования

$$
p_i = \frac{\exp(\gamma \, g(d_t(s_i)))}{\sum_k \exp(\gamma \, g(d_t(s_k)))}
$$

где:

* $\gamma$ — температура ( $\gamma > 0$ → hard-first ),
* $g(d) = +d$ для hard-first, $-d$ для easy-first.

Опциональная смесь с равномерным:

$$
w_i = (1 - \eta) p_i + \eta u_i
$$


---


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
      strategy: difficulty
      seed: 42

      warmup_updates: 100
      success_ema_alpha: 0.3

      gamma: 1.2
      mode: hard_first
      uniform_mix_eta: 0.4
```


* mode = hard_first / easy_first - приоритезация сложных или простых задач (меняется знак для d)
* success_ema_alpha - гиперпараметр сглаживания с предыдущими статистиками
* gamma - температура в softmax'е - чем она выше (для hard_first), тем более агрессивно семплируются сложные задачи:

Рассмотрим два семпла:

$$
d_i - d_j = 0.3
$$

Экспонента (или же ratio):

$$
\frac{p_i}{p_j} = \exp(\gamma (d_i - d_j))
$$

Если выбирать маленькую gamma (например, 0.5):

$$
\exp \approx 1.15
$$

Соответственно, чем выше gamma, тем больше приоретизируются сложные задачи:

$$
\gamma = 4 => \exp \approx 3.3 - \text{очень агрессивный hard-first}
$$

То есть gamma - это как бы контраст между сложными и простыми задачами. 

Чтобы ограничивать агрессивность нужен exploration в uniform. 

Таким образом, варьируя все гиперпараметры можно получить разные стратегии семплирования.