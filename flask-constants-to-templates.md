Щоб зробити константи з файлу `config.py` доступними у всіх шаблонах Flask, можна скористатися контекстними процесорами (`context processor`). Це спеціальна функція у Flask, яка додає дані до контексту кожного шаблону.

### Кроки:

1. **У файлі `config.py` оголосіть ваші константи:**

   ```python
   # config.py
   DOMEN = "https://example.com"
   SITE_NAME = "My Website"
   ```

2. **Імпортуйте ці константи у ваш додаток Flask.**
   У файлі `app.py` або `main.py` підключіть ваш конфігураційний файл:

   ```python
   from flask import Flask
   from config import DOMEN, SITE_NAME

   app = Flask(__name__)
   ```

3. **Створіть контекстний процесор.**
   У Flask можна додати функцію, яка додає змінні до контексту шаблонів:

   ```python
   @app.context_processor
   def inject_constants():
       return {
           'DOMEN': DOMEN,
           'SITE_NAME': SITE_NAME,
       }
   ```

   Ця функція автоматично додає змінні `DOMEN` і `SITE_NAME` до всіх шаблонів.

4. **Використовуйте ці константи в шаблонах.**
   Тепер ви можете отримати доступ до цих констант у будь-якому шаблоні:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <title>{{ SITE_NAME }}</title>
   </head>
   <body>
       <p>Visit us at: <a href="{{ DOMEN }}">{{ DOMEN }}</a></p>
   </body>
   </html>
   ```

### Результат

* Константи `DOMEN` і `SITE_NAME` будуть доступні у всіх шаблонах автоматично.
* Вам не потрібно передавати їх вручну через функції рендерингу шаблонів (`render_template`).

Це зручно для глобальних змінних, які мають використовуватися у багатьох місцях вашого додатка.
