🌍 [English](../../../best-practice/claude-hooks.md) | [中文](../../zh/best-practice/claude-hooks.md) | [日本語](../../ja/best-practice/claude-hooks.md) | **Français** | [Русский](../../ru/best-practice/claude-hooks.md) | [العربية](../../ar/best-practice/claude-hooks.md)

# Bonnes pratiques des Hooks

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Hooks Claude Code — gestionnaires pilotes par les evenements qui s'executent en dehors de la boucle agentique sur des evenements specifiques du cycle de vie.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Vue d'ensemble

Les hooks sont des gestionnaires definis par l'utilisateur qui s'executent **en dehors** de la boucle agentique — ils ne font pas partie du raisonnement ou de la chaine d'appels d'outils de Claude. Ils se declenchent sur des evenements specifiques du cycle de vie et peuvent executer des scripts, injecter des prompts, lancer des agents ou appeler des points de terminaison HTTP. Utilisez les hooks pour ajouter de l'observabilite, appliquer des politiques, automatiser les effets de bord et controler les decisions sans polluer le contexte de conversation.

---

## Evenements de Hook (27)

### Cycle de vie de session

| Evenement | Se declenche quand |
|-------|-----------|
| `SessionStart` | Une nouvelle session interactive commence |
| `SessionEnd` | La session est sur le point de se fermer |
| `Setup` | Premiere execution de Claude Code dans un projet (ou apres `/init`) |

### Cycle de vie des outils

| Evenement | Se declenche quand |
|-------|-----------|
| `PreToolUse` | Avant l'execution d'un appel d'outil. Peut bloquer, autoriser ou modifier l'appel |
| `PostToolUse` | Apres la reussite d'un appel d'outil |
| `PostToolUseFailure` | Apres l'echec d'un appel d'outil avec une erreur |
| `PermissionRequest` | Quand Claude demande la permission pour un outil non pre-approuve par l'utilisateur |
| `PermissionDenied` | Quand une demande de permission d'outil est refusee par l'utilisateur ou une politique |

### Cycle de vie des agents

