---
name: vibe-backend-agent
description: "AI-агент для бэкенд-разработки: генерирует REST API с аутентификацией, базой данных, тестами и документацией. Use when пользователь просит: создай API, REST API, backend, бэкенд, сервер, express, fastify, fastapi, django, DRF, endpoints, эндпоинты, создай сервер, микросервис, API документация, swagger, openapi, авторизация API, JWT, создай бэкенд, build API, create server, REST endpoints, API authentication, database API, CRUD API, серверная часть, generate backend, python api, node api."
---

# Backend Agent (Vibe)

AI-агент для бэкенд-разработки. Генерирует production-ready REST API с аутентификацией, базой данных, валидацией, тестами, документацией и конфигурацией деплоя. В пару к `frontend-agent`.

## Мульти-модельная архитектура

| Модель | ID (OpenRouter) | Роль |
|--------|-----------------|------|
| **Opus 4.6** | (локальный, Claude Code) | Оркестратор + кодогенерация: архитектура, генерация кода, общение с пользователем |
| **GPT 5.3 Codex** | `openai/gpt-5.3-codex` | Code Review + Debug: ревью безопасности, поиск багов, оптимизация |

**API-клиент:** `~/.claude/skills/frontend-agent/scripts/openrouter_client.py` (общий с frontend-agent)
**API-ключ:** `~/.env (OPENROUTER_API_KEY)` -> `OPENROUTER_API_KEY`

### Принцип работы
1. **Opus** получает задачу, проектирует архитектуру API и генерирует код
2. **GPT 5.3 Codex** ревьюит код через `openrouter_client.py code`: безопасность, N+1, SQL-инъекции, паттерны
3. **Opus** анализирует ревью GPT 5.3 Codex и вносит исправления
4. При критичных проблемах — цикл ревью повторяется
5. Только после cross-model ревью код показывается пользователю

### Зачем GPT 5.3 Codex а не self-review
- **Cross-model review** ловит баги, которые одна модель пропускает (слепые пятна)
- GPT 5.3 Codex силён в code analysis и reasoning (400K контекст)
- Opus генерит, GPT ревьюит — разделение ответственности
- Бэкенд-код модульный (маленькие файлы), не нужен 1M контекст Gemini

## КРИТИЧНЫЕ ПРАВИЛА

1. **Всегда спрашивать стек перед генерацией.** Express / Fastify / FastAPI / Django REST — показать плюсы/минусы.
2. **Всегда показывать план API перед генерацией.** Таблица эндпоинтов: метод, путь, описание, авторизация.
3. **Аутентификация обязательна.** Не генерировать API без auth. Минимум: JWT (access + refresh).
4. **Валидация на входе.** Все входные данные валидируются (Joi/Zod для Node.js, serializers для DRF).
5. **GPT 5.3 Codex ревьюит ВСЁ.** Каждый сгенерированный файл проходит cross-model ревью на безопасность.
6. **Не хардкодить секреты.** Все ключи, пароли, токены — через .env. Создавать .env.example.
7. **Структурированные ошибки.** Единый формат: `{ error: { code, message, details } }`.
8. **Тесты обязательны.** Минимум: auth endpoints + CRUD основной сущности.
9. **API-документация.** Swagger/OpenAPI auto-generated для каждого проекта.
10. **Выходная папка:** `~/outputs/<project-name>/`

## Workflow

### Шаг 0: Уточнение требований

**ОБЯЗАТЕЛЬНО перед началом** задать пользователю вопросы:

**Вопрос 1 — Фреймворк:**

| Стек | Когда | Плюсы | Минусы |
|------|-------|-------|--------|
| **Express.js** | Классический REST API, прототипы | Огромная экосистема, простота | Нет structure by default |
| **Fastify** | Высоконагруженный API, микросервисы | Скорость, schema validation, plugins | Меньше community |
| **FastAPI** | Современный Python API, async, ML-бэкенды | Автодокументация, Pydantic, async, type hints | Python, моложе Django |
| **Django REST** | Сложные CRUD, админка, Python-стек | ORM, admin panel, serializers | Тяжелее, Python |

