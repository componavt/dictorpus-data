🌐 [Читать на русском](README.ru.md)

# Synonym sets and collocations

This directory contains VepKar data on synonym sets (synsets)
of Livvi Karelian adjectives (`olo`): set membership, rejected
membership candidates, and members' collocational patterns.

Set members are individual dictionary meanings, not entire lemmas.
Lemma spellings, meaning numbers, and definitions are stored
in [`../meanings_olo.csv.zst`](../meanings_olo.csv.zst).
Its columns are described in the [dictionary exports README](../README.md).

## Files

Full datasets are stored as UTF-8 CSV files compressed with Zstandard.
Small uncompressed examples are available in [`samples/`](samples/).

| File | Contents |
|---|---|
| `synsets_olo.csv.zst` | Synonym sets and their dominant members. |
| `synset_members_olo.csv.zst` | Dictionary meanings included in the sets. |
| `synset_candidates_olo.csv.zst` | Automatically proposed meanings not included by the expert in the corresponding sets. |
| `collocations_olo.csv.zst` | Collocational patterns of meanings belonging to the sets. |

## Synonym sets

Schema of `synsets_olo.csv.zst`:

```csv
synset_id,lang,pos,dominant_meaning_id,russian_synonyms
```

| Column | Meaning |
|---|---|
| `synset_id` | Identifier of the VepKar synonym set. |
| `lang` | Language code; `olo` in the current export. |
| `pos` | Part of speech; `ADJ` in the current export. |
| `dominant_meaning_id` | Identifier of the dictionary meaning serving as the set's dominant member. May be empty. |
| `russian_synonyms` | Russian words and expressions characterizing the set. May be empty. |

## Set members

Schema of `synset_members_olo.csv.zst`:

```csv
synset_id,meaning_id,syntype_id
```

| Column | Meaning |
|---|---|
| `synset_id` | Reference to the synonym set. |
| `meaning_id` | Reference to a dictionary meaning in `../meanings_olo.csv.zst`. |
| `syntype_id` | Synonymy type identifier from the VepKar reference table. |

The synonymy type reference table is not included in the current export.

## Rejected candidates

Schema of `synset_candidates_olo.csv.zst`:

```csv
synset_id,meaning_id
```

| Column | Meaning |
|---|---|
| `synset_id` | Identifier of the set for which the meaning was proposed. |
| `meaning_id` | Identifier of the proposed dictionary meaning not included by the expert. |

Candidates are initially proposed automatically based on Russian
definitions, taking into account the set description and the
definitions of its members. The expert adds suitable meanings
to the set; meanings not included remain in the candidate list.

Thus, under the established expert review workflow, this file
represents rejected candidates, while `synset_members_olo.csv.zst`
contains accepted meanings.

Rejection applies to a specific `synset_id` and `meaning_id` pair:
it does not imply that the meaning cannot belong to another
synonym set.

## Collocations

Schema of `collocations_olo.csv.zst`:

```csv
collocation_id,meaning_id,collocate_meaning_id,concept_category_id,display_gloss
```

| Column | Meaning |
|---|---|
| `collocation_id` | Identifier of the collocation record. |
| `meaning_id` | Dictionary meaning of a synonym set member. |
| `collocate_meaning_id` | Dictionary meaning of the collocation partner. |
| `concept_category_id` | Semantic category of the collocation partner; corresponds to `category_id` in `../../semantics/concept_categories_wdh.tsv`. May be empty. |
| `display_gloss` | Russian gloss used when displaying the collocation. |

Both `meaning_id` and `collocate_meaning_id` reference
`meaning_id` in `../meanings_olo.csv.zst`.

## Links to dictionary meanings

To retrieve a meaning's lemma spelling and definition, join
`../meanings_olo.csv.zst` on `meaning_id`:

| Column in this directory | Column in `meanings_olo.csv.zst` |
|---|---|
| `synsets.dominant_meaning_id` | `meaning_id` |
| `synset_members.meaning_id` | `meaning_id` |
| `synset_candidates.meaning_id` | `meaning_id` |
| `collocations.meaning_id` | `meaning_id` |
| `collocations.collocate_meaning_id` | `meaning_id` |

The main columns for presenting a meaning are `lemma`, `meaning_n`,
`pos`, `meaning_ru`, and `meaning_en`. The `meaning_n` column gives
the meaning number within the dictionary entry; it is not a join key.
