# Мої шпаргалки

Практичні інструкції з програмування, баз даних, адміністрування та AI-інструментів.

## Навігація

- [ChatGPT і Codex](#chatgpt-і-codex)
- [Сервер і Linux](#сервер-і-linux)
- [SSH, доступ і безпека](#ssh-доступ-і-безпека)
- [Git і GitHub](#git-і-github)
- [Python і Flask](#python-і-flask)
- [Бази даних і SQL](#бази-даних-і-sql)
- [Docker](#docker)

## ChatGPT і Codex

- [Корисні плагіни, MCP і скіли](./chatgpt-codex-plugins-skills.md) — добірка для Flask/FastAPI, WordPress, React Native/Expo та Python-ботів; підключення й приклади запитів.
- [Як створити власний скіл](./create-chatgpt-codex-skill.md) — Skill Creator, структура `SKILL.md`, локальне встановлення та перевірка.

## Сервер і Linux

- [Встановлення CloudPanel на Ubuntu](./install-cloudpanel-ubuntu.md) — підготовка VPS і встановлення панелі.
- [Додаткові нотатки про CloudPanel](./temp.md) — підготовка сервера та склад програмного забезпечення; матеріал частково перетинається з основною інструкцією.
- [Встановлення Node.js на Ubuntu](./install-nodejs-ubuntu.md).
- [Запуск Flask/FastAPI через Supervisor](./run-app-supervisor.md).
- [Пошук процесів, що навантажують CPU](./linux-proceses.md) — `top`, `htop`, `ps` та інші команди.

## SSH, доступ і безпека

- [Генерація та підключення SSH-ключів](./generate-ssh-keys.md) — доступ із ПК до VPS і з сервера до GitHub.
- [Спільний доступ двох користувачів до каталогу](./two-users-in-one-dir.md) — групи та права на статику.
- [Захист сервера](./security.md) — UFW і Fail2Ban.

## Git і GitHub

- [Видалення секрету з історії GitHub](./delete-secret-github.md).

## Python і Flask

- [Встановлення Python 3.12 на Ubuntu](./install-python-ubuntu.md).
- [Константи у шаблонах Flask](./flask-constants-to-templates.md) — передавання значень через context processor.

## Бази даних і SQL

- [Встановлення PostgreSQL на Ubuntu](./install-postgresql-ubuntu.md) — база, користувач і міграції для Flask/FastAPI.
- [Помилка PostgreSQL: must be owner of schema public](./must-be-owner-of-schema-public.md).
- [Заміна фрази в конкретному полі таблиці](./sql-replace-text.md) — `REPLACE`, попередній перегляд, транзакції та приклади для WordPress.

## Docker

- [Docker: контейнери та очищення](./docker-helper.md).

## Як користуватися

Обери тему або знайди потрібну команду через пошук GitHub у цьому репозиторії. Команди з правами адміністратора, масові зміни даних і очищення спочатку перевіряй на тестовому середовищі.

Версії програм у старих шпаргалках відповідають часу їх написання; перед встановленням звіряй підтримувані версії з документацією відповідного проєкту.
