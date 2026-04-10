<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-commands.md) | [中文](../../zh/best-practice/claude-commands.md) | [日本語](../../ja/best-practice/claude-commands.md) | [Français](../../fr/best-practice/claude-commands.md) | [Русский](../../ru/best-practice/claude-commands.md) | **العربية**

# أفضل ممارسات Commands

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A35%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-commands-implementation.md)

Claude Code commands — حقول frontmatter وأوامر slash الرسمية المدمجة.

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
| `description` | string | مستحسن | ما يفعله الأمر. يُعرض في الإكمال التلقائي ويستخدمه Claude للاكتشاف التلقائي |
| `argument-hint` | string | لا | تلميح يُعرض أثناء الإكمال التلقائي (مثل `[issue-number]`، `[filename]`) |
| `disable-model-invocation` | boolean | لا | اضبط على `true` لمنع Claude من استدعاء هذا الأمر تلقائياً |
| `user-invocable` | boolean | لا | اضبط على `false` لإخفائه من قائمة `/` — يصبح الأمر معرفة خلفية فقط |
| `paths` | string/list | لا | أنماط Glob التي تحدد متى يتم تفعيل هذه الـ skill. تقبل سلسلة مفصولة بفواصل أو قائمة YAML. عند الضبط، يحمّل Claude الـ skill تلقائياً فقط عند العمل مع ملفات مطابقة |
| `allowed-tools` | string | لا | أدوات مسموح بها بدون طلبات صلاحيات عندما يكون هذا الأمر نشطاً |
| `model` | string | لا | النموذج المستخدم عند تشغيل هذا الأمر (مثل `haiku`، `sonnet`، `opus`) |
| `effort` | string | لا | تجاوز مستوى جهد النموذج عند الاستدعاء (`low`، `medium`، `high`، `max`) |
| `context` | string | لا | اضبط على `fork` لتشغيل الأمر في سياق وكيل فرعي معزول |
| `agent` | string | لا | نوع الوكيل الفرعي عندما يكون `context: fork` مضبوطاً (الافتراضي: `general-purpose`) |
| `shell` | string | لا | Shell لكتل `` !`command` `` — يقبل `bash` (افتراضي) أو `powershell`. يتطلب `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | لا | Hooks لدورة الحياة محصورة في هذا الأمر |

---

## ![Official](../../../!/tags/official.svg) **(65)**

| # | الأمر | الفئة | الوصف |
|---|-------|-------|-------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | تسجيل الدخول إلى حساب Anthropic |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | تسجيل الخروج من حساب Anthropic |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | تكوين مصادقة Amazon Bedrock والمنطقة وتثبيت النموذج عبر معالج تفاعلي. مرئي فقط عند ضبط `CLAUDE_CODE_USE_BEDROCK=1` |
| 4 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | فتح صفحة الترقية للانتقال إلى مستوى خطة أعلى |
| 5 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | تعيين لون شريط الأوامر للجلسة الحالية |
| 6 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | فتح واجهة الإعدادات لضبط السمة والنموذج ونمط الإخراج والتفضيلات الأخرى. الاسم البديل: `/settings` |
| 7 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | فتح أو إنشاء ملف تكوين اختصارات المفاتيح |
| 8 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | إدارة قواعد السماح والسؤال والرفض لصلاحيات الأدوات. الاسم البديل: `/allowed-tools` |
| 9 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | عرض وتحديث إعدادات الخصوصية. متاح فقط لمشتركي خطتي Pro وMax |
| 10 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | تبديل وضع sandbox. متاح على المنصات المدعومة فقط |
| 11 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | تكوين شريط حالة Claude Code |
| 12 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | طلب ملصقات Claude Code |
| 13 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | تكوين اختصارات الطرفية لـ Shift+Enter وغيرها |
| 14 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | تغيير سمة الألوان |
| 15 | `/voice` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | تبديل الإملاء الصوتي بالضغط مع الكلام |
| 16 | `/context` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | عرض استخدام السياق الحالي كشبكة ملونة |
| 17 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | عرض إحصائيات استخدام الرموز |
| 18 | `/extra-usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | تكوين الاستخدام الإضافي للاستمرار عند الوصول لحدود المعدل |
| 19 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | إنشاء تقرير يحلل جلسات Claude Code |
| 20 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | عرض الاستخدام اليومي وتاريخ الجلسات والسلاسل وتفضيلات النموذج |
| 21 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | فتح واجهة الإعدادات (تبويب الحالة) يعرض الإصدار والنموذج والحساب والاتصال |
| 22 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | عرض حدود استخدام الخطة وحالة حد المعدل |
| 23 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | تشخيص والتحقق من تثبيت Claude Code وإعداداته |
| 24 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | إرسال ملاحظات حول Claude Code. الاسم البديل: `/bug` |
| 25 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | عرض المساعدة والأوامر المتاحة |
| 26 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | اكتشاف ميزات Claude Code عبر دروس تفاعلية سريعة مع عروض متحركة |
| 27 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | عرض سجل التغييرات في منتقي إصدارات تفاعلي |
| 28 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | عرض وإدارة المهام الخلفية. الاسم البديل: `/bashes` |
| 29 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | نسخ آخر رد للمساعد إلى الحافظة |
| 30 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | تصدير المحادثة الحالية كنص عادي |
| 31 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | إدارة تكوينات الوكلاء |
| 32 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | تكوين Claude في إعدادات Chrome |
| 33 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | عرض تكوينات Hook لأحداث الأدوات |
| 34 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | إدارة تكاملات IDE وعرض الحالة |
| 35 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | إدارة اتصالات خادم MCP ومصادقة OAuth |
| 36 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | إدارة إضافات Claude Code |
| 37 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | إعادة تحميل جميع الإضافات النشطة لتطبيق التغييرات المعلقة |
| 38 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | عرض قائمة Skills المتاحة |
| 39 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | تحرير ملفات ذاكرة CLAUDE.md وتمكين/تعطيل الذاكرة التلقائية |
| 40 | `/effort [low\|medium\|high\|max\|auto]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | تعيين مستوى جهد النموذج |
| 41 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | تبديل الوضع السريع |
| 42 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | اختيار أو تغيير نموذج AI |
| 43 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | مشاركة أسبوع مجاني من Claude Code مع الأصدقاء |
| 44 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | الدخول في وضع الخطة مباشرة |
| 45 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | صياغة خطة في جلسة ultraplan ومراجعتها في المتصفح |
| 46 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | إضافة مجلد عمل جديد للجلسة الحالية |
| 47 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | فتح عارض فروقات تفاعلي للتغييرات غير المُودعة |
| 48 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | تهيئة المشروع بدليل CLAUDE.md |
| 49 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | مهجور. ثبّت إضافة `code-review` بدلاً من ذلك |
| 50 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | تحليل التغييرات المعلقة للثغرات الأمنية |
| 51 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | متابعة الجلسة الحالية في تطبيق Claude Code Desktop |
| 52 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | إعداد تطبيق Claude GitHub Actions لمستودع |
| 53 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | تثبيت تطبيق Claude Slack |
| 54 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | عرض رمز QR لتنزيل تطبيق Claude للهاتف |
| 55 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | جعل هذه الجلسة متاحة للتحكم عن بُعد من claude.ai |
| 56 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | تكوين البيئة البعيدة الافتراضية |
| 57 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | إنشاء وتحديث وعرض أو تشغيل مهام Cloud المجدولة |
| 58 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | إنشاء فرع من المحادثة الحالية. الاسم البديل: `/fork` |
| 59 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | طرح سؤال جانبي سريع بدون إضافته للمحادثة |
| 60 | `/clear` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | مسح تاريخ المحادثة وتحرير السياق. الأسماء البديلة: `/reset`، `/new` |
| 61 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | ضغط المحادثة مع تعليمات تركيز اختيارية |
| 62 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | الخروج من CLI. الاسم البديل: `/quit` |
| 63 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | إعادة تسمية الجلسة الحالية |
| 64 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | استئناف محادثة بالمعرّف أو الاسم. الاسم البديل: `/continue` |
| 65 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | إرجاع المحادثة و/أو الكود إلى نقطة سابقة. الاسم البديل: `/checkpoint` |

Skills المُجمّعة مثل `/debug` يمكن أن تظهر في قائمة slash commands أيضاً، لكنها ليست أوامر مدمجة.

---

## المصادر

- [أوامر Slash في Claude Code](https://code.claude.com/docs/en/slash-commands)
- [الوضع التفاعلي في Claude Code](https://code.claude.com/docs/en/interactive-mode)
- [سجل تغييرات Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

</div>
