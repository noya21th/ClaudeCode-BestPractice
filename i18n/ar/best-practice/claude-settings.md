<!-- dir: rtl -->
<div dir="rtl">

🌍 [English](../../../best-practice/claude-settings.md) | [中文](../../zh/best-practice/claude-settings.md) | [日本語](../../ja/best-practice/claude-settings.md) | [Français](../../fr/best-practice/claude-settings.md) | [Русский](../../ru/best-practice/claude-settings.md) | **العربية**

# أفضل ممارسات Settings

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%2010%3A16%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.claude/settings.json)

دليل شامل لجميع خيارات التهيئة المتاحة في ملفات `settings.json` الخاصة بـ Claude Code. اعتبارًا من الإصدار v2.1.96، يوفّر Claude Code أكثر من **60 إعدادًا** وأكثر من **170 متغير بيئة** (استخدم حقل `"env"` في `settings.json` لتجنّب كتابة سكربتات مغلّفة).

<table width="100%">
<tr>
<td><a href="../../ar/">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## جدول المحتويات

1. [تسلسل الأولويات في Settings](#تسلسل-الأولويات-في-settings)
2. [التهيئة الأساسية](#التهيئة-الأساسية)
3. [الصلاحيات](#الصلاحيات)
4. [Hooks](#hooks)
5. [خوادم MCP](#خوادم-mcp)
6. [Sandbox](#sandbox)
7. [Plugins](#plugins)
8. [إعدادات النموذج](#إعدادات-النموذج)
9. [العرض وتجربة المستخدم](#العرض-وتجربة-المستخدم)
10. [بيانات اعتماد AWS والسحابة](#بيانات-اعتماد-aws-والسحابة)
11. [متغيرات البيئة](#متغيرات-البيئة-عبر-env)
12. [أوامر مفيدة](#أوامر-مفيدة)

---

## تسلسل الأولويات في Settings

تُطبَّق الإعدادات وفق ترتيب الأولوية (من الأعلى إلى الأدنى):

| الأولوية | الموقع | النطاق | مشترك؟ | الغرض |
|----------|--------|--------|--------|-------|
| 1 | Managed settings | المؤسسة | نعم (يُنشر من قِبل IT) | سياسات أمنية لا يمكن تجاوزها |
| 2 | وسائط سطر الأوامر | الجلسة | غير متاح | تجاوزات مؤقتة لجلسة واحدة |
| 3 | `.claude/settings.local.json` | المشروع | لا (مُتجاهَل في git) | إعدادات شخصية خاصة بالمشروع |
| 4 | `.claude/settings.json` | المشروع | نعم (مُلتزَم في git) | إعدادات مشتركة للفريق |
| 5 | `~/.claude/settings.json` | المستخدم | غير متاح | الإعدادات الشخصية الافتراضية العامة |

**Managed settings** هي إعدادات مفروضة من المؤسسة ولا يمكن تجاوزها بأي مستوى آخر، بما في ذلك وسائط سطر الأوامر. طرق التوزيع:
- **Server-managed** settings (توزيع عن بُعد)
- **ملفات تعريف MDM** — macOS plist في `com.anthropic.claudecode`
- **سياسات Registry** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode` (مسؤول النظام) و `HKCU\SOFTWARE\Policies\ClaudeCode` (مستوى المستخدم، أدنى أولوية سياسة)
- **ملف** — `managed-settings.json` و `managed-mcp.json` (macOS: `/Library/Application Support/ClaudeCode/`، Linux/WSL: `/etc/claude-code/`، Windows: `C:\Program Files\ClaudeCode\`)
- **مجلد إضافي (Drop-in directory)** — `managed-settings.d/` بجانب `managed-settings.json` لأجزاء سياسة مستقلة (v2.1.83). وفقًا لاتفاقية systemd، يُدمج `managed-settings.json` أولًا كقاعدة، ثم تُرتَّب جميع ملفات `*.json` في المجلد الإضافي أبجديًا وتُدمج فوقها. الملفات اللاحقة تتجاوز السابقة للقيم المفردة؛ المصفوفات تُدمج وتُزال التكرارات؛ الكائنات تُدمج بعمق. تُتجاهل الملفات المخفية التي تبدأ بـ `.`. استخدم بادئات رقمية للتحكم في ترتيب الدمج (مثلًا `10-telemetry.json`، `20-security.json`)

ضمن مستوى Managed، ترتيب الأولوية هو: server-managed > سياسات MDM/OS > ملفات (`managed-settings.d/*.json` + `managed-settings.json`) > HKCU registry (Windows فقط). يُستخدم مصدر managed واحد فقط؛ المصادر لا تُدمج عبر المستويات. ضمن مستوى الملفات، تُدمج ملفات Drop-in والملف الأساسي معًا.

> **ملاحظة:** اعتبارًا من v2.1.75، تمت إزالة مسار Windows الاحتياطي القديم `C:\ProgramData\ClaudeCode\managed-settings.json`. استخدم `C:\Program Files\ClaudeCode\managed-settings.json` بدلًا منه.

**هام**:
- قواعد `deny` لها أعلى أولوية أمان ولا يمكن تجاوزها بقواعد allow/ask ذات أولوية أدنى.
- قد تقفل Managed settings السلوك المحلي أو تتجاوزه حتى لو حددت الملفات المحلية قيمًا مختلفة.
- إعدادات المصفوفات (مثل `permissions.allow`) تُدمج وتُزال التكرارات عبر جميع النطاقات — تُجمَع الإدخالات من جميع المستويات ولا تُستبدل.

---

## التهيئة الأساسية

### الإعدادات العامة

| المفتاح | النوع | الافتراضي | الوصف |
|---------|-------|-----------|-------|
| `$schema` | string | - | رابط JSON Schema للتحقق والإكمال التلقائي في بيئة التطوير (مثلًا `"https://json.schemastore.org/claude-code-settings.json"`) |
| `model` | string | `"default"` | تجاوز النموذج الافتراضي. يقبل أسماء مستعارة (`sonnet`، `opus`، `haiku`) أو معرّفات النموذج الكاملة |
| `agent` | string | - | تعيين Agent الافتراضي للمحادثة الرئيسية. القيمة هي اسم Agent من `.claude/agents/`. متاح أيضًا عبر علامة CLI `--agent` |
| `language` | string | `"english"` | لغة الاستجابة المفضلة لـ Claude. تُحدد أيضًا لغة الإملاء الصوتي |
| `cleanupPeriodDays` | number | `30` | تُحذف الجلسات غير النشطة لفترة أطول من هذه المدة عند بدء التشغيل (الحد الأدنى 1). تتحكم أيضًا في حد العمر للإزالة التلقائية لـ worktrees الفرعية اليتيمة عند بدء التشغيل. تعيين القيمة `0` يُرفض مع خطأ تحقق. لتعطيل كتابة النصوص في الوضع غير التفاعلي (`-p`)، استخدم `--no-session-persistence` أو خيار SDK `persistSession: false` |
| `autoUpdatesChannel` | string | `"latest"` | قناة الإصدار: `"stable"` أو `"latest"` |
| `alwaysThinkingEnabled` | boolean | `false` | تفعيل التفكير الموسّع افتراضيًا لجميع الجلسات |
| `skipWebFetchPreflight` | boolean | `false` | تخطي فحص القائمة المحظورة في WebFetch قبل جلب عناوين URL *(في مخطط JSON، وليس في صفحة الإعدادات الرسمية)* |
| `availableModels` | array | - | تقييد النماذج التي يمكن للمستخدمين اختيارها عبر `/model` و `--model` وأداة Config و `ANTHROPIC_MODEL`. لا يؤثر على الخيار الافتراضي. مثال: `["sonnet", "haiku"]` |
| `fastModePerSessionOptIn` | boolean | `false` | يتطلب من المستخدمين الاشتراك في الوضع السريع لكل جلسة |
| `defaultShell` | string | `"bash"` | Shell الافتراضي لأوامر `!` في مربع الإدخال. يقبل `"bash"` (الافتراضي) أو `"powershell"`. تعيين `"powershell"` يوجّه أوامر `!` التفاعلية عبر PowerShell على Windows. يتطلب `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` (v2.1.84) |
| `includeGitInstructions` | boolean | `true` | تضمين تعليمات سير عمل commit و PR المدمجة ولقطة git status في system prompt الخاص بـ Claude. متغير البيئة `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` له الأولوية على هذا الإعداد عند تعيينه |
| `voiceEnabled` | boolean | - | تفعيل الإملاء الصوتي بالضغط المستمر. يُكتب تلقائيًا عند تشغيل `/voice`. يتطلب حساب Claude.ai |
| `showClearContextOnPlanAccept` | boolean | `false` | إظهار خيار "مسح السياق" في شاشة قبول الخطة. عيّنه إلى `true` لاستعادة الخيار (مخفي افتراضيًا منذ v2.1.81) |
| `disableDeepLinkRegistration` | string | - | عيّنه إلى `"disable"` لمنع Claude Code من تسجيل معالج البروتوكول `claude-cli://` مع نظام التشغيل عند بدء التشغيل. تسمح الروابط العميقة للأدوات الخارجية بفتح جلسة Claude Code مع prompt مملوء مسبقًا عبر `claude-cli://open?q=...`. يدعم معامل `q` الأوامر متعددة الأسطر باستخدام أسطر جديدة مشفرة بـ URL (`%0A`). مفيد في البيئات التي يكون فيها تسجيل معالج البروتوكول مقيدًا أو مُدارًا بشكل منفصل |
| `showThinkingSummaries` | boolean | `false` | إظهار ملخصات التفكير الموسّع في الجلسات التفاعلية. عند عدم التعيين أو `false` (الافتراضي في الوضع التفاعلي)، تُحذف كتل التفكير بواسطة API وتُعرض كعنصر مطوي. الحذف يغيّر فقط ما تراه وليس ما ينتجه النموذج — لتقليل إنفاق التفكير، قلّل الميزانية أو عطّل التفكير بدلًا من ذلك. الوضع غير التفاعلي (`-p`) ومستدعو SDK يتلقون الملخصات دائمًا بغض النظر عن هذا الإعداد |
| `disableSkillShellExecution` | boolean | `false` | تعطيل تنفيذ Shell المضمّن لكتل `` !`...` `` في Skills والأوامر المخصصة. تُستبدل الأوامر بـ `[shell command execution disabled by policy]`. لا تتأثر Skills المجمّعة والمُدارة (v2.1.91) |
| `forceRemoteSettingsRefresh` | boolean | `false` | **(Managed فقط)** حظر بدء تشغيل CLI حتى يتم جلب Managed settings عن بُعد بشكل محدّث. إذا فشل الجلب، يخرج CLI (إغلاق عند الفشل). استخدمه في بيئات المؤسسات حيث يجب أن يكون تطبيق السياسة محدّثًا قبل بدء أي جلسة (v2.1.92) |
| `feedbackSurveyRate` | number | - | احتمال (0–1) ظهور استطلاع جودة الجلسة عند الأهلية. يمكن لمسؤولي المؤسسات التحكم في تكرار عرض الاستطلاع. مثال: `0.05` = 5% من الجلسات المؤهلة |

**مثال:**
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

### مجلدات الخطط والذاكرة

تخزين ملفات الخطط والذاكرة التلقائية في مواقع مخصصة.

| المفتاح | النوع | الافتراضي | الوصف |
|---------|-------|-----------|-------|
| `plansDirectory` | string | `~/.claude/plans` | المجلد الذي تُخزَّن فيه مخرجات `/plan` |
| `autoMemoryDirectory` | string | - | مجلد مخصص لتخزين الذاكرة التلقائية. يقبل مسارات مُوسَّعة بـ `~/`. غير مقبول في إعدادات المشروع (`.claude/settings.json`) لمنع إعادة توجيه كتابة الذاكرة إلى مواقع حساسة؛ مقبول من إعدادات السياسة والمحلية والمستخدم |

**مثال:**
```json
{
  "plansDirectory": "./my-plans"
}
```

**حالة الاستخدام:** مفيد لتنظيم مخرجات التخطيط بشكل منفصل عن ملفات Claude الداخلية، أو للاحتفاظ بالخطط في موقع مشترك للفريق.

### إعدادات Worktree

تهيئة كيفية إنشاء وإدارة `--worktree` لـ git worktrees. مفيد لتقليل استخدام القرص ووقت بدء التشغيل في المستودعات الأحادية الكبيرة.

| المفتاح | النوع | الافتراضي | الوصف |
|---------|-------|-----------|-------|
| `worktree.symlinkDirectories` | array | `[]` | المجلدات المطلوب ربطها رمزيًا من المستودع الرئيسي إلى كل worktree لتجنب تكرار المجلدات الكبيرة على القرص |
| `worktree.sparsePaths` | array | `[]` | المجلدات المطلوب سحبها في كل worktree عبر git sparse-checkout (وضع cone). يتم كتابة المسارات المُدرجة فقط على القرص |

**مثال:**
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

### إعدادات الإسناد

تخصيص رسائل الإسناد لـ git commits و pull requests.

| المفتاح | النوع | الافتراضي | الوصف |
|---------|-------|-----------|-------|
| `attribution.commit` | string | Co-authored-by | إسناد git commit (يدعم trailers) |
| `attribution.pr` | string | رسالة مولّدة | إسناد وصف pull request |
| `includeCoAuthoredBy` | boolean | `true` | **مُهمَل** - استخدم `attribution` بدلًا منه |

**مثال:**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**ملاحظة:** عيّنه إلى سلسلة فارغة (`""`) لإخفاء الإسناد بالكامل.

### مساعدات المصادقة

سكربتات لتوليد رموز مصادقة ديناميكية.

| المفتاح | النوع | الوصف |
|---------|-------|-------|
| `apiKeyHelper` | string | مسار سكربت Shell ينتج رمز المصادقة (يُرسل كترويسة `X-Api-Key`) |
| `forceLoginMethod` | string | تقييد تسجيل الدخول على حسابات `"claudeai"` أو `"console"` |
| `forceLoginOrgUUID` | string \| array | يتطلب أن ينتمي تسجيل الدخول إلى مؤسسة محددة. يقبل سلسلة UUID واحدة (والتي تختار تلك المؤسسة مسبقًا أثناء تسجيل الدخول) أو مصفوفة UUIDs حيث يُقبل أي مؤسسة مُدرجة بدون اختيار مسبق. عند التعيين في managed settings، يفشل تسجيل الدخول إذا لم ينتمِ الحساب المصادق عليه إلى مؤسسة مُدرجة؛ المصفوفة الفارغة تُغلق عند الفشل وتحظر تسجيل الدخول برسالة خطأ في التهيئة |

**مثال:**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

### إعلانات الشركة

عرض إعلانات مخصصة للمستخدمين عند بدء التشغيل (تُدوَّر عشوائيًا).

| المفتاح | النوع | الوصف |
|---------|-------|-------|
| `companyAnnouncements` | array | مصفوفة من السلاسل النصية تُعرض عند بدء التشغيل |

**مثال:**
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

## الصلاحيات

التحكم في الأدوات والعمليات التي يمكن لـ Claude تنفيذها.

### هيكل الصلاحيات

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

### مفاتيح الصلاحيات

| المفتاح | النوع | الوصف |
|---------|-------|-------|
| `permissions.allow` | array | قواعد تسمح باستخدام الأدوات دون طلب إذن |
| `permissions.ask` | array | قواعد تتطلب تأكيد المستخدم |
| `permissions.deny` | array | قواعد تحظر استخدام الأدوات (أعلى أولوية) |
| `permissions.additionalDirectories` | array | مجلدات إضافية يمكن لـ Claude الوصول إليها |
| `permissions.defaultMode` | string | وضع الصلاحيات الافتراضي. في البيئات البعيدة، يُحترم فقط `acceptEdits` و `plan` (v2.1.70+) |
| `permissions.disableBypassPermissionsMode` | string | منع تفعيل وضع التجاوز |
| `permissions.skipDangerousModePermissionPrompt` | boolean | تخطي رسالة التأكيد المعروضة قبل الدخول في وضع تجاوز الصلاحيات عبر `--dangerously-skip-permissions` أو `defaultMode: "bypassPermissions"`. يُتجاهل عند التعيين في إعدادات المشروع (`.claude/settings.json`) لمنع المستودعات غير الموثوقة من تجاوز الرسالة تلقائيًا |
| `allowManagedPermissionRulesOnly` | boolean | **(Managed فقط)** تُطبَّق قواعد Managed فقط؛ تُتجاهل قواعد `allow` و `ask` و `deny` الخاصة بالمستخدم/المشروع |
| `allow_remote_sessions` | boolean | **(Managed فقط)** السماح للمستخدمين ببدء جلسات Remote Control والجلسات عبر الويب. الافتراضي `true`. عيّنه إلى `false` لمنع الوصول إلى الجلسات البعيدة *(ليس في المستندات الرسمية — صفحة الصلاحيات الرسمية تنص: "الوصول إلى Remote Control والجلسات عبر الويب لا يُتحكم فيه عبر مفتاح managed settings." في خطط Team و Enterprise، يُفعّل/يُعطّل المسؤولون عبر [إعدادات مسؤول Claude Code](https://claude.ai/admin-settings/claude-code))* |
| `autoMode` | object | تخصيص ما يحظره ويسمح به مُصنّف [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode). يحتوي على `environment` (أوصاف البنية التحتية الموثوقة) و `allow` (استثناءات لقواعد الحظر) و `soft_deny` (قواعد الحظر) — جميعها مصفوفات من السلاسل النصية الوصفية. **لا يُقرأ من إعدادات المشروع المشتركة** (`.claude/settings.json`) لمنع حقن المستودعات. متاح في إعدادات المستخدم والمحلية والمُدارة. تعيين `allow` أو `soft_deny` **يستبدل** القائمة الافتراضية بالكامل لذلك القسم. شغّل `claude auto-mode defaults` لعرض القواعد المدمجة قبل التخصيص |
| `disableAutoMode` | string | عيّنه إلى `"disable"` لمنع تفعيل [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode). يزيل `auto` من دورة `Shift+Tab` ويرفض `--permission-mode auto` عند بدء التشغيل. يمكن تعيينه على أي مستوى إعدادات؛ الأكثر فائدة في managed settings حيث لا يمكن للمستخدمين تجاوزه |
| `useAutoModeDuringPlan` | boolean | ما إذا كان وضع plan يستخدم دلالات auto mode عند توفره. الافتراضي: `true`. لا يُقرأ من إعدادات المشروع المشتركة (`.claude/settings.json`). يظهر في `/config` كـ "Use auto mode during plan" |

### أوضاع الصلاحيات

| الوضع | السلوك |
|-------|--------|
| `"default"` | فحص صلاحيات قياسي مع طلبات إذن |
| `"acceptEdits"` | قبول تعديلات الملفات تلقائيًا دون سؤال |
| `"askEdits"` | السؤال قبل كل عملية *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `"dontAsk"` | رفض تلقائي للأدوات ما لم تكن معتمدة مسبقًا عبر `/permissions` أو قواعد `permissions.allow` |
| `"viewOnly"` | وضع القراءة فقط، بدون تعديلات *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `"bypassPermissions"` | تخطي جميع فحوصات الصلاحيات (خطير) |
| `"auto"` | مُصنّف خلفي يحل محل الطلبات اليدوية (`--enable-auto-mode`). معاينة بحثية — يتطلب خطة Team + Sonnet/Opus 4.6. المُصنّف يوافق تلقائيًا على القراءة فقط وتعديلات الملفات؛ يُرسل كل شيء آخر عبر فحص أمان. يتراجع إلى الطلب بعد 3 حظرات متتالية أو 20 حظرًا إجماليًا. يُهيَّأ عبر إعداد `autoMode` |
| `"plan"` | وضع استكشاف القراءة فقط |

### صياغة صلاحيات الأدوات

| الأداة | الصياغة | أمثلة |
|--------|---------|-------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`، `Bash(* install)`، `Bash(git * main)` |
| `Read` | `Read(path pattern)` | `Read(.env)`، `Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`، `Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`، `Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | بحث ويب عام |
| `Task` | `Task(agent-name)` | `Task(Explore)`، `Task(my-agent)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`، `Agent(*)` — صلاحية مقيّدة بإنشاء subagent |
| `Skill` | `Skill(skill-name)` | `Skill(weather-fetcher)` |
| `MCP` | `mcp__server__tool` أو `MCP(server:tool)` | `mcp__memory__*`، `MCP(github:*)` |

**ترتيب التقييم:** تُقيَّم القواعد بالترتيب: قواعد deny أولًا، ثم ask، ثم allow. أول قاعدة مطابقة تفوز.

**أنماط مسارات Read/Edit:** تدعم قواعد الصلاحيات لـ `Read` و `Edit` و `Write` أنماطًا بنمط gitignore مع أربعة أنواع بادئات:

| البادئة | المعنى | مثال |
|---------|--------|------|
| `//` | مسار مطلق من جذر نظام الملفات | `Read(//Users/alice/file)` |
| `~/` | نسبي لمجلد المستخدم الرئيسي | `Read(~/.zshrc)` |
| `/` | نسبي لجذر المشروع | `Edit(/src/**)` |
| `./` أو بدون بادئة | مسار نسبي (المجلد الحالي) | `Read(.env)`، `Read(*.ts)` |

**ملاحظات حول أحرف البدل في Bash:**
- `*` يمكن أن يظهر في **أي موضع**: بادئة (`Bash(* install)`)، لاحقة (`Bash(npm *)`)، أو وسط (`Bash(git * main)`)
- **حدود الكلمات:** `Bash(ls *)` (مسافة قبل `*`) يطابق `ls -la` لكن ليس `lsof`؛ `Bash(ls*)` (بدون مسافة) يطابق كليهما
- `Bash(*)` يُعامل كمكافئ لـ `Bash` (يطابق جميع أوامر bash)
- قواعد الصلاحيات تدعم إعادة توجيه المخرجات: `Bash(python:*)` يطابق `python script.py > output.txt`
- صياغة `:*` القديمة (مثل `Bash(npm:*)`) مكافئة لـ ` *` لكنها مُهملة

**مثال:**
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

تهيئة Hook (الأحداث والخصائص والمُطابقات وأكواد الخروج ومتغيرات البيئة و HTTP hooks) تُحافَظ في مستودع مخصص:

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — مرجع كامل لـ Hook مع نظام إشعارات صوتية، وجميع أحداث Hook الـ 25، و HTTP hooks، وأنماط المُطابقة، وأكواد الخروج، ومتغيرات البيئة.

مفاتيح الإعدادات المتعلقة بـ Hook (`hooks`، `disableAllHooks`، `allowManagedHooksOnly`، `allowedHttpHookUrls`، `httpHookAllowedEnvVars`) موثّقة هناك.

للاطلاع على مرجع Hooks الرسمي، راجع [وثائق Claude Code Hooks](https://code.claude.com/docs/en/hooks).

---

## خوادم MCP

تهيئة خوادم Model Context Protocol للحصول على قدرات موسّعة.

### إعدادات MCP

| المفتاح | النوع | النطاق | الوصف |
|---------|-------|--------|-------|
| `enableAllProjectMcpServers` | boolean | أي | الموافقة التلقائية على جميع خوادم `.mcp.json` |
| `enabledMcpjsonServers` | array | أي | قائمة سماح بأسماء خوادم محددة |
| `disabledMcpjsonServers` | array | أي | قائمة حظر بأسماء خوادم محددة |
| `allowedMcpServers` | array | Managed فقط | قائمة سماح مع مطابقة الاسم/الأمر/الرابط |
| `deniedMcpServers` | array | Managed فقط | قائمة حظر مع مطابقة |
| `allowManagedMcpServersOnly` | boolean | Managed فقط | السماح فقط بخوادم MCP المُدرجة صراحةً في قائمة السماح المُدارة |
| `channelsEnabled` | boolean | Managed فقط | السماح بـ [channels](https://code.claude.com/docs/en/channels) لمستخدمي Team و Enterprise. عند عدم التعيين أو `false`، يُحظر تسليم رسائل القنوات بغض النظر عن علامة `--channels` |
| `allowedChannelPlugins` | array | Managed فقط | قائمة سماح بـ plugins للقنوات المسموح لها بإرسال الرسائل. تستبدل قائمة السماح الافتراضية من Anthropic عند التعيين. غير معرّف = الرجوع إلى الافتراضي، مصفوفة فارغة = حظر جميع plugins القنوات. يتطلب `channelsEnabled: true`. كل إدخال كائن يحتوي على حقلي `marketplace` و `plugin` (v2.1.84) |

### مطابقة خوادم MCP (Managed Settings)

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

**مثال:**
```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["memory", "github", "filesystem"],
  "disabledMcpjsonServers": ["experimental-server"]
}
```

---

## Sandbox

تهيئة حماية أوامر bash في بيئة معزولة للأمان.

### إعدادات Sandbox

| المفتاح | النوع | الافتراضي | الوصف |
|---------|-------|-----------|-------|
| `sandbox.enabled` | boolean | `false` | تفعيل حماية bash في بيئة معزولة |
| `sandbox.failIfUnavailable` | boolean | `false` | الخروج مع خطأ عندما يكون sandbox مُفعّلًا لكن لا يمكن بدؤه، بدلًا من التشغيل بدون عزل. مفيد لسياسات المؤسسات التي تتطلب عزلًا صارمًا (v2.1.83) |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | الموافقة التلقائية على bash عند التشغيل في بيئة معزولة |
| `sandbox.excludedCommands` | array | `[]` | أوامر تعمل خارج البيئة المعزولة |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | السماح بـ `dangerouslyDisableSandbox`. عند التعيين إلى `false`، يُعطَّل مخرج الطوارئ بالكامل ويجب أن تعمل جميع الأوامر في بيئة معزولة (أو تكون في `excludedCommands`). مفيد لسياسات المؤسسات التي تتطلب عزلًا صارمًا |
| `sandbox.ignoreViolations` | object | `{}` | خريطة أنماط أوامر إلى مصفوفات مسارات — كتم تحذيرات الانتهاكات *(في مخطط JSON، وليس في صفحة الإعدادات الرسمية)* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | بيئة معزولة أضعف لـ Docker (يقلل الأمان) |
| `sandbox.network.allowUnixSockets` | array | `[]` | مسارات Unix socket محددة يمكن الوصول إليها في البيئة المعزولة |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | السماح بجميع Unix sockets (يتجاوز allowUnixSockets) |
| `sandbox.network.allowLocalBinding` | boolean | `false` | السماح بالربط على منافذ localhost (macOS) |
| `sandbox.network.allowedDomains` | array | `[]` | قائمة سماح بالنطاقات الشبكية للبيئة المعزولة |
| `sandbox.network.deniedDomains` | array | `[]` | قائمة حظر بالنطاقات الشبكية للبيئة المعزولة *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `sandbox.network.httpProxyPort` | number | - | منفذ HTTP proxy ‏1-65535 (proxy مخصص) |
| `sandbox.network.socksProxyPort` | number | - | منفذ SOCKS5 proxy ‏1-65535 (proxy مخصص) |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | السماح فقط بالنطاقات في قائمة السماح المُدارة (managed settings) |
| `sandbox.filesystem.allowWrite` | array | `[]` | مسارات إضافية يمكن للأوامر المعزولة الكتابة فيها. تُدمج المصفوفات عبر جميع نطاقات الإعدادات. تُدمج أيضًا مع المسارات من قواعد الصلاحيات `Edit(...)`. البادئة: `/` (مطلق)، `~/` (المجلد الرئيسي)، `./` أو بدون بادئة (نسبي للمشروع في إعدادات المشروع، نسبي لـ `~/.claude` في إعدادات المستخدم). لا تزال البادئة القديمة `//` للمسارات المطلقة تعمل. **ملاحظة:** هذا يختلف عن [صياغة صلاحيات الأدوات](#صياغة-صلاحيات-الأدوات)، التي تستخدم `//` للمطلق و `/` للنسبي للمشروع |
| `sandbox.filesystem.denyWrite` | array | `[]` | مسارات لا يمكن للأوامر المعزولة الكتابة فيها. تُدمج المصفوفات عبر جميع نطاقات الإعدادات. تُدمج أيضًا مع المسارات من قواعد الصلاحيات `Edit(...)`. نفس اتفاقيات بادئات المسارات كـ `allowWrite` |
| `sandbox.filesystem.denyRead` | array | `[]` | مسارات لا يمكن للأوامر المعزولة القراءة منها. تُدمج المصفوفات عبر جميع نطاقات الإعدادات. تُدمج أيضًا مع المسارات من قواعد الصلاحيات `Read(...)`. نفس اتفاقيات بادئات المسارات كـ `allowWrite` |
| `sandbox.filesystem.allowRead` | array | `[]` | مسارات لإعادة السماح بالوصول للقراءة ضمن مناطق `denyRead`. لها الأولوية على `denyRead`. تُدمج المصفوفات عبر جميع نطاقات الإعدادات. نفس اتفاقيات بادئات المسارات كـ `allowWrite` |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **(Managed فقط)** يُحترم فقط مسارات `allowRead` من managed settings. تُتجاهل إدخالات `allowRead` من إعدادات المستخدم والمشروع والمحلية |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | (macOS فقط) السماح بالوصول إلى ثقة TLS النظامية (`com.apple.trustd.agent`)؛ يقلل الأمان |

**مثال:**
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

## Plugins

تهيئة plugins و marketplaces الخاصة بـ Claude Code.

### إعدادات Plugin

| المفتاح | النوع | النطاق | الوصف |
|---------|-------|--------|-------|
| `enabledPlugins` | object | أي | تفعيل/تعطيل plugins محددة |
| `extraKnownMarketplaces` | object | المشروع | إضافة marketplaces مخصصة (مشاركة الفريق عبر `.claude/settings.json`) |
| `strictKnownMarketplaces` | array | Managed فقط | قائمة سماح بـ marketplaces المسموح بها |
| `skippedMarketplaces` | array | أي | Marketplaces التي رفض المستخدم تثبيتها *(في مخطط JSON، وليس في صفحة الإعدادات الرسمية)* |
| `skippedPlugins` | array | أي | Plugins التي رفض المستخدم تثبيتها *(في مخطط JSON، وليس في صفحة الإعدادات الرسمية)* |
| `pluginConfigs` | object | أي | تهيئات خادم MCP لكل plugin (مفتاحة بـ `plugin@marketplace`) *(في مخطط JSON، وليس في صفحة الإعدادات الرسمية)* |
| `blockedMarketplaces` | array | Managed فقط | حظر marketplaces محددة |
| `pluginTrustMessage` | string | Managed فقط | رسالة مخصصة تُعرض عند مطالبة المستخدمين بالوثوق بـ plugins |

**أنواع مصادر Marketplace:** `github`، `git`، `directory`، `hostPattern`، `settings`، `url`، `npm`، `file`. استخدم `source: 'settings'` لتعريف مجموعة صغيرة من plugins مضمّنة دون إعداد مستودع marketplace مُستضاف.

**مثال:**
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

## إعدادات النموذج

### الأسماء المستعارة للنماذج

| الاسم المستعار | الوصف |
|---------------|-------|
| `"default"` | الموصى به لنوع حسابك |
| `"sonnet"` | أحدث نموذج Sonnet (Claude Sonnet 4.6) |
| `"opus"` | أحدث نموذج Opus (Claude Opus 4.6) |
| `"haiku"` | نموذج Haiku السريع |
| `"sonnet[1m]"` | Sonnet مع نافذة سياق 1M token |
| `"opus[1m]"` | Opus مع نافذة سياق 1M token (الافتراضي على Max و Team و Enterprise منذ v2.1.75) |
| `"opusplan"` | Opus للتخطيط، Sonnet للتنفيذ |

**مثال:**
```json
{
  "model": "opus"
}
```

### تجاوزات النموذج

ربط معرّفات نماذج Anthropic بمعرّفات نماذج خاصة بالموفّر لنشر Bedrock أو Vertex أو Foundry.

| المفتاح | النوع | الافتراضي | الوصف |
|---------|-------|-----------|-------|
| `effortLevel` | string | - | الاحتفاظ بمستوى الجهد عبر الجلسات. يقبل `"low"` أو `"medium"` أو `"high"`. يُكتب تلقائيًا عند تشغيل `/effort low` أو `/effort medium` أو `/effort high`. مدعوم على Opus 4.6 و Sonnet 4.6 |
| `modelOverrides` | object | - | ربط إدخالات منتقي النموذج بمعرّفات خاصة بالموفّر (مثلًا ARNs لملفات تعريف استدلال Bedrock). كل مفتاح هو اسم إدخال منتقي النموذج، وكل قيمة هي معرّف نموذج الموفّر |

**مثال:**
```json
{
  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-sonnet-4-6-v1:0"
  }
}
```

### مستوى الجهد

يكشف أمر `/model` عن عنصر تحكم **مستوى الجهد** الذي يضبط مقدار التفكير الذي يطبّقه النموذج لكل استجابة. استخدم مفتاحي السهم ← → في واجهة `/model` للتبديل بين مستويات الجهد.

| مستوى الجهد | الوصف |
|------------|-------|
| عالٍ (الافتراضي) | عمق تفكير كامل، الأفضل للمهام المعقدة |
| متوسط | تفكير متوازن، جيد للمهام اليومية |
| منخفض | أدنى تفكير، أسرع الاستجابات |

**كيفية الاستخدام:**
1. شغّل `/effort low` أو `/effort medium` أو `/effort high` للتعيين مباشرةً (v2.1.76+)
2. أو شغّل `/model` ← اختر نموذجًا ← استخدم مفاتيح الأسهم **← →** للتعديل
3. يُحفظ الإعداد عبر مفتاح `effortLevel` في `settings.json`

**ملاحظة:** مستوى الجهد متاح لـ Opus 4.6 و Sonnet 4.6 على خطط Max و Team. تم تغيير الافتراضي من عالٍ إلى متوسط في v2.1.68، ثم أُعيد إلى **عالٍ** لمستخدمي API-key و Bedrock/Vertex/Foundry و Team و Enterprise في v2.1.94. اعتبارًا من v2.1.75، نافذة سياق 1M لـ Opus 4.6 متاحة افتراضيًا على خطط Max و Team و Enterprise.

### متغيرات بيئة النموذج

التهيئة عبر مفتاح `env`:

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

## العرض وتجربة المستخدم

### إعدادات العرض

| المفتاح | النوع | الافتراضي | الوصف |
|---------|-------|-----------|-------|
| `statusLine` | object | - | تهيئة شريط الحالة المخصص |
| `outputStyle` | string | `"default"` | أسلوب المخرجات (مثلًا `"Explanatory"`) |
| `spinnerTipsEnabled` | boolean | `true` | إظهار نصائح أثناء الانتظار |
| `spinnerVerbs` | object | - | أفعال spinner مخصصة مع `mode` ("append" أو "replace") ومصفوفة `verbs` |
| `spinnerTipsOverride` | object | - | نصائح spinner مخصصة مع `tips` (مصفوفة سلاسل نصية) و `excludeDefault` اختياري (boolean) |
| `respectGitignore` | boolean | `true` | احترام .gitignore في منتقي الملفات |
| `prefersReducedMotion` | boolean | `false` | تقليل الرسوم المتحركة وتأثيرات الحركة في الواجهة |
| `fileSuggestion` | object | - | أمر اقتراح ملفات مخصص (راجع تهيئة اقتراح الملفات أدناه) |

### إعدادات التهيئة العامة (`~/.claude.json`)

تُخزَّن تفضيلات العرض هذه في `~/.claude.json`، **وليس** في `settings.json`. إضافتها إلى `settings.json` ستؤدي إلى خطأ في التحقق من المخطط.

| المفتاح | النوع | الافتراضي | الوصف |
|---------|-------|-----------|-------|
| `autoConnectIde` | boolean | `false` | الاتصال تلقائيًا ببيئة تطوير قيد التشغيل عند بدء Claude Code من terminal خارجي. يظهر في `/config` كـ **Auto-connect to IDE (external terminal)** عند التشغيل خارج terminal VS Code أو JetBrains |
| `autoInstallIdeExtension` | boolean | `true` | تثبيت إضافة Claude Code IDE تلقائيًا عند التشغيل من terminal VS Code. يظهر في `/config` كـ **Auto-install IDE extension**. يمكن تعطيله أيضًا عبر متغير البيئة `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` |
| `editorMode` | string | `"normal"` | وضع اختصارات المفاتيح لموجّه الإدخال: `"normal"` أو `"vim"`. يظهر في `/config` كـ **Editor mode** |
| `showTurnDuration` | boolean | `true` | إظهار رسائل مدة الدور بعد الاستجابات (مثلًا "Cooked for 1m 6s"). عدّل `~/.claude.json` مباشرةً للتغيير |
| `terminalProgressBarEnabled` | boolean | `true` | إظهار شريط تقدم terminal في الطرفيات المدعومة (ConEmu و Ghostty 1.2.0+ و iTerm2 3.6.6+). يظهر في `/config` كـ **Terminal progress bar** |
| `teammateMode` | string | `"in-process"` | كيفية عرض زملاء [agent team](https://code.claude.com/docs/en/agent-teams): `"auto"` (يختار أجزاء مقسمة في tmux أو iTerm2، وداخل العملية في غير ذلك)، `"in-process"`، أو `"tmux"`. راجع [اختيار وضع العرض](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) |

### تهيئة شريط الحالة

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

**حقول إدخال شريط الحالة:**

يتلقى أمر شريط الحالة كائن JSON على stdin مع هذه الحقول البارزة:

| الحقل | الوصف |
|-------|-------|
| `workspace.added_dirs` | المجلدات المُضافة عبر `/add-dir` |
| `context_window.used_percentage` | نسبة استخدام نافذة السياق |
| `context_window.remaining_percentage` | النسبة المتبقية من نافذة السياق |
| `current_usage` | عدد tokens الحالي لنافذة السياق |
| `exceeds_200k_tokens` | ما إذا كان السياق يتجاوز 200k token |
| `rate_limits.five_hour.used_percentage` | نسبة استخدام حد المعدل لخمس ساعات (v2.1.80+) |
| `rate_limits.five_hour.resets_at` | طابع زمني لإعادة تعيين حد معدل الخمس ساعات |
| `rate_limits.seven_day.used_percentage` | نسبة استخدام حد المعدل لسبعة أيام |
| `rate_limits.seven_day.resets_at` | طابع زمني لإعادة تعيين حد معدل السبعة أيام |

### تهيئة اقتراح الملفات

يتلقى سكربت اقتراح الملفات كائن JSON على stdin (مثلًا `{"query": "src/comp"}`) ويجب أن ينتج ما يصل إلى 15 مسار ملف (واحد لكل سطر).

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  },
  "respectGitignore": true
}
```

**مثال:**
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

## بيانات اعتماد AWS والسحابة

### إعدادات AWS

| المفتاح | النوع | الوصف |
|---------|-------|-------|
| `awsAuthRefresh` | string | سكربت لتحديث مصادقة AWS (يُعدّل مجلد `.aws`) |
| `awsCredentialExport` | string | سكربت ينتج JSON ببيانات اعتماد AWS |

**مثال:**
```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

### OpenTelemetry

| المفتاح | النوع | الوصف |
|---------|-------|-------|
| `otelHeadersHelper` | string | سكربت لتوليد ترويسات OpenTelemetry ديناميكية |

**مثال:**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## متغيرات البيئة (عبر `env`)

تعيين متغيرات البيئة لجميع جلسات Claude Code.

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "...",
    "NODE_ENV": "development",
    "DEBUG": "true"
  }
}
```

### متغيرات البيئة الشائعة

| المتغير | الوصف |
|---------|-------|
| `ANTHROPIC_API_KEY` | مفتاح API للمصادقة |
| `ANTHROPIC_AUTH_TOKEN` | رمز OAuth |
| `CLAUDE_CODE_OAUTH_TOKEN` | رمز وصول OAuth لمصادقة Claude.ai. بديل لـ `/login` لبيئات SDK والبيئات المؤتمتة. له الأولوية على بيانات الاعتماد المخزنة في keychain |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | رمز تحديث OAuth لمصادقة Claude.ai. عند التعيين، يتبادل `claude auth login` هذا الرمز مباشرةً بدلًا من فتح المتصفح. يتطلب `CLAUDE_CODE_OAUTH_SCOPES` |
| `CLAUDE_CODE_OAUTH_SCOPES` | نطاقات OAuth مفصولة بمسافات صُدر بها رمز التحديث (مثلًا `"user:profile user:inference user:sessions:claude_code"`). مطلوب عند تعيين `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` |
| `ANTHROPIC_BASE_URL` | نقطة نهاية API مخصصة |
| `ANTHROPIC_BEDROCK_BASE_URL` | تجاوز رابط نقطة نهاية Bedrock |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | تجاوز رابط نقطة نهاية Bedrock Mantle. راجع [نقطة نهاية Mantle](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) |
| `ANTHROPIC_VERTEX_BASE_URL` | تجاوز رابط نقطة نهاية Vertex AI |
| `ANTHROPIC_BETAS` | قيم ترويسات Anthropic beta مفصولة بفواصل |
| `ANTHROPIC_VERTEX_PROJECT_ID` | معرّف مشروع GCP لـ Vertex AI |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | معرّف نموذج لإضافته كإدخال مخصص في منتقي `/model`. استخدمه لجعل نموذج غير قياسي أو خاص ببوابة قابلًا للاختيار دون استبدال الأسماء المستعارة المدمجة |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | اسم العرض لإدخال النموذج المخصص في منتقي `/model`. يعود إلى معرّف النموذج عند عدم التعيين |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | وصف العرض لإدخال النموذج المخصص في منتقي `/model`. يعود إلى `Custom model (<model-id>)` عند عدم التعيين |
| `ANTHROPIC_MODEL` | اسم النموذج المُراد استخدامه. يقبل أسماء مستعارة (`sonnet`، `opus`، `haiku`) أو معرّفات نموذج كاملة. يتجاوز إعداد `model` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | تجاوز الاسم المستعار لنموذج Haiku بمعرّف نموذج مخصص (مثلًا لنشر جهات خارجية) |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | تخصيص تسمية إدخال Haiku في منتقي `/model` عند استخدام نموذج مثبّت على Bedrock/Vertex/Foundry. يعود إلى معرّف النموذج |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | تخصيص وصف إدخال Haiku في منتقي `/model`. يعود إلى `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | تجاوز اكتشاف القدرات لنموذج Haiku مثبّت. قيم مفصولة بفواصل (مثلًا `effort,thinking`). مطلوب عندما يدعم النموذج المثبّت ميزات لا يمكن للاكتشاف التلقائي تأكيدها |
| `CLAUDECODE` | يُعيَّن إلى `1` في بيئات shell التي يُنشئها Claude Code (أداة Bash، جلسات tmux). لا يُعيَّن في hooks أو أوامر شريط الحالة. استخدمه لاكتشاف متى يعمل سكربت داخل shell Claude Code |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | عيّنه إلى `1` للسماح بالوضع السريع عند فشل فحص حالة المؤسسة بسبب خطأ شبكي. مفيد عندما يحظر proxy مؤسسي نقطة نهاية الحالة |
| `CLAUDE_CODE_USE_BEDROCK` | استخدام AWS Bedrock (`1` للتفعيل) |
| `CLAUDE_CODE_USE_VERTEX` | استخدام Google Vertex AI (`1` للتفعيل) |
| `CLAUDE_CODE_USE_FOUNDRY` | استخدام Microsoft Foundry (`1` للتفعيل) |
| `CLAUDE_CODE_USE_MANTLE` | استخدام [نقطة نهاية Mantle](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) لـ Bedrock (`1` للتفعيل) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | عيّنه إلى `1` لتفعيل أداة PowerShell على Windows (معاينة اختيارية). عند التفعيل، يمكن لـ Claude تشغيل أوامر PowerShell أصليًا بدلًا من التوجيه عبر Git Bash. مدعوم فقط على Windows الأصلي وليس WSL (v2.1.84) |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | بادئة لأسماء جلسات Remote Control المُولّدة تلقائيًا. يعود إلى اسم المضيف |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | تفعيل/تعطيل القياس عن بُعد (`0` أو `1`) |
| `DISABLE_ERROR_REPORTING` | تعطيل الإبلاغ عن الأخطاء (`1` للتعطيل) |
| `DISABLE_TELEMETRY` | تعطيل القياس عن بُعد (`1` للتعطيل) |
| `MCP_TIMEOUT` | مهلة بدء تشغيل MCP بالمللي ثانية |
| `MAX_MCP_OUTPUT_TOKENS` | الحد الأقصى لـ tokens مخرجات MCP (الافتراضي: 25000). يُعرض تحذير عندما تتجاوز المخرجات 10,000 token |
| `API_TIMEOUT_MS` | مهلة طلبات API بالمللي ثانية (الافتراضي: 600000) |
| `BASH_MAX_TIMEOUT_MS` | مهلة أوامر Bash |
| `BASH_MAX_OUTPUT_LENGTH` | الحد الأقصى لطول مخرجات bash |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | نسبة عتبة الضغط التلقائي (1-100). الافتراضي ~95%. عيّنه أقل (مثلًا `50`) لتفعيل الضغط مبكرًا. القيم أعلى من 95% ليس لها تأثير. استخدم `/context` لمراقبة الاستخدام الحالي. مثال: `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | الحفاظ على مجلد العمل الحالي بين استدعاءات bash (`1` للتفعيل) |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | تعطيل المهام الخلفية (`1` للتعطيل) |
| `ENABLE_TOOL_SEARCH` | عتبة بحث أدوات MCP (مثلًا `auto:5`) |
| `DISABLE_PROMPT_CACHING` | تعطيل جميع التخزين المؤقت للأوامر (`1` للتعطيل) |
| `DISABLE_PROMPT_CACHING_HAIKU` | تعطيل التخزين المؤقت لـ Haiku |
| `DISABLE_PROMPT_CACHING_SONNET` | تعطيل التخزين المؤقت لـ Sonnet |
| `DISABLE_PROMPT_CACHING_OPUS` | تعطيل التخزين المؤقت لـ Opus |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | طلب TTL للتخزين المؤقت لمدة ساعة على Bedrock (`1` للتفعيل) |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | تعطيل الميزات التجريبية (`1` للتعطيل) |
| `CLAUDE_CODE_SHELL` | تجاوز اكتشاف shell التلقائي |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | تجاوز حد tokens الافتراضي لقراءة الملفات |
| `CLAUDE_CODE_GLOB_HIDDEN` | عيّنه إلى `false` لاستبعاد ملفات dotfiles من النتائج عندما يستدعي Claude أداة Glob. مُضمّنة افتراضيًا. لا يؤثر على الإكمال التلقائي `@` أو `ls` أو Grep أو Read |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | عيّنه إلى `false` لجعل أداة Glob تحترم أنماط `.gitignore`. افتراضيًا، يُرجع Glob جميع الملفات المطابقة بما في ذلك المُتجاهلة في gitignore. لا يؤثر على الإكمال التلقائي `@`، الذي له إعداد `respectGitignore` خاص به |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | مهلة اكتشاف ملفات Glob بالثواني |
| `CLAUDE_CODE_ENABLE_TASKS` | عيّنه إلى `true` لتفعيل تتبع المهام في الوضع غير التفاعلي (علامة `-p`). المهام مُفعّلة افتراضيًا في الوضع التفاعلي |
| `CLAUDE_CODE_SIMPLE` | عيّنه إلى `1` للتشغيل مع system prompt مُبسّط وأدوات Bash وقراءة الملفات وتحرير الملفات فقط |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | الخروج التلقائي من وضع SDK بعد فترة خمول (مللي ثانية) |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | تعطيل التفكير التكيّفي (`1` للتعطيل) |
| `CLAUDE_CODE_DISABLE_THINKING` | فرض تعطيل التفكير الموسّع (`1` للتعطيل) |
| `DISABLE_INTERLEAVED_THINKING` | منع إرسال ترويسة beta التفكير المتداخل (`1` للتعطيل) |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | تعطيل نافذة سياق 1M token (`1` للتعطيل) |
| `CLAUDE_CODE_ACCOUNT_UUID` | تجاوز UUID الحساب للمصادقة |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | تعطيل تعليمات git في system prompt |
| `CLAUDE_CODE_NEW_INIT` | عيّنه إلى `true` لجعل `/init` يُشغّل مسار إعداد تفاعلي. يسأل عن الملفات المطلوب إنشاؤها (CLAUDE.md، skills، hooks) قبل استكشاف قاعدة الكود. بدون هذا، ينشئ `/init` ملف CLAUDE.md تلقائيًا |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | مسار إلى مجلد أو أكثر من مجلدات بذور plugin للقراءة فقط، مفصولة بـ `:` على Unix أو `;` على Windows. قم بتضمين plugins مُعبّأة مسبقًا في صورة حاوية. يُسجّل Claude Code الـ marketplaces من هذه المجلدات عند بدء التشغيل ويستخدم plugins مُخزّنة مؤقتًا مسبقًا بدون إعادة الاستنساخ |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | تفعيل خوادم MCP الخاصة بـ Claude.ai |
| `CLAUDE_CODE_EFFORT_LEVEL` | تعيين مستوى الجهد: `low` أو `medium` أو `high` أو `max` (Opus 4.6 فقط) أو `auto` (استخدام افتراضي النموذج). له الأولوية على `/effort` وإعداد `effortLevel` |
| `CLAUDE_CODE_MAX_TURNS` | الحد الأقصى لدورات Agent قبل التوقف *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | مكافئ لتعيين `DISABLE_AUTOUPDATER` و `DISABLE_FEEDBACK_COMMAND` و `DISABLE_ERROR_REPORTING` و `DISABLE_TELEMETRY` |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | تخطي مسار إعداد الإعدادات الأولى *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | تجاوز سلوك التخزين المؤقت للأوامر *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `CLAUDE_CODE_DISABLE_TOOLS` | قائمة أدوات مفصولة بفواصل للتعطيل *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `CLAUDE_CODE_DISABLE_MCP` | تعطيل جميع خوادم MCP (`1` للتعطيل) *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | الحد الأقصى لـ tokens المخرجات لكل استجابة. الافتراضي: 32,000 (64,000 لـ Opus 4.6 اعتبارًا من v2.1.77). الحد الأعلى: 64,000 (128,000 لـ Opus 4.6 و Sonnet 4.6 اعتبارًا من v2.1.77) |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | تعطيل الوضع السريع بالكامل (`1` للتعطيل) |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | عيّنه إلى `1` لتعطيل الرجوع إلى عدم البث عند فشل طلب بث أثناء التنفيذ. أخطاء البث تُنشر إلى طبقة إعادة المحاولة بدلًا من ذلك. مفيد عندما يتسبب proxy أو بوابة في تنفيذ أداة مكرر عبر الرجوع (v2.1.83) |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | إيقاف البث المتوقف (`1` للتفعيل) |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | تفعيل البث الدقيق للأدوات (`1` للتفعيل) |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | تعطيل الذاكرة التلقائية (`1` للتعطيل) |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | تعطيل نقاط فحص الملفات لـ `/rewind` (`1` للتعطيل) |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | تعطيل معالجة المرفقات (`1` للتعطيل) |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | منع تحميل ملفات CLAUDE.md (`1` للتعطيل) |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | استئناف تلقائي إذا انتهت الجلسة السابقة في منتصف الدور (`1` للتفعيل) |
| `CLAUDE_CODE_USER_EMAIL` | توفير بريد المستخدم بشكل متزامن للمصادقة |
| `CLAUDE_CODE_ORGANIZATION_UUID` | توفير UUID المؤسسة بشكل متزامن للمصادقة |
| `CLAUDE_CONFIG_DIR` | مجلد تهيئة مخصص (يتجاوز الافتراضي `~/.claude`) |
| `CLAUDE_CODE_TMPDIR` | تجاوز مجلد الملفات المؤقتة المُستخدم للملفات المؤقتة الداخلية. يُلحق Claude Code `/claude/` بهذا المسار. الافتراضي: `/tmp` على Unix/macOS، `os.tmpdir()` على Windows |
| `ANTHROPIC_CUSTOM_HEADERS` | ترويسات مخصصة لطلبات API (تنسيق `Name: Value`، مفصولة بأسطر جديدة للترويسات المتعددة) |
| `ANTHROPIC_FOUNDRY_API_KEY` | مفتاح API لمصادقة Microsoft Foundry |
| `ANTHROPIC_FOUNDRY_BASE_URL` | رابط القاعدة لمورد Foundry |
| `ANTHROPIC_FOUNDRY_RESOURCE` | اسم مورد Foundry |
| `AWS_BEARER_TOKEN_BEDROCK` | مفتاح API لمصادقة Bedrock |
| `ANTHROPIC_SMALL_FAST_MODEL` | **مُهمَل** — استخدم `ANTHROPIC_DEFAULT_HAIKU_MODEL` بدلًا منه |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | منطقة AWS لتجاوز نموذج فئة Haiku المُهمل |
| `CLAUDE_CODE_SHELL_PREFIX` | بادئة أمر تُلحق قبل أوامر bash |
| `BASH_DEFAULT_TIMEOUT_MS` | مهلة أوامر bash الافتراضية بالمللي ثانية |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | تخطي مصادقة AWS لـ Bedrock (`1` للتخطي) |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | تخطي مصادقة Azure لـ Foundry (`1` للتخطي) |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | تخطي مصادقة AWS لـ Bedrock Mantle (مثلًا عند استخدام بوابة LLM) |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | تخطي مصادقة Google لـ Vertex (`1` للتخطي) |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | السماح لـ proxy بتنفيذ تحليل DNS |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | فترة تحديث بيانات الاعتماد بالمللي ثانية لـ `apiKeyHelper` |
| `CLAUDE_CODE_CLIENT_CERT` | مسار شهادة العميل لـ mTLS |
| `CLAUDE_CODE_CLIENT_KEY` | مسار المفتاح الخاص للعميل لـ mTLS |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | عبارة مرور للمفتاح المشفّر في mTLS |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | مهلة استنساخ git لـ marketplace في مللي ثانية (الافتراضي: 120000) |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | تجاوز مجلد plugins الجذري |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | تخطي الإضافة التلقائية لـ marketplace الرسمي (`1` للتعطيل) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | انتظار اكتمال تثبيت plugin قبل أول استعلام (`1` للتفعيل) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | مهلة التثبيت المتزامن لـ plugin بالمللي ثانية |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | عيّنه إلى `1` للاحتفاظ بذاكرة marketplace المؤقتة الموجودة عند فشل `git pull` بدلًا من المسح وإعادة الاستنساخ. مفيد في البيئات المعزولة أو بدون اتصال حيث ستفشل إعادة الاستنساخ بنفس الطريقة |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | إخفاء معلومات البريد/المؤسسة من الواجهة *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `CLAUDE_CODE_DISABLE_CRON` | تعطيل المهام المجدولة/cron (`1` للتعطيل) |
| `DISABLE_INSTALLATION_CHECKS` | تعطيل تحذيرات التثبيت |
| `DISABLE_FEEDBACK_COMMAND` | تعطيل أمر `/feedback`. يُقبل أيضًا الاسم القديم `DISABLE_BUG_COMMAND` |
| `DISABLE_DOCTOR_COMMAND` | إخفاء أمر `/doctor` (`1` للتعطيل) |
| `DISABLE_LOGIN_COMMAND` | إخفاء أمر `/login` (`1` للتعطيل) |
| `DISABLE_LOGOUT_COMMAND` | إخفاء أمر `/logout` (`1` للتعطيل) |
| `DISABLE_UPGRADE_COMMAND` | إخفاء أمر `/upgrade` (`1` للتعطيل) |
| `DISABLE_EXTRA_USAGE_COMMAND` | إخفاء أمر `/extra-usage` (`1` للتعطيل) |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | إخفاء أمر `/install-github-app` (`1` للتعطيل) |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | تعطيل النصوص التزيينية واستدعاءات النموذج غير الأساسية *(ليس في المستندات الرسمية — غير مُتحقق منه)* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | تجاوز مسار مجلد ملفات سجلات التصحيح |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | الحد الأدنى لمستوى سجل التصحيح |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | فرض تشغيل المهام الطويلة في الخلفية تلقائيًا (`1` للتفعيل) |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | منع إعادة ربط Opus 4.0/4.1 بنماذج أحدث (`1` للتعطيل) |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | تفعيل النموذج الاحتياطي لجميع النماذج الأساسية وليس الافتراضي فقط (`1` للتفعيل) |
| `CLAUDE_CODE_GIT_BASH_PATH` | مسار ملف Git Bash التنفيذي على Windows (عند بدء التشغيل فقط) |
| `DISABLE_COST_WARNINGS` | تعطيل رسائل تحذير التكلفة |
| `CLAUDE_CODE_SUBAGENT_MODEL` | تجاوز النموذج لـ subagents (مثلًا `haiku`، `sonnet`) |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | عيّنه إلى `1` لإزالة بيانات اعتماد Anthropic وموفّري السحابة من بيئات العمليات الفرعية (أداة Bash، hooks، خوادم MCP stdio). استخدمه للدفاع المتعمّق عندما لا ينبغي للعمليات الفرعية أن ترث مفاتيح API (v2.1.83) |
| `CLAUDE_CODE_MAX_RETRIES` | تجاوز عدد إعادة محاولة طلبات API (الافتراضي: 10) |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | الحد الأقصى للأدوات المتوازية للقراءة فقط (الافتراضي: 10) |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | تعطيل أنواع subagent المدمجة في وضع SDK (`1` للتعطيل) |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | تخطي بادئة `mcp__<server>__` لأدوات MCP في وضع SDK (`1` للتفعيل) |
| `MCP_CONNECTION_NONBLOCKING` | عيّنه إلى `true` في وضع `-p` لتخطي انتظار اتصال MCP بالكامل. يحدّ اتصالات خوادم `--mcp-config` عند 5 ثوانٍ بدلًا من الحظر على أبطأ خادم *(في سجل تغييرات v2.1.89، ليس بعد على صفحة env-vars الرسمية)* |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | مهلة SessionEnd hook بالمللي ثانية (يستبدل الحد الثابت 1.5 ثانية) |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | تعطيل مطالبات استطلاع الملاحظات (`1` للتعطيل) |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | تعطيل تحديثات عنوان terminal (`1` للتعطيل) |
| `CLAUDE_CODE_NO_FLICKER` | عيّنه إلى `1` لتفعيل العرض بدون وميض على الشاشة البديلة. يزيل الوميض البصري أثناء إعادة رسم الشاشة الكاملة (v2.1.88) |
| `CLAUDE_CODE_SCROLL_SPEED` | مُضاعف تمرير عجلة الفأرة للعرض بملء الشاشة. زِد للتمرير الأسرع، قلّل للتحكم الأدق |
| `CLAUDE_CODE_DISABLE_MOUSE` | عيّنه إلى `1` لتعطيل تتبع الفأرة في العرض بملء الشاشة. مفيد عندما تتداخل أحداث الفأرة مع مُضاعفات terminal أو أدوات إمكانية الوصول |
| `CLAUDE_CODE_ACCESSIBILITY` | عيّنه إلى `1` للاحتفاظ بمؤشر terminal الأصلي مرئيًا لقارئات الشاشة وأدوات إمكانية الوصول |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | عيّنه إلى `0` لتعطيل تمييز بناء الجملة في مخرجات diff |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | تخطي التثبيت التلقائي لإضافة IDE (`1` للتخطي) |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | تجاوز سلوك الاتصال التلقائي بـ IDE |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | تجاوز عنوان مضيف IDE للاتصال |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | تخطي التحقق من ملف قفل IDE (`1` للتخطي) |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | فترة تأخير بالمللي ثانية لسكربت مساعد ترويسات OTel |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | مهلة تفريغ OpenTelemetry بالمللي ثانية |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | مهلة إيقاف OpenTelemetry بالمللي ثانية |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | مهلة بالمللي ثانية قبل أن يُغلق مراقب خمول البث اتصالًا متوقفًا. الافتراضي: `90000` (90 ثانية). زِد إذا تسببت الأدوات طويلة التشغيل أو الشبكات البطيئة في أخطاء مهلة مبكرة |
| `OTEL_LOG_TOOL_DETAILS` | عيّنه إلى `1` لتضمين `tool_parameters` في أحداث OpenTelemetry. محذوف افتراضيًا للخصوصية *(في سجل تغييرات v2.1.85، ليس بعد على صفحة env-vars الرسمية)* |
| `CLAUDE_CODE_MCP_SERVER_NAME` | اسم خادم MCP، يُمرّر كمتغير بيئة لسكربتات `headersHelper` لتتمكن من إنشاء ترويسات مصادقة خاصة بالخادم *(في سجل تغييرات v2.1.85، ليس بعد على صفحة env-vars الرسمية)* |
| `CLAUDE_CODE_MCP_SERVER_URL` | رابط خادم MCP، يُمرّر كمتغير بيئة لسكربتات `headersHelper` مع `CLAUDE_CODE_MCP_SERVER_NAME` *(في سجل تغييرات v2.1.85، ليس بعد على صفحة env-vars الرسمية)* |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | تجاوز الاسم المستعار لنموذج Opus (مثلًا `claude-opus-4-6[1m]`) |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | تخصيص تسمية إدخال Opus في منتقي `/model` عند استخدام نموذج مثبّت على Bedrock/Vertex/Foundry. يعود إلى معرّف النموذج |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | تخصيص وصف إدخال Opus في منتقي `/model`. يعود إلى `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | تجاوز اكتشاف القدرات لنموذج Opus مثبّت. قيم مفصولة بفواصل (مثلًا `effort,thinking`). مطلوب عندما يدعم النموذج المثبّت ميزات لا يمكن للاكتشاف التلقائي تأكيدها |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | تجاوز الاسم المستعار لنموذج Sonnet (مثلًا `claude-sonnet-4-6`) |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | تخصيص تسمية إدخال Sonnet في منتقي `/model` عند استخدام نموذج مثبّت على Bedrock/Vertex/Foundry. يعود إلى معرّف النموذج |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | تخصيص وصف إدخال Sonnet في منتقي `/model`. يعود إلى `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | تجاوز اكتشاف القدرات لنموذج Sonnet مثبّت. قيم مفصولة بفواصل (مثلًا `effort,thinking`). مطلوب عندما يدعم النموذج المثبّت ميزات لا يمكن للاكتشاف التلقائي تأكيدها |
| `MAX_THINKING_TOKENS` | الحد الأقصى لـ tokens التفكير الموسّع لكل استجابة |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | تعيين سعة السياق بالـ tokens المُستخدمة لحسابات الضغط التلقائي. يعود إلى نافذة سياق النموذج (200K قياسي، 1M لنماذج السياق الممتد). استخدم قيمة أقل (مثلًا `500000`) على نموذج 1M لمعاملته كـ 500K للضغط. محدود بنافذة السياق الفعلية. يُطبَّق `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` كنسبة مئوية من هذه القيمة. تعيين هذا يفصل عتبة الضغط عن `used_percentage` في شريط الحالة |
| `DISABLE_AUTO_COMPACT` | تعطيل الضغط التلقائي للسياق (`1` للتعطيل). `/compact` اليدوي لا يزال يعمل |
| `DISABLE_COMPACT` | تعطيل جميع عمليات الضغط — التلقائية واليدوية (`1` للتعطيل) |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | تفعيل اقتراحات الأوامر |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | يتطلب وضع plan للجلسات |
| `CLAUDE_CODE_TEAM_NAME` | اسم الفريق لفرق agent |
| `CLAUDE_CODE_TASK_LIST_ID` | معرّف قائمة المهام لتكامل المهام |
| `CLAUDE_ENV_FILE` | مسار ملف بيئة مخصص |
| `FORCE_AUTOUPDATE_PLUGINS` | فرض التحديث التلقائي لـ plugins (`1` للتفعيل) |
| `HTTP_PROXY` | رابط HTTP proxy لطلبات الشبكة |
| `HTTPS_PROXY` | رابط HTTPS proxy لطلبات الشبكة |
| `NO_PROXY` | قائمة مضيفات مفصولة بفواصل تتجاوز proxy |
| `MCP_TOOL_TIMEOUT` | مهلة تنفيذ أداة MCP بالمللي ثانية |
| `MCP_CLIENT_SECRET` | سر عميل MCP OAuth |
| `MCP_OAUTH_CALLBACK_PORT` | منفذ callback لـ MCP OAuth |
| `IS_DEMO` | تفعيل وضع العرض التوضيحي |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | ميزانية الأحرف لمخرجات أداة أوامر slash |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | تجاوز منطقة Vertex AI لـ Claude 3.5 Haiku |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | تجاوز منطقة Vertex AI لـ Claude 3.7 Sonnet |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | تجاوز منطقة Vertex AI لـ Claude 4.0 Opus |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | تجاوز منطقة Vertex AI لـ Claude 4.0 Sonnet |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | تجاوز منطقة Vertex AI لـ Claude 4.1 Opus |

---

## أوامر مفيدة

| الأمر | الوصف |
|-------|-------|
| `/model` | تبديل النماذج وضبط مستوى جهد Opus 4.6 |
| `/effort` | تعيين مستوى الجهد مباشرةً: `low`، `medium`، `high` (v2.1.76+) |
| `/config` | واجهة تهيئة تفاعلية |
| `/memory` | عرض/تحرير جميع ملفات الذاكرة |
| `/agents` | إدارة subagents |
| `/mcp` | إدارة خوادم MCP |
| `/hooks` | عرض hooks المُهيّأة |
| `/plugin` | إدارة plugins |
| `/keybindings` | تهيئة اختصارات لوحة المفاتيح المخصصة |
| `/skills` | عرض وإدارة skills |
| `/permissions` | عرض وإدارة قواعد الصلاحيات |
| `--doctor` | تشخيص مشكلات التهيئة |
| `--debug` | وضع التصحيح مع تفاصيل تنفيذ hook |

---

## مرجع سريع: مثال كامل

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

## المصادر

- [وثائق إعدادات Claude Code](https://code.claude.com/docs/en/settings)
- [مخطط JSON لإعدادات Claude Code](https://json.schemastore.org/claude-code-settings.json)
- [سجل تغييرات Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [أمثلة إعدادات Claude Code على GitHub](https://github.com/feiskyer/claude-code-settings)
- [Shipyard - ورقة غش Claude Code CLI](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [مرجع متغيرات بيئة Claude Code](https://code.claude.com/docs/en/env-vars)
- [مرجع صلاحيات Claude Code](https://code.claude.com/docs/en/permissions)

</div>
