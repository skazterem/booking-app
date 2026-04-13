# Настройка авторизации через Telegram Bot

## Обзор архитектуры

```
Пользователь → /start в @SkazTerem_bot
                ↓
Бот проверяет подписку на канал -1003507317011 (getChatMember)
                ↓
Если подписан → сохраняет chat_id в Supabase (subscribers)
                → генерирует ссылку: app.com/auth?token=UUID
                ↓
Фронтенд читает token → сохраняет в localStorage → пускает в приложение
```

---

## 1. Настройка Supabase

### Шаг 1: Выполнить SQL миграцию

Откройте **Supabase Dashboard** → **SQL Editor** и выполните содержимое файла `supabase-migrations.sql` из корня проекта.

Что делает миграция:
- Создаёт таблицу `subscribers` (хранит chat_id, username, имя подписчика)
- Добавляет колонку `user_id` в таблицу `bookings`
- Включает Row Level Security (RLS)
- Создаёт политики доступа

### Шаг 2: Получить Service Role Key

1. Перейдите в **Settings** → **API**
2. Найдите **Project API keys**
3. Скопируйте ключ с ролью **service_role** (начинается с `eyJ...`)

⚠️ **Никогда не публикуйте service_role key!** Он используется только в Telegram боте.

### Шаг 3: Проверить таблицы

Убедитесь, что таблицы созданы:
- `subscribers` — колонки: `id`, `chat_id`, `username`, `first_name`, `last_name`, `subscribed_at`, `is_active`
- `bookings` — должна появиться колонка `user_id`

---

## 2. Настройка Telegram бота

### Шаг 1: Создать бота через BotFather

