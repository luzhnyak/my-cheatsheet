Добре, CloudPanel — нормальний вибір 👍
Опишу **що зробити ДО встановлення** і **який софт CloudPanel ставить автоматично**, щоб не наступити на типові граблі.

---

## 1️⃣ Що потрібно зробити перед встановленням CloudPanel

### ✅ 1. Чиста Ubuntu (дуже важливо)

CloudPanel **не любить**, коли на сервері вже є:

* Apache
* Nginx
* MySQL / MariaDB
* Docker
* aaPanel / Webuzo / ISPConfig

👉 Якщо це **свіжа Ubuntu** — ідеально.

**Підтримувані версії:**

* Ubuntu 22.04 LTS (рекомендовано)
* Ubuntu 20.04 LTS

Перевір:

```bash
lsb_release -a
```

---

### ✅ 2. Підключення під root

CloudPanel **встановлюється тільки під root**.

```bash
sudo -i
whoami
# має бути root
```

---

### ✅ 3. Оновлення системи

```bash
apt update && apt upgrade -y
reboot
```

Після перезавантаження знову зайди під root.

---

### ✅ 4. Коректне ім’я хоста (hostname)

Обов’язково має бути **FQDN**, не `localhost`.

❌ Погано:

```
localhost
ubuntu
```

✅ Добре:

```
server.yourdomain.com
```

Перевір:

```bash
hostnamectl
```

Якщо треба змінити:

```bash
hostnamectl set-hostname server.yourdomain.com
nano /etc/hosts
```

Додай:

```
127.0.0.1 server.yourdomain.com
```

---

### ✅ 5. Вільні порти

CloudPanel використовує:

| Порт | Для чого      |
| ---- | ------------- |
| 22   | SSH           |
| 80   | HTTP          |
| 443  | HTTPS         |
| 8443 | CloudPanel UI |

Перевір, щоб нічого не слухало:

```bash
ss -tulpn
```

---

### ✅ 6. Swap (рекомендовано, якщо ≤2GB RAM)

```bash
free -h
```

Якщо немає swap:

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

---

### ❌ Що НЕ потрібно робити

* ❌ Не встановлюй nginx/apache вручну
* ❌ Не став MySQL
* ❌ Не налаштовуй UFW (CloudPanel сам керує)
* ❌ Не використовуй aaPanel паралельно

---

## 2️⃣ Як встановити CloudPanel

Офіційний інсталятор:

```bash
curl -sSL https://installer.cloudpanel.io/ce/v2/install.sh | bash
```

Під час встановлення:

* обереш **Web server**
* обереш **PHP**
* обереш **Database**

Після завершення:

```
https://SERVER_IP:8443
```

---

## 3️⃣ Який софт CloudPanel встановлює АВТОМАТИЧНО

### 🌐 Web-сервер

* **Nginx** (власна оптимізована конфігурація)
* **Redis** (кеш)
* **Varnish** (опційно)

---

### 🐘 PHP

* PHP-FPM
* Одразу кілька версій:

  * 8.1
  * 8.2
  * 8.3 (залежить від версії CP)

Можна вибирати версію PHP **на рівні сайту**.

---

### 🗄 База даних

На вибір при встановленні:

* **MariaDB 10.6+** (найчастіше)
* або **MySQL 8.0**

---

### 🔐 Безпека

* Fail2Ban
* ModSecurity
* Auto SSL (Let’s Encrypt)
* Firewall rules
* SSH key management

---

### ⚙️ Інше

* Node.js (для Next.js / React / API)
* Composer
* Git
* Supervisor
* Cron
* FTP (pure-ftpd)
* Postfix (для системної пошти)

---

## 4️⃣ Що CloudPanel НЕ встановлює

* Docker
* Kubernetes
* cPanel-сміття
* зайві панелі/демони

---

## 5️⃣ Чи підійде тобі CloudPanel (з твого досвіду)

З огляду на те, що ти:

* деплоїш **Flask / FastAPI**
* працюєш з **Next.js**
* мав проблеми з aaPanel і майнерами

👉 **CloudPanel значно безпечніший і чистіший**, ніж aaPanel або Webuzo.

---

## 6️⃣ Хочеш — можу:

* показати **як задеплоїти Flask / FastAPI**
* показати **Next.js + Node.js**
* порадити **оптимальні налаштування безпеки**
* розповісти, **як ізолювати сайти**, щоб ніхто нікого не зламав

Просто скажи, що саме плануєш хостити 👌
