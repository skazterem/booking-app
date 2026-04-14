# Деплой на Vercel

## Быстрый деплой

1. Зайдите на [vercel.com/dashboard](https://vercel.com/dashboard)
2. Нажмите **Add New... → Project**
3. Импортируйте репозиторий: `skazterem/booking-app`
4. Нажмите **Deploy**

Vercel автоматически определит Vite-проект и настроит всё сам.

## Переменные окружения

В настройках проекта на Vercel (**Settings → Environment Variables**) добавьте:

| Key | Value |
|-----|-------|
| `VITE_GOOGLE_APPS_SCRIPT_URL` | URL Google Apps Script для синхронизации календарей |

## SPA-роутинг

Файл `vercel.json` уже настроен — все запросы перенаправляются на `index.html`, роутинг работает через React Router.

## Автосборка

При каждом пуше в ветку `main` Vercel автоматически пересобирает и деплоит новую версию.
