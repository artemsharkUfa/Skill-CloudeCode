---
name: seo-optimizer
description: "SEO-оптимизация сайтов по рекомендациям Яндекса и Google. Use when пользователь просит: seo, сео, оптимизация сайта, мета-теги, robots.txt, sitemap, structured data, schema.org, open graph, og image, поисковая оптимизация, индексация, ранжирование, seo аудит, seo оптимизация, проверь seo, добавь мета теги, hreflang, json-ld, яндекс вебмастер, google search console, canonical, сниппет, микроразметка, семантика html, seo check, optimize for search, seo для яндекса, indexnow."
---

# SEO Optimizer

Скилл для SEO-оптимизации сайтов по рекомендациям Яндекса и Google. Основан на официальной документации Яндекс.Вебмастера, данных утечки факторов ранжирования, и актуальных best practices.

## КРИТИЧНЫЕ ПРАВИЛА

1. **Всегда читать index.html перед оптимизацией.** Понять текущее состояние сайта.
2. **Русскоязычный приоритет.** Для Яндекса: description, title, keywords на русском.
3. **Не ломать существующий код.** SEO-изменения минимально инвазивны.
4. **OG-картинка обязательна.** Генерировать через Python/PIL если нет готовой.
5. **Проверять результат.** curl robots.txt, curl sitemap.xml, View Source для мета-тегов.

## Workflow

### Шаг 0: SEO-аудит

Прочитать текущий HTML и определить:

| Что проверить | Как |
|---|---|
| `<html lang="">` | Должен быть основной язык сайта |
| `<title>` | Уникальный, 50-60 символов, ключевые слова |
| `<meta name="description">` | 155-160 символов, привлекательный |
| `<meta name="robots">` | index, follow |
| `<link rel="canonical">` | Полный URL |
| Open Graph теги | og:title, og:description, og:image, og:url |
| JSON-LD | Organization, WebSite |
| Семантический HTML | header, nav, main, section, footer |
| hreflang | Для многоязычных сайтов |
| robots.txt | Есть на сервере? |
| sitemap.xml | Есть на сервере? |

Показать пользователю отчёт: что есть, чего не хватает.

### Шаг 1: Мета-теги в `<head>`

#### Title
```html
<title>Бренд - Ключевая фраза | Дополнение</title>
```
- 50-60 символов (Яндекс обрезает длинные)
- Ключевые слова ближе к началу
- Уникальный для каждой страницы
- Если сайт двуязычный - основной язык + через | второй

#### Description
```html
<meta name="description" content="Описание на 155-160 символов с ключевыми словами и призывом к действию.">
```
- Влияет на CTR в поисковой выдаче (сниппет)
- Должен точно отражать содержимое страницы
- Содержать CTA (бесплатно, узнать больше, и т.д.)

#### Keywords (для Яндекса)
```html
<meta name="keywords" content="ключевое слово 1, фраза 2, фраза 3">
```
- Яндекс учитывает с низким весом
- 5-10 ключевых фраз через запятую
- Не спамить

#### Robots
```html
<meta name="robots" content="index, follow">
```

#### Canonical
```html
<link rel="canonical" href="https://example.ru/">
```
- Абсолютный URL с протоколом
- Указывает поисковику каноническую версию страницы
- Предотвращает дубли (http/https, www/без www, с/без слеша)

#### Author
```html
<meta name="author" content="Имя, Компания">
```

### Шаг 2: Open Graph

```html
<!-- Open Graph -->
<meta property="og:type" content="website">
<meta property="og:url" content="https://example.ru/">
<meta property="og:title" content="Заголовок до 50 символов">
<meta property="og:description" content="Описание в 1-2 предложения">
<meta property="og:image" content="https://example.ru/og-image.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:locale" content="ru_RU">
<meta property="og:locale:alternate" content="en_US">
<meta property="og:site_name" content="Название сайта">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Заголовок">
<meta name="twitter:description" content="Описание">
<meta name="twitter:image" content="https://example.ru/og-image.png">
```

#### Генерация OG-картинки (1200x630)

