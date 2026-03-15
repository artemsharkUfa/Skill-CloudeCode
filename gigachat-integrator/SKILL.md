---
name: gigachat-integrator
description: "Интеграция Sber GigaChat API в проекты (Node.js, Python, PHP). Use when пользователь просит: gigachat, гигачат, sber ai, сбер нейросеть, подключи gigachat, замени модель на gigachat, интеграция gigachat, gigachat api, gigachat sdk, gigachat embedding, gigachat vision, гигачат апи, sber llm, gigachat function calling, gigachat streaming, gigachat oauth."
---

# GigaChat Integrator

Скилл для интеграции Sber GigaChat API в проекты. Поддержка всех моделей линейки GigaChat 2, embeddings, vision, function calling, streaming.

## Линейка моделей (актуально 2026)

### Генерация текста

| Модель | API-имя | Контекст | Input | Output | Цена/1M токенов |
|---|---|---|---|---|---|
| **GigaChat 2 Max** | `GigaChat-2-Max` | 128K | текст, изображения, аудио | текст, изображения | 650 руб |
| **GigaChat 2 Pro** | `GigaChat-2-Pro` | 128K | текст, изображения, аудио | текст, изображения | 500 руб |
| **GigaChat 2 Lite** | `GigaChat-2-Lite` | 128K | текст | текст | 65 руб |

Модели 1-го поколения (GigaChat, GigaChat-Pro, GigaChat-Max) устарели и перенаправляются на 2-е поколение.

### Embeddings

| Модель | API-имя | Контекст | Измерения | Цена/1M токенов |
|---|---|---|---|---|
| **EmbeddingsGigaR** | `EmbeddingsGigaR` | 4096 | 2560 | 40 руб |
| **Embeddings-3B** | `Embeddings-3B-2025-09` | - | - | 40 руб |
| **Embeddings-2** | `Embeddings-2` | 512 | 1024 | 40 руб |
| **Embeddings** | `Embeddings` | 512 | 1024 | 40 руб |

### Выбор модели

| Задача | Модель |
|---|---|
| Сложные задачи, аналитика, длинный контекст | GigaChat-2-Max |
| Баланс цена/качество, чат-виджеты | GigaChat-2-Pro |
| Массовые задачи, FAQ-боты, простые ответы | GigaChat-2-Lite |
| RAG, семантический поиск | EmbeddingsGigaR |

### Бенчмарки GigaChat-2-Max

| Метрика | Значение |
|---|---|
| MMLU (5-shot) | 0.86 |
| ruMMLU (5-shot) | 0.80 |
| Arena-Hard RU | 0.84 |
| MATH (4-shot) | 0.78 |
| MBPP (код, 0-shot) | 0.89 |
| IFEval RU (инструкции) | 0.83 |

## КРИТИЧНЫЕ ПРАВИЛА

1. **SSL-сертификаты.** GigaChat использует самоподписанные сертификаты Минцифры РФ. Всегда `verify_ssl_certs=False` (Python SDK) или `rejectUnauthorized: false` (Node.js).
2. **Токен живёт 30 минут.** Реализовать автообновление OAuth-токена. Не кешировать дольше 29 минут.
3. **Scope правильный.** `GIGACHAT_API_PERS` (физлица), `GIGACHAT_API_B2B` (юрлица предоплата), `GIGACHAT_API_CORP` (юрлица постоплата).
4. **Контент-фильтрация.** GigaChat блокирует запрещённые темы. `finish_reason: "blacklist"` - обрабатывать gracefully.
5. **Лимит потоков.** Физлица: 1 параллельный поток. Юрлица: 10 (расширяется по запросу).
6. **Vision ограничения.** Максимум 1 изображение на сообщение, 10 на диалог. Только Pro и Max.
7. **HTML-разметка в ответах.** GigaChat может возвращать markdown. Санитизировать если нужен plain text.

## Аутентификация (OAuth 2.0)

### Base URLs

```
OAuth:  https://ngw.devices.sberbank.ru:9443/api/v2/oauth
API:    https://gigachat.devices.sberbank.ru/api/v1/
```

### Получение токена

1. В Studio (developers.sber.ru) создать проект, получить Client ID + Client Secret
2. `Authorization Key = Base64(ClientID:ClientSecret)`
3. POST-запрос:

```http
POST https://ngw.devices.sberbank.ru:9443/api/v2/oauth
Content-Type: application/x-www-form-urlencoded
Authorization: Basic <Authorization_Key>
RqUID: <уникальный UUID v4>

scope=GIGACHAT_API_PERS
```

4. Ответ: `{ "access_token": "...", "expires_at": 1234567890 }`
5. Токен действует **30 минут**

