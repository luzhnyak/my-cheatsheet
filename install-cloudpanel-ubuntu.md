Якщо встановлювати CloudPanel на чистий VPS з Ubuntu, то найкраще одразу зробити це правильно. Тоді сервер буде не лише зручним для хостингу сайтів, а й значно безпечнішим.

---

# 1. Підготовка сервера перед встановленням CloudPanel

## Вимоги

Рекомендовано:

* Ubuntu 24.04 LTS або 22.04 LTS
* Мінімум 2 GB RAM
* 2 CPU
* SSD диск
* Статична IP-адреса

## Оновлення системи

Після підключення по SSH:

```bash
apt update && apt upgrade -y
apt autoremove -y
reboot
```

Після перезавантаження знову підключись по SSH.

---

## Створення окремого користувача

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

## Налаштування SSH-ключів

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

## Перенесення SSH-ключів від root до admin

```bash
sudo mkdir -p /home/admin/.ssh
sudo cp /root/.ssh/authorized_keys /home/admin/.ssh/
sudo chown -R admin:admin /home/admin/.ssh
sudo chmod 700 /home/admin/.ssh
sudo chmod 600 /home/admin/.ssh/authorized_keys
```

---

# 2. Захист SSH

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

⚠️ Не закривай поточну SSH-сесію поки не перевіриш, що нове підключення працює.

---

# 3. Налаштування брандмауера

Ubuntu має UFW.

Дозволити:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Увімкнути:

```bash
sudo ufw enable
```

Перевірити:

```bash
sudo ufw status
```

---

# 4. Встановлення Fail2Ban

Захист від перебору паролів.

```bash
sudo apt install fail2ban -y
```

Запуск:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Перевірка:

```bash
sudo fail2ban-client status
```

---

# 5. Встановлення CloudPanel

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

# 6. Відкрити CloudPanel

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

# 7. Перші дії після встановлення

## Оновити CloudPanel

Перевірити версію:

```bash
clpctl version
```

---

## Перевірити сервіси

```bash
systemctl status nginx
systemctl status mysql
systemctl status redis-server
```

---

# 8. Захист панелі CloudPanel

Порт 8443 видно всьому інтернету.

Краще обмежити доступ.

Наприклад дозволити тільки свою IP:

```bash
sudo ufw allow from ВАША_IP to any port 8443
sudo ufw deny 8443
```

Тоді панель буде доступна лише тобі.

---

# 9. Встановлення SSL

CloudPanel автоматично отримує сертифікати через Let's Encrypt.

Для сайту:

1. Створити Site
2. Вказати домен
3. SSL/TLS
4. Issue Let's Encrypt Certificate

---

# 10. Налаштування резервних копій

CloudPanel підтримує:

* S3
* Backblaze B2
* Wasabi
* AWS

Рекомендую не зберігати бекапи на тому самому сервері.

Правильна схема:

```text
VPS
  ↓
Щоденний backup
  ↓
Backblaze B2
```

---

# 11. Автоматичні оновлення безпеки

Встановити:

```bash
sudo apt install unattended-upgrades -y
```

Увімкнути:

```bash
sudo dpkg-reconfigure unattended-upgrades
```

---

# 12. Моніторинг сервера

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

# 13. Що ще варто зробити для продакшн-сервера

### Вимкнути зайві сервіси

Подивитися:

```bash
systemctl list-unit-files --type=service
```

---

### Перевірити відкриті порти

```bash
sudo ss -tulpn
```

Зазвичай повинні бути відкриті:

```text
22
80
443
8443
```

---

### Регулярно перевіряти логи

```bash
journalctl -p err -b
```

```bash
tail -f /var/log/nginx/error.log
```

---

# Оптимальна схема для сервера з CloudPanel

```text
Ubuntu 24.04 LTS
│
├── SSH тільки по ключах
├── Root Login вимкнений
├── UFW
├── Fail2Ban
├── CloudPanel
├── Let's Encrypt
├── Автоматичні оновлення
├── Щоденні резервні копії
└── Моніторинг ресурсів
```

Для серверів з Flask, Node.js або React я б ще додатково налаштував захист PostgreSQL/MySQL від зовнішніх підключень та показав готову конфігурацію UFW і Fail2Ban для CloudPanel. Це особливо актуально, якщо ти плануєш працювати з базами даних віддалено.
