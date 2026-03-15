---
name: telegram-bot-builder
description: "Создаёт красивые Telegram-боты на Node.js с UI-навигацией, цветными кнопками и стримингом. Use when пользователь просит: создай бота, telegram bot, телеграм бот, бот для телеграма, сделай бота, чат-бот, chatbot, бот с кнопками, бот с AI, бот с меню, inline bot, telegram mini app, бот для бизнеса, бот поддержки, бот продаж, автоответчик телеграм, bot builder, сделай телеграм бота, бот с квизом, бот с формой."
---

# Telegram Bot Builder

Скилл для создания продакшен-готовых Telegram-ботов на Node.js с красивым UI, edit-in-place навигацией, цветными кнопками (Bot API 9.4+) и интеграцией AI.

## Стек по умолчанию

| Компонент | Технология |
|---|---|
| Runtime | Node.js |
| Telegram | node-telegram-bot-api (polling) или raw HTTPS API |
| AI (опционально) | OpenRouter API (DeepSeek Chat / Claude / GPT) |
| БД (опционально) | better-sqlite3 (WAL mode) |
| Деплой | PM2 на VPS |

## КРИТИЧНЫЕ ПРАВИЛА

1. **Всегда уточнять задачу перед генерацией.** Спросить: тип бота, функции, нужен ли AI, нужна ли БД.
2. **Edit-in-place навигация.** Никогда не удалять и пересылать сообщения. Только `editMessageText` для callback-кнопок.
3. **Цветные кнопки (Bot API 9.4+).** Использовать поле `style` для визуального разделения действий.
4. **HTML-форматирование.** Всегда `parse_mode: 'HTML'`. Не использовать Markdown.
5. **answerCallbackQuery обязательно.** Каждый callback_query подтверждать, иначе "часики" на кнопке.
6. **Rate limits.** Не более 1 msg/sec в один чат, 20 msg/min в группу. Массовые рассылки: пауза 1с каждые 25 сообщений.
7. **Graceful shutdown.** Всегда закрывать БД и polling при SIGINT/SIGTERM.
8. **Обработка 403.** При блокировке ботом пользователем - помечать в БД, не повторять отправку.
9. **Выходная папка:** `~/outputs/<bot-name>/`

## Workflow

### Шаг 0: Уточнение задачи

Спросить у пользователя:

**Вопрос 1 - Тип бота:**
- Информационный (меню, FAQ, контакты)
- Продажи / лидогенерация (квиз, формы, CRM)
- AI-консультант (чат с AI, стриминг ответов)
- Утилитарный (уведомления, мониторинг, автоматизация)
- Mini App (веб-приложение внутри Telegram)
- Комбинированный

**Вопрос 2 - Нужен ли AI-чат?** Если да - какая модель (DeepSeek, Claude, GPT) через OpenRouter.

**Вопрос 3 - Нужна ли БД?** Если да - better-sqlite3 для хранения пользователей, лидов, аналитики.

**Вопрос 4 - Админ-функции:** рассылки, статистика, модерация?

### Шаг 1: Архитектура

Определить структуру файлов:

```
bot-name/
  bot.js          # Основной файл бота
  streaming.js    # AI-стриминг (если нужен AI)
  database.js     # SQLite модуль (если нужна БД)
  package.json
  .env            # Токены и настройки
  .gitignore
  data/           # Папка для БД (в .gitignore)
```

### Шаг 2: Генерация кода

Следовать паттернам ниже.

### Шаг 3: Тестирование

- Проверить синтаксис: `node -c bot.js`
- Предложить пользователю протестировать основные сценарии

## Telegram Bot API - справочник

### Цветные кнопки (Bot API 9.4+)

Поле `style` для `InlineKeyboardButton` и `KeyboardButton`:

| Значение | Цвет | Когда использовать |
|---|---|---|
| `"success"` | Зелёный | CTA, подтверждение, основное действие |
| `"primary"` | Синий | Навигация, информационные кнопки |
| `"danger"` | Красный | Отмена, удаление, внешние ссылки |
| (без style) | Серый | Стандартные/второстепенные кнопки |

```javascript
const keyboard = [
    [
        { text: 'Услуги', callback_data: 'services', style: 'success' },
        { text: 'О нас', callback_data: 'about', style: 'primary' },
    ],
    [{ text: 'Отменить', callback_data: 'cancel', style: 'danger' }],
];
```

### HTML-форматирование

Поддерживаемые теги (parse_mode: 'HTML'):

| Тег | Результат |
|---|---|
| `<b>text</b>` | **Жирный** |
| `<i>text</i>` | *Курсив* |
| `<u>text</u>` | Подчёркнутый |
| `<s>text</s>` | ~~Зачёркнутый~~ |
| `<code>text</code>` | `Моноширинный` |
| `<pre>text</pre>` | Блок кода |
| `<a href="URL">text</a>` | Ссылка |
| `<tg-spoiler>text</tg-spoiler>` | Спойлер |
| `<blockquote>text</blockquote>` | Цитата |