```python
from PIL import Image, ImageDraw, ImageFont

W, H = 1200, 630
img = Image.new('RGB', (W, H), (фон))
draw = ImageDraw.Draw(img)

# Градиентная полоса сверху/снизу (цвета бренда)
# Название компании (крупный шрифт)
# Слоган / описание (средний шрифт)
# URL сайта (мелкий шрифт, акцентный цвет)

img.save('og-image.png', 'PNG', optimize=True)
```

Шрифт: `/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf`

### Шаг 3: JSON-LD (Schema.org)

#### Organization (обязательно для бизнес-сайтов)
```html
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "Organization",
    "name": "Название компании",
    "alternateName": "English Name",
    "url": "https://example.ru",
    "logo": "https://example.ru/logo.svg",
    "description": "Описание деятельности",
    "founder": {
        "@type": "Person",
        "name": "Имя основателя"
    },
    "sameAs": [
        "https://t.me/channel",
        "https://twitter.com/handle"
    ],
    "contactPoint": {
        "@type": "ContactPoint",
        "contactType": "customer service",
        "url": "https://t.me/bot",
        "availableLanguage": ["Russian", "English"]
    }
}
</script>
```

#### WebSite
```html
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "WebSite",
    "name": "Название",
    "url": "https://example.ru",
    "inLanguage": ["ru", "en"]
}
</script>
```

#### Другие типы (по ситуации)
| Тип | Когда |
|---|---|
| `LocalBusiness` | Локальный бизнес с адресом |
| `Product` + `Offer` | Интернет-магазин |
| `BreadcrumbList` | Многостраничный сайт |
| `Article` / `BlogPosting` | Блог, статьи |
| `FAQPage` + `Question` | Страница FAQ |
| `SoftwareApplication` | Описание ПО |

### Шаг 4: Hreflang (многоязычные сайты)

```html
<link rel="alternate" hreflang="ru" href="https://example.ru/">
<link rel="alternate" hreflang="en" href="https://example.ru/">
<link rel="alternate" hreflang="x-default" href="https://example.ru/">
```

- Каждая языковая версия ссылается на все остальные
- `x-default` - страница по умолчанию
- Если JS-переключатель языка (один URL) - всё равно указать обе версии

В JS-переключателе добавить:
```javascript
document.documentElement.lang = lang;
```

### Шаг 5: Семантический HTML

Проверить и при необходимости добавить:

```html
<header> <!-- Шапка сайта -->
  <nav> <!-- Навигация -->
</header>
<main> <!-- Основной контент (один на страницу) -->
  <section id="services"> <!-- Логические секции -->
  <article> <!-- Самостоятельные блоки контента -->
</main>
<footer> <!-- Подвал -->
</footer>
```

- `<main>` оборачивает всё между header и footer
- `lang="ru"` на `<html>` для русскоязычных сайтов (не `en`)
- Заголовки H1-H6 в правильной иерархии (один H1 на страницу)
- Alt-текст для всех `<img>`

### Шаг 6: robots.txt

```
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /backups/
Disallow: /tmp/

Sitemap: https://example.ru/sitemap.xml
```

#### Яндекс-специфические директивы:

```
User-agent: Yandex
Crawl-delay: 1
Clean-param: utm_source&utm_medium&utm_campaign
```

- `Clean-param` - удаляет UTM-параметры из URL (предотвращает дубли)
- `Crawl-delay` - задержка между запросами бота (не перегружать сервер)
- Директива `Host` устарела (с 2018), НЕ использовать. Вместо неё - 301 редиректы.

### Шаг 7: sitemap.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xhtml="http://www.w3.org/1999/xhtml">
    <url>
        <loc>https://example.ru/</loc>
        <lastmod>2026-03-01</lastmod>
        <changefreq>weekly</changefreq>
        <priority>1.0</priority>
        <!-- Hreflang в sitemap (для многоязычных) -->
        <xhtml:link rel="alternate" hreflang="ru" href="https://example.ru/"/>
        <xhtml:link rel="alternate" hreflang="en" href="https://example.ru/"/>
    </url>
</urlset>
```

- `lastmod` - реальная дата последнего изменения (не фейковая)
- `changefreq` - ориентировочная частота обновления
- `priority` - от 0.0 до 1.0 (главная = 1.0, второстепенные = 0.5-0.8)

### Шаг 8: Проверка и деплой

```bash
# Деплой файлов
scp index.html robots.txt sitemap.xml og-image.png user@server:/var/www/site/

