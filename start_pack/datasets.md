# Datasets

:::info
Требования для каждой чекалки: <https://docs.google.com/document/d/1kJzTMs4FnnajThcpYJ0jkCh4wer6Zkwtl9saCDf9F7g/edit?tab=t.0> 

**База**: \nчекалка на разломанный боксед:

https://git.sberdevices.ru/chatwm/gigachat-data-analitics-projects/revizor/-/tree/master/revizor/checks/text/weired_boxed


соответсвие боксед в системе и в ответе:

https://git.sberdevices.ru/chatwm/gigachat-data-analitics-projects/revizor/-/tree/master/revizor/checks/text/boxed

:::


## Reward

### Markdown


:::info
Тут лежит описание чекалки <https://chatwm.ru/418cd9e668906755bbe18e0a16fc3e45/doc/md-txeD1VYtpo>

:::

### План


1. Прогоны Дани (спросить как он находил проблемные случаи) поставить в редактуру (BEAUTIFUL ANSWER) не весь прогон только то где сломался latex (чекалкой на латех, просто изменение текста: убрать все символы md и сверить до и после)
2. Отдаем @[Владимир Кулешов](mention://b6bf5cd1-d941-41af-81d7-38c2ef84bfe1/user/c4e88fc1-b627-433f-a250-505a007e10c7) сеты чтобы собрать в ревард(страница Reward + Lora формат Point-wise -> "<critic>reasoning</critic>$\\\\boxed{True/False}$") (сеты для лор переписываем в формат, сеты для реварда оставляем как есть)

   
   1. Есть 15к синты (4к из них размечено и обучение и тест)
3. можно попробовать взять данные из трейна лоры (+ попросить Вику найти ризонинг: были просто метка и переписанный ответ, Вика добавила ризонинг)
4. Прогнать чекалку на синте посмотерть корреляцию, далее подумаем



| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| Online RL: only user queries |    |    |    |
| Reward train | \~4000<br> | Готово: [51057587-1af3-4eaf-b0aa-94321d113dd5](https://tagme.sberdevices.ru/company/1bf17a92-158c-4ff6-8e16-83a51386ab09/project/eeb471c8-dfed-459f-bd97-d4babe2e86a3/task/51057587-1af3-4eaf-b0aa-94321d113dd5/overview)<br>Готово: [3e771069-e912-4487-8f33-353b715e9e79](https://tagme.sberdevices.ru/company/1bf17a92-158c-4ff6-8e16-83a51386ab09/project/eeb471c8-dfed-459f-bd97-d4babe2e86a3/task/3e771069-e912-4487-8f33-353b715e9e79/overview) ==+поля собирает из лоры== |    |
| Reward test |    |    |    |
| Прогоны | \~15-20k | `/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/MD/ datasets_for_marcup`простите за пробел - выгружаем правильные метки, чистим все что неверное (отдаем промпт инженерам переписать), отрезаем тестовый сет (800 штук) , остальное в трейн<br><br>трейн - в остров в отдельную ветку от мейна, тест - Нимя (заносим в ОБМ)на разметке было только: <br>`/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/MD/ datasets_for_marcup/generated_answer_md_redactor.tsv` |    |
| Систем и фью-шоты |    | `/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/MD/system_few_shots_messages.json` |    |

### Русификатор


:::info
Тут лежит описание чекалки и систем <https://chatwm.ru/418cd9e668906755bbe18e0a16fc3e45/doc/russifikator-DSz6OLtMnC> 

:::


1. Провести беседу с редакторами (@[Полина Кудрявцева](mention://cbd7aed8-a933-4525-a582-1dffbba7086e/user/5a22ce1c-7512-456e-9e07-f157b7cc3089))
2. Перепроверить метрики @[Виктория Аленчик](mention://325ef509-53fc-48b3-a3c7-2a368d0ca945/user/764de476-4dea-4385-b49c-d7b07f2ad9bd) 
3. дальше думаем

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| Online RL: only user queries | 13583 | `/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/Rusificator/datasets_for_marcup/splits/rollout_rusificator.jsonl` | Нужно отдать Диме, когда он придумает место для хранения. |
| Reward train | \~7600 | В разметке: `[3e23a02d-524f-4d46-9e81-0446804e4969](https://tagme.sberdevices.ru/company/1bf17a92-158c-4ff6-8e16-83a51386ab09/project/eeb471c8-dfed-459f-bd97-d4babe2e86a3/task/3e23a02d-524f-4d46-9e81-0446804e4969/overview)`<br>В разметке: `[a791a507-066b-41cc-bb58-b602f9a13b68](https://tagme.sberdevices.ru/company/1bf17a92-158c-4ff6-8e16-83a51386ab09/project/eeb471c8-dfed-459f-bd97-d4babe2e86a3/task/a791a507-066b-41cc-bb58-b602f9a13b68/overview)`В разметке: `[54f47e0e-de2e-48e8-b75c-e89fca58fb51](https://tagme.sberdevices.ru/company/1bf17a92-158c-4ff6-8e16-83a51386ab09/project/eeb471c8-dfed-459f-bd97-d4babe2e86a3/task/54f47e0e-de2e-48e8-b75c-e89fca58fb51/overview)`==+поля собирает из лоры== |    |
| Reward test | 300 + отрезать из трейна |    |    |
| Прогоны | \~20k | `/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/Rusificator/datasets_for_marcup` |    |
| System/User |    | `/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/Rusificator/rusificator_sys_us.md` |    |

### Эквивалентность фактического ответа голден ответу


:::info
Описание <https://chatwm.ru/418cd9e668906755bbe18e0a16fc3e45/doc/ekvivalentnost-otvet-pravilnomu-WvhJNMHbXM>

:::


1. @[Виктория Аленчик](mention://abd5ea12-eb5b-4433-8343-fa80eaea4d8a/user/764de476-4dea-4385-b49c-d7b07f2ad9bd) ставит тест в разметку **(done)**
2. Заливает трейн в остров **(done)**
3. Отдаем на заварку Диме **(done)**

| Датасеты | Кол-во | Путь |    |
|----|----|----|----|
| Online RL: only user queries | 13800 | Нужно отдать Диме как место найдёт`/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/Check_answer/splits/rollout_equivalence_75pct.jsonl` |    |
| Reward train | 2146 | **В разметке стоит**: `e087e1d7-e878-4159-ab96-4b88458355ab`  Залито в для Lora в train_catalog<br>`https://git.sberdevices.ru/chatwm/gigachat-data-analitics-projects/train_catalog/-/tree/add-reward-check-answer?ref_type=heads` |    |
| Reward test | 1139 | **В разметке стоит**: `19175fe8-82e7-4b11-8288-e1ea62ba93be`  Данные тут (на всякий): `/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/Check_answer/datasets_for_marcup/splits/test.tsv` |    |

### System following bench (Vikacheck - assistant)


1. Заварить в формате реварда
2. Эксп разделить на два реварда под каждую метку
3. Долить в обучение всю разметку (+ кусочек отрезать на тест сложные системы 200)
4. Ускорить Арсения криком

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| Online RL: only user queries |    |    |    |
| Reward train |    |    |    |
| Reward test | 1150 (2 система в файле, по индексам смотреть нужно) | `/home/valenchik/notebooks/prod_cases/prompt_engineers/my/data_train/Reward/vikacheck/vicacheck_test/test_vicacheck_reward.jsonl` |    |

### System following bench (Vikacheck — reasoning)


0. Прогнать SFT (семплы с сорсов)
1. \

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| Online RL: only user queries |    |    |    |
| Reward train |    |    |    |
| Reward test |    |    |    |

### Ты/Вы

[==Инструкция тут==](https://docs.google.com/document/d/1xBrFfmyFbN2eiizL58POIOZGxcKlBmASZ7Z09sLIb_Y/edit?tab=t.0#heading=h.duyesvajwpcg)

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| Online RL: only user queries |    |    |    |
| Reward train | 2073 | salute болталка / сбер кот, в разметке [e457c5c7-f203-4e37-bac2-18aaf063e4e0](https://tagme.sberdevices.ru/company/1bf17a92-158c-4ff6-8e16-83a51386ab09/project/eeb471c8-dfed-459f-bd97-d4babe2e86a3/task/e457c5c7-f203-4e37-bac2-18aaf063e4e0/overview) |    |
|    |    |    |
| Reward test | 288 | гига логи, гпт логи, персонажи |    |

### Безопасность

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| Online RL: only user queries |    |    |    |
| Reward train |    |    |    |
| Reward test |    |    |    |

### Смена языков

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| Online RL: only user queries |    |    |    |
| Reward train | 1086 | Thinking: Разметка завершена ~~в разметке~~ [~~87fb2e7f-ea4f-4817-8831-5c573a2e4d17~~](https://tagme.sberdevices.ru/company/1bf17a92-158c-4ff6-8e16-83a51386ab09/project/eeb471c8-dfed-459f-bd97-d4babe2e86a3/task/87fb2e7f-ea4f-4817-8831-5c573a2e4d17/overview) |    |
| 804 | Answer: в разметке [87ed66e9-ca7c-4bd8-ad77-0e3015b29840](https://tagme.sberdevices.ru/company/1bf17a92-158c-4ff6-8e16-83a51386ab09/project/eeb471c8-dfed-459f-bd97-d4babe2e86a3/task/87ed66e9-ca7c-4bd8-ad77-0e3015b29840/overview) |    |
| Reward test | \~300 | Answer, исправлять формат отдала Косте  |    |

### Арена

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| Online RL: only user queries |    |    |    |
| Reward train | 28000 | <https://git.sberdevices.ru/chatwm/gigachat-data-analitics-projects/train_catalog/-/merge_requests/317> | надо поставить в разметку |
| Reward test | 2000 |    | надо собрать с разметки |

## LoRa

### Markdown

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| LoRa train |    |    |    |
| LoRa test |    |    |    |

### Русификатор

==Комментарий: надо обновить систем==

| Датасеты | Кол-во | Путь | Статус |
|----|----|----|----|
| LoRa train | 3239 | `/data/pkudriavtseva/notebooks/reward/rusificator/lora_train_v2.json` | Залит в остров, заваривается ([тред](https://mm.sberdevices.ru/sberdevices/pl/sa6iizyeb3yqudprn3qet6rdrr)) |
| LoRa test | 300 (150 + 150) | `/data/pkudriavtseva/notebooks/reward/rusificator/lora_test_v2.json` | Не в ОБМ |


\