### Ключевые методы API

```
sendMessage(chat_id, text, {parse_mode, reply_markup})
editMessageText(text, {chat_id, message_id, parse_mode, reply_markup})
answerCallbackQuery(callback_query_id, {text, show_alert})
sendPhoto(chat_id, photo, {caption, parse_mode, reply_markup})
sendDocument(chat_id, document, {caption})
sendChatAction(chat_id, 'typing')  // Индикатор набора
setMyCommands([{command, description}])  // Меню команд
```

### Лимиты

| Что | Лимит |
|---|---|
| Текст сообщения | 4096 символов |
| callback_data | 64 байта |
| Подпись к медиа | 1024 символа |
| Файл (загрузка) | 50 MB |
| Файл (скачивание) | 20 MB |
| Медиагруппа | 2-10 элементов |
| Команды бота | до 100 |
| input_field_placeholder | 64 символа |

## Паттерны (обязательные)

### Паттерн 1: Edit-in-place навигация

```javascript
// Стек навигации для каждого чата
const navHistory = new Map();

function pushNav(chatId, screen) {
    if (!navHistory.has(chatId)) navHistory.set(chatId, []);
    const stack = navHistory.get(chatId);
    if (stack[stack.length - 1] !== screen) stack.push(screen);
    if (stack.length > 10) stack.shift();
}

function popNav(chatId) {
    const stack = navHistory.get(chatId);
    if (!stack || stack.length <= 1) return 'home';
    stack.pop();
    return stack[stack.length - 1] || 'home';
}

// Кнопки навигации (всегда внизу каждого экрана)
const NAV_BACK = [
    { text: '\u25C0 Назад', callback_data: 'go_back', style: 'primary' },
    { text: '\u{1F3E0} В начало', callback_data: 'home', style: 'success' },
];
```

### Паттерн 2: Экранная функция

```javascript
// Каждый экран - функция, принимающая chatId и msgId
// msgId = null для команд (/start), message_id для callback
async function showScreen(chatId, msgId, text, keyboard) {
    if (msgId) {
        // Callback: редактируем существующее сообщение
        try {
            // Эффект перехода: сворачивание
            await editMessage(chatId, msgId, '\u23F3', []);
            await delay(400);
        } catch (e) {}
        try {
            await editMessage(chatId, msgId, text, keyboard);
        } catch (e) {
            // Fallback: новое сообщение
            await sendMessage(chatId, text, keyboard);
        }
    } else {
        // Команда: новое сообщение
        await sendMessage(chatId, text, keyboard);
    }
}
```

### Паттерн 3: Обработка callback_query

```javascript
bot.on('callback_query', async (query) => {
    // 1. ОБЯЗАТЕЛЬНО подтвердить
    await bot.answerCallbackQuery(query.id);

    const chatId = query.message.chat.id;
    const msgId = query.message.message_id;
    const data = query.data;

    // 2. Навигация
    if (data === 'go_back') {
        const screen = popNav(chatId);
        return showScreenByName(chatId, msgId, screen, query.from);
    }
    if (data === 'home') {
        return showHome(chatId, msgId, query.from);
    }

    // 3. Бизнес-логика
    if (data === 'services') return showServices(chatId, msgId);
    if (data === 'contact') return startContactForm(chatId, msgId);
    // ...
});
```

### Паттерн 4: Квиз / воронка

```javascript
const quizzes = new Map(); // chatId -> {step, answers...}

function startQuiz(chatId, msgId) {
    quizzes.set(chatId, { step: 'q1' });
    showScreen(chatId, msgId, '<b>Вопрос 1/4:</b> ...', [
        [{ text: 'Вариант A', callback_data: 'q1_a' }],
        [{ text: 'Вариант B', callback_data: 'q1_b' }],
        [{ text: 'Отменить', callback_data: 'cancel_quiz', style: 'danger' }],
    ]);
}

// В callback handler:
if (data.startsWith('q1_')) {
    const quiz = quizzes.get(chatId);
    if (!quiz || quiz.step !== 'q1') return;
    quiz.answer1 = data.replace('q1_', '');
    quiz.step = 'q2';
    showQuizStep2(chatId, msgId);
}
```

### Паттерн 5: Сбор контактов (текстовый ввод)

```javascript
const contactForms = new Map(); // chatId -> {step, name?, contact?}

function startContactForm(chatId, msgId) {
    contactForms.set(chatId, { step: 'name' });
    showScreen(chatId, msgId, 'Как вас зовут?\n\n<i>Напишите в чат.</i>', [
        [{ text: 'Отменить', callback_data: 'cancel_form', style: 'danger' }],
    ]);
}

// В message handler (перед AI-чатом):
bot.on('message', (msg) => {
    const form = contactForms.get(msg.chat.id);
    if (form) {
        if (form.step === 'name') {
            form.name = msg.text;
            form.step = 'contact';
            // Спросить контакт...
            return;
        }
        if (form.step === 'contact') {
            form.contactInfo = msg.text;
            contactForms.delete(msg.chat.id);
            // Отправить лид админу...
            return;
        }
    }
    // ... далее AI или другая логика
});
```

