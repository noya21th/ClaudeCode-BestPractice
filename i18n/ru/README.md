🌍 [English](../../README.md) | [中文](../zh/README.md) | [日本語](../ja/README.md) | [Français](../fr/README.md) | **Русский** | [العربية](../ar/README.md)

# claude-code-best-practice
от vibe coding к agentic engineering — практика делает Claude совершенным

![updated with Claude Code](https://img.shields.io/badge/updated_with_Claude_Code-v2.1.96%20(Apr%2008%2C%202026%209%3A38%20PM%20PKT)-white?style=flat&labelColor=555)<br>

[![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/) [![Implemented](../../!/tags/implemented.svg)](../../implementation/) [![Orchestration Workflow](../../!/tags/orchestration-workflow.svg)](../../orchestration-workflow/orchestration-workflow.md) [![Claude](../../!/tags/claude.svg)](https://code.claude.com/docs) [![Boris](../../!/tags/boris-cherny.svg)](#-советы-и-рекомендации) [![Community](../../!/tags/community.svg)](#-подписка) ![Click on these badges below to see the actual sources](../../!/tags/click-badges.svg)<br>
<img src="../../!/tags/a.svg" height="14"> = Agents · <img src="../../!/tags/c.svg" height="14"> = Commands · <img src="../../!/tags/s.svg" height="14"> = Skills

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="Маскот Claude Code прыгает" width="120" height="100"><br>

</p>


## 🧠 КОНЦЕПЦИИ

| Возможность | Расположение | Описание |
|-------------|-------------|----------|
| <img src="../../!/tags/a.svg" height="14"> [**Subagents**](https://code.claude.com/docs/en/sub-agents) | `.claude/agents/<name>.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-subagents.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-subagents-implementation.md) Автономный актор в изолированном контексте — пользовательские инструменты, разрешения, модель, память и постоянная идентичность |
| <img src="../../!/tags/c.svg" height="14"> [**Commands**](https://code.claude.com/docs/en/slash-commands) | `.claude/commands/<name>.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-commands.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-commands-implementation.md) Знания, добавляемые в существующий контекст — простые шаблоны промптов для оркестрации рабочих процессов |
| <img src="../../!/tags/s.svg" height="14"> [**Skills**](https://code.claude.com/docs/en/skills) | `.claude/skills/<name>/SKILL.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-skills.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-skills-implementation.md) Знания, добавляемые в существующий контекст — настраиваемые, предзагружаемые, автоматически обнаруживаемые, с форком контекста и прогрессивным раскрытием · [Официальные Skills](https://github.com/anthropics/skills/tree/main/skills) |
| [**Workflows**](https://code.claude.com/docs/en/common-workflows) | [`.claude/commands/weather-orchestrator.md`](../../.claude/commands/weather-orchestrator.md) | [![Orchestration Workflow](../../!/tags/orchestration-workflow.svg)](../../orchestration-workflow/orchestration-workflow.md) |
| [**Hooks**](https://code.claude.com/docs/en/hooks) | `.claude/hooks/` | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-hooks) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/claude-code-hooks) Пользовательские обработчики (скрипты, HTTP, промпты, агенты), выполняемые вне агентного цикла при определённых событиях · [Руководство](https://code.claude.com/docs/en/hooks-guide) |
| [**MCP Servers**](https://code.claude.com/docs/en/mcp) | `.claude/settings.json`, `.mcp.json` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-mcp.md) [![Implemented](../../!/tags/implemented.svg)](../../.mcp.json) Подключения по Model Context Protocol к внешним инструментам, базам данных и API |
| [**Plugins**](https://code.claude.com/docs/en/plugins) | распространяемые пакеты | Наборы skills, subagents, hooks, MCP servers и LSP servers · [Маркетплейсы](https://code.claude.com/docs/en/discover-plugins) · [Создание маркетплейсов](https://code.claude.com/docs/en/plugin-marketplaces) |
| [**Settings**](https://code.claude.com/docs/en/settings) | `.claude/settings.json` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-settings.md) [![Implemented](../../!/tags/implemented.svg)](../../.claude/settings.json) Иерархическая система конфигурации · [Разрешения](https://code.claude.com/docs/en/permissions) · [Конфигурация модели](https://code.claude.com/docs/en/model-config) · [Стили вывода](https://code.claude.com/docs/en/output-styles) · [Песочница](https://code.claude.com/docs/en/sandboxing) · [Горячие клавиши](https://code.claude.com/docs/en/keybindings) · [Быстрый режим](https://code.claude.com/docs/en/fast-mode) |
| [**Status Line**](https://code.claude.com/docs/en/statusline) | `.claude/settings.json` | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-status-line) [![Implemented](../../!/tags/implemented.svg)](../../.claude/settings.json) Настраиваемая строка состояния с информацией об использовании контекста, модели, стоимости и сессии |
| [**Memory**](https://code.claude.com/docs/en/memory) | `CLAUDE.md`, `.claude/rules/`, `~/.claude/rules/`, `~/.claude/projects/<project>/memory/` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-memory.md) [![Implemented](../../!/tags/implemented.svg)](../../CLAUDE.md) Постоянный контекст через файлы CLAUDE.md и импорт `@path` · [Автоматическая память](https://code.claude.com/docs/en/memory) · [Правила](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) |
| [**Checkpointing**](https://code.claude.com/docs/en/checkpointing) | автоматически (на базе git) | Автоматическое отслеживание правок с откатом (`Esc Esc` или `/rewind`) и точечным суммированием |
| [**CLI Startup Flags**](https://code.claude.com/docs/en/cli-reference) | `claude [flags]` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-cli-startup-flags.md) Флаги командной строки, подкоманды и переменные окружения для запуска Claude Code · [Интерактивный режим](https://code.claude.com/docs/en/interactive-mode) |
| **AI Terms** | | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-codex-cursor-gemini/blob/main/reports/ai-terms.md) Agentic Engineering · Context Engineering · Vibe Coding |
| [**Best Practices**](https://code.claude.com/docs/en/best-practices) | | Официальные лучшие практики · [Prompt Engineering](https://github.com/anthropics/prompt-eng-interactive-tutorial) · [Расширение Claude Code](https://code.claude.com/docs/en/features-overview) |

### 🔥 Горячее

| Возможность | Расположение | Описание |
|-------------|-------------|----------|
| [**Power-ups**](../../best-practice/claude-power-ups.md) | `/powerup` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-power-ups.md) Интерактивные уроки, обучающие функциям Claude Code, с анимированными демо (v2.1.90) |
| [**Ultraplan**](https://code.claude.com/docs/en/ultraplan) ![beta](../../!/tags/beta.svg) | `/ultraplan` | Создание планов в облаке с просмотром в браузере, встроенными комментариями и гибким выполнением — удалённо или обратно в терминал |
| [**Claude Code Web**](https://code.claude.com/docs/en/claude-code-on-the-web) ![beta](../../!/tags/beta.svg) | `claude.ai/code` | Запуск задач на облачной инфраструктуре — длительные задачи, автоисправление PR, параллельные сессии без локальной настройки · [Запланированные задачи](https://code.claude.com/docs/en/web-scheduled-tasks) |
| [**No Flicker Mode**](https://code.claude.com/docs/en/fullscreen) ![beta](../../!/tags/beta.svg) | `CLAUDE_CODE_NO_FLICKER=1` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2039421575422980329) Режим без мерцания с поддержкой мыши, стабильной памятью и внутренней прокруткой |
| [**Computer Use**](https://code.claude.com/docs/en/computer-use) ![beta](../../!/tags/beta.svg) | `computer-use` MCP server | Позвольте Claude управлять вашим экраном — открывать приложения, кликать, вводить текст и делать скриншоты на macOS · [Desktop](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer) |
| [**Auto Mode**](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) ![beta](../../!/tags/beta.svg) | `claude --enable-auto-mode` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/claudeai/status/2036503582166393240) Фоновый классификатор безопасности заменяет ручные запросы разрешений — Claude сам решает, что безопасно · Запуск: `claude --enable-auto-mode` (или `--permission-mode auto`), или переключение через `Shift+Tab` во время сессии · [Блог](https://claude.com/blog/auto-mode) |
| [**Channels**](https://code.claude.com/docs/en/channels) ![beta](../../!/tags/beta.svg) | `--channels`, на основе plugin | Отправка событий из Telegram, Discord или вебхуков в запущенную сессию — Claude реагирует, пока вас нет · [Справочник](https://code.claude.com/docs/en/channels-reference) |
| [**Slack**](https://code.claude.com/docs/en/slack) | `@Claude` в Slack | Упомяните @Claude в командном чате с задачей — маршрутизация в веб-сессии Claude Code для исправления багов, code review и параллельного выполнения задач |
| [**Code Review**](https://code.claude.com/docs/en/code-review) ![beta](../../!/tags/beta.svg) | GitHub App (управляемый) | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/claudeai/status/2031088171262554195) Мультиагентный анализ PR, выявляющий баги, уязвимости безопасности и регрессии · [Блог](https://claude.com/blog/code-review) |
| [**GitHub Actions**](https://code.claude.com/docs/en/github-actions) | `.github/workflows/` | Автоматизация code review PR, сортировки задач и генерации кода в CI/CD пайплайнах · [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd) |
| [**Chrome**](https://code.claude.com/docs/en/chrome) ![beta](../../!/tags/beta.svg) | `--chrome`, расширение | [![Best Practice](../../!/tags/best-practice.svg)](../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) Автоматизация браузера через Claude in Chrome — тестирование веб-приложений, отладка через консоль, автоматизация форм, извлечение данных со страниц |
| [**Scheduled Tasks**](https://code.claude.com/docs/en/scheduled-tasks) | `/loop`, `/schedule`, cron tools | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2030193932404150413) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-scheduled-tasks-implementation.md) `/loop` запускает промпты локально по расписанию (до 3 дней) · [`/schedule`](https://code.claude.com/docs/en/web-scheduled-tasks) запускает промпты в облаке на инфраструктуре Anthropic — работает даже когда ваша машина выключена · [Анонс](https://x.com/noahzweben/status/2036129220959805859) |
| [**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation) ![beta](../../!/tags/beta.svg) | `/voice` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/trq212/status/2028628570692890800) Голосовой ввод по нажатию с поддержкой 20 языков и настраиваемой клавишей активации |
| [**Simplify & Batch**](https://code.claude.com/docs/en/skills#bundled-skills) | `/simplify`, `/batch` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2027534984534544489) Встроенные skills для качества кода и массовых операций — simplify рефакторит для повторного использования и эффективности, batch запускает команды по файлам |
| [**Agent Teams**](https://code.claude.com/docs/en/agent-teams) ![beta](../../!/tags/beta.svg) | встроено (env var) | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2019472394696683904) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-agent-teams-implementation.md) Несколько агентов, работающих параллельно над одной кодовой базой с координацией задач |
| [**Remote Control**](https://code.claude.com/docs/en/remote-control) | `/remote-control`, `/rc` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/noahzweben/status/2032533699116355819) Продолжение локальных сессий с любого устройства — телефон, планшет или браузер · [Headless Mode](https://code.claude.com/docs/en/headless) |
| [**Git Worktrees**](https://code.claude.com/docs/en/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees) | встроено | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2025007393290272904) Изолированные git-ветки для параллельной разработки — каждый агент получает свою рабочую копию |
| [**Ralph Wiggum Loop**](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) | plugin | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/ghuntley/how-to-ralph-wiggum) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26) Автономный цикл разработки для длительных задач — итерации до завершения |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

<a id="orchestration-workflow"></a>

## <a href="../../orchestration-workflow/orchestration-workflow.md"><img src="../../!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

Подробности реализации паттерна <img src="../../!/tags/c.svg" height="14"> **Command** → <img src="../../!/tags/a.svg" height="14"> **Agent** → <img src="../../!/tags/s.svg" height="14"> **Skill** смотрите в [orchestration-workflow](../../orchestration-workflow/orchestration-workflow.md).


<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.svg" alt="Архитектурная схема Command Skill Agent" width="100%">
</p>

<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.gif" alt="Демонстрация Orchestration Workflow" width="600">
</p>

![How to Use](../../!/tags/how-to-use.svg)

```bash
claude
/weather-orchestrator
```

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

## ⚙️ РАБОЧИЕ ПРОЦЕССЫ РАЗРАБОТКИ

Все основные рабочие процессы сходятся к одному архитектурному паттерну: **Исследование → План → Выполнение → Ревью → Релиз**

| Название | ★ | Уникальность | План | <img src="../../!/tags/a.svg" height="14"> | <img src="../../!/tags/c.svg" height="14"> | <img src="../../!/tags/s.svg" height="14"> |
|----------|---|--------------|------|---|---|---|
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | 146k | ![instinct scoring](https://img.shields.io/badge/instinct_scoring-ddf4ff) ![AgentShield](https://img.shields.io/badge/AgentShield-ddf4ff) ![multi-lang rules](https://img.shields.io/badge/multi--lang_rules-ddf4ff) | <img src="../../!/tags/a.svg" height="14"> [planner](https://github.com/affaan-m/everything-claude-code/blob/main/agents/planner.md) | 47 | 82 | 182 |
| [Superpowers](https://github.com/obra/superpowers) | 141k | ![TDD-first](https://img.shields.io/badge/TDD--first-ddf4ff) ![Iron Laws](https://img.shields.io/badge/Iron_Laws-ddf4ff) ![whole-plan review](https://img.shields.io/badge/whole--plan_review-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [writing-plans](https://github.com/obra/superpowers/tree/main/skills/writing-plans) | 5 | 3 | 14 |
| [Spec Kit](https://github.com/github/spec-kit) | 86k | ![spec-driven](https://img.shields.io/badge/spec--driven-ddf4ff) ![constitution](https://img.shields.io/badge/constitution-ddf4ff) ![22+ tools](https://img.shields.io/badge/22%2B_tools-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [speckit.plan](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md) | 0 | 9+ | 0 |
| [gstack](https://github.com/garrytan/gstack) | 67k | ![role personas](https://img.shields.io/badge/role_personas-ddf4ff) ![/codex review](https://img.shields.io/badge/%2Fcodex_review-ddf4ff) ![parallel sprints](https://img.shields.io/badge/parallel_sprints-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [autoplan](https://github.com/garrytan/gstack/tree/main/autoplan) | 0 | 0 | 37 |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) | 49k | ![fresh 200K contexts](https://img.shields.io/badge/fresh_200K_contexts-ddf4ff) ![wave execution](https://img.shields.io/badge/wave_execution-ddf4ff) ![XML plans](https://img.shields.io/badge/XML_plans-ddf4ff) | <img src="../../!/tags/a.svg" height="14"> [gsd-planner](https://github.com/gsd-build/get-shit-done/blob/main/agents/gsd-planner.md) | 24 | 68 | 0 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 44k | ![full SDLC](https://img.shields.io/badge/full_SDLC-ddf4ff) ![agent personas](https://img.shields.io/badge/agent_personas-ddf4ff) ![22+ platforms](https://img.shields.io/badge/22%2B_platforms-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [bmad-create-prd](https://github.com/bmad-code-org/BMAD-METHOD/tree/main/src/bmm-skills/2-plan-workflows/bmad-create-prd) | 0 | 0 | 39 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | 38k | ![delta specs](https://img.shields.io/badge/delta_specs-ddf4ff) ![brownfield](https://img.shields.io/badge/brownfield-ddf4ff) ![artifact DAG](https://img.shields.io/badge/artifact_DAG-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [opsx:propose](https://github.com/Fission-AI/OpenSpec/blob/main/src/commands/workflow/new-change.ts) | 0 | 11 | 0 |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 26k | ![teams orchestration](https://img.shields.io/badge/teams_orchestration-ddf4ff) ![tmux workers](https://img.shields.io/badge/tmux_workers-ddf4ff) ![skill auto-inject](https://img.shields.io/badge/skill_auto--inject-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [ralplan](https://github.com/Yeachan-Heo/oh-my-claudecode/tree/main/skills/ralplan) | 19 | 0 | 37 |
| [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | 14k | ![Compound Learning](https://img.shields.io/badge/Compound_Learning-ddf4ff) ![Multi-Platform CLI](https://img.shields.io/badge/Multi--Platform_CLI-ddf4ff) ![Plugin Marketplace](https://img.shields.io/badge/Plugin_Marketplace-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [ce-plan](https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills/ce-plan) | 51 | 4 | 44 |
| [HumanLayer](https://github.com/humanlayer/humanlayer) | 10k | ![RPI](https://img.shields.io/badge/RPI-ddf4ff) ![context engineering](https://img.shields.io/badge/context_engineering-ddf4ff) ![300k+ LOC](https://img.shields.io/badge/300k%2B_LOC-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [create_plan](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/create_plan.md) | 6 | 27 | 0 |

### Другие
- [Cross-Model (Claude Code + Codex) Workflow](../../development-workflows/cross-model-workflow/cross-model-workflow.md) [![Implemented](../../!/tags/implemented.svg)](../../development-workflows/cross-model-workflow/cross-model-workflow.md)
- [RPI](../../development-workflows/rpi/rpi-workflow.md) [![Implemented](../../!/tags/implemented.svg)](../../development-workflows/rpi/rpi-workflow.md)
- [Ralph Wiggum Loop](https://www.youtube.com/watch?v=eAtvoGlpeRU) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26)
- [Рабочий процесс Андрея Карпаты (сооснователь OpenAI)](https://x.com/karpathy/status/2015883857489522876)
- [Рабочий процесс Питера Штайнбергера (создатель OpenClaw)](https://youtu.be/8lF7HmQ_RgY?t=2582)
- Рабочий процесс Бориса Черны (создатель Claude Code) — [13 советов](../../tips/claude-boris-13-tips-03-jan-26.md) · [10 советов](../../tips/claude-boris-10-tips-01-feb-26.md) · [12 советов](../../tips/claude-boris-12-tips-12-feb-26.md) · [2 совета](../../tips/claude-boris-2-tips-25-mar-26.md) · [15 советов](../../tips/claude-boris-15-tips-30-mar-26.md) [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny)

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

## 💡 СОВЕТЫ И РЕКОМЕНДАЦИИ (69)

🚫👶 = не стоять над душой

[Промптинг](#tips-prompting) · [Планирование](#tips-planning) · [CLAUDE.md](#tips-claudemd) · [Agents](#tips-agents) · [Commands](#tips-commands) · [Skills](#tips-skills) · [Hooks](#tips-hooks) · [Рабочие процессы](#tips-workflows) · [Продвинутые](#tips-workflows-advanced) · [Git / PR](#tips-git-pr) · [Отладка](#tips-debugging) · [Утилиты](#tips-utilities) · [Ежедневные](#tips-daily)

![Community](../../!/tags/community.svg)

<a id="tips-prompting"></a>■ **Промптинг (3)**

| Совет | Источник |
|-------|----------|
| бросайте вызов Claude — «разнеси эти изменения и не делай PR, пока я не пройду твой тест» или «докажи мне, что это работает» и пусть Claude сделает diff между main и вашей веткой 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| после посредственного исправления — «зная всё, что ты сейчас знаешь, выбрось это и реализуй элегантное решение» 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| Claude сам исправляет большинство багов — вставьте баг, скажите «исправь», не микроменеджьте 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742750473720121) |

<a id="tips-planning"></a>■ **Планирование/Спецификации (6)**

| Совет | Источник |
|-------|----------|
| всегда начинайте с [plan mode](https://code.claude.com/docs/en/common-workflows) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179845336527000) |
| начните с минимальной спецификации или промпта и попросите Claude провести интервью с помощью инструмента [AskUserQuestion](https://code.claude.com/docs/en/cli-reference), затем создайте новую сессию для выполнения спецификации | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2005315275026260309) |
| всегда составляйте поэтапный план с контрольными точками, где каждый этап имеет тесты (юнит, автоматизация, интеграция) | |
| запустите второй Claude для ревью вашего плана как staff engineer, или используйте [cross-model](../../development-workflows/cross-model-workflow/cross-model-workflow.md) для ревью | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742745365057733) |
| пишите детальные спецификации и снижайте неоднозначность перед передачей работы — чем конкретнее, тем лучше результат | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| прототип > ТЗ — стройте 20-30 версий вместо написания спецификаций, стоимость создания низкая, делайте много попыток | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3630) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3630) |

<a id="tips-claudemd"></a>■ **CLAUDE.md (7)**

| Совет | Источник |
|-------|----------|
| [CLAUDE.md](https://code.claude.com/docs/en/memory) должен быть до [200 строк](https://code.claude.com/docs/en/memory#write-effective-instructions) на файл. [60 строк в humanlayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md) ([всё ещё не 100% гарантия](https://www.reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/)) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179840848597422) [![Dex](../../!/tags/community-dex.svg)](https://www.humanlayer.dev/blog/writing-a-good-claude-md) |
| оборачивайте доменные правила CLAUDE.md в [\<important if="..."\> теги](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md), чтобы Claude не игнорировал их по мере роста файлов | [![Dex](../../!/tags/community-dex.svg)](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md) |
| используйте [несколько CLAUDE.md](../../best-practice/claude-memory.md) для монорепозиториев — загрузка предков + потомков | |
| используйте [.claude/rules/](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) для разделения больших инструкций | |
| [memory.md](https://code.claude.com/docs/en/memory), constitution.md ничего не гарантируют | |
| любой разработчик должен запустить Claude, сказать «запусти тесты» и это должно сработать с первого раза — если нет, в вашем CLAUDE.md не хватает команд для настройки/сборки/тестирования | [![Dex](../../!/tags/community-dex.svg)](https://x.com/dexhorthy/status/2034713765401551053) |
| держите кодовую базу чистой и завершайте миграции — частично мигрированные фреймворки путают модели, которые могут выбрать неправильный паттерн | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=1112) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=1112) |
| используйте [settings.json](../../best-practice/claude-settings.md) для поведения, контролируемого harness (атрибуция, разрешения, модель) — не пишите «НИКОГДА не добавляй Co-Authored-By» в CLAUDE.md, когда `attribution.commit: ""` детерминистично | [![davila7](../../!/tags/community-davila7.svg)](https://x.com/dani_avila7/status/2036182734310195550) |

<a id="tips-agents"></a><img src="../../!/tags/a.svg" height="14"> **Agents (4)**

| Совет | Источник |
|-------|----------|
| создавайте специализированные [sub-agents](https://code.claude.com/docs/en/sub-agents) (дополнительный контекст) со [skills](https://code.claude.com/docs/en/skills) (прогрессивное раскрытие) вместо общих qa, backend engineer | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179850139000872) |
| скажите «используй subagents», чтобы бросить больше вычислительных ресурсов на проблему — разгружайте задачи, чтобы основной контекст оставался чистым 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| [agent teams с tmux](https://code.claude.com/docs/en/agent-teams) и [git worktrees](https://x.com/bcherny/status/2025007393290272904) для параллельной разработки | |
| используйте [test time compute](https://code.claude.com/docs/en/sub-agents) — отдельные контекстные окна дают лучшие результаты; один агент может создать баги, а другой (та же модель) может их найти | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031151689219321886) |

<a id="tips-commands"></a><img src="../../!/tags/c.svg" height="14"> **Commands (3)**

| Совет | Источник |
|-------|----------|
| используйте [commands](https://code.claude.com/docs/en/slash-commands) для рабочих процессов вместо [sub-agents](https://code.claude.com/docs/en/sub-agents) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| используйте [slash commands](https://code.claude.com/docs/en/slash-commands) для каждого «внутреннего цикла» работы, которую вы делаете много раз в день — экономит повторные промпты, commands хранятся в `.claude/commands/` и коммитятся в git | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| если вы делаете что-то больше одного раза в день, превратите это в [skill](https://code.claude.com/docs/en/skills) или [command](https://code.claude.com/docs/en/slash-commands) — создайте `/techdebt`, context-dump или analytics commands | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742748984742078) |

<a id="tips-skills"></a><img src="../../!/tags/s.svg" height="14"> **Skills (9)**

| Совет | Источник |
|-------|----------|
| используйте [context: fork](https://code.claude.com/docs/en/skills), чтобы запускать skill в изолированном subagent — основной контекст видит только финальный результат, а не промежуточные вызовы инструментов. Поле agent позволяет задать тип subagent | [![Lydia](../../!/tags/lydia.svg)](https://x.com/lydiahallie/status/2033603164398883042) |
| используйте [skills в подпапках](../../reports/claude-skills-for-larger-mono-repos.md) для монорепозиториев | |
| skills — это папки, а не файлы — используйте поддиректории references/, scripts/, examples/ для [прогрессивного раскрытия](https://code.claude.com/docs/en/skills) | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| добавляйте секцию Gotchas в каждый skill — контент с наибольшей ценностью, со временем добавляйте точки отказа Claude | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| поле description в skill — это триггер, а не описание — пишите для модели («когда мне активироваться?») | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| не описывайте очевидное в skills — фокусируйтесь на том, что выталкивает Claude из поведения по умолчанию 🚫👶 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| не загоняйте Claude в рамки в skills — давайте цели и ограничения, а не пошаговые инструкции 🚫👶 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| включайте скрипты и библиотеки в skills, чтобы Claude компоновал, а не воссоздавал шаблонный код | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| встраивайте `` !`command` `` в SKILL.md для инъекции динамического вывода shell в промпт — Claude выполняет его при вызове, и модель видит только результат | [![Lydia](../../!/tags/lydia.svg)](https://x.com/lydiahallie/status/2034337963820327017) |

<a id="tips-hooks"></a>■ **Hooks (5)**

| Совет | Источник |
|-------|----------|
| используйте [on-demand hooks](https://code.claude.com/docs/en/skills) в skills — /careful блокирует деструктивные команды, /freeze блокирует правки за пределами директории | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| [измеряйте использование skills](https://code.claude.com/docs/en/skills) с помощью PreToolUse hook для поиска популярных или недостаточно срабатывающих skills | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| используйте [PostToolUse hook](https://code.claude.com/docs/en/hooks) для автоформатирования кода — Claude генерирует хорошо отформатированный код, hook дорабатывает последние 10% для предотвращения сбоев CI | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179852047335529) |
| перенаправляйте [запросы разрешений](https://code.claude.com/docs/en/hooks) на Opus через hook — пусть сканирует атаки и автоматически одобряет безопасные 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| используйте [Stop hook](https://code.claude.com/docs/en/hooks), чтобы побудить Claude продолжить работу или проверить результат в конце хода | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701059253874861) |

<a id="tips-workflows"></a>■ **Рабочие процессы (7)**

| Совет | Источник |
|-------|----------|
| избегайте зоны тупости агента, делайте ручной [/compact](https://code.claude.com/docs/en/interactive-mode) на максимуме 50%. Используйте [/clear](https://code.claude.com/docs/en/cli-reference) для сброса контекста при переключении на новую задачу | |
| обычный cc лучше любых рабочих процессов для мелких задач | |
| используйте [/model](https://code.claude.com/docs/en/model-config) для выбора модели и рассуждений, [/context](https://code.claude.com/docs/en/interactive-mode) для просмотра использования контекста, [/usage](https://code.claude.com/docs/en/costs) для проверки лимитов плана, [/extra-usage](https://code.claude.com/docs/en/interactive-mode) для настройки дополнительного биллинга, [/config](https://code.claude.com/docs/en/settings) для настройки — используйте Opus для plan mode и Sonnet для кода, чтобы получить лучшее от обоих | [![Cat](../../!/tags/cat-wu.svg)](https://x.com/_catwu/status/1955694117264261609) |
| всегда используйте [thinking mode](https://code.claude.com/docs/en/model-config) true (для просмотра рассуждений) и [Output Style](https://code.claude.com/docs/en/output-styles) Explanatory (для детального вывода с блоками ★ Insight) в [/config](https://code.claude.com/docs/en/settings) для лучшего понимания решений Claude | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179838864666847) |
| используйте ключевое слово ultrathink в промптах для [глубокого рассуждения](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking#tips-and-best-practices) | |
| [/rename](https://code.claude.com/docs/en/cli-reference) важных сессий (напр. [TODO - рефакторинг]) и [/resume](https://code.claude.com/docs/en/cli-reference) позже — маркируйте каждый экземпляр при запуске нескольких Claude одновременно | [![Cat](../../!/tags/cat-wu.svg)](https://every.to/podcast/how-to-use-claude-code-like-the-people-who-built-it) |
| используйте [Esc Esc или /rewind](https://code.claude.com/docs/en/checkpointing) для отката, когда Claude уходит в сторону, вместо попыток исправить в том же контексте | |

<a id="tips-workflows-advanced"></a>■ **Продвинутые рабочие процессы (6)**

| Совет | Источник |
|-------|----------|
| используйте ASCII-диаграммы для понимания архитектуры | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742759218794768) |
| используйте [/loop](https://code.claude.com/docs/en/scheduled-tasks) для локального периодического мониторинга (до 3 дней) · используйте [/schedule](https://code.claude.com/docs/en/web-scheduled-tasks) для облачных периодических задач, работающих даже когда машина выключена | |
| используйте [Ralph Wiggum plugin](https://github.com/shanraisshan/novel-llm-26) для длительных автономных задач | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179858435281082) |
| [/permissions](https://code.claude.com/docs/en/permissions) с синтаксисом подстановки (Bash(npm run *), Edit(/docs/**)) вместо dangerously-skip-permissions | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179854077407667) |
| [/sandbox](https://code.claude.com/docs/en/sandboxing) для сокращения запросов разрешений с изоляцией файлов и сети — 84% сокращение внутри компании | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700506465579443) [![Cat](../../!/tags/cat-wu.svg)](https://creatoreconomy.so/p/inside-claude-code-how-an-ai-native-actually-works-cat-wu) |
| инвестируйте в skills для [проверки продукта](https://code.claude.com/docs/en/skills) (signup-flow-driver, checkout-verifier) — стоит потратить неделю на их совершенствование | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |

<a id="tips-git-pr"></a>■ **Git / PR (5)**

| Совет | Источник |
|-------|----------|
| держите PR маленькими и сфокусированными — [медиана 118 строк](../../tips/claude-boris-2-tips-25-mar-26.md) (141 PR, 45K строк изменений за день), одна фича на PR, легче ревьюить и откатывать | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| всегда [squash merge](../../tips/claude-boris-2-tips-25-mar-26.md) PR — чистая линейная история, один коммит на фичу, простой git revert и git bisect | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| коммитьте часто — старайтесь коммитить минимум раз в час, как только задача выполнена | ![Shayan](../../!/tags/community-shayan.svg) |
| тегайте [@claude](https://github.com/apps/claude) на PR коллеги для автогенерации lint-правил на повторяющуюся обратную связь — автоматизируйте себя из code review 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=2715) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=2715) |
| используйте [/code-review](https://code.claude.com/docs/en/code-review) для мультиагентного анализа PR — находит баги, уязвимости безопасности и регрессии перед мержем | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031089411820228645) |

<a id="tips-debugging"></a>■ **Отладка (7)**

| Совет | Источник |
|-------|----------|
| выработайте привычку делать скриншоты и отправлять Claude, когда застряли с проблемой | ![Shayan](../../!/tags/community-shayan.svg) |
| используйте MCP ([Claude in Chrome](https://code.claude.com/docs/en/chrome), [Playwright](https://github.com/microsoft/playwright-mcp), [Chrome DevTools](https://developer.chrome.com/blog/chrome-devtools-mcp)), чтобы Claude сам видел логи консоли Chrome | |
| всегда просите Claude запускать терминал (для просмотра логов) как фоновую задачу для лучшей отладки | |
| [/doctor](https://code.claude.com/docs/en/cli-reference) для диагностики проблем с установкой, аутентификацией и конфигурацией | |
| ошибку при compaction можно решить, выбрав модель с 1M токенов через [/model](https://code.claude.com/docs/en/model-config), затем запустив [/compact](https://code.claude.com/docs/en/interactive-mode) | |
| используйте [cross-model](../../development-workflows/cross-model-workflow/cross-model-workflow.md) для QA — напр. [Codex](https://github.com/shanraisshan/codex-cli-best-practice) для ревью плана и реализации | |
| агентный поиск (glob + grep) превосходит RAG — Claude Code пробовал и отказался от векторных баз данных, потому что код рассинхронизируется и разрешения сложны | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3095) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3095) |

<a id="tips-utilities"></a>■ **Утилиты (5)**

| Совет | Источник |
|-------|----------|
| терминалы [iTerm](https://iterm2.com/)/[Ghostty](https://ghostty.org/)/[tmux](https://github.com/tmux/tmux) вместо IDE ([VS Code](https://code.visualstudio.com/)/[Cursor](https://www.cursor.com/)) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742753971769626) |
| [/voice](https://code.claude.com/docs/en/voice-dictation) или [Wispr Flow](https://wisprflow.ai) для голосового промптинга (производительность x10) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038454362226467112) |
| [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks) для обратной связи от Claude | ![Shayan](../../!/tags/community-shayan.svg) |
| [status line](https://github.com/shanraisshan/claude-code-status-line) для осведомлённости о контексте и быстрого compacting | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700784019452195) ![Shayan](../../!/tags/community-shayan.svg) |
| изучите возможности [settings.json](../../best-practice/claude-settings.md), такие как [Plans Directory](../../best-practice/claude-settings.md#plans-directory), [Spinner Verbs](../../best-practice/claude-settings.md#display--ux) для персонализации | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701145023197516) |

<a id="tips-daily"></a>■ **Ежедневные (2)**

| Совет | Источник |
|-------|----------|
| [обновляйте](https://code.claude.com/docs/en/setup) Claude Code ежедневно | ![Shayan](../../!/tags/community-shayan.svg) |
| начинайте день с чтения [changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) | ![Shayan](../../!/tags/community-shayan.svg) |

![Boris Cherny + Team](../../!/tags/claude.svg)

| Статья / Твит | Источник |
|---------------|----------|
| [15 скрытых и недооценённых функций Claude Code (Борис) \| 30/Мар/26](../../tips/claude-boris-15-tips-30-mar-26.md) | [Твит](https://x.com/bcherny/status/2038454336355999749) |
| [Squash Merging и распределение размеров PR (Борис) \| 25/Мар/26](../../tips/claude-boris-2-tips-25-mar-26.md) | [Твит](https://x.com/bcherny/status/2038552880018538749) |
| [Уроки создания Claude Code: как мы используем Skills (Тарик) \| 17/Мар/26](../../tips/claude-thariq-tips-17-mar-26.md) | [Статья](https://x.com/trq212/status/2033949937936085378) |
| [Code Review и Test Time Compute (Борис) \| 10/Мар/26](../../tips/claude-boris-2-tips-10-mar-26.md) | [Твит](https://x.com/bcherny/status/2031089411820228645) |
| /loop — запланированные периодические задачи до 3 дней (Борис) \| 07 Мар 2026 | [Твит](https://x.com/bcherny/status/2030193932404150413) |
| AskUserQuestion + ASCII Markdowns (Тарик) \| 28 Фев 2026 | [Твит](https://x.com/trq212/status/2027543858289250472) |
| Видеть как агент — уроки создания Claude Code (Тарик) \| 28 Фев 2026 | [Статья](https://x.com/trq212/status/2027463795355095314) |
| Git Worktrees — 5 способов использования от Бориса \| 21 Фев 2026 | [Твит](https://x.com/bcherny/status/2025007393290272904) |
| Уроки создания Claude Code: Prompt Caching — это всё (Тарик) \| 20 Фев 2026 | [Статья](https://x.com/trq212/status/2024574133011673516) |
| [12 способов настройки своих Claude (Борис) \| 12/Фев/26](../../tips/claude-boris-12-tips-12-feb-26.md) | [Твит](https://x.com/bcherny/status/2021699851499798911) |
| [10 советов по использованию Claude Code от команды (Борис) \| 01/Фев/26](../../tips/claude-boris-10-tips-01-feb-26.md) | [Твит](https://x.com/bcherny/status/2017742741636321619) |
| [Как я использую Claude Code — 13 советов из моей удивительно простой настройки (Борис) \| 03/Янв/26](../../tips/claude-boris-13-tips-03-jan-26.md) | [Твит](https://x.com/bcherny/status/2007179832300581177) |
| Попросите Claude провести интервью с помощью инструмента AskUserQuestion (Тарик) \| 28/Дек/25 | [Твит](https://x.com/trq212/status/2005315275026260309) |
| Всегда используйте plan mode, дайте Claude возможность проверки, используйте /code-review (Борис) \| 27/Дек/25 | [Твит](https://x.com/bcherny/status/2004711722926616680) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

## 🎬 ВИДЕО / ПОДКАСТЫ

| Видео / Подкаст | Источник | YouTube |
|-----------------|----------|---------|
| Everything We Got Wrong About Research-Plan-Implement (Dex) \| 24 Мар 2026 \| MLOps Community | [![Dex](../../!/tags/community-dex.svg)](https://x.com/daborhyde) | [YouTube](https://youtu.be/YwZR6tc7qYg) |
| Building Claude Code with Boris Cherny (Борис) \| 04 Мар 2026 \| The Pragmatic Engineer | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/julbw1JuAz0) |
| Head of Claude Code: What happens after coding is solved (Борис) \| 19 Фев 2026 \| Lenny's Podcast | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/We7BZVKbCVw) |
| Inside Claude Code With Its Creator Boris Cherny (Борис) \| 17 Фев 2026 \| Y Combinator | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/PQU9o_5rHC4) |
| Boris Cherny (Creator of Claude Code) On What Grew His Career (Борис) \| 15 Дек 2025 \| Ryan Peterman | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/AmdLVWMdjOk) |
| The Secrets of Claude Code From the Engineers Who Built It (Cat) \| 29 Окт 2025 \| Every | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/IDSAMqip6ms) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

## 🔔 ПОДПИСКА

| Источник | Имя | Бейдж |
|----------|-----|-------|
| ![Reddit](https://img.shields.io/badge/-FF4500?style=flat&logo=reddit&logoColor=white) | [r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/), [r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/), [r/Anthropic](https://www.reddit.com/r/Anthropic/) | ![Boris + Team](../../!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Claude](https://x.com/claudeai), [Anthropic](https://x.com/AnthropicAI), [Boris](https://x.com/bcherny), [Thariq](https://x.com/trq212), [Cat](https://x.com/_catwu), [Lydia](https://x.com/lydiahallie), [Noah](https://x.com/noahzweben), [Anthony](https://x.com/amorriscode), [Alex](https://x.com/alexalbert__), [Kenneth](https://x.com/neilhtennek) | ![Boris + Team](../../!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Jesse Kriss](https://x.com/obra) ([Superpowers](https://github.com/obra/superpowers)), [Affaan Mustafa](https://x.com/affaanmustafa) ([ECC](https://github.com/affaan-m/everything-claude-code)), [Garry Tan](https://x.com/garrytan) ([gstack](https://github.com/garrytan/gstack)), [Dex Horthy](https://x.com/dexhorthy) ([HumanLayer](https://github.com/humanlayer/humanlayer)), [Kieran Klaassen](https://x.com/kieranklaassen) ([Compound Eng](https://github.com/EveryInc/compound-engineering-plugin)), [Tabish Gilani](https://x.com/0xTab) ([OpenSpec](https://github.com/Fission-AI/OpenSpec)), [Brian McAdams](https://x.com/BMadCode) ([BMAD](https://github.com/bmad-code-org/BMAD-METHOD)), [Lex Christopherson](https://x.com/official_taches) ([GSD](https://github.com/gsd-build/get-shit-done)), [Dani Avila](https://x.com/dani_avila7) ([CC Templates](https://github.com/davila7/claude-code-templates)), [Dan Shipper](https://x.com/danshipper) ([Every](https://every.to/)), [Andrej Karpathy](https://x.com/karpathy) ([AutoResearch](https://x.com/karpathy/status/2015883857489522876)), [Peter Steinberger](https://x.com/steipete) ([OpenClaw](https://x.com/openclaw)), [Sigrid Jin](https://x.com/realsigridjin) ([claw-code](https://github.com/ultraworkers/claw-code)), [Yeachan Heo](https://x.com/bellman_ych) ([oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)) | ![Community](../../!/tags/community.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Anthropic](https://www.youtube.com/@anthropic-ai) | ![Boris + Team](../../!/tags/claude.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Lenny's Podcast](https://www.youtube.com/@LennysPodcast), [Y Combinator](https://www.youtube.com/@ycombinator), [The Pragmatic Engineer](https://www.youtube.com/@pragmaticengineer), [Ryan Peterman](https://www.youtube.com/@ryanlpeterman), [Every](https://www.youtube.com/@every_media), [MLOps Community](https://www.youtube.com/@MLOps) | ![Community](../../!/tags/community.svg) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

## ☠️ СТАРТАПЫ / БИЗНЕСЫ

| Claude | Заменяет |
|-|-|
|[**Code Review**](https://code.claude.com/docs/en/code-review)|[Greptile](https://greptile.com), [CodeRabbit](https://coderabbit.ai), [Devin Review](https://devin.ai), [OpenDiff](https://opendiff.com), [Cursor BugBot](https://bugbot.dev)|
|[**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation)|[Wispr Flow](https://wisprflow.ai), [SuperWhisper](https://superwhisper.com/)|
|[**Remote Control**](https://code.claude.com/docs/en/remote-control)|[OpenClaw](https://openclaw.ai/)
|[**Claude in Chrome**](https://code.claude.com/docs/en/chrome)|[Playwright MCP](https://github.com/microsoft/playwright-mcp), [Chrome DevTools MCP](https://developer.chrome.com/blog/chrome-devtools-mcp)|
|[**Computer Use**](https://docs.anthropic.com/en/docs/agents-and-tools/computer-use)|[OpenAI CUA](https://openai.com/index/computer-using-agent/)|
|[**Cowork**](https://claude.com/blog/cowork-research-preview)|[ChatGPT Agent](https://openai.com/chatgpt/agent/), [Perplexity Computer](https://www.perplexity.ai/computer/), [Manus](https://manus.im)|
|[**Tasks**](https://x.com/trq212/status/2014480496013803643)|[Beads](https://github.com/steveyegge/beads)
|[**Plan Mode**](https://code.claude.com/docs/en/common-workflows)|[Agent OS](https://github.com/buildermethods/agent-os)|
|[**Skills / Plugins**](https://code.claude.com/docs/en/plugins)|AI-обёртки стартапов из YC ([reddit](https://reddit.com/r/ClaudeAI/comments/1r6bh4d/claude_code_skills_are_basically_yc_ai_startup/))|

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

<a id="billion-dollar-questions"></a>
![Billion-Dollar Questions](../../!/tags/billion-dollar-questions.svg)

*Обсуждайте эти вопросы в [Issues](https://github.com/noya21th/ClaudeCode-BestPractice/issues)*

**Память и инструкции (4)**

1. Что именно стоит записывать в CLAUDE.md — и что лучше не включать?
2. Если у вас уже есть CLAUDE.md, действительно ли нужен отдельный constitution.md или rules.md?
3. Как часто обновлять CLAUDE.md и как понять, что он устарел?
4. Почему Claude всё ещё игнорирует инструкции из CLAUDE.md — даже когда там написано MUST заглавными буквами? ([reddit](https://reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/))

**Agents, Skills и рабочие процессы (6)**

1. Когда использовать command, когда agent, когда skill — и когда обычный Claude Code просто лучше?
2. Как часто обновлять agents, commands и workflows по мере улучшения моделей?
3. Улучшает ли детальная персона subagent качество? Как выглядит «идеальная персона/промпт» для research/QA subagent?
4. Стоит ли полагаться на встроенный plan mode Claude Code — или строить свой planning command/agent, обеспечивающий рабочий процесс команды?
5. Если у вас есть персональный skill (напр. /implement с вашим стилем кода), как интегрировать community skills (напр. /simplify) без конфликтов — и кто побеждает при разногласиях?
6. Мы уже готовы? Можно ли конвертировать существующую кодовую базу в спецификации, удалить код и заставить AI регенерировать точно такой же код только из спецификаций?

**Спецификации и документация (3)**

1. Должна ли каждая фича в репозитории иметь спецификацию в виде markdown-файла?
2. Как часто нужно обновлять спецификации, чтобы они не устаревали при реализации новых фич?
3. При реализации новой фичи, как справляться с каскадным эффектом на спецификации других фич?

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

## ОТЧЁТЫ

<p align="center">
  <a href="../../reports/claude-agent-sdk-vs-cli-system-prompts.md"><img src="https://img.shields.io/badge/Agent_SDK_vs_CLI-555?style=for-the-badge" alt="Agent SDK vs CLI"></a>
  <a href="../../reports/claude-in-chrome-v-chrome-devtools-mcp.md"><img src="https://img.shields.io/badge/Browser_Automation_MCP-555?style=for-the-badge" alt="Browser Automation MCP"></a>
  <a href="../../reports/claude-global-vs-project-settings.md"><img src="https://img.shields.io/badge/Global_vs_Project_Settings-555?style=for-the-badge" alt="Global vs Project Settings"></a>
  <a href="../../reports/claude-skills-for-larger-mono-repos.md"><img src="https://img.shields.io/badge/Skills_in_Monorepos-555?style=for-the-badge" alt="Skills in Monorepos"></a>
  <br>
  <a href="../../reports/claude-agent-memory.md"><img src="https://img.shields.io/badge/Agent_Memory-555?style=for-the-badge" alt="Agent Memory"></a>
  <a href="../../reports/claude-advanced-tool-use.md"><img src="https://img.shields.io/badge/Advanced_Tool_Use-555?style=for-the-badge" alt="Advanced Tool Use"></a>
  <a href="../../reports/claude-usage-and-rate-limits.md"><img src="https://img.shields.io/badge/Usage_&_Rate_Limits-555?style=for-the-badge" alt="Usage & Rate Limits"></a>
  <a href="../../reports/claude-agent-command-skill.md"><img src="https://img.shields.io/badge/Agents_vs_Commands_vs_Skills-555?style=for-the-badge" alt="Agents vs Commands vs Skills"></a>
  <br>
  <a href="../../reports/llm-day-to-day-degradation.md"><img src="https://img.shields.io/badge/LLM_Degradation-555?style=for-the-badge" alt="LLM Degradation"></a>
</p>

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

![How to Use](../../!/tags/how-to-use.svg)

```
1. Читайте репозиторий как курс — изучите, что такое commands, agents, skills и hooks, прежде чем начинать их использовать.
2. Клонируйте этот репозиторий и поиграйте с примерами — попробуйте /weather-orchestrator, послушайте звуки hooks, запустите agent teams, чтобы увидеть, как всё работает на практике.
3. Перейдите в свой проект и попросите Claude предложить, какие лучшие практики из этого репозитория стоит добавить — дайте ему этот репозиторий в качестве справочника.
```

<a href="https://www.youtube.com/watch?v=AkAhkalkRY4"><img src="../../!/thumbnail/video-1.png" alt="Смотреть на YouTube" width="300"></a>

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="разделитель секций" width="60" height="50">
</p>

## Благодарности

На основе [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice). Спасибо Shayan Rais за вклад в open source.
