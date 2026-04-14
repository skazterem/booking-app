# Деплой Telegram-бота на Railway.app

## Почему Railway?

| Преимущество | Описание |
|---|---|
| **Без банковской карты** | Не нужна верификация |
| **$5 кредит/месяц** | Хватает для бота 24/7 |
| **Python 24/7** | Long polling работает |
| **Автодеплой с GitHub** | Push → автоматический рестарт |

---

## Пошаговая инструкция

### Шаг 1. Зарегистрируйтесь на Railway.app

1. Перейдите на [railway.app](https://railway.app)
2. Нажмите **Login** → **Sign in with GitHub**
3. Авторизуйтесь под аккаунтом, где находится репозиторий

### Шаг 2. Создайте новый проект

1. Нажмите **New Project**
2. Выберите **Deploy from GitHub repo**
3. Выберите репозиторий: `skazterem/booking-app`
4. Railway автоматически определит Python-проект по `requirements.txt` и `Procfile`

### Шаг 3. Добавьте переменные окружения

1. В панели проекта нажмите на сервис → **Variables**
2. Нажмите **Add Variable** → **Raw Editor** (или добавляйте по одной)
3. Добавьте:

```
TELEGRAM_BOT_TOKEN=8283346599:AAGEIT2uf18mxXbKVEyDqlVCwIT4vC1oidM
SUPABASE_URL=https://aveitrccxqbjfxysogiv.supabase.co
SUPABASE_SERVICE_ROLE_KEY=ваш_сервисный_ключ_из_supabase
APP_URL=https://skaz-terem-booking.vercel.app
```

> **Где взять `SUPABASE_SERVICE_ROLE_KEY`:**
> Supabase Dashboard → Settings → API → `service_role` (secret) key

### Шаг 4. Нажмите **Deploy**

Railway автоматически:
1. Склонирует репозиторий
2. Определит Python-проект
3. Установит зависимости из `bot/requirements.txt`
4. Запустит `python bot/main.py` (через `Procfile`)

### Шаг 5. Проверьте логи

В панели проекта откройте **Deployments** → кликните на активный деплой → **View Logs**.

Вы должны увидеть:
```
Bot started successfully ✅
```

### Шаг 6. Остановите локальный бот

```cmd
taskkill /F /IM python.exe
```

### Шаг 7. Протестируйте

Отправьте `/start` боту @SkazTerem_bot в Telegram.

---

## Бесплатный тариф Railway

| Параметр | Значение |
|---|---|
| Кредит | $5/месяц |
| Память | 512 МБ — 1 ГБ |
| Диск | 1 ГБ |
| Время работы | Без ограничений (в рамках кредита) |

> **Расход:** Бот использует ~0.1-0.3 ГБ памяти, что стоит ~$1-2/месяц. $5 кредита хватит на весь месяц.

---

## Структура файлов для Railway

| Файл | Описание |
|------|----------|
| `Procfile` | Указывает Railway, какую команду запустить |
| `runtime.txt` | Версия Python (3.13) |
| `bot/main.py` | Код бота |
| `bot/requirements.txt` | Зависимости Python |

---

## Решение проблем

### Бот не отвечает

1. Проверьте логи в Railway Dashboard → Deployments → View Logs
2. Убедитесь, что все переменные окружения заданы

### Ошибка `TELEGRAM_BOT_TOKEN not set`

Проверьте переменные окружения в Railway:
- **Variables** → убедитесь, что `TELEGRAM_BOT_TOKEN` существует
- Значение без лишних пробелов/кавычек

### Ошибка `No such file or directory: bot/main.py`

Railway не нашёл `Procfile`. Убедитесь, что:
- Файл `Procfile` (без расширения!) находится в корне репозитория
- Содержимое: `worker: python bot/main.py`

### Деплой упал с ошибкой установки зависимостей

Проверьте `bot/requirements.txt` — он должен быть в папке `bot/`, а не в корне.

Railway ищет `requirements.txt` в корне. Если его нет — создайте symlink:

```
# Если нужно, создайте в корне:
pip install -r bot/requirements.txt
```

Но обычно Railway находит его через `Procfile`.

---

## Автосон (idle detection)

Railway **не усыпляет** сервисы, пока есть кредит. Если кредит закончился — сервис останавливается. Пополните аккаунт или дождитесь нового месяца.