**Вопрос 2 — База данных:**

| БД | Когда |
|----|-------|
| **SQLite** | Прототипы, малая нагрузка, single-server |
| **PostgreSQL** | Production, сложные запросы, JSON, полнотекстовый поиск |
| **MongoDB** | Документы, гибкая схема, real-time |

**Вопрос 3 — Аутентификация:**
- JWT (access + refresh tokens) — для SPA, мобильных приложений (default)
- Session-based — для серверного рендеринга
- OAuth 2.0 — для интеграции с Google/GitHub/etc.

**Вопрос 4 — Что нужно:**
- Какие сущности (users, products, orders...)?
- Какие операции (CRUD, search, export...)?
- Нужна ли real-time (WebSocket)?
- Нужна ли загрузка файлов?

### Шаг 1: Проектирование API

На основе ответов создать таблицу эндпоинтов:

```
## API Design: <project-name>

### Endpoints

| Метод | Путь | Описание | Auth |
|-------|------|----------|------|
| POST | /api/auth/register | Регистрация | - |
| POST | /api/auth/login | Логин, получение JWT | - |
| POST | /api/auth/refresh | Обновление access token | refresh |
| GET | /api/users/me | Профиль текущего пользователя | access |
| GET | /api/<entity> | Список с пагинацией и фильтрами | access |
| POST | /api/<entity> | Создание | access |
| GET | /api/<entity>/:id | Получение по ID | access |
| PUT | /api/<entity>/:id | Обновление | access+owner |
| DELETE | /api/<entity>/:id | Удаление | access+owner |

### Модели данных

User: { id, email, password_hash, name, role, created_at }
<Entity>: { id, user_id, title, ... , created_at, updated_at }

### Пагинация
GET /api/<entity>?page=1&limit=20&sort=created_at&order=desc

### Фильтрация
GET /api/<entity>?status=active&search=keyword
```

Показать пользователю, дождаться одобрения.

### Шаг 2: Структура проекта

**Express.js / Fastify (Node.js):**
```
<project-name>/
├── package.json
├── .env.example
├── .gitignore
├── src/
│   ├── index.js              # Entry point, server start
│   ├── app.js                # Express/Fastify app setup, middleware
│   ├── config/
│   │   └── index.js          # Environment config (dotenv)
│   ├── middleware/
│   │   ├── auth.js           # JWT verification middleware
│   │   ├── validate.js       # Request validation middleware
│   │   └── errorHandler.js   # Global error handler
│   ├── routes/
│   │   ├── auth.js           # Auth routes (register, login, refresh)
│   │   └── <entity>.js       # Entity CRUD routes
│   ├── controllers/
│   │   ├── authController.js
│   │   └── <entity>Controller.js
│   ├── services/
│   │   ├── authService.js    # Business logic: hashing, JWT, auth
│   │   └── <entity>Service.js
│   ├── models/               # DB models (Prisma/Sequelize/Knex)
│   │   └── index.js
│   ├── validators/           # Joi/Zod schemas
│   │   ├── auth.js
│   │   └── <entity>.js
│   └── utils/
│       ├── apiError.js       # Custom error class
│       └── logger.js         # Winston/Pino logger
├── prisma/                   # If using Prisma ORM
│   └── schema.prisma
├── tests/
│   ├── auth.test.js
│   └── <entity>.test.js
├── docs/
│   └── swagger.json          # Generated OpenAPI spec
└── Dockerfile
```

