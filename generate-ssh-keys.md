Інструкція для генерації і підключння ssh ключів для доступу ДО СЕРВЕРА і для доступу СЕРВЕРА ДО GITHUB.

---

# 🔐 КЛЮЧ №1 — доступ **З ТВОГО ПК → VPS сервер**

**(ти підключаєшся до сервера через SSH / PuTTY / Terminal)**

## 1️⃣ Генерація ключа НА ТВОЄМУ ПК

### ▶️ Linux / macOS / Git Bash (Windows)

```bash
ssh-keygen -t ed25519 -f ~/.ssh/vps_server_access -C "vps-server-access"
```

✔ Назва ключів:

```text
~/.ssh/vps_server_access
~/.ssh/vps_server_access.pub
```

❗ Пароль — **рекомендовано задати**

---

## 2️⃣ Додаємо ключ на сервер

```bash
ssh-copy-id -i ~/.ssh/vps_server_access.pub user@SERVER_IP
```

або вручну:

```bash
cat ~/.ssh/vps_server_access.pub
```

На сервері:

```bash
nano ~/.ssh/authorized_keys
```

---

## 3️⃣ Підключення до сервера

```bash
ssh -i ~/.ssh/vps_server_access user@SERVER_IP
```

---

## 4️⃣ (Опціонально) Заборонити парольний логін

```bash
sudo nano /etc/ssh/sshd_config
```

```text
PasswordAuthentication no
PermitRootLogin no
```

```bash
sudo systemctl restart ssh
```

---

# 🔐 КЛЮЧ №2 — доступ **СЕРВЕРА → приватний GitHub репозиторій**

**(сервер робить git clone / pull)**

❗ Генерується **НА СЕРВЕРІ**, **під користувачем сайту**

---

## 1️⃣ Переключаємося на користувача сайту

```bash
su - zadachky
```

---

## 2️⃣ Генеруємо ключ для GitHub

```bash
ssh-keygen -t ed25519 -f ~/.ssh/github_repo_deploy -C "github-repo-deploy"
```

✔ Файли:

```text
~/.ssh/github_repo_deploy
~/.ssh/github_repo_deploy.pub
```

❗ **БЕЗ пароля** (Enter → Enter)

---

## 3️⃣ Додаємо GitHub у known_hosts

```bash
ssh-keyscan github.com >> ~/.ssh/known_hosts
```

---

## 4️⃣ Додаємо ключ у GitHub

**GitHub → Repo → Settings → Deploy keys → Add key**

* **Title:** `zadachky-next-deploy`
* **Key:** вміст `github_repo_deploy.pub`
* ✅ **Allow write access**

---

## 5️⃣ Перевірка доступу

```bash
ssh -i ~/.ssh/github_repo_deploy -T git@github.com
```

✔ Очікувано:

```
Hi USERNAME! You've successfully authenticated...
```

---

## 6️⃣ Вказуємо Git використовувати правильний ключ

```bash
nano ~/.ssh/config
```

```text
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github_repo_deploy
  IdentitiesOnly yes  
```

```bash
chmod 600 ~/.ssh/config
```

### Якщо потрібно доступ до двох репозиторіїв (відповідно є два ключі), то

```text
# Репозиторій 1
Host github-repo1
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_repo_deploy1
    IdentitiesOnly yes

# Репозиторій 2
Host github-repo2
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_repo_deploy2
    IdentitiesOnly yes
```
---

## 7️⃣ Клонуємо репозиторій

```bash
git clone git@github.com:luzhnyak/zadachky-next.git
```
### Якщо потрібно доступ до двох репозиторіїв, то

```bash
# Для першого репозиторію
git clone git@github-repo1:username/repo1.git

# Для другого репозиторію
git clone git@github-repo2:username/repo2.git
```

---

# 🔁 КЛЮЧ №3 (опціонально) — GitHub Actions → сервер

**(для деплою через workflow)**

### Генерується **НА ТВОЄМУ ПК**

```bash
ssh-keygen -t ed25519 -f ~/.ssh/github_actions_vps -C "github-actions-vps"
```

---

### Публічний ключ → сервер

```bash
cat ~/.ssh/github_actions_vps.pub
```

Додати в:

```bash
/home/zadachky/.ssh/authorized_keys
```

---

### Приватний ключ → GitHub Secrets

```text
SERVER_IP
SERVER_USER
SSH_PRIVATE_KEY ← github_actions_vps
```

---

# 🧠 КОРОТКА ТАБЛИЦЯ (щоб не плутатись)

| Для чого             | Де  | Назва ключа          |
| -------------------- | --- | -------------------- |
| ПК → VPS             | ПК  | `vps_server_access`  |
| VPS → GitHub         | VPS | `github_repo_deploy` |
| GitHub Actions → VPS | ПК  | `github_actions_vps` |

