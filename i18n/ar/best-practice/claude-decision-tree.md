<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-decision-tree.md) | [中文](../../zh/best-practice/claude-decision-tree.md) | [日本語](../../ja/best-practice/claude-decision-tree.md) | [Français](../../fr/best-practice/claude-decision-tree.md) | [Русский](../../ru/best-practice/claude-decision-tree.md) | **العربية**

# Agent مقابل Skill مقابل Command — شجرة القرار

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

متى تستخدم Agent أو Skill أو Command — إطار اتخاذ القرار مع سيناريوهات واقعية.

<table width="100%">
<tr>
<td><a href="../../ar/README.md">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## مقارنة سريعة

| البُعد | Agent | Command | Skill |
|--------|-------|---------|-------|
| **الموقع** | `.claude/agents/<name>.md` | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |
| **السياق** | سياق معزول جديد | يُحقن في السياق الحالي | يُحقن في السياق الحالي (أو يُفرع بـ `context: fork`) |
| **الاستدعاء** | أداة `Agent(...)`، اكتشاف تلقائي عبر الوصف | `/slash-command` بواسطة المستخدم | `/slash-command`، اكتشاف تلقائي، أو تحميل مسبق بواسطة Agent |
| **الذاكرة** | دائمة (نطاقات `user`/`project`/`local`) | لا يوجد | لا يوجد |
| **الأدوات** | قائمة مخصصة عبر حقل `tools:` | يرث أدوات الجلسة | يرث أدوات الجلسة (أو مقيّد عبر `allowed-tools:`) |
| **النموذج** | قابل للتكوين (`haiku`/`sonnet`/`opus`/`inherit`) | قابل للتكوين | قابل للتكوين |
| **الأذونات** | قابلة للتكوين (`acceptEdits`/`plan`/`bypassPermissions`) | يرث أذونات الجلسة | يرث أذونات الجلسة |
| **الاكتشاف التلقائي** | نعم (عبر `description`، استخدم `PROACTIVELY`) | لا (يُفعّل من المستخدم فقط) | نعم (عبر `description`) |
| **قابل للتحميل المسبق** | لا | لا | نعم (عبر حقل `skills:` في Agent) |
| **التشغيل في الخلفية** | نعم (`background: true`) | لا | لا |
| **عزل worktree** | نعم (`isolation: worktree`) | لا | لا |
| **الأفضل لـ** | المهام المعقدة متعددة الخطوات التي تحتاج عزلاً | سير العمل المُفعّل من المستخدم والتنسيق | المعرفة المتخصصة القابلة لإعادة الاستخدام والإجراءات |

---

## مخطط سير القرار

```
Start: "I need to add a capability to Claude Code"
│
├─ Does it need its own isolated context?
│  ├─ YES ──→ Does it need persistent memory across sessions?
│  │          ├─ YES ──→ AGENT (with memory: user/project/local)
│  │          └─ NO  ──→ AGENT (or Skill with context: fork)
│  │
│  └─ NO ──→ Should only the user trigger it explicitly?
│             ├─ YES ──→ COMMAND (user types /slash-command)
│             └─ NO  ──→ Should Claude auto-discover and invoke it?
│                        ├─ YES ──→ SKILL (with description for discovery)
│                        └─ NO  ──→ Is it reusable domain knowledge?
│                                   ├─ YES ──→ SKILL (with user-invocable: false for preload-only)
│                                   └─ NO  ──→ COMMAND (simple prompt template)
```

### قواعد القرار السريع

- **تحتاج عزل السياق؟** → Agent
- **تحتاج ذاكرة دائمة؟** → Agent
- **تحتاج تشغيلاً في الخلفية؟** → Agent
- **تحتاج عزل worktree؟** → Agent
- **تحتاج تقييد أدوات مخصص؟** → Agent
- **سير عمل يُفعّله المستخدم؟** → Command
- **إجراء أو معرفة متخصصة قابلة لإعادة الاستخدام؟** → Skill
- **قابل للاكتشاف التلقائي بواسطة Claude؟** → Skill (أو Agent مع `PROACTIVELY` في الوصف)
- **قابل للتحميل المسبق في Agent؟** → Skill

---

## سيناريوهات واقعية (12)

