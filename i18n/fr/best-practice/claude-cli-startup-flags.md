🌍 [English](../../../best-practice/claude-cli-startup-flags.md) | [中文](../../zh/best-practice/claude-cli-startup-flags.md) | [日本語](../../ja/best-practice/claude-cli-startup-flags.md) | **Français** | [Русский](../../ru/best-practice/claude-cli-startup-flags.md) | [العربية](../../ar/best-practice/claude-cli-startup-flags.md)

# Bonnes pratiques des CLI Startup Flags

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026-white?style=flat&labelColor=555)

Reference pour les drapeaux de demarrage de Claude Code, les sous-commandes de niveau superieur et les variables d'environnement de demarrage lors du lancement de Claude Code depuis le terminal.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Table des matieres

1. [Gestion de session](#gestion-de-session)
2. [Modele et configuration](#modele--configuration)
3. [Permissions et securite](#permissions--securite)
4. [Sortie et format](#sortie--format)
5. [System Prompt](#system-prompt)
6. [Agent et Subagent](#agent--subagent)
7. [MCP et Plugins](#mcp--plugins)
8. [Repertoire et espace de travail](#repertoire--espace-de-travail)
9. [Budget et limites](#budget--limites)
10. [Integration](#integration)
11. [Initialisation et maintenance](#initialisation--maintenance)
12. [Debug et diagnostics](#debug--diagnostics)
13. [Remplacement de Settings](#remplacement-de-settings)
14. [Version et aide](#version--aide)
15. [Sous-commandes](#sous-commandes)
16. [Variables d'environnement](#variables-denvironnement)

---

## Gestion de session

| Drapeau | Court | Description |
|------|-------|-------------|
| `--continue` | `-c` | Continuer la conversation la plus recente dans le repertoire actuel |
| `--resume` | `-r` | Reprendre une session specifique par ID ou nom, ou afficher le selecteur interactif |
| `--from-pr <NUMBER\|URL>` | | Reprendre les sessions liees a une PR GitHub specifique |
| `--fork-session` | | Creer un nouvel ID de session lors de la reprise (utiliser avec `--resume` ou `--continue`) |
| `--session-id <UUID>` | | Utiliser un ID de session specifique (doit etre un UUID valide) |
| `--no-session-persistence` | | Desactiver la persistance de session (mode print uniquement) |
| `--remote` | | Creer une nouvelle session web sur claude.ai |
| `--teleport` | | Reprendre une session web dans votre terminal local |

---

## Modele et configuration

| Drapeau | Court | Description |
|------|-------|-------------|
| `--model <NAME>` | | Definir le modele avec un alias (`sonnet`, `opus`, `haiku`) ou un identifiant complet |
| `--fallback-model <NAME>` | | Modele de secours automatique quand le modele par defaut est surcharge (mode print uniquement) |
| `--betas <LIST>` | | En-tetes beta a inclure dans les requetes API (utilisateurs de cle API uniquement) |

---

## Permissions et securite

| Drapeau | Court | Description |
|------|-------|-------------|
| `--dangerously-skip-permissions` | | Ignorer TOUTES les demandes de permission. A utiliser avec une extreme prudence |
| `--allow-dangerously-skip-permissions` | | Activer le contournement des permissions comme option sans l'activer |
| `--permission-mode <MODE>` | | Demarrer dans le mode de permission specifie : `default`, `plan`, `acceptEdits`, `bypassPermissions` |
| `--allowedTools <TOOLS>` | | Outils qui s'executent sans demande (syntaxe de regle de permission) |
| `--disallowedTools <TOOLS>` | | Outils retires entierement du contexte du modele |
| `--tools <TOOLS>` | | Restreindre les outils integres que Claude peut utiliser (utilisez `""` pour tout desactiver) |
| `--permission-prompt-tool <TOOL>` | | Specifier un outil MCP pour gerer les demandes de permission en mode non-interactif |

---

## Sortie et format

| Drapeau | Court | Description |
|------|-------|-------------|
| `--print` | `-p` | Imprimer la reponse sans mode interactif (mode headless/SDK) |
| `--output-format <FORMAT>` | | Format de sortie : `text`, `json`, `stream-json` |
| `--input-format <FORMAT>` | | Format d'entree : `text`, `stream-json` |
| `--json-schema <SCHEMA>` | | Obtenir du JSON valide correspondant au schema (mode print uniquement) |
| `--include-partial-messages` | | Inclure les evenements de streaming partiels (necessite `--print` et `--output-format=stream-json`) |
| `--verbose` | | Activer le logging verbeux avec la sortie complete tour par tour |

---

## System Prompt

| Drapeau | Court | Description |
|------|-------|-------------|
| `--system-prompt <TEXT>` | | Remplacer tout le system prompt par un texte personnalise |
| `--system-prompt-file <PATH>` | | Charger le system prompt depuis un fichier, remplacant le defaut (mode print uniquement) |
| `--append-system-prompt <TEXT>` | | Ajouter du texte personnalise au system prompt par defaut |
| `--append-system-prompt-file <PATH>` | | Ajouter le contenu d'un fichier au prompt par defaut (mode print uniquement) |

---

## Agent et Subagent

| Drapeau | Court | Description |
|------|-------|-------------|
| `--agent <NAME>` | | Specifier un agent pour la session en cours |
| `--agents <JSON>` | | Definir des subagents personnalises dynamiquement via JSON |
| `--teammate-mode <MODE>` | | Definir l'affichage de l'equipe d'agents : `auto`, `in-process`, `tmux` |

---

## MCP et Plugins

| Drapeau | Court | Description |
|------|-------|-------------|
| `--mcp-config <PATH\|JSON>` | | Charger les MCP servers depuis un fichier JSON ou une chaine |
| `--strict-mcp-config` | | Utiliser uniquement les MCP servers de `--mcp-config`, ignorer tous les autres |
| `--plugin-dir <PATH>` | | Charger les plugins depuis un repertoire pour cette session uniquement (repetable) |

---

## Repertoire et espace de travail

| Drapeau | Court | Description |
|------|-------|-------------|
| `--add-dir <PATH>` | | Ajouter des repertoires de travail supplementaires pour l'acces de Claude |
| `--worktree` | `-w` | Demarrer Claude dans un git worktree isole (branche depuis HEAD) |

---

## Budget et limites

| Drapeau | Court | Description |
|------|-------|-------------|
| `--max-budget-usd <AMOUNT>` | | Montant maximum en dollars pour les appels API avant l'arret (mode print uniquement) |
| `--max-turns <NUMBER>` | | Limiter le nombre de tours agentiques (mode print uniquement) |

---

## Integration

| Drapeau | Court | Description |
|------|-------|-------------|
| `--chrome` | | Activer l'integration du navigateur Chrome pour l'automatisation web |
| `--no-chrome` | | Desactiver l'integration du navigateur Chrome pour cette session |
| `--ide` | | Se connecter automatiquement a l'IDE au demarrage si exactement un IDE valide est disponible |

---

## Initialisation et maintenance

| Drapeau | Court | Description |
|------|-------|-------------|
| `--init` | | Executer les hooks d'initialisation et demarrer le mode interactif |
| `--init-only` | | Executer les hooks d'initialisation et quitter (pas de session interactive) |
| `--maintenance` | | Executer les hooks de maintenance et quitter |

---

## Debug et diagnostics

| Drapeau | Court | Description |
|------|-------|-------------|
| `--debug <CATEGORIES>` | | Activer le mode debug avec un filtrage optionnel par categorie (ex. `"api,hooks"`) |

---

## Remplacement de Settings

| Drapeau | Court | Description |
|------|-------|-------------|
| `--settings <PATH\|JSON>` | | Chemin vers un fichier JSON de settings ou chaine JSON a charger |
| `--setting-sources <LIST>` | | Liste separee par des virgules des sources a charger : `user`, `project`, `local` |
| `--disable-slash-commands` | | Desactiver tous les skills et slash commands pour cette session |

---

## Version et aide

| Drapeau | Court | Description |
|------|-------|-------------|
| `--version` | `-v` | Afficher le numero de version |
| `--help` | `-h` | Afficher l'aide |

---

## Sous-commandes

Ce sont des commandes de niveau superieur executees sous forme `claude <sous-commande>` :

| Sous-commande | Description |
|------------|-------------|
| `claude` | Demarrer le REPL interactif |
| `claude "requete"` | Demarrer le REPL avec un prompt initial |
| `claude agents` | Lister les agents configures |
| `claude auth` | Gerer l'authentification Claude Code |
| `claude doctor` | Executer les diagnostics depuis la ligne de commande |
| `claude install` | Installer ou changer les builds natifs de Claude Code |
| `claude mcp` | Configurer les MCP servers (`add`, `remove`, `list`, `get`, `enable`) |
| `claude plugin` | Gerer les plugins Claude Code |
| `claude remote-control` | Gerer les sessions de controle a distance |
| `claude setup-token` | Creer un token longue duree pour l'utilisation de l'abonnement |
| `claude update` / `claude upgrade` | Mettre a jour vers la derniere version |

---

## Variables d'environnement

Ces variables d'environnement de demarrage uniquement sont definies dans votre shell avant de lancer Claude Code (elles ne peuvent pas etre configurees via `settings.json`) :

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | Activer les agent teams experimentales |
| `CLAUDE_CODE_TMPDIR` | Remplacer le repertoire temporaire pour les fichiers internes. Aussi configurable via la cle `env` — voir [Reference Settings](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | Activer le chargement de CLAUDE.md des repertoires supplementaires |
| `DISABLE_AUTOUPDATER=1` | Desactiver les mises a jour automatiques |
| `CLAUDE_CODE_EFFORT_LEVEL` | Controler la profondeur de reflexion — voir [Reference Settings](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `USE_BUILTIN_RIPGREP=0` | Utiliser le ripgrep systeme au lieu de l'integre (Alpine Linux) |
| `CLAUDE_CODE_SIMPLE` | Activer le mode simple (outils Bash + Edit uniquement). Aussi configurable via la cle `env` — voir [Reference Settings](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_BASH_NO_LOGIN=1` | Ignorer le shell de connexion pour BashTool |

Pour les variables d'environnement configurables via la cle `"env"` dans `settings.json` (incluant `MAX_THINKING_TOKENS`, `CLAUDE_CODE_SHELL`, `CLAUDE_CODE_ENABLE_TASKS`, `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`, `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`, et plus), voir la [Reference Settings Claude](../../../best-practice/claude-settings.md#environment-variables-via-env).

---

## Sources

- [Reference CLI Claude Code](https://code.claude.com/docs/en/cli-reference)
- [Mode Headless Claude Code](https://code.claude.com/docs/en/headless)
- [Configuration Claude Code](https://code.claude.com/docs/en/setup)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Workflows courants Claude Code](https://code.claude.com/docs/en/common-workflows)
