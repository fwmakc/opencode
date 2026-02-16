# OpenCode Pipeline Project

Проект с автоматизированным конвейером разработки на основе OpenCode AI агентов.

## Архитектура

Система состоит из 4 агентов, работающих в конвейере:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Аналитик   │ →  │   Кодер     │ →  │  Ревьюер    │ →  │  Деплойер   │
│ (PLAN.md)   │    │(CODER_REPORT│    │(REVIEW_REPORT│    │(DEPLOY_STATUS│
└─────────────┘    │  .md)       │    │  .md)       │    │  .md)       │
                   └─────────────┘    └─────────────┘    └─────────────┘
```

## Агенты

| Агент      | Режим    | Описание                           | Инструменты       |
| ---------- | -------- | ---------------------------------- | ----------------- |
| `build`    | primary  | Основной режим разработки          | write, edit, bash |
| `plan`     | primary  | Режим планирования (только чтение) | none              |
| `analyst`  | subagent | Аналитик задач                     | none              |
| `coder`    | subagent | TypeScript разработчик             | write, edit, bash |
| `reviewer` | subagent | Ревьюер безопасности               | bash (read-only)  |
| `deployer` | subagent | DevOps деплой                      | bash (read-only)  |

## Установка

1. Установи OpenCode CLI:

```bash
npm install -g opencode-ai
```

2. Настрой API ключ:

```bash
opencode
/connect
```

3. Установи настройки:

```bash
git clone https://github.com/fwmakc/opencode.git .opencode
```

4. Инициализируй проект:

```bash
opencode
/init
```

## Конфигурация

Задай правильные модели в файле `opencode.json` и файлах агентов `agents`.

Например:

```
model: litellm/qwen3-coder-next
```

> Без этого агент не запустится!

Можно более детально сконфигурировать промты и скилы агентов под свои задачи.

## Использование

### Полный пайплайн (одна команда)

Введи в TUI opencode:

```
/pipeline text of your promt
```

Команда автоматически запустит:

1. **Аналитик** → PLAN.md
2. **Кодер** → CODER_REPORT.md
3. **Ревьюер** → REVIEW_REPORT.md (APPROVED/REJECTED)
4. **Деплойер** → DEPLOY_STATUS.md

При `REJECTED` вернется к Кодеру для исправления (макс. 3 попытки).

### Отдельные команды

| Команда   | Описание                        |
| --------- | ------------------------------- |
| `/plan`   | Создать план разработки         |
| `/coder`  | Реализовать функционал по плану |
| `/review` | Проверить код на качество       |
| `/deploy` | Развернуть и проверить          |

### Использование субагентов напрямую

В TUI opencode:

```
@analyst проанализируй задачу: ...
@coder реализуй функционал: ...
@reviewer проверь код: ...
@deployer разверни проект: ...
```

## Файлы проекта

```
.opencode/
├── agents/               # Агенты (субагенты)
│   ├── analyst.md
│   ├── coder.md
│   ├── reviewer.md
│   └── deployer.md
├── commands/             # Пользовательские команды
│   ├── pipeline.md
│   ├── plan.md
│   ├── coder.md
│   ├── review.md
│   └── deploy.md
├── opencode.json         # Конфигурация агентов
└── AGENTS.md             # Документация агентов
```

## Настройка

Измени `.opencode/opencode.json` для настройки:

- `model` - модель ИИ для каждого агента
- `temperature` - креативность (0.0-1.0)
- `tools` - доступные инструменты
- `permission` - разрешения на действия

## MCP-серверы

Проект настроен на использование Model Context Protocol серверов.

### Установленные MCP-серверы:

| Сервер          | Тип   | Команда                                             | Описание                                                 |
| --------------- | ----- | --------------------------------------------------- | -------------------------------------------------------- |
| `playwright`    | local | `npx -y @modelcontextprotocol/server-playwright`    | Браузерное тестирование, скриншоты, взаимодействие с DOM |
| `google-search` | local | `npx -y @modelcontextprotocol/server-google-search` | Поиск в Google                                           |

### Настройка:

Смотрите `.opencode/opencode.json` в секции `mcp`:

```json
"mcp": {
  "playwright": {
    "type": "local",
    "command": ["npx", "-y", "@modelcontextprotocol/server-playwright"],
    "enabled": true
  },
  "google-search": {
    "type": "local",
    "command": ["npx", "-y", "@modelcontextprotocol/server-google-search"],
    "enabled": true
  }
}
```

### Использование:

- **Playwright**: `@playwright navigate to page`, `take screenshot`, `click button`
- **Google Search**: `@google-search search for "query"`

### Установка зависимостей:

```bash
npm install -D @modelcontextprotocol/server-playwright @modelcontextprotocol/server-google-search
```

## LSP-серверы

Проект настроен на использование LSP-серверов для TypeScript и JavaScript.

### Встроенные серверы:

| Сервер       | Расширения                                                           | Требования                    |
| ------------ | -------------------------------------------------------------------- | ----------------------------- |
| `typescript` | `.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs`, `.mts`, `.cts`         | `typescript` в `package.json` |
| `eslint`     | `.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs`, `.mts`, `.cts`, `.vue` | `eslint` в `package.json`     |

### Настройка:

Смотрите `.opencode/opencode.json` в секции `lsp`:

```json
"lsp": {
  "typescript": {
    "initialization": {
      "preferences": {
        "importModuleSpecifierPreference": "relative"
      }
    }
  },
  "eslint": {
    "initialization": {
      "format": "pretty"
    }
  }
}
```

### Требования

- Node.js 18+
- Один из поддерживаемых провайдеров LLM (OpenAI, Anthropic, OpenCode Zen и др.)
- `npm install` - для установки зависимостей (typescript, eslint)
