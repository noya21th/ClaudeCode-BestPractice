<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../README.md) | [中文](../zh/README.md) | [日本語](../ja/README.md) | [Français](../fr/README.md) | [Русский](../ru/README.md) | **العربية**

# claude-code-best-practice
من البرمجة العفوية إلى الهندسة الوكيلية — الممارسة تُتقن Claude

![updated with Claude Code](https://img.shields.io/badge/updated_with_Claude_Code-v2.1.96%20(Apr%2008%2C%202026%209%3A38%20PM%20PKT)-white?style=flat&labelColor=555)<br>

[![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/) [![Implemented](../../!/tags/implemented.svg)](../../implementation/) [![Orchestration Workflow](../../!/tags/orchestration-workflow.svg)](../../orchestration-workflow/orchestration-workflow.md) [![Claude](../../!/tags/claude.svg)](https://code.claude.com/docs) [![Boris](../../!/tags/boris-cherny.svg)](#-نصائح-وحيل) [![Community](../../!/tags/community.svg)](#-الاشتراك) ![Click on these badges below to see the actual sources](../../!/tags/click-badges.svg)<br>
<img src="../../!/tags/a.svg" height="14"> = Agents · <img src="../../!/tags/c.svg" height="14"> = Commands · <img src="../../!/tags/s.svg" height="14"> = Skills

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="Claude Code mascot jumping" width="120" height="100"><br>

</p>



## 🧠 المفاهيم

| الميزة | الموقع | الوصف |
|--------|--------|-------|
| <img src="../../!/tags/a.svg" height="14"> [**Subagents**](https://code.claude.com/docs/en/sub-agents) | `.claude/agents/<name>.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-subagents.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-subagents-implementation.md) فاعل مستقل في سياق معزول جديد — أدوات مخصصة، صلاحيات، نموذج، ذاكرة، وهوية ثابتة |
| <img src="../../!/tags/c.svg" height="14"> [**Commands**](https://code.claude.com/docs/en/slash-commands) | `.claude/commands/<name>.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-commands.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-commands-implementation.md) معرفة تُحقن في السياق الحالي — قوالب أوامر بسيطة يستدعيها المستخدم لتنظيم سير العمل |
| <img src="../../!/tags/s.svg" height="14"> [**Skills**](https://code.claude.com/docs/en/skills) | `.claude/skills/<name>/SKILL.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-skills.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-skills-implementation.md) معرفة تُحقن في السياق الحالي — قابلة للتكوين، للتحميل المسبق، والاكتشاف التلقائي، مع تفريع السياق والكشف التدريجي · [Skills الرسمية](https://github.com/anthropics/skills/tree/main/skills) |
| [**Workflows**](https://code.claude.com/docs/en/common-workflows) | [`.claude/commands/weather-orchestrator.md`](../../.claude/commands/weather-orchestrator.md) | [![Orchestration Workflow](../../!/tags/orchestration-workflow.svg)](../../orchestration-workflow/orchestration-workflow.md) |
| [**Hooks**](https://code.claude.com/docs/en/hooks) | `.claude/hooks/` | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-hooks) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/claude-code-hooks) معالجات يحددها المستخدم (سكربتات، HTTP، أوامر، وكلاء) تعمل خارج الحلقة الوكيلية عند أحداث محددة · [الدليل](https://code.claude.com/docs/en/hooks-guide) |
| [**MCP Servers**](https://code.claude.com/docs/en/mcp) | `.claude/settings.json`, `.mcp.json` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-mcp.md) [![Implemented](../../!/tags/implemented.svg)](../../.mcp.json) اتصالات Model Context Protocol بأدوات خارجية وقواعد بيانات وواجهات برمجية |
| [**Plugins**](https://code.claude.com/docs/en/plugins) | حزم قابلة للتوزيع | حزم من Skills وSubagents وHooks وMCP Servers وLSP Servers · [الأسواق](https://code.claude.com/docs/en/discover-plugins) · [إنشاء الأسواق](https://code.claude.com/docs/en/plugin-marketplaces) |
| [**Settings**](https://code.claude.com/docs/en/settings) | `.claude/settings.json` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-settings.md) [![Implemented](../../!/tags/implemented.svg)](../../.claude/settings.json) نظام إعدادات هرمي · [الصلاحيات](https://code.claude.com/docs/en/permissions) · [إعداد النموذج](https://code.claude.com/docs/en/model-config) · [أنماط الإخراج](https://code.claude.com/docs/en/output-styles) · [Sandboxing](https://code.claude.com/docs/en/sandboxing) · [اختصارات المفاتيح](https://code.claude.com/docs/en/keybindings) · [الوضع السريع](https://code.claude.com/docs/en/fast-mode) |
| [**Status Line**](https://code.claude.com/docs/en/statusline) | `.claude/settings.json` | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-status-line) [![Implemented](../../!/tags/implemented.svg)](../../.claude/settings.json) شريط حالة قابل للتخصيص يعرض استخدام السياق والنموذج والتكلفة ومعلومات الجلسة |
| [**Memory**](https://code.claude.com/docs/en/memory) | `CLAUDE.md`, `.claude/rules/`, `~/.claude/rules/`, `~/.claude/projects/<project>/memory/` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-memory.md) [![Implemented](../../!/tags/implemented.svg)](../../CLAUDE.md) سياق مستمر عبر ملفات CLAUDE.md واستيراد `@path` · [الذاكرة التلقائية](https://code.claude.com/docs/en/memory) · [القواعد](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) |
| [**Checkpointing**](https://code.claude.com/docs/en/checkpointing) | تلقائي (يعتمد على git) | تتبع تلقائي لتعديلات الملفات مع إمكانية التراجع (`Esc Esc` أو `/rewind`) والتلخيص المستهدف |
| [**CLI Startup Flags**](https://code.claude.com/docs/en/cli-reference) | `claude [flags]` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-cli-startup-flags.md) أعلام سطر الأوامر والأوامر الفرعية ومتغيرات البيئة لتشغيل Claude Code · [الوضع التفاعلي](https://code.claude.com/docs/en/interactive-mode) |
| **AI Terms** | | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-codex-cursor-gemini/blob/main/reports/ai-terms.md) الهندسة الوكيلية · هندسة السياق · البرمجة العفوية |
| [**Best Practices**](https://code.claude.com/docs/en/best-practices) | | أفضل الممارسات الرسمية · [هندسة الأوامر](https://github.com/anthropics/prompt-eng-interactive-tutorial) · [توسيع Claude Code](https://code.claude.com/docs/en/features-overview) |

### 🔥 الجديد والمميز

| الميزة | الموقع | الوصف |
|--------|--------|-------|
| [**Power-ups**](../../best-practice/claude-power-ups.md) | `/powerup` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-power-ups.md) دروس تفاعلية تعلّم ميزات Claude Code مع عروض متحركة (v2.1.90) |
| [**Ultraplan**](https://code.claude.com/docs/en/ultraplan) ![beta](../../!/tags/beta.svg) | `/ultraplan` | صياغة الخطط في السحابة مع مراجعة عبر المتصفح وتعليقات مباشرة وتنفيذ مرن — عن بُعد أو إعادة إرسال إلى الطرفية |
| [**Claude Code Web**](https://code.claude.com/docs/en/claude-code-on-the-web) ![beta](../../!/tags/beta.svg) | `claude.ai/code` | تشغيل المهام على البنية التحتية السحابية — مهام طويلة، إصلاح PR تلقائي، جلسات متوازية بدون إعداد محلي · [المهام المجدولة](https://code.claude.com/docs/en/web-scheduled-tasks) |
| [**No Flicker Mode**](https://code.claude.com/docs/en/fullscreen) ![beta](../../!/tags/beta.svg) | `CLAUDE_CODE_NO_FLICKER=1` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2039421575422980329) عرض بدون وميض على شاشة بديلة مع دعم الماوس وذاكرة مستقرة وتمرير داخلي |
| [**Computer Use**](https://code.claude.com/docs/en/computer-use) ![beta](../../!/tags/beta.svg) | `computer-use` MCP server | اسمح لـ Claude بالتحكم في شاشتك — فتح التطبيقات، النقر، الكتابة، ولقطات الشاشة على macOS · [سطح المكتب](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer) |
| [**Auto Mode**](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) ![beta](../../!/tags/beta.svg) | `claude --enable-auto-mode` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/claudeai/status/2036503582166393240) مصنّف أمان خلفي يستبدل طلبات الصلاحيات اليدوية — Claude يقرر ما هو آمن مع حظر حقن الأوامر والتصعيدات الخطرة · [المدونة](https://claude.com/blog/auto-mode) |
| [**Channels**](https://code.claude.com/docs/en/channels) ![beta](../../!/tags/beta.svg) | `--channels`، قائم على plugin | إرسال أحداث من Telegram وDiscord أو webhooks إلى جلسة جارية — Claude يتفاعل أثناء غيابك · [المرجع](https://code.claude.com/docs/en/channels-reference) |
| [**Slack**](https://code.claude.com/docs/en/slack) | `@Claude` في Slack | اذكر @Claude في محادثة الفريق مع مهمة برمجية — يوجّه إلى جلسات Claude Code الويب لإصلاح الأخطاء ومراجعة الكود وتنفيذ المهام المتوازية |
| [**Code Review**](https://code.claude.com/docs/en/code-review) ![beta](../../!/tags/beta.svg) | GitHub App (مُدار) | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/claudeai/status/2031088171262554195) تحليل PR متعدد الوكلاء يكشف الأخطاء وثغرات الأمان والانحدارات · [المدونة](https://claude.com/blog/code-review) |
| [**GitHub Actions**](https://code.claude.com/docs/en/github-actions) | `.github/workflows/` | أتمتة مراجعات PR وفرز المشكلات وتوليد الكود في خطوط CI/CD · [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd) |
| [**Chrome**](https://code.claude.com/docs/en/chrome) ![beta](../../!/tags/beta.svg) | `--chrome`، extension | [![Best Practice](../../!/tags/best-practice.svg)](../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) أتمتة المتصفح عبر Claude in Chrome — اختبار تطبيقات الويب، التصحيح عبر وحدة التحكم، أتمتة النماذج، استخراج البيانات |
| [**Scheduled Tasks**](https://code.claude.com/docs/en/scheduled-tasks) | `/loop`, `/schedule`, أدوات cron | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2030193932404150413) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-scheduled-tasks-implementation.md) `/loop` يشغّل الأوامر محلياً على جدول متكرر (حتى 3 أيام) · [`/schedule`](https://code.claude.com/docs/en/web-scheduled-tasks) يشغّل الأوامر في السحابة حتى عندما يكون جهازك مغلقاً |
| [**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation) ![beta](../../!/tags/beta.svg) | `/voice` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/trq212/status/2028628570692890800) إدخال صوتي بالضغط مع الكلام للأوامر بدعم 20 لغة ومفتاح تفعيل قابل للتخصيص |
| [**Simplify & Batch**](https://code.claude.com/docs/en/skills#bundled-skills) | `/simplify`, `/batch` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2027534984534544489) Skills مدمجة لجودة الكود والعمليات الجماعية — simplify يعيد الهيكلة لإعادة الاستخدام والكفاءة، batch يشغّل الأوامر عبر ملفات متعددة |
| [**Agent Teams**](https://code.claude.com/docs/en/agent-teams) ![beta](../../!/tags/beta.svg) | مدمج (متغير بيئة) | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2019472394696683904) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-agent-teams-implementation.md) وكلاء متعددون يعملون بالتوازي على نفس قاعدة الكود مع تنسيق مشترك للمهام |
| [**Remote Control**](https://code.claude.com/docs/en/remote-control) | `/remote-control`, `/rc` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/noahzweben/status/2032533699116355819) متابعة الجلسات المحلية من أي جهاز — هاتف، جهاز لوحي، أو متصفح · [وضع Headless](https://code.claude.com/docs/en/headless) |
| [**Git Worktrees**](https://code.claude.com/docs/en/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees) | مدمج | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2025007393290272904) فروع git معزولة للتطوير المتوازي — كل وكيل يحصل على نسخة عمل خاصة به |
| [**Ralph Wiggum Loop**](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) | plugin | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/ghuntley/how-to-ralph-wiggum) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26) حلقة تطوير ذاتية للمهام طويلة الأمد — تتكرر حتى الاكتمال |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

<a id="orchestration-workflow"></a>

## <a href="../../orchestration-workflow/orchestration-workflow.md"><img src="../../!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

راجع [orchestration-workflow](../../orchestration-workflow/orchestration-workflow.md) لتفاصيل تنفيذ نمط <img src="../../!/tags/c.svg" height="14"> **Command** → <img src="../../!/tags/a.svg" height="14"> **Agent** → <img src="../../!/tags/s.svg" height="14"> **Skill**.

<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.svg" alt="مخطط تدفق بنية Command Skill Agent" width="100%">
</p>

<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.gif" alt="عرض توضيحي لسير عمل التنسيق" width="600">
</p>

![How to Use](../../!/tags/how-to-use.svg)

```bash
claude
/weather-orchestrator
```

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

## ⚙️ سير عمل التطوير

تتقارب جميع سير العمل الرئيسية على نفس النمط المعماري: **بحث → تخطيط → تنفيذ → مراجعة → إطلاق**

| الاسم | ★ | التميز | الخطة | <img src="../../!/tags/a.svg" height="14"> | <img src="../../!/tags/c.svg" height="14"> | <img src="../../!/tags/s.svg" height="14"> |
|-------|---|--------|-------|---|---|---|
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | 146k | ![instinct scoring](https://img.shields.io/badge/instinct_scoring-ddf4ff) ![AgentShield](https://img.shields.io/badge/AgentShield-ddf4ff) ![multi-lang rules](https://img.shields.io/badge/multi--lang_rules-ddf4ff) | <img src="../../!/tags/a.svg" height="14"> [planner](https://github.com/affaan-m/everything-claude-code/blob/main/agents/planner.md) | 47 | 82 | 182 |
| [Superpowers](https://github.com/obra/superpowers) | 141k | ![TDD-first](https://img.shields.io/badge/TDD--first-ddf4ff) ![Iron Laws](https://img.shields.io/badge/Iron_Laws-ddf4ff) ![whole-plan review](https://img.shields.io/badge/whole--plan_review-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [writing-plans](https://github.com/obra/superpowers/tree/main/skills/writing-plans) | 5 | 3 | 14 |
| [Spec Kit](https://github.com/github/spec-kit) | 86k | ![spec-driven](https://img.shields.io/badge/spec--driven-ddf4ff) ![constitution](https://img.shields.io/badge/constitution-ddf4ff) ![22+ tools](https://img.shields.io/badge/22%2B_tools-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [speckit.plan](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md) | 0 | 9+ | 0 |
| [gstack](https://github.com/garrytan/gstack) | 67k | ![role personas](https://img.shields.io/badge/role_personas-ddf4ff) ![/codex review](https://img.shields.io/badge/%2Fcodex_review-ddf4ff) ![parallel sprints](https://img.shields.io/badge/parallel_sprints-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [autoplan](https://github.com/garrytan/gstack/tree/main/autoplan) | 0 | 0 | 37 |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) | 49k | ![fresh 200K contexts](https://img.shields.io/badge/fresh_200K_contexts-ddf4ff) ![wave execution](https://img.shields.io/badge/wave_execution-ddf4ff) ![XML plans](https://img.shields.io/badge/XML_plans-ddf4ff) | <img src="../../!/tags/a.svg" height="14"> [gsd-planner](https://github.com/gsd-build/get-shit-done/blob/main/agents/gsd-planner.md) | 24 | 68 | 0 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 44k | ![full SDLC](https://img.shields.io/badge/full_SDLC-ddf4ff) ![agent personas](https://img.shields.io/badge/agent_personas-ddf4ff) ![22+ platforms](https://img.shields.io/badge/22%2B_platforms-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [bmad-create-prd](https://github.com/bmad-code-org/BMAD-METHOD/tree/main/src/bmm-skills/2-plan-workflows/bmad-create-prd) | 0 | 0 | 39 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | 38k | ![delta specs](https://img.shields.io/badge/delta_specs-ddf4ff) ![brownfield](https://img.shields.io/badge/brownfield-ddf4ff) ![artifact DAG](https://img.shields.io/badge/artifact_DAG-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [opsx:propose](https://github.com/Fission-AI/OpenSpec/blob/main/src/commands/workflow/new-change.ts) | 0 | 11 | 0 |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 26k | ![teams orchestration](https://img.shields.io/badge/teams_orchestration-ddf4ff) ![tmux workers](https://img.shields.io/badge/tmux_workers-ddf4ff) ![skill auto-inject](https://img.shields.io/badge/skill_auto--inject-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [ralplan](https://github.com/Yeachan-Heo/oh-my-claudecode/tree/main/skills/ralplan) | 19 | 0 | 37 |
| [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | 14k | ![Compound Learning](https://img.shields.io/badge/Compound_Learning-ddf4ff) ![Multi-Platform CLI](https://img.shields.io/badge/Multi--Platform_CLI-ddf4ff) ![Plugin Marketplace](https://img.shields.io/badge/Plugin_Marketplace-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [ce-plan](https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills/ce-plan) | 51 | 4 | 44 |
| [HumanLayer](https://github.com/humanlayer/humanlayer) | 10k | ![RPI](https://img.shields.io/badge/RPI-ddf4ff) ![context engineering](https://img.shields.io/badge/context_engineering-ddf4ff) ![300k+ LOC](https://img.shields.io/badge/300k%2B_LOC-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [create_plan](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/create_plan.md) | 6 | 27 | 0 |

### أخرى
- [سير عمل عبر النماذج (Claude Code + Codex)](../../development-workflows/cross-model-workflow/cross-model-workflow.md) [![Implemented](../../!/tags/implemented.svg)](../../development-workflows/cross-model-workflow/cross-model-workflow.md)
- [RPI](../../development-workflows/rpi/rpi-workflow.md) [![Implemented](../../!/tags/implemented.svg)](../../development-workflows/rpi/rpi-workflow.md)
- [Ralph Wiggum Loop](https://www.youtube.com/watch?v=eAtvoGlpeRU) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26)
- [سير عمل Andrej Karpathy (عضو مؤسس، OpenAI)](https://x.com/karpathy/status/2015883857489522876)
- [سير عمل Peter Steinberger (مبتكر OpenClaw)](https://youtu.be/8lF7HmQ_RgY?t=2582)
- سير عمل Boris Cherny (مبتكر Claude Code) — [13 نصيحة](../../tips/claude-boris-13-tips-03-jan-26.md) · [10 نصائح](../../tips/claude-boris-10-tips-01-feb-26.md) · [12 نصيحة](../../tips/claude-boris-12-tips-12-feb-26.md) · [نصيحتان](../../tips/claude-boris-2-tips-25-mar-26.md) · [15 نصيحة](../../tips/claude-boris-15-tips-30-mar-26.md) [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny)

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

## 💡 نصائح وحيل (69)

🚫👶 = لا تراقب بشكل مفرط

[صياغة الأوامر](#tips-prompting) · [التخطيط](#tips-planning) · [CLAUDE.md](#tips-claudemd) · [Agents](#tips-agents) · [Commands](#tips-commands) · [Skills](#tips-skills) · [Hooks](#tips-hooks) · [سير العمل](#tips-workflows) · [متقدم](#tips-workflows-advanced) · [Git / PR](#tips-git-pr) · [التصحيح](#tips-debugging) · [أدوات مساعدة](#tips-utilities) · [يومي](#tips-daily)

![Community](../../!/tags/community.svg)

<a id="tips-prompting"></a>■ **صياغة الأوامر (3)**

| النصيحة | المصدر |
|---------|--------|
| تحدَّ Claude — "ناقشني في هذه التغييرات ولا ترسل PR حتى أجتاز اختبارك." أو "أثبت لي أن هذا يعمل" واجعل Claude يقارن بين main وفرعك 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| بعد إصلاح متوسط — "بكل ما تعرفه الآن، ألغِ هذا ونفّذ الحل الأنيق" 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| Claude يصلح معظم الأخطاء بنفسه — الصق الخطأ، قل "أصلح"، لا تدير كل التفاصيل 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742750473720121) |

<a id="tips-planning"></a>■ **التخطيط/المواصفات (6)**

| النصيحة | المصدر |
|---------|--------|
| ابدأ دائماً بـ [plan mode](https://code.claude.com/docs/en/common-workflows) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179845336527000) |
| ابدأ بمواصفات بسيطة واطلب من Claude إجراء مقابلة معك باستخدام أداة [AskUserQuestion](https://code.claude.com/docs/en/cli-reference)، ثم أنشئ جلسة جديدة لتنفيذ المواصفات | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2005315275026260309) |
| ضع دائماً خطة مرحلية مقسّمة، مع اختبارات متعددة لكل مرحلة (وحدة، أتمتة، تكامل) | |
| شغّل Claude ثانٍ لمراجعة خطتك كمهندس أول، أو استخدم [cross-model](../../development-workflows/cross-model-workflow/cross-model-workflow.md) للمراجعة | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742745365057733) |
| اكتب مواصفات تفصيلية وقلّل الغموض قبل تسليم العمل — كلما كنت أكثر تحديداً، كان الناتج أفضل | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| النموذج الأولي أهم من مستند المتطلبات — ابنِ 20-30 نسخة بدلاً من كتابة المواصفات، تكلفة البناء منخفضة فخذ محاولات كثيرة | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3630) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3630) |

<a id="tips-claudemd"></a>■ **CLAUDE.md (7)**

| النصيحة | المصدر |
|---------|--------|
| يجب أن يكون [CLAUDE.md](https://code.claude.com/docs/en/memory) أقل من [200 سطر](https://code.claude.com/docs/en/memory#write-effective-instructions) لكل ملف. [60 سطراً في humanlayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md) ([لا يزال غير مضمون 100%](https://www.reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/)) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179840848597422) [![Dex](../../!/tags/community-dex.svg)](https://www.humanlayer.dev/blog/writing-a-good-claude-md) |
| غلّف قواعد CLAUDE.md الخاصة بالمجال في [علامات `<important if="...">`](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md) لمنع Claude من تجاهلها عندما تطول الملفات | [![Dex](../../!/tags/community-dex.svg)](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md) |
| استخدم [CLAUDE.md متعدد](../../best-practice/claude-memory.md) للمستودعات الأحادية — التحميل الصاعد + التحميل الفرعي | |
| استخدم [.claude/rules/](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) لتقسيم التعليمات الكبيرة | |
| memory.md و constitution.md لا يضمنان شيئاً | |
| أي مطور يجب أن يستطيع تشغيل Claude وقول "شغّل الاختبارات" ويعمل من المحاولة الأولى — إن لم يعمل، فإن CLAUDE.md يفتقد أوامر الإعداد/البناء/الاختبار الأساسية | [![Dex](../../!/tags/community-dex.svg)](https://x.com/dexhorthy/status/2034713765401551053) |
| حافظ على نظافة قواعد الكود وأنهِ عمليات الترحيل — الأطر المرحّلة جزئياً تربك النماذج التي قد تختار النمط الخاطئ | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=1112) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=1112) |
| استخدم [settings.json](../../best-practice/claude-settings.md) للسلوك المفروض من النظام (الإسناد، الصلاحيات، النموذج) — لا تضع "لا تضف أبداً Co-Authored-By" في CLAUDE.md عندما يكون `attribution.commit: ""` حتمياً | [![davila7](../../!/tags/community-davila7.svg)](https://x.com/dani_avila7/status/2036182734310195550) |

<a id="tips-agents"></a><img src="../../!/tags/a.svg" height="14"> **Agents (4)**

| النصيحة | المصدر |
|---------|--------|
| أنشئ [sub-agents](https://code.claude.com/docs/en/sub-agents) خاصة بالميزات (سياق إضافي) مع [skills](https://code.claude.com/docs/en/skills) (كشف تدريجي) بدلاً من وكلاء عامين مثل qa أو backend engineer | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179850139000872) |
| قل "use subagents" لتوجيه المزيد من الحوسبة لمشكلة — فوّض المهام لإبقاء سياقك الرئيسي نظيفاً ومركّزاً 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| [فرق الوكلاء مع tmux](https://code.claude.com/docs/en/agent-teams) و[git worktrees](https://x.com/bcherny/status/2025007393290272904) للتطوير المتوازي | |
| استخدم [حساب وقت الاختبار](https://code.claude.com/docs/en/sub-agents) — نوافذ السياق المنفصلة تحسّن النتائج؛ وكيل واحد قد يسبب أخطاء ووكيل آخر (نفس النموذج) يمكنه اكتشافها | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031151689219321886) |

<a id="tips-commands"></a><img src="../../!/tags/c.svg" height="14"> **Commands (3)**

| النصيحة | المصدر |
|---------|--------|
| استخدم [commands](https://code.claude.com/docs/en/slash-commands) لسير عملك بدلاً من [sub-agents](https://code.claude.com/docs/en/sub-agents) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| استخدم [slash commands](https://code.claude.com/docs/en/slash-commands) لكل سير عمل "حلقة داخلية" تقوم به عدة مرات يومياً — يوفر الكتابة المتكررة، الأوامر تعيش في `.claude/commands/` وتُضاف إلى git | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| إذا كنت تفعل شيئاً أكثر من مرة يومياً، حوّله إلى [skill](https://code.claude.com/docs/en/skills) أو [command](https://code.claude.com/docs/en/slash-commands) — ابنِ `/techdebt`، context-dump، أو أوامر تحليلات | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742748984742078) |

<a id="tips-skills"></a><img src="../../!/tags/s.svg" height="14"> **Skills (9)**

| النصيحة | المصدر |
|---------|--------|
| استخدم [context: fork](https://code.claude.com/docs/en/skills) لتشغيل skill في وكيل فرعي معزول — السياق الرئيسي يرى النتيجة النهائية فقط، وليس الاستدعاءات الوسيطة. حقل agent يتيح تعيين نوع الوكيل الفرعي | [![Lydia](../../!/tags/lydia.svg)](https://x.com/lydiahallie/status/2033603164398883042) |
| استخدم [skills في المجلدات الفرعية](../../reports/claude-skills-for-larger-mono-repos.md) للمستودعات الأحادية | |
| Skills هي مجلدات وليست ملفات — استخدم مجلدات references/ وscripts/ وexamples/ الفرعية لـ[الكشف التدريجي](https://code.claude.com/docs/en/skills) | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| ابنِ قسم Gotchas في كل skill — محتوى ذو إشارة عالية، أضف نقاط فشل Claude بمرور الوقت | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| حقل description في skill هو مشغّل وليس ملخصاً — اكتبه للنموذج ("متى يجب أن أنطلق؟") | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| لا تذكر البديهيات في skills — ركّز على ما يدفع Claude خارج سلوكه الافتراضي 🚫👶 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| لا تُقيّد Claude في skills — أعطِ أهدافاً وقيوداً، وليس تعليمات تفصيلية خطوة بخطوة 🚫👶 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| أدرج سكربتات ومكتبات في skills لكي يركّب Claude بدلاً من إعادة بناء الكود المتكرر | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| ضمّن `` !`command` `` في SKILL.md لحقن مخرجات shell الديناميكية في الأمر — Claude يشغّلها عند الاستدعاء والنموذج يرى النتيجة فقط | [![Lydia](../../!/tags/lydia.svg)](https://x.com/lydiahallie/status/2034337963820327017) |

<a id="tips-hooks"></a>■ **Hooks (5)**

| النصيحة | المصدر |
|---------|--------|
| استخدم [hooks عند الطلب](https://code.claude.com/docs/en/skills) في skills — /careful يمنع الأوامر المدمرة، /freeze يمنع التعديلات خارج مجلد معين | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| [قِس استخدام skill](https://code.claude.com/docs/en/skills) بـ PreToolUse hook لاكتشاف skills الشائعة أو التي لا تنطلق كفاية | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| استخدم [PostToolUse hook](https://code.claude.com/docs/en/hooks) لتنسيق الكود تلقائياً — Claude ينتج كوداً منسقاً جيداً، والـ hook يتولى الـ 10% الأخيرة لتجنب فشل CI | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179852047335529) |
| وجّه [طلبات الصلاحيات](https://code.claude.com/docs/en/hooks) إلى Opus عبر hook — دعه يفحص الهجمات ويوافق تلقائياً على الآمنة 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| استخدم [Stop hook](https://code.claude.com/docs/en/hooks) لحث Claude على المتابعة أو التحقق من عمله في نهاية الدورة | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701059253874861) |

<a id="tips-workflows"></a>■ **سير العمل (7)**

| النصيحة | المصدر |
|---------|--------|
| تجنّب منطقة الغباء الوكيلي، نفّذ [/compact](https://code.claude.com/docs/en/interactive-mode) يدوياً عند 50% كحد أقصى. استخدم [/clear](https://code.claude.com/docs/en/cli-reference) لإعادة تعيين السياق منتصف الجلسة عند الانتقال لمهمة جديدة | |
| Claude Code العادي أفضل من أي سير عمل مع المهام الصغيرة | |
| استخدم [/model](https://code.claude.com/docs/en/model-config) لاختيار النموذج والاستدلال، [/context](https://code.claude.com/docs/en/interactive-mode) لرؤية استخدام السياق، [/usage](https://code.claude.com/docs/en/costs) للتحقق من حدود الخطة، [/extra-usage](https://code.claude.com/docs/en/interactive-mode) لتكوين الفوترة الإضافية، [/config](https://code.claude.com/docs/en/settings) لتكوين الإعدادات — استخدم Opus لوضع الخطة وSonnet للكود للحصول على أفضل النتائج | [![Cat](../../!/tags/cat-wu.svg)](https://x.com/_catwu/status/1955694117264261609) |
| استخدم دائماً [thinking mode](https://code.claude.com/docs/en/model-config) (لرؤية الاستدلال) و[Output Style](https://code.claude.com/docs/en/output-styles) Explanatory (لرؤية مخرجات تفصيلية مع صناديق ★ Insight) في [/config](https://code.claude.com/docs/en/settings) لفهم أفضل لقرارات Claude | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179838864666847) |
| استخدم كلمة ultrathink في الأوامر لـ[استدلال عالي الجهد](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking#tips-and-best-practices) | |
| [/rename](https://code.claude.com/docs/en/cli-reference) للجلسات المهمة (مثل [TODO - refactor task]) و[/resume](https://code.claude.com/docs/en/cli-reference) لاحقاً — سمِّ كل نسخة عند تشغيل عدة Claude في وقت واحد | [![Cat](../../!/tags/cat-wu.svg)](https://every.to/podcast/how-to-use-claude-code-like-the-people-who-built-it) |
| استخدم [Esc Esc أو /rewind](https://code.claude.com/docs/en/checkpointing) للتراجع عندما يخرج Claude عن المسار بدلاً من محاولة الإصلاح في نفس السياق | |

<a id="tips-workflows-advanced"></a>■ **سير عمل متقدم (6)**

| النصيحة | المصدر |
|---------|--------|
| استخدم مخططات ASCII كثيراً لفهم بنيتك | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742759218794768) |
| استخدم [/loop](https://code.claude.com/docs/en/scheduled-tasks) للمراقبة المتكررة المحلية (حتى 3 أيام) · استخدم [/schedule](https://code.claude.com/docs/en/web-scheduled-tasks) للمهام السحابية المتكررة التي تعمل حتى عندما يكون جهازك مغلقاً | |
| استخدم [Ralph Wiggum plugin](https://github.com/shanraisshan/novel-llm-26) للمهام الذاتية طويلة الأمد | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179858435281082) |
| [/permissions](https://code.claude.com/docs/en/permissions) مع صيغة wildcard (Bash(npm run *), Edit(/docs/**)) بدلاً من dangerously-skip-permissions | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179854077407667) |
| [/sandbox](https://code.claude.com/docs/en/sandboxing) لتقليل طلبات الصلاحيات مع عزل الملفات والشبكة — انخفاض 84% داخلياً | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700506465579443) [![Cat](../../!/tags/cat-wu.svg)](https://creatoreconomy.so/p/inside-claude-code-how-an-ai-native-actually-works-cat-wu) |
| استثمر في skills [التحقق من المنتج](https://code.claude.com/docs/en/skills) (signup-flow-driver, checkout-verifier) — يستحق قضاء أسبوع في إتقانها | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |

<a id="tips-git-pr"></a>■ **Git / PR (5)**

| النصيحة | المصدر |
|---------|--------|
| اجعل PRs صغيرة ومركّزة — [متوسط 118 سطراً](../../tips/claude-boris-2-tips-25-mar-26.md) (141 PR، 45 ألف سطر تم تغييرها في يوم)، ميزة واحدة لكل PR، أسهل للمراجعة والتراجع | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| استخدم دائماً [squash merge](../../tips/claude-boris-2-tips-25-mar-26.md) لـ PRs — تاريخ خطي نظيف، commit واحد لكل ميزة، سهولة في git revert وgit bisect | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| أنشئ commit كثيراً — حاول الإنشاء مرة واحدة على الأقل كل ساعة، فور اكتمال المهمة | ![Shayan](../../!/tags/community-shayan.svg) |
| أشِر إلى [@claude](https://github.com/apps/claude) في PR زميل لتوليد قواعد lint تلقائياً لملاحظات المراجعة المتكررة — أتمتة نفسك خارج مراجعة الكود 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=2715) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=2715) |
| استخدم [/code-review](https://code.claude.com/docs/en/code-review) لتحليل PR متعدد الوكلاء — يكشف الأخطاء وثغرات الأمان والانحدارات قبل الدمج | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031089411820228645) |

<a id="tips-debugging"></a>■ **التصحيح (7)**

| النصيحة | المصدر |
|---------|--------|
| اجعل التقاط لقطات الشاشة ومشاركتها مع Claude عادة عند أي مشكلة | ![Shayan](../../!/tags/community-shayan.svg) |
| استخدم MCP ([Claude in Chrome](https://code.claude.com/docs/en/chrome)، [Playwright](https://github.com/microsoft/playwright-mcp)، [Chrome DevTools](https://developer.chrome.com/blog/chrome-devtools-mcp)) للسماح لـ Claude برؤية سجلات وحدة التحكم بنفسه | |
| اطلب دائماً من Claude تشغيل الطرفية (التي تريد رؤية سجلاتها) كمهمة خلفية للتصحيح الأفضل | |
| [/doctor](https://code.claude.com/docs/en/cli-reference) لتشخيص مشاكل التثبيت والمصادقة والتكوين | |
| خطأ أثناء الضغط يمكن حله باستخدام [/model](https://code.claude.com/docs/en/model-config) لاختيار نموذج بمليون رمز، ثم تشغيل [/compact](https://code.claude.com/docs/en/interactive-mode) | |
| استخدم [cross-model](../../development-workflows/cross-model-workflow/cross-model-workflow.md) لضمان الجودة — مثلاً [Codex](https://github.com/shanraisshan/codex-cli-best-practice) لمراجعة الخطة والتنفيذ | |
| البحث الوكيلي (glob + grep) يتفوق على RAG — Claude Code جرّب وتخلّى عن قواعد البيانات المتجهية لأن الكود ينحرف عن التزامن والصلاحيات معقدة | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3095) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3095) |

<a id="tips-utilities"></a>■ **أدوات مساعدة (5)**

| النصيحة | المصدر |
|---------|--------|
| طرفيات [iTerm](https://iterm2.com/)/[Ghostty](https://ghostty.org/)/[tmux](https://github.com/tmux/tmux) بدلاً من IDE ([VS Code](https://code.visualstudio.com/)/[Cursor](https://www.cursor.com/)) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742753971769626) |
| [/voice](https://code.claude.com/docs/en/voice-dictation) أو [Wispr Flow](https://wisprflow.ai) للأوامر الصوتية (إنتاجية 10 أضعاف) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038454362226467112) |
| [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks) لتقييم Claude | ![Shayan](../../!/tags/community-shayan.svg) |
| [status line](https://github.com/shanraisshan/claude-code-status-line) للوعي بالسياق والضغط السريع | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700784019452195) ![Shayan](../../!/tags/community-shayan.svg) |
| استكشف ميزات [settings.json](../../best-practice/claude-settings.md) مثل [Plans Directory](../../best-practice/claude-settings.md#plans-directory) و[Spinner Verbs](../../best-practice/claude-settings.md#display--ux) لتجربة مخصصة | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701145023197516) |

<a id="tips-daily"></a>■ **يومي (2)**

| النصيحة | المصدر |
|---------|--------|
| [حدّث](https://code.claude.com/docs/en/setup) Claude Code يومياً | ![Shayan](../../!/tags/community-shayan.svg) |
| ابدأ يومك بقراءة [سجل التغييرات](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) | ![Shayan](../../!/tags/community-shayan.svg) |

![Boris Cherny + Team](../../!/tags/claude.svg)

| المقال / التغريدة | المصدر |
|-------------------|--------|
| [15 ميزة مخفية وغير مستغلة في Claude Code (Boris) \| 30/مارس/26](../../tips/claude-boris-15-tips-30-mar-26.md) | [تغريدة](https://x.com/bcherny/status/2038454336355999749) |
| [Squash Merging وتوزيع حجم PR (Boris) \| 25/مارس/26](../../tips/claude-boris-2-tips-25-mar-26.md) | [تغريدة](https://x.com/bcherny/status/2038552880018538749) |
| [دروس من بناء Claude Code: كيف نستخدم Skills (Thariq) \| 17/مارس/26](../../tips/claude-thariq-tips-17-mar-26.md) | [مقال](https://x.com/trq212/status/2033949937936085378) |
| [Code Review وحساب وقت الاختبار (Boris) \| 10/مارس/26](../../tips/claude-boris-2-tips-10-mar-26.md) | [تغريدة](https://x.com/bcherny/status/2031089411820228645) |
| /loop — جدولة مهام متكررة لمدة تصل إلى 3 أيام (Boris) \| 07 مارس 2026 | [تغريدة](https://x.com/bcherny/status/2030193932404150413) |
| AskUserQuestion + ASCII Markdowns (Thariq) \| 28 فبراير 2026 | [تغريدة](https://x.com/trq212/status/2027543858289250472) |
| الرؤية كوكيل - دروس من بناء Claude Code (Thariq) \| 28 فبراير 2026 | [مقال](https://x.com/trq212/status/2027463795355095314) |
| Git Worktrees - 5 طرق يستخدمها Boris \| 21 فبراير 2026 | [تغريدة](https://x.com/bcherny/status/2025007393290272904) |
| دروس من بناء Claude Code: التخزين المؤقت للأوامر هو كل شيء (Thariq) \| 20 فبراير 2026 | [مقال](https://x.com/trq212/status/2024574133011673516) |
| [12 طريقة يخصص بها الناس Claude (Boris) \| 12/فبراير/26](../../tips/claude-boris-12-tips-12-feb-26.md) | [تغريدة](https://x.com/bcherny/status/2021699851499798911) |
| [10 نصائح لاستخدام Claude Code من الفريق (Boris) \| 01/فبراير/26](../../tips/claude-boris-10-tips-01-feb-26.md) | [تغريدة](https://x.com/bcherny/status/2017742741636321619) |
| [كيف أستخدم Claude Code — 13 نصيحة من إعدادي البسيط المفاجئ (Boris) \| 03/يناير/26](../../tips/claude-boris-13-tips-03-jan-26.md) | [تغريدة](https://x.com/bcherny/status/2007179832300581177) |
| اطلب من Claude إجراء مقابلة معك باستخدام أداة AskUserQuestion (Thariq) \| 28/ديسمبر/25 | [تغريدة](https://x.com/trq212/status/2005315275026260309) |
| استخدم دائماً plan mode، وفّر لـ Claude طريقة للتحقق، استخدم /code-review (Boris) \| 27/ديسمبر/25 | [تغريدة](https://x.com/bcherny/status/2004711722926616680) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

## 🎬 فيديوهات / بودكاست

| الفيديو / البودكاست | المصدر | YouTube |
|--------------------|--------|---------|
| كل ما أخطأنا فيه بشأن Research-Plan-Implement (Dex) \| 24 مارس 2026 \| MLOps Community | [![Dex](../../!/tags/community-dex.svg)](https://x.com/daborhyde) | [YouTube](https://youtu.be/YwZR6tc7qYg) |
| بناء Claude Code مع Boris Cherny (Boris) \| 04 مارس 2026 \| The Pragmatic Engineer | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/julbw1JuAz0) |
| رئيس Claude Code: ماذا بعد حل مشكلة البرمجة (Boris) \| 19 فبراير 2026 \| Lenny's Podcast | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/We7BZVKbCVw) |
| داخل Claude Code مع مبتكره Boris Cherny (Boris) \| 17 فبراير 2026 \| Y Combinator | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/PQU9o_5rHC4) |
| Boris Cherny (مبتكر Claude Code) عن ما طوّر مسيرته المهنية (Boris) \| 15 ديسمبر 2025 \| Ryan Peterman | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/AmdLVWMdjOk) |
| أسرار Claude Code من المهندسين الذين بنوه (Cat) \| 29 أكتوبر 2025 \| Every | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/IDSAMqip6ms) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

## 🔔 الاشتراك

| المصدر | الاسم | الشارة |
|--------|-------|--------|
| ![Reddit](https://img.shields.io/badge/-FF4500?style=flat&logo=reddit&logoColor=white) | [r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/), [r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/), [r/Anthropic](https://www.reddit.com/r/Anthropic/) | ![Boris + Team](../../!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Claude](https://x.com/claudeai), [Anthropic](https://x.com/AnthropicAI), [Boris](https://x.com/bcherny), [Thariq](https://x.com/trq212), [Cat](https://x.com/_catwu), [Lydia](https://x.com/lydiahallie), [Noah](https://x.com/noahzweben), [Anthony](https://x.com/amorriscode), [Alex](https://x.com/alexalbert__), [Kenneth](https://x.com/neilhtennek) | ![Boris + Team](../../!/tags/claude.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Anthropic](https://www.youtube.com/@anthropic-ai) | ![Boris + Team](../../!/tags/claude.svg) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

## ☠️ الشركات الناشئة / الأعمال

| Claude | استبدل |
|--------|--------|
|[**Code Review**](https://code.claude.com/docs/en/code-review)|[Greptile](https://greptile.com), [CodeRabbit](https://coderabbit.ai), [Devin Review](https://devin.ai), [OpenDiff](https://opendiff.com), [Cursor BugBot](https://bugbot.dev)|
|[**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation)|[Wispr Flow](https://wisprflow.ai), [SuperWhisper](https://superwhisper.com/)|
|[**Remote Control**](https://code.claude.com/docs/en/remote-control)|[OpenClaw](https://openclaw.ai/)|
|[**Claude in Chrome**](https://code.claude.com/docs/en/chrome)|[Playwright MCP](https://github.com/microsoft/playwright-mcp), [Chrome DevTools MCP](https://developer.chrome.com/blog/chrome-devtools-mcp)|
|[**Computer Use**](https://docs.anthropic.com/en/docs/agents-and-tools/computer-use)|[OpenAI CUA](https://openai.com/index/computer-using-agent/)|
|[**Cowork**](https://claude.com/blog/cowork-research-preview)|[ChatGPT Agent](https://openai.com/chatgpt/agent/), [Perplexity Computer](https://www.perplexity.ai/computer/), [Manus](https://manus.im)|
|[**Tasks**](https://x.com/trq212/status/2014480496013803643)|[Beads](https://github.com/steveyegge/beads)|
|[**Plan Mode**](https://code.claude.com/docs/en/common-workflows)|[Agent OS](https://github.com/buildermethods/agent-os)|
|[**Skills / Plugins**](https://code.claude.com/docs/en/plugins)|شركات YC الناشئة لأغلفة AI ([reddit](https://reddit.com/r/ClaudeAI/comments/1r6bh4d/claude_code_skills_are_basically_yc_ai_startup/))|

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

<a id="billion-dollar-questions"></a>
![Billion-Dollar Questions](../../!/tags/billion-dollar-questions.svg)

*ناقش هذه الأسئلة في [Issues](https://github.com/noya21th/ClaudeCode-BestPractice/issues)*

**الذاكرة والتعليمات (4)**

1. ما الذي يجب وضعه بالضبط في CLAUDE.md — وما الذي يجب تركه خارجه؟
2. إذا كان لديك CLAUDE.md بالفعل، هل أنت بحاجة فعلاً إلى constitution.md أو rules.md منفصل؟
3. كم مرة يجب تحديث CLAUDE.md، وكيف تعرف متى أصبح قديماً؟
4. لماذا لا يزال Claude يتجاهل تعليمات CLAUDE.md — حتى عندما تقول MUST بأحرف كبيرة؟ ([reddit](https://reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/))

**Agents وSkills وسير العمل (6)**

1. متى يجب استخدام command مقابل agent مقابل skill — ومتى يكون Claude Code العادي أفضل؟
2. كم مرة يجب تحديث agents وcommands وسير العمل مع تحسّن النماذج؟
3. هل إعطاء subagent شخصية مفصلة يحسّن الجودة؟ كيف يبدو "الأمر/الشخصية المثالية" لـ subagent بحث/ضمان جودة؟
4. هل يجب الاعتماد على plan mode المدمج في Claude Code — أم بناء command/agent تخطيط خاص يفرض سير عمل فريقك؟
5. إذا كان لديك skill شخصي (مثل /implement بأسلوب برمجتك)، كيف تدمج skills المجتمعية (مثل /simplify) بدون تعارضات — ومن يفوز عند الخلاف؟
6. هل وصلنا؟ هل يمكننا تحويل قاعدة كود موجودة إلى مواصفات، حذف الكود، وجعل AI يعيد توليد نفس الكود من تلك المواصفات وحدها؟

**المواصفات والتوثيق (3)**

1. هل يجب أن يكون لكل ميزة في مستودعك مواصفات كملف markdown؟
2. كم مرة تحتاج لتحديث المواصفات حتى لا تصبح قديمة عند تنفيذ ميزة جديدة؟
3. عند تنفيذ ميزة جديدة، كيف تتعامل مع التأثير المتتالي على مواصفات الميزات الأخرى؟

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

## التقارير

<p align="center">
  <a href="../../reports/claude-agent-sdk-vs-cli-system-prompts.md"><img src="https://img.shields.io/badge/Agent_SDK_vs_CLI-555?style=for-the-badge" alt="Agent SDK vs CLI"></a>
  <a href="../../reports/claude-in-chrome-v-chrome-devtools-mcp.md"><img src="https://img.shields.io/badge/Browser_Automation_MCP-555?style=for-the-badge" alt="Browser Automation MCP"></a>
  <a href="../../reports/claude-global-vs-project-settings.md"><img src="https://img.shields.io/badge/Global_vs_Project_Settings-555?style=for-the-badge" alt="Global vs Project Settings"></a>
  <a href="../../reports/claude-skills-for-larger-mono-repos.md"><img src="https://img.shields.io/badge/Skills_in_Monorepos-555?style=for-the-badge" alt="Skills in Monorepos"></a>
  <br>
  <a href="../../reports/claude-agent-memory.md"><img src="https://img.shields.io/badge/Agent_Memory-555?style=for-the-badge" alt="Agent Memory"></a>
  <a href="../../reports/claude-advanced-tool-use.md"><img src="https://img.shields.io/badge/Advanced_Tool_Use-555?style=for-the-badge" alt="Advanced Tool Use"></a>
  <a href="../../reports/claude-usage-and-rate-limits.md"><img src="https://img.shields.io/badge/Usage_&_Rate_Limits-555?style=for-the-badge" alt="Usage & Rate Limits"></a>
  <a href="../../reports/claude-agent-command-skill.md"><img src="https://img.shields.io/badge/Agents_vs_Commands_vs_Skills-555?style=for-the-badge" alt="Agents vs Commands vs Skills"></a>
  <br>
  <a href="../../reports/llm-day-to-day-degradation.md"><img src="https://img.shields.io/badge/LLM_Degradation-555?style=for-the-badge" alt="LLM Degradation"></a>
</p>

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

![How to Use](../../!/tags/how-to-use.svg)

```
1. اقرأ المستودع كدورة تعليمية، تعلّم ما هي commands وagents وskills وhooks قبل محاولة استخدامها.
2. انسخ هذا المستودع والعب بالأمثلة، جرّب /weather-orchestrator، استمع لأصوات hooks، شغّل فرق الوكلاء، لترى كيف تعمل الأشياء فعلاً.
3. اذهب لمشروعك الخاص واطلب من Claude اقتراح أفضل الممارسات من هذا المستودع التي يجب إضافتها، أعطه هذا المستودع كمرجع ليعرف ما هو ممكن.
```

<a href="https://www.youtube.com/watch?v=AkAhkalkRY4"><img src="../../!/thumbnail/video-1.png" alt="شاهد على YouTube" width="300"></a>

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="فاصل قسم" width="60" height="50">
</p>

## شكر وتقدير

مبني على [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice). شكراً لـ Shayan Rais على مساهمته في المصادر المفتوحة.

</div>
