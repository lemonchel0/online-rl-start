# (mmpavlov) GigaGym updates

05\.02

* Добавлена [дока](https://gitlab.ai.cloud.ru/gigallms/gigagym/-/blob/initial_gym/README.md?ref_type=heads) с примерами добавления Tools/Checkers/Envs (в новом API)
* Для нового API - переработана базовая структура схем:
  * [Пример верификатора](https://gitlab.ai.cloud.ru/gigallms/gigagym/-/tree/initial_gym/checkers/new_math_checker?ref_type=heads) (чекер математики) - [docker](http://cr.ai.cloud.ru/19d74875-55e7-4f05-92d7-5bff1b43b8a0/rl_tools/new-api-math-checker:v1)

    [Пример тула](https://gitlab.ai.cloud.ru/gigallms/gigagym/-/tree/initial_gym/tools/new_api_code_executor?ref_type=heads) (code executor) - [docker](http://cr.ai.cloud.ru/19d74875-55e7-4f05-92d7-5bff1b43b8a0/rl_tools/new-api-code-executor:v1)

    [Пример env worker](https://gitlab.ai.cloud.ru/gigallms/gigagym/-/tree/initial_gym/environments/env_worker?ref_type=heads) (для in_process сред) - [docker](http://cr.ai.cloud.ru/19d74875-55e7-4f05-92d7-5bff1b43b8a0/rl_tools/new-api-inproc-env-manager:v1) (для in-proc сред: под обслуживает сразу несколько разных сред/сессий)

    \
    Пример трейсов ответов для env worker

    \
  * ```javascript
    === Testing Environment Worker ===
    Base URL: http://localhost:8000
    
    1. Health check:
    {
      "status": "ok",
      "details": null
    }
    
    2. Version:
    {
      "name": "env-worker",
      "version": "1.0.0",
      "commit": "7c779db",
      "build_date": "2026-02-04T13:24:58Z",
      "schema_version": "1.0"
    }
    
    3. List environments:
    {
      "environments": [
        "example"
      ]
    }
    
    4. Schema (tool definitions):
    {
      "schemas": [
        {
          "type": "function",
          "function": {
            "name": "increment",
            "description": "Increase the counter",
            "parameters": {
              "type": "object",
              "properties": {
                "amount": {
                  "type": "integer",
                  "description": "Amount to increment by",
                  "default": 1
                }
              },
              "required": []
            }
          }
        },
        {
          "type": "function",
          "function": {
            "name": "decrement",
            "description": "Decrease the counter",
            "parameters": {
              "type": "object",
              "properties": {
                "amount": {
                  "type": "integer",
                  "description": "Amount to decrement by",
                  "default": 1
                }
              },
              "required": []
            }
          }
        },
        {
          "type": "function",
          "function": {
            "name": "reset",
            "description": "Reset counter to 0",
            "parameters": {
              "type": "object",
              "properties": {},
              "required": []
            }
          }
        }
      ]
    }
    
    5. Create session (example env):
    {
      "status": "ok",
      "result": {
        "session_id": "ef0426c2-4ef6-422c-aa17-09eb03927ea5",
        "created_at": "2026-02-05T11:47:57.738276+00:00",
        "tool_schemas": [
          {
            "type": "function",
            "function": {
              "name": "increment",
              "description": "Increase the counter",
              "parameters": {
                "type": "object",
                "properties": {
                  "amount": {
                    "type": "integer",
                    "description": "Amount to increment by",
                    "default": 1
                  }
                },
                "required": []
              }
            }
          },
          {
            "type": "function",
            "function": {
              "name": "decrement",
              "description": "Decrease the counter",
              "parameters": {
                "type": "object",
                "properties": {
                  "amount": {
                    "type": "integer",
                    "description": "Amount to decrement by",
                    "default": 1
                  }
                },
                "required": []
              }
            }
          },
          {
            "type": "function",
            "function": {
              "name": "reset",
              "description": "Reset counter to 0",
              "parameters": {
                "type": "object",
                "properties": {},
                "required": []
              }
            }
          }
        ],
        "initial_observation": {
          "counter": 0,
          "target": 5,
          "step": 0,
          "remaining_steps": 20
        }
      },
      "meta": {
        "request_id": "d031d70b-7011-4b83-9e93-f92778b4a9d2",
        "session_id": "ef0426c2-4ef6-422c-aa17-09eb03927ea5",
        "runtime_ms": 1,
        "version": "env-worker@1.0.0",
        "image_digest": null
      }
    }
    Session ID: ef0426c2-4ef6-422c-aa17-09eb03927ea5
    
    6. Step (increment):
    {
      "status": "ok",
      "result": {
        "observation": {
          "counter": 2,
          "target": 5,
          "step": 1,
          "remaining_steps": 19
        },
        "events": [
          {
            "action_index": 0,
            "stdout_ref": null,
            "stderr_ref": null,
            "exit_code": 0
          }
        ],
        "reward": 0.1,
        "terminated": false,
        "truncated": false,
        "state_hash": null
      },
      "meta": {
        "request_id": "7ab3f412-5a44-4439-b876-e84fecbcdfde",
        "session_id": "ef0426c2-4ef6-422c-aa17-09eb03927ea5",
        "runtime_ms": 1,
        "version": "env-worker@1.0.0",
        "image_digest": null
      }
    }
    
    7. Step (increment again):
    {
      "status": "ok",
      "result": {
        "observation": {
          "counter": 5,
          "target": 5,
          "step": 2,
          "remaining_steps": 18
        },
        "events": [
          {
            "action_index": 0,
            "stdout_ref": null,
            "stderr_ref": null,
            "exit_code": 0
          }
        ],
        "reward": 1.0,
        "terminated": true,
        "truncated": false,
        "state_hash": null
      },
      "meta": {
        "request_id": "cd4cf094-b8b0-46cf-a2af-0321eb029036",
        "session_id": "ef0426c2-4ef6-422c-aa17-09eb03927ea5",
        "runtime_ms": 1,
        "version": "env-worker@1.0.0",
        "image_digest": null
      }
    }
    
    8. Verify (check if target reached):
    {
      "status": "ok",
      "result": {
        "reward": 1.0,
        "details": {
          "judge": "ok",
          "explanation": "Counter: 5, Target: 5",
          "accuracy": 1.0,
          "finish_reason": "stop"
        },
        "artifacts": null
      },
      "meta": {
        "request_id": "3c58b1b8-7366-45c0-848f-90a99bd1c03a",
        "session_id": "ef0426c2-4ef6-422c-aa17-09eb03927ea5",
        "runtime_ms": 0,
        "version": "env-worker@1.0.0",
        "image_digest": null
      }
    }
    
    9. Close session:
    {
      "status": "ok",
      "result": {
        "closed": true,
        "message": null
      },
      "meta": {
        "request_id": "87914092-080c-4afa-b46f-be7f883da2c1",
        "session_id": "ef0426c2-4ef6-422c-aa17-09eb03927ea5",
        "runtime_ms": 1,
        "version": "env-worker@1.0.0",
        "image_digest": null
      }
    }
    
    10. Metrics:
    # HELP requests_total Total number of requests
    # TYPE requests_total counter
    requests_total{service="env-worker"} 31
    
    # HELP errors_total Total number of errors
    # TYPE errors_total counter
    errors_total{service="env-worker"} 0
    
    # HELP latency_avg_ms Average latency in milliseconds
    # TYPE latency_avg_ms gauge
    latency_avg_ms{service="env-worker"} 1.69
    
    === Tests completed ===
    
    ```

    \

- [ ] Донести в доку код Максима для SWE
- [ ] Добить versioning
- [ ] Вынести существующие контейнеры для verl в легаси


\