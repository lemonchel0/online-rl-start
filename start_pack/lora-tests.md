# Lora tests

| Название лоры | Тестовый сет | Метрика | Значение метрики |
|----|----|----|----|
| Русификатор | RUSIFICATOR_FLAG_TEST *(пока не в main)* | Сверяем совпадение измененного текста с эталонным (accuracy) | **GigaChat-2-Pro (test_v3):**accuracy	0.4900000000precision	0.4651162791recall	0.1333333333f1	0.2072538860confusion_matrix	tp=20 fp=23 fn=130 tn=127fpr	0.1533333333parse_failed_rate_predict	0parse_failed_rate_answer	0text_accuracy	0.4233333333text_accuracy_positive_only	0.0000000000subset_negative_accuracy	0.8466666667subset_positive_accuracy	0.1333333333<br>**grammar_redactor_set:**accuracy	0.5733333333precision	0.5785714286recall	0.54f1	0.5586206897confusion_matrix	tp=81 fp=59 fn=69 tn=91fpr 0.3933333333parse_failed_rate_predict	0parse_failed_rate_answer	0text_accuracy	0.3366666667text_accuracy_positive_only	0.0666666667subset_negative_accuracy	0.6066666667subset_positive_accuracy	0.54<br>**lora-grammar-fixed:**accuracy	0.7233333333precision	0.7189542484recall	0.7333333333f1	0.7260726073confusion_matrix	tp=110 fp=43 fn=40 tn=107fpr	0.2866666667parse_failed_rate_predict	0parse_failed_rate_answer	0text_accuracy	0.3633333333text_accuracy_positive_only	0.0133333333subset_negative_accuracy	0.7133333333subset_positive_accuracy	0.7333333333  **lora-grammar-fixed (test_v4):**accuracy	0.7766666667precision	0.8246753247recall	0.7604790419f1	0.7912772586confusion_matrix	tp=127 fp=27 fn=40 tn=106fpr	0.2030075188parse_failed_rate_predict	0parse_failed_rate_answer	0text_accuracy	0.4166666667text_accuracy_positive_only	0.1137724551subset_negative_accuracy	0.7969924812subset_positive_accuracy	0.7604790419 |
| Markdown | Добавили в ОБМ, надос отсмотреть | 
1. Сверяем WER для текста до и после без учета символов маркдауна
2. Проверяем правильность маркдауна чекалкой из ревизора | **GigaChat-2-Pro**:* Rows total: 300; Valid for flag metrics: 299 (99.67%)
* Confusion matrix: TP=137, TN=13, FP=0, FN=149
* Accuracy=0.5017; Precision=1.0000; Recall=0.4790; F1=0.6478
* Acc GT+: 0.4790 (137/286); Acc GT-: 1.0000 (13/13)
* bad_gt=0 (0.00%); bad_pred=1 (0.33%); flag=True empty-text fallback=0
* WER mean=0.0468; WER median=0.0000; WER pass rate (WER=0)=83.00%
* Revizor pass rate=262/300 (87.33%); Top buckets: excessive_bold_words(19), lists_have_inconsistency(15), quoted_punctuation_duplication(5), high_bold_percentage(3), too_many_separators(2)  **LoRA:*** Rows total: 300; Valid for flag metrics: 300 (100.00%)
* Confusion matrix: TP=272, TN=11, FP=2, FN=15
* Accuracy=0.9433; Precision=0.9927; Recall=0.9477; F1=0.9697
* Acc GT+: 0.9477 (272/287); Acc GT-: 0.8462 (11/13)
* bad_gt=0 (0.00%); bad_pred=0 (0.00%); flag=True empty-text fallback=0
* WER mean=0.0533; WER median=0.0000; WER pass rate (WER=0)=72.67%
* Revizor pass rate=267/300 (89.00%); Top buckets: lists_have_inconsistency(17), excessive_bold_words(12), quoted_punctuation_duplication(5), high_bold_percentage(1) |
| Смена языков | Добавили в ОБМ, надо отсмотреть | Сверяем совпадение предсказанного флага с эталонным (accuracy) | **GigaChat-2-Pro**:accuracy	0.7133333333precision	0.3968253968recall	0.3676470588f1	0.3816793893confusion_tp	25confusion_fp	38confusion_fn	43confusion_tn	189parse_failed_rate_predict	0.0166666667parse_failed_rate_answer	0processed_failed_rate	0.0166666667subset_negative_accuracy	0.8146551724subset_positive_accuracy	0.3676470588 |
| Викачек (Instruction что-то там check) | Нет в ОБМ | Сверяем совпадение предсказанного флага с эталонным (accuracy) |    |
| Ты/Вы | Добавили в ОБМ, надо отсмотреть | Сверяем совпадение предсказанного флага с эталонным (accuracy) | **GigaChat-2-Pro:**accuracy	0.69precision	0.9recall	0.2307692308f1	0.3673469388confusion_tp	9confusion_fp	1confusion_fn	30confusion_tn	60parse_failed_rate_predict	0parse_failed_rate_answer	0subset_negative_accuracy	0.9836065574subset_positive_accuracy	0.2307692308 |

\n**Русификатор**: /home/nvbotaev/notebooks/TestSetTask_rusificator_december/TestSetBenchmark_rusificator.ipynb\n**Markdown**: /home/nvbotaev/notebooks/TestSetTask_Markdown/MarkdownBenchmark.ipynb\n**Смена языков**: /home/nvbotaev/notebooks/TestSetTask_langchange/Benchmark_lang.ipynb