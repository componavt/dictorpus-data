🌐 [Read this in English](CHANGELOG.md)

## v2026.10 — 2026-11-05

- Добавлено поле `meaning_n` — номер значения внутри словарной статьи —
  в словарные выгрузки `dict/meanings_*.csv.zst`.
- Добавлен каталог `dict/synonyms/` с данными о синонимических рядах
  прилагательных ливвиковского наречия карельского языка (`olo`):
  `synsets_*`, `synset_members_*`, `synset_candidates_*`, `collocations_*`.
  Выгрузки содержат ряды и их участников, отклонённые экспертом
  автоматически предложенные кандидаты и данные о сочетаемости.
- В CSV-выгрузках убраны необязательные кавычки вокруг полей.
- Обновлена документация словарных выгрузок; добавлено описание
  данных о синонимах и их связей со словарными значениями.
- Источник: VepKar / Dictorpus database export.

## v2026.09 — 2026-09-19

- Добавлены словарные выгрузки meanings_* и examples_* в папку /dict.
- Добавлены в папку /corpus:
  candidate_analyses_*, sentences_*, texts_*, words_*.
- Добавлены в папку /semantics файлы concept_categories_wdh.tsv и concepts.
- Источник: VepKar / Dictorpus database export.
