<h1 align="center">AlertStream</h1>

<p align="center">
  <b>Мульти-канальная notification-платформа</b> — единый шлюз для отправки уведомлений<br/>
  через Email, SMS, Push, Slack, Telegram, WhatsApp и Webhooks с очередями, ретраями и аналитикой.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.12%2B-3776AB?logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/Django-5.x-092E20?logo=django&logoColor=white" alt="Django"/>
  <img src="https://img.shields.io/badge/DRF-3.15-A30000?logo=django&logoColor=white" alt="DRF"/>
  <img src="https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/PostgreSQL-16-336791?logo=postgresql&logoColor=white" alt="PostgreSQL"/>
  <img src="https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white" alt="Redis"/>
  <img src="https://img.shields.io/badge/Celery-5.4-37814A?logo=celery&logoColor=white" alt="Celery"/>
  <img src="https://img.shields.io/badge/Channels-WebSocket-7B42BC?logo=django&logoColor=white" alt="Channels"/>
  <img src="https://img.shields.io/badge/Nginx-Reverse%20Proxy-009639?logo=nginx&logoColor=white" alt="Nginx"/>
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow" alt="License"/>
</p>

---

## Содержание

1. [О проекте](#1-о-проекте)
2. [Ключевые возможности](#2-ключевые-возможности)
3. [Технологический стек](#3-технологический-стек)
4. [Структура репозитория](#4-структура-репозитория)
5. [Архитектура и как это работает](#5-архитектура-и-как-это-работает)
6. [Доменная модель (крупными блоками)](#6-доменная-модель-крупными-блоками)
7. [Сервисы в Docker Compose](#7-сервисы-в-docker-compose)
8. [Быстрый старт (локально, Docker)](#8-быстрый-старт-локально-docker)
9. [Основные команды Makefile](#9-основные-команды-makefile)
10. [Ручной запуск frontend и backend](#10-ручной-запуск-frontend-и-backend)
11. [Конфигурация и переменные окружения](#11-конфигурация-и-переменные-окружения)
12. [API, очереди и интеграции](#12-api-очереди-и-интеграции)
13. [Мониторинг и эксплуатация](#13-мониторинг-и-эксплуатация)
14. [CI/CD](#14-cicd)
15. [Безопасность и доставка сообщений](#15-безопасность-и-доставка-сообщений)
16. [Роли компонентов в продакшене](#16-роли-компонентов-в-продакшене)
17. [Лицензия](#17-лицензия)
18. [Поддержка](#18-поддержка)

---

## 1. О проекте

**AlertStream** — это **продуктовая SaaS-платформа** для централизованной отправки уведомлений в любые каналы: транзакционные письма, SMS-коды, push-нотификации, сообщения в мессенджеры и webhooks для внешних систем. Один HTTP-запрос — десятки каналов, шаблоны, ретраи, аудит доставки и аналитика «из коробки».

Платформа спроектирована для команд, которым нужен **единый notification-gateway** вместо зоопарка SDK-ов в каждом микросервисе: разработчики получают **один API**, продукт — **общие шаблоны и метрики**, а оператор — **очереди, ретраи и мониторинг доставки** в Docker.

### Что это за тип системы

По архитектуре AlertStream — **многосервисная асинхронная платформа** (не «всё в одном процессе»):

| Аспект            | Описание                                                                                          |
|-------------------|---------------------------------------------------------------------------------------------------|
| Продукт           | B2B/B2C-шлюз notifications с шаблонами, маршрутизацией, ретраями и аналитикой доставки            |
| Архитектура       | Django REST API + Celery workers + Channels (WebSocket) + React SPA + Nginx + PostgreSQL + Redis  |
| Доставка          | Асинхронная через Celery-очереди с per-channel rate-limit и exponential backoff retry             |
| Мульти-тенантность| Organization-based изоляция: API-ключи, шаблоны, лимиты и аналитика разделены по тенантам         |
| Интеграции        | SMTP / SendGrid / Twilio / FCM / Slack SDK / Telegram Bot / Meta WhatsApp / любые HTTP webhooks   |
| Эксплуатация      | Docker Compose: dev, staging, production — один и тот же образ, разные переменные окружения       |

### Для кого

- **SaaS-команды**, которым нужен один сервис для всех уведомлений в продукте.
- **Маркетинг и Growth** — кампании, шаблоны с переменными, A/B по каналам.
- **DevOps / SRE** — алерты в Slack / Telegram / email + webhooks в инциденты.
- **Интеграторы** — REST API с API-ключами, без UI-зависимости.

---

## 2. Ключевые возможности

### Доставка сообщений
- **Мульти-канальная отправка** — Email, SMS, Push, Slack, Telegram, WhatsApp, Webhook в одном запросе.
- **Smart Routing** — правила маршрутизации по условиям (priority, type, recipient-атрибуты) автоматически выбирают каналы.
- **Шаблонизатор на Jinja2** — переменные, циклы, условия, мульти-язычность.
- **Приоритеты** — `low / normal / high / critical` с раздельными очередями Celery.
- **Retry-логика** — exponential backoff, настраиваемое число попыток на канал.

### Управление и контроль
- **Rate Limiting** — лимиты per-organization и per-channel в Redis sliding window.
- **API Keys** — `django-rest-framework-api-key` с rotation и scope.
- **Multi-tenancy** — организации, изолированные данные, лимиты и шаблоны.
- **Audit Log** — статус каждой доставки, попытки, ответы провайдера.

### Аналитика и UX
- **Dashboard аналитики** — delivery rate, failure breakdown, channel performance (Recharts).
- **Realtime updates** — WebSocket-уведомления о статусе доставки (Django Channels).
- **OpenAPI / Swagger** — авто-документация через `drf-spectacular`.
- **Sentry-интеграция** — error tracking из коробки.

### Эксплуатация
- **Docker Compose** — `up --build` и платформа стартует целиком.
- **Celery Beat** — расписание для отложенных и периодических кампаний.
- **Healthchecks** — Postgres и Redis имеют healthcheck'и в compose.
- **Production-ready ASGI** — Daphne для HTTP + WebSocket в одном процессе.

---

## 3. Технологический стек

| Слой                 | Технология                                                      |
|----------------------|-----------------------------------------------------------------|
| Язык backend         | Python 3.12+                                                     |
| Web-фреймворк        | Django 5.1, Django REST Framework 3.15                          |
| Async / WebSocket    | Django Channels 4.2, channels-redis, Daphne (ASGI)              |
| Очереди              | Celery 5.4 + Redis (брокер) + django-celery-beat / -results     |
| База данных          | PostgreSQL 16                                                    |
| Кэш / брокер         | Redis 7                                                          |
| Аутентификация       | SimpleJWT, django-rest-framework-api-key                        |
| Шаблонизация         | Jinja2 3.1                                                       |
| Провайдеры           | SendGrid, Twilio, Firebase Admin (FCM), Slack SDK, python-telegram-bot |
| Frontend             | React 18, Redux Toolkit, React Router v6, Recharts              |
| Reverse Proxy        | Nginx 1.25                                                       |
| Контейнеризация      | Docker, Docker Compose                                           |
| Документация API     | drf-spectacular (OpenAPI 3.0)                                    |
| Мониторинг           | Sentry SDK                                                       |
| Тесты                | pytest, pytest-django, factory-boy, coverage                    |
| Качество кода        | black, isort, flake8                                            |

---

## 4. Структура репозитория

```
AlertStream/
├── backend/                       # Django + DRF + Celery
│   ├── apps/
│   │   ├── accounts/              # Пользователи, организации, мульти-тенантность
│   │   ├── api_keys/              # API-ключи, scopes, ротация
│   │   ├── notifications/         # Основной домен: notification, attempt
│   │   ├── channels/              # Email/SMS/Push/Slack/Telegram/WhatsApp/Webhook
│   │   ├── templates_engine/      # Jinja2-шаблоны, версии, переменные
│   │   ├── routing/               # Smart routing rules + условия
│   │   ├── delivery/              # Доставщики, retry, статусы, провайдеры
│   │   ├── rate_limiting/         # Per-org / per-channel sliding window
│   │   ├── analytics/             # Метрики доставки, агрегации
│   │   ├── campaigns/             # Отложенные и периодические рассылки
│   │   └── subscribers/           # Получатели, сегменты, контактные данные
│   ├── config/
│   │   ├── settings/              # base / development / production
│   │   ├── asgi.py                # ASGI + Channels routing
│   │   ├── celery.py              # Celery app: 4 очереди
│   │   └── urls.py                # Корневые URL + OpenAPI
│   ├── tasks/                     # Общие Celery-таски
│   ├── utils/                     # Хелперы, миксины
│   ├── Dockerfile
│   ├── manage.py
│   └── requirements.txt
│
├── frontend/                      # React SPA
│   ├── public/
│   ├── src/
│   │   ├── api/                   # axios + endpoints
│   │   ├── components/            # UI-компоненты
│   │   ├── pages/                 # Dashboard, Notifications, Templates, Routing…
│   │   ├── hooks/                 # WebSocket, auth, fetching
│   │   ├── store/                 # Redux Toolkit slices
│   │   ├── styles/                # Темы, глобальные стили
│   │   └── utils/
│   └── Dockerfile
│
├── nginx/
│   └── nginx.conf                 # Reverse proxy: /api/, /ws/, /admin/, статика
│
├── docker-compose.yml             # db, redis, backend, worker, beat, frontend, nginx
├── .env.example                   # Шаблон переменных окружения
├── .gitignore
└── README.md
```

---

## 5. Архитектура и как это работает

```
                          ┌────────────────────────────┐
                          │      Клиент / Браузер      │
                          └──────────────┬─────────────┘
                                         │ HTTPS
                          ┌──────────────▼─────────────┐
                          │      Nginx (80 / 443)      │
                          │  reverse proxy + статика   │
                          └───────┬──────────┬─────────┘
                                  │          │
                  /, /static     │          │  /api/, /ws/, /admin/
                                  │          │
                ┌─────────────────▼──┐  ┌────▼──────────────────────┐
                │   React SPA (3000) │  │   Django + DRF + Channels │
                │   Redux + Recharts │  │   Daphne ASGI (8000)      │
                └────────────────────┘  └──┬───────────────────┬────┘
                                           │                   │
                                  publish  │                   │ read/write
                                           ▼                   ▼
                          ┌─────────────────────┐   ┌──────────────────────┐
                          │  Redis 7 (broker +  │   │   PostgreSQL 16      │
                          │  cache + channels)  │   │   (metadata + audit) │
                          └──────────┬──────────┘   └──────────────────────┘
                                     │
                  ┌──────────────────┼──────────────────┐
                  │                  │                  │
        ┌─────────▼────────┐ ┌───────▼────────┐ ┌───────▼────────┐
        │  Celery Worker   │ │  Celery Worker │ │  Celery Beat   │
        │  Q: notifications│ │  Q: analytics  │ │  scheduler     │
        │     retries      │ │   default      │ │  (cron-like)   │
        └─────────┬────────┘ └────────────────┘ └────────────────┘
                  │
                  │ HTTPS / SMTP / API
                  ▼
   ┌────────────────────────────────────────────────────────────────┐
   │  Провайдеры доставки                                           │
   │  SMTP · SendGrid · Twilio · FCM · Slack · Telegram · WhatsApp  │
   │  · Custom Webhook                                              │
   └────────────────────────────────────────────────────────────────┘
```

### Жизненный цикл одного уведомления

1. **Запрос приходит** в `POST /api/v1/notifications/` с API-ключом.
2. **DRF-валидация** проверяет схему, лимиты, права API-ключа.
3. **Routing-движок** применяет правила и определяет список каналов.
4. **Notification + Attempts** сохраняются в Postgres со статусом `queued`.
5. **Celery** ставит задачу в очередь `notifications` (приоритет → отдельная очередь).
6. **Worker** рендерит шаблон Jinja2, проверяет rate-limit в Redis, дёргает провайдера.
7. **Провайдер отвечает** → статус `sent / failed / rejected` пишется в attempt.
8. **При ошибке** задача уходит в очередь `retries` с exponential backoff.
9. **WebSocket-уведомление** через Channels летит в UI в реальном времени.
10. **Аналитика** агрегирует delivery-метрики в фоновых тасках на очереди `analytics`.

---

## 6. Доменная модель (крупными блоками)

| Сущность              | Назначение                                                                            |
|-----------------------|---------------------------------------------------------------------------------------|
| **Organization**      | Тенант. Изолирует данные, лимиты, ключи и шаблоны.                                    |
| **User**              | Пользователь UI/админки, принадлежит организации, имеет роль.                         |
| **ApiKey**            | Программный доступ к API. Привязан к организации, имеет scope и срок жизни.           |
| **Subscriber**        | Получатель: email, телефон, push-token, slack-id, telegram-id, webhook-url.            |
| **Template**          | Jinja2-шаблон сообщения: subject + body, переменные, версии, локализация.             |
| **Channel**           | Тип канала и его provider-настройки (Email/SMTP, SMS/Twilio, Push/FCM и т.д.).        |
| **RoutingRule**       | Условия выбора каналов: `priority == "critical"` → `[email, sms, slack]`.             |
| **Notification**      | Единица отправки: получатель, шаблон, контекст, приоритет, итоговый статус.            |
| **DeliveryAttempt**   | Попытка отправки в конкретный канал: статус, ответ провайдера, время.                  |
| **RateLimit**         | Per-organization / per-channel лимиты в sliding window Redis.                          |
| **Campaign**          | Отложенная или периодическая рассылка по сегменту получателей.                         |
| **AnalyticsSnapshot** | Агрегаты доставки за период: delivery-rate, failures, channel-mix.                     |

---

## 7. Сервисы в Docker Compose

| Сервис                  | Образ / сборка               | Порт      | Назначение                                            |
|-------------------------|------------------------------|-----------|-------------------------------------------------------|
| `alertstream_db`        | `postgres:16-alpine`         | 5432      | Основная БД: метаданные, audit, шаблоны               |
| `alertstream_redis`     | `redis:7-alpine`             | 6379      | Брокер Celery + кэш + Channels layer                  |
| `alertstream_backend`   | build `./backend`            | 8000      | Daphne ASGI: Django REST + WebSocket                  |
| `alertstream_celery_worker` | build `./backend`        | —         | Воркеры очередей `default / notifications / analytics / retries` |
| `alertstream_celery_beat`   | build `./backend`        | —         | Планировщик (django-celery-beat, DB scheduler)        |
| `alertstream_frontend`  | build `./frontend`           | 3000      | React SPA (Redux Toolkit + Recharts)                  |
| `alertstream_nginx`     | `nginx:1.25-alpine`          | 80 / 443  | Reverse proxy, статика, WebSocket-апгрейд             |

Volumes: `postgres_data`, `redis_data`, `static_volume`, `media_volume`.

---

## 8. Быстрый старт (локально, Docker)

### Требования

- Docker 24+ и Docker Compose v2
- Git
- 4 ГБ свободной RAM, 2 ядра CPU

### Шаги

```bash
# 1. Клонируем репозиторий
git clone https://github.com/NodirOdilov/AlertStream.git
cd AlertStream

# 2. Готовим переменные окружения
cp .env.example .env
# отредактируйте .env: SECRET_KEY, токены провайдеров, пароли

# 3. Поднимаем весь стек
docker compose up --build -d

# 4. Создаём суперпользователя
docker compose exec backend python manage.py createsuperuser

# 5. (опционально) Загружаем демо-данные
docker compose exec backend python manage.py loaddata fixtures/demo.json
```

### Точки входа

| Сервис             | URL                                  |
|--------------------|--------------------------------------|
| Web-приложение     | http://localhost                     |
| REST API           | http://localhost/api/v1/             |
| OpenAPI / Swagger  | http://localhost/api/docs/           |
| Django Admin       | http://localhost/admin/              |
| WebSocket          | ws://localhost/ws/notifications/     |

Логи всех сервисов: `docker compose logs -f` — или конкретного: `docker compose logs -f backend`.

---

## 9. Основные команды Makefile

> Если в репозитории есть `Makefile`, типичные цели выглядят так:

```bash
make up              # docker compose up -d --build
make down            # docker compose down
make logs            # docker compose logs -f
make shell           # bash внутри backend-контейнера
make migrate         # python manage.py migrate
make makemigrations  # python manage.py makemigrations
make superuser       # python manage.py createsuperuser
make test            # pytest + coverage
make lint            # flake8 + black --check + isort --check
make format          # black . + isort .
make worker          # запустить Celery worker вручную
make beat            # запустить Celery beat вручную
```

---

## 10. Ручной запуск frontend и backend

### Backend (без Docker)

```bash
cd backend
python -m venv .venv
source .venv/bin/activate              # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Поднимите локальные Postgres и Redis (или используйте docker compose для них)
export DJANGO_SETTINGS_MODULE=config.settings.development

python manage.py migrate
python manage.py createsuperuser
daphne -b 0.0.0.0 -p 8000 config.asgi:application
```

В отдельных окнах:

```bash
celery -A config worker -l info -Q default,notifications,analytics,retries
celery -A config beat   -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```

### Frontend (без Docker)

```bash
cd frontend
npm install
npm start                      # dev-сервер на http://localhost:3000
# production-сборка
npm run build
```

---

## 11. Конфигурация и переменные окружения

Полный список — в [`.env.example`](.env.example). Ключевые группы:

### Django

| Переменная               | Назначение                                      |
|--------------------------|-------------------------------------------------|
| `SECRET_KEY`             | Секрет Django, обязателен в production          |
| `DEBUG`                  | `1` / `0`                                        |
| `ALLOWED_HOSTS`          | Список хостов через запятую                     |
| `CORS_ALLOWED_ORIGINS`   | Разрешённые источники для SPA                   |

### PostgreSQL / Redis

| Переменная               | Назначение                                      |
|--------------------------|-------------------------------------------------|
| `POSTGRES_DB / USER / PASSWORD / PORT` | Кредиты основной БД                 |
| `REDIS_PASSWORD / PORT`  | Доступ к Redis                                  |
| `CELERY_BROKER_URL`      | URL брокера для Celery                          |

### Провайдеры доставки

| Переменная                            | Канал                          |
|---------------------------------------|--------------------------------|
| `EMAIL_HOST / PORT / USER / PASSWORD` | SMTP                           |
| `SENDGRID_API_KEY`                    | SendGrid (альт. email)         |
| `TWILIO_ACCOUNT_SID / AUTH_TOKEN / PHONE_NUMBER / WHATSAPP_NUMBER` | SMS, WhatsApp |
| `FCM_SERVER_KEY / FCM_PROJECT_ID`     | Push (Firebase)                |
| `SLACK_BOT_TOKEN / SIGNING_SECRET`    | Slack                          |
| `TELEGRAM_BOT_TOKEN`                  | Telegram                       |

### Ретраи и лимиты

| Переменная                       | Назначение                          |
|----------------------------------|-------------------------------------|
| `DEFAULT_RATE_LIMIT_REQUESTS`    | Запросов в окно по умолчанию        |
| `DEFAULT_RATE_LIMIT_WINDOW`      | Длина окна (сек)                    |
| `MAX_RETRY_ATTEMPTS`             | Сколько раз ретраить упавшую отправку|
| `RETRY_BACKOFF_BASE / MULTIPLIER`| Параметры exponential backoff       |

---

## 12. API, очереди и интеграции

### REST API

Все запросы идут с заголовком:

```
Authorization: Api-Key <YOUR_API_KEY>
Content-Type: application/json
```

#### Отправить уведомление

```bash
curl -X POST http://localhost/api/v1/notifications/ \
  -H "Authorization: Api-Key $ALERTSTREAM_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "recipient": "user@example.com",
    "channels": ["email", "slack"],
    "template_id": "welcome-email",
    "context": {
      "user_name": "John Doe",
      "activation_link": "https://example.com/activate/abc123"
    },
    "priority": "high"
  }'
```

#### Получить статус доставки

```bash
curl http://localhost/api/v1/notifications/{id}/status/ \
  -H "Authorization: Api-Key $ALERTSTREAM_KEY"
```

#### Список уведомлений

```bash
curl "http://localhost/api/v1/notifications/?status=failed&channel=sms" \
  -H "Authorization: Api-Key $ALERTSTREAM_KEY"
```

Полная спецификация — `GET /api/docs/` (Swagger UI) и `GET /api/schema/` (OpenAPI JSON).

### Очереди Celery

| Очередь          | Что обрабатывает                                                |
|------------------|------------------------------------------------------------------|
| `default`        | Системные таски, housekeeping                                    |
| `notifications`  | Основная отправка через провайдеров                              |
| `retries`        | Повторные попытки с exponential backoff                          |
| `analytics`      | Агрегация метрик, отчёты, snapshot'ы                             |

Конфигурация воркера (см. `docker-compose.yml`):

```bash
celery -A config worker -l info -c 4 -Q default,notifications,analytics,retries
```

### Routing Rule (пример)

```json
{
  "name": "Critical → All",
  "conditions": [
    {"field": "priority", "operator": "eq", "value": "critical"}
  ],
  "channels": ["email", "sms", "push", "slack"],
  "priority": 1
}
```

### Поддерживаемые провайдеры

| Канал     | Провайдеры                              |
|-----------|------------------------------------------|
| Email     | SMTP, SendGrid, Mailgun, AWS SES         |
| SMS       | Twilio, Vonage, MessageBird              |
| Push      | FCM, APNS, OneSignal                     |
| Slack     | Slack Bot API                            |
| Telegram  | Telegram Bot API                         |
| WhatsApp  | Twilio WhatsApp, Meta Cloud API          |
| Webhook   | Произвольные HTTP endpoints              |

---

## 13. Мониторинг и эксплуатация

- **Healthchecks** Postgres и Redis в `docker-compose.yml` (запуск backend ждёт `service_healthy`).
- **Sentry** — задайте `SENTRY_DSN` в `.env` и ошибки backend/worker полетят в проект.
- **OpenAPI** — авто-документация и схема для контрактных тестов.
- **Логи** — `docker compose logs -f <service>` или агрегатор (ELK / Loki).
- **WebSocket-live** — статусы доставки приходят в UI без перезагрузки.
- **Celery flower** (опционально) — UI для очередей: `celery -A config flower`.

Базовые операции:

```bash
docker compose ps                       # статус сервисов
docker compose restart backend          # перезапуск backend
docker compose exec backend bash        # шелл внутри контейнера
docker compose exec backend python manage.py shell_plus
docker compose down -v                  # снести всё, включая volumes (осторожно!)
```

---

## 14. CI/CD

Рекомендуемый pipeline (GitHub Actions / GitLab CI):

| Стадия     | Действия                                                    |
|------------|-------------------------------------------------------------|
| `lint`     | `black --check .`, `isort --check .`, `flake8`              |
| `test`     | `pytest --cov` (backend), `npm test -- --watchAll=false` (frontend) |
| `build`    | `docker build` для `backend` и `frontend`                   |
| `publish`  | Push образов в registry (GHCR / ECR / Docker Hub)           |
| `deploy`   | `docker compose pull && docker compose up -d` на target-сервере |

Минимальный pre-commit:

```bash
docker compose exec backend bash -lc "black --check . && isort --check . && flake8 && pytest"
```

---

## 15. Безопасность и доставка сообщений

- **API-ключи** — `django-rest-framework-api-key` с хэшированием в БД, ротация и scope.
- **JWT** — `djangorestframework-simplejwt` для пользователей UI.
- **CORS** — белый список через `CORS_ALLOWED_ORIGINS`.
- **Rate Limiting** — sliding window в Redis, per-organization и per-channel.
- **Изоляция тенантов** — все запросы фильтруются по `organization_id`, проверяется владелец API-ключа.
- **Хранение секретов** — токены провайдеров читаются только из `.env` / secret manager, в репозитории — `.env.example`.
- **Webhook-подписи** — рекомендуется подписывать исходящие webhooks HMAC-SHA256.
- **Retry без дублей** — `idempotency_key` в запросе и уникальные `DeliveryAttempt`.

---

## 16. Роли компонентов в продакшене

| Компонент            | Роль в продакшене                                                       |
|----------------------|--------------------------------------------------------------------------|
| **Nginx**            | TLS-терминация, статика, проксирование `/api/`, `/ws/`, `/admin/`        |
| **Daphne (backend)** | ASGI: HTTP + WebSocket в одном процессе                                  |
| **Celery worker**    | Все исходящие к провайдерам, ретраи, аналитика                           |
| **Celery beat**      | Расписание кампаний и периодических job'ов                               |
| **PostgreSQL**       | Источник правды: организации, шаблоны, notification + attempts, audit    |
| **Redis**            | Брокер Celery, кэш, channels layer, sliding-window лимиты                |
| **React SPA**        | Управление шаблонами, ключами, маршрутизацией, мониторинг доставки       |
| **Sentry**           | Захват ошибок backend и воркеров                                         |

---

## 17. Лицензия

Проект распространяется под лицензией **MIT**. Подробнее — в файле [LICENSE](LICENSE).

---

## 18. Поддержка

- **Issues:** https://github.com/NodirOdilov/AlertStream/issues
- **Discussions / вопросы по архитектуре:** вкладка Discussions в GitHub
- **Автор:** [@NodirOdilov](https://github.com/NodirOdilov)

> Если AlertStream был вам полезен — поставьте ⭐ в репозитории, это поддерживает развитие проекта.
