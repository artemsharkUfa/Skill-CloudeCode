---
name: frontend-agent
description: "AI-агент для фронтенд-разработки: конвертирует дизайн в код. Use when пользователь просит: сделай сайт, создай страницу, сверстай, редизайн, landing page, лендинг, дизайн в код, скриншот в код, URL в код, сверстай макет, frontend, фронтенд, HTML страница, сделай как на скриншоте, перерисуй сайт, скопируй дизайн, воспроизведи дизайн, design to code, website, webpage, сделай похожий сайт."
---

# Frontend Agent

AI-агент для фронтенд-разработки. Конвертирует дизайн (скриншот, URL живого сайта, текстовое описание) в готовый адаптивный код с живым превью и итеративными правками.

## Мульти-модельная архитектура

Агент использует три модели с разделением ответственности:

| Модель | ID (OpenRouter) | Роль |
|--------|-----------------|------|
| **Opus 4.6** | (локальный, Claude Code) | Оркестратор: ставит задачи, ревьюит результат, общается с пользователем |
| **Gemini 3.1 Pro** | `google/gemini-3.1-pro-preview` | Рабочая лошадка: анализ дизайна, генерация кода (1M контекст) |
| **Nano Banana 2** | `google/gemini-3.1-flash-image-preview` | Генерация изображений: иконки, иллюстрации, фоны, мокапы |

**Скрипт API-клиента:** `scripts/openrouter_client.py`
**API-ключ:** `~/.env (OPENROUTER_API_KEY)` -> `OPENROUTER_API_KEY`

### Принцип работы

1. **Opus** получает задачу от пользователя и формирует промт
2. **Opus** вызывает `openrouter_client.py` для Gemini (код) или Nano Banana (изображения)
3. **Opus** ревьюит результат: проверяет качество, семантику, адаптивность
4. Если результат не устраивает — **Opus** уточняет промт и запрашивает повторно
5. Только после ревью результат показывается пользователю

### Когда использовать какую модель

- **Gemini 3.1 Pro** — анализ скриншотов, генерация HTML/CSS/JS, React-компонентов, рефакторинг
- **Nano Banana 2** — генерация иконок, hero-изображений, иллюстраций, фонов, favicon
- **Opus** — оркестрация, ревью кода, общение с пользователем, финальные правки

## КРИТИЧНЫЕ ПРАВИЛА

1. **Всегда спрашивать стек перед генерацией.** Показать варианты из `references/stacks-guide.md`.
2. **Всегда показывать план перед генерацией.** Не генерировать код без одобрения структуры.
3. **Превью обязательно.** После генерации всегда показать результат через Playwright.
4. **Mobile-first.** Все стили начинаются с мобильной версии, desktop через media queries.
5. **Семантический HTML.** Использовать header, nav, main, section, article, footer.
6. **CSS custom properties.** Все цвета, шрифты, отступы через переменные в :root.
7. **Opus ревьюит всё.** Код от Gemini ВСЕГДА проверяется Opus перед показом пользователю.
8. **Защита от копирования.** Каждый сгенерированный сайт включает user-select: none + JS-блокировку copy/contextmenu.
9. **Выходная папка:** `~/outputs/<project-name>/`
10. **Accessibility обязательна.** WCAG 2.1 AA: alt-тексты, aria-атрибуты, focus-стили, контраст 4.5:1.

## Правила качества кода

Применять ко ВСЕМУ генерируемому коду (и от Gemini, и от Opus):

**JavaScript:**
- **Early returns** — граничные проверки в начале функции, не вложенные if/else
- **const arrow functions** — `const handleClick = () => {}` вместо function declarations
- **handle* prefix** — обработчики: handleClick, handleSubmit, handleChange
- **JSDoc** — на всех функциях в main.js и animations.js
- **data-testid** — на ключевых интерактивных элементах

**Accessibility:**
- **ARIA** — aria-label на иконках-кнопках, aria-expanded на меню, role где нужно
- **Alt-тексты** — описательные, не "image" или "logo"
- **Focus** — видимый :focus-visible на кнопках, ссылках, инпутах
- **Skip-link** — "Перейти к содержимому" для клавиатурной навигации

