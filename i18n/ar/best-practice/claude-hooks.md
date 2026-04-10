<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-hooks.md) | [中文](../../zh/best-practice/claude-hooks.md) | [日本語](../../ja/best-practice/claude-hooks.md) | [Français](../../fr/best-practice/claude-hooks.md) | [Русский](../../ru/best-practice/claude-hooks.md) | **العربية**

# أفضل ممارسات Hooks

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Hooks في Claude Code — معالجات مدفوعة بالأحداث تعمل خارج الحلقة الوكيلية على أحداث محددة في دورة الحياة.

<table width="100%">
<tr>
<td><a href="../../ar/README.md">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## نظرة عامة

Hooks هي معالجات يحددها المستخدم وتُنفَّذ **خارج** الحلقة الوكيلية — فهي ليست جزءاً من استدلال Claude أو سلسلة استدعاء الأدوات. تنطلق عند أحداث محددة في دورة الحياة ويمكنها تشغيل سكريبتات، أو حقن prompts، أو إنشاء agents، أو استدعاء نقاط نهاية HTTP. استخدم hooks لإضافة المراقبة، وتطبيق السياسات، وأتمتة التأثيرات الجانبية، والتحكم في القرارات دون تلويث سياق المحادثة.

---

## أحداث Hook (27)

### دورة حياة الجلسة

| الحدث | ينطلق عندما |
|-------|------------|
| `SessionStart` | تبدأ جلسة تفاعلية جديدة |
| `SessionEnd` | الجلسة على وشك الإغلاق |
| `Setup` | أول تشغيل لـ Claude Code في مشروع (أو بعد `/init`) |

### دورة حياة الأدوات

| الحدث | ينطلق عندما |
|-------|------------|
| `PreToolUse` | قبل تنفيذ استدعاء أداة. يمكنه حظر الاستدعاء أو السماح به أو تعديله |
| `PostToolUse` | بعد نجاح استدعاء أداة |
| `PostToolUseFailure` | بعد فشل استدعاء أداة مع خطأ |
| `PermissionRequest` | عندما يطلب Claude إذناً لأداة لم يوافق عليها المستخدم مسبقاً |
| `PermissionDenied` | عندما يُرفض طلب إذن أداة من قبل المستخدم أو سياسة |

### دورة حياة الـ Agents

| الحدث | ينطلق عندما |
|-------|------------|
| `SubagentStart` | يبدأ subagent (عبر أداة `Agent(...)`) في التنفيذ |
| `SubagentStop` | ينتهي subagent من التنفيذ |
| `Stop` | ينهي الـ Agent **الرئيسي** دور استجابته |
| `StopFailure` | يفشل دور استجابة الـ Agent الرئيسي مع خطأ |

### المهام / الفرق

| الحدث | ينطلق عندما |
|-------|------------|
| `TeammateIdle` | يصبح زميل في جلسة Agent Teams خاملاً ومتاحاً |
| `TaskCreated` | تُنشأ مهمة في الخلفية |
| `TaskCompleted` | تنتهي مهمة في الخلفية |

### السياق

| الحدث | ينطلق عندما |
|-------|------------|
| `PreCompact` | قبل تشغيل ضغط السياق |
| `PostCompact` | بعد اكتمال ضغط السياق |
| `UserPromptSubmit` | يُرسل المستخدم prompt (قبل أن يعالجه Claude) |
| `Notification` | يُرسل Claude إشعاراً (مثلاً، اكتملت المهمة، مطلوب إذن) |

### التكوين / البيئة

| الحدث | ينطلق عندما |
|-------|------------|
| `ConfigChange` | يُعدَّل ملف إعدادات |
| `CwdChanged` | يتغير مجلد العمل (مثلاً عبر `/add-dir`) |
| `FileChanged` | يُعدَّل ملف مُراقَب على القرص |
| `InstructionsLoaded` | تُحمَّل ملفات CLAUDE.md أو rules في السياق |

### Worktree

| الحدث | ينطلق عندما |
|-------|------------|
| `WorktreeCreate` | يُنشأ git worktree لـ agent معزول |
| `WorktreeRemove` | يُنظَّف git worktree |

### MCP / Elicitation

| الحدث | ينطلق عندما |
|-------|------------|
| `Elicitation` | يحتاج Claude إلى إدخال منظّم من المستخدم عبر MCP elicitation |
| `ElicitationResult` | يستجيب المستخدم لـ MCP elicitation |

---

## أنواع Hook (4)

| النوع | الصيغة | الوصف |
|-------|--------|-------|
| `command` | `{"type": "command", "command": "python3 script.py"}` | يشغّل أمر shell. يستقبل بيانات الحدث عبر متغيرات البيئة. يُحقن stdout في السياق (لـ `PreToolUse`/`Stop`). كود خروج غير صفري يحظر الإجراء (لـ `PreToolUse`) |
| `prompt` | `{"type": "prompt", "prompt": "راجع هذا بحثاً عن مشاكل أمنية"}` | يحقن prompt في سياق Claude عند نقطة الـ hook. مفيد لإضافة تعليمات أو قيود |
| `agent` | `{"type": "agent", "agent": "security-reviewer"}` | يُنشئ subagent لمعالجة الحدث. يعمل الـ Agent في سياقه الخاص ويمكنه استخدام الأدوات |
| `http` | `{"type": "http", "url": "https://example.com/webhook"}` | يرسل بيانات الحدث إلى نقطة نهاية HTTP عبر POST. يُحقن جسم الاستجابة في السياق |

