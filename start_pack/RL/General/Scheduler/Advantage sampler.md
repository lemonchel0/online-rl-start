# Advantage sampler

### Обозначения

* Pool size: $N$
* $i \in \{1, \dots, N\}$
* $s_i = \text{sample\_id}(\text{pool}[i])$
* $S[s]$ — статистики для соответствующего `sample_id`


---

### Warmup

*(update count < warmup updates)*

$$
w_i = \frac{1}{N}
$$


---

### После warmup

* $r_i = S[s_i]$, иначе $-\infty$

Softmax-распределение:

$$
p_i = \frac{\exp(r_i - \max_j r_j)}{\sum_k \exp(r_k - \max_j r_j)}
$$

Если все $r_i = -\infty$, используется равномерное распределение.

Равномерное распределение:

$$
u_i = \frac{1}{N}
$$

Итоговое распределение:

$$
w_i = (1 - \eta) p_i + \eta u_i 
$$


---

### Обновление статистик

Для каждого `sample_id = s`:

* старое значение: $S_{\text{old}}[s]$;
* агрегированная метрика батча: $m_s$;

EMA-обновление:

$$
S_{\text{new}}[s] = (1 - \alpha) S_{\text{old}}[s] + \alpha m_s
$$

При использовании **abs advantage mean**:

$$
m_s = \operatorname{mean}\left(\, |adv_j| \;:\; j \in \text{rows with } sample\_id = s \right)
$$


---