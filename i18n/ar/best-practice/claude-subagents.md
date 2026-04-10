<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-subagents.md) | [中文](../../zh/best-practice/claude-subagents.md) | [日本語](../../ja/best-practice/claude-subagents.md) | [Français](../../fr/best-practice/claude-subagents.md) | [Русский](../../ru/best-practice/claude-subagents.md) | **العربية**

# أفضل ممارسات Sub-agents

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A34%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-subagents-implementation.md)

Claude Code subagents — حقول frontmatter وأنواع الوكلاء الرسمية المدمجة.

<table width="100%">
<tr>
<td><a href="../../ar/README.md">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## حقول Frontmatter (16)

| الحقل | النوع | مطلوب | الوصف |
|-------|-------|-------|-------|
| `name` | string | نعم | معرّف فريد باستخدام أحرف صغيرة وشرطات |
| `description` | string | نعم | متى يتم الاستدعاء. استخدم `"PROACTIVELY"` للاستدعاء التلقائي بواسطة Claude |
| `tools` | string/list | لا | قائمة مسموحة من الأدوات مفصولة بفواصل (مثل `Read, Write, Edit, Bash`). يرث جميع الأدوات إذا حُذف. يدعم صيغة `Agent(agent_type)` لتقييد الوكلاء الفرعيين القابلين للإنشاء؛ الصيغة القديمة `Task(agent_type)` لا تزال تعمل |
| `disallowedTools` | string/list | لا | أدوات مرفوضة، تُزال من القائمة الموروثة أو المحددة |
| `model` | string | لا | النموذج المستخدم: `sonnet`، `opus`، `haiku`، معرّف نموذج كامل (مثل `claude-opus-4-6`)، أو `inherit` (الافتراضي: `inherit`) |
| `permissionMode` | string | لا | وضع الصلاحيات: `default`، `acceptEdits`، `auto`، `dontAsk`، `bypassPermissions`، أو `plan` |
| `maxTurns` | integer | لا | الحد الأقصى لعدد الدورات الوكيلية قبل توقف الوكيل الفرعي |
| `skills` | list | لا | أسماء Skills للتحميل المسبق في سياق الوكيل عند بدء التشغيل (يُحقن المحتوى الكامل، وليس مجرد إتاحته) |
| `mcpServers` | list | لا | خوادم MCP لهذا الوكيل الفرعي — سلاسل أسماء خوادم أو كائنات `{name: config}` مباشرة |
| `hooks` | object | لا | Hooks لدورة الحياة محصورة في هذا الوكيل الفرعي. جميع أحداث Hook مدعومة؛ `PreToolUse` و`PostToolUse` و`Stop` هي الأكثر شيوعاً |
| `memory` | string | لا | نطاق الذاكرة المستمرة: `user`، `project`، أو `local` |
| `background` | boolean | لا | اضبط على `true` للتشغيل دائماً كمهمة خلفية (الافتراضي: `false`) |
| `effort` | string | لا | تجاوز مستوى الجهد عندما يكون هذا الوكيل الفرعي نشطاً: `low`، `medium`، `high`، `max` (Opus 4.6 فقط). الافتراضي: يرث من الجلسة |
| `isolation` | string | لا | اضبط على `"worktree"` للتشغيل في git worktree مؤقت (يُنظّف تلقائياً إذا لم تكن هناك تغييرات) |
| `initialPrompt` | string | لا | يُرسل تلقائياً كأول دورة مستخدم عندما يعمل هذا الوكيل كوكيل الجلسة الرئيسي (عبر `--agent` أو إعداد `agent`). تُعالج Commands وSkills. يُضاف قبل أي أمر يقدمه المستخدم |
| `color` | string | لا | لون العرض للوكيل الفرعي في قائمة المهام والنص: `red`، `blue`، `green`، `yellow`، `purple`، `orange`، `pink`، أو `cyan` |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Agent | النموذج | الأدوات | الوصف |
|---|-------|---------|---------|-------|
| 1 | `general-purpose` | inherit | الكل | مهام معقدة متعددة الخطوات — نوع الوكيل الافتراضي للبحث واستكشاف الكود والعمل المستقل |
| 2 | `Explore` | haiku | قراءة فقط (بدون Write, Edit) | بحث واستكشاف سريع لقاعدة الكود — محسّن لإيجاد الملفات والبحث في الكود والإجابة على أسئلة قاعدة الكود |
| 3 | `Plan` | inherit | قراءة فقط (بدون Write, Edit) | بحث تخطيط مسبق في وضع الخطة — يستكشف قاعدة الكود ويصمم مناهج التنفيذ قبل كتابة الكود |
| 4 | `statusline-setup` | sonnet | Read, Edit | يكوّن إعداد شريط حالة Claude Code للمستخدم |
| 5 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | يجيب على أسئلة حول ميزات Claude Code وAgent SDK وClaude API |

---

## المصادر

- [إنشاء subagents مخصصة — وثائق Claude Code](https://code.claude.com/docs/en/sub-agents)
- [مرجع CLI — وثائق Claude Code](https://code.claude.com/docs/en/cli-reference)
- [سجل تغييرات Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

</div>
