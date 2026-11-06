🌐 [Читать на русском](README.ru.md)

# dictorpus-data

Versioned source-data exports from the VepKar corpus and dictionaries.

This repository contains read-only, language-specific CSV datasets for research, reproducible processing, and downstream applications. It contains source exports and compact samples, not derived model outputs, temporary files, experimental splits, or personal annotations.

```text
                    ┌──────────────────────
                    │ corpus/
                    │ texts → sentences
                    │       → words
                    │       → candidates
                    └──────────────────────
                               │
                               │ corpus word / context
                               │
┌──────────────────────        │       ┌──────────────────────
│ dict/                        │       │ semantics/
│ meanings              ◄──────┼──────►│ concepts
│ examples                     │       │ category → WDH
│ morphology/                  │       │
│ grammar/                     │       │
│                              │       │
│ meaning_id                   │       │ concept_id
│ concept_id            ───────┘       │ category_id
│ category_id                          │ wdh
└──────────────────────                └──────────────────────
```

## Contents

| Directory | Question answered | Main datasets |
|---|---|---|
| [`corpus/`](corpus/README.md) | Which words occur in texts, in which context, and which morphological analyses are possible? | `texts_*`, `sentences_*`, `words_*`, `candidate_analyses_*` |
| [`dict/`](dict/README.md) | Which lemmas and dictionary meanings exist, which Russian/English meanings do they have, and which examples illustrate them? | `meanings_*`, `examples_*` |
| [`dict/morphology/`](dict/morphology/README.md) | Which lemmas does the dictionary contain, what are their wordforms, and which grammatical sets and affixes are linked to them? | `lemmas_*`, `lemma_wordforms_*` |
| [`dict/grammar/`](dict/grammar/README.md) | Which grammemes make up grammatical sets, and which parts of speech do the sets apply to? | `grams.csv`, `gramsets.csv`, `gramset_pos.csv`, `parts_of_speech.csv` |
| [`dict/synonyms/`](dict/synonyms/README.md) | Which dictionary meanings form synonym sets, and what are their collocational patterns? | `synsets_*`, `synset_members_*`, `synset_candidates_*`, `collocations_*` |
| [`semantics/`](semantics/README.md) | Which semantic concepts and WordNet Domain Hierarchy labels can be associated with dictionary meanings? | `concepts.csv`, `concept_categories_wdh.tsv` |

The `corpus/`, `dict/`, and `semantics/` directories are complementary. They should not be treated as three alternative versions of the same dataset.

## Language codes

| File suffix | Language variety |
|---|---|
| `vep` | Vepsian |
| `krl` | Karelian Proper |
| `olo` | Livvi Karelian |
| `lud` | Ludic |

For example, `dict/meanings_krl.csv.zst` contains Karelian Proper dictionary meanings, while `corpus/words_krl.csv.zst` contains Karelian Proper corpus word occurrences.

## File format

The full datasets are UTF-8 CSV files compressed with Zstandard:

```text
*.csv.zst
```

Only small reference tables are stored uncompressed: `dict/grammar/*.csv` and the files in `semantics/`.

The repository also provides small uncompressed CSV examples in `samples/` directories for quick inspection and schema testing.

Inspect a compressed file without creating an uncompressed copy:

```bash
zstd -dc dict/meanings_krl.csv.zst | head
```

Read it with pandas:

```python
import pandas as pd

meanings = pd.read_csv(
    "dict/meanings_krl.csv.zst",
    compression="zstd",
    dtype=str,
)
```

CSV fields may contain commas, quotation marks, and line-like punctuation. Use a standard CSV parser; do not split lines manually on commas.

In `dict/morphology/` and `dict/grammar/`, SQL `NULL` is written as `\N`, and an empty cell means an empty string. The reading rule is described in the [dictionary morphology README](dict/morphology/README.md#empty-values). Other files do not use this marker.

## Data policy

- Treat full `.csv.zst` files as immutable source snapshots.
- Do not write derived translations, model outputs, train/dev/test splits, predictions, or manual annotations back into this repository.
- Keep reproducible derived datasets in a separate repository or clearly separate results directory.
- Preserve identifier columns when joining datasets. They are the primary links between exports.

## Key joins

```text
corpus/
  texts_<lang>.csv        text_id
       │
       ▼
  sentences_<lang>.csv    sentence_id, text_id
       │
       ▼
  words_<lang>.csv        word_id, sentence_id
       │
       ▼
  candidate_analyses_<lang>.csv    word_id, wordform_id, gramset_id


dict/
  meanings_<lang>.csv     meaning_id, lemma_id, concept_id, category_id
       │
       ▼
  examples_<lang>.csv     meaning_id

dict/morphology/
  lemmas_<lang>.csv           lemma_id, pos_id
       │
       ▼
  lemma_wordforms_<lang>.csv  wordform_id, lemma_id, gramset_id

dict/grammar/
  gramsets.csv            gramset_id, gram_id_*
  grams.csv               gram_id
  gramset_pos.csv         gramset_id, pos_id
  parts_of_speech.csv     pos_id

semantics/
  concepts.csv            concept_id, category_id
  concept_categories_wdh.tsv
                          category_id → wdh
```


## License

Scripts and other source code in this repository are licensed under the
MIT License. Linguistic datasets in `corpus/`, `dict/`, and `semantics/`,
including sample data, are licensed under Creative Commons Attribution
4.0 International (CC BY 4.0).

## Attribution

When redistributing or adapting these data, please credit the VepKar project
(The Open Corpus of the Veps and Karelian Languages), link to
https://dictorpus.krc.karelia.ru/ and to the version of this repository used,
identify the license as CC BY 4.0, and indicate whether you modified the data.

Suggested attribution:
"VepKar — The Open Corpus of the Veps and Karelian Languages;
dictorpus-data, version v2026.11; licensed under CC BY 4.0."

## Citation

If you use this dataset in an academic publication, please cite the dataset
version you used. We also encourage you to cite a relevant VepKar publication
from https://dictorpus.krc.karelia.ru/en/page/publ.

See the README in each directory for its exact schema, identifiers, and join rules.
