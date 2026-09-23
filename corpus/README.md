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
    word_id, sentence_id, word_number, word
       │
       ▼
candidate_analyses_<lang>.csv
    word_id, wordform_id, gramset, relevance
```

## `candidate_analyses_*.csv`

This is the central file for morphological candidate ranking. Each row represents one candidate morphological analysis associated with a corpus word.

Example:

```csv
word_id,wordform_id,gramset,relevance
3675349,8063,SG+NOM,1
3675349,8063,SG+ACC,1
3675350,2111182,SG+IN,1
```

| Column | Type | Description |
|---|---|---|
| `word_id` | integer | Identifier of a corpus word. It corresponds to `words.id` and joins this table to `words_*.csv`. |
| `wordform_id` | integer | Identifier of the dictionary wordform candidate. It corresponds to `text_wordform.wordform_id`. |
| `gramset` | string | Compact representation of the candidate grammatical feature set. It is constructed from VepKar grammatical attributes, using `lgr` codes with `name_short_en` as a fallback when needed. Examples: `SG+NOM`, `SG+IPFV.PST+1+IND+POS`, `INF`, `ACT+IPFV.PTCP`. |
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

candidate 1: wordform_id = 8063, gramset = SG+NOM
candidate 2: wordform_id = 8063, gramset = SG+ACC
```

Derived fields such as candidate count, ambiguity status, gold label, train/dev/test split, and data-quality flags are intentionally not stored in the source export. They must be created by reproducible preparation scripts in a separate results directory.

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
text_id,corpus_id,dialect_code,genre_id,year_recorded
1566,1,lud-svt,11,1969
1601,3,,,2018
```

| Column | Type | Description |
|---|---|---|
| `text_id` | integer | Identifier of the VepKar text. It joins this table to `sentences_*.csv`. |
| `corpus_id` | integer | Identifier of the corpus or subcorpus associated with the text. |
| `dialect_code` | string or empty | VepKar dialect code associated with the text. Multiple codes, if present, are separated by `|`. An empty value means that no dialect code is available in the export. |
| `genre_id` | integer or empty | Identifier of the genre associated with the text. Multiple identifiers, if present, are separated by `|`. An empty value means that no genre identifier is available in the export. |
| `year_recorded` | integer or empty | Year in which the source material was recorded, when known. It is empty when the recording year is unavailable. |

Text metadata may be used as optional features in experiments, especially for evaluating whether dialect, genre, corpus membership, or recording year improve morphological candidate ranking.

## `words_*.csv`

Each row represents one word recognized by VepKar in one corpus sentence.

Example:

```csv
word_id,sentence_id,word_number,word
3675348,41509,1,kouz
3675349,41509,2,minä
3675350,41509,3,pеlduoižis
3675351,41509,4,elin
```

| Column | Type | Description |
|---|---|---|
| `word_id` | integer | Identifier of the corpus word. It corresponds to `words.id` and joins the word to `candidate_analyses_*.csv`. |
| `sentence_id` | integer | Identifier of the containing sentence. It joins the word to `sentences_*.csv`. |
| `word_number` | integer | Position of the word within the sentence. Sort words by this field to reconstruct the normalized word sequence. |
| `word` | string | Normalized VepKar word form used for search and morphological matching. Normalization includes lowercasing, removal of apostrophe-like softening marks, and context-dependent normalization of `w` according to vowel harmony. |

### Word normalization

The `word` column is intentionally not always identical to the graphical form visible inside `sentence_xml`.

```text
sentence_xml
    → original sentence representation, punctuation, original spelling

words.word
    → normalized form used for corpus matching and morphology
```

The normalized `word` form is the primary input form for morphological models in this project. Original surface forms remain recoverable from `sentence_xml` when a future experiment requires them.

## CSV rules

All files use:

```text
Encoding:       UTF-8
Delimiter:      comma (,)
Quote character: double quote (")
Escaping:       embedded double quotes are doubled ("")
```

Textual values such as XML sentences and Russian translations are quoted. Therefore, commas, quotation marks, and punctuation inside sentence text do not change the CSV column structure.
