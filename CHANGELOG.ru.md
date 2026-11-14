🌐 [Read this in English](CHANGELOG.md)

## v2026.13 — 2026-11-14

- Добавлен каталог `corpus/reference/` с несжатыми справочниками
  подкорпусов, жанров и диалектов: `corpora.csv` (13 строк),
  `genres.csv` (75), `dialects.csv` (58). Они расшифровывают
  `corpus_id`, `genre_id` и `dialect_code` в `texts_*`. SQL `NULL`
  записывается как `\N`, как в `dict/morphology/` и `dict/grammar/`.
- **Изменение схемы:** в конец `corpus/texts_*.csv.zst` добавлен столбец
  `year_published` — год публикации из связанного источника
  (`sources.year`). Он не заменяет `year_recorded`.
- **Изменение схемы:** в конец `corpus/words_*.csv.zst` после `word`
  добавлены столбцы `w_id` и `s_id` — идентификаторы элементов
  `<w id>` и `<s id>` в исходной XML-разметке. Загрузчики, проверяющие
  точный состав столбцов, нужно обновить.
- Обновлён корпус: `krl` +5 текстов, +149 предложений, +1 709 слов,
  +9 275 кандидатов; `olo` +7 текстов, +177 предложений, +2 198 слов,
  +24 204 кандидата; `vep` +8 685 кандидатов; `lud` +955 кандидатов.
  Эксперты изменили `relevance` у части существующих кандидатов
  (`vep` 268, `lud` 129). У двух предложений `olo` появилась XML-разметка.
- В существующих строках `words_*` поле `word` приведено к нормализованной
  форме (нижний регистр, без апострофов): `vep` 1 796, `olo` 998,
  `krl` 758, `lud` 67 строк.
- В `candidate_analyses_*` у всех строк с пустым `gramset` теперь
  `gramset_id = 0` (13 853 строки); значения `7` в `lud` больше нет.
- Обновлён словарь: `meanings` `krl` +3, `lud` +10; `examples` `vep` +83,
  `lud` +103, `olo` +27; `lemmas` `krl` +3, `lud` +7; `lemma_wordforms`
  `krl` +100, `lud` +287, `olo` −1, `vep` −3 связей, у части связей
  пересчитан `affix`. В `meanings_krl`, `meanings_lud`, `examples_lud`
  и `examples_olo` изменилась нумерация `id`.
- В `dict/synonyms/synset_candidates_olo.csv.zst` заменены 19 пар
  «ряд — кандидат»; общее число строк прежнее (523).
- Документация: добавлены `corpus/reference/README*`; в README корпуса
  описаны новые столбцы и связь слов с XML; обновлены числовые сведения
  о кандидатах без `gramset` и о `gramset_id` в словарной морфологии.
- Примеры: в `corpus/samples/texts_krl.csv` и `words_krl.csv` добавлены
  новые столбцы.
- Источник: VepKar / Dictorpus database export.

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
