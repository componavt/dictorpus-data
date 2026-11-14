🌐 [Читать на русском](README.ru.md)

# VepKar Morphological Disambiguation Dataset

This directory contains source data exported from the **VepKar** corpus for the project **Contextual Ranking of Morphological Analyses in VepKar**.

The project studies **morphological candidate ranking**:

```text
word in a sentence
        +
its candidate morphological analyses
        ↓
rank/select the correct wordform + gramset analysis
```

The dataset does **not** target word-sense disambiguation, lemma selection, or semantic annotation. The primary candidate identity is the pair:

```text
wordform_id + gramset
```

The numeric `gramset_id` links a candidate to the [`../dict/grammar/gramsets.csv`](../dict/grammar/README.md) reference table and to the [`../dict/morphology/`](../dict/morphology/README.md) dictionary morphology.

The values of `corpus_id`, `genre_id`, and `dialect_code` in text metadata are explained by the [`reference/`](reference/README.md) tables: `corpora.csv`, `genres.csv`, and `dialects.csv`.

## Storage format

The full VepKar source data are stored as individually compressed UTF-8 CSV files in Zstandard format:

```text
*.csv.zst
```

Each CSV file is compressed separately using Zstandard level `-19`.

For example:

```text
candidate_analyses_lud.csv.zst
sentences_vep.csv.zst
texts_krl.csv.zst
words_olo.csv.zst
```

The four language codes used in filenames are:

| Code | Language variety |
|---|---|
| `vep` | Vepsian |
| `krl` | Karelian Proper |
| `olo` | Livvi Karelian |
| `lud` | Ludic Karelian |

The repository also contains small uncompressed example files in [`samples/`](samples/). They are intended for GitHub browsing, documentation, quick inspection, and local debugging. The sample files are not a benchmark subset and must not be used for research evaluation.

## File overview

For every language variety, the dataset includes four file types:

```text
candidate_analyses_<lang>.csv.zst
sentences_<lang>.csv.zst
texts_<lang>.csv.zst
words_<lang>.csv.zst
```

The relations between the files are:

```text
texts
  └── sentences
        └── words
              └── candidate analyses
```

```text
texts_<lang>.csv
    text_id
       │
       ▼
sentences_<lang>.csv
    sentence_id, text_id
       │
       ▼
words_<lang>.csv
    word_id, sentence_id, word_number, word, w_id, s_id
       │
       ▼
candidate_analyses_<lang>.csv
    word_id, wordform_id, gramset_id, gramset, relevance
```

## `candidate_analyses_*.csv`

This is the central file for morphological candidate ranking. Each row represents one candidate morphological analysis associated with a corpus word.

Example:

```csv
word_id,wordform_id,gramset_id,gramset,relevance
3675349,8063,1,SG+NOM,1
3675349,8063,56,SG+ACC,1
3675350,2111182,8,SG+IN,2
```

