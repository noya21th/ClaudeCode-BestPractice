🌍 [English](../../../best-practice/claude-commands.md) | [中文](../../zh/best-practice/claude-commands.md) | [日本語](../../ja/best-practice/claude-commands.md) | [Français](../../fr/best-practice/claude-commands.md) | **Русский** | [العربية](../../ar/best-practice/claude-commands.md)

# Лучшие практики Commands

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A35%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-commands-implementation.md)

Commands Claude Code — поля frontmatter и официальные встроенные slash commands.

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
| `description` | string | Рекомендуется | Что делает command. Отображается в автодополнении и используется Claude для автообнаружения |
| `argument-hint` | string | Нет | Подсказка при автодополнении (напр. `[issue-number]`, `[filename]`) |
| `disable-model-invocation` | boolean | Нет | Установите `true`, чтобы запретить Claude автоматически вызывать эту command |
| `user-invocable` | boolean | Нет | Установите `false`, чтобы скрыть из меню `/` — command становится фоновыми знаниями |
| `paths` | string/list | Нет | Glob-паттерны, ограничивающие активацию skill. Принимает строку через запятую или YAML-список. Claude загружает skill автоматически только при работе с подходящими файлами |
| `allowed-tools` | string | Нет | Инструменты, разрешённые без запросов разрешений при активной command |
| `model` | string | Нет | Модель при запуске command (напр. `haiku`, `sonnet`, `opus`) |
| `effort` | string | Нет | Переопределение уровня усилий при вызове (`low`, `medium`, `high`, `max`) |
| `context` | string | Нет | Установите `fork` для запуска command в изолированном контексте subagent |
| `agent` | string | Нет | Тип subagent при установленном `context: fork` (по умолчанию: `general-purpose`) |
| `shell` | string | Нет | Shell для блоков `` !`command` `` — принимает `bash` (по умолчанию) или `powershell`. Требует `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | Нет | Lifecycle hooks, привязанные к этой command |

---

## ![Official](../../../!/tags/official.svg) **(65)**

| # | Command | Тег | Описание |
|---|---------|-----|----------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Вход в аккаунт Anthropic |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Выход из аккаунта Anthropic |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Настройка аутентификации Amazon Bedrock, региона и привязки моделей через интерактивный мастер. Видна только при `CLAUDE_CODE_USE_BEDROCK=1`. Впервые пользователи Bedrock могут запустить мастер с экрана входа |
| 4 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Открыть страницу обновления для перехода на более высокий тарифный план |
| 5 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Установить цвет панели промпта для текущей сессии. Доступные цвета: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`. Используйте `default` для сброса |
| 6 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Открыть интерфейс Settings для настройки темы, модели, стиля вывода и других параметров. Алиас: `/settings` |
| 7 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Открыть или создать файл конфигурации горячих клавиш |
| 8 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Управление правилами allow, ask и deny для разрешений инструментов. Открывает интерактивный диалог для просмотра правил по области, добавления/удаления правил, управления рабочими директориями и просмотра недавних отказов auto mode. Алиас: `/allowed-tools` |
| 9 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Просмотр и обновление настроек конфиденциальности. Только для подписчиков Pro и Max |
| 10 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Включение/выключение режима песочницы. Доступно только на поддерживаемых платформах |
| 11 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Настройка status line Claude Code. Опишите, что хотите, или запустите без аргументов для автоконфигурации из промпта shell |
| 12 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Заказать стикеры Claude Code |
| 13 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Настройка горячих клавиш терминала для Shift+Enter и других комбинаций. Видна только в терминалах, которым это нужно, таких как VS Code, Alacritty или Warp |
| 14 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Смена цветовой темы. Включает светлые и тёмные варианты, темы для дальтоников (daltonized) и ANSI-темы, использующие палитру вашего терминала |
| 15 | `/voice` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Включение/выключение голосовой диктовки по нажатию. Требует аккаунт Claude.ai |
| 16 | `/context` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Визуализация текущего использования контекста в виде цветной сетки. Показывает рекомендации по оптимизации для инструментов с большим контекстом, раздутия памяти и предупреждения о ёмкости |
| 17 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Показать статистику использования токенов. Подробности по подписке см. в руководстве по отслеживанию затрат |
| 18 | `/extra-usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Настройка дополнительного использования для продолжения работы при достижении лимитов |
| 19 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Генерация отчёта, анализирующего ваши сессии Claude Code: области проекта, паттерны взаимодействия и точки трения |
| 20 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Визуализация ежедневного использования, истории сессий, серий и предпочтений моделей |
| 21 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Открыть интерфейс Settings (вкладка Status): версия, модель, аккаунт и подключение. Работает во время ответа Claude, не дожидаясь завершения |
| 22 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Показать лимиты плана и состояние ограничений запросов |
| 23 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Диагностика и проверка установки и настроек Claude Code |
| 24 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Отправить отзыв о Claude Code. Алиас: `/bug` |
| 25 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Показать справку и доступные commands |
| 26 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Изучение функций Claude Code через быстрые интерактивные уроки с анимированными демо |
| 27 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Просмотр changelog в интерактивном переключателе версий. Выберите конкретную версию или покажите все |
| 28 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Список и управление фоновыми задачами. Алиас: `/bashes` |
| 29 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | Копировать последний ответ ассистента в буфер обмена. Передайте число `N` для копирования N-го ответа: `/copy 2` копирует предпоследний. При наличии блоков кода показывает интерактивный выбор. Нажмите `w` для записи в файл вместо буфера (полезно через SSH) |
| 30 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | Экспорт текущего разговора как простого текста. С именем файла — запись напрямую. Без — диалог копирования или сохранения |
| 31 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Управление конфигурациями agents |
| 32 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Настройка Claude in Chrome |
| 33 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Просмотр конфигураций hooks для событий инструментов |
| 34 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Управление интеграциями с IDE и отображение статуса |
| 35 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Управление подключениями MCP servers и OAuth-аутентификацией |
| 36 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Управление plugins Claude Code |
| 37 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Перезагрузка всех активных plugins для применения ожидающих изменений без перезапуска. Показывает количество перезагруженных компонентов и отмечает ошибки загрузки |
| 38 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Список доступных skills |
| 39 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | Редактирование файлов памяти `CLAUDE.md`, включение/отключение автопамяти и просмотр записей автопамяти |
| 40 | `/effort [low\|medium\|high\|max\|auto]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Установить уровень усилий модели. `low`, `medium` и `high` сохраняются между сессиями. `max` действует только для текущей сессии и требует Opus 4.6. `auto` сбрасывает к значению по умолчанию. Без аргумента показывает текущий уровень. Вступает в силу немедленно |
| 41 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Включение/выключение быстрого режима |
| 42 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Выбор или смена AI-модели. Для моделей с поддержкой используйте стрелки влево/вправо для регулировки уровня усилий. Изменение вступает в силу немедленно |
| 43 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Поделиться бесплатной неделей Claude Code с друзьями. Видна только если ваш аккаунт подходит |
| 44 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Вход в plan mode прямо из промпта. С описанием — вход в plan mode и немедленный старт с этой задачей, напр. `/plan fix the auth bug` |
| 45 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Создание плана в сессии ultraplan, просмотр в браузере, затем выполнение удалённо или отправка обратно в терминал |
| 46 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Добавить новую рабочую директорию к текущей сессии |
| 47 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Открыть интерактивный просмотрщик diff с незакоммиченными изменениями и diff по ходам. Стрелки влево/вправо для переключения между текущим git diff и отдельными ходами Claude, вверх/вниз для навигации по файлам |
| 48 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Инициализация проекта с файлом `CLAUDE.md`. Установите `CLAUDE_CODE_NEW_INIT=1` для интерактивного потока с настройкой skills, hooks и файлов персональной памяти |
| 49 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Устарела. Вместо этого установите plugin `code-review`: `claude plugin install code-review@claude-plugins-official` |
| 50 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Анализ ожидающих изменений текущей ветки на уязвимости безопасности. Проверяет git diff и выявляет риски: инъекции, проблемы авторизации, утечки данных |
| 51 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Продолжить текущую сессию в приложении Claude Code Desktop. Только macOS и Windows. Алиас: `/app` |
| 52 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Настройка приложения Claude GitHub Actions для репозитория. Проводит через выбор репозитория и настройку интеграции |
| 53 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Установка приложения Claude Slack. Открывает браузер для завершения OAuth-потока |
| 54 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Показать QR-код для загрузки мобильного приложения Claude. Алиасы: `/ios`, `/android` |
| 55 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Сделать сессию доступной для удалённого управления с claude.ai. Алиас: `/rc` |
| 56 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Настройка окружения по умолчанию для удалённых веб-сессий, запущенных с `--remote` |
| 57 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Создание, обновление, просмотр или запуск облачных запланированных задач. Claude проводит через настройку в диалоговом формате |
| 58 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Создать ветку текущего разговора в этой точке. Алиас: `/fork` |
| 59 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Задать быстрый побочный вопрос без добавления в разговор |
| 60 | `/clear` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Очистить историю разговора и освободить контекст. Алиасы: `/reset`, `/new` |
| 61 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Сжатие разговора с необязательными инструкциями фокусировки |
| 62 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Выход из CLI. Алиас: `/quit` |
| 63 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Переименовать текущую сессию и показать имя на панели промпта. Без имени — автогенерация из истории разговора |
| 64 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Возобновить разговор по ID или имени, или открыть выбор сессий. Алиас: `/continue` |
| 65 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Откатить разговор и/или код к предыдущей точке, или создать суммаризацию с выбранного сообщения. См. checkpointing. Алиас: `/checkpoint` |

Встроенные skills, такие как `/debug`, тоже могут появляться в меню slash commands, но они не являются встроенными commands.

---

## Источники

- [Slash Commands Claude Code](https://code.claude.com/docs/en/slash-commands)
- [Интерактивный режим Claude Code](https://code.claude.com/docs/en/interactive-mode)
- [CHANGELOG Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