---

## تسلسل التكوين الهرمي

يمكن تعريف Hooks على مستويات متعددة. المستويات الأعلى تتجاوز المستويات الأدنى لنفس الحدث:

| المستوى | الموقع | النطاق |
|---------|--------|--------|
| **مُدار** | `managed-settings.json` (MDM / Registry) | مفروض من المؤسسة، لا يمكن تجاوزه |
| **إعدادات محلية** | `.claude/settings.local.json` | تجاوزات شخصية للمشروع (مُتجاهل بـ git) |
| **إعدادات مشتركة** | `.claude/settings.json` | hooks المشروع المشتركة للفريق |
| **إعدادات عامة** | `~/.claude/settings.json` | إعدادات شخصية افتراضية لجميع المشاريع |
| **Frontmatter الـ agent** | `.claude/agents/<name>.md` حقل `hooks:` | مقيّد بـ subagent محدد |
| **Frontmatter الـ skill** | `.claude/skills/<name>/SKILL.md` حقل `hooks:` | مقيّد بـ skill محدد |

لتعطيل جميع hooks، اضبط `"disableAllHooks": true` في `.claude/settings.local.json`.

---

## صيغة Matcher

تُصفّي Matchers أي استدعاءات أدوات تُطلق hook. تنطبق على أحداث `PreToolUse` و `PostToolUse` و `PostToolUseFailure`.

| Matcher | يطابق |
|---------|-------|
| `"Bash"` | جميع استدعاءات أداة Bash |
| `"Read"` | جميع استدعاءات أداة Read |
| `"Write"` | جميع استدعاءات أداة Write |
| `"Edit"` | جميع استدعاءات أداة Edit |
| `"Glob"` | جميع استدعاءات أداة Glob |
| `"Grep"` | جميع استدعاءات أداة Grep |
| `"Agent"` | جميع استدعاءات أداة Agent (subagent) |
| `"Skill"` | جميع استدعاءات أداة Skill |
| `"WebFetch"` | جميع استدعاءات أداة WebFetch |
| `"WebSearch"` | جميع استدعاءات أداة WebSearch |
| `"mcp__<server>__<tool>"` | أداة MCP محددة على خادم محدد |
| `"*"` | جميع استدعاءات الأدوات (wildcard) |

يمكن دمج عدة matchers في مصفوفة: `["Bash", "Write", "Edit"]`.

---

## متغيرات البيئة

تستقبل hooks من نوع command بيانات الحدث عبر متغيرات البيئة:

| المتغير | متاح في | الوصف |
|---------|---------|-------|
| `CLAUDE_HOOK_EVENT` | الكل | اسم الحدث (مثلاً `PreToolUse`، `Stop`) |
| `CLAUDE_HOOK_TOOL_NAME` | أحداث الأدوات | الأداة المُستدعاة (مثلاً `Bash`، `Write`) |
| `CLAUDE_HOOK_TOOL_INPUT` | `PreToolUse` | سلسلة JSON لمعاملات إدخال الأداة |
| `CLAUDE_HOOK_TOOL_OUTPUT` | `PostToolUse` | سلسلة JSON لمخرجات الأداة |
| `CLAUDE_HOOK_ERROR` | أحداث الفشل | رسالة خطأ العملية الفاشلة |
| `CLAUDE_HOOK_SESSION_ID` | الكل | معرّف الجلسة الحالية |
| `CLAUDE_HOOK_CWD` | الكل | مجلد العمل الحالي |
| `CLAUDE_HOOK_PROJECT_DIR` | الكل | المجلد الجذري للمشروع |
| `CLAUDE_HOOK_AGENT_NAME` | أحداث الـ Agents | اسم الـ subagent المعني |
| `CLAUDE_HOOK_MATCHED` | أحداث الأدوات | الـ matcher الذي أطلق هذا الـ hook |

---

## أمثلة عملية

### إشعار صوتي عند اكتمال المهمة

```json
{
  "hooks": {
    "Stop": [
      {
        "type": "command",
        "command": "afplay ~/.claude/hooks/sounds/done.mp3",
        "async": true
      }
    ]
  }
}
```

### تسجيل جميع استدعاءات الأدوات

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "type": "command",
        "command": "echo \"$(date -u +%Y-%m-%dT%H:%M:%SZ) $CLAUDE_HOOK_TOOL_NAME\" >> ~/.claude/logs/tools.log",
        "async": true
      }
    ]
  }
}
```

### تدقيق أمني — حظر الأوامر الخطرة

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "command",
        "command": "python3 .claude/hooks/scripts/security-check.py"
      }
    ]
  }
}
```

