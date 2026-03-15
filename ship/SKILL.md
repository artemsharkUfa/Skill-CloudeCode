---
name: ship
description: |
  Полная автоматизация релиза: merge main, тесты, ревью, changelog, push, PR.
  Одна команда — готовый PR. Используй когда: ветка готова к мержу. Команда: /ship
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# Ship: Полностью автоматический релиз

Ты выполняешь `/ship`. Это **неинтерактивный, полностью автоматизированный** workflow. НЕ спрашивай подтверждения на каждом шаге. Пользователь сказал `/ship` — значит ДЕЛАЙ.

**Останавливайся ТОЛЬКО если:**
- На ветке `main` (abort)
- Merge conflicts которые нельзя авто-разрешить
- Тесты упали
- Pre-landing review нашёл КРИТИЧЕСКИЕ проблемы
- Пользователь не авторизован в `gh`

**НЕ останавливайся для:**
- Uncommitted changes (всегда включай их)
- Выбор версии (auto-decide)
- CHANGELOG контент (авто-генерация)
- Commit message (авто-генерация)

---

## Шаг 1: Pre-flight

```bash
BRANCH=$(git branch --show-current)
echo "Branch: $BRANCH"
```

Если на `main` — **abort**: "Ты на main. Ship только с feature branch."

```bash
git status
git diff main...HEAD --stat 2>/dev/null
git log main..HEAD --oneline 2>/dev/null
```

---

## Шаг 2: Merge origin/main

Синхронизация с main ДО тестов:

```bash
git fetch origin main --quiet && git merge origin/main --no-edit
```

**Merge conflicts:** попробуй авто-разрешить простые (package-lock.json, CHANGELOG порядок). Сложные — **СТОП**, покажи конфликты.

**Уже синхронизировано:** продолжай молча.

---

## Шаг 3: Тесты

Авто-определи и запусти тесты:

```bash
# Определи тест-раннер
if [ -f "package.json" ]; then
  # Node.js
  if grep -q '"test"' package.json; then
    npm test 2>&1 | tail -20
  fi
  # Vitest / Jest
  if grep -q '"vitest\|jest"' package.json; then
    npx vitest run 2>&1 | tail -20 || npx jest 2>&1 | tail -20
  fi
fi

if [ -f "pytest.ini" ] || [ -f "pyproject.toml" ] || [ -d "tests" ]; then
  # Python
  python -m pytest 2>&1 | tail -20 || true
fi

if [ -f "phpunit.xml" ] || [ -f "phpunit.xml.dist" ]; then
  # PHP
  ./vendor/bin/phpunit 2>&1 | tail -20 || true
fi
```

**Тесты упали → СТОП.** Покажи ошибки.
**Тесты прошли → продолжай.** Кратко отметь количество.
**Тестов нет → предупреди**, но продолжай.

---

## Шаг 4: Pre-Landing Review

Прочитай `~/.claude/skills/pre-landing-review/SKILL.md` и выполни ревью diff.

Если файл недоступен — выполни упрощённый чек:

```bash
git diff origin/main
```

Проверь на:
- SQL injection / string interpolation в запросах
- XSS: innerHTML, dangerouslySetInnerHTML с пользовательскими данными
- Пустые catch блоки
- console.log / print() дебаг-вывод
- Захардкоженные секреты, API ключи
- Race conditions

**КРИТИЧЕСКИЕ найдены:** для каждой — покажи проблему, предложи фикс, варианты (А/Б/В). **СТОП**, жди ответ.

Если пользователь выбрал "Исправить" — примени фиксы, закоммить, и скажи запустить `/ship` заново для ретеста.

**Только информационные или нет проблем:** продолжай.

Сохрани результат ревью — пойдёт в PR body.

---

## Шаг 5: Version bump (если есть)

Определи систему версионирования:

```bash
# package.json
[ -f "package.json" ] && grep '"version"' package.json

# VERSION file
[ -f "VERSION" ] && cat VERSION

# Git tags
git tag --sort=-v:refname | head -5
```

**Если есть package.json с version:**
- < 50 строк изменений → patch bump
- 50+ строк → minor bump

```bash
# Пример для npm
npm version patch --no-git-tag-version 2>/dev/null
```

**Если VERSION файл:** инкрементируй аналогично.

**Если ничего нет:** пропусти этот шаг.

---

## Шаг 6: CHANGELOG (авто-генерация)

Если есть `CHANGELOG.md`:

1. Прочитай формат существующих записей
2. Сгенерируй запись из всех коммитов на ветке:

```bash
git log main..HEAD --oneline
git diff main...HEAD --stat
```

3. Категоризируй:
   - `### Added` — новые фичи
   - `### Changed` — изменения существующего
   - `### Fixed` — баг-фиксы
   - `### Removed` — удалённое

4. Вставь после заголовка, датируй сегодняшним числом

**Если CHANGELOG.md нет:** пропусти.

---

## Шаг 7: Commit

Проанализируй diff и сгруппируй в логические коммиты:

**Для маленьких изменений (< 50 строк, < 4 файла):** один коммит.

**Для больших:** разбей по логике:
1. Инфраструктура (миграции, конфиг)
2. Модели/сервисы + их тесты
3. Контроллеры/views + их тесты
4. VERSION + CHANGELOG (финальный коммит)

```bash
# Добавляй конкретные файлы, НЕ используй git add -A
git add <конкретные-файлы>
git commit -m "$(cat <<'EOF'
feat: краткое описание изменения

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
EOF
)"
```

**Каждый коммит должен быть валидным** — без сломанных импортов.

---

## Шаг 8: Push

```bash
git push -u origin $(git branch --show-current)
```

---

## Шаг 9: Создать PR

```bash
gh pr create --title "<type>: <summary>" --body "$(cat <<'EOF'
## Summary
<буллеты из CHANGELOG или из анализа коммитов>

## Pre-Landing Review
<результаты из Шага 4, или "Проблем не найдено.">

## Test plan
- [x] Тесты пройдены (N тестов)
- [ ] Ручная проверка на staging

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**Выведи URL созданного PR** — это последнее что видит пользователь.

---

## Итог

Цель: пользователь говорит `/ship`, следующее что он видит — URL готового PR.

```
/ship
  ├─ Pre-flight (проверка ветки, статус)
  ├─ Merge main (синхронизация)
  ├─ Tests (авто-определение, запуск)
  ├─ Review (pre-landing чеклист)
  ├─ Version bump (если есть)
  ├─ CHANGELOG (авто-генерация)
  ├─ Commit (логические chunks)
  ├─ Push (с upstream tracking)
  └─ PR (gh pr create → URL)
```