**Оптимизация изображений:**
- **loading="lazy"** — на img ниже первого экрана
- **width/height** — на всех img для предотвращения CLS
- **WebP** — <picture> с webp + fallback jpg/png

## Workflow

### Шаг 0: Выбор стека и уточнение задачи

**ОБЯЗАТЕЛЬНО перед началом работы** прочитать `references/stacks-guide.md` и задать пользователю уточняющие вопросы:

**Вопрос 1 - Тип проекта:**
- Лендинг / промо-страница
- Продуктовый сайт / SaaS
- Дашборд / админка
- Портфолио
- Блог / контентный сайт
- Интернет-магазин

**Вопрос 2 - Стек (предложить на основе типа проекта):**
- HTML/CSS/JS + GSAP (простой лендинг, нулевые зависимости)
- React + Tailwind CSS (интерактивный SPA)
- React + shadcn/ui (продуктовый интерфейс, дашборд, SaaS)
- React + Aceternity UI (лендинг с wow-анимациями)
- React + Magic UI (маркетинговый лендинг с анимациями)
- Astro (контентный сайт, блог, максимальная скорость)
- Svelte + SvelteKit (высокая производительность)
- Другой (пользователь указывает сам)

**Вопрос 3 - Уровень анимации** (см. Шаг 2.5)

**Вопрос 4 - Design-система или референс:**
- Есть скриншот/URL для копирования?
- Есть конкретная design-система (Material, Carbon, Ant Design...)?
- Свободный дизайн на усмотрение агента?

На основе ответов выбрать стек и подход к генерации. Показать пользователю что агент умеет с выбранным стеком.

### Шаг 1: Определить тип входа

Автоматически определить по входным данным:

| Вход | Признак | Действие |
|------|---------|----------|
| Изображение | Путь к файлу (.png, .jpg, .webp, .svg) | Прочитать через Read tool (vision) |
| URL | Начинается с http:// или https:// | Открыть через Playwright, сделать скриншот, извлечь данные |
| Текст | Всё остальное | Уточнить требования если нужно |

**Для режима URL — выполнить:**
1. `browser_navigate` на URL
2. `browser_take_screenshot` (type: png, fullPage: true) — сохранить как reference
3. `browser_evaluate` — извлечь:
   - document.title
   - Все CSS custom properties из :root
   - Computed styles основных элементов (цвета, шрифты)
   - Мета-теги (description, viewport)
4. `browser_snapshot` — получить accessibility tree для структуры
5. `browser_resize` к 375x812 + повторный скриншот мобильной версии

**Для режима Image:**
1. Прочитать файл через Read tool — Claude Vision проанализирует изображение
2. Если пользователь дал несколько изображений — проанализировать все

### Шаг 2: Анализ дизайна

**Вариант A: Через Gemini 3.1 Pro (рекомендуется для скриншотов):**

```bash
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py analyze \
  --image "/path/to/screenshot.png" \
  --output "/tmp/analysis.md"
```

Затем Opus читает результат, проверяет качество анализа и дополняет если нужно.

**Вариант B: Через Claude Vision (если нет API-ключа или для быстрого анализа):**

Прочитать изображение через Read tool и заполнить шаблон анализа (см. `references/analysis-template.md`).

Результат анализа показать пользователю в формате:

```
## Анализ дизайна

**Layout:** [описание сетки и структуры]
**Секции:** [список секций сверху вниз]
**Цветовая палитра:**
- Основной: #XXXXXX
- Акцент: #XXXXXX
- Фон: #XXXXXX
- Текст: #XXXXXX

**Типографика:**
- Заголовки: [шрифт]
- Основной текст: [шрифт]

**Компоненты:** [кнопки, карточки, формы...]
```

### Шаг 2.5: Анимации (GSAP)

**По умолчанию всегда используй СРЕДНИЙ уровень** (gsap + ScrollTrigger + ScrollSmoother + SplitText + ScrollToPlugin). Минимальный - только если пользователь явно попросил "без анимаций" или "просто".

