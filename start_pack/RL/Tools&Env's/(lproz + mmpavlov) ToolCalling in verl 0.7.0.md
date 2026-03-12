# (lproz + mmpavlov) ToolCalling in verl 0.7.0

* в ветке feature/math_with_tool_usage_070 дотащили изменения с 050 (Лиля) 
  * основные изменения по лосс-маскам для токенов
  * парсер тулколов
  * тулы серча/код экзекьютора
* переработаны конфиги запуска (Миша)
* сделаны работы с environments (много тулов в рамках одного тула) - пока тестится (Миша)

# Unified Agent Loop

## Ключевые отличия от оригинального `tool_agent_loop.py`

### 1. Единый реестр Tools + Environments

**Было:** Инструменты создавались из общего конфига для всех сэмплов.

**Стало:** `UnifiedRegistry` хранит и tools, и environments на уровне класса. Для каждого сэмпла через `tools_kwargs` выбирается нужный набор:

```python

tools_kwargs = {
    "code_interpreter": {},           # активировать tool
    "workplace_assistant": {          # активировать environment с параметрами
        "task": "...",
        "ground_truth": [...]
    }
}
```

### 2. Environments (сессионные окружения)

Новая сущность с жизненным циклом:

```
create_session → get_tool_schemas → execute_tool* → verify_session → close_session
```

* Схемы инструментов могут приходить из конфига, из ответа create_session, или запрашиваться отдельно
* `**/verify**` **вызывается автоматически** сразу после завершения роллаута (перед `close_session`) — не нужно вызывать вручную
* В verify передаётся история диалога и `ground_truth` из `tools_kwargs` для валидации результатов

### 3. Handlers (обработчики запросов/ответов)

Единый интерфейс `ToolEnvHandler` для кастомизации:

* Формирование запросов (`build_execute_request`, `build_verify_session_request`)
* Парсинг ответов (`parse_execute_response`, `parse_verify_session_response`)
* Определение успешности (`is_successful_response`)

Пример: `@register_handler("workplace_assistant")`

### 4. Per-tool/env Rate Limiting

Каждый tool/environment имеет свой независимый `TokenBucketWorker` для rate limiting.

### 5. Расширенные метрики

**По источникам данных:**

* `tool_usage/ds/{data_source}/...`
* `tool_usage/domain/{data_domain}/...`

**По типам ошибок API:**

* `connection_errors` — сервер недоступен
* `timeout_errors` — таймаут
* `http_errors` — HTTP 4xx/5xx

**Верификация:**

* `tool_usage/verification/pass_rate`
* `tool_usage/verification/env/{env}/pass_rate`

## Структура файлов

```
sber_src/dev/agent_loop/
├── unified_tool_agent_loop.py   # Основной loop
├── unified_registry.py          # Реестр tools/envs
├── remote_tool.py               # RemoteTool
├── base_environment.py          # BaseEnvironment (ABC)
├── remote_environment.py        # RemoteEnvironment (HTTP)
└── utils/
    ├── handlers.py              # ToolEnvHandler + реализации
    └── metrics.py               # Агрегация метрик
```

## Конфигурация

См. `sber_src/configs/tools/experimental_tools_config.yaml`


# SWE

# SWE-bench: заметки по интеграции и перформансу

## Что было сделано

### Наша сторона (verl)

* Написан `SweBenchHandler` (`sber_src/dev/agent_loop/handlers/swe_bench.py`) — обработчик запросов/ответов для SWE-bench API (create session, step, verify, close)
* Добавлена конфигурация окружения `swe_bench` в `tool_api_config.yaml` с эндпоинтами `/v1/sessions/*`
* Системный промпт `SWE-bench_Verified` в `tool_system_config.yaml` — инструкция для модели по работе с bash в `/testbed`
* Реорганизация `agent_loop` — handlers/tools/environments/registry разнесены в отдельные подпакеты

### Серверная сторона (Вадим)

Реализован SWE-bench сервис с поддержкой batch API.

**Реальные результаты перформанса сервера (500 сессий):**

| Mode | CREATE Time | CREATE Rate | DELETE Rate |
|----|----|----|----|
| Individual async | 45.4s | 11.0/sec | 52.6/sec |
| Batch API | 31.2s | 16.0/sec | 233.9/sec |


---

## Эксперименты с concurrency

Пробовали крутить параметры конкурентности на нашей стороне:

* Увеличивали `rate_limit` и `num_workers` в конфиге окружения
* Уменьшали `timeout`

**Результат: существенных приростов нет.**

### Профиль времени (средние на один роллаут)

```
agent_loop/generate_sequences/mean:  148.4 sec

agent_loop/tool_calls/mean:           25.2 sec
```

Это суммарные времена на все хопы (assistant turns) одного роллаута. Включают время ожидания в очереди на старт запроса, т.е. реальное wall-clock время от сабмита таски до получения ответа.

На валидацию SWE-Bench Verifieed 500 семплов с `n=1` уходит \~10 мин на полную джобу (9b + MAX_ASSISTANT_TURNS=20 + 20k токенов)

### Вывод

\~85% времени роллаута — это генерация модели, \~15% — тул-коллы. Узкое место — inference, а не сервис окружений. Крутить `rate_limit`/`num_workers` на стороне мастера бесполезно — мы упираемся в генерацию.

**Следующий шаг:** асинхронная валидация (Вадим)(verify не блокирует роллаут-луп), чтобы не ждать verify в конце каждого батча.


### Результаты 9b

mean@1 = 1/500 (T=0)

mean@1 = 4/500 (T=0)


Решает:

* простые задачи на фикс одной строчки 
* делает pip install какой-то версии библиотек (непонятно, насколько это корректно)