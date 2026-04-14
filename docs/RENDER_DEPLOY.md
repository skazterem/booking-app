# Деплой Telegram-бота на Render.com

## Описание

Инструкция по настройке постоянного работы Telegram-бота @SkazTerem_bot на Render.com.

## Почему не Vercel?

Vercel не подходит для Telegram-бота:

| Характеристика | Vercel | Требование бота |
|---|---|---|
| Тип | Serverless-функции (макс. 10 сек) | Постоянно работающий процесс |
| Long polling | ❌ Не поддерживается | Требует постоянного соединения |
| Python serverless | Только в Next.js проектах | У нас Vite React SPA |

Render.com поддерживает Python 24/7 с бесплатным тарифом (750 часов/месяц).

---

## Пошаговая инструкция

### Шаг 1. Зарегистрируйтесь на Render.com

1. Перейдите на [render.com](https://render.com)
2. Нажмите **Get Started for Free**
3. Войдите через GitHub (рекомендуется — привяжет к репозиторию автоматически)

### Шаг 2. Создайте новый сервис

1. Нажмите **New+ → Web Service**
2. Выберите репозиторий: `digistaff-team/skaz-terem-booking-app`
3. Render автоматически распознаёт `render.yaml` в корне проекта

### Шаг 3. Настройте параметры сервиса

| Поле | Значение |
|------|----------|
| **Name** | `skaz-terem-bot` |
| **Region** | Frankfurt (ближе к России) |
| **Root Directory** | Оставьте пустым |
| **Environment** | `Python` |
| **Build Command** | `pip install -r bot/requirements.txt` |
| **Start Command** | `python bot/main.py` |
| **Instance Type** | `Free` |

### Шаг 4. Добавьте переменные окружения

В разделе **Environment** нажмите **Add Environment Variable** и добавьте:

| Key | Value | Где взять |
|-----|-------|-----------|
| `TELEGRAM_BOT_TOKEN` | `8283346599:AAGEIT2uf18mxXbKVEyDqlVCwIT4vC1oidM` | BotFather в Telegram |
| `SUPABASE_URL` | `https://aveitrccxqbjfxysogiv.supabase.co` | Supabase → Settings → API |
| `SUPABASE_SERVICE_ROLE_KEY` | *(ваш ключ)* | Supabase → Settings → API → `service_role` (secret) |
| `APP_URL` | `https://skaz-terem-booking.vercel.app` | URL вашего веб-приложения |

> **⚠️ Важно:** Не используйте `anon` ключ для Supabase — боту нужен `service_role` (он начинается с `eyJ...` и длиннее).

### Шаг 5. Нажмите «Create Web Service»

Render начнёт деплой:
1. Клонирует репозиторий
2. Установит зависимости (~1-2 мин)
3. Запустит `python bot/main.py`

### Шаг 6. Проверьте логи

В панели сервиса откройте вкладку **Logs**. Вы должны увидеть:

```
Bot started successfully ✅
```

### Шаг 7. Остановите локальный бот

Если бот запущен на вашем ПК, остановите его (иначе будут конфликты):

```cmd
taskkill /F /IM python.exe
```

### Шаг 8. Протестируйте

Отправьте `/start` боту @SkazTerem_bot в Telegram. Бот должен ответить.

---

## Как работает автодеплой

Render настроил **Automatic Deploys** — при каждом пуше в ветку `main` бот автоматически пересоберётся и перезапустится.

Если нужно остановить/перезапустить бот вручную:
- **Manual Deploy** — пересобрать и запустить заново
- **Restart** — перезапустить без пересборки
- **Shutdown** — остановить бот

---

## Бесплатный тариф Render.com

| Лимит | Значение |
|-------|----------|
| Часы работы | 750 часов/месяц (хватит на один сервис 24/7) |
| Память | 512 МБ |
| Диск | 1 ГБ |
| Спящий режим | Сервис «засыпает» после 15 мин бездействия, но бот не засыпает — long polling держит соединение |

> **Примечание:** Если бот всё-таки заснёт, он автоматически проснётся при следующем запросе от Telegram. Это может добавить 10-30 сек задержки к первому сообщению после простоя.

---

## Решение проблем

### Бот не отвечает

1. Проверьте логи в Render Dashboard
2. Убедитесь, что все переменные окружения заданы
3. Проверьте, что `TELEGRAM_BOT_TOKEN` правильный

### Ошибка `TELEGRAM_BOT_TOKEN not set`

Переменная окружения не задана. Проверьте:
- Render Dashboard → Environment Variables → `TELEGRAM_BOT_TOKEN` существует
- Значение не содержит лишних пробелов или кавычек

### Ошибка подключения к Supabase

1. Проверьте `SUPABASE_URL` и `SUPABASE_SERVICE_ROLE_KEY`
2. Убедитесь, что таблица `subscribers` существует в Supabase
3. Проверьте, что RLS не блокирует запросы

### Бот засыпает

На бесплатном тарифе Render может приостанавливать сервисы. Для стабильной работы рассмотрите:
- **Railway.app** ($5 кредит/месяц бесплатно)
- **Fly.io** (3 бесплатых VM)
- Платный Render ($7/месяц)

---

## Файлы проекта

| Файл | Описание |
|------|----------|
| `render.yaml` | Конфигурация сервиса для Render |
| `bot/main.py` | Основной код бота |
| `bot/requirements.txt` | Зависимости Python |
| `bot/.env.example` | Шаблон переменных окружения |
