# Clip-range

```javascript
=====EXP_1=====
project_name: hyperparameters_tuning
exp_name: 10b_toolmind_prod_setup_batch_128_minibatch_64_clip_low_0.0
sh script: 10b.toolmind.ProdSetup.b128.mb64.clip_low_0.0.sh
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
clip_low = 1.0
clip_high = 0.28
entropy_coeff = 0
use kl = False
overlong = False
sampler: random
weight_decay = 0.1
mixture:                   
  code: 0.285
  math: 0.285
  structured_output: 0.01
  mcqa: 0.09
  nemotron: 0.33
```


```javascript
=====EXP_2=====
project_name: hyperparameters_tuning
exp_name: 10b_toolmind_prod_setup_batch_128_minibatch_64_clip_high_4.0
sh script: 10b.toolmind.ProdSetup.b128.mb64.clip_hig_4.0.sh
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
clip_low = 0.2
clip_high = 4.0
entropy_coeff = 0
use kl = False
overlong = False
sampler: random
weight_decay = 0.1
mixture:                   
  code: 0.285
  math: 0.285
  structured_output: 0.01
  mcqa: 0.09
  nemotron: 0.33
```


\
```javascript
=====EXP_3=====
project_name: hyperparameters_tuning
exp_name: 10b_toolmind_prod_setup_batch_128_minibatch_64_clip_low_0.0_clip_high_4.0
sh script: 10b.toolmind.ProdSetup.b128.mb64.clip_low_0.0.clip_hig_4.0.sh
nnodes: 4

model: 10b_toolmind
loss_mode: cispo_prod
batchsize = 128
minibatch size = 64
max_response_len = 8k -> 16k
lr = 1e-6
lr warmup = 25
mask = False
n_resp_per_prompt=16
grad_clip = 1.0
clip_low = 1.0
clip_high = 4.0
entropy_coeff = 0
use kl = False
overlong = False
sampler: random
weight_decay = 0.1
mixture:                   
  code: 0.285
  math: 0.285
  structured_output: 0.01
  mcqa: 0.09
  nemotron: 0.33
```