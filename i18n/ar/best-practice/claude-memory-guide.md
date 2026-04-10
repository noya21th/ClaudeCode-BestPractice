<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-memory-guide.md) | [中文](../../zh/best-practice/claude-memory-guide.md) | [日本語](../../ja/best-practice/claude-memory-guide.md) | [Français](../../fr/best-practice/claude-memory-guide.md) | [Русский](../../ru/best-practice/claude-memory-guide.md) | **العربية**

# دليل الذاكرة وإدارة السياق

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

دليل شامل لأنظمة ذاكرة Claude Code — ملفات CLAUDE.md، والذاكرة التلقائية، وذاكرة الـ Agent، والقواعد.

<table width="100%">
<tr>
<td><a href="../../ar/README.md">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## نظرة عامة على أنواع الذاكرة

يمتلك Claude Code أربعة أنظمة ذاكرة متميزة، كل منها يخدم غرضاً مختلفاً:

| نوع الذاكرة | الموقع | النطاق | الاستمرارية | وقت التحميل |
|-------------|--------|--------|------------|------------|
| **CLAUDE.md** | مجلدات المشروع | لكل مشروع، هرمي | دائمة (مبنية على ملفات) | بدء الجلسة (الأسلاف) أو عند الوصول للملفات (الفروع) |
| **الذاكرة التلقائية** | `~/.claude/memory/` و `~/.claude/projects/<hash>/memory/` | عامة للمستخدم أو لكل مشروع | دائمة (مبنية على ملفات) | بدء الجلسة |
| **ذاكرة الـ Agent** | `~/.claude/agent-memory/<agent>/` أو `.claude/agent-memory/<agent>/` | لكل Agent، مقيّدة بحقل `memory:` | دائمة (مبنية على ملفات) | بدء تشغيل الـ Agent |
| **القواعد** | `.claude/rules/*.md` و `~/.claude/rules/*.md` | مقيّدة بأنماط الملفات | دائمة (مبنية على ملفات) | عند الوصول لملفات تطابق أنماط glob |

---

## تسلسل CLAUDE.md الهرمي

### مواقع الملفات

| المستوى | المسار | مشترك | الوصف |
|---------|--------|-------|-------|
| **عام** | `~/.claude/CLAUDE.md` | لا (شخصي) | يُطبّق على جميع جلسات Claude Code على هذا الجهاز |
| **جذر المشروع** | `./CLAUDE.md` | نعم (متتبّع بـ git) | تعليمات على مستوى المشروع مشتركة للفريق |
| **محلي للمشروع** | `./CLAUDE.local.md` | لا (مُتجاهل بـ git) | تجاوزات شخصية خاصة بالمشروع |
| **المكوّن** | `./frontend/CLAUDE.md` | نعم (متتبّع بـ git) | تعليمات خاصة بالمكوّن |
| **متداخل** | `./src/api/v2/CLAUDE.md` | نعم (متتبّع بـ git) | تعليمات خاصة بمجلد عميق |

### آليات التحميل

يستخدم Claude Code آليتين متميزتين لاكتشاف ملفات CLAUDE.md:

#### تحميل الأسلاف (صعوداً في الشجرة) — فوري

عند بدء الجلسة، يمشي Claude **صعوداً** من مجلد العمل الحالي إلى جذر نظام الملفات ويحمّل كل CLAUDE.md يجده. يتم تحميلها **فوراً عند بدء التشغيل**.

```
/home/user/projects/myapp/src/   ← cwd
/home/user/projects/myapp/       ← CLAUDE.md loaded (ancestor)
/home/user/projects/             ← CLAUDE.md loaded (ancestor)
/home/user/                      ← CLAUDE.md loaded if exists (ancestor)
~/.claude/                       ← CLAUDE.md loaded (global)
```

#### تحميل الفروع (نزولاً في الشجرة) — كسول

