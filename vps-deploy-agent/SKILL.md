---
name: vps-deploy-agent
description: "Автоматизирует деплой готовых проектов на Ubuntu VPS. Use when пользователь просит: задеплой, деплой на сервер, деплой на VPS, deploy, деплой сайта, деплой бота, деплой API, деплой фронтенда, деплой бэкенда, настроить nginx, настроить pm2, настроить gunicorn, настроить docker, ssl сертификат, настроить сервер, production деплой, продакшен, nginx конфиг, запустить на сервере, разместить сайт, vps setup, server config, деплой react, деплой vue, деплой node, деплой python, деплой fastapi, деплой django, настроить ufw, настроить fail2ban, деплой телеграм бота."
---

# VPS Deploy Agent

Скилл для автоматизации деплоя проектов на Ubuntu VPS. Генерирует конфиги Nginx, PM2, Systemd, Docker Compose, скрипты деплоя и настраивает безопасность.

## Поддерживаемые сценарии

| Тип проекта | Технологии |
|---|---|
| Статический фронтенд | HTML/CSS/JS, React build, Vue build |
| Node.js бэкенд | Express, Fastify, NestJS + PM2 |
| Python бэкенд | FastAPI + Uvicorn, Django + Gunicorn |
| Telegram-бот | Node.js + PM2 |
| Docker multi-service | frontend + backend + bot + PostgreSQL/Redis |
| Полный стек | Все компоненты разом |

## КРИТИЧНЫЕ ПРАВИЛА

1. **Всегда уточнять стек перед генерацией конфигов.** Задать вопросы из Шага 0.
2. **Всегда проверять nginx -t** перед nginx reload.
3. **Никогда не открывать порты приложений наружу.** Только через Nginx.
4. **Секреты только в .env с chmod 600.** Никакого хардкода.
5. **Бэкап перед деплоем.** Всегда создавать backup текущей версии.
6. **UFW включать ПОСЛЕ разрешения SSH.** Иначе потеря доступа к серверу.
7. **certbot --nginx** запускать только после настройки nginx.
8. **Выходная папка:** `~/outputs/<project-name>-deploy/`

## Workflow

### Шаг 0: Уточнение задачи

Задать пользователю вопросы:

**Вопрос 1 - Что деплоим:**
- Только фронтенд (статика / SPA)?
- Только бэкенд (Node.js / Python)?
- Только Telegram-бот?
- Полный стек (несколько компонентов)?
- Docker Compose?

**Вопрос 2 - Параметры сервера:**
- IP или домен VPS?
- ОС (Ubuntu 22.04 / 24.04)?
- SSH-доступ: пользователь, порт, ключ или пароль?
- Домен(ы)? (например `app.example.com`, `api.example.com`)

**Вопрос 3 - Стек приложения:**
- Фронтенд: HTML статика / React build / Vue build?
- Бэкенд: Node.js (Express/Fastify) / Python (FastAPI/Django)?
- БД: PostgreSQL / MySQL / SQLite / Redis?
- Telegram-бот: есть?

**Вопрос 4 - Что уже есть на сервере:**
- Nginx установлен?
- SSL настроен?
- UFW включён?
- PM2 / Docker установлен?

**Вопрос 5 - Путь к проекту:**
- Откуда деплоить (локальная папка / GitHub репо)?
- Куда на сервере (`/var/www/<name>/` или `/home/deploy/<name>/`)?

### Шаг 1: Составить план деплоя

На основе ответов составить таблицу компонентов:

```
## План деплоя: <project-name>

| Компонент | Технология | Путь на сервере | Порт | Домен |
|---|---|---|---|---|
| Frontend | React (build) | /var/www/app/ | — | app.example.com |
| Backend API | Node.js + PM2 | /var/www/api/ | 3001 (внутренний) | api.example.com |
| Telegram Bot | Node.js + PM2 | /var/www/bot/ | — | — |
| Database | PostgreSQL | Docker volume | 5432 (внутренний) | — |

## Что будет сделано:
1. [ ] Установить зависимости (nginx, certbot, pm2, node, python...)
2. [ ] Настроить UFW (порты 22, 80, 443)
3. [ ] Загрузить файлы проекта на сервер
4. [ ] Настроить Nginx (reverse proxy + SSL)
5. [ ] Получить SSL сертификат (Let's Encrypt)
6. [ ] Запустить сервисы (PM2 / systemd / Docker)
7. [ ] Настроить безопасность (fail2ban, security headers)
8. [ ] Настроить автозапуск при ребуте
```

Показать пользователю, дождаться одобрения.

### Шаг 2: Генерация конфигов и скриптов

Сгенерировать нужные файлы в `~/outputs/<project-name>-deploy/`.

### Шаг 3: Выполнение деплоя

Выполнить команды через SSH (если у пользователя есть доступ к серверу из текущей сессии).
Если нет - предоставить готовые скрипты для выполнения вручную.

### Шаг 4: Проверка

