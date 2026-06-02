На Ubuntu 22.04 Python 3.12 **нема в стандартних репозиторіях**, тому ставиться через PPA (найправильніший спосіб) або збірку з вихідників.

Я покажу нормальний production-варіант.

---

# ✅ Варіант 1 (рекомендовано): через deadsnakes PPA

## 1. Оновити систему

```bash id="u1k9xq"
sudo apt update && sudo apt upgrade -y
```

---

## 2. Встановити залежності

```bash id="k3m9qp"
sudo apt install -y software-properties-common
```

---

## 3. Додати PPA

```bash id="r8p2aa"
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
```

---

## 4. Встановити Python 3.12

```bash id="z9d3lm"
sudo apt install -y python3.12
```

---

## 5. Перевірка

```bash id="x2q8vd"
python3.12 --version
```

має бути:

```text id="okv9aa"
Python 3.12.x
```

---

# ⚙️ Додатково (дуже важливо)

## pip для Python 3.12

```bash id="m2v8ps"
sudo apt install -y python3.12 python3.12-venv
```

Потім:

```bash id="t7n1cw"
python3.12 -m ensurepip --upgrade
```

або:

```bash id="p9v4sk"
curl -sS https://bootstrap.pypa.io/get-pip.py | python3.12
```

---

# 🧪 Створення venv (рекомендовано для проєктів)

```bash id="v5k2aa"
python3.12 -m venv venv
source venv/bin/activate
```

Перевірка:

```bash
python3.12 -m pip --version
```

---