| Уровень | Что включает | GSAP-плагины |
|---------|-------------|-------------|
| Без анимации | Только если пользователь явно попросил | - |
| Минимальный | Fade/slide при скролле, hover | gsap + ScrollTrigger |
| **Средний (по умолчанию)** | **Text reveal, parallax, smooth scroll, counters, navbar** | **gsap + ScrollTrigger + ScrollSmoother + SplitText + ScrollToPlugin** |
| Максимальный | + Horizontal scroll, SVG draw, Flip | + все плагины |

**КРИТИЧНО:** Не импровизируй анимации. Копируй готовые пресеты из `references/gsap-animations.md` (раздел "Готовые пресеты для типовых секций"). Доступные пресеты: Smooth Scroll, Hero, Navbar, Features/Services, Stats, Testimonials, CTA, Footer, About, Pricing, FAQ. Каждый пресет — готовая функция с полным кодом. Копируй целиком, адаптируй только CSS-селекторы. В конце вставь шаблон инициализации из раздела "Полная инициализация".

### Шаг 3: План разработки

Заполнить шаблон плана (см. `references/plan-template.md`).

Показать пользователю для одобрения:
```
## План: <имя проекта>

**Стек:** [HTML/CSS/JS | React+Tailwind | ...]
**Файлы:**
- index.html
- css/style.css
- js/main.js
- [images/...]

**Секции страницы:**
1. Header / Navigation
2. Hero
3. ...
N. Footer

**Шрифты:** Google Fonts — [список]
**Палитра:** [hex-коды из анализа]
```

Ждать одобрения. Если пользователь вносит правки — обновить план.

### Шаг 3.5: Content Manifest (анти-галлюцинация)

Opus создаёт JSON-манифест контента на основе анализа (см. `references/content-protocol.md`):

1. Извлечь **все** тексты из анализа дизайна / скриншота / URL
2. Для каждого текста указать точное значение без перефразирования
3. Для билингвальных сайтов указать обе версии (en/ru)
4. Включить: заголовки, описания, числа, статистику, имена, должности, ссылки, alt-тексты
5. Сохранить манифест: `~/outputs/<project>/content.json`
6. Показать манифест пользователю для проверки

Манифест передаётся в Gemini через `--content` и гарантирует сохранность данных при генерации.

### Шаг 4: Генерация кода

**Определить стек:**
- Если пользователь указал стек — использовать его
- Если нет — по умолчанию чистый HTML/CSS/JS для лендингов и простых сайтов
- React+Tailwind предлагать для SPA и сложных интерфейсов

**Генерация через Gemini 3.1 Pro (рекомендуется):**

Opus формирует детальный промт на основе анализа и плана, затем:

```bash
# Генерация основного HTML (с манифестом контента)
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py code \
  --prompt "Промт с деталями из анализа и плана (ТОЛЬКО layout/styling инструкции)" \
  --content "~/outputs/<project>/content.json" \
  --image "/path/to/reference-screenshot.png" \
  --output "~/outputs/<project>/index.html"

# Генерация CSS
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py code \
  --prompt "Generate CSS for the following HTML..." \
  --output "~/outputs/<project>/css/style.css"

# Патч существующего файла (для итеративных правок)
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py patch \
  --file "~/outputs/<project>/index.html" \
  --prompt "Описание изменений в layout/styling" \
  --content "~/outputs/<project>/content.json" \
  --output "~/outputs/<project>/index.html"
```

**После получения кода от Gemini — Opus ОБЯЗАН:**

A. **Content Fidelity Review (если использовался манифест):**
1. Все заголовки секций совпадают с манифестом
2. Все описания/тексты воспроизведены без перефразирования
3. Все числа и статистика точные
4. Имена, должности, бренды точные
5. Ссылки (href) из манифеста
6. Alt-тексты изображений из манифеста
7. Нет добавленных текстов, которых нет в манифесте
8. Нет удалённых секций/карточек, которые есть в манифесте

