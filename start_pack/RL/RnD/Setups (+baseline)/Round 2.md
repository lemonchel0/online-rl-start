# Round 2

### Пробежавшие эксперименты:


```javascript
=====EXP_1=====
project_name: hyperparameters_tuning
exp_name: 10b_toolmind_prod_setup_batch_128_minibatch_64
sh script: 10b.toolmind.ProdSetup.b128.mb64.sh
nnodes: 4

model: 10b_toolmind
loss_mode: cispo_prod
batchsize = 128
minibatch size = 64
max_response_len = 8k -> 16k -> 24k -> 32k
lr = 1e-6
lr warmup = 25
mask = False
n_resp_per_prompt=16
grad_clip = 1.0
clip_low = 0.2
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
exp_name: CISPO-10b_toolmind_prod_setup_batch_128_minibatch_64_context_16k
sh script: 10b.toolmind.ProdSetup.b128.mb64.c16k.sh
nnodes: 4

model: 10b_toolmind
loss_mode: cispo_prod
batchsize = ?
minibatch size = ?
max_response_len = 16k -> 32k
lr = 1e-6
lr warmup = 25
mask = False
n_resp_per_prompt=16
grad_clip = 1.0
clip_low = 0.2
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
=====EXP_3=====
project_name: hyperparameters_tuning
exp_name: CISPO_MASKED-10b_toolmind_prod_setup_batch_128_minibatch_64_mask_05_context_8k
sh script: 10b.toolmind.ProdSetup.b128.mb64.c8k.m05.sh
nnodes: 4

model: 10b_toolmind
loss_mode: cispo_masked
batchsize = 128
minibatch size = 64
max_response_len = 8k
lr = 1e-6
lr warmup = 25
mask = 0.5
n_resp_per_prompt=16
grad_clip = 1.0
clip_low = 0.2
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
=====EXP_4=====
project_name: hyperparameters_tuning
exp_name: CISPO_MASKED-10b_toolmind_prod_setup_batch_128_minibatch_64_mask10_context_8k
sh script: 10b.toolmind.ProdSetup.b128.mb64.c8k.m10.sh
nnodes: 4

model: 10b_toolmind
loss_mode: cispo_masked
batchsize = 128
minibatch size = 64
max_response_len = 8k
lr = 1e-6
lr warmup = 25
mask = 1.0
n_resp_per_prompt=16
grad_clip = 1.0
clip_low = 0.2
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
=====EXP_5=====
project_name: hyperparameters_tuning
exp_name: CISPO_MASKED-10b_toolmind_prod_setup_batch_128_minibatch_64_mask15_context_8k
sh script: 10b.toolmind.ProdSetup.b128.mb64.c8k.m10.sh
nnodes: 4

model: 10b_toolmind
loss_mode: cispo_masked
batchsize = 128
minibatch size = 64
max_response_len = 8k
lr = 1e-6
lr warmup = 25
mask = 1.5
n_resp_per_prompt=16
grad_clip = 1.0
clip_low = 0.2
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
=====LR_SCHEDULER_EXPS=====
```

```javascript
=====EXP_7,8,9+=====
default math setup + разные гиперпараметры семплера
как только будет лучший - запуск в прод сетапе
```


### Результаты по разным max resp len:


```javascript
max resp len = 8k vs 16k
```


 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/a1e0fa7d-9801-4439-89b4-bb4ed1ef919c/image.png " =1207x618")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/cd099094-d08a-409c-bdb4-56b14dc9fb23/image.png " =1213x620")


В целом нет особо различий во времени развала 16к контекста со старта прожило чуть-чуть дольше, но не ощутимо. 


 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/822d400f-ba15-4c22-bc73-d3d3c650e261/image.png " =1213x625")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/eeec375c-8fb0-40ca-9701-522449a93681/image.png " =1219x623")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/b4b0f50c-9e08-4e0d-97ca-382e61dc422d/image.png " =1219x621")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/de230d78-38b6-4533-b95e-cf6ce455d37a/image.png " =1215x620")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/946ebeb8-4865-465e-bfde-7744402e18eb/image.png " =803x303")

16к со старта ощутимо лучше разве что для aime, в остальном сильной разницы нет. 

Лучший aime - 0.374:

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/b799857d-ea13-4c78-8565-7a6942a83c88/image.png " =610x57")


\
### Стейдж контекста:


Оба сетапа затем перезапускались с увеличенным контекстом перед развалом:

8k → 16k → 24k → 32k

16k → 32k

Идея - понять, что лучше - плавно повышать контекст или же резко. 


```javascript
8k:    0-320
16k: 250-340
24k: 300-390
32k: 340-390
```


 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/0703594a-4a7a-4a31-8bdf-0fb87804261e/image.png " =1217x625")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/494a3ee7-6921-4103-b8ab-8d096b41eec0/image.png " =1217x621")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/b02d55f1-af58-44e1-942d-94c314236c25/image.png " =1218x622")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/2df02549-0197-4d37-9b78-6ab914aa3128/image.png " =1214x622")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/7bc2384a-4ce4-4a1a-ba7d-3f44463b0508/image.png " =1219x624")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/6e4406f5-1484-4ae6-a86b-48467e0c1734/image.png " =802x305")


В целом более менее все метрики чуть подросли в такой стратегии, достаточно неплохо вырос код и aime:

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/f2a13a90-1332-4013-9998-d9b6ec572696/image.png " =407x67")

math_500 остался примерно на том же уровне, как будто бы как и биология, химия, …, structured output.

В такой стратегии повышения контекста эксп продержался примерно до 360-370 степов, то есть модель прожила чуть дольше, чем базовые 8k. 