| Column | Type | Description |
|---|---|---|
| `word_id` | integer | Identifier of a corpus word. It corresponds to `words.id` and joins this table to `words_*.csv`. |
| `wordform_id` | integer | Identifier of the dictionary wordform candidate. It corresponds to `text_wordform.wordform_id`. |
| `gramset_id` | integer | Identifier of the grammatical set. Join to `../dict/grammar/gramsets.csv`; see [below](#candidates-without-a-grammatical-set) for special values. |
| `gramset` | string or empty | Compact representation of the candidate grammatical feature set. It is built only from the `lgr` codes of grammemes and matches `gramsets.csv` for the same `gramset_id`. Examples: `SG+NOM`, `SG+IPFV.PST+1+IND+POS`, `INF`, `ACT+IPFV.PTCP`. |
| `relevance` | integer | Status of the morphological candidate in VepKar: `0` = rejected candidate; `1` = automatically proposed, not expert-selected candidate; `2` = candidate selected by an expert as correct. |

### Important rule: gold candidate

For morphological disambiguation, a candidate is considered expert-selected only when:

```text
relevance = 2
```

Do not use the condition `relevance > 1` for this dataset. Values greater than one have a different meaning in semantic/example-selection data and are not the morphological target defined here.

### Candidate groups

All rows with the same `word_id` form the candidate set for one corpus word.

```text
word_id = 3675349

candidate 1: wordform_id = 8063, gramset_id = 1,  gramset = SG+NOM
candidate 2: wordform_id = 8063, gramset_id = 56, gramset = SG+ACC
```

Derived fields such as candidate count, ambiguity status, gold label, train/dev/test split, and data-quality flags are intentionally not stored in the source export. They must be created by reproducible preparation scripts in a separate results directory.

### Candidates without a grammatical set

In 13,853 rows, `gramset` is empty, and all of them have `gramset_id = 0` (88 in `vep`, 487 in `krl`, 13,099 in `olo`, 179 in `lud`). The value `0` is not present in `gramsets.csv`, so an inner join with the reference table will lose such rows. They include rows with `relevance = 2`.

The triple `word_id + wordform_id + gramset_id` is unique in all files.

## `sentences_*.csv`

Each row represents one VepKar sentence.

Example:

```csv
sentence_id,text_id,sentence_xml,sentence_ru
41509,1566,"<s id=""1""><w id=""1"">Kouz</w> ... </s>","Когда я жила в Пелдоже ..."
```

| Column | Type | Description |
|---|---|---|
| `sentence_id` | integer | Identifier of the sentence. It joins this table to `words_*.csv`. |
| `text_id` | integer | Identifier of the VepKar text containing the sentence. It joins this table to `texts_*.csv`. |
| `sentence_xml` | string | Original sentence represented in VepKar XML markup. Word elements are encoded as `<w id="...">...</w>`. Punctuation remains in the XML sentence text. |
| `sentence_ru` | string or empty | Russian sentence translation aligned with the source sentence, when available. It is empty when no Russian parallel translation exists. |

### Sentence context

The primary context for the project is one sentence. The source word sequence can be recovered from `words_*.csv` by selecting all rows with the same `sentence_id` and sorting by `word_number`.

`sentence_xml` preserves the original sentence presentation, including punctuation and the original graphical form of words. `words.word`, in contrast, is the normalized form used by VepKar for matching and morphological processing.

## `texts_*.csv`

Each row describes metadata for one VepKar text.

Example:

```csv
text_id,corpus_id,dialect_code,genre_id,year_recorded,year_published
1566,1,lud-svt,11,1969,1978
1601,3,,,2018,
2042,1|4,krl-tlm,12,1960,2016
```

| Column | Type | Description |
|---|---|---|
| `text_id` | integer | Identifier of the VepKar text. It joins this table to `sentences_*.csv`. |
| `corpus_id` | string or empty | One or more identifiers in [`reference/corpora.csv`](reference/README.md#corporacsv), separated by `\|`. An empty value means that no corpus/subcorpus is available in the export. |
| `dialect_code` | string or empty | One or more codes in [`reference/dialects.csv`](reference/README.md#dialectscsv), separated by `\|`. An empty value means that no dialect code is available in the export. |
| `genre_id` | string or empty | One or more identifiers in [`reference/genres.csv`](reference/README.md#genrescsv), separated by `\|`. An empty value means that no genre annotation is available in the export. |
| `year_recorded` | integer or empty | Year in which the source material was recorded; exported from `events.date`. |
| `year_published` | integer or empty | Publication year recorded in the linked source; exported from `sources.year`. |

`year_recorded` and `year_published` are independent fields: neither is used to fill missing values in the other. Both are exported as stored, without type conversion; in the current export, all nonempty values are four-digit years. An empty cell indicates an unavailable value; these fields do not use the `\N` marker.

The rules for joining multi-valued fields to the reference tables are described in the [reference tables README](reference/README.md#joining-to-text-metadata).

Text metadata may be used as optional features in experiments, especially for evaluating whether dialect, genre, corpus membership, or recording year improve morphological candidate ranking.

## `words_*.csv`

Each row represents one word recognized by VepKar in one corpus sentence.

Example:

```csv
word_id,sentence_id,word_number,word,w_id,s_id
3675348,41509,1,kouz,1,1
3675349,41509,2,minä,2,1
3675350,41509,3,pеlduoižis,3,1
3675351,41509,4,elin,4,1
```

| Column | Type | Description |
|---|---|---|
| `word_id` | integer | Identifier of the corpus word. It corresponds to `words.id` and joins the word to `candidate_analyses_*.csv`. |
| `sentence_id` | integer | Identifier of the containing sentence. It joins the word to `sentences_*.csv`. |
| `word_number` | integer | Position of the word within the sentence. Sort words by this field to reconstruct the normalized word sequence. |
| `word` | string | Normalized VepKar word form used for search and morphological matching. Normalization includes lowercasing, removal of apostrophe-like softening marks, and context-dependent normalization of `w` according to vowel harmony. |
| `w_id` | integer | Identifier of the `<w id="…">` element in the text XML; exported from `words.w_id`. |
| `s_id` | integer | Identifier of the `<s id="…">` element in the text XML; exported from `words.s_id`. |

### Word normalization

The `word` column is intentionally not always identical to the graphical form visible inside `sentence_xml`.

```text
sentence_xml
    → original sentence representation, punctuation, original spelling

words.word
    → normalized form used for corpus matching and morphology
```

The normalized `word` form is the primary input form for morphological models in this project. Original surface forms remain recoverable from `sentence_xml` via `w_id` (see below).

### Linking to the Original XML

```text
word_id      -> words.id          database record
sentence_id  -> sentences.id      database record

w_id         -> <w id="...">      element in the text XML
s_id         -> <s id="...">      element in the text XML
```

`w_id` and `s_id` number elements within the XML of a single text, so they are not globally unique. `word_number` specifies the position within a sentence and is not a substitute for `w_id`.

To recover the original spelling of a word:

1. Locate the sentence by `sentence_id` in `sentences_*.csv`.
2. Check that the `<s id="…">` attribute in `sentence_xml` equals `s_id`.
3. Locate the `<w>` element in the sentence whose `id` equals `w_id`.
4. Read the element's text with an XML parser.

Do not look up words in the XML by the normalized `word`: the spelling may differ, and identical words may occur more than once in a sentence.

`w_id` and `s_id` are exported from the database without recalculation. In the current export, both fields are filled in every row. For words whose sentence is present in `sentences_*.csv`, a few matches fail: in `vep`, 124 words have no `<w>` with that `w_id`; in `olo`, 4 words have an `s_id` that differs from `<s id>`, and 6 words have no matching `w_id`. Such words are not removed from the export.

### Words without a sentence

Some rows reference a `sentence_id` that is absent from `sentences_*.csv`: 1,648 in `krl`, 449 in `olo`, and 173 in `vep`. An inner join will lose these words.

## CSV rules

All files use:

```text
Encoding:       UTF-8
Delimiter:      comma (,)
Quote character: double quote (")
Escaping:       embedded double quotes are doubled ("")
```

A value is quoted only when its content requires it, for example when the text contains a comma or a quotation mark. Therefore, read the files with a CSV parser rather than by splitting lines.
