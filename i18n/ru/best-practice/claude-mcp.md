🌍 [English](../../../best-practice/claude-mcp.md) | [中文](../../zh/best-practice/claude-mcp.md) | [日本語](../../ja/best-practice/claude-mcp.md) | [Français](../../fr/best-practice/claude-mcp.md) | **Русский** | [العربية](../../ar/best-practice/claude-mcp.md)

# Лучшие практики MCP Servers

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026%2012%3A30%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.mcp.json)

MCP (Model Context Protocol) servers расширяют Claude Code подключениями к внешним инструментам, базам данных и API. Это руководство охватывает рекомендуемые серверы для повседневного использования и лучшие практики конфигурации.

<table width="100%">
<tr>
<td><a href="../README.md">← Назад к Claude Code Best Practice</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## MCP Servers для повседневного использования

> *«Увлёкся 15 MCP servers, думая что больше = лучше. В итоге ежедневно использую только 4.»* — [r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/) (682 голоса)

| MCP Server | Что делает | Ресурсы |
|------------|-----------|---------|
| [**Context7**](https://github.com/upstash/context7) | Загружает актуальную документацию библиотек в контекст. Предотвращает галлюцинации API из устаревших данных обучения | [Reddit: «безусловно лучший MCP для кодинга»](https://reddit.com/r/mcp/comments/1qarjqm/) · [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | Автоматизация браузера — реализация, тестирование и проверка UI-функций автономно. Скриншоты, навигация, тестирование форм | [Reddit: незаменим для фронтенда](https://reddit.com/r/mcp/comments/1m59pk0/) · [Документация](https://playwright.dev/) |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | Подключает Claude к вашему реальному браузеру Chrome — инспекция консоли, сети, DOM. Отладка того, что видят пользователи | [Reddit: «game changer» для отладки](https://reddit.com/r/mcp/comments/1qarjqm/5_mcps_that_have_genuinely_made_me_10x_faster/nza0i7t/) · [Сравнительный отчёт](../../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | Загружает структурированную wiki-документацию для любого GitHub-репозитория — архитектура, API-поверхность, взаимосвязи | [Reddit: «используйте за шлюзом вместе с Context7»](https://reddit.com/r/mcp/comments/1qarjqm/) |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | Генерация архитектурных диаграмм, блок-схем и системных дизайнов в виде рисованных скетчей Excalidraw по промптам | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

Исследование (Context7/DeepWiki) -> Отладка (Playwright/Chrome) -> Документирование (Excalidraw)

---

## Конфигурация

MCP servers настраиваются в `.mcp.json` в корне проекта (на уровне проекта) или в `~/.claude.json` (на уровне пользователя).

### Типы серверов

| Тип | Транспорт | Пример |
|-----|-----------|--------|
| **stdio** | Запускает локальный процесс | `npx`, `python`, бинарник |
| **http** | Подключается к удалённому URL | HTTP/SSE endpoint |

### Пример `.mcp.json`

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

Используйте подстановку переменных окружения для секретов вместо коммита API-ключей в `.mcp.json`:

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

### Настройки для MCP Servers

Эти настройки в `.claude/settings.json` контролируют одобрение MCP servers:

| Ключ | Тип | Описание |
|------|-----|----------|
| `enableAllProjectMcpServers` | boolean | Автоматическое одобрение всех серверов из `.mcp.json` без запроса |
| `enabledMcpjsonServers` | array | Белый список конкретных имён серверов для автоодобрения |
| `disabledMcpjsonServers` | array | Чёрный список конкретных имён серверов для отклонения |

### Правила разрешений для инструментов MCP

Инструменты MCP следуют соглашению об именовании `mcp__<server>__<tool>` в правилах разрешений:

```json
{
  "permissions": {
    "allow": [
      "mcp__*",
      "mcp__context7__*",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

---

## Области действия MCP

MCP servers могут быть определены на трёх уровнях:

| Область | Расположение | Назначение |
|---------|-------------|----------|
| **Проект** | `.mcp.json` (корень репозитория) | Серверы, общие для команды, коммитятся в git |
| **Пользователь** | `~/.claude.json` (ключ `mcpServers`) | Персональные серверы для всех проектов |
| **Subagent** | Frontmatter агента (поле `mcpServers`) | Серверы, привязанные к конкретному subagent |

Приоритет: Subagent > Проект > Пользователь

---

## Источники

- [MCP Servers — Документация Claude Code](https://code.claude.com/docs/en/mcp)
- [Спецификация Model Context Protocol](https://modelcontextprotocol.io/)
- [5 MCP, которые действительно ускорили меня в 10 раз — r/mcp](https://reddit.com/r/mcp/comments/1qarjqm/)
- [Обсуждение перегрузки MCP Server — r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)