### 1. مراجعة الكود

**التوصية**: Agent

**السبب**: مراجعة الكود تحتاج سياقاً معزولاً حتى لا تلوث نتائج المراجعة المحادثة الرئيسية. يمكن أن يمتلك الـ Agent أدواته الخاصة (للقراءة فقط)، ونموذجه (haiku للسرعة)، وذاكرة دائمة لتعلم أنماط المراجعة الخاصة بالفريق بمرور الوقت.

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes for bugs, security, and style"
tools: Read, Glob, Grep, Bash
model: sonnet
memory: project
permissionMode: plan
---
```

### 2. جلب الطقس (سلسلة Command → Agent → Skill)

**التوصية**: Command ينسّق Agent مع Skill محمّل مسبقاً

**السبب**: يُفعّله المستخدم صراحةً (Command)، ومنطق الجلب يعمل في عزلة (Agent)، وتعليمات جلب البيانات هي معرفة متخصصة قابلة لإعادة الاستخدام (Skill محمّل مسبقاً في الـ Agent).

```
/weather-orchestrator  →  Agent(weather-agent)  →  Skill(weather-svg-creator)
     Command                   Agent                      Skill
```

### 3. إنشاء هيكل الملفات

**التوصية**: Skill

**السبب**: قوالب الهيكلة هي معرفة متخصصة قابلة لإعادة الاستخدام. يمكن لـ Claude اكتشاف الـ Skill تلقائياً عندما يطلب المستخدم إنشاء مكوّن أو مسار أو وحدة جديدة. لا حاجة للعزل — تُكتب الملفات في المشروع الحالي.

```yaml
# .claude/skills/react-scaffold/SKILL.md
---
name: react-scaffold
description: "Scaffold React components with TypeScript, tests, and Storybook stories"
paths: "src/components/**"
---
```

### 4. تنسيق CI/CD

**التوصية**: Command

**السبب**: سير عمل CI/CD يُفعّل دائماً من المستخدم (`/deploy`، `/release`). يتبع تسلسلاً ثابتاً ويحتاج الوصول إلى سياق الجلسة الحالي (الفرع، التغييرات الأخيرة).

```yaml
# .claude/commands/deploy.md
---
name: deploy
description: "Build, test, and deploy to staging or production"
argument-hint: "[staging|production]"
---
```

### 5. اختبار المتصفح

**التوصية**: Agent

**السبب**: اختبار المتصفح طويل التشغيل، قد يحتاج تشغيلاً في الخلفية، ولا ينبغي أن يعطّل المحادثة الرئيسية. يمكن أن يمتلك الـ Agent خوادم MCP محددة (Playwright، Chrome DevTools) ويعمل في worktree للعزل.

```yaml
# .claude/agents/browser-tester.md
---
name: browser-tester
description: "Run end-to-end browser tests against the dev server"
tools: Bash, Read, WebFetch, mcp__playwright__*
background: true
model: sonnet
---
```

### 6. عرض الوقت

**التوصية**: Skill

**السبب**: خفيف الوزن، لا يحتاج عزلاً، قابل للاكتشاف التلقائي. يستدعي Claude الـ Skill عندما يسأل أحدهم "كم الساعة؟"

### 7. نشر متعدد المراحل

**التوصية**: Command ينسّق عدة Agents

**السبب**: الأمر هو نقطة الدخول المواجهة للمستخدم (`/full-deploy`). يُنشئ Agents لكل مرحلة — Agent للبناء، Agent للاختبار، Agent للنشر — كل منها يعمل في سياقه الخاص بالأدوات المناسبة.

### 8. التنسيق والفحص (Linting / Formatting)

**التوصية**: Skill (يُستدعى تلقائياً)

**السبب**: قواعد الفحص هي معرفة متخصصة قابلة لإعادة الاستخدام. يتفعّل الـ Skill تلقائياً عبر `paths:` عندما يعدّل Claude ملفات مطابقة. لا حاجة لتفاعل المستخدم.

```yaml
# .claude/skills/python-linting/SKILL.md
---
name: python-linting
description: "Apply Black formatting and ruff linting to Python files"
paths: "**/*.py"
---
```

### 9. مساعد الأبحاث

**التوصية**: Agent

**السبب**: مهام البحث طويلة التشغيل، تستفيد من الذاكرة (تذكّر الأبحاث السابقة)، تحتاج أدوات ويب (`WebFetch`، `WebSearch`)، وينبغي أن تعمل في سياق معزول لتجنب ازدحام المحادثة الرئيسية.

### 10. ترحيل البيانات

**التوصية**: Agent مع عزل worktree

**السبب**: عمليات ترحيل قاعدة البيانات تعدّل ملفات حرجة وينبغي اختبارها في عزلة. التشغيل في git worktree يعني أن الفرع الرئيسي لا يتأثر حتى يتم التحقق من الترحيل.

```yaml
# .claude/agents/data-migrator.md
---
name: data-migrator
description: "Create and test database migration scripts"
isolation: worktree
model: sonnet
---
```

### 11. توليد التوثيق

**التوصية**: Skill

**السبب**: توليد التوثيق إجراء قابل لإعادة الاستخدام. يمكن لـ Claude اكتشافه تلقائياً عند طلب "اكتب التوثيق" أو "أنشئ مرجع API". يعمل في السياق الحالي ويكتب الملفات مباشرة في المشروع.

### 12. إعداد المشروع

**التوصية**: Command (مع Setup Hook)

**السبب**: تهيئة المشروع تُفعّل دائماً من المستخدم وتعمل مرة واحدة. ادمج أمر `/setup` مع `Setup` Hook للتشغيل التلقائي عند أول استنساخ.

---

## أنماط التركيب

### Command → Agent → Skill

نمط التنسيق الأساسي. الأمر هو نقطة دخول المستخدم، والـ Agent يوفر العزل والتحكم بالأدوات، والـ Skills توفر المعرفة المتخصصة.

```
User types /command
  └─ Command prompt runs in current context
       └─ Agent(...) spawned with isolated context
            └─ Preloaded skill provides instructions
                 └─ Agent completes, result surfaces to command context
                      └─ Skill tool invoked for final output
