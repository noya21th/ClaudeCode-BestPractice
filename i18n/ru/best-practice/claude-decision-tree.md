🌍 [English](../../../best-practice/claude-decision-tree.md) | [中文](../../zh/best-practice/claude-decision-tree.md) | [日本語](../../ja/best-practice/claude-decision-tree.md) | [Français](../../fr/best-practice/claude-decision-tree.md) | **Русский** | [العربية](../../ar/best-practice/claude-decision-tree.md)

# Agent, Skill или Command — дерево решений

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Когда использовать Agent, Skill или Command — фреймворк принятия решений с реальными сценариями.

<table width="100%">
<tr>
<td><a href="../">← Назад к Claude Code Best Practice</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Быстрое сравнение

| Параметр | Agent | Command | Skill |
|----------|-------|---------|-------|
| **Расположение** | `.claude/agents/<name>.md` | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |
| **Контекст** | Свежий изолированный контекст | Внедряется в текущий контекст | Внедряется в текущий контекст (или форк через `context: fork`) |
| **Вызов** | Инструмент `Agent(...)`, автообнаружение через description | `/slash-command` пользователем | `/slash-command`, автообнаружение или предзагрузка в Agent |
| **Память** | Постоянная (`user`/`project`/`local`) | Нет | Нет |
| **Инструменты** | Собственный список через `tools:` | Наследует инструменты сессии | Наследует инструменты сессии (или ограничение через `allowed-tools:`) |
| **Модель** | Настраиваемая (`haiku`/`sonnet`/`opus`/`inherit`) | Настраиваемая | Настраиваемая |
| **Разрешения** | Настраиваемые (`acceptEdits`/`plan`/`bypassPermissions`) | Наследует разрешения сессии | Наследует разрешения сессии |
| **Автообнаружение** | Да (через `description`, использовать `PROACTIVELY`) | Нет (только по инициативе пользователя) | Да (через `description`) |
| **Предзагрузка** | Нет | Нет | Да (через поле `skills:` в Agent) |
| **Фоновый режим** | Да (`background: true`) | Нет | Нет |
| **Изоляция worktree** | Да (`isolation: worktree`) | Нет | Нет |
| **Лучше всего для** | Сложных многошаговых задач, требующих изоляции | Рабочих процессов, запускаемых пользователем | Переиспользуемых доменных знаний и процедур |

---

## Блок-схема принятия решений

```
Старт: «Мне нужно добавить возможность в Claude Code»
│
├─ Нужен ли собственный изолированный контекст?
│  ├─ ДА ──→ Нужна ли постоянная память между сессиями?
│  │          ├─ ДА ──→ AGENT (с memory: user/project/local)
│  │          └─ НЕТ ──→ AGENT (или Skill с context: fork)
│  │
│  └─ НЕТ ──→ Должен ли только пользователь запускать это явно?
│             ├─ ДА ──→ COMMAND (пользователь вводит /slash-command)
│             └─ НЕТ ──→ Должен ли Claude автоматически обнаруживать и вызывать это?
│                        ├─ ДА ──→ SKILL (с description для обнаружения)
│                        └─ НЕТ ──→ Это переиспользуемые доменные знания?
│                                   ├─ ДА ──→ SKILL (с user-invocable: false только для предзагрузки)
│                                   └─ НЕТ ──→ COMMAND (простой шаблон промпта)
```

### Быстрые правила принятия решений

- **Нужна изоляция контекста?** → Agent
- **Нужна постоянная память?** → Agent
- **Нужно фоновое выполнение?** → Agent
- **Нужна изоляция worktree?** → Agent
- **Нужны ограничения инструментов?** → Agent
- **Рабочий процесс, запускаемый пользователем?** → Command
- **Переиспользуемая процедура или доменные знания?** → Skill
- **Автообнаружение Claude?** → Skill (или Agent с `PROACTIVELY` в description)
- **Предзагрузка в Agent?** → Skill

---

## Реальные сценарии (12)

### 1. Код-ревью

**Рекомендация**: Agent

**Почему**: Код-ревью требует изолированного контекста, чтобы результаты проверки не засоряли основной диалог. Agent может иметь собственные инструменты (только чтение), модель (haiku для скорости) и постоянную память для изучения специфичных для команды паттернов проверки.

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes for bugs, security, and style"
tools: Read, Glob, Grep, Bash
model: sonnet
memory: project
permissionMode: plan
---
```

### 2. Получение погоды (цепочка Command → Agent → Skill)

**Рекомендация**: Command, оркестрирующий Agent с предзагруженным Skill

**Почему**: Пользователь запускает явно (Command), логика получения данных работает в изоляции (Agent), а инструкции по получению данных — это переиспользуемые доменные знания (Skill, предзагруженный в Agent).

```
/weather-orchestrator  →  Agent(weather-agent)  →  Skill(weather-svg-creator)
     Command                   Agent                      Skill
