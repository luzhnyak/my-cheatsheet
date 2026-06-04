Ок, зробимо через **Supervisor** — це простий і дуже надійний варіант для Flask/FastAPI на VPS під CloudPanel.

---

# 🧱 1. Встановлюємо Supervisor

```bash id="sup1"
sudo apt update
sudo apt install -y supervisor
```

Увімкнути сервіс:

```bash
sudo systemctl enable supervisor
sudo systemctl start supervisor
```

Перевірка:

```bash
sudo systemctl status supervisor
```

---

# 🐍 2. Налаштування venv

Установка venv:

```bash
sudo apt install python3.12-venv
``` 

Створення venv:

```bash
python3 -m venv venv
source venv/bin/activate
```

Встановлення модулів:

```bash
pip install -r requirements.txt
```

---

# 🐍 3. Приклад команд ручного запуску

Спочатку переконайся, що проєкт стартує вручну.

## Flask:

```bash
/home/goldfishnet/htdocs/goldfishnet.in.ua/venv/bin/gunicorn -w 2 -b 127.0.0.1:8091 main:app
```

## FastAPI:

```bash
/home/goldfishnet-api/htdocs/api.goldfishnet.in.ua/venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
```

👉 Якщо це не працює — Supervisor теж не запустить.

---

# ⚙️ 4. Створюємо Supervisor конфіг

```bash
sudo nano /etc/supervisor/conf.d/example.conf
```

## Flask:

```ini
[program:goldfishnet.in.ua]
directory=/home/goldfishnet/htdocs/goldfishnet.in.ua
command=/home/goldfishnet/htdocs/goldfishnet.in.ua/venv/bin/gunicorn main:app --bind 127.0.0.1:8090 --workers 2
user=goldfishnet
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
stdout_logfile=/home/goldfishnet/htdocs/goldfishnet.in.ua/supervisor.out.log
stderr_logfile=/home/goldfishnet/htdocs/goldfishnet.in.ua/supervisor.err.log
environment=PATH="/home/goldfishnet/htdocs/goldfishnet.in.ua/venv/bin"
```
---

## FastAPI:

```ini
[program:goldfishnet-api]
directory=/home/goldfishnet-api/htdocs/api.goldfishnet.in.ua
command=/home/goldfishnet-api/htdocs/api.goldfishnet.in.ua/venv/bin/gunicorn app.main:app -k uvicorn.workers.UvicornWorker --bind 127.0.0.1:8095 --workers 2
user=goldfishnet-api
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
stdout_logfile=/home/goldfishnet-api/htdocs/api.goldfishnet.in.ua/supervisor.out.log
stderr_logfile=/home/goldfishnet-api/htdocs/api.goldfishnet.in.ua/supervisor.err.log
environment=PYTHONUNBUFFERED=1,PYTHONPATH="/home/goldfishnet-api/htdocs/api.goldfishnet.in.ua"
```
---

## Django:

```ini
[program:itshnyk.in.ua]
directory=/home/itshnyk/htdocs/itshnyk.in.ua
command=/home/itshnyk/htdocs/itshnyk.in.ua/venv/bin/gunicorn app.wsgi:application --bind 127.0.0.1:8092 --workers 2
user=itshnyk
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
stdout_logfile=/home/itshnyk/htdocs/itshnyk.in.ua/supervisor.out.log
stderr_logfile=/home/itshnyk/htdocs/itshnyk.in.ua/supervisor.err.log
environment=PATH="/home/itshnyk/htdocs/itshnyk.in.ua/venv/bin"
```
---

## Next.js:

```ini
[program:zadachky.in.ua]
directory=/home/zadachky/htdocs/zadachky.in.ua
command=/usr/bin/npm run start
autostart=true
autorestart=true
stdout_logfile=/home/zadachky/htdocs/zadachky.in.ua/supervisor.out.log
stderr_logfile=/home/zadachky/htdocs/zadachky.in.ua/supervisor.err.log
user=zadachky
environment=NODE_ENV=production,PORT=3000
```
---

## Python:

```ini
[program:goldfishnet-bot]
directory=/home/goldfishnet/goldfishnet-bot
command=/home/goldfishnet/goldfishnet-bot/venv/bin/python start.py
user=goldfishnet
autostart=true
autorestart=true
stderr_logfile=/var/log/goldfishnet-bot.err.log
stdout_logfile=/var/log/goldfishnet-bot.out.log
environment=PYTHONUNBUFFERED=1
```
---

# ▶️ 5. Активуємо конфіг

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

Перевірка:

```bash
sudo supervisorctl status
```

Має бути щось типу:

```text
flask-api   RUNNING
```

або

```text
fastapi     RUNNING
```

---

# 🔁 6. Керування сервісом

```bash
sudo supervisorctl restart flask-api
sudo supervisorctl stop flask-api
sudo supervisorctl start flask-api
```

---

# 📜 7. Логи

## Flask:

```bash
tail -f /home/goldfishnet/htdocs/goldfishnet.in.ua/flask-api.out.log
tail -f /home/goldfishnet/htdocs/goldfishnet.in.ua/flask-api.err.log
```

---

# 📜 8. Подивись помилки запуску (ВАЖЛИВО)

```bash
sudo journalctl -u supervisor -n 100 --no-pager
```

---

# 🔒 9. Блокування порту (важливо)

Щоб API не був доступний напряму:

```bash
sudo ufw deny 8000
```
---

# 🔄 10. Повний рестарт

```bash
sudo systemctl daemon-reload
sudo systemctl restart supervisor
```