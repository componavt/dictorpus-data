🌐 [Читать на русском](CHANGELOG.ru.md)

## v2026.13 — 2026-11-14

- Added the `corpus/reference/` directory with uncompressed reference tables
  for corpora/subcorpora, genres, and dialects: `corpora.csv` (13 rows),
  `genres.csv` (75), `dialects.csv` (58). They explain `corpus_id`,
  `genre_id`, and `dialect_code` in `texts_*`. SQL `NULL` is written
  as `\N`, as in `dict/morphology/` and `dict/grammar/`.
- **Schema change:** added the `year_published` column at the end of
  `corpus/texts_*.csv.zst` — the publication year from the linked source
  (`sources.year`). It does not replace `year_recorded`.
- **Schema change:** added the `w_id` and `s_id` columns after `word`
  at the end of `corpus/words_*.csv.zst` — identifiers of the `<w id>`
  and `<s id>` elements in the original XML markup. Loaders that check
  the exact set of columns must be updated.
- Updated the corpus: `krl` +5 texts, +149 sentences, +1,709 words,
  +9,275 candidates; `olo` +7 texts, +177 sentences, +2,198 words,
  +24,204 candidates; `vep` +8,685 candidates; `lud` +955 candidates.
  Experts changed `relevance` for some existing candidates
  (`vep` 268, `lud` 129). Two `olo` sentences received XML markup.
- In existing `words_*` rows, the `word` field was brought to its
  normalized form (lowercase, no apostrophes): `vep` 1,796, `olo` 998,
  `krl` 758, `lud` 67 rows.
- In `candidate_analyses_*`, all rows with an empty `gramset` now have
  `gramset_id = 0` (13,853 rows); the value `7` in `lud` no longer occurs.
- Updated the dictionary: `meanings` `krl` +3, `lud` +10; `examples`
  `vep` +83, `lud` +103, `olo` +27; `lemmas` `krl` +3, `lud` +7;
  `lemma_wordforms` `krl` +100, `lud` +287, `olo` −1, `vep` −3 links;
  `affix` was recalculated for some links. `id` numbering changed in
  `meanings_krl`, `meanings_lud`, `examples_lud`, and `examples_olo`.
- In `dict/synonyms/synset_candidates_olo.csv.zst`, 19 synset–candidate
  pairs were replaced; the total number of rows is unchanged (523).
- Documentation: added `corpus/reference/README*`; the corpus README now
  describes the new columns and the link between words and XML; updated
  the figures for candidates without `gramset` and for `gramset_id`
  in dictionary morphology.
- Samples: added the new columns to `corpus/samples/texts_krl.csv`
  and `words_krl.csv`.
- Source: VepKar / Dictorpus database export.

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
