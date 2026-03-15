---
name: skill-builder
description: Creates skills for Claude following documentation and best practices. Use when user wants to create a new skill, build skill, write SKILL.md, develop Claude capability module, or mentions "create skill", "build skill", "make skill for Claude", "создай скилл", "напиши skill".
---

# Skill Builder

Скилл для создания и оптимизации скиллов Claude Code.

## Философия: Explain the Why

Скиллы должны объяснять модели ПОЧЕМУ важно то или иное действие, а не просто диктовать ALWAYS/NEVER.

Плохо: `ALWAYS use kebab-case for names.`
Хорошо: `Use kebab-case for names because Claude's skill discovery system uses hyphens as word boundaries for matching.`

Когда модель понимает причину, она лучше адаптируется к edge cases.

## Обязательный рабочий процесс

### Шаг 1: Сбор требований

Узнать у пользователя (не больше 2-3 вопросов за раз):
1. Какую задачу решает скилл? Какую проблему закрывает?
2. Какие триггерные фразы (RU + EN) должны его активировать?
3. Нужны ли скрипты, референсы или ассеты?

### Шаг 2: Проверка документации

Прочитать `references/complete-guide.md` для:
- Структуры SKILL.md и правил frontmatter
- Progressive disclosure (3 уровня: metadata -> body -> resources)
- Рекомендации <500 строк body, >300 строк reference -> добавить TOC

### Шаг 3: Поиск актуальных практик

Выполнить веб-поиск:
```
Claude skills SKILL.md best practices [текущий год]
```

### Шаг 4: Написание скилла

**Структура папки:**
```
skill-name/
├── SKILL.md           # Обязательно
├── scripts/           # Опционально - исполняемый код
├── references/        # Опционально - документация
└── assets/            # Опционально - шаблоны, иконки
```

**Frontmatter (допустимые свойства):**
```yaml
---
name: skill-name          # kebab-case, max 64 символа
description: ...          # max 1024 символов, без < >
license: MIT              # опционально
allowed-tools: [...]      # опционально
metadata: {}              # опционально
compatibility: "..."      # опционально, max 500 символов
---
```

Никаких других свойств в frontmatter быть не должно. Claude игнорирует неизвестные свойства, что приводит к тихим ошибкам.

**Правила description (триггеринг):**
- Фокус на INTENT пользователя, не на реализации скилла
- Императивный стиль: "Use when..." не "This skill does..."
- Включить синонимы и триггерные фразы (RU + EN если двуязычный)
- 100-200 слов оптимально, конкурирует с другими скиллами за внимание

**Правила body:**
- Только информация, которую Claude НЕ знает
- Примеры вместо длинных объяснений
- Детали вынести в references/
- Объяснять ПОЧЕМУ, не только ЧТО

### Шаг 5: Валидация

Запустить `scripts/validate_skill.py <path>` для проверки:
- Whitelist свойств frontmatter
- kebab-case имя (max 64, без --, без forbidden words)
- description max 1024, без angle brackets
- Отсутствие запрещённых файлов (README.md, CHANGELOG.md)

### Шаг 6: Оптимизация триггеринга (опционально)

Для максимальной точности триггеринга использовать eval pipeline:

```bash
cd scripts/

# 1. Сгенерировать eval set (10 positive + 8 negative queries)
python generate_eval_set.py --skill-path ../my-skill --output evals.json

# 2. Проверить и отредактировать evals.json вручную

# 3. Прогнать оптимизацию (3-5 итераций)
python optimize_description.py \
    --eval-set evals.json \
    --skill-path ../my-skill \
    --max-iterations 5 \
    --verbose

# 4. Применить лучший description автоматически
python optimize_description.py \
    --eval-set evals.json \
    --skill-path ../my-skill \
    --apply
```

Как это работает:
1. `generate_eval_set.py` генерирует тестовые запросы через LLM
2. `run_eval.py` запускает `claude -p` для каждого запроса, проверяет триггерится ли скилл
3. `optimize_description.py` в цикле: eval -> анализ ошибок -> улучшение description -> eval
4. Train/test split 60/40 предотвращает overfitting
5. Лучший результат выбирается по TEST score

Требования: `pip install httpx pyyaml`, `claude` CLI в PATH, `OPENROUTER_API_KEY` в env.

### Шаг 7: Eval Loop (тестирование скилла субагентами)

После написания скилла - проверить его работу на реальных задачах. Это превращает процесс из "написал и надеюсь" в "написал, проверил, улучшил".

**Когда нужен eval loop:** скиллы с объективно проверяемыми результатами (генерация файлов, код, data extraction, workflow). Для субъективных скиллов (стиль текста, дизайн) - достаточно ручной проверки.

#### 7.1 Создать тест-кейсы

Придумать 2-3 реалистичных prompt-а - то, что реальный пользователь напишет. Показать пользователю для подтверждения.

Сохранить в `<skill-name>-workspace/evals/evals.json`:

```json
{
  "skill_name": "xlsx-maker",
  "evals": [
    {
      "id": 1,
      "prompt": "Создай таблицу расходов за квартал с формулами итогов",
      "expected_output": "Excel файл с 3 листами, формулами SUM, форматированием",
      "assertions": [
        "Файл .xlsx создан и не пустой",
        "Содержит формулу SUM",
        "Есть форматирование заголовков (жирный шрифт или цвет)"
      ]
    }
  ]
}
```

Assertions - объективно проверяемые утверждения. Не проверяй субъективное ("красивый дизайн").

См. `references/schemas.md` для полной схемы.

#### 7.2 Запустить прогоны (with_skill + baseline)

Для каждого тест-кейса запустить два субагента **параллельно в одном ходу** через Agent tool:

