🌍 [English](../../../best-practice/claude-cli-startup-flags.md) | [中文](../../zh/best-practice/claude-cli-startup-flags.md) | [日本語](../../ja/best-practice/claude-cli-startup-flags.md) | [Français](../../fr/best-practice/claude-cli-startup-flags.md) | **Русский** | [العربية](../../ar/best-practice/claude-cli-startup-flags.md)

# Лучшие практики CLI Startup Flags

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026-white?style=flat&labelColor=555)

Справочник по флагам запуска Claude Code, подкомандам верхнего уровня и переменным окружения при запуске Claude Code из терминала.

<table width="100%">
<tr>
<td><a href="../README.md">← Назад к Claude Code Best Practice</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Содержание

1. [Управление сессиями](#управление-сессиями)
2. [Модель и конфигурация](#модель-и-конфигурация)
3. [Разрешения и безопасность](#разрешения-и-безопасность)
4. [Вывод и формат](#вывод-и-формат)
5. [Системный промпт](#системный-промпт)
6. [Agent и Subagent](#agent-и-subagent)
7. [MCP и Plugins](#mcp-и-plugins)
8. [Директории и рабочее пространство](#директории-и-рабочее-пространство)
9. [Бюджет и лимиты](#бюджет-и-лимиты)
10. [Интеграция](#интеграция)
11. [Инициализация и обслуживание](#инициализация-и-обслуживание)
12. [Отладка и диагностика](#отладка-и-диагностика)
13. [Переопределение настроек](#переопределение-настроек)
14. [Версия и справка](#версия-и-справка)
15. [Подкоманды](#подкоманды)
16. [Переменные окружения](#переменные-окружения)

---

## Управление сессиями

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--continue` | `-c` | Продолжить последний разговор в текущей директории |
| `--resume` | `-r` | Возобновить конкретную сессию по ID или имени, или показать интерактивный выбор |
| `--from-pr <NUMBER\|URL>` | | Возобновить сессии, связанные с конкретным GitHub PR |
| `--fork-session` | | Создать новый ID сессии при возобновлении (используйте с `--resume` или `--continue`) |
| `--session-id <UUID>` | | Использовать конкретный ID сессии (должен быть валидный UUID) |
| `--no-session-persistence` | | Отключить сохранение сессии (только print mode) |
| `--remote` | | Создать новую веб-сессию на claude.ai |
| `--teleport` | | Возобновить веб-сессию в локальном терминале |

---

## Модель и конфигурация

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--model <NAME>` | | Установить модель алиасом (`sonnet`, `opus`, `haiku`) или полным ID |
| `--fallback-model <NAME>` | | Автоматическая резервная модель при перегрузке основной (только print mode) |
| `--betas <LIST>` | | Beta-заголовки для API-запросов (только для пользователей с API-ключом) |

---

## Разрешения и безопасность

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--dangerously-skip-permissions` | | Пропустить ВСЕ запросы разрешений. Используйте с крайней осторожностью |
| `--allow-dangerously-skip-permissions` | | Включить возможность обхода разрешений без активации |
| `--permission-mode <MODE>` | | Начать в указанном режиме разрешений: `default`, `plan`, `acceptEdits`, `bypassPermissions` |
| `--allowedTools <TOOLS>` | | Инструменты, выполняемые без запроса (синтаксис правил разрешений) |
| `--disallowedTools <TOOLS>` | | Инструменты, полностью удалённые из контекста модели |
| `--tools <TOOLS>` | | Ограничить доступные встроенные инструменты Claude (используйте `""` для отключения всех) |
| `--permission-prompt-tool <TOOL>` | | Указать инструмент MCP для обработки запросов разрешений в неинтерактивном режиме |

---

## Вывод и формат

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--print` | `-p` | Вывести ответ без интерактивного режима (headless/SDK mode) |
| `--output-format <FORMAT>` | | Формат вывода: `text`, `json`, `stream-json` |
| `--input-format <FORMAT>` | | Формат ввода: `text`, `stream-json` |
| `--json-schema <SCHEMA>` | | Получить валидированный JSON по схеме (только print mode) |
| `--include-partial-messages` | | Включить частичные события стриминга (требует `--print` и `--output-format=stream-json`) |
| `--verbose` | | Включить подробное логирование с полным пошаговым выводом |

---

## Системный промпт

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--system-prompt <TEXT>` | | Заменить весь системный промпт пользовательским текстом |
| `--system-prompt-file <PATH>` | | Загрузить системный промпт из файла, заменив стандартный (только print mode) |
| `--append-system-prompt <TEXT>` | | Добавить пользовательский текст к стандартному системному промпту |
| `--append-system-prompt-file <PATH>` | | Добавить содержимое файла к стандартному промпту (только print mode) |

---

## Agent и Subagent

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--agent <NAME>` | | Указать agent для текущей сессии |
| `--agents <JSON>` | | Определить пользовательские subagents динамически через JSON |
| `--teammate-mode <MODE>` | | Установить режим отображения agent team: `auto`, `in-process`, `tmux` |

---

## MCP и Plugins

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--mcp-config <PATH\|JSON>` | | Загрузить MCP servers из JSON-файла или строки |
| `--strict-mcp-config` | | Использовать только MCP servers из `--mcp-config`, игнорировать остальные |
| `--plugin-dir <PATH>` | | Загрузить plugins из директории только для этой сессии (можно повторять) |

---

## Директории и рабочее пространство

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--add-dir <PATH>` | | Добавить дополнительные рабочие директории для доступа Claude |
| `--worktree` | `-w` | Запустить Claude в изолированном git worktree (ветка от HEAD) |

---

## Бюджет и лимиты

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--max-budget-usd <AMOUNT>` | | Максимальная сумма в долларах для API-вызовов перед остановкой (только print mode) |
| `--max-turns <NUMBER>` | | Ограничить количество агентных ходов (только print mode) |

---

## Интеграция

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--chrome` | | Включить интеграцию с браузером Chrome для веб-автоматизации |
| `--no-chrome` | | Отключить интеграцию с Chrome для этой сессии |
| `--ide` | | Автоматически подключиться к IDE при запуске, если доступна ровно одна подходящая IDE |

---

## Инициализация и обслуживание

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--init` | | Запустить hooks инициализации и начать интерактивный режим |
| `--init-only` | | Запустить hooks инициализации и выйти (без интерактивной сессии) |
| `--maintenance` | | Запустить hooks обслуживания и выйти |

---

## Отладка и диагностика

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--debug <CATEGORIES>` | | Включить режим отладки с фильтрацией по категориям (напр. `"api,hooks"`) |

---

## Переопределение настроек

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--settings <PATH\|JSON>` | | Путь к JSON-файлу настроек или JSON-строка для загрузки |
| `--setting-sources <LIST>` | | Список источников для загрузки через запятую: `user`, `project`, `local` |
| `--disable-slash-commands` | | Отключить все skills и slash commands для этой сессии |

---

## Версия и справка

| Флаг | Короткий | Описание |
|------|----------|----------|
| `--version` | `-v` | Вывести номер версии |
| `--help` | `-h` | Показать справку |

---

## Подкоманды

Это команды верхнего уровня, запускаемые как `claude <subcommand>`:

| Подкоманда | Описание |
|------------|----------|
| `claude` | Запустить интерактивный REPL |
| `claude "query"` | Запустить REPL с начальным промптом |
| `claude agents` | Список настроенных agents |
| `claude auth` | Управление аутентификацией Claude Code |
| `claude doctor` | Запуск диагностики из командной строки |
| `claude install` | Установка или переключение нативных сборок Claude Code |
| `claude mcp` | Настройка MCP servers (`add`, `remove`, `list`, `get`, `enable`) |
| `claude plugin` | Управление plugins Claude Code |
| `claude remote-control` | Управление сессиями удалённого управления |
| `claude setup-token` | Создание долгоживущего токена для использования подписки |
| `claude update` / `claude upgrade` | Обновление до последней версии |

---

## Переменные окружения

Эти переменные окружения задаются в shell перед запуском Claude Code (их нельзя настроить через `settings.json`):

| Переменная | Описание |
|-----------|----------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | Включить экспериментальные agent teams |
| `CLAUDE_CODE_TMPDIR` | Переопределить директорию для временных файлов. Также настраивается через ключ `env` — см. [Справочник Settings](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | Включить загрузку CLAUDE.md из дополнительных директорий |
| `DISABLE_AUTOUPDATER=1` | Отключить автообновление |
| `CLAUDE_CODE_EFFORT_LEVEL` | Управление глубиной размышлений — см. [Справочник Settings](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `USE_BUILTIN_RIPGREP=0` | Использовать системный ripgrep вместо встроенного (Alpine Linux) |
| `CLAUDE_CODE_SIMPLE` | Включить упрощённый режим (только инструменты Bash + Edit). Также настраивается через ключ `env` — см. [Справочник Settings](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_BASH_NO_LOGIN=1` | Пропустить login shell для BashTool |

Для переменных окружения, настраиваемых через ключ `"env"` в `settings.json` (включая `MAX_THINKING_TOKENS`, `CLAUDE_CODE_SHELL`, `CLAUDE_CODE_ENABLE_TASKS`, `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`, `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` и другие), см. [Справочник Settings Claude Code](../../../best-practice/claude-settings.md#environment-variables-via-env).

---

## Источники

- [Справочник CLI Claude Code](https://code.claude.com/docs/en/cli-reference)
- [Headless Mode Claude Code](https://code.claude.com/docs/en/headless)
- [Установка Claude Code](https://code.claude.com/docs/en/setup)
- [CHANGELOG Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Типичные рабочие процессы Claude Code](https://code.claude.com/docs/en/common-workflows)
