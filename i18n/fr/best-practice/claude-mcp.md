🌍 [English](../../../best-practice/claude-mcp.md) | [中文](../../zh/best-practice/claude-mcp.md) | [日本語](../../ja/best-practice/claude-mcp.md) | **Français** | [Русский](../../ru/best-practice/claude-mcp.md) | [العربية](../../ar/best-practice/claude-mcp.md)

# Bonnes pratiques MCP Servers

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026%2012%3A30%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.mcp.json)

Les MCP (Model Context Protocol) servers etendent Claude Code avec des connexions vers des outils externes, bases de donnees et API. Ce guide couvre les serveurs recommandes pour un usage quotidien et les bonnes pratiques de configuration.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## MCP Servers pour un usage quotidien

> *"J'ai abuse avec 15 MCP servers en pensant que plus = mieux. J'ai fini par n'en utiliser que 4 au quotidien."* — [r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/) (682 upvotes)

| MCP Server | Fonction | Ressources |
|------------|-------------|-----------|
| [**Context7**](https://github.com/upstash/context7) | Recupere la documentation a jour des bibliotheques dans le contexte. Empeche les API hallucinees provenant de donnees d'entrainement obsoletes | [Reddit : "de loin le meilleur MCP pour le code"](https://reddit.com/r/mcp/comments/1qarjqm/) · [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | Automatisation du navigateur — implementez, testez et verifiez les fonctionnalites UI de maniere autonome. Captures d'ecran, navigation, test de formulaires | [Reddit : essentiel pour le frontend](https://reddit.com/r/mcp/comments/1m59pk0/) · [Docs](https://playwright.dev/) |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | Connecte Claude a votre vrai navigateur Chrome — inspectez la console, le reseau, le DOM. Debuguez ce que les utilisateurs voient reellement | [Reddit : "game changer" pour le debugging](https://reddit.com/r/mcp/comments/1qarjqm/5_mcps_that_have_genuinely_made_me_10x_faster/nza0i7t/) · [Rapport comparatif](../../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | Recupere de la documentation structuree de type wiki pour n'importe quel repo GitHub — architecture, surface API, relations | [Reddit : "mettez-le derriere une passerelle avec Context7"](https://reddit.com/r/mcp/comments/1qarjqm/) |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | Generez des diagrammes d'architecture, organigrammes et conceptions systeme sous forme de croquis Excalidraw dessines a la main a partir de prompts | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

Recherche (Context7/DeepWiki) -> Debug (Playwright/Chrome) -> Documentation (Excalidraw)

---

## Configuration

Les MCP servers sont configures dans `.mcp.json` a la racine du projet (portee projet) ou dans `~/.claude.json` (portee utilisateur).

### Types de serveurs

| Type | Transport | Exemple |
|------|-----------|---------|
| **stdio** | Lance un processus local | `npx`, `python`, binaire |
| **http** | Se connecte a une URL distante | point de terminaison HTTP/SSE |

### Exemple `.mcp.json`

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

Utilisez l'expansion des variables d'environnement pour les secrets au lieu de commiter des cles API dans `.mcp.json` :

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

### Parametres pour les MCP Servers

Ces parametres dans `.claude/settings.json` controlent l'approbation des MCP servers :

| Cle | Type | Description |
|-----|------|-------------|
| `enableAllProjectMcpServers` | boolean | Approuver automatiquement tous les serveurs `.mcp.json` sans demande |
| `enabledMcpjsonServers` | array | Liste blanche de noms de serveurs specifiques a approuver automatiquement |
| `disabledMcpjsonServers` | array | Liste noire de noms de serveurs specifiques a rejeter |

### Regles de permission pour les outils MCP

Les outils MCP suivent la convention de nommage `mcp__<server>__<tool>` dans les regles de permission :

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

## Portees MCP

Les MCP servers peuvent etre definis a trois niveaux :

| Portee | Emplacement | Objectif |
|-------|----------|---------|
| **Projet** | `.mcp.json` (racine du repo) | Serveurs partages par l'equipe, versiones dans git |
| **Utilisateur** | `~/.claude.json` (cle `mcpServers`) | Serveurs personnels pour tous les projets |
| **Subagent** | Frontmatter de l'agent (champ `mcpServers`) | Serveurs limites a un subagent specifique |

Priorite : Subagent > Projet > Utilisateur

---

## Sources

- [MCP Servers — Documentation Claude Code](https://code.claude.com/docs/en/mcp)
- [Specification Model Context Protocol](https://modelcontextprotocol.io/)
- [5 MCPs qui m'ont vraiment rendu 10x plus rapide — r/mcp](https://reddit.com/r/mcp/comments/1qarjqm/)
- [Discussion sur la surcharge de MCP Servers — r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)
