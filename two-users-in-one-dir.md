Тут усе вирішується **правильними правами доступу в Linux**, і робиться це **без костилів** 👍
Поясню по кроках і дам **2 правильні варіанти** (обери, що більше подобається).

---

## 📌 Вхідні дані (як я розумію)

* користувач **goldfishnet**

  ```
  /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
  ```

* користувач **goldfishnet-api**

  * FastAPI / Flask
  * nginx / gunicorn запускається від `goldfishnet-api`

* потрібно **читати статику**, не писати

---

# ✅ РЕКОМЕНДОВАНИЙ ВАРІАНТ — через GROUP

### 🎯 Ідея

> **Одна спільна група → контрольований доступ → без небезпеки**

---

## 1️⃣ Створюємо групу

```bash
sudo groupadd goldfishnet-static
```

---

## 2️⃣ Додаємо обох користувачів у групу

```bash
sudo usermod -aG goldfishnet-static goldfishnet
sudo usermod -aG goldfishnet-static goldfishnet-api
```

⚠️ **Важливо:** перелогінитись або:

```bash
newgrp goldfishnet-static
```

---

## 3️⃣ Міняємо групу папки static

```bash
sudo chgrp -R goldfishnet-static /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
```

---

## 4️⃣ Виставляємо правильні права

```bash
sudo chmod -R 750 /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
```

👉 означає:

* `rwx` — власник (goldfishnet)
* `r-x` — група (goldfishnet-api)
* `---` — всі інші

---

## 5️⃣ (ДУЖЕ ВАЖЛИВО) setgid для нових файлів

Щоб **всі нові файли автоматично мали правильну групу**:

```bash
sudo find /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static -type d -exec chmod g+s {} \;
```

---

## ✅ Результат

* goldfishnet-api **може читати статику**
* не може її ламати
* без `777`
* без symlink-хаосу
* без security-проблем

---

# ⚠️ ВАРІАНТ 2 — через ACL (якщо не хочеш групи)

### ✔ Плюс

* дуже гнучко

### ❌ Мінус

* складніше підтримувати

---

## Надати доступ лише на читання

```bash
sudo setfacl -R -m u:goldfishnet-api:rx \
/home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
```

### Для нових файлів:

```bash
sudo setfacl -R -d -m u:goldfishnet-api:rx \
/home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
```

Перевірка:

```bash
getfacl /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
```

---

# ❌ ЩО НЕ ТРЕБА РОБИТИ

❌ `chmod 777`
❌ запускати gunicorn від root
❌ копіювати статику між проєктами
❌ symlink без прав (nginx може впасти)

---

# 🧠 Що Я РЕКОМЕНДУ

👉 **Group-підхід** — це:

* production-ready
* правильно з точки зору Linux
* ідеально для CloudPanel + nginx

---

Якщо хочеш — можу:

* перевірити **nginx user**
* показати **як nginx краще віддавати цю статику**
* запропонувати **best practice для media/static між сервісами**

Просто скажи 👌


Все ок, ти якраз натрапив на **типову Linux-пастку з правами на батьківські каталоги** 👍
Тут **статика може мати права**, але **один із каталогів вище їх блокує**.

Пояснюю чітко і по діагностиці.

---

## ❗ Чому так відбувається

У Linux, щоб **зайти (`cd`) у каталог**, потрібне право **`x` (execute)** **на КОЖЕН каталог у шляху**, а не тільки на кінцевий.

Тобто для цього шляху:

```
/home/goldfishnet/htdocs/www.goldfishnet.in.ua/static/img/places/resize/1
```

потрібно `x` на:

```
/home
/home/goldfishnet
/home/goldfishnet/htdocs
/home/goldfishnet/htdocs/www.goldfishnet.in.ua
/home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
/img
/places
/resize
/1
```

👉 **Десь вище немає `x` для групи**

---

## 1️⃣ Перевіряємо, де саме блокує

Запусти **під goldfishnet-api**:

```bash
namei -l /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static/img/places/resize/1
```

Приклад проблемного виводу:

```
drwx------ goldfishnet goldfishnet /home/goldfishnet
```

Ось тут і буде стоп 🚫

---

## 2️⃣ Мінімально правильні права (рекомендую)

### `/home/goldfishnet` — має бути доступний для traversal

```bash
sudo chmod 750 /home/goldfishnet
sudo chgrp goldfishnet-static /home/goldfishnet
```

⚠️ **НЕ 755**, тільки `x` для групи, без читання home

---

### `/home/goldfishnet/htdocs`

```bash
sudo chgrp goldfishnet-static /home/goldfishnet/htdocs
sudo chmod 750 /home/goldfishnet/htdocs
```

---

### `/home/goldfishnet/htdocs/www.goldfishnet.in.ua`

```bash
sudo chgrp goldfishnet-static /home/goldfishnet/htdocs/www.goldfishnet.in.ua
sudo chmod 750 /home/goldfishnet/htdocs/www.goldfishnet.in.ua
```

---

### Статика (вже майже напевно ок, але на всяк)

```bash
sudo chgrp -R goldfishnet-static /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
sudo chmod -R 750 /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
sudo find /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static -type d -exec chmod g+s {} \;
```

---

## 3️⃣ Перевірка

```bash
sudo -u goldfishnet-api ls /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static
sudo -u goldfishnet-api cd /home/goldfishnet/htdocs/www.goldfishnet.in.ua/static/img/places/resize/1
```

👉 **Permission denied бути не повинно**

---

## 🧠 Важливе пояснення (чому так правильно)

* `x` на каталог = **можу зайти / пройти**
* `r` на каталог = **можу бачити список файлів**
* ми даємо **тільки те, що потрібно**
* це **production-safe**
* nginx + gunicorn працюють стабільно

---

## ❌ Чого не роби

❌ `chmod 755 /home/goldfishnet`
❌ `chmod 777`
❌ запуск nginx або gunicorn від root
❌ symlink без прав (CloudPanel може їх різати)

---

## Якщо хочеш — зробимо ідеально

Я можу:

* перевірити **user nginx** (`ps aux | grep nginx`)
* сказати **чи треба окрема група для nginx**
* підказати **як CloudPanel краще віддає статику**
* показати **ACL vs group на практиці**

Скинь `namei -l` — і я скажу точно, де вузьке місце 👌
