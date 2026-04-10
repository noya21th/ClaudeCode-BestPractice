<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-troubleshooting.md) | [中文](../../zh/best-practice/claude-troubleshooting.md) | [日本語](../../ja/best-practice/claude-troubleshooting.md) | [Français](../../fr/best-practice/claude-troubleshooting.md) | [Русский](../../ru/best-practice/claude-troubleshooting.md) | **العربية**

# دليل استكشاف الأخطاء وإصلاحها

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

دليل تشخيصي لمشاكل Claude Code الشائعة — منظّم حسب الأعراض مع حلول خطوة بخطوة.

<table width="100%">
<tr>
<td><a href="../../ar/README.md">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## التشخيص السريع

شغّل هذه الأوامر أولاً عندما لا يعمل شيء ما:

| الأمر | ما يفحصه |
|-------|----------|
| `claude --version` | الإصدار المثبّت — قارن مع [أحدث إصدار](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) |
| `/doctor` | صحة التثبيت، إصدار Node.js، المصادقة، صلاحية الإعدادات |
| `/status` | النموذج الحالي، الحساب، اتصال API، مستوى الاشتراك |
| `/usage` | حدود الخطة، حالة حد المعدل، الحصة المتبقية |
| `/context` | استخدام نافذة السياق، ملفات الذاكرة المحمّلة، الـ Skills النشطة |
| `/hooks` | تكوينات Hook النشطة ومصادرها |
| `/mcp` | حالة اتصال خادم MCP |

---

## مشاكل التثبيت

### فشل تثبيت Claude Code

**الأعراض**: يفشل `npm install -g @anthropic-ai/claude-code` بأخطاء أذونات أو تعارضات إصدارات.

**السبب**: إصدار Node.js قديم جداً، مشاكل أذونات npm، أو حزم عامة متعارضة.

**الحل**:

```bash
# Check Node.js version (requires 18+)
node --version

# If version is too old, update via nvm
nvm install 20
nvm use 20

# Fix npm permissions (macOS/Linux)
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH  # Add to ~/.zshrc or ~/.bashrc

# Clean install
npm install -g @anthropic-ai/claude-code
```

### أمر Claude غير موجود

**الأعراض**: الطرفية تقول `claude: command not found` بعد تثبيت ناجح.

**السبب**: مجلد ثنائيات npm العامة ليس في PATH الخاص بك.

**الحل**:

```bash
# Find where npm installs global binaries
npm config get prefix

# Add to your shell profile (~/.zshrc, ~/.bashrc)
export PATH="$(npm config get prefix)/bin:$PATH"

# Reload shell
source ~/.zshrc
```

---

## مشاكل المصادقة

### مفتاح API غير معروف

**الأعراض**: يبدأ Claude Code لكن لا يستطيع إجراء استدعاءات API. رسائل الخطأ تذكر المصادقة أو مفتاح غير صالح.

**السبب**: مفتاح API مفقود أو منتهي الصلاحية. متغير بيئة خاطئ.

**الحل**:

```bash
# Re-authenticate
claude /login

# Or set the API key directly
export ANTHROPIC_API_KEY="sk-ant-..."

# For Bedrock users
export CLAUDE_CODE_USE_BEDROCK=1
claude /setup-bedrock
```

### أخطاء حد المعدل

**الأعراض**: أخطاء `429 Too Many Requests`. يتوقف Claude عن الاستجابة في منتصف المهمة.

**السبب**: تجاوز حدود معدل الخطة. جلسات أو Agents متزامنة كثيرة جداً.

**الحل**:

```bash
# Check current usage
/usage

# Options:
# 1. Wait for rate limit to reset (check Retry-After header)
# 2. Switch to a lighter model for high-volume tasks
/model haiku

# 3. Reduce concurrent agents
# 4. Enable extra usage
/extra-usage
```

---

## مشاكل الأداء

### ردود بطيئة

**الأعراض**: يستغرق Claude أكثر من 30 ثانية للاستجابة. توقفات طويلة بين استدعاءات الأدوات.

**السبب**: نافذة السياق كبيرة جداً. الكثير من الـ Skills محمّلة. Hooks متزامنة تعطّل الردود.

**الحل**:

```bash
# Check context usage
/context

# Compact if above 50%
/compact

# Check for slow synchronous hooks
/hooks
# Make non-critical hooks async: "async": true

# Reduce loaded skills — check if unnecessary skills are auto-activating
/skills
```

### استهلاك عالٍ للرموز

**الأعراض**: استنفاد الحصة أسرع من المتوقع. `/usage` يُظهر استهلاكاً عالياً.

**السبب**: ملفات CLAUDE.md كبيرة. العديد من الـ Skills محمّلة مسبقاً. استدعاء Agent مع وراثة كاملة للسياق. لا ضغط.