- Открыть сайт в браузере через Playwright
- Проверить SSL (https://)
- Проверить API endpoints
- Показать результат

---

## ЧАСТЬ 1: СТАТИЧЕСКИЙ ФРОНТЕНД

### Nginx конфиг для HTML статики

```nginx
# /etc/nginx/sites-available/<domain>
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;

    root /var/www/html;
    index index.html;

    # SSL (certbot заполнит автоматически)
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # TLS оптимизации
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 1.1.1.1 8.8.8.8 valid=300s;
    resolver_timeout 5s;

    # Gzip сжатие
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 4;
    gzip_types text/plain text/css text/xml text/javascript
               application/javascript application/x-javascript application/json
               application/xml application/rss+xml font/truetype font/opentype
               application/vnd.ms-fontobject image/svg+xml;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;

    # Статика: не-хешированные файлы (30 дней)
    location ~* \.(css|js|png|jpg|jpeg|gif|webp|ico|woff|woff2|ttf|svg)$ {
        expires 30d;
        add_header Cache-Control "public, max-age=2592000" always;
        access_log off;
    }

    # Блокировка опасных путей
    location ~ /\. { deny all; }
    location ~ /\.git { deny all; }
    location ~ /\.env { deny all; }
    location ~ /backups { deny all; }
    location ~ \.(log|conf|bak|sql|sh)$ { deny all; }

    # Основной файл
    location / {
        try_files $uri $uri/ =404;
    }

    # Логи
    access_log /var/log/nginx/example.com.access.log;
    error_log  /var/log/nginx/example.com.error.log warn;
}
```

### Nginx конфиг для SPA (React / Vue)

Отличие от статики: `try_files $uri $uri/ /index.html` + раздельное кеширование index.html и assets.

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name app.example.com;

    root /var/www/app/dist;
    index index.html;

    # SSL (аналогично статике)
    ssl_certificate /etc/letsencrypt/live/app.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 1.1.1.1 8.8.8.8 valid=300s;

    # Gzip
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 4;
    gzip_types text/plain text/css text/xml text/javascript
               application/javascript application/json application/xml
               image/svg+xml font/woff font/woff2;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # index.html - NO CACHE (чтобы деплой сразу применялся)
    location = /index.html {
        add_header Cache-Control "no-cache, no-store, must-revalidate" always;
        add_header Pragma "no-cache" always;
    }

    # Хешированные ассеты (Vite/CRA: /assets/main.abc123.js) - LONG CACHE
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, max-age=31536000, immutable" always;
        access_log off;
    }

    # Прочие статические файлы
    location ~* \.(png|jpg|jpeg|gif|webp|ico|woff|woff2|ttf|svg)$ {
        expires 30d;
        add_header Cache-Control "public, max-age=2592000" always;
        access_log off;
    }

    # SPA: все маршруты -> index.html (React Router / Vue Router)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Блокировка
    location ~ /\. { deny all; }
    location ~ /\.git { deny all; }
    location ~ /\.env { deny all; }

    access_log /var/log/nginx/app.example.com.access.log;
    error_log  /var/log/nginx/app.example.com.error.log warn;
}

# HTTP -> HTTPS
server {
    listen 80;
    server_name app.example.com;
    return 301 https://$host$request_uri;
}
```

### Команды деплоя фронтенда

```bash
# 1. Создать директорию
sudo mkdir -p /var/www/app
sudo chown -R www-data:www-data /var/www/app
sudo chmod -R 755 /var/www/app

