[Читать на русском](README.ru.md)

# Corpus Metadata Reference Tables

This directory contains VepKar reference tables for corpora/subcorpora, genres, and dialects. They explain the values of `corpus_id`, `genre_id`, and `dialect_code` in [`texts_<lang>.csv.zst`](../README.md).

The tables include all source records, not only records referenced by texts in the four project languages.

```text
texts_<lang>.csv
    |
    +-- corpus_id ------> corpora.csv
    |
    +-- genre_id -------> genres.csv
    |                         |
    |                         +-- parent_id --> genres.csv
    |                         +-- corpus_id --> corpora.csv
    |
    +-- dialect_code ---> dialects.csv
```

## Format

These small tables are stored as uncompressed CSV files:

- encoding: UTF-8;
- delimiter: comma;
- first row: column names;
- embedded quotation marks in quoted fields are doubled;
- SQL `NULL`: `\N`;
- an empty cell: an empty string;
- rows are sorted by their primary identifier.

Source values are exported without corrections or substitutions. For example, a missing English name is not replaced with its Russian equivalent.

A literal source value of `\N` conflicts with the null marker. The exporter checks for such collisions and fails if one is found.

## `corpora.csv`

Each row represents one VepKar corpus or subcorpus.

Source table: `corpuses`.

```csv
corpus_id,name_en,name_ru
```

| Column | Description |
|---|---|
| `corpus_id` | Primary identifier, `corpuses.id` |
| `name_en` | English name |
| `name_ru` | Russian name |

`corpus_id` joins to the corresponding field in `texts_*` and to `genres.csv.corpus_id`.

## `genres.csv`

Each row represents one genre. The table preserves the genre hierarchy and the association between a genre and a corpus/subcorpus.

Source table: `genres`.

```csv
genre_id,parent_id,corpus_id,sequence_number,name_en,name_ru,name_pl_en,name_pl_ru
```

| Column | Description |
|---|---|
| `genre_id` | Primary identifier, `genres.id` |
| `parent_id` | Parent genre identifier; `0` denotes a root genre. SQL `NULL`, if present in the source, is preserved as `\N` |
| `corpus_id` | Corpus/subcorpus identifier in `corpora.csv` |
| `sequence_number` | VepKar display order; not a genre identifier |
| `name_en`, `name_ru` | Singular names |
| `name_pl_en`, `name_pl_ru` | Plural names |

`parent_id = 0` is a root marker, not a reference requiring a row with `genre_id = 0`.

Empty and missing names are preserved. The exporter does not generate plural names or replace them with singular names.

The hierarchy can be used to derive broader research categories. Such groupings are derived data and must not be written back into this reference table.

## `dialects.csv`

Each row represents one dialect or local variety registered in VepKar.

Sources: `dialects`, with the language code joined from `langs`.

```csv
dialect_id,dialect_code,lang_code,sequence_number,name_en,name_ru
```

| Column | Description |
|---|---|
| `dialect_id` | Primary identifier, `dialects.id` |
| `dialect_code` | Dialect code, `dialects.code`; used in `texts_*.dialect_code` |
| `lang_code` | Language code from `langs.code` |
| `sequence_number` | VepKar display order |
| `name_en`, `name_ru` | English and Russian names |

`dialect_id` also explains dialect references in [`dict/morphology/lemma_wordforms_*`](../../dict/morphology/README.md).

Language codes are joined using a left join. A missing language record does not remove the dialect from the export; it produces `\N` and requires a data-quality check.

## Joining to Text Metadata

In `texts_*`, the fields `corpus_id`, `genre_id`, and `dialect_code` may contain multiple values separated by `|`.

For example:

```text
corpus_id = 1|3
```

represents two references to `corpora.csv`, not a single compound identifier.

Before joining:

1. Read the file with a standard CSV parser.
2. Split nonempty values of the relevant field on `|`.
3. Join individual values to the reference table.

Element order does not indicate priority. Account for multiple memberships when calculating statistics: a text may belong to more than one group.

Do not silently discard source records with unresolved references. Use left joins and inspect missing matches separately.

## Reading with pandas

```python
import pandas as pd

genres = pd.read_csv(
    "corpus/reference/genres.csv",
    dtype=str,
    keep_default_na=False,
    na_values=[r"\N"],
)
```

This converts `\N` to a missing value while preserving empty cells as empty strings.

The `\N` rule applies to these reference tables. Do not automatically apply it to all other corpus CSV files; see the [corpus README](../README.md) for their conventions.

***

## 3. Обновить описание `texts_*` в `corpus/README.ru.md`

Новая схема:

```csv
text_id,corpus_id,dialect_code,genre_id,year_recorded,year_published
```

Таблицу полей заменить на следующую:

