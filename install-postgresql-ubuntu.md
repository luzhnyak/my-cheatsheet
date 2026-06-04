Ок, зробимо нормальну production-схему: **PostgreSQL + міграції (Alembic)** для Flask або FastAPI на твоєму VPS.

Я покажу універсальний варіант, який підходить для обох.

---

# 🐘 1. Встановлення PostgreSQL

```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
```

Перевірка:

```bash
sudo systemctl status postgresql
```

---

# 🔐 2. Створюємо базу і користувача

Заходимо в postgres:

```bash
sudo -u postgres psql
```

---

## Створити користувача:

```sql
CREATE USER api_user WITH PASSWORD 'strong_password_here';
```

---

## Створити БД:

```sql
CREATE DATABASE api_db;
```

але краще вказати одразу власника, щоб далі не було проблем

```sql
CREATE DATABASE api_db OWNER api_user;
```

---


## Дати доступ:

```sql
ALTER ROLE api_user SET client_encoding TO 'utf8';
ALTER ROLE api_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE api_user SET timezone TO 'UTC';

GRANT ALL PRIVILEGES ON DATABASE api_db TO api_user;
```

Вихід:

```sql
\q
```

---

# 🧩 3. Встановлюємо драйвери Python

У venv:

```bash
source /var/www/api/venv/bin/activate
pip install psycopg2-binary
```


# 🔒 4. Важлива безпека (обов’язково)

## Закрити/відкрити доступ до PostgreSQL ззовні

Перевір:

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

(версія може бути 14/15/16 — перевір через ls /etc/postgresql/)

закрити:

```text
listen_addresses = 'localhost'
```

відкрити:

```text
listen_addresses = '*'
```

---

## Обмежити доступ по IP в pg_hba.conf:

Відкриваємо:

```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

має бути:

```text
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    api_db          api_user        146.120.47.22/32        md5
```

пояснення:

```text
host      → TCP підключення
api_db    → база
api_user  → користувач
IP/32     → тільки один IP
md5       → парольна автентифікація
```

---

## перезапуск:

```bash
sudo systemctl restart postgresql
```

## Відкрити порт у firewall (UFW) або через CloudPanel:

```bash
sudo ufw allow from 146.120.47.22 to any port 5432
```