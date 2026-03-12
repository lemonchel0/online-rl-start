# (v_min) benches + traces

* Парсинг:
* * BFCL под капотом использует tree sitter для Парсинга — надо писать свой \`class QwenFCHandler(OSSHandler): **—** тут закордкоженый regex в **def _extract_tool_calls(input_string):**  + доп функции для корректной обработки
  * tau^2 — тоже свой какой-то хэндлер: Implement the LocalAgent class in src/tau2/agent/base.py + Register your agent in src/tau2/agent/registry.py

  \
* Стоимость прогона бенча:
* * BFCLv4 — можем взять свой поиск — тогда не надо платить за API + нужно поднимать БД
  * tau^2 — Artificial Analysis для прогона использует Qwen3 235b, можем поступить так же и не платить за API для GPT5.1


\
 ![](uploads/41fef399-e949-4a0a-8386-1ea8ef24f9e7/1df178d5-43ee-4d7b-804b-cfa1cf19d7bb/Screenshot%202026-02-06%20at%2019.06.59.png " =1293x644")


\
После анализа генераций на бейзлайне с code exec выяснилось, что мы практически никогда не вызываем тулы на LCB + на LCB гораздо чаще наблюдаются зацикливания:

 ![](uploads/41fef399-e949-4a0a-8386-1ea8ef24f9e7/b862f04f-fa96-4207-a0b1-f5701f5101c8/Screenshot%202026-02-10%20at%2017.09.25.png " =1002x588")


Вот так выглядит график количества использования code exec на коде в первые шаги трейна:

 ![](uploads/41fef399-e949-4a0a-8386-1ea8ef24f9e7/e84e1320-04c6-4b03-b210-2a2297292125/Screenshot%202026-02-10%20at%2022.24.37.png " =1004x607")


Также оказалось, что в SFT данных в принципе нет примеров использования code exec для решения задач с кодом