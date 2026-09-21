🌐 [Читать на русском](README.ru.md)

# Dictionary exports

This directory contains per-meaning VepKar dictionary exports for four language varieties.

```text
meanings_<lang>.csv.zst
        │
        │ meaning_id
        ▼
examples_<lang>.csv.zst
```

The files describe dictionary meanings, not merely lemma headwords. A single lemma can have several rows in `meanings_<lang>.csv.zst`, one per distinct dictionary meaning.

## Files

| File pattern | One row represents | Main identifier |
|---|---|---|
| `meanings_<lang>.csv.zst` | A dictionary meaning of a lemma | `meaning_id` |
| `examples_<lang>.csv.zst` | An example associated with a dictionary meaning | `id` |
| `samples/meanings_krl.csv` | Small uncompressed Karelian Proper example | — |
| `samples/examples_krl.csv` | Small uncompressed Karelian Proper example | — |

`<lang>` is one of `vep`, `krl`, `olo`, or `lud`.

## `meanings_<lang>.csv.zst`

Schema:

```csv
id,meaning_id,lemma_id,lemma,lang,pos,meaning_ru,meaning_en,concept_id,category_id
```

| Column | Meaning |
|---|---|
| `id` | Export-row identifier. It is useful for traceability inside this export. |
| `meaning_id` | Identifier of the VepKar dictionary meaning. Use it to join examples. |
| `lemma_id` | Identifier of the dictionary lemma/headword. |
| `lemma` | Lemma spelling in the relevant language variety. |
| `lang` | Language code: `vep`, `krl`, `olo`, or `lud`. |
| `pos` | Part of speech, such as `NOUN`, `VERB`, `ADJ`, `ADV`, or `PART`. |
| `meaning_ru` | Russian dictionary meaning or definition. It may be empty or may contain punctuation, parentheses, and several comma-separated senses. |
| `meaning_en` | English translation supplied by dictionary editors, when available. Empty values mean that no English translation is present in this source export. |
| `concept_id` | Optional identifier of a semantic concept associated with the meaning. Join to `../semantics/concepts.csv` on `concept_id`. |
| `category_id` | Optional semantic category identifier. Join to `../semantics/concept_categories_wdh.tsv` on `category_id`. |

Example:

```csv
id,meaning_id,lemma_id,lemma,lang,pos,meaning_ru,meaning_en,concept_id,category_id
1522,48286,42267,aškel,krl,NOUN,шаг,step,492,B15
```

Interpretation:

```text
Karelian Proper lemma: aškel
part of speech:         NOUN
Russian meaning:         шаг
English meaning:         step
semantic concept:        492
semantic category:       B15
```

### Semantic links

The semantic fields are optional source annotations. Blank `concept_id` or `category_id` means that the export does not provide that link; it must not be replaced with a synthetic value.

```text
meanings_<lang>.csv
  concept_id ─────────────► ../semantics/concepts.csv
  category_id ────────────► ../semantics/concept_categories_wdh.tsv
```

`concept_id` and `category_id` are metadata links, not substitutes for `meaning_ru` or `meaning_en`. In particular, do not automatically replace a meaning's English translation with `concept_en` from `concepts.csv`.

## `examples_<lang>.csv.zst`

Schema:

```csv
id,meaning_id,example
```

| Column | Meaning |
|---|---|
| `id` | Export-row identifier for the example record. |
| `meaning_id` | Identifier of the dictionary meaning illustrated by the example. Join to `meanings_<lang>.csv.zst.meaning_id`. |
| `example` | Example sentence or fragment. It may contain commas, quotation marks, punctuation, or repeated source examples. |

Example:

```csv
id,meaning_id,example
1,81276,"Делали печи, из этих же кирпичей складывали кучу как печки..."
```

Several example rows may refer to the same `meaning_id`. Repeated text is preserved from the source export and must not be silently deduplicated.

## Join pattern

```python
import pandas as pd

meanings = pd.read_csv("meanings_krl.csv.zst", compression="zstd", dtype=str)
examples = pd.read_csv("examples_krl.csv.zst", compression="zstd", dtype=str)

joined = examples.merge(
    meanings,
    on="meaning_id",
    how="left",
    validate="many_to_one",
)
```

The `many_to_one` relation is expected:

```text
many example rows
        │
        ▼
one dictionary meaning
```

## Reading files

```bash
zstd -dc meanings_krl.csv.zst | head
zstd -dc examples_krl.csv.zst | head
```

Use a CSV parser rather than manual string splitting. Both `meaning_ru` and `example` can contain commas and quotation marks.
