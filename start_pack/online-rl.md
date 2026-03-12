# Online RL

## Какие датасеты делаем, зачем

Все три датасета должны быть **строго непересекающимися** (без дубликатов и утечек между ними) и **сбалансированно распределены** по выбранным осям стратификации — домену, дисциплине, источнику или иным релевантным категориям.\n- only user queries (много)\n- Reward train (2-4к)\n- Reward test (1-2к)

### Online RL: only user queries (Много)

**Содержимое:** только пользовательские сообщения + контекст диалога (без ассистентских ответов) для онлайн-роллаутов.

**Формат:** собираем в формате API, пример: 

```json
{
  "id": "sample_0001",
  "dataset_id": "online_rl_v1",
  "messages": [
    {
      "role": "system",
      "content": ""
    },
    {
      "role": "user",
      "content": ""
    }
  ]
}
```

**Хранение:** централизованное место загрузки/хранения ==(ответственный: **Дима**, должен придумать куда грузить).==

### Reward train (2-4к)

**Содержимое**: стандартный чат train catalog (без megas или пуфик какой)


:::info
Формат assistant зафиксированный для ревардов (строка с тэгами): 

Пример итогового вида строки:

<critic>Хороший ответ</critic>$\boxed{True}$

True - ошибка есть, False - ошибки нет

```javascript
Point-wise -> "<critic>reasoning</critic>$\\boxed{True/False}$"
Pair-wise -> "<critic>reasoning</critic>$\\boxed{A>>B|A>B|A==B|A<B|A<<B}$"
N-rank -> "<critic>reasoning</critic>$\\boxed{1|2|3|4|5}$"
```

:::

**Хранение**: Train Catalog, от ветки `main`

### Reward test (1-2к)

**Формат:** идентичен train, чтобы метрики были сопоставимы.\n**Хранение:** в ОБМ


## Вспомогательные функции для сбора user-запросов и не только

Необходимо подготовить функции для сбора чатов под конкретные реварды. \nГде хранить и куда заливать: @[Дмитрий Меньших (Прораб)](mention://1172ba63-ba07-4add-b097-28343c39dfe5/user/b5545d1f-e6ee-4120-8a60-cd26d60090a3) @[Сергей Графов (Машинист)](mention://61cfbe9a-400e-417a-8cca-72c425f39ab9/user/357da567-1034-498d-b25a-50e09a93f5c9) \n

## Обучение Lora для Reward

### путь через фронт


0. Как делаются дампы?  @[Сергей Графов (Машинист)](mention://cef6c0cb-392a-4335-a366-09f66eeb1907/user/357da567-1034-498d-b25a-50e09a93f5c9) 

Заходим на <http://d-gcproddata-applrdata-adv-msk01:8080/docs>

Выбираем ручку `/v1/tasks` и нажимаем `Try it out`

Заполняем json_file  Если есть функции их тоже передаем

 ![](attachments/1d69d27d-a794-4ee1-9bf1-f77c2e5f16c8.png " =597x308")

Дальше во всех полях кроме(model, adapter_id, lora_name) стираем дефолтное знаеение и убирвем галочку(должно быть как training_file на скрине) 

 ![](attachments/e0ec6106-fc3e-4e52-acf2-bcc74538c9be.png " =581x276")

Нажимаем `Execute` запоминаем id адаптера из ответа

### путь через курл

```bash
curl -X 'POST' \
  'http://d-gcproddata-applrdata-adv-msk01:8080/v1/tasks?bucket_name=d-gccore-finetuning' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'model=GigaChat-Lite-2:28-2' \
  -F 'adapter_id=qwe' \
  -F 'json_file=@data.json;type=application/json' \
  -F 'lora_name=qwe'
```

как узнать доступные модели? отправить случайные буквы в модель, сервер вернет доступные для обучения модели. 

Там сейчас нужны разные виды дампов для разных линеек моделей, линейка гигачат2 учится на старом темплейте, гигачат3 на новом

ОБМ есть только на гигачат3!