**الحل**:

1. أبقِ CLAUDE.md تحت 200 سطر
2. استخدم `user-invocable: false` على الـ Skills التي ينبغي تحميلها مسبقاً فقط بواسطة Agents محددة
3. عيّن `model: haiku` على الـ Agents التي لا تحتاج القدرات الكاملة
4. اضغط بانتظام باستخدام `/compact`
5. استخدم `effort: low` أو `effort: medium` للمهام الروتينية

---

## مشاكل الـ Hooks

### Hook لا يعمل

**الأعراض**: كوّنت Hook في الإعدادات لكنه لا يُنفّذ أبداً.

**السبب**: ملف إعدادات خاطئ. مطابق مفقود أو غير متطابق. Hook معطّل. تجاوز تسلسل الإعدادات.

**الحل**:

```bash
# 1. Check which hooks are active and their sources
/hooks

# 2. Verify the hook is in the correct settings file
# Team hooks: .claude/settings.json
# Personal hooks: .claude/settings.local.json
# Global hooks: ~/.claude/settings.json

# 3. Check if hooks are globally disabled
# Look for "disableAllHooks": true in any settings file

# 4. Verify the matcher matches the tool name exactly
# "Bash" not "bash", "Write" not "write"

# 5. Check settings hierarchy — a higher-priority file may override
```

### Hook يعطّل الردود

**الأعراض**: توقفات طويلة بعد استدعاءات الأدوات. Claude يبدو متجمداً لعدة ثوانٍ.

**السبب**: Hook متزامن يشغّل أمراً بطيئاً. استدعاء شبكة في Hook غير متزامن.

**الحل**:

```json
// Add "async": true to non-blocking hooks
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "type": "command",
        "command": "python3 log.py",
        "async": true
      }
    ]
  }
}
```

### انتهاء مهلة Hook

**الأعراض**: يبدأ Hook لكن النتيجة لا تُستخدم أبداً. تحذير حول انتهاء مهلة Hook في السجلات.

**السبب**: أمر Hook يستغرق وقتاً طويلاً. خدمة خارجية غير متاحة. السكربت يتعلّق.

**الحل**: اختبر أمر Hook يدوياً. أضف مهلة إلى السكربت نفسه. تأكد أن الخدمات الخارجية قابلة للوصول. فكّر في جعل Hook غير متزامن إذا لم تكن النتيجة حرجة.

---

## مشاكل MCP

### خادم MCP لا يتصل

**الأعراض**: أدوات MCP غير متاحة. `/mcp` يُظهر الخادم كمنقطع الاتصال.

**السبب**: ملف الخادم غير موجود. تكوين خاطئ. تعارض منافذ. متغيرات بيئة مفقودة.

**الحل**:

```bash
# 1. Check MCP status
/mcp

# 2. Verify server configuration in .mcp.json or settings.json
# Ensure the command path is correct and executable

# 3. Check if required environment variables are set
# MCP servers often need API keys in .env

# 4. Try restarting the MCP connection
# Exit and restart Claude Code

# 5. Test the MCP server manually
npx @modelcontextprotocol/server-name
```

### رفض أذونات MCP

**الأعراض**: أدوات MCP تظهر لكن تفشل بأخطاء أذونات عند الاستدعاء.

**السبب**: الأداة ليست في القائمة المسموح بها. وضع الأذونات مقيّد جداً.

**الحل**: أضف أداة MCP إلى قائمة `allowedTools` في الإعدادات، أو وافق عليها عند المطالبة. للـ Agents، أضف `mcp__<server>__<tool>` في حقل `tools:`.

---

## مشاكل الـ Agents

### Agent لا يُستدعى تلقائياً

**الأعراض**: أنشأت Agent بوصف يحتوي على `PROACTIVELY` لكن Claude لا يستدعيه تلقائياً أبداً.

**السبب**: الوصف لا يصف بوضوح متى يُستدعى. أداة Agent غير متاحة في السياق الحالي. آلية أخرى تتعامل مع المهمة.

**الحل**:

1. تأكد أن حقل `description` يستخدم `PROACTIVELY` ويذكر شرط التشغيل بوضوح
2. تحقق أن ملف الـ Agent في `.claude/agents/` بامتداد `.md`
3. تأكد أن Claude لديه وصول إلى أداة `Agent` (غير مقيّد بحقل `tools:` الخاص بالـ Agent الأب)

### Agent يستدعي Bash بدلاً من أداة Agent

**الأعراض**: يشغّل Claude `claude --agent my-agent` عبر Bash بدلاً من استخدام أداة `Agent(...)`.

**السبب**: تعريف Agent يستخدم لغة غامضة مثل "launch" أو "run" التي يفسّرها Claude كأمر shell.