ملفات CLAUDE.md في المجلدات الفرعية أسفل مجلد العمل الحالي **لا** تُحمّل عند بدء التشغيل. تُضمّن فقط عندما يقرأ Claude أو يعدّل ملفات في تلك المجلدات الفرعية أثناء الجلسة.

```
/myapp/                 ← cwd, CLAUDE.md loaded at startup
/myapp/frontend/        ← CLAUDE.md loaded when Claude accesses frontend/ files
/myapp/backend/         ← CLAUDE.md loaded when Claude accesses backend/ files
/myapp/backend/api/     ← CLAUDE.md loaded when Claude accesses backend/api/ files
```

#### القواعد الأساسية

- **الأسلاف تُحمّل دائماً عند بدء التشغيل** — فورياً، صعوداً من مجلد العمل الحالي
- **الفروع تُحمّل بشكل كسول** — فقط عند الوصول لملفات في ذلك المجلد
- **الأشقاء لا تُحمّل أبداً** — العمل في `frontend/` لا يحمّل `backend/CLAUDE.md`
- **العام يُحمّل دائماً** — `~/.claude/CLAUDE.md` يُطبّق على كل جلسة

---

## استراتيجيات Monorepo

### CLAUDE.md الجذري — الاتفاقيات المشتركة

أبقِ الملف الجذري مركّزاً على المعايير المشتركة للفريق التي تُطبّق في كل مكان:

```markdown
# CLAUDE.md (root)

## Coding Standards
- TypeScript strict mode everywhere
- No `any` types without justification comments
- Prefer functional components with hooks

## Git Conventions
- Conventional commits: feat/fix/chore/docs/refactor
- Squash merge to main

## Testing
- Every PR must include tests
- Minimum 80% coverage for new code
```

### CLAUDE.md المكوّن — التفاصيل

كل مكوّن يحصل على تعليماته المركّزة الخاصة:

```markdown
# frontend/CLAUDE.md

## Stack
- Next.js 15 App Router
- Tailwind CSS + shadcn/ui
- React Query for server state

## Patterns
- Server Components by default
- Client Components only for interactivity
- Use app/api/ for API routes
```

### CLAUDE.local.md — التفضيلات الشخصية

تجاوزات شخصية لا ينبغي إرسالها إلى Git أبداً:

```markdown
# CLAUDE.local.md (git-ignored)

- Use verbose comments — I am learning this codebase
- Run prettier after every file edit
- Always explain your reasoning before making changes
```

---

## نظام الذاكرة التلقائية

الذاكرة التلقائية هي نظام Claude Code لتذكّر الأشياء عبر المحادثات. يخزّن الملاحظات والدروس المستفادة التي تستمر بين الجلسات.

### الأنواع

| النوع | مخزّن في | الوصف |
|-------|----------|-------|
| `user` | `~/.claude/memory/` | تفضيلات المستخدم والأنماط العامة — تُطبّق على جميع المشاريع |
| `feedback` | `~/.claude/memory/` | التصحيحات والتعديلات السلوكية من ملاحظات المستخدم |
| `project` | `~/.claude/projects/<hash>/memory/` | دروس مستفادة وسياق خاص بالمشروع |
| `reference` | `~/.claude/projects/<hash>/memory/` | مواد مرجعية وجدها Claude مفيدة لهذا المشروع |

### كيف يعمل

1. يلاحظ Claude نمطاً أو تفضيلاً أو تصحيحاً أثناء الجلسة
2. إذا كانت الملاحظة مفيدة غالباً في جلسات مستقبلية، يخزّنها Claude كذاكرة تلقائية
3. عند بدء الجلسة التالية، تُحمّل إدخالات الذاكرة التلقائية ذات الصلة في السياق
4. يمكن للمستخدمين عرض وإدارة الإدخالات عبر `/memory`

### متى تستخدم الذاكرة التلقائية مقابل CLAUDE.md

