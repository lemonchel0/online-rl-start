# dr.grpo like

```javascript
=====EXP_1=====
project_name: hyperparameters_tuning
exp_name: 10b_toolmind_prod_setup_batch_128_minibatch_64_adv_normalize_False
sh script: 10b.toolmind.ProdSetup.b128.mb64.adv_normalize.False.sh
nnodes: 4

model: 10b_toolmind
loss_mode: cispo_prod
batchsize = 128
minibatch size = 64
max_response_len = ??
lr = 1e-6
lr warmup = 25
mask = False
n_resp_per_prompt=16
grad_clip = 1.0
clip_low = ??
clip_high = ??
entropy_coeff = 0
use kl = False
overlong = False
sampler: random
weight_decay = 0.1
adv_normalize = False (посмотреть как это правильно задавать в верле)
mixture:                   
  code: 0.285
  math: 0.285
  structured_output: 0.01
  mcqa: 0.09
  nemotron: 0.33
```


```javascript
=====EXP_1=====
project_name: hyperparameters_tuning
exp_name: 10b_toolmind_prod_setup_batch_128_minibatch_64_adv_normalize_False_dr_grpo_loss_mode
sh script: 10b.toolmind.ProdSetup.b128.mb64.dr_grpo_loss_mode.sh
nnodes: 4

model: 10b_toolmind
loss_mode: cispo_prod
batchsize = 128
minibatch size = 64
max_response_len = ??
lr = 1e-6
lr warmup = 25
mask = False
n_resp_per_prompt=16
grad_clip = 1.0
clip_low = ??
clip_high = ??
entropy_coeff = 0
use kl = False
overlong = False
sampler: random
weight_decay = 0.1
adv_normalize = False (посмотреть как это правильно задавать в верле)
loss_agg_mode = dr_grpo (посмотреть как это правилно задавать в верле)
mixture:                   
  code: 0.285
  math: 0.285
  structured_output: 0.01
  mcqa: 0.09
  nemotron: 0.33
```