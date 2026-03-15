---
name: pre-landing-review
description: |
  Pre-landing ревью ветки перед мержем. Анализ diff против main на структурные проблемы,
  которые тесты не ловят. SQL safety, race conditions, XSS, trust boundaries.
  Используй когда: готов мержить, перед PR, перед деплоем. Команда: /pre-landing-review
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Pre-Landing Review

Ты проводишь `/pre-landing-review`. Анализируй diff текущей ветки против main на структурные проблемы, которые тесты не ловят.

---

## Шаг 1: Проверка ветки

```bash
BRANCH=$(git branch --show-current 2>/dev/null)
echo "Branch: $BRANCH"
```

Если на `main` — выведи: **"Нечего ревьюить — ты на main."** и остановись.

```bash
git fetch origin main --quiet 2>/dev/null
git diff origin/main --stat 2>/dev/null
```

Если diff пуст — то же сообщение, остановись.

---

## Шаг 2: Получить diff

```bash
git fetch origin main --quiet
git diff origin/main
```

---

## Шаг 3: Двухпроходное ревью

### Pass 1 — КРИТИЧЕСКИЕ (блокируют мерж)

#### SQL/NoSQL инъекции
- String interpolation в запросах (даже с `.toString()`, `parseInt()`)
- Шаблонные литералы в SQL: `` `SELECT * FROM ${table}` ``
- Несанитизированный ввод в ORM raw queries, `$where`, MongoDB `$regex`
- Mongoose/Sequelize/Knex: `.whereRaw()`, `.raw()` без параметризации

#### Race Conditions & Concurrency
- Read-check-write без unique constraint или retry на дубликат
- `findOrCreate` / `upsert` на колонках без unique DB index
- Переходы статусов без атомарного `WHERE old_status = ? UPDATE SET new_status`
- Конкурентная запись в файлы без блокировки

#### XSS
- `innerHTML`, `dangerouslySetInnerHTML`, `v-html` с пользовательскими данными
- `document.write()`, `eval()` с внешним вводом
- Template literals вставленные в HTML без экранирования
- `html_safe` / `|safe` / `{!! !!}` на пользовательских данных

#### LLM Output Trust Boundary
- Данные от LLM (emails, URLs, имена) записываются в БД без валидации формата
- Structured output от AI (массивы, объекты) принимаются без проверки типов/формы перед записью в БД
- Ответ модели используется для формирования SQL, shell команд, или URL без санитизации

---

### Pass 2 — ИНФОРМАЦИОННЫЕ (не блокируют, но включаются в PR)

#### Conditional Side Effects
- Ветвление по условию, но одна ветка забывает side effect (отправить email, обновить счётчик, записать лог)
- Лог утверждает что действие произошло, но действие было условно пропущено

#### Magic Numbers & String Coupling
- Числовые литералы используемые в нескольких файлах — должны быть named constants
- Строки ошибок используемые как фильтры в другом месте

#### Dead Code & Consistency
- Переменные присвоенные но не прочитанные
- Комментарии/docstrings описывающие старое поведение после изменения кода
- `console.log` / `print()` дебаг-вывод оставленный в коде

#### Error Handling
- Пустые `catch` блоки: `catch(e) {}` или `except: pass`
- `catch(e) { console.log(e) }` без re-throw или graceful degradation
- Промисы без `.catch()` или `try/catch` для `await`
- Missing error handler в Express/FastAPI middleware

#### Test Gaps
- Negative-path тесты проверяют тип/статус, но не side effects
- Отсутствие тестов для новых error paths
- Security enforcement (auth, rate limiting) без integration тестов
- Assertions на наличие строки без проверки формата

#### Crypto & Entropy
- `Math.random()` / `random.random()` для security-sensitive значений — используй `crypto.randomBytes` / `secrets`
- Non-constant-time сравнение (`===`) для секретов/токенов — уязвимо к timing attacks
- Truncation данных вместо хеширования (последние N символов вместо SHA-256)

#### Type Coercion
- Значения пересекающие границы (JSON сериализация) где тип может измениться: числа ↔ строки
- Hash/digest inputs без `.toString()` перед сериализацией

#### Performance
- N+1 запросы: обращение к связанным данным в циклах без eager loading
- `Array.find()` / `.filter()` в цикле вместо индексации через Map/Set
- Запросы к БД без индексов на фильтруемых/сортируемых полях
- Inline стили/скрипты в шаблонах (re-parsed каждый рендер)

---

## Шаг 4: Вывод

**Формат:**
```
Pre-Landing Review: N проблем (X критических, Y информационных)

**КРИТИЧЕСКИЕ** (блокируют мерж):
- [файл:строка] Описание проблемы
  Фикс: предложенное решение

**Информационные** (рекомендации):
- [файл:строка] Описание проблемы
  Фикс: предложенное решение
```

Если проблем нет: `Pre-Landing Review: Проблем не найдено.`

**Если найдены КРИТИЧЕСКИЕ:**
Для КАЖДОЙ критической проблемы — отдельный блок:
```
КРИТИЧЕСКАЯ ПРОБЛЕМА #N: [описание]
Файл: [путь:строка]
Рекомендуемый фикс: [описание]

Варианты:
  А) Исправить сейчас (рекомендую)
  Б) Принять риск и мержить
  В) Это false positive — пропустить
```

**СТОП.** Жди ответ по каждой критической проблеме. Если пользователь выбрал А — примени фикс.

---

## Suppressions — НЕ флагай

- Избыточный код, который не вредит и помогает читаемости
- "Добавь комментарий объясняющий почему выбран этот порог" — пороги меняются, комментарии гниют
- Consistency-only изменения (обернуть значение в условие чтобы совпадало с другим стилем)
- Regex не обрабатывает edge case X, когда ввод ограничен и X никогда не встречается
- Безвредные no-op операции
- ВСЁ что уже исправлено в том же diff — прочитай ВЕСЬ diff перед комментированием

## Важные правила

- **Прочитай ВЕСЬ diff перед комментированием.** Не флагай то, что уже исправлено в diff.
- **Read-only по умолчанию.** Меняй файлы только если пользователь явно выбрал "Исправить сейчас".
- **Будь кратким.** Одна строка — проблема, одна строка — фикс. Без преамбул.
- **Только реальные проблемы.** Пропускай всё что в порядке.
