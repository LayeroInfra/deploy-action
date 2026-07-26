# Deploy to Layero — GitHub Action

> **Layero** — российская платформа хостинга и деплоя фронтенд-приложений.
> Деплой одной командой `npx layero deploy`, серверы и CDN внутри России,
> поддержка Next.js / Vite / Astro / SvelteKit / Nuxt и деплой прямо из
> AI-агентов (Cursor, Claude Code).

🌐 Сайт: <https://layero.ru> · 📚 Документация: <https://docs.layero.ru> · 📦 npm: <https://www.npmjs.com/package/layero>

Публикует сайт или приложение на Layero из GitHub Actions.

## Быстрый старт

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: LayeroInfra/deploy-action@v1
        with:
          token: ${{ secrets.LAYERO_TOKEN }}
          prod: true
```

Токен создаётся на странице [app.layero.ru/settings/cli](https://app.layero.ru/settings/cli)
и кладётся в секреты репозитория как `LAYERO_TOKEN`. Он **бессрочный** — в
отличие от обычной сессии входа не протухает через неделю; отключается там же,
где создавался.

## Когда это нужно

Если репозиторий просто связан с Layero, сборка запускается вебхуком сама и
Action не нужен.

Action нужен, **когда сборку надо выполнить в CI**: нужны секреты, к которым у
платформы нет доступа, приватные npm-зависимости, свои шаги проверки перед
публикацией. Тогда собираете у себя и публикуете готовый результат:

```yaml
      - run: npm ci && npm run build
        env:
          API_KEY: ${{ secrets.API_KEY }}
      - uses: LayeroInfra/deploy-action@v1
        with:
          token: ${{ secrets.LAYERO_TOKEN }}
          prebuilt: dist
          prod: true
```

## Параметры

| Параметр | Обязателен | По умолчанию | Что делает |
|---|---|---|---|
| `token` | да | — | CI-токен Layero |
| `project` | нет | из `.layero/project.json` | Проект-получатель (id или slug) |
| `prod` | нет | `false` | Публиковать в production, а не в preview |
| `branch` | нет | — | Окружение конкретной ветки. Приоритетнее `prod` |
| `prebuilt` | нет | — | Каталог с готовой сборкой — не собирать на Layero |
| `type` | нет | автоопределение | Фреймворк: `vite`, `next`, `astro`, `static`… |
| `root` | нет | — | Монорепа: подкаталог, который считать корнем приложения |
| `working-directory` | нет | `.` | Откуда запускать деплой |
| `version` | нет | `latest` | Версия npm-пакета `layero` |

## Результаты

| Выход | Что содержит |
|---|---|
| `url` | Адрес, по которому опубликован деплой |
| `deploy-id` | Идентификатор деплоя |

```yaml
      - uses: LayeroInfra/deploy-action@v1
        id: deploy
        with:
          token: ${{ secrets.LAYERO_TOKEN }}
      - run: echo "Опубликовано на ${{ steps.deploy.outputs.url }}"
```

Адрес также попадает в summary задачи — его видно на странице запуска, без
чтения логов.

## Превью на пулл-реквесты

```yaml
on: [pull_request]

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: LayeroInfra/deploy-action@v1
        id: deploy
        with:
          token: ${{ secrets.LAYERO_TOKEN }}
          branch: pr-${{ github.event.number }}
```

Без `prod` деплой уходит в preview-окружение и продакшн не трогает.

## Безопасность

Токен передаётся в CLI **через переменную окружения, а не аргументом**:
аргументы процесса видны другим процессам на раннере и попадают в трассировки.

У токена узнаваемый префикс `layero_ci_` — если он случайно попадёт в
публичный репозиторий, это заметно и автоматическими сканерами секретов.

⚠️ Токен даёт права уровня аккаунта, как обычная сессия входа. Заводите
отдельный токен под каждый репозиторий: тогда компрометацию одного можно
закрыть, не ломая остальные.

## Лицензия

[MIT](./LICENSE)
