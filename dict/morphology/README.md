🌐 [Читать на русском](README.ru.md)

# Dictionary morphology

This directory contains all VepKar lemmas and the links between lemmas and wordforms for four language varieties. The export is not limited to words that occur in the corpus.

```text
lemmas_<lang>.csv.zst
        │
        │ lemma_id
        ▼
lemma_wordforms_<lang>.csv.zst
        │
        │ gramset_id
        ▼
../grammar/gramsets.csv
```

`<lang>` is one of `vep`, `krl`, `olo`, or `lud`. A file's language is determined by the language of the lemma.

The [`samples/`](samples/) directory contains small uncompressed Karelian Proper examples. The rows were selected by hand to show different field values: this is not a random sample and must not be used for research evaluation.

## Empty values

These files distinguish SQL `NULL` from an empty string:

| Cell value | Meaning |
|---|---|
| `\N` | SQL `NULL` |
| empty cell | empty string |
| anything else | source value |

To preserve this distinction in pandas, disable the default missing-value handling:

```python
import pandas as pd

wordforms = pd.read_csv(
    "lemma_wordforms_krl.csv.zst",
    compression="zstd",
    dtype=str,
    keep_default_na=False,
    na_values=[r"\N"],
)
```

## `lemmas_<lang>.csv.zst`

Schema:

```csv
lemma_id,lemma,lang,pos_id,pos
```

| Column | Meaning |
|---|---|
| `lemma_id` | Lemma identifier. It is unique and is referenced by `lemma_wordforms_<lang>` and `../meanings_<lang>.csv.zst`. |
| `lemma` | Lemma spelling. |
| `lang` | Language code. |
| `pos_id` | Part-of-speech identifier. Join to `../grammar/parts_of_speech.csv`. May be `\N`. |
| `pos` | Part-of-speech code. May be `\N`. |

The file also includes lemmas that have no wordforms or no dictionary meanings.

## `lemma_wordforms_<lang>.csv.zst`

Schema:

```csv
wordform_id,wordform,wordform_for_search,lemma_id,gramset_id,dialect_id,dialect_code,affix
```

| Column | Meaning |
|---|---|
| `wordform_id` | Wordform identifier. It matches `wordform_id` in `../../corpus/candidate_analyses_<lang>.csv.zst`. |
| `wordform` | Wordform in its dictionary spelling. |
| `wordform_for_search` | Stored search form. It may differ from `wordform`, for example `aštuis’` → `aštuis`. |
| `lemma_id` | Lemma identifier. |
| `gramset_id` | Grammatical set identifier. Join to `../grammar/gramsets.csv`. May be `\N`. |
| `dialect_id` | Dialect identifier. May be `\N`. |
| `dialect_code` | Dialect code. May be `\N`. |
| `affix` | Stored result of extracting the final part of the wordform; see below. |

One row describes one link, not one wordform. The same form may repeat with different lemmas, parts of speech, grammatical sets, and dialects. Such rows are not collapsed.

```csv
wordform_id,wordform,wordform_for_search,lemma_id,gramset_id,dialect_id,dialect_code,affix
8337,avun,avun,2942,3,46,krl-new,vun
8337,avun,avun,2942,56,46,krl-new,vun
```

### The `affix` field

`affix` is the result, stored in VepKar, of extracting the final part of a wordform relative to the stem recorded for its lemma. The value is not recalculated during export and is not a full morphemic annotation. The stem itself is not part of the export.

| Cell value | Meaning | Example |
|---|---|---|
| non-empty string other than `#` | The remainder of the wordform after the stem. | `avun` → `vun` |
| empty cell | The form matched the stem and the remainder is empty. | `apu` |
| `#` | Service marker: the form did not fit the “stem + remainder” pattern. | `kuččuu` |
| `\N` | The value is undefined or was not calculated. | `en aššu` |

`#` is not an affix and not a linguistic conclusion. The marker does not prove the absence of a shared stem or suppletion: the stored stem itself may also be the cause.

`\N` appears on all forms that contain a space. Forms without a grammatical set and forms of lemmas without a stored stem also receive it. The field does not record the reason, so `\N` in a particular row cannot be explained from this file alone.

Features for models should keep these four states distinct.

## Link to the corpus

Corpus candidates join on a pair of columns:

```text
candidate_analyses_<lang>.wordform_id + gramset_id
        │
        ▼
lemma_wordforms_<lang>.wordform_id + gramset_id
```

The join may return several rows per candidate because of multiple lemmas or dialects. In addition, not every candidate has a dictionary link. A missing link means a lack of coverage, not a reason to delete the candidate.

## Notes on the current export

- `gramset_id = 0` occurs in 12 rows (`vep`, `olo`, `lud`). This value is not present in `gramsets.csv`.
- `lemma_wordforms_vep` contains 4 fully identical rows. They are preserved as in the source.
- `dialect_code` does not always start with the file's language code: for example, `vep` has rows with `krl-*`, and `krl` has the prefix `klr-`.