| استخدم الذاكرة التلقائية عندما | استخدم CLAUDE.md عندما |
|-------------------------------|----------------------|
| نشأت التعليمات من ملاحظات أثناء الجلسة | التعليمات مُخططة مسبقاً |
| إنها تفضيل شخصي (الأسلوب، التفصيل، سير العمل) | إنها اتفاقية فريق (معايير الترميز، البنية) |
| تتطوّر بمرور الوقت من خلال الاستخدام | مستقرة وتُراجع في مراجعة الكود |
| تُطبّق على المستخدم عبر المشاريع | تُطبّق على المشروع بغض النظر عن المستخدم |

---

## ذاكرة الـ Agent

الـ Agents التي تمتلك حقل `memory:` في frontmatter الخاص بها تحافظ على ذاكرة دائمة تبقى عبر الجلسات. هذا يسمح للـ Agents بالتعلّم والتحسّن بمرور الوقت.

### النطاقات

| النطاق | مخزّن في | مشترك | الوصف |
|--------|----------|-------|-------|
| `user` | `~/.claude/agent-memory/<agent>/` | لا | ذاكرة عبر المشاريع لهذا الـ Agent — تتبع المستخدم |
| `project` | `.claude/agent-memory/<agent>/` | نعم | ذاكرة خاصة بالمشروع — مشتركة مع الفريق عبر git |
| `local` | `.claude/agent-memory/<agent>/` (مُتجاهل بـ git) | لا | ذاكرة خاصة بالمشروع، شخصية |

### التكوين

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes"
memory: project
---
Review code changes. After each review, note patterns you see frequently
so you can flag them earlier in future reviews.
```

### حالات الاستخدام

- **مراجع الكود** مع `memory: project` — يتعلّم أنماط الفريق المحددة، الأخطاء الشائعة، تفضيلات الأسلوب
- **مساعد الأبحاث** مع `memory: user` — يتذكّر المواضيع المستكشفة، المصادر المفضّلة، أسلوب الكتابة
- **Agent النشر** مع `memory: project` — يتتبّع سجل النشر، خصوصيات البيئة المعروفة

---

## القواعد (.claude/rules/)

القواعد هي ملفات markdown مقيّدة بأنماط glob تُحمّل تلقائياً عندما يصل Claude لملفات تطابق نمط glob.

### الموقع

| المسار | النطاق |
|--------|--------|
| `.claude/rules/*.md` | قواعد على مستوى المشروع (متتبّعة بـ git، مشتركة للفريق) |
| `~/.claude/rules/*.md` | قواعد عامة (شخصية، تُطبّق على جميع المشاريع) |

### الصيغة

كل ملف قاعدة يبدأ بسطر `# Glob:` يحدد متى تتفعّل القاعدة:

```markdown
# Glob: **/*.tsx

## React Component Rules

- Always use functional components
- Export components as named exports, not default
- Co-locate styles in a .module.css file
```

```markdown
# Glob: **/*.test.*

## Testing Rules