**With-skill прогон:**
```
Выполни задачу:
- Прочитай скилл: <path-to-skill>/SKILL.md и следуй его инструкциям
- Задача: <eval prompt>
- Входные файлы: <если есть>
- Сохрани результаты в: <workspace>/iteration-1/eval-<name>/with_skill/outputs/
```

**Baseline прогон** (тот же prompt, без скилла):
```
Выполни задачу:
- Задача: <eval prompt>
- Входные файлы: <если есть>
- Сохрани результаты в: <workspace>/iteration-1/eval-<name>/without_skill/outputs/
- НЕ используй никаких скиллов
```

Создать `eval_metadata.json` для каждого eval с описательным именем.

**Структура workspace:**
```
<skill-name>-workspace/
├── evals/
│   └── evals.json
├── iteration-1/
│   ├── eval-simple-table/
│   │   ├── eval_metadata.json
│   │   ├── with_skill/
│   │   │   ├── outputs/
│   │   │   ├── grading.json
│   │   │   └── timing.json
│   │   └── without_skill/
│   │       ├── outputs/
│   │       ├── grading.json
│   │       └── timing.json
│   ├── benchmark.json
│   └── benchmark.md
├── iteration-2/
│   └── ...
└── history.json
```

#### 7.3 Grading (оценка результатов)

После завершения прогонов - запустить grader-субагент для каждого прогона.

Grader читает `agents/grader.md` и оценивает каждый assertion по output-файлам. Результат сохраняет в `grading.json`.

Для assertions, которые можно проверить программно (файл существует, содержит формулу, количество слайдов) - написать и запустить скрипт проверки. Скрипты надёжнее визуальной оценки.

#### 7.4 Агрегация и анализ

1. **Агрегировать** результаты:
```bash
python ~/.claude/skills/skill-builder/scripts/aggregate_benchmark.py \
  <workspace>/iteration-1 --skill-name <name>
```
Создаёт `benchmark.json` и `benchmark.md` с pass_rate, timing, tokens для каждой конфигурации.

2. **Analyst pass** - запустить analyzer-субагент (читает `agents/analyzer.md`), который находит паттерны:
   - Assertions, которые всегда PASS в обеих конфигурациях (не дискриминируют)
   - Высокая variance (flaky)
   - Tradeoffs время/токены vs качество

3. **Показать результаты** пользователю: benchmark.md + конкретные примеры output. Спросить feedback.

#### 7.5 Итерационный цикл

После feedback пользователя:

1. **Обобщить feedback** - изменения должны быть general, не overfit к конкретным примерам
2. **Держать промт lean** - убрать то, что не работает. Читать транскрипты - если скилл заставляет модель тратить время впустую, убрать эти части
3. **Объяснять why** - вместо ALWAYS/NEVER объяснить причину, модель адаптируется лучше
4. **Найти повторную работу** - если все прогоны независимо написали похожий скрипт, вынести его в `scripts/`
5. Применить улучшения к скиллу
6. Повторить прогоны в `iteration-2/`
7. Показать результаты пользователю

Продолжать пока:
- Пользователь доволен
- Все assertions PASS
- Прогресс остановился

#### 7.6 Трекинг итераций

Обновлять `history.json` в корне workspace после каждой итерации:

```json
{
  "started_at": "2026-03-07T10:00:00Z",
  "skill_name": "xlsx-maker",
  "current_best": "iteration-2",
  "iterations": [
    {"version": "iteration-1", "parent": null, "pass_rate": 0.65, "result": "baseline"},
    {"version": "iteration-2", "parent": "iteration-1", "pass_rate": 0.90, "result": "won", "is_current_best": true}
  ]
}
```

### Шаг 8: Упаковка

Скилл готов - он в `~/.claude/skills/<name>/`. Упаковка не требуется (установка локальная).

## Паттерны по типам скиллов

### Document & Asset Creation
- Встроенные style guides и шаблоны
- Чеклисты качества на каждом шаге
- Валидационные скрипты для output

### Workflow Automation
- Пошаговые процессы с валидацией
- Decision tree для ветвления
- Validation loop: изменение -> проверка -> исправление -> повтор

### MCP Enhancement
- Координация вызовов MCP-серверов
- Доменная экспертиза поверх MCP tools

## Частые ошибки и решения

**Скилл не триггерится:**
Причина: description слишком общий или не содержит слов, которые пользователь реально использует.
Решение: добавить конкретные триггерные фразы, синонимы, прогнать `optimize_description.py`.

**Скилл триггерится на чужие запросы:**
Причина: description пересекается с другими скиллами.
Решение: сузить scope, сделать description более специфичным.

**Инструкции не выполняются:**
Причина: body слишком длинный, Claude теряет фокус.
Решение: сократить до <500 строк, вынести детали в references/, объяснить ПОЧЕМУ каждый шаг важен.

## Ссылки

- `references/complete-guide.md` - полное руководство Anthropic
- `references/best-practices.md` - лучшие практики
- `references/examples.md` - примеры готовых скиллов
- `references/schemas.md` - JSON-схемы для eval loop (evals.json, grading.json, benchmark.json, timing.json)
- `agents/grader.md` - инструкции для grader-субагента (оценка assertions)
- `agents/analyzer.md` - инструкции для analyzer-субагента (анализ бенчмарков)
- `scripts/validate_skill.py` - валидатор frontmatter
- `scripts/aggregate_benchmark.py` - агрегация результатов прогонов в benchmark.json/md
- `scripts/generate_eval_set.py` - генерация тестовых запросов для триггеринга
- `scripts/run_eval.py` - оценка триггеринга
- `scripts/optimize_description.py` - оптимизация description
