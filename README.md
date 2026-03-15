# Claude Code Skills Collection

Коллекция кастомных скиллов для Claude Code — AI-ассистента в терминале от Anthropic.

## Установка

Скопируй нужный скилл в `~/.claude/skills/`:

```bash
# Один скилл
cp -r plan-ceo-review ~/.claude/skills/

# Все скиллы
cp -r */ ~/.claude/skills/
```

## Скиллы

### Разработка
| Скилл | Описание |
|---|---|
| `frontend-agent` | Design-to-code: конвертирует дизайн/скриншот в HTML+CSS+JS с анимациями |
| `vibe-backend-agent` | Генерация REST API с auth, БД, тестами, swagger |
| `telegram-bot-builder` | Telegram-боты на Node.js с UI, кнопками, AI |
| `vps-deploy-agent` | Деплой на Ubuntu VPS: nginx, pm2, docker, SSL |

### Планирование и ревью
| Скилл | Описание |
|---|---|
| `plan-ceo-review` | CEO/founder-mode ревью плана. 3 режима: расширение/удержание/сокращение |
| `pre-landing-review` | Чеклист ревью перед мержем: SQL, XSS, race conditions |
| `ship` | Полный автомат релиза: merge → тесты → ревью → PR |

### Контент и документы
| Скилл | Описание |
|---|---|
| `content-maker` | Посты для Telegram-каналов в авторском стиле |
| `prompt-master` | Промты для любых AI-моделей с веб-исследованием |
| `pptx-maker` | PowerPoint презентации с брендированными темами |
| `xlsx-maker` | Excel таблицы с формулами и стилями |
| `site-content` | Тексты для сайтов: заголовки, CTA, FAQ |

### SEO и аналитика
| Скилл | Описание |
|---|---|
| `seo-optimizer` | SEO по рекомендациям Яндекса и Google |
| `site-analytics` | Яндекс.Метрика + GA4 + Яндекс.Директ |
| `web-asset-generator` | Favicon, OG images, PWA icons |

### Интеграции
| Скилл | Описание |
|---|---|
| `gigachat-integrator` | Sber GigaChat API для Node.js, Python, PHP |
| `funnel-optimizer` | Оптимизация воронки продаж, A/B тесты |

### Утилиты
| Скилл | Описание |
|---|---|
| `skill-builder` | Создание новых скиллов для Claude Code |

## Требования

- [Claude Code](https://claude.com/code) 2.1.70+
- Node.js 18+ (для telegram-bot-builder, frontend-agent)
- Python 3.11+ (для pptx-maker, xlsx-maker)

## Лицензия

MIT