### Автообновление токена (паттерн)

```javascript
let accessToken = null;
let tokenExpiresAt = 0;

async function getToken() {
    if (accessToken && Date.now() < tokenExpiresAt - 60000) return accessToken;

    const authKey = Buffer.from(`${CLIENT_ID}:${CLIENT_SECRET}`).toString('base64');
    const { v4: uuid } = require('uuid');

    const res = await fetch('https://ngw.devices.sberbank.ru:9443/api/v2/oauth', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': `Basic ${authKey}`,
            'RqUID': uuid(),
        },
        body: 'scope=GIGACHAT_API_PERS',
        // ВАЖНО: самоподписанный сертификат
        agent: new (require('https').Agent)({ rejectUnauthorized: false }),
    });

    const data = await res.json();
    accessToken = data.access_token;
    tokenExpiresAt = data.expires_at; // unix ms
    return accessToken;
}
```

## API эндпоинты

| Метод | Эндпоинт | Назначение |
|---|---|---|
| POST | `/api/v2/oauth` | Получение access token |
| GET | `/api/v1/models` | Список моделей |
| POST | `/api/v1/chat/completions` | Генерация текста (chat) |
| POST | `/api/v1/embeddings` | Генерация embeddings |
| POST | `/api/v1/tokens/count` | Подсчёт токенов |
| POST | `/api/v1/files` | Загрузка файла |
| GET | `/api/v1/files/{file_id}/content` | Скачивание файла |
| DELETE | `/api/v1/files/{file_id}` | Удаление файла |
| GET | `/api/v1/balance` | Проверка баланса |

## Интеграция: Node.js (raw API)

### Chat Completion

```javascript
const https = require('https');

const agent = new https.Agent({ rejectUnauthorized: false });

async function chatCompletion(messages, options = {}) {
    const token = await getToken();
    const body = {
        model: options.model || 'GigaChat-2-Max',
        messages,
        temperature: options.temperature ?? 0.7,
        max_tokens: options.max_tokens ?? 1024,
        stream: options.stream ?? false,
    };

    if (options.functions) body.functions = options.functions;
    if (options.function_call) body.function_call = options.function_call;

    const res = await fetch('https://gigachat.devices.sberbank.ru/api/v1/chat/completions', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`,
        },
        body: JSON.stringify(body),
        agent,
    });

    return await res.json();
}
```

### Streaming

```javascript
async function chatStream(messages, onChunk, options = {}) {
    const token = await getToken();
    const body = {
        model: options.model || 'GigaChat-2-Max',
        messages,
        stream: true,
        update_interval: options.updateInterval ?? 0,
    };

    const res = await fetch('https://gigachat.devices.sberbank.ru/api/v1/chat/completions', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`,
        },
        body: JSON.stringify(body),
        agent,
    });

    const reader = res.body.getReader();
    const decoder = new TextDecoder();
    let buffer = '';

    while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        buffer += decoder.decode(value, { stream: true });
        const lines = buffer.split('\n');
        buffer = lines.pop();

        for (const line of lines) {
            if (!line.startsWith('data: ')) continue;
            const data = line.slice(6);
            if (data === '[DONE]') return;

            try {
                const chunk = JSON.parse(data);
                const content = chunk.choices?.[0]?.delta?.content;
                if (content) onChunk(content);
            } catch (e) {}
        }
    }
}
```

### Vision (анализ изображений)

```javascript
// Только GigaChat-2-Pro и GigaChat-2-Max
const messages = [
    {
        role: 'user',
        content: [
            { type: 'text', text: 'Что изображено на картинке?' },
            { type: 'image_url', image_url: { url: 'data:image/jpeg;base64,...' } },
        ],
    },
];

const response = await chatCompletion(messages, { model: 'GigaChat-2-Max' });
```

### Function Calling

```javascript
const functions = [
    {
        name: 'get_weather',
        description: 'Получить погоду в городе',
        parameters: {
            type: 'object',
            properties: {
                city: { type: 'string', description: 'Название города' },
            },
            required: ['city'],
        },
    },
];

const response = await chatCompletion(messages, {
    functions,
    function_call: 'auto',
});

// Если модель вызвала функцию:
// response.choices[0].finish_reason === 'function_call'
// response.choices[0].message.function_call.name
// response.choices[0].message.function_call.arguments (JSON string)
```

### Embeddings

```javascript
async function getEmbeddings(texts, model = 'EmbeddingsGigaR') {
    const token = await getToken();
    const res = await fetch('https://gigachat.devices.sberbank.ru/api/v1/embeddings', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`,
        },
        body: JSON.stringify({ model, input: texts }),
        agent,
    });
    return await res.json();
}
```

## Интеграция: Python SDK

```python
from gigachat import GigaChat

