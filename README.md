# ru-nlpaug
```markdown
# ru_nlpaug

Библиотека для морфологически согласованной аугментации русскоязычных текстов, построенная поверх [nlpaug](https://github.com/makcedward/nlpaug).

## Описание

**ru_nlpaug** адаптирует англоязычный инструментарий nlpaug под особенности русского языка:

- свободный порядок слов,
- богатая морфология (падежи, род, число, время),
- ограниченное покрытие русскоязычных WordNet-подобных ресурсов.

Библиотека добавляет токенизационно-морфологический слой (razdel + Stanza + PyMorphy2) и набор русскоязычных аугментеров, сохраняя совместимость с API nlpaug.

## Возможности

| Аугментер | Описание |
|-----------|----------|
| `RuSynonymAug` | Замена слов на синонимы из словаря ruAntSynNet |
| `RuAntonymAug` | Замена слов на антонимы из словаря ruAntSynNet |
| `RuWordEmbsAug` | Замена слов на семантически близкие (Word2Vec / Tayga) |
| `RuBackTranslationAug` | Перефразирование через обратный перевод (ru → en → ru) |

Все аугментеры автоматически согласуют морфологию заменяемых слов с контекстом.

## Установка

```bash
# Клонировать репозиторий
git clone https://github.com/your-username/ru_nlpaug.git
cd ru_nlpaug

# Создать виртуальное окружение
python -m venv venv
source venv/bin/activate  # Linux/Mac
# или
.\venv\Scripts\Activate.ps1  # Windows PowerShell

# Установить зависимости
pip install -r requirements.txt

# Скачать модели для Stanza (один раз)
python -c "import stanza; stanza.download('ru')"
```

## Быстрый старт

### Синонимическая аугментация

```python
from ru_nlpaug.augmenter.word import RuSynonymAug

aug = RuSynonymAug(
    dict_path="data/ant_syn_pairs.csv",
    aug_p=0.3,      # вероятность замены слова
    aug_min=1,      # минимум замен
    aug_max=5       # максимум замен
)

text = "Самый интересный курс по математике!"
result = aug.augment(text)
print(result)
# Пример вывода: "Самый прелестный курс по математике!"
```

### Антонимическая аугментация

```python
from ru_nlpaug.augmenter.word import RuAntonymAug

aug = RuAntonymAug(
    dict_path="data/ant_syn_pairs.csv",
    aug_p=0.3
)

text = "Лекции скучные, материал плохой."
result = aug.augment(text)
print(result)
# Пример вывода: "Лекции интересные, материал хороший."
```

### Аугментация с помощью векторов W2V

```python
from ru_nlpaug.augmenter.word import RuWordEmbsAug

aug = RuWordEmbsAug(
    model_path="path/to/tayga_model.bin",
    aug_p=0.2
)

text = "Отличный преподаватель объясняет понятно."
result = aug.augment(text)
print(result)
```

### Обратный перевод

```python
from ru_nlpaug.augmenter.word import RuBackTranslationAug

aug = RuBackTranslationAug()

text = "Курс помог разобраться в сложной теме."
result = aug.augment(text)
print(result)
# Пример вывода: "Курс помог мне понять сложную тему."
```

## Структура проекта

```
ru_nlpaug/
├── augmenter/
│   └── word/
│       ├── ru_synonym_aug.py      # RuSynonymAug
│       ├── ru_antonym_aug.py      # RuAntonymAug
│       ├── ru_word_embs_aug.py    # RuWordEmbsAug
│       └── ru_backtranslation_aug.py
├── util/
│   └── text/
│       ├── ru_tokenizer.py        # токенизация (razdel)
│       └── ru_part_of_speech.py   # морфология (Stanza + PyMorphy2)
├── data/
│   ├── ant_syn_pairs.csv          # словарь синонимов/антонимов
│   ├── stop.txt                   # стоп-слова
│   └── preps.txt                  # предлоги
└── requirements.txt
```

## Лексический ресурс

Синонимы и антонимы берутся из корпуса [ruAntSynNet](https://github.com/dasha-b/ruAntSynNet) (Бородина, 2023).

Формат файла `ant_syn_pairs.csv`:

```
ID,word1,word2,relation_type
1,чистый,белый,S
2,добрый,злой,A
...
```

- `S` — синонимы
- `A` — антонимы

**Ограничение:** корпус содержит преимущественно прилагательные.

## Результаты экспериментов

### Классификация отзывов Stepik

| Сценарий | Accuracy | Macro F1 |
|----------|----------|----------|
| Без аугментации (40 train) | 66.7% | 0.61 |
| С RuSynonymAug (40 → 80 train) | **78.3%** | **0.77** |

### Метрики качества аугментации

| Метрика | RuSynonymAug |
|---------|--------------|
| Jaccard | 0.909 |
| BLEU-4 | 0.918 |
| ROUGE-L F1 | 0.300 |

## Ограничения

1. **Охват частей речи** — словарь ruAntSynNet содержит в основном прилагательные; существительные, глаголы и наречия не затрагиваются.

2. **Выбор синонима без учёта контекста** — синоним выбирается случайно, что может приводить к стилистически неуместным сочетаниям.

3. **Зависимость от качества морфоанализа** — корректность согласования определяется точностью Stanza и PyMorphy2.

## Зависимости

- Python 3.9+
- numpy
- nlpaug
- razdel
- stanza
- pymorphy2
- nltk (для BLEU)
- rouge-score (для ROUGE)

Полный список — в `requirements.txt`.

## Авторы

- [Алёна Берлин]
- [Полина Павлухина]


## Ссылки

- [nlpaug](https://github.com/makcedward/nlpaug) — оригинальная библиотека
- [ruAntSynNet](https://github.com/dasha-b/ruAntSynNet) — словарь синонимов и антонимов
- [Stanza](https://stanfordnlp.github.io/stanza/) — NLP-пайплайн Stanford
- [PyMorphy2](https://pymorphy2.readthedocs.io/) — морфологический анализатор
- [razdel](https://github.com/natasha/razdel) — токенизатор для русского языка
```
