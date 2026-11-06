🌐 [Read this in English](README.md)

# Грамматические справочники

Каталог содержит общие для всех языков справочники ВепКар: граммемы, грамматические наборы и части речи. Файлы небольшие, поэтому хранятся несжатыми UTF-8 CSV, и отдельные примеры для них не нужны.

```text
parts_of_speech.csv        grams.csv
        ▲                      ▲
        │ pos_id               │ gram_id_*
        │                      │
gramset_pos.csv ─────────► gramsets.csv
                gramset_id     ▲
                               │ gramset_id
             ┌─────────────────┴──────────────────┐
../morphology/lemma_wordforms_<lang>   ../../corpus/candidate_analyses_<lang>
```

## Файлы

| Файл | Строк | Что представляет строка |
|---|---|---|
| `grams.csv` | 44 | Граммема. |
| `gramsets.csv` | 213 | Грамматический набор. |
| `gramset_pos.csv` | 1 704 | Применимость набора к части речи в языке. |
| `parts_of_speech.csv` | 21 | Часть речи. |

## `grams.csv`

Схема:

```csv
gram_id,gram_category_id,category_en,category_ru,name_en,name_ru,name_short_en,name_short_ru,lgr,unimorph,conll,sequence_number
```

| Поле | Значение |
|---|---|
| `gram_id` | Идентификатор граммемы. |
| `gram_category_id` | Идентификатор грамматической категории. |
| `category_en`, `category_ru` | Название категории, например `case` / `падеж`. |
| `name_en`, `name_ru` | Название граммемы. |
| `name_short_en`, `name_short_ru` | Краткое название. Может быть пустым. |
| `lgr` | Код, из которого строится поле `gramset`. |
| `unimorph` | Соответствие в UniMorph. Может быть пустым. |
| `conll` | Соответствие в формате CoNLL. Может быть пустым. |
| `sequence_number` | Порядковый номер, заданный в ВепКар. |

`unimorph` и `conll` — исходные справочные поля, поэтому опечатка `VebForm` в `conll` сохранена как в источнике.

## `gramsets.csv`

Схема:

```csv
gramset_id,gramset,gram_id_number,gram_id_case,gram_id_tense,gram_id_person,gram_id_mood,gram_id_negation,gram_id_infinitive,gram_id_voice,gram_id_participle,gram_id_reflexive
```

| Поле | Значение |
|---|---|
| `gramset_id` | Идентификатор набора. |
| `gramset` | Текстовое представление набора. Совпадает с полем `gramset` в `candidate_analyses_*.csv` для того же `gramset_id`. |
| `gram_id_*` | Ссылки на `grams.csv.gram_id` по десяти категориям. `\N` означает, что категория в наборе не задана. |

Пример:

```csv
gramset_id,gramset,gram_id_number,gram_id_case,gram_id_tense,gram_id_person,gram_id_mood,gram_id_negation,gram_id_infinitive,gram_id_voice,gram_id_participle,gram_id_reflexive
1,SG+NOM,1,3,\N,\N,\N,\N,\N,\N,\N,\N
```

`gramset` собирается из кодов `lgr` через `+` в порядке столбцов `gram_id_*`.

Не разбирайте `gramset` по знаку `+`, если нужны отдельные граммемы: код граммемы 52 сам равен `AD+ALL`, например в наборе `SG+AD+ALL`. Используйте поля `gram_id_*`.

В текущей выгрузке `gram_id_reflexive` равен `\N` во всех наборах.

## `gramset_pos.csv`

Схема:

```csv
gramset_id,lang,pos_id
```

| Поле | Значение |
|---|---|
| `gramset_id` | Ссылка на `gramsets.csv`. |
| `lang` | Код языка: `vep`, `krl`, `olo` или `lud`. |
| `pos_id` | Ссылка на `parts_of_speech.csv`. |

Файл описывает, какие наборы заданы для части речи в языке. Он не гарантирует, что у конкретной словоформы встречается такой набор.

## `parts_of_speech.csv`

Схема:

```csv
pos_id,pos,name_en,name_ru
```

| Поле | Значение |
|---|---|
| `pos_id` | Идентификатор части речи. |
| `pos` | Код части речи, например `NOUN`. |
| `name_en`, `name_ru` | Название части речи. |

## Чтение файлов

`\N` используется в этих справочниках только в полях `gram_id_*`. Правило чтения то же, что и в [словарной морфологии](../morphology/README.ru.md#пустые-значения):

```python
import pandas as pd

gramsets = pd.read_csv(
    "gramsets.csv",
    dtype=str,
    keep_default_na=False,
    na_values=[r"\N"],
)
```
