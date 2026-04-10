🌍 [English](../../../best-practice/claude-skills.md) | [中文](../../zh/best-practice/claude-skills.md) | [日本語](../../ja/best-practice/claude-skills.md) | [Français](../../fr/best-practice/claude-skills.md) | **Русский** | [العربية](../../ar/best-practice/claude-skills.md)

# Лучшие практики Skills

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-skills-implementation.md)

Skills Claude Code — поля frontmatter, официальные встроенные skills, таксономия типов, руководство по созданию, паттерны вызова и обнаружение в монорепозиториях.

<table width="100%">
<tr>
<td><a href="../README.md">← Назад к Claude Code Best Practice</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Поля Frontmatter (13)

| Поле | Тип | Обязательное | Описание |
|------|-----|-------------|----------|
| `name` | string | Нет | Отображаемое имя и идентификатор `/slash-command`. По умолчанию — имя директории |
| `description` | string | Рекомендуется | Что делает skill. Отображается в автодополнении и используется Claude для автообнаружения |
| `argument-hint` | string | Нет | Подсказка при автодополнении (напр. `[issue-number]`, `[filename]`) |
| `disable-model-invocation` | boolean | Нет | Установите `true`, чтобы запретить Claude автоматически вызывать этот skill |
| `user-invocable` | boolean | Нет | Установите `false`, чтобы скрыть из меню `/` — skill становится фоновыми знаниями, предназначенными для предзагрузки в агенты |
| `allowed-tools` | string | Нет | Инструменты, разрешённые без запросов разрешений при активном skill |
| `model` | string | Нет | Модель при запуске skill (напр. `haiku`, `sonnet`, `opus`) |
| `effort` | string | Нет | Переопределение уровня усилий при вызове (`low`, `medium`, `high`, `max`) |
| `context` | string | Нет | Установите `fork` для запуска skill в изолированном контексте subagent |
| `agent` | string | Нет | Тип subagent при установленном `context: fork` (по умолчанию: `general-purpose`) |
| `hooks` | object | Нет | Lifecycle hooks, привязанные к этому skill |
| `paths` | string/list | Нет | Glob-паттерны, ограничивающие автоактивацию skill. Принимает строку через запятую или YAML-список — Claude загружает skill только при работе с подходящими файлами |
| `shell` | string | Нет | Shell для блоков `` !`command` `` — `bash` (по умолчанию) или `powershell`. Требует `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Skill | Описание |
|---|-------|----------|
| 1 | `simplify` | Проверка изменённого кода на повторное использование, качество и эффективность — рефакторинг для устранения дублирования |
| 2 | `batch` | Запуск команд по нескольким файлам массово |
| 3 | `debug` | Отладка неработающих команд или проблем с кодом |
| 4 | `loop` | Запуск промпта или slash command по расписанию (до 3 дней) |
| 5 | `claude-api` | Создание приложений с Claude API или Anthropic SDK — срабатывает при импорте `anthropic` / `@anthropic-ai/sdk` |

См. также: [Официальный репозиторий Skills](https://github.com/anthropics/skills/tree/main/skills) для устанавливаемых skills от сообщества.

---

## Таксономия типов Skills (9)

На основе [классификации Тарика](https://www.thariq.io/blog/claude-code-best-practices/) — для каких задач skills подходят лучше всего. Каждый тип представляет отдельную категорию доменных знаний или автоматизации.

| # | Тип | Назначение | Пример |
|---|-----|-----------|--------|
| 1 | **Справочник API/библиотеки** | Доменные знания для API, фреймворков или SDK — предотвращает галлюцинации сигнатур методов | Skill со встроенной актуальной API-поверхностью Next.js App Router |
| 2 | **Проверка продукта** | Процедуры тестирования и валидации — определяет, как проверить работоспособность фичи | Skill, запускающий полную проверку потока через UI, API и слои данных |
| 3 | **Получение данных** | Извлечение внешних данных из API, парсинг источников или файловых систем | Skill, получающий данные о погоде из Open-Meteo API |
| 4 | **Бизнес-процесс** | Организационные рабочие процессы и процедуры — кодирование командных руководств | Skill, проводящий через чек-лист релиза перед деплоем |
| 5 | **Генерация кода** | Шаблоны проектов или файлов — стандартизация boilerplate | Skill, генерирующий маршрут Next.js с тестами, типами и валидацией |
| 6 | **Качество кода** | Линтинг, форматирование и стандарты ревью — обеспечение командных соглашений | Skill, запускающий ESLint + Prettier + пользовательские правила на изменённых файлах |
| 7 | **CI/CD** | Автоматизация сборки, деплоя и релизов — кодификация логики пайплайнов | Skill, запускающий staging-деплой и ожидающий health checks |
| 8 | **Документация** | Генерация документации, форматирование и стандарты — синхронизация документации с кодом | Skill, генерирующий API-справочник из спецификации OpenAPI |
| 9 | **Безопасность** | Сканирование уязвимостей, процедуры аудита и проверки соответствия | Skill, сканирующий зависимости на известные CVE перед мержем |

---

## Создание пользовательских Skills

### Шаг 1: Создайте директорию

```
.claude/skills/my-skill/
```

### Шаг 2: Создайте SKILL.md с frontmatter

```yaml
---
name: my-skill
description: What this skill does — used for auto-discovery and autocomplete
argument-hint: "[optional-argument]"
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# My Skill