# Проверка
curl -s https://example.ru/robots.txt
curl -s https://example.ru/sitemap.xml
curl -s https://example.ru/ | head -50  # Проверить мета-теги
```

## Факторы ранжирования Яндекса (справочник)

### Топ-факторы (по данным утечки 2023)

| Категория | Вес | Что делать |
|---|---|---|
| **Поведенческие** | 19% | Снижать bounce rate, увеличивать time on site, depth |
| **Ссылки** | 6% | Качественные входящие ссылки с релевантных сайтов |
| **Релевантность** | 6% | Контент отвечает на запрос пользователя |
| **URL-факторы** | - | Короткие, читаемые URL, без лишних параметров |
| **Возраст** | - | Старые страницы имеют бонус; новые набирают со временем |
| **Ошибки** | - | 4xx/5xx негативно влияют |

### Поведенческие факторы (ключевая особенность Яндекса)

| Метрика | Что значит | Как улучшить |
|---|---|---|
| Bounce rate | Визит < 15 сек | Вовлекающий контент, быстрая загрузка |
| Time on site | Время сессии | Полезный контент, видео, интерактив |
| Scroll depth | Глубина прокрутки | Интересная структура, визуальные блоки |
| CTR в выдаче | Клики / показы | Привлекательный title + description |
| Pogo-sticking | Быстрый возврат в поиск | Контент отвечает на запрос |

### Чеклист технического SEO

- [ ] HTTPS (SSL сертификат)
- [ ] Mobile-friendly (responsive дизайн)
- [ ] Быстрая загрузка (< 3 сек)
- [ ] Gzip/Brotli сжатие
- [ ] Кеширование статики
- [ ] Оптимизация изображений (WebP, lazy loading)
- [ ] Нет битых ссылок (404)
- [ ] Нет дублей контента
- [ ] Корректный robots.txt
- [ ] XML sitemap
- [ ] Canonical URLs
- [ ] Структурированные данные (JSON-LD)
- [ ] Open Graph теги
- [ ] Адаптивный viewport

### Яндекс.Метрика (рекомендация)

Установить счётчик Яндекс.Метрики:
```html
<!-- Яндекс.Метрика -->
<script type="text/javascript">
(function(m,e,t,r,i,k,a){m[i]=m[i]||function(){(m[i].a=m[i].a||[]).push(arguments)};
m[i].l=1*new Date();
for(var j=0;j<document.scripts.length;j++){if(document.scripts[j].src===r){return;}}
k=e.createElement(t),a=e.getElementsByTagName(t)[0],k.async=1,k.src=r,a.parentNode.insertBefore(k,a)})
(window, document, "script", "https://mc.yandex.ru/metrika/tag.js", "ym");
ym(XXXXXXXX, "init", {
    clickmap:true,
    trackLinks:true,
    accurateTrackBounce:true,
    webvisor:true
});
</script>
```

- Не является прямым фактором ранжирования (по заявлению Яндекса)
- Но даёт Яндексу полную видимость поведения пользователей
- Вебвизор, карта кликов, аналитика форм - полезны для оптимизации

### IndexNow (мгновенная индексация)

Для уведомления Яндекса об изменениях:

```bash
curl "https://yandex.com/indexnow?url=https://example.ru/&key=YOUR_KEY"
```

- Генерация ключа в Яндекс.Вебмастере
- Файл подтверждения: `https://example.ru/YOUR_KEY.txt`
- Поддерживает: Yandex, Bing, Seznam, Naver
- Google НЕ поддерживает IndexNow

## Алгоритмы Яндекса (знать для избежания санкций)

| Алгоритм | Что штрафует |
|---|---|
| **Минусинск** | Покупные/спамные ссылки |
| **АГС** | Сайты-сателлиты, дорвеи |
| **Баден-Баден** | Переоптимизированные тексты, SEO-спам |
| **Поведенческий фильтр** | Накрутка поведенческих факторов |

Главный принцип Яндекса: **"Хороший сайт - это сайт для людей"**. Делать контент для пользователей, не для роботов.
