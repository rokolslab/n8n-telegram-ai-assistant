# n8n Telegram AI Assistant

[English version](README.md)

Портфельный проект на n8n: Telegram-бот принимает сообщения, нормализует входные данные, показывает статус обработки, маршрутизирует команды и использует OpenAI для генерации текста и изображений.

> Основной акцент — на оркестрации workflow, Telegram Bot API, безопасных n8n Expressions, маршрутизации команд и реальных webhook-запусках.

![Архитектура](docs/images/architecture.svg)

## Что делает workflow

```text
Сообщение Telegram
       │
       ▼
Нормализация входа
       │
       ▼
Настройки ─────────────► Chat Action
       │                    │
       └──────────────► Merge ◄──┘
                          │
                          ▼
                    Route Command
                    /      |      \
               /start   /image   fallback
                  │        │         │
                  ▼        ▼         ▼
               OpenAI    OpenAI    Help message
                text      image
                  │        │
                  ▼        ▼
              Telegram  Telegram
                text      photo
```

## Поддерживаемые команды

- `/start` — генерирует приветствие и кратко объясняет доступные команды.
- `/image <описание>` — генерирует изображение `1024x1024` и отправляет его в Telegram.
- остальные команды, включая `/help` в протестированной реализации, попадают в fallback и получают контролируемое сообщение-подсказку.

## Что демонстрирует проект

- **Безопасная нормализация Telegram update.** Optional chaining и значения по умолчанию снижают риск падения на неполных входных данных.
- **Разбор команд через Expressions.** Команда и prompt извлекаются детерминированно до обращения к AI.
- **Статус обработки.** Бот отправляет `typing` или `upload_photo` в зависимости от выбранной ветки.
- **Явная маршрутизация.** `Switch` разделяет text, image и fallback сценарии.
- **Генерация текста и изображений через OpenAI.** `/start` использует text generation, `/image` — image generation.
- **Работа с binary data.** Сгенерированное изображение передаётся в Telegram send-photo node.
- **Fallback без лишнего AI-вызова.** Неподдерживаемые команды не расходуют модельный вызов.

## Что фактически проверено

Исходная реализация тестировалась реальными Telegram webhook-запусками.

| Команда | Результат |
|---|---|
| `/start` | Успешный запуск; ответ OpenAI отправлен в Telegram |
| `/image футуристический офис с ИИ-ассистентом` | Успешный запуск; PNG `1024x1024` отправлен в Telegram |
| `/help` | Успешный fallback; сообщение-подсказка отправлено в Telegram |

Во время тестирования были обнаружены и исправлены две runtime-проблемы: неподдерживаемое значение OpenAI `text.verbosity` и выражение отправки текста, из-за которого Telegram мог получить JSON-объект вместо чистого текста модели.

Подробнее: [Testing](docs/testing.md) и [Evidence](docs/evidence.md).

## Стек

`n8n` · `Telegram Bot API` · `OpenAI` · `GPT image generation` · `Expressions` · `Switch` · `Merge`

## Структура репозитория

```text
.
├── workflow/
│   └── telegram-ai-assistant.json
├── examples/
│   └── commands.md
├── docs/
│   ├── architecture.md
│   ├── setup.md
│   ├── testing.md
│   ├── evidence.md
│   └── images/
│       └── architecture.svg
├── .gitignore
├── LICENSE
├── README.md
└── README.ru.md
```

## Быстрый запуск

1. Импортируйте [`workflow/telegram-ai-assistant.json`](workflow/telegram-ai-assistant.json) в n8n.
2. Подключите собственные Telegram и OpenAI credentials.
3. Проверьте доступность выбранных text/image моделей и при необходимости замените их эквивалентами в своей среде.
4. Включите test/listening mode либо активируйте workflow, чтобы Telegram мог обратиться к webhook.
5. Отправьте команды из [`examples/commands.md`](examples/commands.md).
6. Проверьте text, image и fallback ветки до использования workflow за пределами тестового бота.

Подробности: [Setup](docs/setup.md).

## Ключевые Expressions

Безопасное извлечение текста:

```text
{{ $json?.message?.text ?? "" }}
```

Выделение команды:

```text
{{ ($json?.message?.text ?? "").trim().split(/\s+/)[0]?.toLowerCase() || "" }}
```

Извлечение prompt для `/image`:

```text
{{ ($json?.message?.text ?? "").replace(/^\/image\s*/i, "").trim() }}
```

Выбор Telegram chat action:

```text
{{ $json.message.command === "/image" ? "upload_photo" : "typing" }}
```

## Безопасность и переносимость

Публичный экспорт workflow очищен от исходных workflow/version ID, webhook UUID, Telegram bot token, OpenAI API key и ссылок на credentials.

После импорта credentials подключаются вручную. Для тестирования лучше использовать отдельного Telegram-бота и не публиковать скриншоты с bot token, приватными chat data, именами credentials или production-идентификаторами.

## Ограничения

- Workflow ориентирован на команды, а не на свободный диалог.
- Если `/image` отправлен без описания, используется default prompt; в production можно заменить это явной валидацией.
- Все неподдерживаемые команды обрабатываются одной fallback-веткой.
- Для production следует добавить централизованную обработку ошибок, observability, rate limiting, anti-abuse механизмы и при необходимости квоты на image generation.

## Лицензия

MIT — см. [`LICENSE`](LICENSE).
