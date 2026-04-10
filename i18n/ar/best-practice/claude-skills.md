<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-skills.md) | [中文](../../zh/best-practice/claude-skills.md) | [日本語](../../ja/best-practice/claude-skills.md) | [Français](../../fr/best-practice/claude-skills.md) | [Русский](../../ru/best-practice/claude-skills.md) | **العربية**

# أفضل ممارسات Skills

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A33%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-skills-implementation.md)

Claude Code skills — حقول frontmatter والمهارات الرسمية المُجمّعة.

<table width="100%">
<tr>
<td><a href="../../ar/README.md">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## حقول Frontmatter (13)

| الحقل | النوع | مطلوب | الوصف |
|-------|-------|-------|-------|
| `name` | string | لا | اسم العرض ومعرّف `/slash-command`. يأخذ اسم المجلد افتراضياً إذا حُذف |
| `description` | string | مستحسن | ما تفعله الـ skill. تُعرض في الإكمال التلقائي ويستخدمها Claude للاكتشاف التلقائي |
| `argument-hint` | string | لا | تلميح يُعرض أثناء الإكمال التلقائي (مثل `[issue-number]`، `[filename]`) |
| `disable-model-invocation` | boolean | لا | اضبط على `true` لمنع Claude من استدعاء هذه الـ skill تلقائياً |
| `user-invocable` | boolean | لا | اضبط على `false` لإخفائها من قائمة `/` — تصبح الـ skill معرفة خلفية فقط، مخصصة للتحميل المسبق في الوكلاء |
| `allowed-tools` | string | لا | أدوات مسموح بها بدون طلبات صلاحيات عندما تكون هذه الـ skill نشطة |
| `model` | string | لا | النموذج المستخدم عند تشغيل هذه الـ skill (مثل `haiku`، `sonnet`، `opus`) |
| `effort` | string | لا | تجاوز مستوى جهد النموذج عند الاستدعاء (`low`، `medium`، `high`، `max`) |
| `context` | string | لا | اضبط على `fork` لتشغيل الـ skill في سياق وكيل فرعي معزول |
| `agent` | string | لا | نوع الوكيل الفرعي عندما يكون `context: fork` مضبوطاً (الافتراضي: `general-purpose`) |
| `hooks` | object | لا | Hooks لدورة الحياة محصورة في هذه الـ skill |
| `paths` | string/list | لا | أنماط Glob التي تحدد متى تنشط الـ skill تلقائياً. تقبل سلسلة مفصولة بفواصل أو قائمة YAML — Claude يحمّل الـ skill فقط عند العمل مع ملفات مطابقة |
| `shell` | string | لا | Shell لكتل `` !`command` `` — `bash` (افتراضي) أو `powershell`. يتطلب `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Skill | الوصف |
|---|-------|-------|
| 1 | `simplify` | مراجعة الكود المعدّل لإعادة الاستخدام والجودة والكفاءة — يعيد الهيكلة للقضاء على التكرار |
| 2 | `batch` | تشغيل الأوامر عبر ملفات متعددة بشكل مجمّع |
| 3 | `debug` | تصحيح الأوامر الفاشلة أو مشاكل الكود |
| 4 | `loop` | تشغيل أمر أو slash command على فترات متكررة (حتى 3 أيام) |
| 5 | `claude-api` | بناء تطبيقات باستخدام Claude API أو Anthropic SDK — ينشط عند استيراد `anthropic` / `@anthropic-ai/sdk` |

راجع أيضاً: [مستودع Skills الرسمي](https://github.com/anthropics/skills/tree/main/skills) للمهارات القابلة للتثبيت التي يُديرها المجتمع.

---

## المصادر

- [Claude Code Skills — الوثائق](https://code.claude.com/docs/en/skills)
- [اكتشاف Skills في المستودعات الأحادية](../../../reports/claude-skills-for-larger-mono-repos.md)
- [سجل تغييرات Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

</div>