# 2. Загрузить файлы (из локальной папки)
# Вариант A: rsync (рекомендуется, только изменённые файлы)
rsync -avz --delete ./dist/ user@SERVER_IP:/var/www/app/
# Вариант B: scp
scp -r ./dist/* user@SERVER_IP:/var/www/app/

# 3. Активировать конфиг nginx
sudo ln -s /etc/nginx/sites-available/app.example.com /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

# 4. Получить SSL
sudo certbot --nginx -d app.example.com --non-interactive --agree-tos --email admin@example.com
```

---

## ЧАСТЬ 2: NODE.JS БЭКЕНД (PM2)

### PM2 ecosystem.config.js

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'api',
      script: './src/index.js',
      // Для CPU-intensive: cluster mode
      exec_mode: 'cluster',
      instances: 'max',        // по числу ядер CPU
      // Для I/O-intensive (API, бот): fork mode
      // exec_mode: 'fork',
      // instances: 1,

      // Перезапуск при утечке памяти
      max_memory_restart: '500M',

      // Логи
      merge_logs: true,
      log_date_format: 'YYYY-MM-DD HH:mm:ss',
      error_file: '/var/log/pm2/api-error.log',
      out_file: '/var/log/pm2/api-out.log',

      // Переменные окружения
      env: {
        NODE_ENV: 'production',
        PORT: 3001,
      },
      env_development: {
        NODE_ENV: 'development',
        PORT: 3001,
      },

      // Автоперезапуск при крэше
      autorestart: true,
      max_restarts: 10,
      restart_delay: 3000,

      // Graceful shutdown
      kill_timeout: 5000,
      wait_ready: true,
      listen_timeout: 10000,
    },

    // Telegram-бот (fork mode, 1 инстанс)
    {
      name: 'telegram-bot',
      script: './bot.js',
      exec_mode: 'fork',
      instances: 1,
      max_memory_restart: '200M',
      merge_logs: true,
      error_file: '/var/log/pm2/bot-error.log',
      out_file: '/var/log/pm2/bot-out.log',
      env: {
        NODE_ENV: 'production',
      },
      autorestart: true,
      max_restarts: 10,
      restart_delay: 5000,
    },
  ],
};
```

### Graceful shutdown в Node.js приложении

```javascript
// В index.js добавить:
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down...');
  // Закрыть HTTP-сервер (перестать принимать новые)
  server.close(async () => {
    // Закрыть БД
    await db.$disconnect();
    console.log('Graceful shutdown complete');
    process.exit(0);
  });
  // Форс через 10 сек
  setTimeout(() => process.exit(1), 10000);
});

// Сообщить PM2 о готовности (wait_ready: true)
process.send('ready');
```

### Nginx reverse proxy для Node.js

```nginx
# /etc/nginx/sites-available/api.example.com
upstream nodejs_api {
    server 127.0.0.1:3001;
    keepalive 64;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 1.1.1.1 8.8.8.8 valid=300s;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Gzip для JSON ответов
    gzip on;
    gzip_types application/json application/javascript text/plain;
    gzip_min_length 1024;

    # Лимит размера запроса (загрузка файлов)
    client_max_body_size 10M;

    location / {
        proxy_pass http://nodejs_api;
        proxy_http_version 1.1;

        # Передать реальный IP клиента
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;

        # WebSocket поддержка
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Таймауты
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # Буферизация
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # Health check endpoint (не логировать)
    location /health {
        proxy_pass http://nodejs_api;
        access_log off;
    }

    # Блокировка
    location ~ /\. { deny all; }

    access_log /var/log/nginx/api.example.com.access.log;
    error_log  /var/log/nginx/api.example.com.error.log warn;
}

server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;
}
```

### Команды деплоя Node.js

```bash
# На сервере (первый раз):
# 1. Установить Node.js (LTS)
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# 2. Установить PM2 глобально
sudo npm install -g pm2

# 3. Создать папку приложения
sudo mkdir -p /var/www/api
sudo chown -R $USER:$USER /var/www/api

# 4. Загрузить файлы
rsync -avz --exclude='node_modules' --exclude='.git' --exclude='.env' \
  ./api/ user@SERVER_IP:/var/www/api/

# 5. На сервере: установить зависимости
ssh user@SERVER_IP "cd /var/www/api && npm ci --production"

# 6. Скопировать .env
scp ./.env.production user@SERVER_IP:/var/www/api/.env
ssh user@SERVER_IP "chmod 600 /var/www/api/.env"

# 7. Запустить через PM2
ssh user@SERVER_IP "cd /var/www/api && pm2 start ecosystem.config.js --env production"
ssh user@SERVER_IP "pm2 save && pm2 startup"
# PM2 startup выдаёт команду - выполнить её!

# Zero-downtime reload при обновлении:
ssh user@SERVER_IP "cd /var/www/api && git pull && npm ci --production && pm2 reload api"
```

### PM2: полезные команды

```bash
pm2 list                    # Список процессов
pm2 logs api                # Логи (последние 200 строк + realtime)
pm2 logs api --lines 500    # Последние N строк
pm2 monit                   # Дашборд мониторинга
pm2 reload api              # Zero-downtime reload
pm2 restart api             # Полный перезапуск
pm2 stop api                # Остановить
pm2 delete api              # Удалить из PM2
pm2 show api                # Детали процесса
pm2 save                    # Сохранить текущее состояние
pm2 startup                 # Автозапуск при ребуте
pm2 unstartup               # Отключить автозапуск
```

### Logrotate для PM2

```bash
# Установить pm2-logrotate
pm2 install pm2-logrotate

# Настроить
pm2 set pm2-logrotate:max_size 50M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'
```

---

## ЧАСТЬ 3: PYTHON БЭКЕНД (FastAPI / Django)

### Systemd unit для FastAPI + Uvicorn

```ini
# /etc/systemd/system/fastapi.service
[Unit]
Description=FastAPI Application
After=network.target
Requires=network.target

[Service]
Type=notify
User=www-data
Group=www-data
WorkingDirectory=/var/www/api
Environment=PATH=/var/www/api/venv/bin
ExecStart=/var/www/api/venv/bin/gunicorn app.main:app \
    --workers 4 \
    --worker-class uvicorn.workers.UvicornWorker \
    --bind unix:/run/fastapi.sock \
    --pid /run/fastapi.pid \
    --access-logfile /var/log/gunicorn/access.log \
    --error-logfile /var/log/gunicorn/error.log \
    --log-level info \
    --timeout 30
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true
Restart=on-failure
RestartSec=5s
KillMode=mixed
TimeoutStopSec=30

[Install]
WantedBy=multi-user.target
```

Число workers: `(2 * CPU_cores) + 1` для CPU-bound, для async (FastAPI/aiohttp) = число ядер.

### Systemd unit для Django + Gunicorn

```ini
# /etc/systemd/system/django.service
[Unit]
Description=Django Application (Gunicorn)
After=network.target postgresql.service
Requires=postgresql.service

[Service]
Type=notify
User=www-data
Group=www-data
WorkingDirectory=/var/www/django
Environment=PATH=/var/www/django/venv/bin
Environment=DJANGO_SETTINGS_MODULE=config.settings.production
EnvironmentFile=/var/www/django/.env
ExecStart=/var/www/django/venv/bin/gunicorn \
    --workers 3 \
    --bind unix:/run/django.sock \
    --pid /run/django.pid \
    --access-logfile /var/log/gunicorn/access.log \
    --error-logfile /var/log/gunicorn/error.log \
    --log-level info \
    --timeout 30 \
    config.wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID
PrivateTmp=true
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### Nginx reverse proxy для Python (Unix socket)

```nginx
# /etc/nginx/sites-available/api.example.com
upstream python_api {
    server unix:/run/fastapi.sock fail_timeout=0;
    # Или для TCP: server 127.0.0.1:8000;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # SSL
    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 1.1.1.1 8.8.8.8 valid=300s;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    client_max_body_size 10M;

    location / {
        include proxy_params;
        proxy_pass http://python_api;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_read_timeout 60s;
        proxy_send_timeout 60s;
    }

    # Кешировать статику Django (если collectstatic)
    location /static/ {
        alias /var/www/django/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, max-age=2592000" always;
        access_log off;
    }

    location /media/ {
        alias /var/www/django/media/;
        expires 7d;
        access_log off;
    }

    location ~ /\. { deny all; }

    access_log /var/log/nginx/api.example.com.access.log;
    error_log  /var/log/nginx/api.example.com.error.log warn;
}

server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;
}
```

### Команды деплоя Python

```bash
# На сервере (первый раз):
# 1. Создать папку и venv
sudo mkdir -p /var/www/api
sudo chown -R www-data:www-data /var/www/api
cd /var/www/api
python3 -m venv venv

# 2. Загрузить код
rsync -avz --exclude='.git' --exclude='venv' --exclude='__pycache__' \
  ./api/ user@SERVER_IP:/var/www/api/

# 3. Установить зависимости
/var/www/api/venv/bin/pip install -r /var/www/api/requirements.txt

# 4. .env
scp ./.env.production user@SERVER_IP:/var/www/api/.env
ssh user@SERVER_IP "chmod 600 /var/www/api/.env"

# 5. Создать папки для логов
sudo mkdir -p /var/log/gunicorn
sudo chown www-data:www-data /var/log/gunicorn

# 6. FastAPI: запустить через systemd
sudo systemctl daemon-reload
sudo systemctl enable fastapi
sudo systemctl start fastapi
sudo systemctl status fastapi

# 7. Django: миграции + collectstatic
/var/www/api/venv/bin/python manage.py migrate --noinput
/var/www/api/venv/bin/python manage.py collectstatic --noinput

# 8. Zero-downtime reload:
sudo systemctl reload fastapi  # для Gunicorn: kill -HUP
```

### Logrotate для Gunicorn

```
# /etc/logrotate.d/gunicorn
/var/log/gunicorn/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ -f /run/fastapi.pid ] && kill -USR1 $(cat /run/fastapi.pid)
    endscript
}
```

---

## ЧАСТЬ 4: DOCKER COMPOSE ДЕПЛОЙ

### docker-compose.yml для полного стека

```yaml
# docker-compose.yml
version: '3.9'

services:
  # Frontend (Nginx + статика)
  frontend:
    image: nginx:alpine
    container_name: frontend
    volumes:
      - ./frontend/dist:/usr/share/nginx/html:ro
      - ./nginx/frontend.conf:/etc/nginx/conf.d/default.conf:ro
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
    networks:
      - app-network

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: backend
    restart: unless-stopped
    env_file: ./backend/.env
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    networks:
      - app-network
    # Порты НЕ открывать наружу - только через nginx

  # Telegram Bot
  bot:
    build:
      context: ./bot
      dockerfile: Dockerfile
    container_name: telegram-bot
    restart: unless-stopped
    env_file: ./bot/.env
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network

  # PostgreSQL
  db:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    networks:
      - app-network
    # ВАЖНО: порт наружу не открывать!

  # Redis (опционально: кеш, сессии, очереди)
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD} --appendonly yes
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "--pass", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
    networks:
      - app-network

  # Nginx reverse proxy (главный)
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/sites:/etc/nginx/sites-enabled:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
      - /var/www/certbot:/var/www/certbot:ro
    depends_on:
      - frontend
      - backend
    restart: unless-stopped
    networks:
      - app-network

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

networks:
  app-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### .env для docker-compose

```bash
# .env (chmod 600, не в git!)
POSTGRES_DB=myapp
POSTGRES_USER=myapp_user
POSTGRES_PASSWORD=very_strong_password_here
REDIS_PASSWORD=redis_strong_password
```

### Dockerfile для Node.js (multi-stage)

```dockerfile
# Dockerfile
# Stage 1: Build
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
# Если нужна сборка:
# RUN npm run build

# Stage 2: Production
FROM node:22-alpine AS production
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && adduser -S nodeuser -u 1001

COPY package*.json ./
RUN npm ci --production && npm cache clean --force

COPY --from=builder /app/src ./src
# COPY --from=builder /app/dist ./dist

# Не запускать от root
USER nodeuser

EXPOSE 3001

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3001/health', r => r.statusCode === 200 ? process.exit(0) : process.exit(1))"

CMD ["node", "src/index.js"]
```

### Dockerfile для Python/FastAPI (multi-stage)

```dockerfile
# Dockerfile
FROM python:3.12-slim AS base

RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
RUN adduser --disabled-password --gecos '' appuser

FROM base AS dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM base AS production
COPY --from=dependencies /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=dependencies /usr/local/bin /usr/local/bin
COPY . .

USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### Команды Docker деплоя

```bash
# Первый деплой
docker compose pull
docker compose up -d

# Обновление (zero-downtime для stateless сервисов)
docker compose pull backend
docker compose up -d --no-deps --build backend

# Логи
docker compose logs -f backend
docker compose logs --tail=100 bot

# Статус
docker compose ps
docker compose top

# Перезапуск одного сервиса
docker compose restart backend

# Остановить всё
docker compose down

# Остановить + удалить volumes (ОСТОРОЖНО: потеря данных БД)
docker compose down -v

# Бэкап PostgreSQL (из Docker)
docker compose exec db pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} | \
  gzip > /root/backups/db_$(date +%Y%m%d_%H%M%S).sql.gz
```

---

## ЧАСТЬ 5: CI/CD ЧЕРЕЗ SSH (deploy.sh)

### Скрипт деплоя deploy.sh

```bash
#!/bin/bash
# deploy.sh - Скрипт деплоя на VPS через SSH
# Использование: ./deploy.sh [frontend|backend|bot|all]

set -euo pipefail  # Останавливаться при ошибке

# ======== КОНФИГУРАЦИЯ ========
SERVER_USER="root"
SERVER_HOST="192.168.1.1"
SERVER_SSH_PORT="22"
SSH_KEY="~/.ssh/id_rsa"  # Или убрать, если пароль
DEPLOY_COMPONENT="${1:-all}"

# Пути
FRONTEND_LOCAL="./frontend/dist"
FRONTEND_REMOTE="/var/www/app"

BACKEND_LOCAL="./backend"
BACKEND_REMOTE="/var/www/api"

BOT_LOCAL="./bot"
BOT_REMOTE="/var/www/bot"

BACKUP_DIR="/root/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# ======== ФУНКЦИИ ========
log() { echo "[$(date '+%H:%M:%S')] $*"; }
err() { echo "[ERROR] $*" >&2; exit 1; }

ssh_cmd() {
    ssh -p "$SERVER_SSH_PORT" -i "$SSH_KEY" -o StrictHostKeyChecking=no \
        "${SERVER_USER}@${SERVER_HOST}" "$@"
}

rsync_upload() {
    local src="$1" dst="$2"
    shift 2
    rsync -avz --delete \
        -e "ssh -p $SERVER_SSH_PORT -i $SSH_KEY -o StrictHostKeyChecking=no" \
        "$@" \
        "${src}/" "${SERVER_USER}@${SERVER_HOST}:${dst}/"
}

# ======== ДЕПЛОЙ FRONTEND ========
deploy_frontend() {
    log "Деплой фронтенда..."

    # Бэкап текущей версии
    ssh_cmd "[ -d ${FRONTEND_REMOTE} ] && cp -r ${FRONTEND_REMOTE} ${BACKUP_DIR}/frontend_${TIMESTAMP} || true"

    # Загрузить новые файлы
    rsync_upload "$FRONTEND_LOCAL" "$FRONTEND_REMOTE"

    # Права
    ssh_cmd "sudo chown -R www-data:www-data ${FRONTEND_REMOTE} && sudo chmod -R 755 ${FRONTEND_REMOTE}"

    # Проверить и перезагрузить nginx
    ssh_cmd "sudo nginx -t && sudo systemctl reload nginx"

    log "Фронтенд задеплоен."
}

# ======== ДЕПЛОЙ BACKEND (Node.js + PM2) ========
deploy_backend() {
    log "Деплой бэкенда..."

    # Бэкап
    ssh_cmd "[ -d ${BACKEND_REMOTE} ] && cp -r ${BACKEND_REMOTE} ${BACKUP_DIR}/backend_${TIMESTAMP} || true"

    # Загрузить код (без node_modules и .env)
    rsync_upload "$BACKEND_LOCAL" "$BACKEND_REMOTE" \
        --exclude='node_modules' \
        --exclude='.env' \
        --exclude='.git' \
        --exclude='*.log'

    # Установить зависимости
    ssh_cmd "cd ${BACKEND_REMOTE} && npm ci --production"

    # Zero-downtime reload (если PM2 уже запущен)
    if ssh_cmd "pm2 list | grep -q 'api'"; then
        ssh_cmd "cd ${BACKEND_REMOTE} && pm2 reload api"
        log "PM2 reload выполнен (zero-downtime)."
    else
        ssh_cmd "cd ${BACKEND_REMOTE} && pm2 start ecosystem.config.js --env production && pm2 save"
        log "PM2 запущен."
    fi
}

# ======== ДЕПЛОЙ БОТА ========
deploy_bot() {
    log "Деплой Telegram-бота..."

    rsync_upload "$BOT_LOCAL" "$BOT_REMOTE" \
        --exclude='node_modules' \
        --exclude='.env' \
        --exclude='.git' \
        --exclude='data/'  # SQLite не перезаписывать!

    ssh_cmd "cd ${BOT_REMOTE} && npm ci --production"

    if ssh_cmd "pm2 list | grep -q 'telegram-bot'"; then
        ssh_cmd "pm2 restart telegram-bot"
    else
        ssh_cmd "cd ${BOT_REMOTE} && pm2 start bot.js --name telegram-bot && pm2 save"
    fi

    log "Бот задеплоен."
}

# ======== ROLLBACK ========
rollback() {
    local component="$1"
    local backup_path="${BACKUP_DIR}/${component}_${TIMESTAMP}"
    log "Rollback ${component}..."
    # Найти последний бэкап
    local last_backup
    last_backup=$(ssh_cmd "ls -t ${BACKUP_DIR}/${component}_* 2>/dev/null | head -1")
    if [ -z "$last_backup" ]; then
        err "Бэкапов не найдено для ${component}"
    fi
    ssh_cmd "cp -r ${last_backup}/ /var/www/${component}/"
    log "Rollback выполнен из: ${last_backup}"
}

# ======== ОСНОВНАЯ ЛОГИКА ========
# Создать папку для бэкапов
ssh_cmd "mkdir -p ${BACKUP_DIR}"

case "$DEPLOY_COMPONENT" in
    frontend)   deploy_frontend ;;
    backend)    deploy_backend ;;
    bot)        deploy_bot ;;
    rollback)   rollback "${2:-backend}" ;;
    all)
        deploy_frontend
        deploy_backend
        deploy_bot
        ;;
    *)
        echo "Использование: $0 [frontend|backend|bot|all|rollback <component>]"
        exit 1
        ;;
esac

log "Деплой завершён успешно."
```

---

## ЧАСТЬ 6: БЕЗОПАСНОСТЬ VPS

### Полная настройка безопасности (скрипт)

```bash
#!/bin/bash
# security-setup.sh - Первичная настройка безопасности Ubuntu VPS

set -euo pipefail

echo "=== UFW Firewall ==="
# ВАЖНО: сначала разрешить SSH!
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp comment 'SSH'
ufw allow 80/tcp comment 'HTTP'
ufw allow 443/tcp comment 'HTTPS'
# Опционально: нестандартный порт SSH
# ufw allow 2222/tcp comment 'SSH custom'
ufw --force enable
ufw status verbose

echo "=== fail2ban ==="
apt install -y fail2ban

cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime  = 86400    ; 24 часа
findtime = 600      ; 10 минут
maxretry = 3
banaction = ufw
banaction_allports = ufw
backend = systemd

[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
maxretry = 3
bantime = 86400

[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 5

[nginx-botsearch]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 2
findtime = 60
bantime = 86400
EOF

systemctl enable fail2ban
systemctl restart fail2ban
fail2ban-client status

echo "=== SSH hardening ==="
# Отключить root логин по паролю (использовать ключи)
# sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config
# sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
# systemctl reload sshd

echo "=== Автоматические обновления безопасности ==="
apt install -y unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades

echo "=== Готово ==="
```

### Nginx Security Headers (глобальный конфиг)

```nginx
# /etc/nginx/snippets/security-headers.conf
# Подключать: include snippets/security-headers.conf;

# Запрет MIME-sniffing
add_header X-Content-Type-Options "nosniff" always;

# Запрет clickjacking
add_header X-Frame-Options "SAMEORIGIN" always;

# XSS Protection (legacy, но нужен для старых браузеров)
add_header X-XSS-Protection "1; mode=block" always;

# HTTPS на год вперёд
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

# Referrer Policy
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# Запрет отслеживания
add_header Permissions-Policy "camera=(), microphone=(), geolocation=(), payment=()" always;

# Скрыть версию nginx
server_tokens off;
```

```nginx
# /etc/nginx/snippets/block-sensitive.conf
# Блокировать доступ к чувствительным файлам и путям

# .git, .env, .htaccess
location ~ /\.(git|env|htaccess|htpasswd) {
    deny all;
    return 404;
}

# Бэкапы
location ~ /(backups?|backup|\.backup) {
    deny all;
    return 404;
}

# Конфиги
location ~ \.(conf|config|cfg|ini|log|bak|old|sql|sh|py)$ {
    deny all;
    return 404;
}

# node_modules
location ~ /node_modules {
    deny all;
    return 404;
}

# package.json, composer.json
location ~ /(package\.json|composer\.json|composer\.lock|package-lock\.json) {
    deny all;
    return 404;
}
```

### SSL с оценкой A+ (SSL Labs)

```nginx
# /etc/nginx/snippets/ssl-params.conf

# Только TLS 1.2 и 1.3
ssl_protocols TLSv1.2 TLSv1.3;

# Предпочитать шифры сервера
ssl_prefer_server_ciphers on;

# Сильные шифры
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256;

# Сессионный кеш
ssl_session_cache shared:SSL:50m;
ssl_session_timeout 1d;
ssl_session_tickets off;  # PFS

# OCSP Stapling
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

### Certbot: получение и автопродление

```bash
# Установка certbot
sudo apt install -y certbot python3-certbot-nginx

# Получить сертификат (Nginx плагин автонастраивает конфиг)
sudo certbot --nginx -d example.com -d www.example.com \
  --non-interactive --agree-tos --email admin@example.com

# Получить для нескольких доменов
sudo certbot --nginx \
  -d app.example.com \
  -d api.example.com \
  --non-interactive --agree-tos --email admin@example.com

# Проверить автопродление
sudo certbot renew --dry-run

# Принудительное продление
sudo certbot renew --force-renewal

# Статус сертификатов
sudo certbot certificates

# Автопродление через systemd timer (уже встроен в certbot)
sudo systemctl status certbot.timer
```

---

## ЧАСТЬ 7: DNS И ДОМЕНЫ

### A-записи (настройка в панели DNS)

```
# Основные записи:
@         A    192.168.1.1   (основной домен)
www       A    192.168.1.1   (www.example.com)
api       A    192.168.1.1   (api.example.com)
bot       A    192.168.1.1   (bot.example.com - если нужен вебхук)

# TTL: 300 (5 минут) при активной настройке
# TTL: 3600 (1 час) для продакшена
```

### Nginx: несколько доменов на одном IP

```nginx
# /etc/nginx/nginx.conf (фрагмент http блока)
http {
    # Включить все конфиги из sites-enabled
    include /etc/nginx/sites-enabled/*.conf;

    # Дефолтный сервер (отклоняет запросы не на наши домены)
    server {
        listen 80 default_server;
        listen 443 ssl default_server;
        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
        server_name _;
        return 444;  # Закрыть соединение без ответа
    }
}
```

### Cloudflare (опционально)

Преимущества: DDoS защита, CDN, скрытие реального IP, бесплатный SSL прокси.

```
# Настройка:
# 1. Добавить домен в Cloudflare
# 2. Сменить NS-серверы у регистратора на Cloudflare
# 3. A-запись: IP сервера, Proxy status: Proxied (оранжевое облако)
# 4. SSL/TLS mode: Full (strict) - если на сервере есть Let's Encrypt
# 5. Security > WAF: включить Managed Ruleset

# Nginx: получать реальный IP из CF-Connecting-IP заголовка
# /etc/nginx/conf.d/cloudflare-real-ip.conf
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
# ... (все IP диапазоны Cloudflare)
real_ip_header CF-Connecting-IP;
```

---

## ЧАСТЬ 8: УСТАНОВКА ЗАВИСИМОСТЕЙ (первый запуск)

```bash
#!/bin/bash
# server-init.sh - Первоначальная настройка сервера Ubuntu 22.04/24.04

set -euo pipefail

echo "=== Обновление системы ==="
apt update && apt upgrade -y
apt install -y curl wget git unzip htop net-tools

echo "=== Nginx ==="
apt install -y nginx
systemctl enable nginx
systemctl start nginx

echo "=== Node.js 22 LTS ==="
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
node --version && npm --version

echo "=== PM2 ==="
npm install -g pm2
pm2 --version

echo "=== Python 3 + pip ==="
apt install -y python3 python3-pip python3-venv python3-dev

echo "=== Certbot ==="
apt install -y certbot python3-certbot-nginx

echo "=== Docker ==="
curl -fsSL https://get.docker.com | sh
systemctl enable docker
systemctl start docker
# Добавить пользователя в группу docker (если не root)
# usermod -aG docker $USER

echo "=== Docker Compose ==="
# Уже включён в Docker Desktop и как плагин
docker compose version

echo "=== Папки ==="
mkdir -p /var/www
mkdir -p /root/backups
mkdir -p /var/log/pm2
mkdir -p /var/log/gunicorn

echo "=== Готово! ==="
echo "Следующий шаг: запустить security-setup.sh"
```

---

## ЧЕКЛИСТ БЕЗОПАСНОСТИ

### Обязательный минимум

```
[ ] UFW включён: только 22, 80, 443 открыты
[ ] fail2ban установлен и настроен (SSH + nginx jails)
[ ] Порты приложений (3001, 8000) НЕ открыты наружу
[ ] .env файлы имеют права 600 (chmod 600)
[ ] .env файлы НЕ в git репозитории (.gitignore)
[ ] Nginx: server_tokens off
[ ] Nginx: security headers добавлены
[ ] Nginx: блокировка .git, .env, backups
[ ] SSL: только TLS 1.2 и 1.3
[ ] SSL: HSTS включён
[ ] Certbot: автопродление работает (certbot renew --dry-run)
[ ] Приложения не запускаются от root (www-data / отдельный пользователь)
[ ] Docker: порты БД НЕ открыты наружу (только 127.0.0.1)
[ ] SSH: ключи вместо паролей (рекомендуется)
```

### Проверка безопасности

```bash
# Проверить открытые порты
sudo ss -tlnp

# Проверить UFW
sudo ufw status numbered

# Проверить fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Проверить SSL (онлайн)
# https://www.ssllabs.com/ssltest/

# Проверить security headers (онлайн)
# https://securityheaders.com/

# Проверить nginx конфиг
sudo nginx -t

# Просмотр попыток взлома
sudo journalctl -u ssh --since "24 hours ago" | grep "Failed\|Invalid" | tail -20
sudo fail2ban-client status sshd | grep "Banned IP"
```

---

## ЧАСТЬ 9: МОНИТОРИНГ И ЛОГИ

### Настройка logrotate для Nginx

```
# /etc/logrotate.d/nginx (обычно уже есть)
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        if [ -f /var/run/nginx.pid ]; then
            kill -USR1 `cat /var/run/nginx.pid`
        fi
    endscript
}
```

### Мониторинг доступности (cron)

```bash
# /root/scripts/health-check.sh
#!/bin/bash
URLS=("https://app.example.com" "https://api.example.com/health")
TELEGRAM_TOKEN="YOUR_BOT_TOKEN"
TELEGRAM_CHAT_ID="YOUR_CHAT_ID"

for url in "${URLS[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 "$url")
    if [ "$status" != "200" ]; then
        message="ALERT: $url вернул $status"
        curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
            -d "chat_id=${TELEGRAM_CHAT_ID}" \
            -d "text=${message}" > /dev/null
    fi
done

# Добавить в cron (каждые 5 минут):
# */5 * * * * /root/scripts/health-check.sh
```

### Бэкап PostgreSQL (cron)

```bash
#!/bin/bash
# /root/scripts/backup-db.sh
DB_NAME="myapp"
DB_USER="postgres"
BACKUP_DIR="/root/backups/db"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
KEEP_DAYS=7

