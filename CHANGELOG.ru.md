🌐 [Read this in English](CHANGELOG.md)

## v2026.11 — 2026-11-06

- Добавлен каталог `dict/morphology/`: `lemmas_*`, `lemma_wordforms_*`
  для всех четырёх языковых разновидностей, включая поле `affix`.
- Добавлен каталог `dict/grammar/` с несжатыми справочниками:
  `grams.csv`, `gramsets.csv`, `gramset_pos.csv`, `parts_of_speech.csv`.
- **Изменение схемы:** в `corpus/candidate_analyses_*.csv.zst` добавлен
  столбец `gramset_id` после `wordform_id`. Загрузчики, читающие столбцы
  по позиции, нужно обновить.
- В новых каталогах SQL `NULL` записывается как `\N`, пустая ячейка
  означает пустую строку. Формат остальных файлов не менялся.
- Обновлён корпус: `krl` +2 текста, +46 предложений, +492 слова,
  +466 кандидатов; `olo` +1 текст, +36 предложений, +544 слова,
  +674 кандидата. Существующие строки не изменились.
- В `dict/meanings_lud.csv.zst` изменились значения `id`; содержание то же.
- Документация: исправлено описание построения `gramset` (используется
  только `lgr`) и правила кавычек в CSV; описаны повторяющиеся `meaning_id`.
- Примеры: в `corpus/samples/candidate_analyses_krl.csv` добавлен
  `gramset_id`, в `dict/samples/examples_krl.csv` обновлены `id`,
  добавлен `dict/morphology/samples/`.
- Источник: VepKar / Dictorpus database export.

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
