🌐 [Читать на русском](README.ru.md)

# Grammar reference tables

This directory contains VepKar reference tables shared by all languages: grammemes, grammatical sets, and parts of speech. The files are small, so they are stored as uncompressed UTF-8 CSV and need no separate samples.

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

## Files

| File | Rows | One row represents |
|---|---|---|
| `grams.csv` | 44 | A grammeme. |
| `gramsets.csv` | 213 | A grammatical set. |
| `gramset_pos.csv` | 1,704 | Applicability of a set to a part of speech in a language. |
| `parts_of_speech.csv` | 21 | A part of speech. |

## `grams.csv`

Schema:

```csv
gram_id,gram_category_id,category_en,category_ru,name_en,name_ru,name_short_en,name_short_ru,lgr,unimorph,conll,sequence_number
```

| Column | Meaning |
|---|---|
| `gram_id` | Grammeme identifier. |
| `gram_category_id` | Grammatical category identifier. |
| `category_en`, `category_ru` | Category name, for example `case` / `падеж`. |
| `name_en`, `name_ru` | Grammeme name. |
| `name_short_en`, `name_short_ru` | Short name. May be empty. |
| `lgr` | Code used to build the `gramset` field. |
| `unimorph` | UniMorph equivalent. May be empty. |
| `conll` | CoNLL-format equivalent. May be empty. |
| `sequence_number` | Sequence number assigned in VepKar. |

`unimorph` and `conll` are source reference fields, so the typo `VebForm` in `conll` is preserved as in the source.

## `gramsets.csv`

Schema:

```csv
gramset_id,gramset,gram_id_number,gram_id_case,gram_id_tense,gram_id_person,gram_id_mood,gram_id_negation,gram_id_infinitive,gram_id_voice,gram_id_participle,gram_id_reflexive
```

| Column | Meaning |
|---|---|
| `gramset_id` | Grammatical set identifier. |
| `gramset` | Text representation of the set. It matches the `gramset` field in `candidate_analyses_*.csv` for the same `gramset_id`. |
| `gram_id_*` | References to `grams.csv.gram_id` for ten categories. `\N` means the category is not set in this grammatical set. |

Example:

```csv
gramset_id,gramset,gram_id_number,gram_id_case,gram_id_tense,gram_id_person,gram_id_mood,gram_id_negation,gram_id_infinitive,gram_id_voice,gram_id_participle,gram_id_reflexive
1,SG+NOM,1,3,\N,\N,\N,\N,\N,\N,\N,\N
```

`gramset` is built from `lgr` codes joined with `+`, in the order of the `gram_id_*` columns.

Do not split `gramset` on `+` to obtain individual grammemes: the code of grammeme 52 is itself `AD+ALL`, as in the set `SG+AD+ALL`. Use the `gram_id_*` columns instead.

In the current export, `gram_id_reflexive` is `\N` in all sets.

## `gramset_pos.csv`

Schema:

```csv
gramset_id,lang,pos_id
```

| Column | Meaning |
|---|---|
| `gramset_id` | Reference to `gramsets.csv`. |
| `lang` | Language code: `vep`, `krl`, `olo`, or `lud`. |
| `pos_id` | Reference to `parts_of_speech.csv`. |

The file lists which sets are defined for a part of speech in a language. It does not guarantee that a particular wordform occurs with such a set.

## `parts_of_speech.csv`

Schema:

```csv
pos_id,pos,name_en,name_ru
```

| Column | Meaning |
|---|---|
| `pos_id` | Part-of-speech identifier. |
| `pos` | Part-of-speech code, for example `NOUN`. |
| `name_en`, `name_ru` | Part-of-speech name. |

## Reading the files

In these tables, `\N` is used only in the `gram_id_*` columns. The reading rule is the same as in [dictionary morphology](../morphology/README.md#empty-values):

```python
import pandas as pd

gramsets = pd.read_csv(
    "gramsets.csv",
    dtype=str,
    keep_default_na=False,
    na_values=[r"\N"],
)
```