يفحص السكريبت `CLAUDE_HOOK_TOOL_INPUT` بحثاً عن أنماط خطرة (`rm -rf /`، `curl | bash`، `chmod 777`). كود خروج غير صفري يحظر استدعاء الأداة.

### تنسيق تلقائي عند كتابة الملفات

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ["Write", "Edit"],
        "type": "command",
        "command": "prettier --write \"$CLAUDE_HOOK_TOOL_INPUT\" 2>/dev/null || true",
        "async": true
      }
    ]
  }
}
```

---

## أنماط التحكم في القرارات

يمكن لـ hooks من نوع `PreToolUse` إرجاع إشارات قرار عبر stdout أو كود الخروج:

| النمط | الآلية | التأثير |
|-------|--------|---------|
| **Allow** | كود خروج `0`، stdout يحتوي `ALLOW` | يستمر استدعاء الأداة بدون طلب إذن |
| **Deny** | كود خروج غير صفري | يُحظر استدعاء الأداة، تُعرض رسالة خطأ |
| **Ask** | كود خروج `0`، stdout يحتوي `ASK` | يُعرض طلب الإذن القياسي للمستخدم |
| **Defer** | كود خروج `0`، بدون مخرجات أو stdout فارغ | يُمرَّر إلى معالجة الأذونات الافتراضية |

بالنسبة لـ hooks من نوع `prompt` و `agent`، فإن الاستجابة المحقونة توجّه سلوك Claude بدلاً من التحكم المباشر في التنفيذ.

---

## أفضل الممارسات

1. **استخدم `async: true` للتأثيرات الجانبية** — الإشعارات الصوتية والتسجيل والقياس عن بُعد لا ينبغي أن تحظر الحلقة الوكيلية أبداً. اضبط `"async": true` على hooks التي لا تحتاج للتأثير على سلوك Claude.

2. **استخدم `once: true` لـ hooks الجلسة** — hooks من نوع `SessionStart` التي تنفّذ إعداداً (تثبيت التبعيات، فحص البيئة) يجب أن تعمل مرة واحدة فقط لكل جلسة.

3. **أبقِ المهلات قصيرة** — hooks المتزامنة تحظر استجابة Claude. إذا استغرق hook الخاص بك أكثر من 2-3 ثوانٍ، اجعله غير متزامن أو حسّن السكريبت.

4. **حدِّد نطاق hooks باستخدام matchers** — تجنّب تشغيل hooks على كل استدعاء أداة عندما تحتاج فقط لأدوات محددة. استخدم matchers لتقليل العبء.

5. **استخدم `command` للتحكم، و `prompt` للتوجيه** — عندما تحتاج تحكماً صارماً allow/deny، استخدم hook من نوع command مع أكواد الخروج. عندما تريد التأثير على نهج Claude، استخدم hook من نوع prompt.

6. **اختبر hooks بشكل منعزل** — شغّل سكريبتات hooks يدوياً مع متغيرات بيئة نموذجية قبل إضافتها إلى الإعدادات.

7. **سجّل إخفاقات hooks** — غلّف أوامر hooks بمنطق معالجة أخطاء يكتب في ملف سجل، حتى تكون الإخفاقات الصامتة قابلة للاكتشاف.

---

## الأخطاء الشائعة

| الخطأ | التفاصيل |
|-------|----------|
| **الخلط بين `Stop` و `SubagentStop`** | `Stop` ينطلق للـ Agent **الرئيسي** فقط. استخدم `SubagentStop` لأحداث اكتمال الـ subagent. في الإصدارات السابقة، كان `Stop` يُسمى `AgentStop` — تمت إعادة تسميته |
| **عبء إنشاء العمليات** | كل hook من نوع `command` يُنشئ عملية shell جديدة. للأحداث عالية التردد مثل `PostToolUse` مع matcher من نوع wildcard، هذا يضيف تأخيراً. فكّر في استخدام `async: true` أو دمج المنطق في سكريبت واحد |
| **نسيان الـ matcher** | hook من نوع `PreToolUse` بدون حقل `matcher` ينطلق لـ **جميع** استدعاءات الأدوات. اضبط دائماً matcher إلا إذا كنت تريد تغطية شاملة عمداً |
| **hooks المتزامنة تحظر الاستجابات** | hook متزامن بطيء على `Stop` يؤخر رؤية المستخدم لاستجابة Claude. اجعل دائماً hooks غير الحرجة غير متزامنة |
| **مخرجات hook تلوّث السياق** | stdout من hooks المتزامنة من نوع command يُحقن في سياق Claude. أبقِ المخرجات في حدها الأدنى أو أعد توجيهها إلى ملف سجل |
| **مفاجآت تسلسل الإعدادات** | hook معرَّف في `.claude/settings.json` يمكن أن يُتجاوز بواسطة `.claude/settings.local.json`. تحقق من كلا الملفين عند تصحيح أخطاء hook لا ينطلق |

---

## المصادر

- [Claude Code Hooks — التوثيق](https://code.claude.com/docs/en/hooks)
- [دليل Claude Code Hooks](https://code.claude.com/docs/en/hooks-guide)
- [Claude Code Settings — التوثيق](https://code.claude.com/docs/en/settings)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

</div>
