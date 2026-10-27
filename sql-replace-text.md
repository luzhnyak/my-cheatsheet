# SQL: замінити фразу в конкретному полі таблиці

[← Усі шпаргалки](./README.md)

Задача: у таблиці `posts`, у текстовому полі `content`, замінити всі входження `стара фраза` на `нова фраза`.

## Швидкий шаблон

Для PostgreSQL, MySQL/MariaDB і SQLite:

```sql
UPDATE posts
SET content = REPLACE(content, 'стара фраза', 'нова фраза')
WHERE content LIKE '%стара фраза%';
```

Замінити `posts`, `content` та обидві фрази своїми значеннями. `REPLACE` змінює всі входження всередині кожного вибраного рядка; `WHERE` обмежує рядки. Це заміна підрядка, тому він може змінитися і всередині довшого слова.

## 1. Спочатку подивитися результат

Зробити резервну копію. Приклади припускають наявність первинного ключа `id`.

```sql
SELECT id,
       content AS old_content,
       REPLACE(content, 'стара фраза', 'нова фраза') AS new_content
FROM posts
WHERE content LIKE '%стара фраза%'
ORDER BY id
LIMIT 20;

SELECT COUNT(*) AS matching_rows
FROM posts
WHERE content LIKE '%стара фраза%';
```

`matching_rows` — кількість рядків, що відповідають фільтру, а не кількість входжень. `LIKE` може мати інші правила регістру, ніж `REPLACE`, тому частина відібраних рядків може залишитися незміненою.

## 2. Заміна з можливістю відкотити

PostgreSQL і SQLite:

```sql
BEGIN;

UPDATE posts
SET content = REPLACE(content, 'стара фраза', 'нова фраза')
WHERE content LIKE '%стара фраза%';

-- Подивитися дані в тій самій сесії до завершення транзакції.
SELECT id, content FROM posts ORDER BY id LIMIT 20;

-- Безпечна проба: скасувати зміни.
ROLLBACK;
```

Для збереження повторити транзакцію та після перевірки виконати **`COMMIT;` замість `ROLLBACK;`**. Не виконувати обидві команди поспіль і не відкривати іншу сесію для перевірки незавершеної транзакції.

MySQL/MariaDB: замість `BEGIN;` можна використати `START TRANSACTION;`. Відкат працює для транзакційних таблиць, наприклад InnoDB; для MyISAM `ROLLBACK` не відновить дані. Перевірити рушій таблиці через `SHOW TABLE STATUS LIKE 'posts';`. У графічному клієнті також перевірити режим autocommit.

## 3. Лише один запис або вибрані записи

Один запис:

```sql
UPDATE posts
SET content = REPLACE(content, 'стара фраза', 'нова фраза')
WHERE id = 123;
```

Кілька записів:

```sql
UPDATE posts
SET content = REPLACE(content, 'стара фраза', 'нова фраза')
WHERE id IN (123, 456, 789);
```

Якщо потрібно замінити **все значення поля**, а не фразу всередині:

```sql
UPDATE posts
SET content = 'нове значення'
WHERE id = 123;
```

## 4. Регістр, лапки та спеціальні символи

- `REPLACE` виконує буквальну заміну; для звичайного тексту розрізняє регістр. `Карась` і `карась` — різні фрагменти.
- У MySQL/MariaDB `LIKE` залежить від collation; у SQLite типовий `LIKE` ігнорує регістр ASCII, але не забезпечує повного Unicode-порівняння. У PostgreSQL `LIKE` зазвичай чутливий до регістру; поведінка також залежить від collation. Не використовувати `LIKE` як гарантію однакової поведінки всіх СУБД.
- `%` і `_` у `LIKE` — шаблони, а в аргументах `REPLACE` — звичайні символи.
- Апостроф у SQL-літералі подвоювати: `'об''єм води'`.
- Значення `NULL` не перетвориться на текст. Порожню шукану фразу не використовувати.

Щоб шукати буквально, без шаблонів `LIKE`, застосувати відповідний фільтр:

| СУБД | Фільтр для буквального підрядка |
| --- | --- |
| PostgreSQL | `WHERE strpos(content, 'стара фраза') > 0` |
| MySQL/MariaDB | `WHERE LOCATE(CAST('стара фраза' AS BINARY), CAST(content AS BINARY)) > 0` |
| SQLite | `WHERE instr(content, 'стара фраза') > 0` |

У програмному коді передавати фрази параметрами драйвера, без конкатенації SQL. Назви таблиць і полів вибирати з дозволеного списку: параметри значень не підставляють SQL-ідентифікатори.

## 5. Приклад для WordPress

У стандартній таблиці записів текст — `post_content`, ключ — `ID` (регістр імені ключа тут відрізняється від попередніх прикладів). Префікс `wp_` замінити фактичним:

```sql
SELECT ID, post_title,
       REPLACE(post_content, 'стара фраза', 'нова фраза') AS new_content
FROM wp_posts
WHERE post_type = 'post'
  AND post_content LIKE '%стара фраза%'
LIMIT 20;

UPDATE wp_posts
SET post_content = REPLACE(post_content, 'стара фраза', 'нова фраза')
WHERE post_type = 'post'
  AND post_content LIKE '%стара фраза%';
```

Для сторінок використати `post_type = 'page'`, для власного типу — його slug. SQL напряму обходить хуки WordPress і не оновлює `post_modified` та кеші автоматично.

**Серіалізовані PHP-дані** в `options`/`postmeta` не змінювати звичайним SQL `REPLACE`: можна пошкодити записану довжину рядків. Для таких даних використовувати WP-CLI. Із кореня WordPress у Bash:

```bash
wp db export before-replace.sql
wp db prefix

# wp_ нижче замінити префіксом із попередньої команди.
wp search-replace 'стара фраза' 'нова фраза' wp_postmeta --include-columns=meta_value --precise --dry-run

# Після перевірки звіту — виконання.
wp search-replace 'стара фраза' 'нова фраза' wp_postmeta --include-columns=meta_value --precise
```

Ці команди обробляють усі рядки поля `meta_value` вибраної таблиці, без фільтра за `post_id` чи `meta_key`. Для вужчого набору потрібен окремий скрипт через WordPress API. Зберігати SQL-дамп поза публічним web-каталогом.

## Джерела

- [PostgreSQL: рядкові функції](https://www.postgresql.org/docs/current/functions-string.html).
- [MySQL: REPLACE та інші рядкові функції](https://dev.mysql.com/doc/refman/8.4/en/string-functions.html).
- [SQLite: replace та instr](https://www.sqlite.org/lang_corefunc.html).
- [WordPress: wp search-replace](https://developer.wordpress.org/cli/commands/search-replace/).
