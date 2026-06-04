На Ubuntu 22.04 Python 3.12 **нема в стандартних репозиторіях**, тому ставиться через PPA (найправильніший спосіб) або збірку з вихідників.

---

# ✅ Через deadsnakes PPA

## 1. Оновити систему

```bash id="u1k9xq"
sudo apt update && sudo apt upgrade -y
```

---

## 2. Встановити залежності

```bash
sudo apt install -y software-properties-common
```

---

## 3. Додати PPA

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
```

---

## 4. Встановити Python 3.12

```bash
sudo apt install -y python3.12
```

---

## 5. Перевірка

```bash
python3.12 --version
```

має бути:

```text
Python 3.12.x
```

---

# ⚙️ Додатково (дуже важливо)

## pip для Python 3.12

```bash
sudo apt install -y python3.12 python3.12-venv
```

Потім:

```bash
python3.12 -m ensurepip --upgrade
```

або:

```bash
curl -sS https://bootstrap.pypa.io/get-pip.py | python3.12
```

---

# 🧪 Створення venv (рекомендовано для проєктів)

```bash
python3.12 -m venv venv
source venv/bin/activate
```

Перевірка:

```bash
python3.12 -m pip --version
```