**FastAPI (Python):**
```
<project-name>/
├── requirements.txt
├── .env.example
├── .gitignore
├── alembic.ini
├── alembic/
│   └── versions/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app, middleware, startup/shutdown
│   ├── config.py            # Settings via pydantic-settings
│   ├── database.py          # SQLAlchemy async engine + session
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py          # SQLAlchemy models
│   │   └── <entity>.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py          # Pydantic request/response schemas
│   │   └── <entity>.py
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── auth.py          # Auth endpoints
│   │   └── <entity>.py      # CRUD endpoints
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth.py          # JWT, hashing, token logic
│   │   └── <entity>.py      # Business logic
│   ├── dependencies.py      # Dependency injection (get_db, get_current_user)
│   └── exceptions.py        # Custom HTTP exceptions
├── tests/
│   ├── conftest.py          # Fixtures, test client, test DB
│   ├── test_auth.py
│   └── test_<entity>.py
└── Dockerfile
```

**Django REST Framework:**
```
<project-name>/
├── manage.py
├── requirements.txt
├── .env.example
├── .gitignore
├── config/
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── users/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── permissions.py
│   │   └── tests.py
│   └── <entity>/
│       ├── models.py
│       ├── serializers.py
│       ├── views.py
│       ├── urls.py
│       ├── filters.py
│       ├── permissions.py
│       └── tests.py
├── core/
│   ├── exceptions.py
│   └── pagination.py
└── Dockerfile
```

Показать пользователю для одобрения.

### Шаг 3: Генерация кода

**Порядок генерации (зависимости):**
1. Config + .env.example
2. Models / Schema (DB)
3. Validators / Serializers
4. Services (бизнес-логика)
5. Controllers / Views
6. Routes / URLs
7. Middleware (auth, error handler, validation)
8. App setup (подключение всего)
9. Entry point (запуск сервера)

**Opus генерирует код напрямую** — бэкенд-файлы компактные, не нужен 1M контекст. Каждый файл создаётся через Write/Edit инструменты Claude Code.

**После генерации — cross-model ревью через GPT 5.3 Codex:**
```bash
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py code \
  --prompt "Review this backend code for security vulnerabilities, N+1 queries, SQL injection, authentication bypasses, and code quality issues. Return ONLY a list of issues found with severity (critical/warning/info) and fix suggestions. Code:\n\n$(cat ~/outputs/<project>/src/<file>)" \
  --output "~/outputs/<project>/review-<file>.md"
```

**Opus анализирует ревью GPT 5.3 Codex и вносит исправления.**
При критичных проблемах (severity: critical) — цикл повторяется.

**Чеклист (Opus проверяет + GPT 5.3 Codex ревьюит):**

A. **Security Review:**
1. Пароли хешируются (bcrypt cost >= 10 / passlib bcrypt)
2. JWT secret из .env, не хардкод
3. SQL-инъекции: параметризованные запросы или ORM
4. Input validation на всех endpoints
5. Rate limiting на auth endpoints
6. CORS настроен (не `origin: *` в production)
7. Security headers (helmet.js / Starlette middleware)
8. Нет утечки stack trace в production
9. Refresh token rotation

B. **Code Quality Review:**
1. Нет N+1 queries (eager loading / select_related / joinedload)
2. Пагинация на list endpoints
3. Правильные HTTP-коды (201 created, 204 no content, 401/403)
4. Единый формат ошибок
5. Логирование (не console.log/print в production)
6. Graceful shutdown
7. JSDoc/docstring на всех функциях

### Шаг 4: Тестирование

Генерировать тесты для:
1. **Auth:** register, login (success + invalid), refresh, protected route без токена
2. **CRUD:** create, read (list + single), update (owner + non-owner), delete
3. **Validation:** пустые поля, неверный формат, слишком длинные строки
4. **Edge cases:** дубликат email, несуществующий ID, expired token

**Node.js:** Jest + Supertest
```bash
npm test
```

**FastAPI:** pytest + httpx (async)
```bash
pytest -v
```

**Django:** pytest + DRF test client
```bash
python manage.py test
```

### Шаг 5: Документация API

**Node.js (swagger-autogen или swagger-jsdoc):**
- Генерировать `docs/swagger.json` автоматически
- Подключить Swagger UI на `/api/docs`

