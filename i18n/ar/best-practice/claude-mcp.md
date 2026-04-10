<!-- dir: rtl -->

<div dir="rtl">

🌍 [English](../../../best-practice/claude-mcp.md) | [中文](../../zh/best-practice/claude-mcp.md) | [日本語](../../ja/best-practice/claude-mcp.md) | [Français](../../fr/best-practice/claude-mcp.md) | [Русский](../../ru/best-practice/claude-mcp.md) | **العربية**

# أفضل ممارسات MCP Servers

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026%2012%3A30%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.mcp.json)

خوادم MCP (Model Context Protocol) توسّع Claude Code باتصالات بأدوات خارجية وقواعد بيانات وواجهات برمجية. يغطي هذا الدليل الخوادم الموصى بها للاستخدام اليومي وأفضل ممارسات التكوين.

<table width="100%">
<tr>
<td><a href="../../ar/README.md">→ العودة إلى أفضل ممارسات Claude Code</a></td>
<td align="left"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## خوادم MCP للاستخدام اليومي

> *"بالغت بـ 15 خادم MCP ظناً أن الأكثر = الأفضل. انتهى بي الأمر باستخدام 4 فقط يومياً."* — [r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/) (682 تصويت إيجابي)

| خادم MCP | ما يفعله | الموارد |
|----------|----------|---------|
| [**Context7**](https://github.com/upstash/context7) | يجلب وثائق المكتبات المحدّثة في السياق. يمنع واجهات برمجية متخيّلة من بيانات تدريب قديمة | [Reddit: "أفضل MCP للبرمجة بلا منازع"](https://reddit.com/r/mcp/comments/1qarjqm/) · [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | أتمتة المتصفح — تنفيذ واختبار والتحقق من ميزات واجهة المستخدم بشكل مستقل. لقطات شاشة وتنقل واختبار نماذج | [Reddit: أساسي للواجهة الأمامية](https://reddit.com/r/mcp/comments/1m59pk0/) · [الوثائق](https://playwright.dev/) |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | يربط Claude بمتصفح Chrome الحقيقي — فحص وحدة التحكم والشبكة وDOM. تصحيح ما يراه المستخدمون فعلاً | [Reddit: "محوّل اللعبة" للتصحيح](https://reddit.com/r/mcp/comments/1qarjqm/5_mcps_that_have_genuinely_made_me_10x_faster/nza0i7t/) · [تقرير المقارنة](../../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | يجلب وثائق بنمط wiki منظمة لأي مستودع GitHub — البنية وسطح API والعلاقات | [Reddit: "ضعه خلف بوابة مع Context7"](https://reddit.com/r/mcp/comments/1qarjqm/) |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | إنشاء مخططات بنية ومخططات تدفق وتصاميم أنظمة كرسومات Excalidraw يدوية من الأوامر | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

بحث (Context7/DeepWiki) -> تصحيح (Playwright/Chrome) -> توثيق (Excalidraw)

---

## التكوين

تُكوّن خوادم MCP في `.mcp.json` في جذر المشروع (نطاق المشروع) أو في `~/.claude.json` (نطاق المستخدم).

### أنواع الخوادم

| النوع | النقل | مثال |
|-------|-------|------|
| **stdio** | يُنشئ عملية محلية | `npx`، `python`، ملف تنفيذي |
| **http** | يتصل بعنوان URL بعيد | نقطة نهاية HTTP/SSE |

### مثال على `.mcp.json`

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

استخدم توسيع متغيرات البيئة للأسرار بدلاً من إيداع مفاتيح API في `.mcp.json`:

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

### إعدادات خوادم MCP

هذه الإعدادات في `.claude/settings.json` تتحكم في الموافقة على خوادم MCP:

| المفتاح | النوع | الوصف |
|---------|-------|-------|
| `enableAllProjectMcpServers` | boolean | الموافقة التلقائية على جميع خوادم `.mcp.json` بدون سؤال |
| `enabledMcpjsonServers` | array | قائمة سماح بأسماء خوادم محددة للموافقة التلقائية |
| `disabledMcpjsonServers` | array | قائمة حظر بأسماء خوادم محددة للرفض |

### قواعد الصلاحيات لأدوات MCP

أدوات MCP تتبع اصطلاح تسمية `mcp__<server>__<tool>` في قواعد الصلاحيات:

```json
{
  "permissions": {
    "allow": [
      "mcp__*",
      "mcp__context7__*",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

---

## نطاقات MCP

يمكن تعريف خوادم MCP على ثلاثة مستويات:

| النطاق | الموقع | الغرض |
|--------|--------|-------|
| **المشروع** | `.mcp.json` (جذر المستودع) | خوادم مشتركة مع الفريق، تُودع في git |
| **المستخدم** | `~/.claude.json` (مفتاح `mcpServers`) | خوادم شخصية عبر جميع المشاريع |
| **الوكيل الفرعي** | frontmatter الوكيل (حقل `mcpServers`) | خوادم محصورة في وكيل فرعي محدد |

الأولوية: الوكيل الفرعي > المشروع > المستخدم

---

## المصادر

- [خوادم MCP — وثائق Claude Code](https://code.claude.com/docs/en/mcp)
- [مواصفات Model Context Protocol](https://modelcontextprotocol.io/)
- [5 MCPs جعلتني أسرع 10 أضعاف فعلاً — r/mcp](https://reddit.com/r/mcp/comments/1qarjqm/)
- [نقاش حول تحميل خوادم MCP الزائد — r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)

</div>