- Use describe/it blocks, not test()
- Mock external dependencies, not internal modules
- Assert on behavior, not implementation
```

### متى تستخدم القواعد مقابل CLAUDE.md

| استخدم القواعد عندما | استخدم CLAUDE.md عندما |
|---------------------|----------------------|
| التعليمات تُطبّق على أنواع ملفات محددة | التعليمات تُطبّق على المشروع بأكمله |
| تريد تحميلاً كسولاً بناءً على الوصول للملفات | تريد تحميلاً فورياً عند بدء الجلسة |
| الإرشادات مقيّدة بنمط (`*.py`، `*.tsx`، `migrations/*.sql`) | الإرشادات عامة (اتفاقيات git، قرارات البنية) |

---

## جدول المقارنة

| الميزة | CLAUDE.md | الذاكرة التلقائية | ذاكرة الـ Agent | القواعد |
|--------|-----------|------------------|----------------|---------|
| **مُنشأ بواسطة** | المطوّر (يدوياً) | Claude (تلقائياً) | Agent (تلقائياً) | المطوّر (يدوياً) |
| **مشترك مع الفريق** | نعم (باستثناء `.local.md`) | لا | نعم (إذا نطاق `project`) | نعم |
| **يُحمّل عند بدء التشغيل** | نعم (الأسلاف) | نعم | نعم (بدء الـ Agent) | لا (كسول) |
| **مقيّد بأنواع الملفات** | لا | لا | لا | نعم (أنماط glob) |
| **قابل للتعديل من المستخدم** | نعم (تعديل الملف) | نعم (`/memory`) | نعم (تعديل الملف) | نعم (تعديل الملف) |
| **يبقى عبر الجلسات** | نعم | نعم | نعم | نعم |
| **الأفضل لـ** | اتفاقيات المشروع | التفضيلات المُتعلّمة | تحسين الـ Agent | إرشادات خاصة بأنواع الملفات |

---

## أفضل الممارسات

### أبقِ CLAUDE.md تحت 200 سطر

ملفات CLAUDE.md الطويلة تسبب تضخّم السياق وقد يتجاهل Claude التعليمات القريبة من النهاية. قسّم الملفات الكبيرة إلى ملفات CLAUDE.md على مستوى المكوّنات وقواعد.

### استخدم القواعد لإرشادات أنواع الملفات

لا تضع "عند تعديل ملفات Python، افعل X" في CLAUDE.md. أنشئ `.claude/rules/python.md` مع `# Glob: **/*.py` بدلاً من ذلك. هذا يُحمّل فقط عند الحاجة.

### استخدم ذاكرة الـ Agent للـ Agents المتطوّرة

إذا كان ينبغي أن يصبح الـ Agent أذكى بمرور الوقت، أعطه نطاق `memory:`. يخزّن الـ Agent ملاحظات تحسّن عمليات التشغيل المستقبلية — أنماط المراجعة، روائح الكود، خصوصيات النشر.

### اضغط عند 50% من استخدام السياق

شغّل `/context` دورياً للتحقق من الاستخدام. عندما يصل إلى 50%، شغّل `/compact` مع تعليمات تركيز اختيارية: `/compact focus on the authentication refactor`. هذا يحفظ السياق المهم مع تحرير المساحة.

### تجاهل الملفات الشخصية في Git

أضف دائماً إلى `.gitignore`:

```
CLAUDE.local.md
.claude/settings.local.json
```

### افصل التعليمات المستقرة عن المتطوّرة

- **المستقرة** (معايير الترميز، البنية) → CLAUDE.md والقواعد
- **المتطوّرة** (التفضيلات، الأنماط، الدروس المستفادة) → الذاكرة التلقائية وذاكرة الـ Agent

---

## التقادم والتنظيف

### علامات الذاكرة المتقادمة

- CLAUDE.md يشير إلى APIs مُهملة أو ميزات محذوفة
- إدخالات الذاكرة التلقائية تتعارض مع تعليمات CLAUDE.md الحالية
- ذاكرة الـ Agent تحتوي على أنماط خاصة بالمشروع قديمة
- القواعد تشير إلى أنماط ملفات لم تعد موجودة في قاعدة الكود

### استراتيجيات التنظيف

1. **راجع CLAUDE.md ربع سنوياً** — اقرأ كل تعليمة وتأكد أنها لا تزال سارية
2. **راجع الذاكرة التلقائية** — شغّل `/memory` واحذف الإدخالات التي لم تعد ذات صلة
3. **نظّف ذاكرة الـ Agent** — تحقق من `.claude/agent-memory/` بحثاً عن ملاحظات قديمة
4. **حدّث القواعد** — تأكد أن أنماط glob لا تزال تطابق ملفات المشروع
5. **تتبّع بالتعليقات** — أضف `<!-- Last reviewed: 2026-04-09 -->` لأقسام CLAUDE.md

---

## المصادر

- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code Rules — Docs](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules)
- [Claude Code Sub-agents (Memory) — Docs](https://code.claude.com/docs/en/sub-agents)
- [CLAUDE.md in Monorepos](../../../best-practice/claude-memory.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

</div>
