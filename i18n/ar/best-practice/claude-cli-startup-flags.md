<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-cli-startup-flags.md) | [中文](../../zh/best-practice/claude-cli-startup-flags.md) | [日本語](../../ja/best-practice/claude-cli-startup-flags.md) | [Français](../../fr/best-practice/claude-cli-startup-flags.md) | [Русский](../../ru/best-practice/claude-cli-startup-flags.md) | **العربية**

# أفضل ممارسات CLI Startup Flags

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026-white?style=flat&labelColor=555)

مرجع لأعلام بدء تشغيل Claude Code والأوامر الفرعية العليا ومتغيرات بيئة بدء التشغيل عند تشغيل Claude Code من الطرفية.

<table width="100%">
<tr>
<td><a href="../../ar/README.md">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## جدول المحتويات

1. [إدارة الجلسات](#إدارة-الجلسات)
2. [النموذج والتكوين](#النموذج-والتكوين)
3. [الصلاحيات والأمان](#الصلاحيات-والأمان)
4. [الإخراج والتنسيق](#الإخراج-والتنسيق)
5. [System Prompt](#system-prompt)
6. [Agent وSubagent](#agent-وsubagent)
7. [MCP والإضافات](#mcp-والإضافات)
8. [المجلد ومساحة العمل](#المجلد-ومساحة-العمل)
9. [الميزانية والحدود](#الميزانية-والحدود)
10. [التكامل](#التكامل)
11. [التهيئة والصيانة](#التهيئة-والصيانة)
12. [التصحيح والتشخيص](#التصحيح-والتشخيص)
13. [تجاوز الإعدادات](#تجاوز-الإعدادات)
14. [الإصدار والمساعدة](#الإصدار-والمساعدة)
15. [الأوامر الفرعية](#الأوامر-الفرعية)
16. [متغيرات البيئة](#متغيرات-البيئة)

---

## إدارة الجلسات

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--continue` | `-c` | متابعة أحدث محادثة في المجلد الحالي |
| `--resume` | `-r` | استئناف جلسة محددة بالمعرّف أو الاسم، أو عرض منتقي تفاعلي |
| `--from-pr <NUMBER\|URL>` | | استئناف الجلسات المرتبطة بـ GitHub PR محدد |
| `--fork-session` | | إنشاء معرّف جلسة جديد عند الاستئناف (استخدم مع `--resume` أو `--continue`) |
| `--session-id <UUID>` | | استخدام معرّف جلسة محدد (يجب أن يكون UUID صالح) |
| `--no-session-persistence` | | تعطيل حفظ الجلسة (وضع الطباعة فقط) |
| `--remote` | | إنشاء جلسة ويب جديدة على claude.ai |
| `--teleport` | | استئناف جلسة ويب في طرفيتك المحلية |

---

## النموذج والتكوين

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--model <NAME>` | | تعيين النموذج باسم مستعار (`sonnet`، `opus`، `haiku`) أو معرّف نموذج كامل |
| `--fallback-model <NAME>` | | نموذج احتياطي تلقائي عندما يكون النموذج الافتراضي مُحمّلاً بشكل زائد (وضع الطباعة فقط) |
| `--betas <LIST>` | | رؤوس beta لتضمينها في طلبات API (مستخدمي مفتاح API فقط) |

---

## الصلاحيات والأمان

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--dangerously-skip-permissions` | | تخطي جميع طلبات الصلاحيات. استخدم بحذر شديد |
| `--allow-dangerously-skip-permissions` | | تمكين تجاوز الصلاحيات كخيار بدون تفعيله |
| `--permission-mode <MODE>` | | البدء في وضع صلاحيات محدد: `default`، `plan`، `acceptEdits`، `bypassPermissions` |
| `--allowedTools <TOOLS>` | | أدوات تُنفّذ بدون سؤال (صيغة قواعد الصلاحيات) |
| `--disallowedTools <TOOLS>` | | أدوات تُزال من سياق النموذج بالكامل |
| `--tools <TOOLS>` | | تقييد الأدوات المدمجة التي يمكن لـ Claude استخدامها (استخدم `""` لتعطيل الكل) |
| `--permission-prompt-tool <TOOL>` | | تحديد أداة MCP للتعامل مع طلبات الصلاحيات في الوضع غير التفاعلي |

---

## الإخراج والتنسيق

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--print` | `-p` | طباعة الرد بدون الوضع التفاعلي (وضع headless/SDK) |
| `--output-format <FORMAT>` | | تنسيق الإخراج: `text`، `json`، `stream-json` |
| `--input-format <FORMAT>` | | تنسيق الإدخال: `text`، `stream-json` |
| `--json-schema <SCHEMA>` | | الحصول على JSON مُتحقق منه يطابق المخطط (وضع الطباعة فقط) |
| `--include-partial-messages` | | تضمين أحداث البث الجزئية (يتطلب `--print` و`--output-format=stream-json`) |
| `--verbose` | | تمكين التسجيل المفصل مع إخراج كامل دورة بدورة |

---

## System Prompt

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--system-prompt <TEXT>` | | استبدال كامل system prompt بنص مخصص |
| `--system-prompt-file <PATH>` | | تحميل system prompt من ملف، يستبدل الافتراضي (وضع الطباعة فقط) |
| `--append-system-prompt <TEXT>` | | إلحاق نص مخصص بـ system prompt الافتراضي |
| `--append-system-prompt-file <PATH>` | | إلحاق محتويات ملف بالأمر الافتراضي (وضع الطباعة فقط) |

---

## Agent وSubagent

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--agent <NAME>` | | تحديد وكيل للجلسة الحالية |
| `--agents <JSON>` | | تعريف وكلاء فرعيين مخصصين ديناميكياً عبر JSON |
| `--teammate-mode <MODE>` | | تعيين عرض فريق الوكلاء: `auto`، `in-process`، `tmux` |

---

## MCP والإضافات

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--mcp-config <PATH\|JSON>` | | تحميل خوادم MCP من ملف JSON أو سلسلة |
| `--strict-mcp-config` | | استخدام خوادم MCP من `--mcp-config` فقط، تجاهل الباقي |
| `--plugin-dir <PATH>` | | تحميل إضافات من مجلد لهذه الجلسة فقط (قابل للتكرار) |

---

## المجلد ومساحة العمل

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--add-dir <PATH>` | | إضافة مجلدات عمل إضافية لوصول Claude |
| `--worktree` | `-w` | بدء Claude في git worktree معزول (متفرع من HEAD) |

---

## الميزانية والحدود

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--max-budget-usd <AMOUNT>` | | الحد الأقصى بالدولار لاستدعاءات API قبل التوقف (وضع الطباعة فقط) |
| `--max-turns <NUMBER>` | | تحديد عدد الدورات الوكيلية (وضع الطباعة فقط) |

---

## التكامل

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--chrome` | | تمكين تكامل متصفح Chrome لأتمتة الويب |
| `--no-chrome` | | تعطيل تكامل متصفح Chrome لهذه الجلسة |
| `--ide` | | الاتصال تلقائياً بـ IDE عند بدء التشغيل إذا كان هناك IDE واحد صالح متاح بالضبط |

---

## التهيئة والصيانة

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--init` | | تشغيل hooks التهيئة وبدء الوضع التفاعلي |
| `--init-only` | | تشغيل hooks التهيئة والخروج (بدون جلسة تفاعلية) |
| `--maintenance` | | تشغيل hooks الصيانة والخروج |

---

## التصحيح والتشخيص

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--debug <CATEGORIES>` | | تمكين وضع التصحيح مع تصفية فئات اختيارية (مثل `"api,hooks"`) |

---

## تجاوز الإعدادات

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--settings <PATH\|JSON>` | | مسار ملف JSON للإعدادات أو سلسلة JSON للتحميل |
| `--setting-sources <LIST>` | | قائمة مصادر مفصولة بفواصل للتحميل: `user`، `project`، `local` |
| `--disable-slash-commands` | | تعطيل جميع skills وslash commands لهذه الجلسة |

---

## الإصدار والمساعدة

| العلم | المختصر | الوصف |
|-------|---------|-------|
| `--version` | `-v` | إخراج رقم الإصدار |
| `--help` | `-h` | عرض معلومات المساعدة |

---

## الأوامر الفرعية

هذه أوامر عليا تُشغّل كـ `claude <subcommand>`:

| الأمر الفرعي | الوصف |
|-------------|-------|
| `claude` | بدء REPL التفاعلي |
| `claude "query"` | بدء REPL مع أمر أولي |
| `claude agents` | عرض قائمة الوكلاء المكوّنين |
| `claude auth` | إدارة مصادقة Claude Code |
| `claude doctor` | تشغيل التشخيصات من سطر الأوامر |
| `claude install` | تثبيت أو تبديل بنيات Claude Code الأصلية |
| `claude mcp` | تكوين خوادم MCP (`add`، `remove`، `list`، `get`، `enable`) |
| `claude plugin` | إدارة إضافات Claude Code |
| `claude remote-control` | إدارة جلسات التحكم عن بُعد |
| `claude setup-token` | إنشاء رمز طويل الأمد لاستخدام الاشتراك |
| `claude update` / `claude upgrade` | التحديث إلى أحدث إصدار |

---

## متغيرات البيئة

متغيرات بيئة بدء التشغيل فقط تُضبط في shell قبل تشغيل Claude Code (لا يمكن تكوينها عبر `settings.json`):

| المتغير | الوصف |
|---------|-------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | تمكين فرق الوكلاء التجريبية |
| `CLAUDE_CODE_TMPDIR` | تجاوز مجلد temp للملفات الداخلية |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | تمكين تحميل CLAUDE.md من مجلدات إضافية |
| `DISABLE_AUTOUPDATER=1` | تعطيل التحديثات التلقائية |
| `CLAUDE_CODE_EFFORT_LEVEL` | التحكم في عمق التفكير |
| `USE_BUILTIN_RIPGREP=0` | استخدام ripgrep النظام بدلاً من المدمج (Alpine Linux) |
| `CLAUDE_CODE_SIMPLE` | تمكين الوضع البسيط (أدوات Bash + Edit فقط) |
| `CLAUDE_BASH_NO_LOGIN=1` | تخطي shell تسجيل الدخول لـ BashTool |

لمتغيرات البيئة القابلة للتكوين عبر مفتاح `"env"` في `settings.json` (بما في ذلك `MAX_THINKING_TOKENS` و`CLAUDE_CODE_SHELL` و`CLAUDE_CODE_ENABLE_TASKS` والمزيد)، راجع [مرجع إعدادات Claude](../../../best-practice/claude-settings.md#environment-variables-via-env).

---

## المصادر

- [مرجع CLI لـ Claude Code](https://code.claude.com/docs/en/cli-reference)
- [وضع Headless لـ Claude Code](https://code.claude.com/docs/en/headless)
- [إعداد Claude Code](https://code.claude.com/docs/en/setup)
- [سجل تغييرات Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [سير العمل الشائعة لـ Claude Code](https://code.claude.com/docs/en/common-workflows)

</div>