| Evenement | Se declenche quand |
|-------|-----------|
| `SubagentStart` | Un subagent (via l'outil `Agent(...)`) commence son execution |
| `SubagentStop` | Un subagent termine son execution |
| `Stop` | L'agent **principal** termine son tour de reponse |
| `StopFailure` | Le tour de reponse de l'agent principal echoue avec une erreur |

### Taches / Equipes

| Evenement | Se declenche quand |
|-------|-----------|
| `TeammateIdle` | Un coequipier dans une session Agent Teams devient inactif et disponible |
| `TaskCreated` | Une tache en arriere-plan est creee |
| `TaskCompleted` | Une tache en arriere-plan se termine |

### Contexte

| Evenement | Se declenche quand |
|-------|-----------|
| `PreCompact` | Avant l'execution de la compaction du contexte |
| `PostCompact` | Apres la compaction du contexte |
| `UserPromptSubmit` | L'utilisateur soumet un prompt (avant que Claude ne le traite) |
| `Notification` | Claude envoie une notification (ex. tache terminee, permission necessaire) |

### Configuration / Environnement

| Evenement | Se declenche quand |
|-------|-----------|
| `ConfigChange` | Un fichier de settings est modifie |
| `CwdChanged` | Le repertoire de travail change (ex. via `/add-dir`) |
| `FileChanged` | Un fichier surveille est modifie sur le disque |
| `InstructionsLoaded` | Les fichiers CLAUDE.md ou rules sont charges en contexte |

### Worktree

| Evenement | Se declenche quand |
|-------|-----------|
| `WorktreeCreate` | Un git worktree est cree pour un agent isole |
| `WorktreeRemove` | Un git worktree est nettoye |

### MCP / Elicitation

| Evenement | Se declenche quand |
|-------|-----------|
| `Elicitation` | Claude a besoin d'une entree structuree de l'utilisateur via une elicitation MCP |
| `ElicitationResult` | L'utilisateur repond a une elicitation MCP |

---

## Types de Hook (4)

| Type | Syntaxe | Description |
|------|--------|-------------|
| `command` | `{"type": "command", "command": "python3 script.py"}` | Execute une commande shell. Reçoit les donnees de l'evenement via des variables d'environnement. La sortie stdout est reinjectee dans le contexte (pour `PreToolUse`/`Stop`). Un code de sortie non nul bloque l'action (pour `PreToolUse`) |
| `prompt` | `{"type": "prompt", "prompt": "Verifie ceci pour les problemes de securite"}` | Injecte un prompt dans le contexte de Claude au point du hook. Utile pour ajouter des instructions ou des contraintes |
| `agent` | `{"type": "agent", "agent": "security-reviewer"}` | Lance un subagent pour gerer l'evenement. L'agent s'execute dans son propre contexte et peut utiliser des outils |
| `http` | `{"type": "http", "url": "https://example.com/webhook"}` | Envoie les donnees de l'evenement a un point de terminaison HTTP via POST. Le corps de la reponse est reinjecte dans le contexte |

---

## Hierarchie de configuration

Les hooks peuvent etre definis a plusieurs niveaux. Les niveaux superieurs remplacent les niveaux inferieurs pour le meme evenement :

| Niveau | Emplacement | Portee |
|-------|----------|-------|
| **Manage** | `managed-settings.json` (MDM / Registry) | Impose par l'organisation, ne peut pas etre remplace |
| **Settings locaux** | `.claude/settings.local.json` | Remplacements personnels du projet (git-ignore) |
| **Settings partages** | `.claude/settings.json` | Hooks du projet partages par l'equipe |
| **Settings globaux** | `~/.claude/settings.json` | Valeurs par defaut personnelles pour tous les projets |
| **Frontmatter d'agent** | `.claude/agents/<name>.md` champ `hooks:` | Limite a un subagent specifique |
| **Frontmatter de skill** | `.claude/skills/<name>/SKILL.md` champ `hooks:` | Limite a un skill specifique |

Pour desactiver tous les hooks, definissez `"disableAllHooks": true` dans `.claude/settings.local.json`.

---

## Syntaxe des matchers

Les matchers filtrent quelles invocations d'outils declenchent un hook. Ils s'appliquent aux evenements `PreToolUse`, `PostToolUse` et `PostToolUseFailure`.

| Matcher | Correspond a |
|---------|---------|
| `"Bash"` | Tous les appels d'outil Bash |
| `"Read"` | Tous les appels d'outil Read |
| `"Write"` | Tous les appels d'outil Write |
| `"Edit"` | Tous les appels d'outil Edit |
| `"Glob"` | Tous les appels d'outil Glob |
| `"Grep"` | Tous les appels d'outil Grep |
| `"Agent"` | Tous les appels d'outil Agent (subagent) |
| `"Skill"` | Tous les appels d'outil Skill |
| `"WebFetch"` | Tous les appels d'outil WebFetch |
| `"WebSearch"` | Tous les appels d'outil WebSearch |
| `"mcp__<server>__<tool>"` | Un outil MCP specifique sur un serveur specifique |
| `"*"` | Tous les appels d'outils (wildcard) |

Plusieurs matchers peuvent etre combines dans un tableau : `["Bash", "Write", "Edit"]`.

---

## Variables d'environnement

Les hooks de type command reçoivent les donnees de l'evenement via des variables d'environnement :

| Variable | Disponible dans | Description |
|----------|-------------|-------------|
| `CLAUDE_HOOK_EVENT` | Tous | Le nom de l'evenement (ex. `PreToolUse`, `Stop`) |
| `CLAUDE_HOOK_TOOL_NAME` | Evenements d'outils | L'outil appele (ex. `Bash`, `Write`) |
| `CLAUDE_HOOK_TOOL_INPUT` | `PreToolUse` | Chaine JSON des parametres d'entree de l'outil |
| `CLAUDE_HOOK_TOOL_OUTPUT` | `PostToolUse` | Chaine JSON de la sortie de l'outil |
| `CLAUDE_HOOK_ERROR` | Evenements d'echec | Message d'erreur de l'operation echouee |
| `CLAUDE_HOOK_SESSION_ID` | Tous | Identifiant de la session en cours |
| `CLAUDE_HOOK_CWD` | Tous | Repertoire de travail actuel |
| `CLAUDE_HOOK_PROJECT_DIR` | Tous | Repertoire racine du projet |
| `CLAUDE_HOOK_AGENT_NAME` | Evenements d'agents | Nom du subagent concerne |
| `CLAUDE_HOOK_MATCHED` | Evenements d'outils | Le matcher qui a declenche ce hook |

---

## Exemples pratiques

### Notification sonore a la fin d'une tache

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

### Journalisation de tous les appels d'outils

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

### Audit de securite — Bloquer les commandes dangereuses

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

Le script verifie `CLAUDE_HOOK_TOOL_INPUT` pour les patterns dangereux (`rm -rf /`, `curl | bash`, `chmod 777`). Un code de sortie non nul bloque l'appel d'outil.

### Auto-formatage a l'ecriture de fichiers

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

## Patterns de controle de decision

Les hooks `PreToolUse` peuvent retourner des signaux de decision via stdout ou le code de sortie :

| Pattern | Mecanisme | Effet |
|---------|-----------|--------|
| **Allow** | Code de sortie `0`, stdout contient `ALLOW` | L'appel d'outil se poursuit sans demande de permission |
| **Deny** | Code de sortie non nul | L'appel d'outil est bloque, message d'erreur affiche |
| **Ask** | Code de sortie `0`, stdout contient `ASK` | La demande de permission standard est presentee a l'utilisateur |
| **Defer** | Code de sortie `0`, pas de sortie ou stdout vide | Delegation au traitement de permission par defaut |

Pour les hooks de type `prompt` et `agent`, la reponse injectee guide le comportement de Claude plutot que de controler directement l'execution.

---

## Bonnes pratiques

1. **Utilisez `async: true` pour les effets de bord** — Les notifications sonores, la journalisation et la telemetrie ne devraient jamais bloquer la boucle agentique. Definissez `"async": true` sur les hooks qui n'ont pas besoin d'influencer le comportement de Claude.

2. **Utilisez `once: true` pour les hooks de session** — Les hooks `SessionStart` qui effectuent une configuration (installation de dependances, verification d'environnement) ne devraient s'executer qu'une fois par session.

3. **Gardez les timeouts courts** — Les hooks synchrones bloquent la reponse de Claude. Si votre hook prend plus de 2-3 secondes, rendez-le asynchrone ou optimisez le script.

4. **Ciblez les hooks avec des matchers** — Evitez d'executer des hooks sur chaque appel d'outil quand vous n'avez besoin que d'outils specifiques. Utilisez des matchers pour reduire la surcharge.

5. **Utilisez `command` pour le controle, `prompt` pour le guidage** — Quand vous avez besoin d'un controle strict allow/deny, utilisez un hook command avec des codes de sortie. Quand vous voulez influencer l'approche de Claude, utilisez un hook prompt.

6. **Testez les hooks en isolation** — Executez vos scripts de hooks manuellement avec des variables d'environnement d'exemple avant de les ajouter aux settings.

7. **Journalisez les echecs de hooks** — Enveloppez les commandes de hooks dans une logique de gestion d'erreurs qui ecrit dans un fichier journal, pour que les echecs silencieux soient detectables.

---

## Pieges courants

| Piege | Details |
|---------|---------|
| **Confusion `Stop` vs `SubagentStop`** | `Stop` se declenche uniquement pour l'agent **principal**. Utilisez `SubagentStop` pour les evenements de completion de subagent. Dans les versions anterieures, `Stop` s'appelait `AgentStop` — il a ete renomme |
| **Surcharge de creation de processus** | Chaque hook `command` lance un nouveau processus shell. Pour les evenements a haute frequence comme `PostToolUse` avec un matcher wildcard, cela ajoute de la latence. Envisagez d'utiliser `async: true` ou de consolider la logique dans un seul script |
| **Oubli du matcher** | Un hook `PreToolUse` sans champ `matcher` se declenche pour **tous** les appels d'outils. Definissez toujours un matcher sauf si vous voulez intentionnellement une couverture universelle |
| **Hooks synchrones bloquant les reponses** | Un hook synchrone lent sur `Stop` retarde l'affichage de la reponse de Claude pour l'utilisateur. Rendez toujours les hooks non critiques asynchrones |
| **Sortie de hook polluant le contexte** | La sortie stdout des hooks command synchrones est injectee dans le contexte de Claude. Gardez la sortie minimale ou redirigez vers un fichier journal |
| **Surprises de hierarchie de settings** | Un hook defini dans `.claude/settings.json` peut etre remplace par `.claude/settings.local.json`. Verifiez les deux fichiers lors du debug d'un hook qui ne se declenche pas |

---

## Sources

- [Hooks Claude Code — Documentation](https://code.claude.com/docs/en/hooks)
- [Guide des Hooks Claude Code](https://code.claude.com/docs/en/hooks-guide)
- [Settings Claude Code — Documentation](https://code.claude.com/docs/en/settings)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