```javascript
16k: 0- ~350
32k: 300-380
```


 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/b6fad5ad-4bf6-41df-a972-d1dbc4ea21e7/image.png " =1216x619")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/8622a7fd-e257-4309-84a2-573d710f7d14/image.png " =1217x621")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/ca1f2338-ce3c-472e-8ed7-fec9bb941810/image.png " =1214x617")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/e534afe1-5118-43c1-b9df-9fd53cfc2e5f/image.png " =1214x620")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/72f6b836-cace-4163-b645-e2537b420ee2/image.png " =1214x624")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/fa224cbc-7b32-4dda-a07f-d137d3a770e2/image.png " =802x309")

Суть примерно такая же, однако тут даже особо обучение продлить не получилось эксп развалился достаточно быстро. 

aime:

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/357ed6d9-c330-40b2-83e6-d124b983ba65/image.png " =435x49")

Остальные метрики особо вырасти не успели при повышении контекста до 32k. 


#### Вывод:


В целом итоговое качество как будто бы ничем не отличается - метрики растут примерно до тех же значениях в обоих стратегиях. 

То же самое можно сказать и о стабильности - предел до 400 степа. 

Единственный плюс у первой стратегии - начинать с 8 и затем повышать плавно - только в том, что обучение идёт быстрее, что логично - меньше max len. 


\
### Маски:


Успел запустить даже 3 сетапа с разными значениями масок: 0.5, 1.0, 1.5. 

В сравнении с идентичным сетапом, но без масок:

Бейзлайн - жёлтый пунктир.  


 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/98aefa47-b342-4216-ae68-8a4b5a4b8c99/image.png " =1647x828")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/0cc680a6-6d9b-4c76-bea1-edc67206875e/image.png " =1223x619")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/87a8ec02-81b1-4f9d-9528-bb1944bd948b/image.png " =1223x624")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/96ff551d-7a8e-4189-97dd-066643c826eb/image.png " =1219x620")


Видно, что все маски развалились раньше бейзлайна без них. 


#### Вывод:

Мне кажется пока что маски лучше не использовать, потому что прироста в стабильности они не дали. Нужно более детально с ними поэкспериментировать, чтобы понять - нужно ли их вообще использовать. 


### lr шедулеры:


Основное сравнение - косинусный шедулер из верла и resp len lr шедулер. 

Косинусный - ? запускал не я

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/2969d015-f246-43ef-a292-752c07a4b915/image.png " =1214x624")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/d89eb328-b495-460b-97df-ae492ac727d8/image.png " =1218x619")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/63d2d567-c72b-42de-8ebb-13ffab763830/image.png " =1217x614")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/90b88f08-7207-42e5-a5dc-a8751b435790/image.png " =1216x618")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/d845b0d8-566f-4134-8251-2ccc8a324ce4/image.png " =1216x620")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/663f7129-96f1-4630-a5ed-c1d01d6259ab/image.png " =802x301")


Начало обучения совсем шумное, потому что эксперимент в сетапе 128/16 батчи, чтобы можно было честно сравнить с косинусный шедулером (он запускался ещё до выбора батчей). 

Но явно видно, что обучение не разваливается и живёт сильно дольше (что логично ведь lr понижается). 

В сравнении с сетапом 128/16 без шедулера:


 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/288c3308-26f3-44f5-8c01-7f561afabbd9/image.png " =1215x618")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/662b04e6-cbc0-441d-bc45-20742d51c84f/image.png " =1213x622")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/64a38477-00d0-4a2d-94f6-6d3fe3d924f2/image.png " =1219x620")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/d01736cc-ce7c-4655-9c3d-015eb9bdeafe/image.png " =1214x619")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/e445927e-6eba-42bd-90d2-fe77eda3db40/image.png " =1214x623")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/48ee2e6e-55e3-4a60-861c-4fc0c33df3f5/image.png " =810x304")


При этом даже при снижении lr метрики понемногу растут. 


#### Вывод:


Для стабильности шедулер точно нужно использовать, особенно учитывая агрессивный 1е-6. 

Но есть несколько моментов:

* Нужно аккуратно подобрать шаг снижения lr, для этого я бы поставил 2-3 эксперимента с немного разными степами снижения. 
* Очевидно обучение в какой-то момент просто выйдет на плато - будет стабильным, но метрики перестанут расти, это видно в этом эксперименте (тут косинусный шедулер с быстрым снижением, но суть та же):

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/b31e3149-81d0-49fb-9735-36f9f3e90380/image.png " =1215x622")

 ![](uploads/6938f512-38d7-408b-959d-95e5567f3b28/ea864806-1db7-4e88-8d1e-bbed737fd07b/image.png " =2440x436")

Поэтому начиная с момента, когда обучение выходит на плато, я бы попробовал добавить advantage семплер - он будет давать семплы с высокими по модулю адвантажами и сдвигать с места обучение, что поможет вытянуть более высокие метрики (но нужно аккуратно подобрать гиперпараметры самого семплера). 

* Я бы также всё равно уже с нужными батчами запустил бы оба вида шедулеров, чтобы точно понять, какой из них всё-таки ведёт себя эффективнее. 


\

### Краткий итог:


1\. Сильной разницы в начальном выборе контекста не видно, при этом его увеличение даёт прирост метрик. 

2\. Пока что маски не прибавляют стабильности, поэтому в прод сетапе их использовать на данный момент не стоит. 

3\. lr шедулер ощутимо добавляет стабильности, но нужно аккуратно подобрать lr decay и в будущем поэкспериментировать с тем, чтобы модель сдвигалась с плато (иначе это по сути бесполезные степы).