**الحل**: في CLAUDE.md أو تعليمات الـ Agent، اذكر صراحةً: "Use the Agent tool, not bash commands, to spawn subagents." تجنّب مصطلحات مثل "launch" أو "execute" في أوصاف الـ Agent.

### Agent يتجاوز maxTurns

**الأعراض**: يتوقف الـ Agent في منتصف المهمة بنتيجة غير مكتملة.

**السبب**: `maxTurns` معيّن بقيمة منخفضة جداً لتعقيد المهمة. الـ Agent يدور في حلقة أو يعيد المحاولة.

**الحل**: زِد `maxTurns` في frontmatter الخاص بالـ Agent. راجع سلوك الـ Agent بحثاً عن حلقات غير ضرورية. قسّم المهام المعقدة إلى مهام فرعية أصغر.

---

## مشاكل الـ Skills

### Skill لا يظهر في القائمة /

**الأعراض**: أنشأت Skill لكنه لا يظهر عندما تكتب `/`.

**السبب**: `user-invocable: false` معيّن. ملف الـ Skill ليس في الموقع الصحيح. خطأ في صيغة YAML frontmatter.

**الحل**:

```bash
# 1. Check skill listing
/skills

# 2. Verify file location: .claude/skills/<name>/SKILL.md

# 3. Check frontmatter — remove user-invocable: false
# or set user-invocable: true

# 4. Validate YAML — common errors:
# - Missing --- delimiters
# - Incorrect indentation
# - Unquoted special characters
```

### Skill لا يُكتشف تلقائياً

**الأعراض**: الـ Skill موجود ولديه وصف لكن Claude لا يستدعيه تلقائياً أبداً.

**السبب**: `disable-model-invocation: true` معيّن. الوصف لا يطابق السياق الذي يصادفه Claude. الكثير من الـ Skills تتنافس على نفس المحفّز.

**الحل**:

1. أزِل `disable-model-invocation: true` إذا كان الاكتشاف التلقائي مطلوباً
2. اكتب `description` واضحاً باستخدام كلمات مفتاحية يصادفها Claude بشكل طبيعي
3. قلّل عدد الـ Skills — الكثير من الـ Skills القابلة للاكتشاف التلقائي يخلق غموضاً

---

## مشاكل الذاكرة

### CLAUDE.md لا يُحمّل

**الأعراض**: التعليمات في CLAUDE.md يتم تجاهلها. Claude لا يتبع اتفاقيات المشروع.

**السبب**: الملف في مجلد فرعي (تحميل كسول، ليس تحميلاً فورياً). خطأ مطبعي في اسم الملف. تجاوز الإعدادات عبر سياسة مُدارة.

**الحل**:

```bash
# 1. Check loaded memory files
/context

# 2. Verify file name is exactly "CLAUDE.md" (case-sensitive)

# 3. Understand loading rules:
# Ancestors (upward from cwd): loaded eagerly at startup
# Descendants (downward from cwd): loaded lazily when files in that dir are accessed
# Siblings: never loaded

# 4. If you need it always loaded, move it to an ancestor directory
# or reference it from root CLAUDE.md
```

### CLAUDE.local.md يتم إرساله إلى Git

**الأعراض**: التفضيلات الشخصية تظهر لزملاء الفريق. `.claude/settings.local.json` في سجل git.

**السبب**: الملفات المحلية ليست في `.gitignore`.

**الحل**:

```bash
# Add to .gitignore
echo "CLAUDE.local.md" >> .gitignore
echo ".claude/settings.local.json" >> .gitignore

# Remove from tracking if already committed
git rm --cached CLAUDE.local.md
git rm --cached .claude/settings.local.json
```

---

## مشاكل Git

### تنظيف Worktree

**الأعراض**: worktrees قديمة متبقية بعد تنفيذ الـ Agent. مساحة القرص تنمو.

**السبب**: Agent مع `isolation: worktree` تعطّل أو قُطع قبل التنظيف.

**الحل**:

```bash
# List worktrees
git worktree list

# Remove stale worktrees
git worktree prune

# Force remove a specific worktree
git worktree remove /path/to/worktree --force
```

### مؤلف الـ Commit

**الأعراض**: عمليات Git commit مُسندة إلى مؤلف خاطئ عندما ينشئها Claude Code.

**السبب**: مؤلف Git غير مكوّن لجلسات Claude Code.

**الحل**:

```bash
# Set git author for the project
git config user.name "Your Name"
git config user.email "your@email.com"

# Or use --author flag in hooks/scripts
git commit --author="Your Name <your@email.com>" -m "message"
```

---

## المصادر

- [Claude Code Troubleshooting — Docs](https://code.claude.com/docs/en/troubleshooting)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Hooks — Docs](https://code.claude.com/docs/en/hooks)
- [Claude Code MCP — Docs](https://code.claude.com/docs/en/mcp)
- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

</div>
