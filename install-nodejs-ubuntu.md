Для Ubuntu з CloudPanel найкраще встановити Node.js 22 через офіційний репозиторій NodeSource. Це дозволить отримувати оновлення через `apt` і уникнути проблем зі старими версіями з репозиторіїв Ubuntu.

### 1. Перевірте поточну версію Node.js

```bash
node -v
npm -v
```

Якщо Node.js вже встановлений, подивіться версію. За потреби стару версію можна видалити.

---

### 2. Додайте репозиторій Node.js 22

Оновіть систему:

```bash
apt update
apt upgrade -y
```

Встановіть необхідні пакети:

```bash
apt install -y curl ca-certificates gnupg
```

Додайте репозиторій NodeSource:

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
```

---

### 3. Встановіть Node.js

```bash
apt install -y nodejs
```

Перевірте:

```bash
node -v
npm -v
```

Очікувано:

```text
v22.x.x
10.x.x
```

---

### 4. Перевірте шлях до Node.js

Supervisor краще працює з абсолютними шляхами:

```bash
which node
which npm
```

Зазвичай:

```text
/usr/bin/node
/usr/bin/npm
```

---

### 5. Встановлення залежностей і запуск застосунку

```bash
npm install
npm run build
npm run start
```

---