```

### 3. Генерация шаблонов файлов

**Рекомендация**: Skill

**Почему**: Шаблоны — это переиспользуемые доменные знания. Claude может автоматически обнаружить Skill, когда пользователь просит создать новый компонент, маршрут или модуль. Изоляция не нужна — файлы записываются в текущий проект.

```yaml
# .claude/skills/react-scaffold/SKILL.md
---
name: react-scaffold
description: "Scaffold React components with TypeScript, tests, and Storybook stories"
paths: "src/components/**"
---
```

### 4. Оркестрация CI/CD

**Рекомендация**: Command

**Почему**: Рабочие процессы CI/CD всегда запускаются пользователем (`/deploy`, `/release`). Они следуют фиксированной последовательности и нуждаются в доступе к текущему контексту сессии (ветка, недавние изменения).

```yaml
# .claude/commands/deploy.md
---
name: deploy
description: "Build, test, and deploy to staging or production"
argument-hint: "[staging|production]"
---
```

### 5. Тестирование в браузере

**Рекомендация**: Agent

**Почему**: Тестирование в браузере — долгий процесс, который может выполняться в фоне и не должен блокировать основной диалог. Agent может иметь специфические MCP-серверы (Playwright, Chrome DevTools) и работать в worktree для изоляции.

```yaml
# .claude/agents/browser-tester.md
---
name: browser-tester
description: "Run end-to-end browser tests against the dev server"
tools: Bash, Read, WebFetch, mcp__playwright__*
background: true
model: sonnet
---
```

### 6. Показ времени

**Рекомендация**: Skill

**Почему**: Легковесный, изоляция не нужна, автообнаруживаемый. Claude вызывает Skill, когда кто-то спрашивает «который час?»

### 7. Многошаговое развёртывание

**Рекомендация**: Command, оркестрирующий Agent'ы

**Почему**: Command — это пользовательская точка входа (`/full-deploy`). Он порождает Agent'ы для каждого этапа — Agent сборки, Agent тестирования, Agent развёртывания — каждый работает в своём контексте с соответствующими инструментами.

### 8. Линтинг / Форматирование

**Рекомендация**: Skill (автовызов)

**Почему**: Правила линтинга — это переиспользуемые доменные знания. Skill автоматически активируется через `paths:`, когда Claude редактирует подходящие файлы. Взаимодействие с пользователем не требуется.

```yaml
# .claude/skills/python-linting/SKILL.md
---
name: python-linting
description: "Apply Black formatting and ruff linting to Python files"
paths: "**/*.py"
---
```

### 9. Исследовательский ассистент

**Рекомендация**: Agent

**Почему**: Исследовательские задачи долгие, выигрывают от памяти (запоминание прошлых исследований), требуют веб-инструментов (`WebFetch`, `WebSearch`) и должны выполняться в изолированном контексте, чтобы не засорять основной диалог.

### 10. Миграция данных

**Рекомендация**: Agent с изоляцией worktree

**Почему**: Миграции баз данных изменяют критически важные файлы и должны тестироваться изолированно. Работа в git worktree означает, что основная ветка не затрагивается, пока миграция не будет проверена.

```yaml
# .claude/agents/data-migrator.md
---
name: data-migrator
description: "Create and test database migration scripts"
isolation: worktree
model: sonnet
---
```

### 11. Генерация документации

**Рекомендация**: Skill

**Почему**: Генерация документации — это переиспользуемая процедура. Claude может автоматически обнаружить её, когда его просят «написать документацию» или «сгенерировать API-справочник». Она работает в текущем контексте и записывает файлы напрямую в проект.

### 12. Настройка проекта

**Рекомендация**: Command (с Setup Hook)

**Почему**: Инициализация проекта всегда запускается пользователем и выполняется один раз. Сочетайте Command `/setup` с Hook `Setup` для автозапуска при первом клонировании.

---

## Паттерны композиции

### Command → Agent → Skill

Каноничный паттерн оркестрации. Command — это точка входа для пользователя, Agent обеспечивает изоляцию и контроль инструментов, а Skill предоставляет доменные знания.

```
Пользователь вводит /command
  └─ Промпт Command выполняется в текущем контексте
       └─ Agent(...) порождается с изолированным контекстом
            └─ Предзагруженный Skill предоставляет инструкции
                 └─ Agent завершается, результат всплывает в контекст Command
                      └─ Инструмент Skill вызывается для финального вывода
```

### Agent с предзагруженными Skill'ами

Agent загружает содержимое Skill при запуске через поле `skills:`. Содержимое Skill внедряется в системный промпт Agent, давая ему доменную экспертизу без явного вызова пользователем.

```yaml
# .claude/agents/api-builder.md
---
name: api-builder
skills:
  - openapi-validator
  - rest-conventions
  - error-handling-patterns
---
```

### Skill с форком контекста

Skill, которому нужна изоляция, но не полноценное определение Agent. Установка `context: fork` запускает Skill в контексте субагента, сохраняя простой формат файла Skill.

```yaml
# .claude/skills/security-scan/SKILL.md
---
name: security-scan
description: "Scan the codebase for security vulnerabilities"
context: fork
agent: general-purpose
---
```

---

## Антипаттерны выбора механизма

| Антипаттерн | Проблема | Лучший подход |
|-------------|----------|--------------|
| Agent для простых задач | Избыточная изоляция контекста, когда нужно просто внедрить инструкции | Используйте Skill |
| Command для автообнаружения | Command'ы не могут быть автоматически вызваны Claude | Используйте Skill с `description` |
| Skill для долгих задач | Skill работает в основном контексте и блокирует диалог | Используйте Agent с `background: true` |
| Agent для общих соглашений | Контекст Agent изолирован — соглашения не переносятся в основную сессию | Используйте CLAUDE.md или Rules |
| Command для предзагружаемых знаний | Command'ы нельзя предзагрузить в Agent'ы | Используйте Skill с `user-invocable: false` |
| Skill без `description` | Claude не может автоматически обнаружить его | Всегда добавляйте осмысленное description |

---

## Источники

- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Slash Commands — Docs](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Common Workflows — Docs](https://code.claude.com/docs/en/common-workflows)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
