Ок, зробимо нормальну production-схему: **PostgreSQL + міграції (Alembic)** для Flask або FastAPI на твоєму VPS.

Я покажу універсальний варіант, який підходить для обох.

---

# 🐘 1. Встановлення PostgreSQL

```bash id="pg1"
sudo apt update
sudo apt install -y postgresql postgresql-contrib
```

Перевірка:

```bash id="pg2"
sudo systemctl status postgresql
```

---

# 🔐 2. Створюємо базу і користувача

Заходимо в postgres:

```bash id="pg3"
sudo -u postgres psql
```

---

## Створити БД:

```sql id="pg4"
CREATE DATABASE api_db;
```

---

## Створити користувача:

```sql id="pg5"
CREATE USER api_user WITH PASSWORD 'strong_password_here';
```

---

## Дати доступ:

```sql id="pg6"
ALTER ROLE api_user SET client_encoding TO 'utf8';
ALTER ROLE api_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE api_user SET timezone TO 'UTC';

GRANT ALL PRIVILEGES ON DATABASE api_db TO api_user;
```

Вихід:

```sql id="pg7"
\q
```

---

# 🧩 3. Встановлюємо драйвери Python

У venv:

```bash id="py1"
source /var/www/api/venv/bin/activate
pip install psycopg2-binary sqlalchemy
```


# 🔒 11. Важлива безпека (обов’язково)

## Закрити/відкрити доступ до PostgreSQL ззовні

Перевір:

```bash id="sec1"
sudo nano /etc/postgresql/14/main/postgresql.conf
```

закрити:

```text id="sec2"
listen_addresses = 'localhost'
```

відкрити:

```text id="sec2.1"
listen_addresses = 'localhost'
```

---

## Обмежитидоступ по IP в pg_hba.conf:

```bash id="sec3"
sudo nano /etc/postgresql/14/main/pg_hba.conf
```

має бути:

```text id="sec4"
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
```

---

## перезапуск:

```bash id="sec5"
sudo systemctl restart postgresql
```

## відкрити порт бази даних:

