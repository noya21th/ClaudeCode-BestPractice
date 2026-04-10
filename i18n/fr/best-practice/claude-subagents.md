🌍 [English](../../../best-practice/claude-subagents.md) | [中文](../../zh/best-practice/claude-subagents.md) | [日本語](../../ja/best-practice/claude-subagents.md) | **Français** | [Русский](../../ru/best-practice/claude-subagents.md) | [العربية](../../ar/best-practice/claude-subagents.md)

# Bonnes pratiques des Sub-agents

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A34%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-subagents-implementation.md)

Subagents Claude Code — champs frontmatter et types d'agents officiels integres.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Champs frontmatter (16)

| Champ | Type | Requis | Description |
|-------|------|----------|-------------|
| `name` | string | Oui | Identifiant unique utilisant des lettres minuscules et des tirets |
| `description` | string | Oui | Quand invoquer. Utilisez `"PROACTIVELY"` pour l'invocation automatique par Claude |
| `tools` | string/list | Non | Liste blanche d'outils separee par des virgules (ex. `Read, Write, Edit, Bash`). Herite de tous les outils si omis. Supporte la syntaxe `Agent(agent_type)` pour restreindre les subagents invocables ; l'ancien alias `Task(agent_type)` fonctionne toujours |
| `disallowedTools` | string/list | Non | Outils a refuser, retires de la liste heritee ou specifiee |
| `model` | string | Non | Modele a utiliser : `sonnet`, `opus`, `haiku`, un identifiant complet (ex. `claude-opus-4-6`), ou `inherit` (defaut : `inherit`) |
| `permissionMode` | string | Non | Mode de permission : `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, ou `plan` |
| `maxTurns` | integer | Non | Nombre maximum de tours agentiques avant l'arret du subagent |
| `skills` | list | Non | Noms des skills a precharger dans le contexte de l'agent au demarrage (contenu complet injecte, pas seulement rendu disponible) |
| `mcpServers` | list | Non | MCP servers pour ce subagent — chaines de noms de serveurs ou objets inline `{name: config}` |
| `hooks` | object | Non | Hooks de cycle de vie limites a ce subagent. Tous les evenements de hook sont supportes ; `PreToolUse`, `PostToolUse` et `Stop` sont les plus courants |
| `memory` | string | Non | Portee de la memoire persistante : `user`, `project`, ou `local` |
| `background` | boolean | Non | Definir a `true` pour toujours executer en tache de fond (defaut : `false`) |
| `effort` | string | Non | Niveau d'effort remplace quand ce subagent est actif : `low`, `medium`, `high`, `max` (Opus 4.6 uniquement). Defaut : herite de la session |
| `isolation` | string | Non | Definir a `"worktree"` pour executer dans un git worktree temporaire (nettoye automatiquement si aucun changement) |
| `initialPrompt` | string | Non | Soumis automatiquement comme premier tour utilisateur lorsque cet agent s'execute comme agent de session principal (via `--agent` ou le parametre `agent`). Les commands et skills sont traites. Ajoute avant tout prompt fourni par l'utilisateur |
| `color` | string | Non | Couleur d'affichage pour le subagent dans la liste des taches et la transcription : `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, ou `cyan` |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Agent | Modele | Outils | Description |
|---|-------|-------|-------|-------------|
| 1 | `general-purpose` | inherit | Tous | Taches complexes multi-etapes — le type d'agent par defaut pour la recherche, la recherche de code et le travail autonome |
| 2 | `Explore` | haiku | Lecture seule (pas de Write, Edit) | Recherche et exploration rapide du codebase — optimise pour trouver des fichiers, rechercher du code et repondre aux questions sur le codebase |
| 3 | `Plan` | inherit | Lecture seule (pas de Write, Edit) | Recherche de pre-planification en plan mode — explore le codebase et conçoit des approches d'implementation avant d'ecrire du code |
| 4 | `statusline-setup` | sonnet | Read, Edit | Configure le parametre status line de Claude Code de l'utilisateur |
| 5 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | Repond aux questions sur les fonctionnalites de Claude Code, l'Agent SDK et l'API Claude |

---

## Sources

- [Creer des subagents personnalises — Documentation Claude Code](https://code.claude.com/docs/en/sub-agents)
- [Reference CLI — Documentation Claude Code](https://code.claude.com/docs/en/cli-reference)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
