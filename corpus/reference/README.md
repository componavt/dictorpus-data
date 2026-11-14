🌐 [Читать на русском](README.ru.md)

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
