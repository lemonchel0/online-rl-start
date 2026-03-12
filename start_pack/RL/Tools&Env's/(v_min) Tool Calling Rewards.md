# (v_min) Tool Calling Rewards

**Гипотеза**

Дополнительные реварды за корректный вызов тулов улучшат качество обучения

**Сетап**

Бейзлайн: [http://wandb.su:8081/alignment/debug-runs/runs/5hhfea3d](http://wandb.su:8081/alignment/debug-runs/runs/5hhfea3d?nw=nwuserolyandrevn)

Данные: `rl_data_filtered_dapo_skywork/math_code_processed/train`

Ключевая особенность: следуя Kimi Researcher + TORL

> ```
> Format Reward: The model is penalized for trajectories 
> that include invalid tool calls or if the context/iteration 
> exceeds the maximum limitation. — Kimi Researcher
> ```


 ![](uploads/41fef399-e949-4a0a-8386-1ea8ef24f9e7/837d5bac-241a-4aef-bfa2-44b4151d0234/Screenshot%202026-01-29%20at%2013.57.19.png " =457x319")

 ![](uploads/41fef399-e949-4a0a-8386-1ea8ef24f9e7/d46e8b2b-4f79-40c4-8eba-efe3391f052d/Screenshot%202026-01-29%20at%2014.29.17.png " =455x312")

Весь ответ штрафуется:

* если код не запустился – на 0.2
* если код отработал с ошибкой – на 0.1

Для этого было добавлено пробрасывание наград через `ToolAgentLoop` с помощью `AgentLoopOutput` — с пересчетом `reward_tensor` внутри `GIGARewardManager` 

**Результаты**

TBA: пока слишком мало степов, ожидаем увидеть бОльший `code_success_pct`


\
**Baseline Generations Analysis:**


 ![](uploads/41fef399-e949-4a0a-8386-1ea8ef24f9e7/1b7eaf6d-e0ca-4727-8067-ba7deb6d7e08/Screenshot%202026-01-26%20at%2016.11.47.png " =494x431")


 ![](uploads/41fef399-e949-4a0a-8386-1ea8ef24f9e7/450c3a74-996a-4932-bf4d-e8e2fadf3b6a/Screenshot%202026-01-26%20at%2016.12.32.png " =993x616")

 ![](uploads/41fef399-e949-4a0a-8386-1ea8ef24f9e7/0362e68b-f711-429c-b6ba-c23e4cca7026/Screenshot%202026-01-26%20at%2016.19.37.png " =989x597")

**SFT thinking**

Проблемы:

Использование интерпретатора для проверки ответа, не для вычислений:

```javascript
            "function_call": {
                "name": "code_interpreter",
                "arguments": {
                    "code": "# Создаем ответ о жизненном цикле бабочки\nresponse = \"\"\"\nthe life cycle of a butterfly is a remarkable transformation process consisting of four distinct stages, each playing a crucial role in the insect's development. this complete metamorphosis is one of nature's most fascinating phenomena, allowing butterflies to adapt to different environmental conditions and maximize their survival chances.\n\n* the first stage begins when a female butterfly lays eggs on specific host plants that will serve as food for the emerging caterpillars, ensuring their survival through careful maternal selection.\n* the second stage is the larva or caterpillar phase, where the insect focuses entirely on eating and growing, shedding its skin multiple times while storing energy necessary for the upcoming transformation.\n* the third stage is the pupa or chrysalis, appearing dormant externally while dramatic internal changes occur, as the caterpillar's tissues break down and reorganize into adult butterfly structures.\n* the final stage is the adult butterfly or imago, which emerges from the chrysalis with wings that must be expanded and dried before it can fulfill its primary purposes of reproduction and pollination.\n\np.s. butterflies are not only beautiful creatures but also important pollinators that contribute significantly to ecosystem health and biodiversity.\n\nis there anything else i can help with?\n\"\"\"\n\n# Проверяем соответствие требованиям\nrequirements = {\n    \"all_lowercase\": response.islower(),\n    \"bullet_points\": response.count(\"* \"),\n    \"sentences\": len([s for s in response.split(\". \") if len(s) > 5]),\n    \"has_postscript\": \"p.s.\" in response.lower(),\n    \"ending_phrase\": response.strip().endswith(\"is there anything else i can help with?\")\n}\n\nprint(\"Проверка требований:\")\nfor req, value in requirements.items():\n    print(f\"{req}: {value}\")\n\nif requirements[\"bullet_points\"] != 4:\n    print(f\"ОШИБКА: Должно быть ровно 4 маркера, сейчас: {requirements['bullet_points']}\")\n\nif not requirements[\"all_lowercase\"]:\n    print(\"ОШИБКА: Весь текст должен быть в нижнем регистре\")\n    \nif requirements[\"sentences\"] < 4:\n    print(f\"ОШИБКА: Должно быть минимум 4 предложения, сейчас: {requirements['sentences']}\")\n    \nif not requirements[\"has_postscript\"]:\n    print(\"ОШИБКА: Отсутствует постскриптум (P.S.)\")\n    \nif not requirements[\"ending_phrase\"]:\n    print(\"ОШИБКА: Отсутствует завершающая фраза\")\n\n# Выводим готовый ответ, если все требования выполнены\nif all(requirements.values()):\n    print(\"\\nВсе требования выполнены!\")\n    print(\"\\nОтвет:\")\n    print(response.strip())\nelse:\n    print(\"\\nНе все требования выполнены. Исправьте ошибки.\")"
                }
            },
```

Иногда происходит shuffle ответов от тулов:

```javascript
"function_call": {
                "name": "code_interpreter",
                "arguments": {
                    "code": "import numpy as np\nfrom sympy import symbols, Eq, solve, simplify, expand\n\n# Вероятность выпадения ровно k орлов в n бросках монеты с вероятностью p\ndef binomial_probability(n, k, p):\n    from math import comb\n    return comb(n, k) * (p**k) * ((1-p)**(n-k))\n\n# Известные данные\nn = 5  # Количество бросков\nk = 3  # Нужно ровно 3 орла\nw = 144/625  # Заданная вероятность\n\n# Проверим, удовлетворяет ли p = 2/5 или p = 3/5 нашему условию\np_values = [2/5, 3/5]\nfor p in p_values:\n    probability = binomial_probability(n, k, p)\n    print(f\"При p = {p}: вероятность = {probability}, w = {w}\")\n    print(f\"Равны: {np.isclose(probability, w)}\")\n\n# Решим уравнение напрямую\nprint(\"\\nРешение уравнения 10*p^3*(1-p)^2 = 144/625:\")\n\n# Преобразуем уравнение\n# 10*p^3*(1-p)^2 = 144/625\n# 10*p^3*(1-2p+p^2) = 144/625\n# 10*(p^3-2p^4+p^5) = 144/625\n# 10p^5-20p^4+10p^3 = 144/625\n# 625*(10p^5-20p^4+10p^3) = 144\n# 6250p^5-12500p^4+6250p^3 = 144\n\np = symbols('p', real=True)  # Указываем, что p - действительное число\nequation = Eq(10*p**3*(1-p)**2, 144/625)\nequation_expanded = expand(equation.lhs - equation.rhs)\nprint(f\"Уравнение в раскрытой форме: {equation_expanded} = 0\")\n\n# Решаем уравнение\nsolutions = solve(equation_expanded, p)\nprint(\"\\nРешения:\")\n\nreal_solutions = []\nfor solution in solutions:\n    try:\n        solution_value = float(solution)\n        if 0 <= solution_value <= 1:  # p должно быть вероятностью, т.е. от 0 до 1\n            real_solutions.append(solution_value)\n            print(f\"p = {solution_value}\")\n            # Проверка\n            probability = binomial_probability(n, k, solution_value)\n            print(f\"Проверка: вероятность = {probability}, w = {w}\")\n            print(f\"Равны: {np.isclose(probability, w)}\")\n    except (TypeError, ValueError):\n        # Пропускаем комплексные корни\n        pass\n\n# Проверяем условия для ответов\nprint(\"\\nПроверка вариантов ответов:\")\nif any(np.isclose(sol, 2/5) for sol in real_solutions):\n    print(\"(A) p = 2/5 - верно\")\nelif any(np.isclose(sol, 3/5) for sol in real_solutions):\n    print(\"(B) p = 3/5 - верно\")\nelif all(sol > 3/5 for sol in real_solutions):\n    print(\"(C) p > 3/5 - верно\")\nelif len(real_solutions) > 1:\n    print(\"(D) p не определяется однозначно - верно\")\nelif len(real_solutions) == 0:\n    print(\"(E) Нет значения p, для которого w = 144/625 - верно\")\n\n# Посмотрим, какие значения p дают w = 144/625\nprint(\"\\nДополнительное исследование:\")\np_test = np.linspace(0, 1, 100)\nfor p_val in p_test:\n    prob = binomial_probability(n, k, p_val)\n    if np.isclose(prob, w, atol=1e-4):\n        print(f\"p ≈ {p_val:.4f} дает вероятность {prob:.6f}\")"
                }
            },
            "functions": null,
            "thinking_functions": null,
            "trainable": true,
            "index": 5
        },
        {
            "role": "function",
            "content": "{\"status\": \"success\", \"stdout\": \"Количество предложений: 16\\nКоличество разделов с markdown: 2\\nВстречаемость ключевых слов:\\n  *neural network*: 2\\n  *handwritten digits*: 5\\n  *training*: 2\\n  *MNIST dataset*: 3\\n  *activation function*: 1\\nНаличие постскриптума: True\\n\\nВсе требования выполнены: True\\n\\nОтвет:\\n\\n## Introduction to Neural Network Training for Digit Recognition\\n\\nTraining a *neural network* for recognizing *handwritten digits* typically starts with choosing the right dataset. The *MNIST dataset* is the gold standard for this task, containing 60,000 training images and 10,000 testing images of *handwritten digits* from 0-9. This dataset provides a balanced representation of different handwriting styles and is perfectly sized for experimentation.\\n\\nThe architecture of your neural network plays a crucial role in recognition accuracy. A common approach is to use a Convolutional Neural Network (CNN) that can automatically extract spatial features from images, though a simple feedforward network with 1-2 hidden layers can achieve decent results for beginners. When implementing the network, choosing the right *activation function* such as ReLU (Rectified Linear Unit) for hidden layers and Softmax for the output layer is essential for effective learning.\\n\\n## Optimization Techniques for Better Results\\n\\nThe *training* process requires careful tuning of hyperparameters. Start with a reasonable learning rate (0.01-0.001) and batch size (32-128), then experiment with different optimizers like Adam or SGD with momentum. Regularization techniques such as dropout (typically 0.2-0.5) can significantly prevent overfitting, especially when working with limited data.\\n\\nData augmentation is another powerful technique that artificially expands your training set by creating modified versions of existing images through rotations, shifts, and elastic deformations. This helps make your *neural network* more robust to variations in *handwritten digits* that may not be well-represented in your original dataset.\\n\\nMonitoring validation accuracy during *training* is critical - you should implement early stopping to prevent overfitting. For *handwritten digits*, a well-trained model should achieve at least 98% accuracy on the *MNIST dataset*, with state-of-the-art models reaching over 99.7% accuracy.\\n\\nP.S. Remember that while the techniques described above work well for the *MNIST dataset*, you may need to adapt your approach when working with your own *handwritten digits* data, which might have different characteristics.\", \"stderr\": \"\"}",
            "files": [],
            "trainable": false,
            "images": [],
            "index": 6
        },
```


**Future Work**

Следуя **SimpleTIR** → фильтрация траекторий, в которых turn не заканчивается вызовом тула или ответом. В нашем случае – будем отфильтровывать траектории, в которых нет ответа внутри `\boxed{}` или же произошло зацикливание – утверждается, что такое простое решение повышает стабильность обучения в Multi-Turn постановке

Следуя **LLDS** → хотя в нашем сетапе коллапса не происходит, в отличие от Multi-Turn постановки у авторов, предлагаемая регуляризация encourage использование тулов


\