B. **Code Quality Review:**
1. Семантика HTML (header, nav, main, section, footer)
2. CSS custom properties в :root
3. Mobile-first media queries
4. BEM-именование классов
5. Защита от копирования: user-select: none + JS блокировка (contextmenu, copy, cut, Ctrl+C/A/U)
6. Accessibility: alt-тексты, aria-атрибуты, :focus-visible, skip-link
7. JS quality: early returns, const arrow functions, handle* prefix, JSDoc
8. Image optimization: loading="lazy", width/height на img, <picture> для webp
9. Исправить проблемы если есть

Только после обоих ревью переходить к превью.

**Генерация изображений через Nano Banana 2:**

Перед КАЖДОЙ генерацией изображений спросить у пользователя качество:

| Качество | Разрешение | Стоимость | Когда использовать |
|----------|-----------|-----------|-------------------|
| 1K | 1024x1024 | Дешевле | Иконки, favicon, мелкие элементы |
| 2K | 2048x2048 | Средне | Большинство задач (default) |
| 4K | 4096x4096 | Дороже | Hero-изображения, фоны, баннеры |

```bash
# Иконки
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py image \
  --prompt "Flat minimalist icon of [описание], single color #XXXXXX, transparent background, 512x512" \
  --output "~/outputs/<project>/images/icon-name.png"

# Hero-изображение
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py image \
  --prompt "Hero background: [описание сцены], modern, professional" \
  --aspect-ratio "16:9" \
  --output "~/outputs/<project>/images/hero-bg.png"

# Favicon
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py image \
  --prompt "Favicon for [бренд], simple geometric, works at 32x32" \
  --output "~/outputs/<project>/images/favicon.png"
```

**Без OpenRouter API (fallback):**

Если API-ключ не настроен, Opus генерирует код самостоятельно, а для изображений использует placeholder-сервисы (placehold.co) или SVG-иконки.

**Для HTML/CSS/JS стека:**

Создать файлы в `~/outputs/<project-name>/`:
```
<project-name>/
├── index.html
├── css/
│   └── style.css
├── js/
│   └── main.js
└── images/
    └── (если нужны placeholder-изображения)
```

**Правила генерации HTML:**
- DOCTYPE html, lang="ru" (или en по контексту)
- Meta viewport, charset utf-8
- Подключение Google Fonts через <link>
- Семантические теги
- BEM-подобное именование классов (block__element--modifier)
- Все стили в отдельном CSS файле, не inline

**Правила генерации CSS:**
- CSS custom properties в :root для всех переменных
- Mobile-first media queries
- Flexbox/Grid для layout
- Smooth transitions для интерактивных элементов
- Минимум 3 breakpoints: mobile (default), tablet (768px), desktop (1024px)
- Защита от копирования текста: `user-select: none` на body, исключение для input/textarea

**Защита от копирования (обязательно для всех проектов):**

CSS (добавлять в конец стилей):
```css
body { -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none; }
input, textarea { -webkit-user-select: text; -moz-user-select: text; -ms-user-select: text; user-select: text; }
```

JS (добавлять перед </body>):
```js
document.addEventListener('contextmenu', e => e.preventDefault());
document.addEventListener('keydown', e => {
    if (e.ctrlKey && (e.key === 'c' || e.key === 'C' || e.key === 'u' || e.key === 'U' || e.key === 'a' || e.key === 'A')) {
        e.preventDefault();
    }
});
document.addEventListener('copy', e => e.preventDefault());
document.addEventListener('cut', e => e.preventDefault());
```

**Правила генерации JS:**
- Vanilla JS, без фреймворков (для HTML/CSS/JS стека)
- GSAP для анимаций (подключать через CDN, см. `references/gsap-animations.md`)
- DOMContentLoaded для инициализации
- Event delegation где возможно
- GSAP-анимации всегда в отдельном файле `js/animations.js`

**Для React+Tailwind стека:**