## Task
Describe what the skill accomplishes.

## Instructions
1. First step
2. Second step
3. Third step

## Output
What the skill produces or where it writes results.
```

### Шаг 3: Добавьте вспомогательные файлы (необязательно)

```
.claude/skills/my-skill/
  SKILL.md           # Обязательно — frontmatter + инструкции
  reference.md       # Необязательно — API-поверхность, определения типов
  examples.md        # Необязательно — примеры использования, крайние случаи
```

Вспомогательные файлы ссылаются из тела SKILL.md через относительные ссылки. Claude читает их при вызове skill, давая доступ к детальным доменным знаниям без раздувания основного файла инструкций.

### Шаг 4: Тестируйте через slash command

```bash
$ claude
> /my-skill
```

### Шаг 5: Проверьте автообнаружение

Если у вашего skill есть поле `description` и `disable-model-invocation` не установлен в `true`, Claude автоматически обнаружит и вызовет skill, когда контекст разговора совпадёт. Протестируйте, описав задачу без явного вызова slash command — Claude должен подхватить skill самостоятельно.

---

## Паттерны Skills

Четыре паттерна вызова и загрузки skills:

| Паттерн | Ключевая конфигурация | Как это работает |
|---------|----------------------|-----------------|
| **User Skill** | `user-invocable: true` (по умолчанию) | Отображается в меню автодополнения `/`. Пользователь вызывает явно через `/skill-name` или Claude обнаруживает по `description` |
| **Agent Skill** | `user-invocable: false` | Скрыт из меню `/`. Предзагружается в subagent через поле `skills:` в frontmatter. Полное содержимое внедряется в контекст агента при запуске — действует как доменные знания, а не отдельная команда |
| **Auto-invoke Skill** | `description` активирует обнаружение | Claude читает `description` всех skills и автоматически вызывает skill при совпадении контекста разговора. Slash command не нужна. Отключение: `disable-model-invocation: true` |
| **Forked Skill** | `context: fork` | Запускается в изолированном контексте subagent. Skill получает свой собственный поток разговора и не засоряет основной разговор. Используйте `agent:` для указания типа subagent |

### Пример User Skill

```yaml
---
name: weather-svg-creator
description: Creates an SVG weather card for Dubai
---
```

Вызывается через `/weather-svg-creator` или автоматически обнаруживается, когда пользователь спрашивает о визуализации погоды.

### Пример Agent Skill

```yaml
---
name: weather-fetcher
description: Instructions for fetching weather data from Open-Meteo API
user-invocable: false
---
```

Предзагружается в weather-agent через:

```yaml
# В frontmatter .claude/agents/weather-agent.md
skills:
  - weather-fetcher
```

### Пример Forked Skill

```yaml
---
name: security-audit
description: Run a security audit on the current project
context: fork
agent: general-purpose
allowed-tools: Read, Bash, Grep, Glob
---
```

Запускается изолированно — результаты аудита возвращаются в основной разговор по завершении форкнутого контекста, но промежуточные шаги не отображаются в основном потоке.

---

## Обнаружение Skills в монорепозиториях

Claude обнаруживает skills, обходя дерево директорий от корня проекта. В монорепозиториях с вложенными проектами это означает, что skills контекстуальны по отношению к тому, где Claude работает.

| Правило | Поведение |
|---------|----------|
| **Обнаружение от корня** | Skills в `.claude/skills/` в корне проекта доступны всегда |
| **Вложенное обнаружение** | Когда Claude заходит в поддиректорию со своим `.claude/skills/`, эти skills обнаруживаются и становятся доступны |
| **Загрузка по description** | Для обнаружения загружается только поле `description` из frontmatter. Полное содержимое SKILL.md загружается при вызове — это делает обнаружение лёгким |
| **Приоритет: ближайший первый** | Если два skills имеют одинаковое имя, побеждает тот, что в ближайшей директории `.claude/skills/` |
| **Изоляция вложенных проектов** | Поддиректория со своим `.claude/` считается вложенным проектом. Её skills доступны при работе Claude в этом поддереве |

### Пример: структура монорепозитория

```
my-monorepo/
  .claude/skills/
    shared-lint/SKILL.md         # Доступен везде
  packages/
    frontend/
      .claude/skills/
        component-gen/SKILL.md   # Доступен при работе в packages/frontend/
    backend/
      .claude/skills/
        api-scaffold/SKILL.md    # Доступен при работе в packages/backend/
```

Когда Claude работает с файлом в `packages/frontend/`, он видит и `shared-lint` (из корня), и `component-gen` (из ближайшего `.claude/skills/`). Он не видит `api-scaffold`, пока не перейдёт в `packages/backend/`.

---

## Источники

- [Skills Claude Code — Документация](https://code.claude.com/docs/en/skills)
- [Обнаружение Skills в монорепозиториях](../../../reports/claude-skills-for-larger-mono-repos.md)
- [CHANGELOG Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Лучшие практики Claude Code от Тарика — Типы Skills](https://www.thariq.io/blog/claude-code-best-practices/)
- [Взаимосвязь Agent, Command, Skill](../../../reports/claude-agent-command-skill.md)