1. Откройте [@BotFather](https://t.me/BotFather) в Telegram
2. Отправьте `/newbot`
3. Следуйте инструкциям — получите **BOT_TOKEN**
4. Установите описание: `/setdescription` → "Авторизация в Сказочном Тереме"
5. Установите команду: `/setcommands` → `start - Войти в приложение`

### Шаг 2: Добавить бота в канал как администратора

Бот должен иметь доступ к каналу для проверки подписки:

1. Откройте ваш канал
2. **Управление каналом** → **Администраторы** → **Добавить администратора**
3. Найдите вашего бота и добавьте его

> **Важно:** Если канал публичный — используйте username (`@mychannel`). Если приватный — используйте ID (`-1003507317011`).

### Шаг 3: Создать файл .env

Скопируйте `bot/.env.example` в `bot/.env`:

```bash
cp bot/.env.example bot/.env
```

Заполните переменные:

```env
# Токен бота от @BotFather
TELEGRAM_BOT_TOKEN=123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11

# URL вашего Supabase проекта
SUPABASE_URL=https://aveitrccxqbjfxysogiv.supabase.co

# Service Role Key (из шага 1.2)
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# URL приложения (без слэша на конце)
APP_URL=https://skaz-terem-booking.vercel.app
```

### Шаг 4: Установить зависимости и запустить

```bash
cd bot
pip install -r requirements.txt
python main.py
```

Бот запущен и готов принимать `/start`.

### Шаг 5: Деплой бота (опционально)

Для продакшена рекомендуется запустить бота на сервере:

**Варианты хостинга:**
- [PythonAnywhere](https://www.pythonanywhere.com/) — бесплатный тариф
- [Heroku](https://heroku.com/) — нужен метод оплаты
- [Railway](https://railway.app/) — бесплатный кредит $5/мес
- VPS (DigitalOcean, Hetzner, Timeweb)

Для долгосрочной работы используйте `systemd` или Docker:

```bash
# systemd сервис
sudo nano /etc/systemd/system/skazterem-bot.service
```

```ini
[Unit]
Description=SkazTerem Telegram Bot
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/skazterem-bot
ExecStart=/usr/bin/python3 main.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable skazterem-bot
sudo systemctl start skazterem-bot
sudo systemctl status skazterem-bot
```

---

## 3. Настройка фронтенда

### Переменные окружения на Vercel

В разделе **Settings** → **Environment Variables** на Vercel добавьте:

| Переменная | Значение |
|------------|----------|
| `VITE_TELEGRAM_BOT_USERNAME` | `SkazTerem_bot` (username бота без @) |

> Supabase URL и ключ уже захардкожены в `src/integrations/supabase/client.ts`. Для безопасности рекомендуется перенести их в переменные окружения.

### Обновление ссылки на бота

Если username бота отличается от `SkazTerem_bot`, обновите ссылки:

1. `src/pages/Index.tsx` — строка с `https://t.me/SkazTerem_bot`
2. `src/pages/Auth.tsx` — строка с `https://t.me/SkazTerem_bot`
3. `bot/main.py` — константа `CHANNEL_ID` и текст сообщения

---

## 4. Проверка работы

### Тест 1: Авторизация через бота

1. Откройте @SkazTerem_bot в Telegram
2. Нажмите **/start**
3. Если не подписаны на канал — бот попросит подписаться
4. Если подписаны — бот пришлёт ссылку "🏡 Войти в Сказочный Терем"
5. Нажмите на ссылку — откроется приложение
6. Вы увидите своё имя в шапке

### Тест 2: Создание бронирования

1. Нажмите **Забронировать**
2. Выберите помещение → дату → время
3. На шаге "Детали" — поле имени **отсутствует** (берётся из профиля)
4. Подтвердите бронирование
5. Проверьте в Supabase — в `bookings` должна появиться запись с `user_id`

### Тест 3: Разделение бронирований

1. Зайдите под другим пользователем (другой Telegram аккаунт)
2. Откройте **Мои брони**
3. Вы должны видеть **только свои** бронирования

### Тест 4: Выход из системы

1. Нажмите на имя в шапке (кнопка с иконкой выхода)
2. Кнопка "Мои брони" исчезнет
3. Появится "Войти через Telegram"

---

## 5. Возможные проблемы

### Бот отвечает "Произошла ошибка при сохранении"

**Причина:** Неверный Service Role Key или таблица `subscribers` не создана.

**Решение:**
1. Проверьте `SUPABASE_SERVICE_ROLE_KEY` в `.env`
2. Убедитесь, что миграция выполнена (таблица существует)
3. Проверьте логи бота

### Ссылка не работает / "Неверная ссылка"

**Причина:** Токен не найден в таблице `subscribers`.

**Решение:**
1. Проверьте, что бот сохранил подписчика: `SELECT * FROM subscribers;`
2. Убедитесь, что `APP_URL` в `.env` бота совпадает с реальным URL приложения

### Пользователь видит чужие брони

**Причина:** RLS отключён или не настроен.

**Решение:**
1. Проверьте: `ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;`
2. Убедитесь, что политики созданы правильно
3. Или временно отключите RLS для тестирования:
   ```sql
   ALTER TABLE bookings DISABLE ROW LEVEL SECURITY;
   ```

### Поле "Ваше имя" не исчезло

**Причина:** Старая версия приложения (кеш браузера).

**Решение:** Нажмите `Ctrl+Shift+R` (жёсткий перезапуск).

---

## 6. Рекомендации по безопасности

| Что | Зачем |
|-----|-------|
| Не коммить `.env` в git | Содержит секреты |
| Использовать service_role только в боте | Не на фронтенде! |
| Включить RLS на Supabase | Защита данных на уровне БД |
| Обновлять зависимости | Исправления уязвимостей |
| Мониторить логи бота | Раннее обнаружение проблем |

---

## 7. Структура файлов

```
├── bot/
│   ├── main.py              # Telegram бот
│   ├── requirements.txt     # Python зависимости
│   └── .env.example         # Шаблон переменных
├── src/
│   ├── lib/
│   │   └── auth.ts          # Auth модуль (React Context + hook)
│   └── pages/
│       └── Auth.tsx         # Страница /auth?token=
├── supabase-migrations.sql  # SQL миграция
└── docs/
    └── TELEGRAM_AUTH.md     # Этот файл
```
