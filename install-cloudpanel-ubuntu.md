Якщо встановлювати CloudPanel на чистий VPS з Ubuntu, то найкраще одразу зробити це правильно. Тоді сервер буде не лише зручним для хостингу сайтів, а й значно безпечнішим.

---

# Підготовка сервера перед встановленням CloudPanel

## ✅ 1. Вимоги

Рекомендовано:

* Ubuntu 24.04 LTS або 22.04 LTS
* Мінімум 2 GB RAM
* 2 CPU
* SSD диск
* Статична IP-адреса

Перевір версію:

```bash
lsb_release -a
```

---

## ✅ 2. Підключення під root

CloudPanel **встановлюється тільки під root**.

```bash
sudo -i whoami
# має бути root
```

---

## ✅ 3. Оновлення системи

Після підключення по SSH:

```bash
apt update && apt upgrade -y
apt autoremove -y
reboot
```

Після перезавантаження знову підключись по SSH.

---

## ✅ 4. Коректне ім’я хоста (hostname)

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

## ✅ 5. Вільні порти

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

## ✅ 6. Swap (рекомендовано, якщо ≤2GB RAM)

```bash
free -h
```

Якщо немає swap:

```bash
fallocate -l 3G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

---

## ❌ 7. Що НЕ потрібно робити

* ❌ Не встановлюй nginx/apache вручну
* ❌ Не став MySQL
* ❌ Не налаштовуй UFW (CloudPanel сам керує)
* ❌ Не використовуй aaPanel паралельно

👉 Якщо це **свіжа Ubuntu** — ідеально.

---

# Встановлення CloudPanel

CloudPanel рекомендує встановлювати на чисту систему.

Для Ubuntu 24.04:

```bash
curl -sSL https://installer.cloudpanel.io/ce/v2/install.sh | sudo bash
```

Якщо не підхоплює базу даних, то:

```bash
curl -sSL https://installer.cloudpanel.io/ce/v2/install.sh -o install.sh
sudo DB_ENGINE=MARIADB_10.11 bash install.sh
```

Після завершення:

```bash
sudo cloudpanel
```

Побачиш дані для входу.

---

## 1. Відкрити CloudPanel

В браузері:

```text
https://IP_СЕРВЕРА:8443
```

або

```text
https://your-domain.com:8443
```

Створити адміністратора.

---

# Перші дії після встановлення

## 1. Оновити CloudPanel

Перевірити версію:

```bash
clpctl version
```

---

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

## Перевірити сервіси

```bash
systemctl status nginx
systemctl status mysql
systemctl status redis-server
```

---

# Безпека

## 1. Створення окремого користувача

Не працюй постійно під root.

Створити користувача:

```bash
adduser admin
```

Додати в sudo:

```bash
usermod -aG sudo admin
```

Перевірити:

```bash
su - admin
sudo whoami
```

Має повернути:

```text
root
```

---

## 2. Налаштування SSH-ключів

На локальному ПК:

```bash
ssh-keygen -t ed25519
```

Скопіювати ключ:

```bash
ssh-copy-id admin@IP_СЕРВЕРА
```

Перевірити вхід:

```bash
ssh admin@IP_СЕРВЕРА
```

---

## 3. Перенесення SSH-ключів від root до admin

```bash
sudo mkdir -p /home/admin/.ssh
sudo cp /root/.ssh/authorized_keys /home/admin/.ssh/
sudo chown -R admin:admin /home/admin/.ssh
sudo chmod 700 /home/admin/.ssh
sudo chmod 600 /home/admin/.ssh/authorized_keys
```

---

## 4. Захист SSH

Відредагувати:

```bash
sudo nano /etc/ssh/sshd_config
```

Змінити:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Перезапустити:

```bash
sudo systemctl restart ssh
```

### Змінити стандартний порт SSH 22:

```bash
sudo nano /etc/ssh/sshd_config
```

Знайдіть рядок:

```text
# Port 22
```

Розкоментуйте і змініть на потрібний порт (наприклад, 2222):

```text
Port 2222
```

Якщо використовується UFW — дозвольте новий порт

Якщо ssh.socket активний і слухає порт 22, ігноруючи sshd_config. Виконайте по порядку:

1. Зупиніть і вимкніть socket

```bash
bashsudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
```

2. Перезапустіть ssh без socket

```bash
bashsudo systemctl restart ssh
```

3. Перевірте результат

```bash
bashsudo ss -tlnp | grep sshd
```

⚠️ Не закривай поточну SSH-сесію поки не перевіриш, що нове підключення працює.

---

# Моніторинг сервера

Корисні команди:

CPU:

```bash
htop
```

RAM:

```bash
free -h
```

Диск:

```bash
df -h
```

Мережа:

```bash
ss -tulpn
```

---

## Регулярно перевіряти логи

```bash
journalctl -p err -b
```

```bash
tail -f /var/log/nginx/error.log
```

---
