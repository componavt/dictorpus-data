🌐 [Читать на русском](CHANGELOG.ru.md)

## v2026.11 — 2026-11-06

- Added the `dict/morphology/` directory: `lemmas_*`, `lemma_wordforms_*`
  for all four language varieties, including the `affix` column.
- Added the `dict/grammar/` directory with uncompressed reference tables:
  `grams.csv`, `gramsets.csv`, `gramset_pos.csv`, `parts_of_speech.csv`.
- **Schema change:** added the `gramset_id` column after `wordform_id`
  in `corpus/candidate_analyses_*.csv.zst`. Loaders that read columns
  by position must be updated.
- In the new directories, SQL `NULL` is written as `\N`, and an empty cell
  means an empty string. The format of other files has not changed.
- Updated the corpus: `krl` +2 texts, +46 sentences, +492 words,
  +466 candidates; `olo` +1 text, +36 sentences, +544 words,
  +674 candidates. Existing rows have not changed.
- In `dict/meanings_lud.csv.zst`, `id` values changed; the content is the same.
- Documentation: corrected the description of how `gramset` is built
  (only `lgr` is used) and the CSV quoting rules; described repeated
  `meaning_id` values.
- Samples: added `gramset_id` to `corpus/samples/candidate_analyses_krl.csv`,
  updated `id` values in `dict/samples/examples_krl.csv`, and added
  `dict/morphology/samples/`.
- Source: VepKar / Dictorpus database export.

## v2026.10 — 2026-11-05

- Added the `meaning_n` column — the meaning number within a dictionary
  entry — to the `dict/meanings_*.csv.zst` dictionary exports.
- Added the `dict/synonyms/` directory with synonym set data
  for adjectives in the Livvi dialect of Karelian (`olo`):
  `synsets_*`, `synset_members_*`, `synset_candidates_*`, `collocations_*`.
  The exports contain synonym sets and their members, automatically
  proposed candidates rejected by the expert, and collocation data.
- Removed unnecessary quotation marks around fields in CSV exports.
- Updated the dictionary export documentation; added documentation
  for synonym data and its links to dictionary meanings.
- Source: VepKar / Dictorpus database export.

## v2026.09 — 2026-09-19

- Added the meanings_* and examples_* dictionary exports to /dict.
- Added the following exports to /corpus:
  candidate_analyses_*, sentences_*, texts_*, words_*.
- Added concept_categories_wdh.tsv and concepts to /semantics.
- Source: VepKar / Dictorpus database export.