### Паттерн 6: AI-стриминг

```javascript
// streaming.js - стриминг ответов AI через OpenRouter
// Ключевые принципы:
// 1. Отправить начальное сообщение с "Думаю..."
// 2. Запросить OpenRouter с stream: true
// 3. Парсить SSE-события, накапливать текст
// 4. Каждые N мс редактировать сообщение с новым текстом
// 5. Финальное редактирование с полным ответом
// 6. Санитизация: убирать HTML-теги и markdown из AI-ответов
```

### Паттерн 7: SQLite база данных

```javascript
// database.js
const Database = require('better-sqlite3');
const db = new Database('data/bot.db');
db.pragma('journal_mode = WAL');
db.pragma('foreign_keys = ON');

// Таблицы: users, leads, broadcasts
// Prepared statements для производительности
// upsertUser(from) - ON CONFLICT DO UPDATE
// saveLead(data) - сохранение лидов
// getSubscribedUsers() - для рассылок
// getStats() - статистика для админа
```

### Паттерн 8: Админ-команды

```javascript
const ADMIN_CHAT_ID = parseInt(process.env.ADMIN_CHAT_ID);

// /stats - статистика (только админ)
bot.onText(/\/stats/, (msg) => {
    if (msg.chat.id !== ADMIN_CHAT_ID) return;
    const stats = database.getStats();
    bot.sendMessage(msg.chat.id, `Пользователей: ${stats.users}\n...`);
});

// /broadcast <текст> - рассылка (только админ)
bot.onText(/\/broadcast (.+)/s, async (msg, match) => {
    if (msg.chat.id !== ADMIN_CHAT_ID) return;
    // Отправка с паузой 1с каждые 25 сообщений
    // Обработка 403 -> markBlocked()
});
```

### Паттерн 9: Rate limiting

```javascript
const rateLimits = new Map();
const RATE_PER_MIN = 5;
const DAILY_LIMIT = 30;

function checkRateLimit(chatId) {
    // Per-minute: фильтровать timestamps > 1 минуты назад
    // Daily: счётчик с ключом YYYY-MM-DD
    // Возвращать { ok: true } или { ok: false, reason: '...' }
    // Админ освобождён от лимитов
}
```

### Паттерн 10: System prompt для AI

```javascript
const SYSTEM_PROMPT = `Ты - [роль]. Твое имя: [имя].

ИДЕНТИЧНОСТЬ:
- Ты НЕ ChatGPT, НЕ GPT, НЕ другая модель.
- Если спрашивают "кто ты" - отвечай своим именем и ролью.
- Не раскрывай техническую реализацию.

ТЕМАТИКА:
- Отвечай ТОЛЬКО на вопросы по теме [тема].
- Вне темы - вежливо отказывай и предлагай вернуться к теме.

ПРАВИЛА:
- Кратко (3-5 предложений), по делу
- Не используй длинные тире, эмодзи, markdown
- Говори на русском`;
```

## .env шаблон

```
BOT_TOKEN=
OPENROUTER_API_KEY=
AI_MODEL=deepseek/deepseek-chat
ADMIN_CHAT_ID=
STREAM_INTERVAL_MS=800
MAX_HISTORY=10
MAX_USER_MSG_LENGTH=1000
RATE_LIMIT_PER_MIN=5
DAILY_LIMIT=30
```

## Mini Apps (Web Apps)

Если пользователю нужен Mini App:

```javascript
// Запуск через inline-кнопку
[{ text: 'Открыть приложение', web_app: { url: 'https://example.com/app' } }]

// Запуск через keyboard-кнопку
bot.sendMessage(chatId, 'Нажмите для запуска:', {
    reply_markup: {
        keyboard: [[{ text: 'Открыть App', web_app: { url: 'https://example.com/app' } }]],
        resize_keyboard: true,
    }
});
```

Mini App получает:
- `Telegram.WebApp.initData` - данные пользователя
- `Telegram.WebApp.themeParams` - тема (dark/light)
- `Telegram.WebApp.MainButton` - кнопка внизу
- `Telegram.WebApp.BackButton` - кнопка назад
- `Telegram.WebApp.CloudStorage` - хранилище (1024 элемента)

## Деплой

```bash
# На VPS (PM2):
npm install
mkdir -p data
pm2 start bot.js --name "bot-name"
pm2 save
pm2 startup

# Мониторинг:
pm2 logs bot-name
pm2 monit
```
