🌍 [English](../../../best-practice/claude-subagents.md) | [中文](../../zh/best-practice/claude-subagents.md) | [日本語](../../ja/best-practice/claude-subagents.md) | [Français](../../fr/best-practice/claude-subagents.md) | **Русский** | [العربية](../../ar/best-practice/claude-subagents.md)

# Лучшие практики Sub-agents

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A34%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-subagents-implementation.md)

Subagents Claude Code — поля frontmatter и официальные встроенные типы агентов.

<table width="100%">
<tr>
<td><a href="../README.md">← Назад к Claude Code Best Practice</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Поля Frontmatter (16)

| Поле | Тип | Обязательное | Описание |
|------|-----|-------------|----------|
| `name` | string | Да | Уникальный идентификатор из строчных букв и дефисов |
| `description` | string | Да | Когда вызывать. Используйте `"PROACTIVELY"` для автоматического вызова Claude |
| `tools` | string/list | Нет | Список разрешённых инструментов через запятую (напр. `Read, Write, Edit, Bash`). Наследует все инструменты, если не указан. Поддерживает синтаксис `Agent(agent_type)` для ограничения порождаемых subagents; старый алиас `Task(agent_type)` тоже работает |
| `disallowedTools` | string/list | Нет | Запрещённые инструменты, удаляемые из унаследованного или указанного списка |
| `model` | string | Нет | Модель: `sonnet`, `opus`, `haiku`, полный ID модели (напр. `claude-opus-4-6`) или `inherit` (по умолчанию: `inherit`) |
| `permissionMode` | string | Нет | Режим разрешений: `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions` или `plan` |
| `maxTurns` | integer | Нет | Максимальное количество агентных ходов до остановки subagent |
| `skills` | list | Нет | Имена skills для предзагрузки в контекст агента при запуске (полное содержимое внедряется, а не просто становится доступным) |
| `mcpServers` | list | Нет | MCP servers для этого subagent — строки с именами серверов или inline-объекты `{name: config}` |
| `hooks` | object | Нет | Lifecycle hooks, привязанные к этому subagent. Поддерживаются все события hooks; наиболее распространены `PreToolUse`, `PostToolUse` и `Stop` |
| `memory` | string | Нет | Область постоянной памяти: `user`, `project` или `local` |
| `background` | boolean | Нет | Установите `true` для постоянного запуска как фоновой задачи (по умолчанию: `false`) |
| `effort` | string | Нет | Переопределение уровня усилий при активном subagent: `low`, `medium`, `high`, `max` (только Opus 4.6). По умолчанию: наследуется от сессии |
| `isolation` | string | Нет | Установите `"worktree"` для запуска во временном git worktree (автоматически очищается при отсутствии изменений) |
| `initialPrompt` | string | Нет | Автоматически отправляется как первый пользовательский ход при запуске агента как основного сессионного агента (через `--agent` или настройку `agent`). Commands и skills обрабатываются. Добавляется перед любым промптом пользователя |
| `color` | string | Нет | Цвет отображения subagent в списке задач и транскрипте: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink` или `cyan` |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Agent | Модель | Инструменты | Описание |
|---|-------|--------|-------------|----------|
| 1 | `general-purpose` | inherit | Все | Сложные многоэтапные задачи — тип агента по умолчанию для исследований, поиска по коду и автономной работы |
| 2 | `Explore` | haiku | Только чтение (без Write, Edit) | Быстрый поиск и исследование кодовой базы — оптимизирован для поиска файлов, поиска по коду и ответов на вопросы о кодовой базе |
| 3 | `Plan` | inherit | Только чтение (без Write, Edit) | Предварительное исследование в plan mode — изучает кодовую базу и разрабатывает подходы к реализации перед написанием кода |
| 4 | `statusline-setup` | sonnet | Read, Edit | Настраивает status line в Claude Code пользователя |
| 5 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | Отвечает на вопросы о функциях Claude Code, Agent SDK и Claude API |

---

## Источники

- [Создание пользовательских subagents — Документация Claude Code](https://code.claude.com/docs/en/sub-agents)
- [Справочник CLI — Документация Claude Code](https://code.claude.com/docs/en/cli-reference)
- [CHANGELOG Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
