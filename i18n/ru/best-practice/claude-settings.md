🌍 [English](../../../best-practice/claude-settings.md) | [中文](../../zh/best-practice/claude-settings.md) | [日本語](../../ja/best-practice/claude-settings.md) | [Français](../../fr/best-practice/claude-settings.md) | **Русский** | [العربية](../../ar/best-practice/claude-settings.md)

# Лучшие практики Settings

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%2010%3A16%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.claude/settings.json)

Полное руководство по всем доступным параметрам конфигурации в файлах `settings.json` Claude Code. Начиная с v2.1.96, Claude Code предоставляет **более 60 настроек** и **более 170 переменных окружения** (используйте поле `"env"` в `settings.json`, чтобы избежать скриптов-обёрток).

<table width="100%">
<tr>
<td><a href="../README.md">← Назад к лучшим практикам Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## Содержание

1. [Иерархия настроек](#иерархия-настроек)
2. [Базовая конфигурация](#базовая-конфигурация)
3. [Разрешения](#разрешения)
4. [Hooks](#hooks)
5. [Серверы MCP](#серверы-mcp)
6. [Sandbox](#sandbox)
7. [Плагины](#плагины)
8. [Конфигурация моделей](#конфигурация-моделей)
9. [Отображение и UX](#отображение-и-ux)
10. [AWS и облачные учётные данные](#aws-и-облачные-учётные-данные)
11. [Переменные окружения](#переменные-окружения-через-env)
12. [Полезные команды](#полезные-команды)

---

## Иерархия настроек

Настройки применяются в порядке приоритета (от высшего к низшему):

| Приоритет | Расположение | Область | Общий? | Назначение |
|-----------|-------------|---------|--------|------------|
| 1 | Managed settings | Организация | Да (развёрнуто IT) | Политики безопасности, которые нельзя переопределить |
| 2 | Аргументы командной строки | Сессия | Н/Д | Временные переопределения для одной сессии |
| 3 | `.claude/settings.local.json` | Проект | Нет (git-ignored) | Персональные настройки проекта |
| 4 | `.claude/settings.json` | Проект | Да (зафиксирован) | Общие настройки команды |
| 5 | `~/.claude/settings.json` | Пользователь | Н/Д | Глобальные персональные значения по умолчанию |

**Managed settings** устанавливаются организацией и не могут быть переопределены ни на одном другом уровне, включая аргументы командной строки. Методы доставки:
- **Server-managed** настройки (удалённая доставка)
- **Профили MDM** — macOS plist по адресу `com.anthropic.claudecode`
- **Политики реестра** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode` (администратор) и `HKCU\SOFTWARE\Policies\ClaudeCode` (уровень пользователя, наименьший приоритет политик)
- **Файл** — `managed-settings.json` и `managed-mcp.json` (macOS: `/Library/Application Support/ClaudeCode/`, Linux/WSL: `/etc/claude-code/`, Windows: `C:\Program Files\ClaudeCode\`)
- **Drop-in директория** — `managed-settings.d/` рядом с `managed-settings.json` для независимых фрагментов политик (v2.1.83). Следуя соглашению systemd, `managed-settings.json` объединяется первым как базовый, затем все файлы `*.json` в drop-in директории сортируются по алфавиту и объединяются поверх. Последующие файлы переопределяют предыдущие для скалярных значений; массивы конкатенируются и дедуплицируются; объекты объединяются глубоко. Скрытые файлы, начинающиеся с `.`, игнорируются. Используйте числовые префиксы для управления порядком объединения (например, `10-telemetry.json`, `20-security.json`)

В рамках managed-уровня приоритет: server-managed > MDM/политики уровня ОС > файловые (`managed-settings.d/*.json` + `managed-settings.json`) > реестр HKCU (только Windows). Используется только один managed-источник; источники не объединяются между уровнями. В рамках файлового уровня drop-in файлы и базовый файл объединяются вместе.

> **Примечание:** Начиная с v2.1.75, устаревший путь Windows `C:\ProgramData\ClaudeCode\managed-settings.json` удалён. Используйте `C:\Program Files\ClaudeCode\managed-settings.json` вместо него.

**Важно**:
- Правила `deny` имеют наивысший приоритет безопасности и не могут быть переопределены правилами allow/ask с более низким приоритетом.
- Managed settings могут блокировать или переопределять локальное поведение, даже если локальные файлы указывают другие значения.
- Настройки типа массив (например, `permissions.allow`) **конкатенируются и дедуплицируются** между областями — записи со всех уровней объединяются, а не заменяются.

---

## Базовая конфигурация

### Общие настройки

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `$schema` | string | - | URL JSON Schema для валидации и автодополнения в IDE (например, `"https://json.schemastore.org/claude-code-settings.json"`) |
| `model` | string | `"default"` | Переопределить модель по умолчанию. Принимает псевдонимы (`sonnet`, `opus`, `haiku`) или полные ID моделей |
| `agent` | string | - | Установить агента по умолчанию для основного разговора. Значение — имя агента из `.claude/agents/`. Также доступно через CLI-флаг `--agent` |
| `language` | string | `"english"` | Предпочтительный язык ответов Claude. Также устанавливает язык голосовой диктовки |
| `cleanupPeriodDays` | number | `30` | Сессии, неактивные дольше этого периода, удаляются при запуске (минимум 1). Также контролирует возрастной лимит для автоматического удаления осиротевших worktree субагентов при запуске. Значение `0` отклоняется с ошибкой валидации. Для отключения записи транскриптов в неинтерактивном режиме (`-p`) используйте `--no-session-persistence` или SDK-опцию `persistSession: false` |
| `autoUpdatesChannel` | string | `"latest"` | Канал обновлений: `"stable"` или `"latest"` |
| `alwaysThinkingEnabled` | boolean | `false` | Включить расширенное мышление по умолчанию для всех сессий |
| `skipWebFetchPreflight` | boolean | `false` | Пропустить проверку чёрного списка WebFetch перед получением URL *(в JSON-схеме, не на официальной странице настроек)* |
| `availableModels` | array | - | Ограничить модели, которые пользователи могут выбрать через `/model`, `--model`, инструмент Config или `ANTHROPIC_MODEL`. Не влияет на опцию Default. Пример: `["sonnet", "haiku"]` |
| `fastModePerSessionOptIn` | boolean | `false` | Требовать от пользователей включение fast mode в каждой сессии |
| `defaultShell` | string | `"bash"` | Shell по умолчанию для команд `!` в поле ввода. Принимает `"bash"` (по умолчанию) или `"powershell"`. Установка `"powershell"` направляет интерактивные команды `!` через PowerShell на Windows. Требует `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` (v2.1.84) |
| `includeGitInstructions` | boolean | `true` | Включить встроенные инструкции по workflow коммитов и PR, а также снимок git status в системный промпт Claude. Переменная окружения `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` имеет приоритет над этой настройкой |
| `voiceEnabled` | boolean | - | Включить голосовую диктовку push-to-talk. Записывается автоматически при выполнении `/voice`. Требует аккаунт Claude.ai |
| `showClearContextOnPlanAccept` | boolean | `false` | Показывать опцию "clear context" на экране принятия плана. Установите `true` для восстановления опции (скрыта по умолчанию с v2.1.81) |
| `disableDeepLinkRegistration` | string | - | Установите `"disable"` чтобы запретить Claude Code регистрировать обработчик протокола `claude-cli://` в операционной системе при запуске. Deep links позволяют внешним инструментам открывать сессию Claude Code с предзаполненным промптом через `claude-cli://open?q=...`. Параметр `q` поддерживает многострочные промпты с URL-кодированными переносами строк (`%0A`). Полезно в средах, где регистрация обработчика протокола ограничена или управляется отдельно |
| `showThinkingSummaries` | boolean | `false` | Показывать сводки расширенного мышления в интерактивных сессиях. Когда не установлено или `false` (по умолчанию в интерактивном режиме), блоки мышления скрываются API и отображаются как свёрнутая заглушка. Скрытие меняет только то, что вы видите, а не то, что генерирует модель — для снижения затрат на мышление уменьшите бюджет или отключите мышление. Неинтерактивный режим (`-p`) и SDK-вызовы всегда получают сводки независимо от этой настройки |
| `disableSkillShellExecution` | boolean | `false` | Отключить inline-выполнение shell для блоков `` !`...` `` в skills и пользовательских командах. Команды заменяются на `[shell command execution disabled by policy]`. Bundled и managed skills не затрагиваются (v2.1.91) |
| `forceRemoteSettingsRefresh` | boolean | `false` | **(Только managed)** Блокировать запуск CLI до получения свежих удалённых managed settings. При неудаче получения CLI завершается (fail-closed). Используйте в корпоративных средах, где применение политик должно быть актуальным перед началом любой сессии (v2.1.92) |
| `feedbackSurveyRate` | number | - | Вероятность (0–1) появления опроса о качестве сессии при соответствии условиям. Корпоративные администраторы могут контролировать частоту показа. Пример: `0.05` = 5% подходящих сессий |

**Пример:**
```json
{
  "model": "opus",
  "agent": "code-reviewer",
  "language": "japanese",
  "cleanupPeriodDays": 60,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true
}
```

### Директории планов и памяти

Хранение файлов планов и автоматической памяти в пользовательских местах.

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `plansDirectory` | string | `~/.claude/plans` | Директория для хранения выходных данных `/plan` |
| `autoMemoryDirectory` | string | - | Пользовательская директория для хранения автоматической памяти. Принимает пути с расширением `~/`. Не принимается в настройках проекта (`.claude/settings.json`) для предотвращения перенаправления записей памяти в чувствительные расположения; принимается из настроек политик, локальных и пользовательских |

**Пример:**
```json
{
  "plansDirectory": "./my-plans"
}
```

**Случай использования:** Полезно для организации артефактов планирования отдельно от внутренних файлов Claude или для хранения планов в общем командном расположении.

### Настройки Worktree

Настройка создания и управления worktree через `--worktree`. Полезно для уменьшения использования диска и времени запуска в больших монорепозиториях.

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `worktree.symlinkDirectories` | array | `[]` | Директории для символической ссылки из основного репозитория в каждый worktree, чтобы избежать дублирования больших директорий на диске |
| `worktree.sparsePaths` | array | `[]` | Директории для извлечения в каждый worktree через git sparse-checkout (режим cone). Только указанные пути записываются на диск |

**Пример:**
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

### Настройки атрибуции

Настройка сообщений атрибуции для git-коммитов и pull request.

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `attribution.commit` | string | Co-authored-by | Атрибуция git-коммита (поддерживает trailers) |
| `attribution.pr` | string | Сгенерированное сообщение | Атрибуция описания pull request |
| `includeCoAuthoredBy` | boolean | `true` | **УСТАРЕЛО** — Используйте `attribution` вместо этого |

**Пример:**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**Примечание:** Установите пустую строку (`""`) для полного скрытия атрибуции.

### Помощники аутентификации

Скрипты для динамической генерации токенов аутентификации.

| Ключ | Тип | Описание |
|------|-----|----------|
| `apiKeyHelper` | string | Путь к shell-скрипту, который выводит токен аутентификации (отправляется как заголовок `X-Api-Key`) |
| `forceLoginMethod` | string | Ограничить вход аккаунтами `"claudeai"` или `"console"` |
| `forceLoginOrgUUID` | string \| array | Требовать принадлежность входа к определённой организации. Принимает одну строку UUID (которая также предварительно выбирает эту организацию при входе) или массив UUID, где любая указанная организация принимается без предварительного выбора. При установке в managed settings вход не удаётся, если аутентифицированный аккаунт не принадлежит указанной организации; пустой массив закрывается с ошибкой и блокирует вход с сообщением о неправильной конфигурации |

**Пример:**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

### Корпоративные объявления

Отображение пользовательских объявлений при запуске (случайная ротация).

| Ключ | Тип | Описание |
|------|-----|----------|
| `companyAnnouncements` | array | Массив строк, отображаемых при запуске |

**Пример:**
```json
{
  "companyAnnouncements": [
    "Welcome to Acme Corp!",
    "Remember to run tests before committing!",
    "Check the wiki for coding standards"
  ]
}
```

---

## Разрешения

Управление инструментами и операциями, которые может выполнять Claude.

### Структура разрешений

```json
{
  "permissions": {
    "allow": [],
    "ask": [],
    "deny": [],
    "additionalDirectories": [],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable"
  }
}
```

### Ключи разрешений

| Ключ | Тип | Описание |
|------|-----|----------|
| `permissions.allow` | array | Правила, разрешающие использование инструментов без запроса |
| `permissions.ask` | array | Правила, требующие подтверждения пользователя |
| `permissions.deny` | array | Правила, блокирующие использование инструментов (наивысший приоритет) |
| `permissions.additionalDirectories` | array | Дополнительные директории, к которым Claude может получить доступ |
| `permissions.defaultMode` | string | Режим разрешений по умолчанию. В удалённых средах учитываются только `acceptEdits` и `plan` (v2.1.70+) |
| `permissions.disableBypassPermissionsMode` | string | Предотвратить активацию режима bypass |
| `permissions.skipDangerousModePermissionPrompt` | boolean | Пропустить запрос подтверждения, отображаемый перед входом в режим bypass permissions через `--dangerously-skip-permissions` или `defaultMode: "bypassPermissions"`. Игнорируется при установке в настройках проекта (`.claude/settings.json`), чтобы ненадёжные репозитории не могли автоматически обходить запрос |
| `allowManagedPermissionRulesOnly` | boolean | **(Только managed)** Применяются только managed-правила разрешений; пользовательские/проектные правила `allow`, `ask`, `deny` игнорируются |
| `allow_remote_sessions` | boolean | **(Только managed)** Разрешить пользователям запускать сессии Remote Control и web. По умолчанию `true`. Установите `false` для запрета доступа к удалённым сессиям *(не в официальной документации — официальная страница разрешений гласит "Access to Remote Control and web sessions is not controlled by a managed settings key." На планах Team и Enterprise администраторы включают/отключают через [настройки администрирования Claude Code](https://claude.ai/admin-settings/claude-code))* |
| `autoMode` | object | Настроить, что классификатор [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode) блокирует и разрешает. Содержит `environment` (описания доверенной инфраструктуры), `allow` (исключения из правил блокировки) и `soft_deny` (правила блокировки) — всё массивы прозаических строк. **Не читается из общих настроек проекта** (`.claude/settings.json`) для предотвращения инъекции через репозиторий. Доступно в пользовательских, локальных и managed настройках. Установка `allow` или `soft_deny` **заменяет** весь список по умолчанию для этого раздела. Выполните `claude auto-mode defaults` для просмотра встроенных правил перед настройкой |
| `disableAutoMode` | string | Установите `"disable"` для предотвращения активации [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode). Удаляет `auto` из цикла `Shift+Tab` и отклоняет `--permission-mode auto` при запуске. Может быть установлено на любом уровне настроек; наиболее полезно в managed settings, где пользователи не могут переопределить |
| `useAutoModeDuringPlan` | boolean | Использует ли режим plan семантику auto mode, когда auto mode доступен. По умолчанию: `true`. Не читается из общих настроек проекта (`.claude/settings.json`). Отображается в `/config` как "Use auto mode during plan" |

### Режимы разрешений

| Режим | Поведение |
|-------|----------|
| `"default"` | Стандартная проверка разрешений с запросами |
| `"acceptEdits"` | Автоматическое принятие правок файлов без запроса |
| `"askEdits"` | Запрашивать перед каждой операцией *(не в официальной документации — не проверено)* |
| `"dontAsk"` | Автоматически отклоняет инструменты, если они не предварительно одобрены через `/permissions` или правила `permissions.allow` |
| `"viewOnly"` | Режим только чтение, без модификаций *(не в официальной документации — не проверено)* |
| `"bypassPermissions"` | Пропуск всех проверок разрешений (опасно) |
| `"auto"` | Фоновый классификатор заменяет ручные запросы (`--enable-auto-mode`). Исследовательский предпросмотр — требует план Team + Sonnet/Opus 4.6. Классификатор автоматически одобряет операции только чтения и правки файлов; всё остальное отправляет через проверку безопасности. Возвращается к запросам после 3 последовательных или 20 суммарных блокировок. Настраивается через параметр `autoMode` |
| `"plan"` | Режим исследования только для чтения |

### Синтаксис разрешений инструментов

| Инструмент | Синтаксис | Примеры |
|-----------|----------|---------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`, `Bash(* install)`, `Bash(git * main)` |
| `Read` | `Read(path pattern)` | `Read(.env)`, `Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`, `Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`, `Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | Глобальный веб-поиск |
| `Task` | `Task(agent-name)` | `Task(Explore)`, `Task(my-agent)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`, `Agent(*)` — разрешение ограничено запуском субагентов |
| `Skill` | `Skill(skill-name)` | `Skill(weather-fetcher)` |
| `MCP` | `mcp__server__tool` или `MCP(server:tool)` | `mcp__memory__*`, `MCP(github:*)` |

**Порядок оценки:** Правила оцениваются по порядку: сначала правила deny, затем ask, затем allow. Побеждает первое совпавшее правило.

**Паттерны путей Read/Edit:** Правила разрешений для `Read`, `Edit` и `Write` поддерживают паттерны в стиле gitignore с четырьмя типами префиксов:

| Префикс | Значение | Пример |
|---------|----------|--------|
| `//` | Абсолютный путь от корня файловой системы | `Read(//Users/alice/file)` |
| `~/` | Относительно домашней директории | `Read(~/.zshrc)` |
| `/` | Относительно корня проекта | `Edit(/src/**)` |
| `./` или нет | Относительный путь (текущая директория) | `Read(.env)`, `Read(*.ts)` |

**Примечания по wildcard в Bash:**
- `*` может появляться в **любой позиции**: префикс (`Bash(* install)`), суффикс (`Bash(npm *)`), или середина (`Bash(git * main)`)
- **Граница слова:** `Bash(ls *)` (пробел перед `*`) соответствует `ls -la`, но НЕ `lsof`; `Bash(ls*)` (без пробела) соответствует обоим
- `Bash(*)` считается эквивалентным `Bash` (соответствует всем bash-командам)
- Правила разрешений поддерживают перенаправления вывода: `Bash(python:*)` соответствует `python script.py > output.txt`
- Устаревший суффиксный синтаксис `:*` (например, `Bash(npm:*)`) эквивалентен ` *`, но является устаревшим

**Пример:**
```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*"
    ],
    "ask": [
      "Bash(rm *)",
      "Bash(git push *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)",
      "Bash(curl *)"
    ],
    "additionalDirectories": ["../shared-libs/"]
  }
}
```

---

## Hooks

Конфигурация hooks (события, свойства, сопоставители, коды выхода, переменные окружения и HTTP hooks) поддерживается в отдельном репозитории:

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — Полный справочник по hooks с системой звуковых уведомлений, всеми 25 событиями hooks, HTTP hooks, паттернами сопоставителей, кодами выхода и переменными окружения.

Ключи настроек, связанные с hooks (`hooks`, `disableAllHooks`, `allowManagedHooksOnly`, `allowedHttpHookUrls`, `httpHookAllowedEnvVars`), документированы там.

Для официального справочника по hooks см. [Документацию Claude Code Hooks](https://code.claude.com/docs/en/hooks).

---

## Серверы MCP

Настройка серверов Model Context Protocol для расширенных возможностей.

### Настройки MCP

| Ключ | Тип | Область | Описание |
|------|-----|---------|----------|
| `enableAllProjectMcpServers` | boolean | Любая | Автоматически одобрять все серверы `.mcp.json` |
| `enabledMcpjsonServers` | array | Любая | Белый список определённых имён серверов |
| `disabledMcpjsonServers` | array | Любая | Чёрный список определённых имён серверов |
| `allowedMcpServers` | array | Только managed | Белый список с сопоставлением имени/команды/URL |
| `deniedMcpServers` | array | Только managed | Чёрный список с сопоставлением |
| `allowManagedMcpServersOnly` | boolean | Только managed | Разрешить только серверы MCP, явно указанные в managed белом списке |
| `channelsEnabled` | boolean | Только managed | Разрешить [channels](https://code.claude.com/docs/en/channels) для пользователей Team и Enterprise. Когда не установлено или `false`, доставка сообщений каналов блокируется независимо от флага `--channels` |
| `allowedChannelPlugins` | array | Только managed | Белый список плагинов каналов, которые могут отправлять сообщения. Заменяет белый список Anthropic по умолчанию при установке. Не определено = использовать список по умолчанию, пустой массив = блокировать все плагины каналов. Требует `channelsEnabled: true`. Каждая запись — объект с полями `marketplace` и `plugin` (v2.1.84) |

### Сопоставление серверов MCP (Managed Settings)

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": "npx @modelcontextprotocol/*" },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" }
  ]
}
```

**Пример:**
```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["memory", "github", "filesystem"],
  "disabledMcpjsonServers": ["experimental-server"]
}
```

---

## Sandbox

Настройка песочницы для bash-команд для обеспечения безопасности.

### Настройки Sandbox

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `sandbox.enabled` | boolean | `false` | Включить sandbox для bash |
| `sandbox.failIfUnavailable` | boolean | `false` | Завершиться с ошибкой, когда sandbox включён, но не может запуститься, вместо запуска без sandbox. Полезно для корпоративных политик, требующих строгой песочницы (v2.1.83) |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | Автоматически одобрять bash при работе в sandbox |
| `sandbox.excludedCommands` | array | `[]` | Команды для выполнения за пределами sandbox |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | Разрешить `dangerouslyDisableSandbox`. При установке `false` аварийный выход полностью отключается и все команды должны выполняться в sandbox (или быть в `excludedCommands`). Полезно для корпоративных политик, требующих строгой песочницы |
| `sandbox.ignoreViolations` | object | `{}` | Сопоставление паттернов команд с массивами путей — подавляет предупреждения о нарушениях *(в JSON-схеме, не на официальной странице настроек)* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | Ослабленный sandbox для Docker (снижает безопасность) |
| `sandbox.network.allowUnixSockets` | array | `[]` | Конкретные пути Unix-сокетов, доступные в sandbox |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | Разрешить все Unix-сокеты (переопределяет allowUnixSockets) |
| `sandbox.network.allowLocalBinding` | boolean | `false` | Разрешить привязку к портам localhost (macOS) |
| `sandbox.network.allowedDomains` | array | `[]` | Белый список сетевых доменов для sandbox |
| `sandbox.network.deniedDomains` | array | `[]` | Чёрный список сетевых доменов для sandbox *(не в официальной документации — не проверено)* |
| `sandbox.network.httpProxyPort` | number | - | Порт HTTP-прокси 1-65535 (пользовательский прокси) |
| `sandbox.network.socksProxyPort` | number | - | Порт SOCKS5-прокси 1-65535 (пользовательский прокси) |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | Разрешить только домены из managed белого списка (managed settings) |
| `sandbox.filesystem.allowWrite` | array | `[]` | Дополнительные пути, куда sandbox-команды могут писать. Массивы объединяются между всеми областями настроек. Также объединяются с путями из правил разрешений `Edit(...)` allow. Префикс: `/` (абсолютный), `~/` (домашняя директория), `./` или нет (относительно проекта в настройках проекта, относительно `~/.claude` в пользовательских настройках). Старый префикс `//` для абсолютных путей всё ещё работает. **Примечание:** Это отличается от [правил разрешений Read/Edit](#синтаксис-разрешений-инструментов), которые используют `//` для абсолютного и `/` для относительного к проекту |
| `sandbox.filesystem.denyWrite` | array | `[]` | Пути, куда sandbox-команды не могут писать. Массивы объединяются между всеми областями настроек. Также объединяются с путями из правил разрешений `Edit(...)` deny. Те же соглашения по префиксам путей, что и `allowWrite` |
| `sandbox.filesystem.denyRead` | array | `[]` | Пути, откуда sandbox-команды не могут читать. Массивы объединяются между всеми областями настроек. Также объединяются с путями из правил разрешений `Read(...)` deny. Те же соглашения по префиксам путей, что и `allowWrite` |
| `sandbox.filesystem.allowRead` | array | `[]` | Пути для повторного разрешения доступа на чтение в областях `denyRead`. Имеет приоритет над `denyRead`. Массивы объединяются между всеми областями настроек. Те же соглашения по префиксам путей, что и `allowWrite` |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **(Только managed)** Учитываются только пути `allowRead` из managed settings. Записи `allowRead` из пользовательских, проектных и локальных настроек игнорируются |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | (только macOS) Разрешить доступ к системному TLS trust (`com.apple.trustd.agent`); снижает безопасность |

**Пример:**
```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["git", "docker", "gh"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true
    }
  }
}
```

---

## Плагины

Настройка плагинов и маркетплейсов Claude Code.

### Настройки плагинов

| Ключ | Тип | Область | Описание |
|------|-----|---------|----------|
| `enabledPlugins` | object | Любая | Включить/отключить определённые плагины |
| `extraKnownMarketplaces` | object | Проект | Добавить пользовательские маркетплейсы плагинов (общий доступ через `.claude/settings.json`) |
| `strictKnownMarketplaces` | array | Только managed | Белый список разрешённых маркетплейсов |
| `skippedMarketplaces` | array | Любая | Маркетплейсы, от установки которых пользователь отказался *(в JSON-схеме, не на официальной странице настроек)* |
| `skippedPlugins` | array | Любая | Плагины, от установки которых пользователь отказался *(в JSON-схеме, не на официальной странице настроек)* |
| `pluginConfigs` | object | Любая | Конфигурации серверов MCP для каждого плагина (ключ по `plugin@marketplace`) *(в JSON-схеме, не на официальной странице настроек)* |
| `blockedMarketplaces` | array | Только managed | Заблокировать определённые маркетплейсы плагинов |
| `pluginTrustMessage` | string | Только managed | Пользовательское сообщение, отображаемое при запросе доверия к плагинам |

**Типы источников маркетплейса:** `github`, `git`, `directory`, `hostPattern`, `settings`, `url`, `npm`, `file`. Используйте `source: 'settings'` для объявления небольшого набора плагинов inline без настройки размещённого репозитория маркетплейса.

**Пример:**
```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "experimental@acme-tools": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "inline-tools": {
      "source": {
        "source": "settings",
        "name": "inline-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": { "source": "github", "repo": "acme-corp/code-formatter" }
          }
        ]
      }
    }
  }
}
```

---

## Конфигурация моделей

### Псевдонимы моделей

| Псевдоним | Описание |
|----------|----------|
| `"default"` | Рекомендуемая для вашего типа аккаунта |
| `"sonnet"` | Последняя модель Sonnet (Claude Sonnet 4.6) |
| `"opus"` | Последняя модель Opus (Claude Opus 4.6) |
| `"haiku"` | Быстрая модель Haiku |
| `"sonnet[1m]"` | Sonnet с контекстом 1M токенов |
| `"opus[1m]"` | Opus с контекстом 1M токенов (по умолчанию на Max, Team и Enterprise с v2.1.75) |
| `"opusplan"` | Opus для планирования, Sonnet для выполнения |

**Пример:**
```json
{
  "model": "opus"
}
```

### Переопределения моделей

Сопоставление ID моделей Anthropic с ID моделей провайдеров для развёртываний Bedrock, Vertex или Foundry.

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `effortLevel` | string | - | Сохранять уровень усилий между сессиями. Принимает `"low"`, `"medium"` или `"high"`. Записывается автоматически при выполнении `/effort low`, `/effort medium` или `/effort high`. Поддерживается на Opus 4.6 и Sonnet 4.6 |
| `modelOverrides` | object | - | Сопоставить записи средства выбора моделей с ID моделей провайдеров (например, ARN профилей вывода Bedrock). Каждый ключ — имя записи средства выбора, каждое значение — ID модели провайдера |

**Пример:**
```json
{
  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-sonnet-4-6-v1:0"
  }
}
```

### Уровень усилий

Команда `/model` предоставляет контроль **уровня усилий**, который настраивает глубину рассуждения модели за ответ. Используйте клавиши со стрелками ← → в UI `/model` для переключения между уровнями усилий.

| Уровень усилий | Описание |
|---------------|----------|
| High (по умолчанию) | Полная глубина рассуждения, лучше для сложных задач |
| Medium | Сбалансированное рассуждение, подходит для повседневных задач |
| Low | Минимальное рассуждение, самые быстрые ответы |

**Как использовать:**
1. Выполните `/effort low`, `/effort medium` или `/effort high` для прямой установки (v2.1.76+)
2. Или выполните `/model` → выберите модель → используйте клавиши **← →** для настройки
3. Настройка сохраняется через ключ `effortLevel` в `settings.json`

**Примечание:** Уровень усилий доступен для Opus 4.6 и Sonnet 4.6 на планах Max и Team. Значение по умолчанию было изменено с High на Medium в v2.1.68, затем возвращено на **High** для пользователей API-key, Bedrock/Vertex/Foundry, Team и Enterprise в v2.1.94. Начиная с v2.1.75, окно контекста 1M для Opus 4.6 доступно по умолчанию на планах Max, Team и Enterprise.

### Переменные окружения для моделей

Настройка через ключ `env`:

```json
{
  "env": {
    "ANTHROPIC_MODEL": "sonnet",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "custom-haiku-model",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "custom-sonnet-model",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "custom-opus-model",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku",
    "MAX_THINKING_TOKENS": "10000"
  }
}
```

---

## Отображение и UX

### Настройки отображения

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `statusLine` | object | - | Пользовательская конфигурация строки состояния |
| `outputStyle` | string | `"default"` | Стиль вывода (например, `"Explanatory"`) |
| `spinnerTipsEnabled` | boolean | `true` | Показывать подсказки во время ожидания |
| `spinnerVerbs` | object | - | Пользовательские глаголы спиннера с `mode` ("append" или "replace") и массивом `verbs` |
| `spinnerTipsOverride` | object | - | Пользовательские подсказки спиннера с `tips` (массив строк) и необязательным `excludeDefault` (boolean) |
| `respectGitignore` | boolean | `true` | Учитывать .gitignore в средстве выбора файлов |
| `prefersReducedMotion` | boolean | `false` | Уменьшить анимации и эффекты движения в UI |
| `fileSuggestion` | object | - | Пользовательская команда предложения файлов (см. Конфигурация предложений файлов ниже) |

### Настройки глобальной конфигурации (`~/.claude.json`)

Эти настройки отображения хранятся в `~/.claude.json`, **не** в `settings.json`. Их добавление в `settings.json` вызовет ошибку валидации схемы.

| Ключ | Тип | По умолчанию | Описание |
|------|-----|-------------|----------|
| `autoConnectIde` | boolean | `false` | Автоматически подключаться к работающей IDE при запуске Claude Code из внешнего терминала. Отображается в `/config` как **Auto-connect to IDE (external terminal)** при запуске вне терминала VS Code или JetBrains |
| `autoInstallIdeExtension` | boolean | `true` | Автоматически устанавливать расширение IDE Claude Code при запуске из терминала VS Code. Отображается в `/config` как **Auto-install IDE extension**. Также может быть отключено через переменную `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` |
| `editorMode` | string | `"normal"` | Режим клавиатурных привязок для поля ввода: `"normal"` или `"vim"`. Отображается в `/config` как **Editor mode** |
| `showTurnDuration` | boolean | `true` | Показывать сообщения о длительности хода после ответов (например, "Cooked for 1m 6s"). Измените `~/.claude.json` напрямую для изменения |
| `terminalProgressBarEnabled` | boolean | `true` | Показывать прогресс-бар терминала в поддерживаемых терминалах (ConEmu, Ghostty 1.2.0+ и iTerm2 3.6.6+). Отображается в `/config` как **Terminal progress bar** |
| `teammateMode` | string | `"in-process"` | Как отображаются коллеги [agent team](https://code.claude.com/docs/en/agent-teams): `"auto"` (выбирает разделённые панели в tmux или iTerm2, in-process в противном случае), `"in-process"` или `"tmux"`. См. [выбор режима отображения](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) |

### Конфигурация строки состояния

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

**Входные поля строки состояния:**

Команда строки состояния получает JSON-объект на stdin с этими примечательными полями:

| Поле | Описание |
|------|----------|
| `workspace.added_dirs` | Директории, добавленные через `/add-dir` |
| `context_window.used_percentage` | Процент использования окна контекста |
| `context_window.remaining_percentage` | Процент оставшегося окна контекста |
| `current_usage` | Текущий счётчик токенов окна контекста |
| `exceeds_200k_tokens` | Превышает ли контекст 200k токенов |
| `rate_limits.five_hour.used_percentage` | Процент использования лимита за 5 часов (v2.1.80+) |
| `rate_limits.five_hour.resets_at` | Временная метка сброса лимита за 5 часов |
| `rate_limits.seven_day.used_percentage` | Процент использования лимита за 7 дней |
| `rate_limits.seven_day.resets_at` | Временная метка сброса лимита за 7 дней |

### Конфигурация предложений файлов

Скрипт предложений файлов получает JSON-объект на stdin (например, `{"query": "src/comp"}`) и должен выводить до 15 путей файлов (по одному на строку).

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  },
  "respectGitignore": true
}
```

**Пример:**
```json
{
  "statusLine": {
    "type": "command",
    "command": "git branch --show-current 2>/dev/null || echo 'no-branch'"
  },
  "spinnerTipsEnabled": true,
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": ["Cooking", "Brewing", "Crafting", "Conjuring"]
  },
  "spinnerTipsOverride": {
    "tips": ["Use /compact at ~50% context", "Start with plan mode for complex tasks"],
    "excludeDefault": true
  }
}
```

---

## AWS и облачные учётные данные

### Настройки AWS

| Ключ | Тип | Описание |
|------|-----|----------|
| `awsAuthRefresh` | string | Скрипт для обновления аутентификации AWS (модифицирует директорию `.aws`) |
| `awsCredentialExport` | string | Скрипт, выводящий JSON с учётными данными AWS |

**Пример:**
```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

### OpenTelemetry

| Ключ | Тип | Описание |
|------|-----|----------|
| `otelHeadersHelper` | string | Скрипт для генерации динамических заголовков OpenTelemetry |

**Пример:**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## Переменные окружения (через `env`)

Установка переменных окружения для всех сессий Claude Code.

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "...",
    "NODE_ENV": "development",
    "DEBUG": "true"
  }
}
```

### Основные переменные окружения

| Переменная | Описание |
|-----------|----------|
| `ANTHROPIC_API_KEY` | API-ключ для аутентификации |
| `ANTHROPIC_AUTH_TOKEN` | OAuth-токен |
| `CLAUDE_CODE_OAUTH_TOKEN` | OAuth-токен доступа для аутентификации Claude.ai. Альтернатива `/login` для SDK и автоматизированных сред. Имеет приоритет над учётными данными из keychain |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | OAuth-токен обновления для аутентификации Claude.ai. При установке `claude auth login` обменивает этот токен напрямую вместо открытия браузера. Требует `CLAUDE_CODE_OAUTH_SCOPES` |
| `CLAUDE_CODE_OAUTH_SCOPES` | OAuth-области через пробел, с которыми был выпущен токен обновления (например, `"user:profile user:inference user:sessions:claude_code"`). Обязательно при установке `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` |
| `ANTHROPIC_BASE_URL` | Пользовательская конечная точка API |
| `ANTHROPIC_BEDROCK_BASE_URL` | Переопределение URL конечной точки Bedrock |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | Переопределение URL конечной точки Bedrock Mantle. См. [конечная точка Mantle](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) |
| `ANTHROPIC_VERTEX_BASE_URL` | Переопределение URL конечной точки Vertex AI |
| `ANTHROPIC_BETAS` | Значения beta-заголовка Anthropic через запятую |
| `ANTHROPIC_VERTEX_PROJECT_ID` | ID проекта GCP для Vertex AI |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | ID модели для добавления как пользовательской записи в средство выбора `/model`. Используйте для доступности нестандартной или gateway-специфичной модели без замены встроенных псевдонимов |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | Отображаемое имя для пользовательской записи модели в средстве выбора `/model`. По умолчанию — ID модели |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | Описание для пользовательской записи модели в средстве выбора `/model`. По умолчанию `Custom model (<model-id>)` |
| `ANTHROPIC_MODEL` | Имя используемой модели. Принимает псевдонимы (`sonnet`, `opus`, `haiku`) или полные ID моделей. Переопределяет настройку `model` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Переопределить псевдоним модели Haiku пользовательским ID модели (например, для сторонних развёртываний) |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | Настроить метку записи Haiku в средстве выбора `/model` при использовании фиксированной модели на Bedrock/Vertex/Foundry. По умолчанию — ID модели |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | Настроить описание записи Haiku в средстве выбора `/model`. По умолчанию `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | Переопределить обнаружение возможностей для фиксированной модели Haiku. Значения через запятую (например, `effort,thinking`). Обязательно, когда фиксированная модель поддерживает функции, которые автообнаружение не может подтвердить |
| `CLAUDECODE` | Устанавливается в `1` в shell-средах, которые создаёт Claude Code (инструмент Bash, сессии tmux). Не устанавливается в hooks или командах строки состояния. Используйте для определения, выполняется ли скрипт внутри shell Claude Code |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | Установите `1` для разрешения fast mode при неудаче проверки статуса организации из-за сетевой ошибки. Полезно, когда корпоративный прокси блокирует конечную точку статуса |
| `CLAUDE_CODE_USE_BEDROCK` | Использовать AWS Bedrock (`1` для включения) |
| `CLAUDE_CODE_USE_VERTEX` | Использовать Google Vertex AI (`1` для включения) |
| `CLAUDE_CODE_USE_FOUNDRY` | Использовать Microsoft Foundry (`1` для включения) |
| `CLAUDE_CODE_USE_MANTLE` | Использовать [конечную точку Mantle](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) Bedrock (`1` для включения) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Установите `1` для включения инструмента PowerShell на Windows (opt-in предпросмотр). При включении Claude может выполнять команды PowerShell нативно вместо маршрутизации через Git Bash. Поддерживается только на нативном Windows, не WSL (v2.1.84) |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | Префикс для автоматически генерируемых имён сессий Remote Control. По умолчанию — имя хоста машины |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | Включить/отключить телеметрию (`0` или `1`) |
| `DISABLE_ERROR_REPORTING` | Отключить отчёты об ошибках (`1` для отключения) |
| `DISABLE_TELEMETRY` | Отключить телеметрию (`1` для отключения) |
| `MCP_TIMEOUT` | Таймаут запуска MCP в мс |
| `MAX_MCP_OUTPUT_TOKENS` | Макс. токены вывода MCP (по умолчанию: 25000). Предупреждение отображается при превышении 10 000 токенов |
| `API_TIMEOUT_MS` | Таймаут в мс для API-запросов (по умолчанию: 600000) |
| `BASH_MAX_TIMEOUT_MS` | Таймаут bash-команд |
| `BASH_MAX_OUTPUT_LENGTH` | Макс. длина вывода bash |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Порог авто-компактирования в процентах (1-100). По умолчанию ~95%. Установите ниже (например, `50`) для более раннего запуска компактирования. Значения выше 95% не действуют. Используйте `/context` для мониторинга текущего использования. Пример: `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Сохранять cwd между вызовами bash (`1` для включения) |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Отключить фоновые задачи (`1` для отключения) |
| `ENABLE_TOOL_SEARCH` | Порог поиска инструментов MCP (например, `auto:5`) |
| `DISABLE_PROMPT_CACHING` | Отключить всё кэширование промптов (`1` для отключения) |
| `DISABLE_PROMPT_CACHING_HAIKU` | Отключить кэширование промптов Haiku |
| `DISABLE_PROMPT_CACHING_SONNET` | Отключить кэширование промптов Sonnet |
| `DISABLE_PROMPT_CACHING_OPUS` | Отключить кэширование промптов Opus |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | Запросить TTL кэша 1 час на Bedrock (`1` для включения) |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | Отключить экспериментальные beta-функции (`1` для отключения) |
| `CLAUDE_CODE_SHELL` | Переопределить автоматическое определение shell |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | Переопределить лимит токенов по умолчанию для чтения файлов |
| `CLAUDE_CODE_GLOB_HIDDEN` | Установите `false` для исключения dotfiles из результатов при использовании инструмента Glob. Включены по умолчанию. Не влияет на автодополнение `@`, `ls`, Grep или Read |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | Установите `false` чтобы инструмент Glob учитывал паттерны `.gitignore`. По умолчанию Glob возвращает все соответствующие файлы, включая gitignored. Не влияет на автодополнение `@`, у которого своя настройка `respectGitignore` |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Таймаут в секундах для обнаружения файлов Glob |
| `CLAUDE_CODE_ENABLE_TASKS` | Установите `true` для включения отслеживания задач в неинтерактивном режиме (флаг `-p`). Задачи включены по умолчанию в интерактивном режиме |
| `CLAUDE_CODE_SIMPLE` | Установите `1` для запуска с минимальным системным промптом и только инструментами Bash, чтения файлов и редактирования файлов |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | Автовыход из режима SDK после простоя (мс) |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | Отключить адаптивное мышление (`1` для отключения) |
| `CLAUDE_CODE_DISABLE_THINKING` | Принудительно отключить расширенное мышление (`1` для отключения) |
| `DISABLE_INTERLEAVED_THINKING` | Запретить отправку beta-заголовка interleaved-thinking (`1` для отключения) |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | Отключить окно контекста 1M токенов (`1` для отключения) |
| `CLAUDE_CODE_ACCOUNT_UUID` | Переопределить UUID аккаунта для аутентификации |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | Отключить git-инструкции системного промпта |
| `CLAUDE_CODE_NEW_INIT` | Установите `true` чтобы `/init` запускал интерактивный поток настройки. Спрашивает, какие файлы генерировать (CLAUDE.md, skills, hooks) перед исследованием кодовой базы. Без этого `/init` генерирует CLAUDE.md автоматически |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | Путь к одной или нескольким директориям seed-плагинов только для чтения, разделённых `:` на Unix или `;` на Windows. Встраивайте предзаполненные плагины в образ контейнера. Claude Code регистрирует маркетплейсы из этих директорий при запуске и использует предкэшированные плагины без повторного клонирования |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | Включить серверы MCP Claude.ai |
| `CLAUDE_CODE_EFFORT_LEVEL` | Установить уровень усилий: `low`, `medium`, `high`, `max` (только Opus 4.6) или `auto` (использовать значение модели по умолчанию). Имеет приоритет над `/effort` и настройкой `effortLevel` |
| `CLAUDE_CODE_MAX_TURNS` | Максимальное количество агентных ходов перед остановкой *(не в официальной документации — не проверено)* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Эквивалент установки `DISABLE_AUTOUPDATER`, `DISABLE_FEEDBACK_COMMAND`, `DISABLE_ERROR_REPORTING` и `DISABLE_TELEMETRY` |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | Пропустить поток настройки первого запуска *(не в официальной документации — не проверено)* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | Переопределить поведение кэширования промптов *(не в официальной документации — не проверено)* |
| `CLAUDE_CODE_DISABLE_TOOLS` | Список инструментов для отключения через запятую *(не в официальной документации — не проверено)* |
| `CLAUDE_CODE_DISABLE_MCP` | Отключить все серверы MCP (`1` для отключения) *(не в официальной документации — не проверено)* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Макс. токены вывода за ответ. По умолчанию: 32 000 (64 000 для Opus 4.6 с v2.1.77). Верхний предел: 64 000 (128 000 для Opus 4.6 и Sonnet 4.6 с v2.1.77) |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | Полностью отключить fast mode (`1` для отключения) |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | Установите `1` для отключения non-streaming fallback при сбое streaming-запроса в процессе. Ошибки streaming передаются на уровень повторных попыток. Полезно, когда прокси или gateway вызывает дублирование выполнения инструментов через fallback (v2.1.83) |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | Прерывать зависшие потоки (`1` для включения) |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | Включить детальный streaming инструментов (`1` для включения) |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | Отключить автоматическую память (`1` для отключения) |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | Отключить checkpointing файлов для `/rewind` (`1` для отключения) |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | Отключить обработку вложений (`1` для отключения) |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | Запретить загрузку файлов CLAUDE.md (`1` для отключения) |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | Автовозобновление, если предыдущая сессия завершилась посреди хода (`1` для включения) |
| `CLAUDE_CODE_USER_EMAIL` | Предоставить email пользователя синхронно для аутентификации |
| `CLAUDE_CODE_ORGANIZATION_UUID` | Предоставить UUID организации синхронно для аутентификации |
| `CLAUDE_CONFIG_DIR` | Пользовательская директория конфигурации (переопределяет `~/.claude` по умолчанию) |
| `CLAUDE_CODE_TMPDIR` | Переопределить временную директорию для внутренних временных файлов. Claude Code добавляет `/claude/` к этому пути. По умолчанию: `/tmp` на Unix/macOS, `os.tmpdir()` на Windows |
| `ANTHROPIC_CUSTOM_HEADERS` | Пользовательские заголовки для API-запросов (формат `Name: Value`, разделённые переносами строк для нескольких заголовков) |
| `ANTHROPIC_FOUNDRY_API_KEY` | API-ключ для аутентификации Microsoft Foundry |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Базовый URL для ресурса Foundry |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Имя ресурса Foundry |
| `AWS_BEARER_TOKEN_BEDROCK` | API-ключ Bedrock для аутентификации |
| `ANTHROPIC_SMALL_FAST_MODEL` | **УСТАРЕЛО** — Используйте `ANTHROPIC_DEFAULT_HAIKU_MODEL` вместо этого |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | Регион AWS для устаревшего переопределения модели класса Haiku |
| `CLAUDE_CODE_SHELL_PREFIX` | Командный префикс, добавляемый перед bash-командами |
| `BASH_DEFAULT_TIMEOUT_MS` | Таймаут bash-команд по умолчанию в мс |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | Пропустить аутентификацию AWS для Bedrock (`1` для пропуска) |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | Пропустить аутентификацию Azure для Foundry (`1` для пропуска) |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | Пропустить аутентификацию AWS для Bedrock Mantle (например, при использовании LLM-шлюза) |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | Пропустить аутентификацию Google для Vertex (`1` для пропуска) |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | Разрешить прокси выполнять DNS-разрешение |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | Интервал обновления учётных данных в мс для `apiKeyHelper` |
| `CLAUDE_CODE_CLIENT_CERT` | Путь к клиентскому сертификату для mTLS |
| `CLAUDE_CODE_CLIENT_KEY` | Путь к закрытому ключу клиента для mTLS |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | Парольная фраза для зашифрованного ключа mTLS |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | Таймаут git clone маркетплейса плагинов в мс (по умолчанию: 120000) |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | Переопределить корневую директорию плагинов |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | Пропустить автоматическое добавление официального маркетплейса (`1` для отключения) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | Ждать завершения установки плагина перед первым запросом (`1` для включения) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | Таймаут в мс для синхронной установки плагинов |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | Установите `1` для сохранения существующего кэша маркетплейса при неудаче `git pull` вместо удаления и повторного клонирования. Полезно в оффлайн или airgapped средах, где повторное клонирование не удастся тем же способом |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | Скрыть информацию email/org из UI *(не в официальной документации — не проверено)* |
| `CLAUDE_CODE_DISABLE_CRON` | Отключить запланированные/cron задачи (`1` для отключения) |
| `DISABLE_INSTALLATION_CHECKS` | Отключить предупреждения об установке |
| `DISABLE_FEEDBACK_COMMAND` | Отключить команду `/feedback`. Старое имя `DISABLE_BUG_COMMAND` также принимается |
| `DISABLE_DOCTOR_COMMAND` | Скрыть команду `/doctor` (`1` для отключения) |
| `DISABLE_LOGIN_COMMAND` | Скрыть команду `/login` (`1` для отключения) |
| `DISABLE_LOGOUT_COMMAND` | Скрыть команду `/logout` (`1` для отключения) |
| `DISABLE_UPGRADE_COMMAND` | Скрыть команду `/upgrade` (`1` для отключения) |
| `DISABLE_EXTRA_USAGE_COMMAND` | Скрыть команду `/extra-usage` (`1` для отключения) |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | Скрыть команду `/install-github-app` (`1` для отключения) |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | Отключить декоративный текст и несущественные вызовы модели *(не в официальной документации — не проверено)* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | Переопределить путь директории логов отладки |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | Минимальный уровень логов отладки |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | Принудительный автоматический перевод длительных задач в фон (`1` для включения) |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | Предотвратить переназначение Opus 4.0/4.1 на более новые модели (`1` для отключения) |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | Активировать fallback-модель для всех основных моделей, а не только для модели по умолчанию (`1` для включения) |
| `CLAUDE_CODE_GIT_BASH_PATH` | Путь к исполняемому файлу Windows Git Bash (только при запуске) |
| `DISABLE_COST_WARNINGS` | Отключить предупреждения о стоимости |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Переопределить модель для субагентов (например, `haiku`, `sonnet`) |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | Установите `1` для удаления учётных данных Anthropic и облачных провайдеров из сред подпроцессов (инструмент Bash, hooks, MCP stdio серверы). Используйте для глубокой защиты, когда подпроцессы не должны наследовать API-ключи (v2.1.83) |
| `CLAUDE_CODE_MAX_RETRIES` | Переопределить количество повторных попыток API-запросов (по умолчанию: 10) |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | Макс. параллельных инструментов только для чтения (по умолчанию: 10) |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | Отключить встроенные типы субагентов в режиме SDK (`1` для отключения) |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | Пропустить префикс `mcp__<server>__` для инструментов MCP в режиме SDK (`1` для включения) |
| `MCP_CONNECTION_NONBLOCKING` | Установите `true` в режиме `-p` для полного пропуска ожидания подключения MCP. Ограничивает подключения серверов `--mcp-config` до 5с вместо блокировки на самом медленном сервере *(в changelog v2.1.89, ещё не на официальной странице переменных окружения)* |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | Таймаут hooks SessionEnd в мс (заменяет жёсткий лимит 1.5с) |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | Отключить запросы опроса обратной связи (`1` для отключения) |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | Отключить обновления заголовка терминала (`1` для отключения) |
| `CLAUDE_CODE_NO_FLICKER` | Установите `1` для включения рендеринга alt-screen без мерцания. Устраняет визуальное мерцание при полноэкранных перерисовках (v2.1.88) |
| `CLAUDE_CODE_SCROLL_SPEED` | Множитель скорости прокрутки колёсика мыши для полноэкранного рендеринга. Увеличьте для более быстрой прокрутки, уменьшите для более точного управления |
| `CLAUDE_CODE_DISABLE_MOUSE` | Установите `1` для отключения отслеживания мыши в полноэкранном рендеринге. Полезно, когда события мыши мешают терминальным мультиплексорам или инструментам доступности |
| `CLAUDE_CODE_ACCESSIBILITY` | Установите `1` для сохранения видимости нативного курсора терминала для программ чтения с экрана и инструментов доступности |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | Установите `0` для отключения подсветки синтаксиса в выводе diff |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | Пропустить автоматическую установку расширения IDE (`1` для пропуска) |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | Переопределить поведение автоподключения к IDE |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | Переопределить адрес хоста IDE для подключения |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | Пропустить проверку lockfile IDE (`1` для пропуска) |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | Интервал debounce в мс для скрипта помощника заголовков OTel |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | Таймаут в мс для flush OpenTelemetry |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | Таймаут в мс для завершения OpenTelemetry |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | Таймаут в мс перед тем, как watchdog простоя потока закроет зависшее соединение. По умолчанию: `90000` (90 секунд). Увеличьте, если длительные инструменты или медленные сети вызывают преждевременные ошибки таймаута |
| `OTEL_LOG_TOOL_DETAILS` | Установите `1` для включения `tool_parameters` в события OpenTelemetry. По умолчанию не включается для конфиденциальности *(в changelog v2.1.85, ещё не на официальной странице переменных окружения)* |
| `CLAUDE_CODE_MCP_SERVER_NAME` | Имя сервера MCP, передаваемое как переменная окружения скриптам `headersHelper` для генерации серверо-специфичных заголовков аутентификации *(в changelog v2.1.85, ещё не на официальной странице переменных окружения)* |
| `CLAUDE_CODE_MCP_SERVER_URL` | URL сервера MCP, передаваемый как переменная окружения скриптам `headersHelper` вместе с `CLAUDE_CODE_MCP_SERVER_NAME` *(в changelog v2.1.85, ещё не на официальной странице переменных окружения)* |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Переопределить псевдоним модели Opus (например, `claude-opus-4-6[1m]`) |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | Настроить метку записи Opus в средстве выбора `/model` при использовании фиксированной модели на Bedrock/Vertex/Foundry. По умолчанию — ID модели |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | Настроить описание записи Opus в средстве выбора `/model`. По умолчанию `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | Переопределить обнаружение возможностей для фиксированной модели Opus. Значения через запятую (например, `effort,thinking`). Обязательно, когда фиксированная модель поддерживает функции, которые автообнаружение не может подтвердить |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Переопределить псевдоним модели Sonnet (например, `claude-sonnet-4-6`) |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | Настроить метку записи Sonnet в средстве выбора `/model` при использовании фиксированной модели на Bedrock/Vertex/Foundry. По умолчанию — ID модели |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | Настроить описание записи Sonnet в средстве выбора `/model`. По умолчанию `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | Переопределить обнаружение возможностей для фиксированной модели Sonnet. Значения через запятую (например, `effort,thinking`). Обязательно, когда фиксированная модель поддерживает функции, которые автообнаружение не может подтвердить |
| `MAX_THINKING_TOKENS` | Макс. токены расширенного мышления за ответ |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | Установить ёмкость контекста в токенах для расчётов автокомпактирования. По умолчанию — окно контекста модели (200K стандартное, 1M для моделей с расширенным контекстом). Используйте меньшее значение (например, `500000`) на модели 1M для расчёта компактирования как 500K. Ограничено фактическим окном контекста. `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` применяется как процент от этого значения. Установка этого разделяет порог компактирования и `used_percentage` строки состояния |
| `DISABLE_AUTO_COMPACT` | Отключить автоматическое компактирование контекста (`1` для отключения). Ручной `/compact` продолжает работать |
| `DISABLE_COMPACT` | Отключить всё компактирование — автоматическое и ручное (`1` для отключения) |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | Включить предложения промптов |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | Требовать режим плана для сессий |
| `CLAUDE_CODE_TEAM_NAME` | Имя команды для agent teams |
| `CLAUDE_CODE_TASK_LIST_ID` | ID списка задач для интеграции задач |
| `CLAUDE_ENV_FILE` | Пользовательский путь к файлу окружения |
| `FORCE_AUTOUPDATE_PLUGINS` | Принудительное автообновление плагинов (`1` для включения) |
| `HTTP_PROXY` | URL HTTP-прокси для сетевых запросов |
| `HTTPS_PROXY` | URL HTTPS-прокси для сетевых запросов |
| `NO_PROXY` | Список хостов через запятую, обходящих прокси |
| `MCP_TOOL_TIMEOUT` | Таймаут выполнения инструмента MCP в мс |
| `MCP_CLIENT_SECRET` | Секрет клиента OAuth MCP |
| `MCP_OAUTH_CALLBACK_PORT` | Порт обратного вызова OAuth MCP |
| `IS_DEMO` | Включить демо-режим |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | Бюджет символов для вывода инструмента slash-команд |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Переопределение региона Vertex AI для Claude 3.5 Haiku |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Переопределение региона Vertex AI для Claude 3.7 Sonnet |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Переопределение региона Vertex AI для Claude 4.0 Opus |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Переопределение региона Vertex AI для Claude 4.0 Sonnet |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Переопределение региона Vertex AI для Claude 4.1 Opus |

---

## Полезные команды

| Команда | Описание |
|---------|----------|
| `/model` | Переключение моделей и настройка уровня усилий Opus 4.6 |
| `/effort` | Прямая установка уровня усилий: `low`, `medium`, `high` (v2.1.76+) |
| `/config` | Интерактивный UI конфигурации |
| `/memory` | Просмотр/редактирование всех файлов памяти |
| `/agents` | Управление субагентами |
| `/mcp` | Управление серверами MCP |
| `/hooks` | Просмотр настроенных hooks |
| `/plugin` | Управление плагинами |
| `/keybindings` | Настройка пользовательских клавиатурных сочетаний |
| `/skills` | Просмотр и управление skills |
| `/permissions` | Просмотр и управление правилами разрешений |
| `--doctor` | Диагностика проблем конфигурации |
| `--debug` | Режим отладки с деталями выполнения hooks |

---

## Краткий справочник: полный пример

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "agent": "code-reviewer",
  "language": "english",
  "cleanupPeriodDays": 30,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "includeGitInstructions": true,
  "defaultShell": "bash",
  "plansDirectory": "./plans",
  "effortLevel": "medium",

  "worktree": {
    "symlinkDirectories": ["node_modules"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  },

  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0"
  },

  "autoMode": {
    "environment": [
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted internal domains: *.internal.example.com"
    ]
  },

  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*",
      "Agent(*)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../shared/"],
    "defaultMode": "acceptEdits"
  },

  "enableAllProjectMcpServers": true,

  "sandbox": {
    "enabled": true,
    "excludedCommands": ["git", "docker"],
    "filesystem": {
      "denyRead": ["./secrets/"],
      "denyWrite": ["./.env"]
    }
  },

  "attribution": {
    "commit": "Generated with Claude Code",
    "pr": ""
  },

  "statusLine": {
    "type": "command",
    "command": "git branch --show-current"
  },

  "spinnerTipsEnabled": true,
  "spinnerTipsOverride": {
    "tips": ["Custom tip 1", "Custom tip 2"],
    "excludeDefault": false
  },
  "prefersReducedMotion": false,

  "env": {
    "NODE_ENV": "development",
    "CLAUDE_CODE_EFFORT_LEVEL": "medium"
  }
}
```

---

## Источники

- [Документация настроек Claude Code](https://code.claude.com/docs/en/settings)
- [JSON Schema настроек Claude Code](https://json.schemastore.org/claude-code-settings.json)
- [Журнал изменений Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Примеры настроек Claude Code на GitHub](https://github.com/feiskyer/claude-code-settings)
- [Shipyard — шпаргалка CLI Claude Code](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [Справочник переменных окружения Claude Code](https://code.claude.com/docs/en/env-vars)
- [Справочник разрешений Claude Code](https://code.claude.com/docs/en/permissions)
