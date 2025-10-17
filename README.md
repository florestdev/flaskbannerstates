# Flask-Banner

> Библиотека для жесткой блокировки нежелательных стран во Flask-приложениях.
> Быстро, просто и эффективно.

---

## 🔹 Описание

`Flask-Banner` позволяет мгновенно блокировать пользователей по географическому положению (IP-адресу).
Особенности:

* Поддержка **синхронного** (`Banner`) и **асинхронного** (`AsyncBanner`) режимов.
* Игнорирует **локальные IP** (127.0.0.1, 192.168.x.x, ::1 и т.д.).
* Поддержка **белого списка эндпоинтов** — запросы к этим функциям всегда пропускаются.
* Кеширование последних 200 IP для ускорения работы.
* Мгновенный отклик — блокировка с первого запроса.
* Можно отдавать как **JSON**, так и **HTML** ответы.

---

## 🔹 Установка

```bash
pip install flask
# Для асинхронного режима:
pip install "flask[async]" aiohttp
```

---

## 🔹 Синхронный Banner

```python
from flask import Flask
from flaskbanstates import Banner  # Импорт вашего класса Banner

app = Flask(__name__)

# Инициализация Banner
Banner(
    app,
    banned_countries=['UA', 'RU'],      # Список заблокированных стран
    return_response={"ok": False, "error": "Forbidden 403."},  # Ответ при бане
    type_response='json',                # 'json' или 'html'
    whitelisted_endpoint=['index']       # Эндпоинты, которые всегда доступны
)

@app.route('/')
def index():
    return 'Hi!'

@app.route('/about')
def about():
    return 'About page'

app.run('0.0.0.0', port=5000)
```

**Принцип работы:**

1. При каждом запросе Flask вызывает `before_request`.
2. Проверяет IP пользователя и кеш.
3. Если IP в заблокированной стране — возвращает 403.
4. Белые эндпоинты и локальные IP игнорируются.

---

## 🔹 Асинхронный Banner

```python
from flask import Flask
from flaskbanstates import AsyncBanner

app = Flask(__name__)

# Для работы AsyncBanner нужно Flask >=2.3 и запуск через ASGI сервер
AsyncBanner(
    app,
    banned_countries=['UA', 'RU'],
    return_response={"ok": False, "error": "Forbidden 403."},
    type_response='json',
    whitelisted_endpoint=['index']
)

@app.route('/')
async def index():
    return 'Hi async!'

# Для асинхронного запуска используйте ASGI сервер:
# hypercorn your_app:app
```

**Особенности AsyncBanner:**

* Асинхронно делает запрос к `ip-api` через `aiohttp`.
* Требует Flask с async-поддержкой (`pip install "flask[async]"`).
* Подходит для больших нагрузок и ASGI серверов (Hypercorn, Uvicorn).

---

## 🔹 Логика работы

* Локальные IP (`127.0.0.1`, `192.168.*.*`, `10.*.*.*`, `::1`) **не блокируются**.
* Повторные запросы одного и того же IP не вызывают повторных логов (чистый консольный вывод).
* Белый список эндпоинтов позволяет исключить отдельные функции из проверки.
* Кеш последних 200 IP ускоряет работу и снижает нагрузку на внешний API.
* При превышении лимита кеш очищается автоматически.

---

## 🔹 Ответ при бане

* **JSON (по умолчанию):**

```json
{
  "ok": false,
  "error": "Forbidden 403."
}
```

* **HTML (если указано `type_response='html'`):**
  Можно отдавать кастомную страницу банов.

---

## 🔹 Пример HTML страницы для банов

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Access Denied</title>
    <style>
        body {
            background: #f44336;
            color: white;
            text-align: center;
            font-family: sans-serif;
            padding: 50px;
        }
        h1 {
            font-size: 4em;
        }
        p {
            font-size: 1.5em;
        }
        a {
            color: #fff;
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <h1>🚫 Access Denied</h1>
    <p>Your country is blocked from accessing this site.</p>
    <p>If you think this is a mistake, <a href="mailto:support@example.com">contact support</a>.</p>
</body>
</html>
```

---

## 🔹 Быстрый старт

1. Установить Flask.
2. Инициализировать Banner или AsyncBanner в приложении.
3. Добавить свои страны в `banned_countries`.
4. Опционально добавить белые эндпоинты.
5. Готово — каждый запрос будет проверяться автоматически.

```python
from flask import Flask
from flaskbanstates import Banner

app = Flask(__name__)
Banner(app, banned_countries=['UA', 'RU'], whitelisted_endpoint=['index'])

@app.route('/')
def index():
    return 'Hi!'

app.run()
```

---

## 🔹 Лицензия

The unlicense — используйте на здоровье! 🚀