Создать файлы:
```
<project-name>/
├── package.json
├── index.html (Vite entry)
├── src/
│   ├── main.jsx
│   ├── App.jsx
│   ├── components/
│   │   ├── Header.jsx
│   │   ├── Hero.jsx
│   │   └── ...
│   └── index.css (Tailwind directives)
├── tailwind.config.js
└── vite.config.js
```

### Шаг 5: Превью через Playwright

Следовать инструкциям из `references/preview-guide.md`.

Краткий порядок:
1. Запустить http-сервер: `python3 -m http.server 8080` в папке проекта (фоном)
2. `browser_navigate` к `http://localhost:8080`
3. `browser_resize` к 1440x900 + `browser_take_screenshot` — desktop
4. `browser_resize` к 375x812 + `browser_take_screenshot` — mobile
5. Показать оба скриншота пользователю
6. Остановить http-сервер

Если React/Vite — сначала `npm install && npm run dev`, затем Playwright.

### Шаг 6: Цикл итераций

После показа превью:
1. Спросить пользователя: "Что нужно изменить?"
2. Если пользователь даёт фидбэк — внести правки в файлы
3. Повторить превью (Шаг 5)
4. Повторять до "готово" / "отлично" / "принято"

При каждой итерации:
- Показывать что именно было изменено (список файлов и правок)
- Делать новые скриншоты для сравнения

### Шаг 6.5: Security Check (перед финализацией)

Opus проверяет код на типичные фронтенд-уязвимости:
1. **XSS** — нет innerHTML с пользовательскими данными, используй textContent
2. **Формы** — валидация на клиенте (pattern, required, maxlength, type="email")
3. **CDN integrity** — внешние скрипты с integrity + crossorigin="anonymous"
4. **localStorage** — не хранить токены/пароли/персональные данные
5. **HTTPS** — все внешние ресурсы через https://, нет mixed content
6. **Зависимости** — проверить актуальность версий CDN-библиотек

### Завершение

Когда пользователь одобряет результат:
1. Показать путь к файлам: `~/outputs/<project-name>/`
2. Показать список всех созданных файлов
3. Предложить: "Хочешь задеплоить или нужны ещё правки?"

## Обработка особых случаев

**Пользователь даёт только URL без инструкций:**
Спросить: "Что сделать с этим сайтом? Скопировать дизайн, улучшить, или сделать редизайн?"

**Многостраничный сайт:**
Спросить какие страницы нужны. Генерировать по одной, начиная с главной.

**Пользователь хочет конкретный стек:**
Принять указание пользователя. Если стек незнаком — предупредить и предложить альтернативу.

**Пользователь загружает несколько скриншотов:**
Проанализировать все, определить это страницы одного сайта или варианты дизайна. Спросить если непонятно.

## Справочник команд API-клиента

Скрипт: `~/.claude/skills/frontend-agent/scripts/openrouter_client.py`

### Анализ дизайна (Gemini 3.1 Pro)
```bash
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py analyze \
  --image "screenshot.png" \
  --output "analysis.md"
```

### Генерация кода (Gemini 3.1 Pro)
```bash
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py code \
  --prompt "Create responsive landing page with hero, features, pricing sections" \
  --content "content.json" \
  --image "reference.png" \
  --output "index.html"
```

### Патч существующего кода (Gemini 3.1 Pro)
```bash
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py patch \
  --file "index.html" \
  --prompt "Change hero layout to centered, add gradient background" \
  --content "content.json" \
  --output "index.html"
```

### Генерация изображений (Nano Banana 2)
```bash
python3 ~/.claude/skills/frontend-agent/scripts/openrouter_client.py image \
  --prompt "Flat icon of a gear, blue #3B82F6, transparent bg" \
  --aspect-ratio "1:1" \
  --quality "2k" \
  --output "icon.png"
```

### Параметры изображений
- **aspect-ratio:** `1:1` | `16:9` | `9:16` | `4:3` | `3:4`
- **quality:** `1k` (1024px) | `2k` (2048px, default) | `4k` (4096px)

### Переменные окружения
- `OPENROUTER_API_KEY` — ключ API (из `~/.env (OPENROUTER_API_KEY)`)