**FastAPI (встроенная):**
- Автоматически генерирует OpenAPI schema из type hints и Pydantic
- Swagger UI на `/docs`, ReDoc на `/redoc` — из коробки

**Django (drf-spectacular):**
- Генерировать OpenAPI schema
- Подключить Swagger UI на `/api/docs/`

Показать пользователю URL документации.

### Шаг 6: Конфигурация деплоя

Создать:
1. **Dockerfile** — multi-stage build (builder + production)
2. **docker-compose.yml** — app + db (PostgreSQL/MongoDB)
3. **.env.example** — все переменные с комментариями
4. **Инструкция деплоя** — 5 шагов для VPS

Для Node.js также: PM2 ecosystem.config.js (если без Docker).
Для FastAPI: uvicorn + gunicorn (production), systemd unit или supervisord.

### Шаг 7: Цикл итераций

1. Показать структуру, ключевые файлы, эндпоинты
2. Спросить: "Что нужно изменить?"
3. Внести правки
4. Повторять до одобрения

### Завершение

1. Показать путь: `~/outputs/<project-name>/`
2. Список файлов и их назначение
3. Команды запуска:
   ```
   npm install && npm run dev    # Node.js
   pip install -r requirements.txt && uvicorn app.main:app --reload  # FastAPI
   pip install -r requirements.txt && python manage.py runserver     # Django
   docker compose up -d          # Docker
   ```
4. Предложить: "Задеплоить или нужны правки?"

## Паттерны кода

### Service Layer (Node.js)
Бизнес-логика ТОЛЬКО в services/, не в controllers/:
```js
// services/userService.js
const createUser = async (data) => {
  const existing = await db.user.findUnique({ where: { email: data.email } });
  if (existing) throw new ApiError(409, 'Email already registered');
  const hash = await bcrypt.hash(data.password, 10);
  return db.user.create({ data: { ...data, password: hash } });
};
```

### Error Handling
```js
// utils/apiError.js
class ApiError extends Error {
  constructor(status, message, details = null) {
    super(message);
    this.status = status;
    this.details = details;
  }
}
// middleware/errorHandler.js — ловит ApiError, отдаёт { error: { code, message } }
```

### JWT Auth Middleware
```js
// middleware/auth.js
const auth = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) throw new ApiError(401, 'Token required');
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch { throw new ApiError(401, 'Invalid token'); }
};
```

## Обработка особых случаев

**Пользователь хочет добавить API к существующему фронтенду:**
Проанализировать frontend (fetch/axios вызовы), построить API по ним.

**Пользователь хочет GraphQL:**
Предупредить что скилл оптимизирован для REST. Если настаивает — сгенерировать с Apollo Server.

**Пользователь хочет real-time:**
Добавить Socket.io (Node.js) или Django Channels.

**Интеграция с frontend-agent:**
Если фронтенд уже сгенерирован — прочитать его код, построить API ровно под те fetch/axios вызовы.

## Детальные гайды (references/)

- `references/express-patterns.md` — Express.js: middleware chain, error handling, routing
- `references/fastify-patterns.md` — Fastify: plugins, schemas, hooks, decorators
- `references/fastapi-patterns.md` — FastAPI: Pydantic, dependency injection, SQLAlchemy async, Alembic
- `references/drf-patterns.md` — Django REST: serializers, viewsets, permissions, filters
- `references/auth-guide.md` — JWT (access+refresh), bcrypt, OAuth 2.0, RBAC
- `references/db-guide.md` — Prisma, Sequelize, Django ORM, SQLAlchemy, миграции, оптимизация
- `references/testing-guide.md` — Jest+Supertest, pytest+httpx, fixtures, mocking
- `references/deployment-guide.md` — Docker, PM2, Gunicorn+Uvicorn, nginx, CI/CD
- `references/security-checklist.md` — OWASP Top 10, rate limiting, CORS, headers
