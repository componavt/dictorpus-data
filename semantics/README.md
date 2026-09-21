🌐 [Читать на русском](README.ru.md)

# Semantic resources

This directory contains semantic metadata used to relate VepKar dictionary meanings to concept records and WordNet Domain Hierarchy (WDH) labels.

```text
../dict/meanings_<lang>.csv.zst
      │                 │
      │ concept_id      │ category_id
      ▼                 ▼
concepts.csv      concept_categories_wdh.tsv
      │                 │
      └──── category_id ┴────► wdh
```

The files are small, uncompressed UTF-8 tab-separated tables. They are reference data, not generated model output.

## Files

| File | Purpose |
|---|---|
| `concepts.csv` | Concept catalogue with Russian and English labels and definitions. |
| `concept_categories_wdh.tsv` | Mapping from semantic category identifiers to WordNet Domain Hierarchy labels. |

## `concepts.csv`

Schema:

```text
category_id	pos	concept_id	concept_ru	concept_en	definition_ru	definition_en
```

| Column | Meaning |
|---|---|
| `category_id` | Semantic category identifier, such as `A11`. Join to `concept_categories_wdh.tsv.category_id`. |
| `pos` | Part of speech associated with the concept. |
| `concept_id` | Identifier of the semantic concept. Join from `../dict/meanings_<lang>.csv.zst.concept_id`. |
| `concept_ru` | Russian concept name. |
| `concept_en` | English concept name. |
| `definition_ru` | Russian explanatory definition of the concept. |
| `definition_en` | English explanatory definition of the concept. |

Example:

```text
A11	NOUN	1	земля, суша	land, ground	часть земной коры, выступающая над уровнем Мирового океана...	the part of Earth which is not covered by oceans or other bodies of water
```

A concept is broader than any one dictionary meaning. Its labels and definitions are semantic reference data; they must not automatically overwrite `meaning_ru` or `meaning_en` in dictionary exports.

## `concept_categories_wdh.tsv`

Schema:

```text
category_id	name_ru	name_en	wdh
```

| Column | Meaning |
|---|---|
| `category_id` | Semantic category identifier, such as `A11`. |
| `name_ru` | Russian category name. |
| `name_en` | English category name. |
| `wdh` | One or more comma-separated WordNet Domain Hierarchy labels. |

Example:

```text
A11	Небо; небесные тела	Sky; celestial bodies	astronomy
A21	Воды; моря, реки	Waters; seas, rivers	geography, oceanography
```

The `wdh` field may contain several labels separated by commas. Consumers that need atomic labels should split on commas, strip surrounding whitespace, and preserve the original source table unchanged.

## Join patterns

### Dictionary meanings to concepts

```python
meanings.merge(
    concepts,
    on="concept_id",
    how="left",
    validate="many_to_one",
)
```

### Concepts or meanings to WDH categories

```python
concepts.merge(
    categories_wdh,
    on="category_id",
    how="left",
    validate="many_to_one",
)
```

For dictionary meanings, a direct join is also possible:

```python
meanings.merge(
    categories_wdh,
    on="category_id",
    how="left",
    validate="many_to_one",
)
```

These joins enrich records with semantic evidence. They do not prove that a particular dictionary meaning has an English translation or a WordNet synset.

## Missing links

In dictionary exports, `concept_id` and `category_id` may be blank. A blank field means that the source export has no corresponding semantic link. Do not invent identifiers, infer a WDH label from a lemma spelling, or replace missing values with a generic domain such as `factotum`.
