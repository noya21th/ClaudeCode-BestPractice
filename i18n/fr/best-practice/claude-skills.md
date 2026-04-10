🌍 [English](../../../best-practice/claude-skills.md) | [中文](../../zh/best-practice/claude-skills.md) | [日本語](../../ja/best-practice/claude-skills.md) | **Français** | [Русский](../../ru/best-practice/claude-skills.md) | [العربية](../../ar/best-practice/claude-skills.md)

# Bonnes pratiques des Skills

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A33%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-skills-implementation.md)

Skills Claude Code — champs frontmatter et skills officiels integres.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Champs frontmatter (13)

| Champ | Type | Requis | Description |
|-------|------|----------|-------------|
| `name` | string | Non | Nom d'affichage et identifiant `/slash-command`. Utilise le nom du repertoire par defaut si omis |
| `description` | string | Recommande | Ce que fait le skill. Affiche dans l'autocompletion et utilise par Claude pour la decouverte automatique |
| `argument-hint` | string | Non | Indication affichee pendant l'autocompletion (ex. `[issue-number]`, `[filename]`) |
| `disable-model-invocation` | boolean | Non | Definir a `true` pour empecher Claude d'invoquer automatiquement ce skill |
| `user-invocable` | boolean | Non | Definir a `false` pour masquer du menu `/` — le skill devient uniquement une connaissance de fond, destine au prechargement par l'agent |
| `allowed-tools` | string | Non | Outils autorises sans demande de permission lorsque ce skill est actif |
| `model` | string | Non | Modele a utiliser lorsque ce skill s'execute (ex. `haiku`, `sonnet`, `opus`) |
| `effort` | string | Non | Remplace le niveau d'effort du modele lors de l'invocation (`low`, `medium`, `high`, `max`) |
| `context` | string | Non | Definir a `fork` pour executer le skill dans un contexte de subagent isole |
| `agent` | string | Non | Type de subagent quand `context: fork` est defini (defaut : `general-purpose`) |
| `hooks` | object | Non | Hooks de cycle de vie limites a ce skill |
| `paths` | string/list | Non | Patterns glob qui limitent l'activation automatique du skill. Accepte une chaine separee par des virgules ou une liste YAML — Claude charge le skill uniquement lorsqu'il travaille avec des fichiers correspondants |
| `shell` | string | Non | Shell pour les blocs `` !`command` `` — `bash` (defaut) ou `powershell`. Necessite `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Skill | Description |
|---|-------|-------------|
| 1 | `simplify` | Examine le code modifie pour la reutilisation, la qualite et l'efficacite — refactorise pour eliminer la duplication |
| 2 | `batch` | Execute des commandes sur plusieurs fichiers en masse |
| 3 | `debug` | Debogue les commandes en echec ou les problemes de code |
| 4 | `loop` | Execute un prompt ou une slash command sur un intervalle recurrent (jusqu'a 3 jours) |
| 5 | `claude-api` | Construire des apps avec l'API Claude ou le SDK Anthropic — se declenche sur les imports `anthropic` / `@anthropic-ai/sdk` |

Voir aussi : [Depot officiel de Skills](https://github.com/anthropics/skills/tree/main/skills) pour les skills installables maintenus par la communaute.

---

## Sources

- [Skills Claude Code — Documentation](https://code.claude.com/docs/en/skills)
- [Decouverte des Skills dans les Monorepos](../../../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