mkdir -p "$BACKUP_DIR"

# Бэкап
pg_dump -U "$DB_USER" "$DB_NAME" | gzip > "${BACKUP_DIR}/db_${TIMESTAMP}.sql.gz"

# Удалить старые бэкапы
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$KEEP_DAYS -delete

echo "Бэкап: ${BACKUP_DIR}/db_${TIMESTAMP}.sql.gz"

# Добавить в cron:
# 0 3 * * * /root/scripts/backup-db.sh
```

---

## ЧАСТЫЕ ОШИБКИ И РЕШЕНИЯ

| Ошибка | Причина | Решение |
|---|---|---|
| `502 Bad Gateway` | Nginx не может достучаться до backend | Проверить `pm2 list` / `systemctl status`, порт, upstream в nginx |
| `403 Forbidden` | Неправильные права на файлы | `chown -R www-data:www-data /var/www/` + `chmod 755` |
| `certbot: domain not pointing` | DNS не настроен | Проверить A-запись: `dig app.example.com` |
| `CORS error` | Backend не отдаёт CORS headers | Добавить `Access-Control-Allow-Origin` в бэкенд или nginx |
| `nginx: [emerg] unknown directive` | Синтаксическая ошибка конфига | `nginx -t` покажет строку ошибки |
| `PM2 not found` | PM2 не в PATH для системных юзеров | Использовать полный путь `/usr/local/bin/pm2` |
| `connection refused` порт 5432 | PostgreSQL слушает только localhost | Правильно, порт не должен быть открыт, использовать Docker network |
| `ssl_stapling: failed` | Неправильный resolver или нет chain.pem | Добавить `ssl_trusted_certificate /etc/letsencrypt/live/domain/chain.pem` |
| SPA 404 на обновление страницы | Нет `try_files $uri $uri/ /index.html` | Добавить в nginx location / блок |
| `pm2 startup` не работает | Не выполнена команда которую выдал PM2 | Скопировать и выполнить команду которую выдаёт `pm2 startup` |

---

## ШАБЛОНЫ .env

### .env для Node.js бэкенда

```bash
# /var/www/api/.env (chmod 600)
NODE_ENV=production
PORT=3001
HOST=127.0.0.1

DATABASE_URL=postgresql://user:password@127.0.0.1:5432/myapp
REDIS_URL=redis://:password@127.0.0.1:6379

JWT_SECRET=minimum_32_chars_random_string_here
JWT_ACCESS_EXPIRES=15m
JWT_REFRESH_EXPIRES=7d

CORS_ORIGINS=https://app.example.com,https://www.example.com
```

### .env для Telegram-бота

```bash
# /var/www/bot/.env (chmod 600)
BOT_TOKEN=1234567890:AAHxxx
OPENROUTER_API_KEY=sk-or-xxx
AI_MODEL=deepseek/deepseek-chat
ADMIN_CHAT_ID=123456789
DATABASE_PATH=./data/bot.db
```

### .env для Python FastAPI

```bash
# /var/www/api/.env (chmod 600)
DATABASE_URL=postgresql+asyncpg://user:password@127.0.0.1:5432/myapp
SECRET_KEY=minimum_32_chars_random_string_here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=15
ALLOWED_ORIGINS=["https://app.example.com"]
DEBUG=false
```