client = GigaChat(
    credentials="<Base64_authorization_key>",
    scope="GIGACHAT_API_PERS",
    model="GigaChat-2-Max",
    verify_ssl_certs=False,
)

# Chat
response = client.chat("Привет, расскажи о себе")
print(response.choices[0].message.content)

# Streaming
for chunk in client.stream("Напиши план"):
    print(chunk.choices[0].delta.content, end="", flush=True)

# Embeddings
result = client.embeddings(["Текст для векторизации"], model="EmbeddingsGigaR")

# Token count
count = client.tokens_count(["Сколько здесь токенов?"])

# Balance
balance = client.get_balance()
```

## Интеграция: PHP (REST API)

```php
function getGigaChatToken($authKey, $scope = 'GIGACHAT_API_PERS') {
    $ch = curl_init('https://ngw.devices.sberbank.ru:9443/api/v2/oauth');
    curl_setopt_array($ch, [
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => 'scope=' . $scope,
        CURLOPT_HTTPHEADER => [
            'Content-Type: application/x-www-form-urlencoded',
            'Authorization: Basic ' . $authKey,
            'RqUID: ' . wp_generate_uuid4(), // или другой UUID v4
        ],
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_SSL_VERIFYPEER => false, // самоподписанный сертификат
    ]);
    $result = json_decode(curl_exec($ch), true);
    curl_close($ch);
    return $result['access_token'];
}

function gigaChatChat($token, $messages, $model = 'GigaChat-2-Max') {
    $ch = curl_init('https://gigachat.devices.sberbank.ru/api/v1/chat/completions');
    curl_setopt_array($ch, [
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => json_encode([
            'model' => $model,
            'messages' => $messages,
            'temperature' => 0.7,
            'max_tokens' => 1024,
        ]),
        CURLOPT_HTTPHEADER => [
            'Content-Type: application/json',
            'Authorization: Bearer ' . $token,
        ],
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_SSL_VERIFYPEER => false,
    ]);
    $result = json_decode(curl_exec($ch), true);
    curl_close($ch);
    return $result;
}
```

## .env шаблон

```
GIGACHAT_CLIENT_ID=
GIGACHAT_CLIENT_SECRET=
GIGACHAT_SCOPE=GIGACHAT_API_PERS
GIGACHAT_MODEL=GigaChat-2-Max
```

## Обработка ошибок

| Код | Причина | Действие |
|---|---|---|
| 401 | Токен истёк или невалидный | Обновить токен, повторить запрос |
| 429 | Превышен лимит запросов | Подождать, повторить с backoff |
| 400 + `finish_reason: "blacklist"` | Контент-фильтр | Вернуть пользователю вежливый отказ |
| 400 + `finish_reason: "length"` | Превышен max_tokens | Увеличить max_tokens или разбить на части |
| SSL Error | Самоподписанный сертификат | Добавить `rejectUnauthorized: false` |

## Миграция с OpenRouter/DeepSeek

При замене DeepSeek на GigaChat:

1. **System prompt:** убрать блок "ты НЕ ChatGPT" - GigaChat не представляется GPT
2. **HTML в ответах:** GigaChat может возвращать markdown - добавить санитизацию
3. **Rate limits:** учитывать 1 поток (физлица) или 10 потоков (юрлица)
4. **Streaming формат:** SSE-совместимый, аналогичен OpenAI - минимальные правки
5. **Токенизация:** русский текст ~3-4 символа = 1 токен (эффективнее чем у GPT для русского)

## Лимиты

| Параметр | Физлица | Юрлица |
|---|---|---|
| Параллельные потоки | 1 | 10+ |
| Контекст (вход+выход) | 128K | 128K |
| Vision: картинок на сообщение | 1 | 1 |
| Vision: картинок на диалог | 10 | 10 |
| Аудио | до 30 МБ, до 1 часа | до 30 МБ, до 1 часа |
| Жизнь OAuth-токена | 30 минут | 30 минут |

## Полезные ссылки

- Документация: https://developers.sber.ru/docs/ru/gigachat/api/overview
- Studio (ключи): https://developers.sber.ru/studio
- Python SDK: https://github.com/ai-forever/gigachat
- Node.js SDK: https://github.com/ai-forever/gigachat-js
- GigaChain (LangChain): https://github.com/ai-forever/gigachain
- Тарифы: https://developers.sber.ru/docs/ru/gigachat/api/tariffs
- Поддержка: https://t.me/gigachat_helpbot
- Email: gigachat@sberbank.ru