| Поле | Тип | Описание |
|---|---|---|
| `text_id` | integer | Идентификатор текста ВепКар |
| `corpus_id` | string or empty | Один или несколько идентификаторов из [`reference/corpora.csv`](reference/README.ru.md#corporacsv), разделённых `\|`; пустая ячейка означает отсутствие значения в экспорте |
| `dialect_code` | string or empty | Один или несколько кодов из [`reference/dialects.csv`](reference/README.ru.md#dialectscsv), разделённых `\|` |
| `genre_id` | string or empty | Один или несколько идентификаторов из [`reference/genres.csv`](reference/README.ru.md#genrescsv), разделённых `\|` |
| `year_recorded` | source value or empty | Год записи исходного материала; экспортируется из `events.date` |
| `year_published` | source value or empty | Год публикации, указанный в связанном источнике; экспортируется из `sources.year` |

После таблицы добавить:

> `year_recorded` и `year_published` — независимые поля. Отсутствующее значение одного не заполняется значением другого. Экспортёр сохраняет исходные значения без приведения к целому числу. Пустая ячейка означает отсутствие значения в экспорте; маркер `\N` в этих полях не используется.

Тип `source value or empty` здесь выбран намеренно: пока не подтверждён фактический тип и диапазон значений в БД, документация не должна обещать исключительно целочисленные годы.

## 4. Обновить описание `words_*` в `corpus/README.ru.md`

Новая схема:

```csv
word_id,sentence_id,word_number,word,w_id,s_id
```

Добавить к существующей таблице:

| Поле | Тип | Описание |
|---|---|---|
| `w_id` | integer or empty | Исходный идентификатор элемента `<w id="…">` в XML текста; источник — `words.w_id` |
| `s_id` | integer or empty | Исходный идентификатор элемента `<s id="…">` в XML текста; источник — `words.s_id` |

И отдельный подраздел:

### Связь с исходной XML-разметкой

```text
word_id      -> words.id
sentence_id  -> sentences.id

w_id         -> <w id="...">
s_id         -> <s id="...">
```

`word_id` и `sentence_id` — идентификаторы записей БД. `w_id` и `s_id` — идентификаторы в XML исходного текста; их нельзя считать глобально уникальными между текстами.

Для получения исходного написания:

1. Найдите предложение по `sentence_id` в `sentences_*`.
2. Проверьте соответствие его атрибута `<s id="…">` значению `s_id`.
3. Найдите внутри предложения элемент `<w>` с атрибутом `id`, равным `w_id`.
4. Получите текстовое содержимое элемента XML-парсером.

Поле `word` нормализовано и не используется как ключ поиска XML-элемента. `word_number` задаёт позицию слова в предложении и не заменяет `w_id`.

Поля `w_id` и `s_id` выгружаются из БД без пересчёта. При SQL `NULL` существующий корпусный CSV-writer записывает пустую ячейку.

Отсутствующие предложения или XML-соответствия не исправляются экспортёром и не служат основанием для удаления слов из выгрузки.

## 5. Английские вставки для `corpus/README.md`

В разделе `texts_*` обновить схему:

```csv
text_id,corpus_id,dialect_code,genre_id,year_recorded,year_published
```

Добавить или обновить описания:

| Column | Type | Description |
|---|---|---|
| `corpus_id` | string or empty | One or more identifiers in [`reference/corpora.csv`](reference/README.md#corporacsv), separated by `\|` |
| `dialect_code` | string or empty | One or more codes in [`reference/dialects.csv`](reference/README.md#dialectscsv), separated by `\|` |
| `genre_id` | string or empty | One or more identifiers in [`reference/genres.csv`](reference/README.md#genrescsv), separated by `\|` |
| `year_recorded` | source value or empty | Recording year of the source material, exported from `events.date` |
| `year_published` | source value or empty | Publication year recorded in the linked source, exported from `sources.year` |

После таблицы:

> `year_recorded` and `year_published` are independent fields. Neither is used to fill missing values in the other. Source values are exported without integer conversion. An empty cell indicates an unavailable value in the export; these fields do not use the `\N` marker.

В разделе `words_*` обновить схему:

```csv
word_id,sentence_id,word_number,word,w_id,s_id
```

Добавить поля:

| Column | Type | Description |
|---|---|---|
| `w_id` | integer or empty | Original `<w id="…">` identifier in the text XML, from `words.w_id` |
| `s_id` | integer or empty | Original `<s id="…">` identifier in the text XML, from `words.s_id` |

И пояснение:

### Linking to the Original XML

`word_id` and `sentence_id` identify database records. `w_id` and `s_id` identify XML elements within the source text and must not be treated as globally unique across texts.

To recover the original spelling, locate the sentence using `sentence_id`, check its `<s id="…">` against `s_id`, and locate the `<w>` element whose `id` equals `w_id`. Read its textual content with an XML parser.

The normalized `word` field is not an XML lookup key. `word_number` specifies the position within a sentence and is not a substitute for `w_id`.

The exporter preserves the stored XML identifiers without recalculation. SQL `NULL` is written as an empty cell by the existing corpus CSV writer.

Missing sentences or XML matches are not repaired by the exporter and do not cause word records to be removed.