```

### Agent مع Skills محمّلة مسبقاً

يحمّل الـ Agent محتوى الـ Skill عند بدء التشغيل عبر حقل `skills:`. يُحقن محتوى الـ Skill في موجّه النظام الخاص بالـ Agent، مما يمنحه خبرة متخصصة دون أن يستدعي المستخدم أي شيء.

```yaml
# .claude/agents/api-builder.md
---
name: api-builder
skills:
  - openapi-validator
  - rest-conventions
  - error-handling-patterns
---
```

### Skill مع فرع السياق

Skill يحتاج عزلاً لكن ليس بكامل ثقل تعريف Agent. إعداد `context: fork` يشغّل الـ Skill في سياق Agent فرعي مع الحفاظ على تنسيق ملف Skill البسيط.

```yaml
# .claude/skills/security-scan/SKILL.md
---
name: security-scan
description: "Scan the codebase for security vulnerabilities"
context: fork
agent: general-purpose
---
```

---

## الأنماط المضادة في اختيار الآلية

| النمط المضاد | المشكلة | النهج الأفضل |
|--------------|---------|-------------|
| Agent لمهام بسيطة | عبء السياق المعزول عندما تحتاج فقط حقن تعليمات | استخدم Skill |
| Command للاكتشاف التلقائي | لا يمكن استدعاء Commands تلقائياً بواسطة Claude | استخدم Skill مع `description` |
| Skill لمهام طويلة التشغيل | الـ Skills تعمل في السياق الرئيسي وتعطّل المحادثة | استخدم Agent مع `background: true` |
| Agent لاتفاقيات مشتركة | سياق الـ Agent معزول — الاتفاقيات لا تنتقل للجلسة الرئيسية | استخدم CLAUDE.md أو Rules |
| Command لمعرفة قابلة للتحميل المسبق | لا يمكن تحميل Commands مسبقاً في Agents | استخدم Skill مع `user-invocable: false` |
| Skill بدون `description` | لا يستطيع Claude اكتشافه تلقائياً | أضف دائماً وصفاً ذا معنى |

---

## المصادر

- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Slash Commands — Docs](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Common Workflows — Docs](https://code.claude.com/docs/en/common-workflows)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

</div>
