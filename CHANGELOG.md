🌐 [Читать на русском](CHANGELOG.ru.md)

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
