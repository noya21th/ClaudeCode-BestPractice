🌍 [English](../../../best-practice/claude-settings.md) | [中文](../../zh/best-practice/claude-settings.md) | [日本語](../../ja/best-practice/claude-settings.md) | **Français** | [Русский](../../ru/best-practice/claude-settings.md) | [العربية](../../ar/best-practice/claude-settings.md)

# Bonnes pratiques des Settings

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%2010%3A16%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.claude/settings.json)

Un guide complet de toutes les options de configuration disponibles dans les fichiers `settings.json` de Claude Code. Depuis la v2.1.96, Claude Code expose **plus de 60 settings** et **plus de 170 variables d'environnement** (utilisez le champ `"env"` dans `settings.json` pour eviter les scripts wrapper).

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## Table des matieres

1. [Hierarchie des settings](#hierarchie-des-settings)
2. [Configuration de base](#configuration-de-base)
3. [Permissions](#permissions)
4. [Hooks](#hooks)
5. [Serveurs MCP](#serveurs-mcp)
6. [Sandbox](#sandbox)
7. [Plugins](#plugins)
8. [Configuration des modeles](#configuration-des-modeles)
9. [Affichage & UX](#affichage--ux)
10. [AWS & identifiants cloud](#aws--identifiants-cloud)
11. [Variables d'environnement](#variables-denvironnement-via-env)
12. [Commandes utiles](#commandes-utiles)

---

## Hierarchie des settings

Les settings s'appliquent par ordre de priorite (du plus eleve au plus bas) :

| Priorite | Emplacement | Portee | Partage ? | Objectif |
|----------|-------------|--------|-----------|----------|
| 1 | Managed settings | Organisation | Oui (deploye par l'IT) | Politiques de securite non modifiables |
| 2 | Arguments de ligne de commande | Session | N/A | Remplacements temporaires pour une session |
| 3 | `.claude/settings.local.json` | Projet | Non (git-ignored) | Settings personnels specifiques au projet |
| 4 | `.claude/settings.json` | Projet | Oui (commite) | Settings partages en equipe |
| 5 | `~/.claude/settings.json` | Utilisateur | N/A | Valeurs par defaut personnelles globales |

Les **managed settings** sont imposes par l'organisation et ne peuvent etre remplaces par aucun autre niveau, y compris les arguments de ligne de commande. Methodes de deploiement :
- Settings **server-managed** (deploiement a distance)
- **Profils MDM** — plist macOS a `com.anthropic.claudecode`
- **Politiques de registre** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode` (admin) et `HKCU\SOFTWARE\Policies\ClaudeCode` (niveau utilisateur, priorite la plus basse)
- **Fichier** — `managed-settings.json` et `managed-mcp.json` (macOS : `/Library/Application Support/ClaudeCode/`, Linux/WSL : `/etc/claude-code/`, Windows : `C:\Program Files\ClaudeCode\`)
- **Repertoire drop-in** — `managed-settings.d/` a cote de `managed-settings.json` pour des fragments de politique independants (v2.1.83). Suivant la convention systemd, `managed-settings.json` est fusionne en premier comme base, puis tous les fichiers `*.json` du repertoire drop-in sont tries par ordre alphabetique et fusionnes par-dessus. Les fichiers suivants remplacent les precedents pour les valeurs scalaires ; les tableaux sont concatenes et dedupliques ; les objets sont fusionnes en profondeur. Les fichiers caches commencant par `.` sont ignores. Utilisez des prefixes numeriques pour controler l'ordre de fusion (ex. `10-telemetry.json`, `20-security.json`)

Au sein du niveau managed, la priorite est : server-managed > politiques MDM/OS > basees sur fichier (`managed-settings.d/*.json` + `managed-settings.json`) > registre HKCU (Windows uniquement). Une seule source managed est utilisee ; les sources ne fusionnent pas entre niveaux. Au sein du niveau base sur fichier, les fichiers drop-in et le fichier de base sont fusionnes ensemble.

> **Note :** Depuis la v2.1.75, le chemin Windows obsolete `C:\ProgramData\ClaudeCode\managed-settings.json` a ete supprime. Utilisez `C:\Program Files\ClaudeCode\managed-settings.json` a la place.

**Important** :
- Les regles `deny` ont la priorite de securite la plus elevee et ne peuvent etre remplacees par des regles allow/ask de priorite inferieure.
- Les managed settings peuvent verrouiller ou remplacer le comportement local meme si les fichiers locaux specfient des valeurs differentes.
- Les settings de type tableau (ex. `permissions.allow`) sont **concatenes et dedupliques** entre les portees — les entrees de tous les niveaux sont combinees, pas remplacees.

---

## Configuration de base

### Settings generaux

| Cle | Type | Defaut | Description |
|-----|------|--------|-------------|
| `$schema` | string | - | URL du JSON Schema pour la validation et l'auto-completion dans l'IDE (ex. `"https://json.schemastore.org/claude-code-settings.json"`) |
| `model` | string | `"default"` | Remplacer le modele par defaut. Accepte des alias (`sonnet`, `opus`, `haiku`) ou des IDs de modele complets |
| `agent` | string | - | Definir l'agent par defaut pour la conversation principale. La valeur est le nom de l'agent depuis `.claude/agents/`. Aussi disponible via le flag CLI `--agent` |
| `language` | string | `"english"` | Langue de reponse preferee de Claude. Definit aussi la langue de dictee vocale |
| `cleanupPeriodDays` | number | `30` | Les sessions inactives au-dela de cette periode sont supprimees au demarrage (minimum 1). Controle aussi l'age limite pour la suppression automatique des worktrees subagent orphelins. La valeur `0` est rejetee avec une erreur de validation. Pour desactiver l'ecriture des transcripts en mode non interactif (`-p`), utilisez `--no-session-persistence` ou l'option SDK `persistSession: false` |
| `autoUpdatesChannel` | string | `"latest"` | Canal de mise a jour : `"stable"` ou `"latest"` |
| `alwaysThinkingEnabled` | boolean | `false` | Activer la reflexion etendue par defaut pour toutes les sessions |
| `skipWebFetchPreflight` | boolean | `false` | Ignorer la verification de la blocklist WebFetch avant de recuperer les URLs *(dans le schema JSON, pas sur la page officielle des settings)* |
| `availableModels` | array | - | Restreindre les modeles que les utilisateurs peuvent selectionner via `/model`, `--model`, l'outil Config ou `ANTHROPIC_MODEL`. N'affecte pas l'option Default. Exemple : `["sonnet", "haiku"]` |
| `fastModePerSessionOptIn` | boolean | `false` | Obliger les utilisateurs a activer le fast mode a chaque session |
| `defaultShell` | string | `"bash"` | Shell par defaut pour les commandes `!` de la zone de saisie. Accepte `"bash"` (defaut) ou `"powershell"`. Definir `"powershell"` redirige les commandes interactives `!` via PowerShell sous Windows. Necessite `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` (v2.1.84) |
| `includeGitInstructions` | boolean | `true` | Inclure les instructions integrees de workflow commit et PR ainsi que le snapshot git status dans le system prompt de Claude. La variable d'environnement `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` a priorite sur ce setting |
| `voiceEnabled` | boolean | - | Activer la dictee vocale push-to-talk. Ecrit automatiquement lorsque vous executez `/voice`. Necessite un compte Claude.ai |
| `showClearContextOnPlanAccept` | boolean | `false` | Afficher l'option "clear context" sur l'ecran d'acceptation du plan. Definir a `true` pour restaurer l'option (masquee par defaut depuis la v2.1.81) |
| `disableDeepLinkRegistration` | string | - | Definir a `"disable"` pour empecher Claude Code d'enregistrer le gestionnaire de protocole `claude-cli://` aupres du systeme d'exploitation au demarrage. Les deep links permettent aux outils externes d'ouvrir une session Claude Code avec un prompt pre-rempli via `claude-cli://open?q=...`. Le parametre `q` supporte les prompts multi-lignes avec des retours a la ligne encodes en URL (`%0A`). Utile dans les environnements ou l'enregistrement du gestionnaire de protocole est restreint ou gere separement |
| `showThinkingSummaries` | boolean | `false` | Afficher les resumes de reflexion etendue dans les sessions interactives. Quand non defini ou `false` (defaut en mode interactif), les blocs de reflexion sont masques par l'API et affiches comme un stub repliable. Le masquage ne change que ce que vous voyez, pas ce que le modele genere — pour reduire les depenses de reflexion, baissez le budget ou desactivez la reflexion. Le mode non interactif (`-p`) et les appelants SDK recoivent toujours les resumes quel que soit ce setting |
| `disableSkillShellExecution` | boolean | `false` | Desactiver l'execution shell inline pour les blocs `` !`...` `` dans les skills et commandes personnalisees. Les commandes sont remplacees par `[shell command execution disabled by policy]`. Les skills bundled et managed ne sont pas affectes (v2.1.91) |
| `forceRemoteSettingsRefresh` | boolean | `false` | **(Managed uniquement)** Bloquer le demarrage de la CLI jusqu'a ce que les managed settings distants soient fraichement recuperes. Si la recuperation echoue, la CLI se ferme (fail-closed). A utiliser dans les environnements d'entreprise ou l'application des politiques doit etre a jour avant le debut de toute session (v2.1.92) |
| `feedbackSurveyRate` | number | - | Probabilite (0-1) que l'enquete de qualite de session apparaisse lorsque eligible. Les administrateurs d'entreprise peuvent controler la frequence d'affichage. Exemple : `0.05` = 5% des sessions eligibles |

**Exemple :**
```json
{
  "model": "opus",
  "agent": "code-reviewer",
  "language": "japanese",
  "cleanupPeriodDays": 60,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true
}
```

### Repertoires de plans et memoire

Stocker les fichiers de plans et de memoire automatique dans des emplacements personnalises.

| Cle | Type | Defaut | Description |
|-----|------|--------|-------------|
| `plansDirectory` | string | `~/.claude/plans` | Repertoire ou les sorties `/plan` sont stockees |
| `autoMemoryDirectory` | string | - | Repertoire personnalise pour le stockage de la memoire automatique. Accepte les chemins avec expansion `~/`. Non accepte dans les settings de projet (`.claude/settings.json`) pour empecher la redirection des ecritures memoire vers des emplacements sensibles ; accepte depuis les settings de politique, locaux et utilisateur |

**Exemple :**
```json
{
  "plansDirectory": "./my-plans"
}
```

**Cas d'utilisation :** Utile pour organiser les artefacts de planification separement des fichiers internes de Claude, ou pour conserver les plans dans un emplacement partage en equipe.

### Settings Worktree

Configurer comment `--worktree` cree et gere les worktrees git. Utile pour reduire l'utilisation disque et le temps de demarrage dans les grands monorepos.

| Cle | Type | Defaut | Description |
|-----|------|--------|-------------|
| `worktree.symlinkDirectories` | array | `[]` | Repertoires a lier symboliquement depuis le depot principal vers chaque worktree pour eviter de dupliquer de grands repertoires sur le disque |
| `worktree.sparsePaths` | array | `[]` | Repertoires a extraire dans chaque worktree via git sparse-checkout (mode cone). Seuls les chemins listes sont ecrits sur le disque |

**Exemple :**
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

### Settings d'attribution

Personnaliser les messages d'attribution pour les commits git et les pull requests.

| Cle | Type | Defaut | Description |
|-----|------|--------|-------------|
| `attribution.commit` | string | Co-authored-by | Attribution de commit git (supporte les trailers) |
| `attribution.pr` | string | Message genere | Attribution de description de pull request |
| `includeCoAuthoredBy` | boolean | `true` | **OBSOLETE** - Utilisez `attribution` a la place |

**Exemple :**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**Note :** Definir a une chaine vide (`""`) pour masquer completement l'attribution.

### Helpers d'authentification

Scripts pour la generation dynamique de tokens d'authentification.

| Cle | Type | Description |
|-----|------|-------------|
| `apiKeyHelper` | string | Chemin du script shell qui produit un token d'authentification (envoye comme en-tete `X-Api-Key`) |
| `forceLoginMethod` | string | Restreindre la connexion aux comptes `"claudeai"` ou `"console"` |
| `forceLoginOrgUUID` | string \| array | Exiger que la connexion appartienne a une organisation specifique. Accepte une seule chaine UUID (qui pre-selectionne aussi cette organisation lors de la connexion) ou un tableau d'UUIDs ou toute organisation listee est acceptee sans pre-selection. Lorsque defini dans les managed settings, la connexion echoue si le compte authentifie n'appartient pas a une organisation listee ; un tableau vide echoue de maniere fermee et bloque la connexion avec un message d'erreur de configuration |

**Exemple :**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

### Annonces d'entreprise

Afficher des annonces personnalisees aux utilisateurs au demarrage (en rotation aleatoire).

| Cle | Type | Description |
|-----|------|-------------|
| `companyAnnouncements` | array | Tableau de chaines affichees au demarrage |

**Exemple :**
```json
{
  "companyAnnouncements": [
    "Welcome to Acme Corp!",
    "Remember to run tests before committing!",
    "Check the wiki for coding standards"
  ]
}
```

---

## Permissions

Controler les outils et operations que Claude peut effectuer.

### Structure des permissions

```json
{
  "permissions": {
    "allow": [],
    "ask": [],
    "deny": [],
    "additionalDirectories": [],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable"
  }
}
```

### Cles de permissions

| Cle | Type | Description |
|-----|------|-------------|
| `permissions.allow` | array | Regles autorisant l'utilisation d'outils sans demander |
| `permissions.ask` | array | Regles necessitant la confirmation de l'utilisateur |
| `permissions.deny` | array | Regles bloquant l'utilisation d'outils (priorite la plus elevee) |
| `permissions.additionalDirectories` | array | Repertoires supplementaires auxquels Claude peut acceder |
| `permissions.defaultMode` | string | Mode de permission par defaut. Dans les environnements distants, seuls `acceptEdits` et `plan` sont respectes (v2.1.70+) |
| `permissions.disableBypassPermissionsMode` | string | Empecher l'activation du mode bypass |
| `permissions.skipDangerousModePermissionPrompt` | boolean | Ignorer l'invite de confirmation affichee avant d'entrer en mode bypass permissions via `--dangerously-skip-permissions` ou `defaultMode: "bypassPermissions"`. Ignore lorsque defini dans les settings de projet (`.claude/settings.json`) pour empecher les depots non fiables de contourner automatiquement l'invite |
| `allowManagedPermissionRulesOnly` | boolean | **(Managed uniquement)** Seules les regles de permission managed s'appliquent ; les regles `allow`, `ask`, `deny` utilisateur/projet sont ignorees |
| `allow_remote_sessions` | boolean | **(Managed uniquement)** Autoriser les utilisateurs a demarrer des sessions Remote Control et web. Par defaut `true`. Definir a `false` pour empecher l'acces aux sessions distantes *(pas dans la documentation officielle — la page officielle des permissions indique "Access to Remote Control and web sessions is not controlled by a managed settings key." Sur les plans Team et Enterprise, les admins activent/desactivent via les [parametres admin Claude Code](https://claude.ai/admin-settings/claude-code))* |
| `autoMode` | object | Personnaliser ce que le classificateur [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode) bloque et autorise. Contient `environment` (descriptions d'infrastructure de confiance), `allow` (exceptions aux regles de blocage) et `soft_deny` (regles de blocage) — tous des tableaux de chaines en prose. **Non lu depuis les settings de projet partages** (`.claude/settings.json`) pour empecher l'injection par le depot. Disponible dans les settings utilisateur, locaux et managed. Definir `allow` ou `soft_deny` **remplace** toute la liste par defaut pour cette section. Executez `claude auto-mode defaults` pour voir les regles integrees avant de personnaliser |
| `disableAutoMode` | string | Definir a `"disable"` pour empecher l'activation du [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode). Supprime `auto` du cycle `Shift+Tab` et rejette `--permission-mode auto` au demarrage. Peut etre defini a n'importe quel niveau de settings ; plus utile dans les managed settings ou les utilisateurs ne peuvent pas le remplacer |
| `useAutoModeDuringPlan` | boolean | Si le mode plan utilise la semantique auto mode lorsque le auto mode est disponible. Par defaut : `true`. Non lu depuis les settings de projet partages (`.claude/settings.json`). Apparait dans `/config` comme "Use auto mode during plan" |

### Modes de permissions

| Mode | Comportement |
|------|-------------|
| `"default"` | Verification standard des permissions avec invites |
| `"acceptEdits"` | Acceptation automatique des modifications de fichiers sans demander |
| `"askEdits"` | Demander avant chaque operation *(pas dans la documentation officielle — non verifie)* |
| `"dontAsk"` | Refuse automatiquement les outils sauf ceux pre-approuves via `/permissions` ou les regles `permissions.allow` |
| `"viewOnly"` | Mode lecture seule, aucune modification *(pas dans la documentation officielle — non verifie)* |
| `"bypassPermissions"` | Ignorer toutes les verifications de permissions (dangereux) |
| `"auto"` | Le classificateur en arriere-plan remplace les invites manuelles (`--enable-auto-mode`). Apercu recherche — necessite un plan Team + Sonnet/Opus 4.6. Le classificateur approuve automatiquement la lecture seule et les modifications de fichiers ; envoie tout le reste via une verification de securite. Revient aux invites apres 3 blocages consecutifs ou 20 au total. Configurer avec le setting `autoMode` |
| `"plan"` | Mode exploration en lecture seule |

### Syntaxe des permissions d'outils

| Outil | Syntaxe | Exemples |
|-------|---------|----------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`, `Bash(* install)`, `Bash(git * main)` |
| `Read` | `Read(path pattern)` | `Read(.env)`, `Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`, `Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`, `Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | Recherche web globale |
| `Task` | `Task(agent-name)` | `Task(Explore)`, `Task(my-agent)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`, `Agent(*)` — permission limitee au lancement de subagents |
| `Skill` | `Skill(skill-name)` | `Skill(weather-fetcher)` |
| `MCP` | `mcp__server__tool` ou `MCP(server:tool)` | `mcp__memory__*`, `MCP(github:*)` |

**Ordre d'evaluation :** Les regles sont evaluees dans l'ordre : regles deny en premier, puis ask, puis allow. La premiere regle correspondante l'emporte.

**Patterns de chemin Read/Edit :** Les regles de permissions pour `Read`, `Edit` et `Write` supportent les patterns de style gitignore avec quatre types de prefixes :

| Prefixe | Signification | Exemple |
|---------|---------------|---------|
| `//` | Chemin absolu depuis la racine du systeme de fichiers | `Read(//Users/alice/file)` |
| `~/` | Relatif au repertoire home | `Read(~/.zshrc)` |
| `/` | Relatif a la racine du projet | `Edit(/src/**)` |
| `./` ou aucun | Chemin relatif (repertoire courant) | `Read(.env)`, `Read(*.ts)` |

**Notes sur les wildcards Bash :**
- `*` peut apparaitre a **n'importe quelle position** : prefixe (`Bash(* install)`), suffixe (`Bash(npm *)`), ou milieu (`Bash(git * main)`)
- **Limite de mot :** `Bash(ls *)` (espace avant `*`) correspond a `ls -la` mais PAS a `lsof` ; `Bash(ls*)` (sans espace) correspond aux deux
- `Bash(*)` est traite comme equivalent a `Bash` (correspond a toutes les commandes bash)
- Les regles de permissions supportent les redirections de sortie : `Bash(python:*)` correspond a `python script.py > output.txt`
- L'ancienne syntaxe suffixe `:*` (ex. `Bash(npm:*)`) est equivalente a ` *` mais est obsolete

**Exemple :**
```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*"
    ],
    "ask": [
      "Bash(rm *)",
      "Bash(git push *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)",
      "Bash(curl *)"
    ],
    "additionalDirectories": ["../shared-libs/"]
  }
}
```

---

## Hooks

La configuration des hooks (evenements, proprietes, matchers, codes de sortie, variables d'environnement et hooks HTTP) est maintenue dans un depot dedie :

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — Reference complete des hooks avec systeme de notification sonore, les 25 evenements de hook, hooks HTTP, patterns de matcher, codes de sortie et variables d'environnement.

Les cles de settings liees aux hooks (`hooks`, `disableAllHooks`, `allowManagedHooksOnly`, `allowedHttpHookUrls`, `httpHookAllowedEnvVars`) y sont documentees.

Pour la reference officielle des hooks, consultez la [Documentation des Hooks Claude Code](https://code.claude.com/docs/en/hooks).

---

## Serveurs MCP

Configurer les serveurs Model Context Protocol pour des capacites etendues.

### Settings MCP

| Cle | Type | Portee | Description |
|-----|------|--------|-------------|
| `enableAllProjectMcpServers` | boolean | Tout | Approuver automatiquement tous les serveurs `.mcp.json` |
| `enabledMcpjsonServers` | array | Tout | Liste blanche de noms de serveurs specifiques |
| `disabledMcpjsonServers` | array | Tout | Liste noire de noms de serveurs specifiques |
| `allowedMcpServers` | array | Managed uniquement | Liste blanche avec correspondance nom/commande/URL |
| `deniedMcpServers` | array | Managed uniquement | Liste noire avec correspondance |
| `allowManagedMcpServersOnly` | boolean | Managed uniquement | N'autoriser que les serveurs MCP explicitement listes dans la liste blanche managed |
| `channelsEnabled` | boolean | Managed uniquement | Autoriser les [channels](https://code.claude.com/docs/en/channels) pour les utilisateurs Team et Enterprise. Quand non defini ou `false`, la livraison des messages de channel est bloquee quel que soit le flag `--channels` |
| `allowedChannelPlugins` | array | Managed uniquement | Liste blanche des plugins de channel pouvant envoyer des messages. Remplace la liste blanche par defaut d'Anthropic lorsque definie. Non defini = revenir a la liste par defaut, tableau vide = bloquer tous les plugins de channel. Necessite `channelsEnabled: true`. Chaque entree est un objet avec les champs `marketplace` et `plugin` (v2.1.84) |

### Correspondance de serveur MCP (Managed Settings)

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": "npx @modelcontextprotocol/*" },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" }
  ]
}
```

**Exemple :**
```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["memory", "github", "filesystem"],
  "disabledMcpjsonServers": ["experimental-server"]
}
```

---

## Sandbox

Configurer le sandboxing des commandes bash pour la securite.

### Settings Sandbox

| Cle | Type | Defaut | Description |
|-----|------|--------|-------------|
| `sandbox.enabled` | boolean | `false` | Activer le sandboxing bash |
| `sandbox.failIfUnavailable` | boolean | `false` | Quitter avec erreur lorsque le sandbox est active mais ne peut pas demarrer, au lieu d'executer sans sandbox. Utile pour les politiques d'entreprise exigeant un sandboxing strict (v2.1.83) |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | Approuver automatiquement bash lorsque sandboxe |
| `sandbox.excludedCommands` | array | `[]` | Commandes a executer en dehors du sandbox |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | Autoriser `dangerouslyDisableSandbox`. Lorsque defini a `false`, la sortie de secours est completement desactivee et toutes les commandes doivent etre sandboxees (ou etre dans `excludedCommands`). Utile pour les politiques d'entreprise exigeant un sandboxing strict |
| `sandbox.ignoreViolations` | object | `{}` | Correspondance de patterns de commandes vers des tableaux de chemins — supprime les avertissements de violation *(dans le schema JSON, pas sur la page officielle des settings)* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | Sandbox plus faible pour Docker (reduit la securite) |
| `sandbox.network.allowUnixSockets` | array | `[]` | Chemins specifiques de sockets Unix accessibles dans le sandbox |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | Autoriser tous les sockets Unix (remplace allowUnixSockets) |
| `sandbox.network.allowLocalBinding` | boolean | `false` | Autoriser la liaison aux ports localhost (macOS) |
| `sandbox.network.allowedDomains` | array | `[]` | Liste blanche de domaines reseau pour le sandbox |
| `sandbox.network.deniedDomains` | array | `[]` | Liste noire de domaines reseau pour le sandbox *(pas dans la documentation officielle — non verifie)* |
| `sandbox.network.httpProxyPort` | number | - | Port proxy HTTP 1-65535 (proxy personnalise) |
| `sandbox.network.socksProxyPort` | number | - | Port proxy SOCKS5 1-65535 (proxy personnalise) |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | N'autoriser que les domaines de la liste blanche managed (managed settings) |
| `sandbox.filesystem.allowWrite` | array | `[]` | Chemins supplementaires ou les commandes sandboxees peuvent ecrire. Les tableaux sont fusionnes entre toutes les portees de settings. Aussi fusionnes avec les chemins des regles de permission `Edit(...)` allow. Prefixe : `/` (absolu), `~/` (home), `./` ou aucun (relatif au projet dans les settings de projet, relatif a `~/.claude` dans les settings utilisateur). L'ancien prefixe `//` pour les chemins absolus fonctionne encore. **Note :** Cela differe des [regles de permission Read/Edit](#syntaxe-des-permissions-doutils), qui utilisent `//` pour absolu et `/` pour relatif au projet |
| `sandbox.filesystem.denyWrite` | array | `[]` | Chemins ou les commandes sandboxees ne peuvent pas ecrire. Les tableaux sont fusionnes entre toutes les portees de settings. Aussi fusionnes avec les chemins des regles de permission `Edit(...)` deny. Memes conventions de prefixe de chemin que `allowWrite` |
| `sandbox.filesystem.denyRead` | array | `[]` | Chemins ou les commandes sandboxees ne peuvent pas lire. Les tableaux sont fusionnes entre toutes les portees de settings. Aussi fusionnes avec les chemins des regles de permission `Read(...)` deny. Memes conventions de prefixe de chemin que `allowWrite` |
| `sandbox.filesystem.allowRead` | array | `[]` | Chemins pour re-autoriser l'acces en lecture dans les regions `denyRead`. A priorite sur `denyRead`. Les tableaux sont fusionnes entre toutes les portees de settings. Memes conventions de prefixe de chemin que `allowWrite` |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **(Managed uniquement)** Seuls les chemins `allowRead` des managed settings sont respectes. Les entrees `allowRead` des settings utilisateur, projet et locaux sont ignorees |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | (macOS uniquement) Autoriser l'acces au trust TLS systeme (`com.apple.trustd.agent`) ; reduit la securite |

**Exemple :**
```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["git", "docker", "gh"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true
    }
  }
}
```

---

## Plugins

Configurer les plugins et marketplaces Claude Code.

### Settings des plugins

| Cle | Type | Portee | Description |
|-----|------|--------|-------------|
| `enabledPlugins` | object | Tout | Activer/desactiver des plugins specifiques |
| `extraKnownMarketplaces` | object | Projet | Ajouter des marketplaces de plugins personnalises (partage en equipe via `.claude/settings.json`) |
| `strictKnownMarketplaces` | array | Managed uniquement | Liste blanche des marketplaces autorisees |
| `skippedMarketplaces` | array | Tout | Marketplaces que l'utilisateur a refuse d'installer *(dans le schema JSON, pas sur la page officielle des settings)* |
| `skippedPlugins` | array | Tout | Plugins que l'utilisateur a refuse d'installer *(dans le schema JSON, pas sur la page officielle des settings)* |
| `pluginConfigs` | object | Tout | Configurations de serveur MCP par plugin (cle par `plugin@marketplace`) *(dans le schema JSON, pas sur la page officielle des settings)* |
| `blockedMarketplaces` | array | Managed uniquement | Bloquer des marketplaces de plugins specifiques |
| `pluginTrustMessage` | string | Managed uniquement | Message personnalise affiche lors de l'invite de confiance des plugins |

**Types de source marketplace :** `github`, `git`, `directory`, `hostPattern`, `settings`, `url`, `npm`, `file`. Utilisez `source: 'settings'` pour declarer un petit ensemble de plugins en ligne sans configurer un depot marketplace heberge.

**Exemple :**
```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "experimental@acme-tools": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "inline-tools": {
      "source": {
        "source": "settings",
        "name": "inline-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": { "source": "github", "repo": "acme-corp/code-formatter" }
          }
        ]
      }
    }
  }
}
```

---

## Configuration des modeles

### Alias de modeles

| Alias | Description |
|-------|-------------|
| `"default"` | Recommande pour votre type de compte |
| `"sonnet"` | Dernier modele Sonnet (Claude Sonnet 4.6) |
| `"opus"` | Dernier modele Opus (Claude Opus 4.6) |
| `"haiku"` | Modele Haiku rapide |
| `"sonnet[1m]"` | Sonnet avec contexte de 1M tokens |
| `"opus[1m]"` | Opus avec contexte de 1M tokens (par defaut sur Max, Team et Enterprise depuis la v2.1.75) |
| `"opusplan"` | Opus pour la planification, Sonnet pour l'execution |

**Exemple :**
```json
{
  "model": "opus"
}
```

### Surcharges de modeles

Mapper les IDs de modeles Anthropic vers des IDs de modeles specifiques aux fournisseurs pour les deploiements Bedrock, Vertex ou Foundry.

| Cle | Type | Defaut | Description |
|-----|------|--------|-------------|
| `effortLevel` | string | - | Persister le niveau d'effort entre les sessions. Accepte `"low"`, `"medium"` ou `"high"`. Ecrit automatiquement lorsque vous executez `/effort low`, `/effort medium` ou `/effort high`. Supporte sur Opus 4.6 et Sonnet 4.6 |
| `modelOverrides` | object | - | Mapper les entrees du selecteur de modeles vers des IDs de fournisseurs specifiques (ex. ARNs de profil d'inference Bedrock). Chaque cle est un nom d'entree du selecteur, chaque valeur est l'ID du modele fournisseur |

**Exemple :**
```json
{
  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-sonnet-4-6-v1:0"
  }
}
```

### Niveau d'effort

La commande `/model` expose un controle de **niveau d'effort** qui ajuste la profondeur de raisonnement du modele par reponse. Utilisez les touches flechees ← → dans l'UI `/model` pour parcourir les niveaux d'effort.

| Niveau d'effort | Description |
|-----------------|-------------|
| High (defaut) | Profondeur de raisonnement complete, ideal pour les taches complexes |
| Medium | Raisonnement equilibre, bon pour les taches quotidiennes |
| Low | Raisonnement minimal, reponses les plus rapides |

**Comment utiliser :**
1. Executez `/effort low`, `/effort medium` ou `/effort high` pour definir directement (v2.1.76+)
2. Ou executez `/model` → selectionnez un modele → utilisez les touches **← →** pour ajuster
3. Le setting persiste via la cle `effortLevel` dans `settings.json`

**Note :** Le niveau d'effort est disponible pour Opus 4.6 et Sonnet 4.6 sur les plans Max et Team. La valeur par defaut a ete changee de High a Medium dans la v2.1.68, puis remise a **High** pour les utilisateurs API-key, Bedrock/Vertex/Foundry, Team et Enterprise dans la v2.1.94. Depuis la v2.1.75, la fenetre de contexte 1M pour Opus 4.6 est disponible par defaut sur les plans Max, Team et Enterprise.

### Variables d'environnement de modeles

Configurer via la cle `env` :

```json
{
  "env": {
    "ANTHROPIC_MODEL": "sonnet",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "custom-haiku-model",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "custom-sonnet-model",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "custom-opus-model",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku",
    "MAX_THINKING_TOKENS": "10000"
  }
}
```

---

## Affichage & UX

### Settings d'affichage

| Cle | Type | Defaut | Description |
|-----|------|--------|-------------|
| `statusLine` | object | - | Configuration personnalisee de la ligne de statut |
| `outputStyle` | string | `"default"` | Style de sortie (ex. `"Explanatory"`) |
| `spinnerTipsEnabled` | boolean | `true` | Afficher des astuces pendant l'attente |
| `spinnerVerbs` | object | - | Verbes de spinner personnalises avec `mode` ("append" ou "replace") et tableau `verbs` |
| `spinnerTipsOverride` | object | - | Astuces de spinner personnalisees avec `tips` (tableau de chaines) et `excludeDefault` optionnel (booleen) |
| `respectGitignore` | boolean | `true` | Respecter .gitignore dans le selecteur de fichiers |
| `prefersReducedMotion` | boolean | `false` | Reduire les animations et effets de mouvement dans l'UI |
| `fileSuggestion` | object | - | Commande personnalisee de suggestion de fichiers (voir Configuration des suggestions de fichiers ci-dessous) |

### Settings de configuration globale (`~/.claude.json`)

Ces preferences d'affichage sont stockees dans `~/.claude.json`, **pas** dans `settings.json`. Les ajouter a `settings.json` declenchera une erreur de validation de schema.

| Cle | Type | Defaut | Description |
|-----|------|--------|-------------|
| `autoConnectIde` | boolean | `false` | Se connecter automatiquement a un IDE en cours d'execution lorsque Claude Code demarre depuis un terminal externe. Apparait dans `/config` comme **Auto-connect to IDE (external terminal)** lors de l'execution en dehors d'un terminal VS Code ou JetBrains |
| `autoInstallIdeExtension` | boolean | `true` | Installer automatiquement l'extension IDE Claude Code lors de l'execution depuis un terminal VS Code. Apparait dans `/config` comme **Auto-install IDE extension**. Peut aussi etre desactive via la variable d'env `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` |
| `editorMode` | string | `"normal"` | Mode de raccourcis clavier pour l'invite de saisie : `"normal"` ou `"vim"`. Apparait dans `/config` comme **Editor mode** |
| `showTurnDuration` | boolean | `true` | Afficher les messages de duree de tour apres les reponses (ex. "Cooked for 1m 6s"). Modifier directement `~/.claude.json` pour changer |
| `terminalProgressBarEnabled` | boolean | `true` | Afficher la barre de progression du terminal dans les terminaux supportes (ConEmu, Ghostty 1.2.0+ et iTerm2 3.6.6+). Apparait dans `/config` comme **Terminal progress bar** |
| `teammateMode` | string | `"in-process"` | Comment les coequipiers [agent team](https://code.claude.com/docs/en/agent-teams) s'affichent : `"auto"` (choisit les panneaux divises dans tmux ou iTerm2, in-process sinon), `"in-process"`, ou `"tmux"`. Voir [choisir un mode d'affichage](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) |

### Configuration de la ligne de statut

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

**Champs d'entree de la ligne de statut :**

La commande de la ligne de statut recoit un objet JSON sur stdin avec ces champs notables :

| Champ | Description |
|-------|-------------|
| `workspace.added_dirs` | Repertoires ajoutes via `/add-dir` |
| `context_window.used_percentage` | Pourcentage d'utilisation de la fenetre de contexte |
| `context_window.remaining_percentage` | Pourcentage restant de la fenetre de contexte |
| `current_usage` | Nombre actuel de tokens de la fenetre de contexte |
| `exceeds_200k_tokens` | Si le contexte depasse 200k tokens |
| `rate_limits.five_hour.used_percentage` | Pourcentage d'utilisation de la limite de taux sur 5 heures (v2.1.80+) |
| `rate_limits.five_hour.resets_at` | Horodatage de reinitialisation de la limite de taux sur 5 heures |
| `rate_limits.seven_day.used_percentage` | Pourcentage d'utilisation de la limite de taux sur 7 jours |
| `rate_limits.seven_day.resets_at` | Horodatage de reinitialisation de la limite de taux sur 7 jours |

### Configuration des suggestions de fichiers

Le script de suggestion de fichiers recoit un objet JSON sur stdin (ex. `{"query": "src/comp"}`) et doit produire jusqu'a 15 chemins de fichiers (un par ligne).

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  },
  "respectGitignore": true
}
```

**Exemple :**
```json
{
  "statusLine": {
    "type": "command",
    "command": "git branch --show-current 2>/dev/null || echo 'no-branch'"
  },
  "spinnerTipsEnabled": true,
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": ["Cooking", "Brewing", "Crafting", "Conjuring"]
  },
  "spinnerTipsOverride": {
    "tips": ["Use /compact at ~50% context", "Start with plan mode for complex tasks"],
    "excludeDefault": true
  }
}
```

---

## AWS & identifiants cloud

### Settings AWS

| Cle | Type | Description |
|-----|------|-------------|
| `awsAuthRefresh` | string | Script pour rafraichir l'authentification AWS (modifie le repertoire `.aws`) |
| `awsCredentialExport` | string | Script produisant un JSON avec les identifiants AWS |

**Exemple :**
```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

### OpenTelemetry

| Cle | Type | Description |
|-----|------|-------------|
| `otelHeadersHelper` | string | Script pour generer des en-tetes OpenTelemetry dynamiques |

**Exemple :**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## Variables d'environnement (via `env`)

Definir des variables d'environnement pour toutes les sessions Claude Code.

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "...",
    "NODE_ENV": "development",
    "DEBUG": "true"
  }
}
```

### Variables d'environnement courantes

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | Cle API pour l'authentification |
| `ANTHROPIC_AUTH_TOKEN` | Token OAuth |
| `CLAUDE_CODE_OAUTH_TOKEN` | Token d'acces OAuth pour l'authentification Claude.ai. Alternative a `/login` pour le SDK et les environnements automatises. A priorite sur les identifiants stockes dans le keychain |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | Token de rafraichissement OAuth pour l'authentification Claude.ai. Lorsque defini, `claude auth login` echange ce token directement au lieu d'ouvrir un navigateur. Necessite `CLAUDE_CODE_OAUTH_SCOPES` |
| `CLAUDE_CODE_OAUTH_SCOPES` | Portees OAuth separees par des espaces avec lesquelles le token de rafraichissement a ete emis (ex. `"user:profile user:inference user:sessions:claude_code"`). Requis lorsque `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` est defini |
| `ANTHROPIC_BASE_URL` | Point de terminaison API personnalise |
| `ANTHROPIC_BEDROCK_BASE_URL` | Remplacer l'URL du point de terminaison Bedrock |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | Remplacer l'URL du point de terminaison Bedrock Mantle. Voir [point de terminaison Mantle](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) |
| `ANTHROPIC_VERTEX_BASE_URL` | Remplacer l'URL du point de terminaison Vertex AI |
| `ANTHROPIC_BETAS` | Valeurs d'en-tete beta Anthropic separees par des virgules |
| `ANTHROPIC_VERTEX_PROJECT_ID` | ID de projet GCP pour Vertex AI |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | ID de modele a ajouter comme entree personnalisee dans le selecteur `/model`. A utiliser pour rendre un modele non standard ou specifique a un gateway selectionnable sans remplacer les alias integres |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | Nom d'affichage pour l'entree de modele personnalise dans le selecteur `/model`. Par defaut l'ID du modele |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | Description d'affichage pour l'entree de modele personnalise dans le selecteur `/model`. Par defaut `Custom model (<model-id>)` |
| `ANTHROPIC_MODEL` | Nom du modele a utiliser. Accepte des alias (`sonnet`, `opus`, `haiku`) ou des IDs de modele complets. Remplace le setting `model` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Remplacer l'alias du modele Haiku avec un ID de modele personnalise (ex. pour les deploiements tiers) |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | Personnaliser le libelle de l'entree Haiku dans le selecteur `/model` lors de l'utilisation d'un modele fixe sur Bedrock/Vertex/Foundry. Par defaut l'ID du modele |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | Personnaliser la description de l'entree Haiku dans le selecteur `/model`. Par defaut `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | Remplacer la detection des capacites pour un modele Haiku fixe. Valeurs separees par des virgules (ex. `effort,thinking`). Requis lorsque le modele fixe supporte des fonctionnalites que la detection automatique ne peut pas confirmer |
| `CLAUDECODE` | Defini a `1` dans les environnements shell que Claude Code cree (outil Bash, sessions tmux). Non defini dans les hooks ou commandes de ligne de statut. A utiliser pour detecter quand un script s'execute dans un shell Claude Code |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | Defini a `1` pour autoriser le fast mode lorsque la verification du statut de l'organisation echoue en raison d'une erreur reseau. Utile quand un proxy d'entreprise bloque le point de terminaison de statut |
| `CLAUDE_CODE_USE_BEDROCK` | Utiliser AWS Bedrock (`1` pour activer) |
| `CLAUDE_CODE_USE_VERTEX` | Utiliser Google Vertex AI (`1` pour activer) |
| `CLAUDE_CODE_USE_FOUNDRY` | Utiliser Microsoft Foundry (`1` pour activer) |
| `CLAUDE_CODE_USE_MANTLE` | Utiliser le [point de terminaison Mantle](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) Bedrock (`1` pour activer) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Defini a `1` pour activer l'outil PowerShell sous Windows (apercu opt-in). Lorsque active, Claude peut executer des commandes PowerShell nativement au lieu de passer par Git Bash. Supporte uniquement sous Windows natif, pas WSL (v2.1.84) |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | Prefixe pour les noms de session Remote Control generes automatiquement. Par defaut le nom d'hote de la machine |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | Activer/desactiver la telemetrie (`0` ou `1`) |
| `DISABLE_ERROR_REPORTING` | Desactiver le rapport d'erreurs (`1` pour desactiver) |
| `DISABLE_TELEMETRY` | Desactiver la telemetrie (`1` pour desactiver) |
| `MCP_TIMEOUT` | Timeout de demarrage MCP en ms |
| `MAX_MCP_OUTPUT_TOKENS` | Tokens de sortie MCP max (defaut : 25000). Avertissement affiche lorsque la sortie depasse 10 000 tokens |
| `API_TIMEOUT_MS` | Timeout en ms pour les requetes API (defaut : 600000) |
| `BASH_MAX_TIMEOUT_MS` | Timeout des commandes bash |
| `BASH_MAX_OUTPUT_LENGTH` | Longueur max de sortie bash |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Seuil de compactage automatique en pourcentage (1-100). Par defaut ~95%. Definir plus bas (ex. `50`) pour declencher le compactage plus tot. Les valeurs au-dessus de 95% n'ont aucun effet. Utilisez `/context` pour surveiller l'utilisation actuelle. Exemple : `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Conserver le cwd entre les appels bash (`1` pour activer) |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Desactiver les taches en arriere-plan (`1` pour desactiver) |
| `ENABLE_TOOL_SEARCH` | Seuil de recherche d'outils MCP (ex. `auto:5`) |
| `DISABLE_PROMPT_CACHING` | Desactiver tout le cache de prompts (`1` pour desactiver) |
| `DISABLE_PROMPT_CACHING_HAIKU` | Desactiver le cache de prompts Haiku |
| `DISABLE_PROMPT_CACHING_SONNET` | Desactiver le cache de prompts Sonnet |
| `DISABLE_PROMPT_CACHING_OPUS` | Desactiver le cache de prompts Opus |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | Demander un TTL de cache de 1 heure sur Bedrock (`1` pour activer) |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | Desactiver les fonctionnalites beta experimentales (`1` pour desactiver) |
| `CLAUDE_CODE_SHELL` | Remplacer la detection automatique du shell |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | Remplacer la limite de tokens par defaut pour la lecture de fichiers |
| `CLAUDE_CODE_GLOB_HIDDEN` | Definir a `false` pour exclure les dotfiles des resultats lorsque Claude utilise l'outil Glob. Inclus par defaut. N'affecte pas l'auto-completion `@`, `ls`, Grep ou Read |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | Definir a `false` pour que l'outil Glob respecte les patterns `.gitignore`. Par defaut, Glob retourne tous les fichiers correspondants y compris ceux gitignores. N'affecte pas l'auto-completion `@`, qui a son propre setting `respectGitignore` |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Timeout en secondes pour la decouverte de fichiers Glob |
| `CLAUDE_CODE_ENABLE_TASKS` | Definir a `true` pour activer le suivi des taches en mode non interactif (flag `-p`). Les taches sont activees par defaut en mode interactif |
| `CLAUDE_CODE_SIMPLE` | Definir a `1` pour executer avec un system prompt minimal et uniquement les outils Bash, lecture de fichier et edition de fichier |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | Quitter automatiquement le mode SDK apres une duree d'inactivite (ms) |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | Desactiver la reflexion adaptative (`1` pour desactiver) |
| `CLAUDE_CODE_DISABLE_THINKING` | Forcer la desactivation de la reflexion etendue (`1` pour desactiver) |
| `DISABLE_INTERLEAVED_THINKING` | Empecher l'envoi de l'en-tete beta interleaved-thinking (`1` pour desactiver) |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | Desactiver la fenetre de contexte 1M tokens (`1` pour desactiver) |
| `CLAUDE_CODE_ACCOUNT_UUID` | Remplacer l'UUID du compte pour l'authentification |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | Desactiver les instructions git du system prompt |
| `CLAUDE_CODE_NEW_INIT` | Definir a `true` pour que `/init` execute un flux de configuration interactif. Demande quels fichiers generer (CLAUDE.md, skills, hooks) avant d'explorer le codebase. Sans cela, `/init` genere un CLAUDE.md automatiquement |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | Chemin vers un ou plusieurs repertoires de plugins pre-remplis en lecture seule, separes par `:` sur Unix ou `;` sur Windows. Integrer des plugins pre-remplis dans une image de conteneur. Claude Code enregistre les marketplaces depuis ces repertoires au demarrage et utilise les plugins pre-caches sans re-clonage |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | Activer les serveurs MCP Claude.ai |
| `CLAUDE_CODE_EFFORT_LEVEL` | Definir le niveau d'effort : `low`, `medium`, `high`, `max` (Opus 4.6 uniquement), ou `auto` (utiliser la valeur par defaut du modele). A priorite sur `/effort` et le setting `effortLevel` |
| `CLAUDE_CODE_MAX_TURNS` | Nombre maximum de tours agentiques avant arret *(pas dans la documentation officielle — non verifie)* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Equivalent a definir `DISABLE_AUTOUPDATER`, `DISABLE_FEEDBACK_COMMAND`, `DISABLE_ERROR_REPORTING` et `DISABLE_TELEMETRY` |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | Ignorer le flux de configuration du premier lancement *(pas dans la documentation officielle — non verifie)* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | Remplacer le comportement du cache de prompts *(pas dans la documentation officielle — non verifie)* |
| `CLAUDE_CODE_DISABLE_TOOLS` | Liste d'outils a desactiver separee par des virgules *(pas dans la documentation officielle — non verifie)* |
| `CLAUDE_CODE_DISABLE_MCP` | Desactiver tous les serveurs MCP (`1` pour desactiver) *(pas dans la documentation officielle — non verifie)* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Tokens de sortie max par reponse. Defaut : 32 000 (64 000 pour Opus 4.6 depuis la v2.1.77). Limite superieure : 64 000 (128 000 pour Opus 4.6 et Sonnet 4.6 depuis la v2.1.77) |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | Desactiver completement le fast mode (`1` pour desactiver) |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | Definir a `1` pour desactiver le fallback non-streaming lorsqu'une requete streaming echoue en cours de route. Les erreurs de streaming se propagent a la couche de retry a la place. Utile lorsqu'un proxy ou gateway cause une execution d'outil en double avec le fallback (v2.1.83) |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | Abandonner les streams bloques (`1` pour activer) |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | Activer le streaming fin d'outils (`1` pour activer) |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | Desactiver la memoire automatique (`1` pour desactiver) |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | Desactiver le checkpointing de fichiers pour `/rewind` (`1` pour desactiver) |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | Desactiver le traitement des pieces jointes (`1` pour desactiver) |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | Empecher le chargement des fichiers CLAUDE.md (`1` pour desactiver) |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | Reprendre automatiquement si la session precedente s'est terminee en plein tour (`1` pour activer) |
| `CLAUDE_CODE_USER_EMAIL` | Fournir l'email utilisateur de maniere synchrone pour l'authentification |
| `CLAUDE_CODE_ORGANIZATION_UUID` | Fournir l'UUID de l'organisation de maniere synchrone pour l'authentification |
| `CLAUDE_CONFIG_DIR` | Repertoire de configuration personnalise (remplace le defaut `~/.claude`) |
| `CLAUDE_CODE_TMPDIR` | Remplacer le repertoire temporaire utilise pour les fichiers temporaires internes. Claude Code ajoute `/claude/` a ce chemin. Defaut : `/tmp` sur Unix/macOS, `os.tmpdir()` sur Windows |
| `ANTHROPIC_CUSTOM_HEADERS` | En-tetes personnalises pour les requetes API (format `Name: Value`, separes par des retours a la ligne pour plusieurs en-tetes) |
| `ANTHROPIC_FOUNDRY_API_KEY` | Cle API pour l'authentification Microsoft Foundry |
| `ANTHROPIC_FOUNDRY_BASE_URL` | URL de base pour la ressource Foundry |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Nom de la ressource Foundry |
| `AWS_BEARER_TOKEN_BEDROCK` | Cle API Bedrock pour l'authentification |
| `ANTHROPIC_SMALL_FAST_MODEL` | **OBSOLETE** — Utilisez `ANTHROPIC_DEFAULT_HAIKU_MODEL` a la place |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | Region AWS pour le remplacement obsolete du modele de classe Haiku |
| `CLAUDE_CODE_SHELL_PREFIX` | Prefixe de commande ajoute aux commandes bash |
| `BASH_DEFAULT_TIMEOUT_MS` | Timeout par defaut des commandes bash en ms |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | Ignorer l'authentification AWS pour Bedrock (`1` pour ignorer) |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | Ignorer l'authentification Azure pour Foundry (`1` pour ignorer) |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | Ignorer l'authentification AWS pour Bedrock Mantle (ex. lors de l'utilisation d'un gateway LLM) |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | Ignorer l'authentification Google pour Vertex (`1` pour ignorer) |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | Permettre au proxy d'effectuer la resolution DNS |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | Intervalle de rafraichissement des identifiants en ms pour `apiKeyHelper` |
| `CLAUDE_CODE_CLIENT_CERT` | Chemin du certificat client pour mTLS |
| `CLAUDE_CODE_CLIENT_KEY` | Chemin de la cle privee client pour mTLS |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | Phrase de passe pour la cle mTLS chiffree |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | Timeout du clone git du marketplace de plugins en ms (defaut : 120000) |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | Remplacer le repertoire racine des plugins |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | Ignorer l'ajout automatique du marketplace officiel (`1` pour desactiver) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | Attendre la fin de l'installation du plugin avant la premiere requete (`1` pour activer) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | Timeout en ms pour l'installation synchrone des plugins |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | Definir a `1` pour conserver le cache du marketplace existant lorsqu'un `git pull` echoue au lieu de supprimer et re-cloner. Utile dans les environnements hors ligne ou air-gapped ou le re-clonage echouerait de la meme maniere |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | Masquer les informations email/org de l'UI *(pas dans la documentation officielle — non verifie)* |
| `CLAUDE_CODE_DISABLE_CRON` | Desactiver les taches planifiees/cron (`1` pour desactiver) |
| `DISABLE_INSTALLATION_CHECKS` | Desactiver les avertissements d'installation |
| `DISABLE_FEEDBACK_COMMAND` | Desactiver la commande `/feedback`. L'ancien nom `DISABLE_BUG_COMMAND` est aussi accepte |
| `DISABLE_DOCTOR_COMMAND` | Masquer la commande `/doctor` (`1` pour desactiver) |
| `DISABLE_LOGIN_COMMAND` | Masquer la commande `/login` (`1` pour desactiver) |
| `DISABLE_LOGOUT_COMMAND` | Masquer la commande `/logout` (`1` pour desactiver) |
| `DISABLE_UPGRADE_COMMAND` | Masquer la commande `/upgrade` (`1` pour desactiver) |
| `DISABLE_EXTRA_USAGE_COMMAND` | Masquer la commande `/extra-usage` (`1` pour desactiver) |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | Masquer la commande `/install-github-app` (`1` pour desactiver) |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | Desactiver le texte decoratif et les appels non essentiels au modele *(pas dans la documentation officielle — non verifie)* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | Remplacer le chemin du repertoire des logs de debogage |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | Niveau minimum des logs de debogage |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | Forcer la mise en arriere-plan automatique des taches longues (`1` pour activer) |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | Empecher le remapping d'Opus 4.0/4.1 vers les modeles plus recents (`1` pour desactiver) |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | Declencher le modele de fallback pour tous les modeles principaux, pas uniquement le defaut (`1` pour activer) |
| `CLAUDE_CODE_GIT_BASH_PATH` | Chemin de l'executable Git Bash Windows (demarrage uniquement) |
| `DISABLE_COST_WARNINGS` | Desactiver les messages d'avertissement de cout |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Remplacer le modele pour les subagents (ex. `haiku`, `sonnet`) |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | Definir a `1` pour supprimer les identifiants Anthropic et des fournisseurs cloud des environnements de sous-processus (outil Bash, hooks, serveurs MCP stdio). A utiliser pour la defense en profondeur lorsque les sous-processus ne doivent pas heriter des cles API (v2.1.83) |
| `CLAUDE_CODE_MAX_RETRIES` | Remplacer le nombre de tentatives de requete API (defaut : 10) |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | Outils paralleles en lecture seule max (defaut : 10) |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | Desactiver les types de subagents integres en mode SDK (`1` pour desactiver) |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | Ignorer le prefixe `mcp__<server>__` pour les outils MCP en mode SDK (`1` pour activer) |
| `MCP_CONNECTION_NONBLOCKING` | Definir a `true` en mode `-p` pour ignorer completement l'attente de connexion MCP. Limite les connexions des serveurs `--mcp-config` a 5s au lieu de bloquer sur le serveur le plus lent *(dans le changelog v2.1.89, pas encore sur la page officielle des variables d'env)* |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | Timeout des hooks SessionEnd en ms (remplace la limite fixe de 1.5s) |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | Desactiver les invites d'enquete de feedback (`1` pour desactiver) |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | Desactiver les mises a jour du titre du terminal (`1` pour desactiver) |
| `CLAUDE_CODE_NO_FLICKER` | Definir a `1` pour activer le rendu alt-screen sans scintillement. Elimine le scintillement visuel lors des redessinages plein ecran (v2.1.88) |
| `CLAUDE_CODE_SCROLL_SPEED` | Multiplicateur de vitesse de defilement de la molette pour le rendu plein ecran. Augmenter pour un defilement plus rapide, diminuer pour un controle plus fin |
| `CLAUDE_CODE_DISABLE_MOUSE` | Definir a `1` pour desactiver le suivi de la souris dans le rendu plein ecran. Utile lorsque les evenements souris interferent avec les multiplexeurs de terminal ou les outils d'accessibilite |
| `CLAUDE_CODE_ACCESSIBILITY` | Definir a `1` pour garder le curseur natif du terminal visible pour les lecteurs d'ecran et les outils d'accessibilite |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | Definir a `0` pour desactiver la coloration syntaxique dans la sortie diff |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | Ignorer l'installation automatique de l'extension IDE (`1` pour ignorer) |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | Remplacer le comportement de connexion automatique a l'IDE |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | Remplacer l'adresse de l'hote IDE pour la connexion |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | Ignorer la validation du lockfile IDE (`1` pour ignorer) |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | Intervalle de debounce en ms pour le script de helper d'en-tetes OTel |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | Timeout en ms pour le flush OpenTelemetry |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | Timeout en ms pour l'arret OpenTelemetry |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | Timeout en ms avant que le watchdog de stream inactif ferme une connexion bloquee. Defaut : `90000` (90 secondes). Augmenter si des outils longs ou des reseaux lents causent des erreurs de timeout prematurees |
| `OTEL_LOG_TOOL_DETAILS` | Definir a `1` pour inclure `tool_parameters` dans les evenements OpenTelemetry. Omis par defaut pour la confidentialite *(dans le changelog v2.1.85, pas encore sur la page officielle des variables d'env)* |
| `CLAUDE_CODE_MCP_SERVER_NAME` | Nom du serveur MCP, passe comme variable d'environnement aux scripts `headersHelper` pour qu'ils puissent generer des en-tetes d'authentification specifiques au serveur *(dans le changelog v2.1.85, pas encore sur la page officielle des variables d'env)* |
| `CLAUDE_CODE_MCP_SERVER_URL` | URL du serveur MCP, passee comme variable d'environnement aux scripts `headersHelper` avec `CLAUDE_CODE_MCP_SERVER_NAME` *(dans le changelog v2.1.85, pas encore sur la page officielle des variables d'env)* |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Remplacer l'alias du modele Opus (ex. `claude-opus-4-6[1m]`) |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | Personnaliser le libelle de l'entree Opus dans le selecteur `/model` lors de l'utilisation d'un modele fixe sur Bedrock/Vertex/Foundry. Par defaut l'ID du modele |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | Personnaliser la description de l'entree Opus dans le selecteur `/model`. Par defaut `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | Remplacer la detection des capacites pour un modele Opus fixe. Valeurs separees par des virgules (ex. `effort,thinking`). Requis lorsque le modele fixe supporte des fonctionnalites que la detection automatique ne peut pas confirmer |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Remplacer l'alias du modele Sonnet (ex. `claude-sonnet-4-6`) |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | Personnaliser le libelle de l'entree Sonnet dans le selecteur `/model` lors de l'utilisation d'un modele fixe sur Bedrock/Vertex/Foundry. Par defaut l'ID du modele |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | Personnaliser la description de l'entree Sonnet dans le selecteur `/model`. Par defaut `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | Remplacer la detection des capacites pour un modele Sonnet fixe. Valeurs separees par des virgules (ex. `effort,thinking`). Requis lorsque le modele fixe supporte des fonctionnalites que la detection automatique ne peut pas confirmer |
| `MAX_THINKING_TOKENS` | Tokens de reflexion etendue max par reponse |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | Definir la capacite de contexte en tokens utilisee pour les calculs de compactage automatique. Par defaut la fenetre de contexte du modele (200K standard, 1M pour les modeles a contexte etendu). Utiliser une valeur plus basse (ex. `500000`) sur un modele 1M pour le traiter comme 500K pour le compactage. Plafonne a la fenetre de contexte reelle. `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` est applique en pourcentage de cette valeur. Definir ceci decouple le seuil de compactage du `used_percentage` de la ligne de statut |
| `DISABLE_AUTO_COMPACT` | Desactiver le compactage automatique du contexte (`1` pour desactiver). Le `/compact` manuel fonctionne toujours |
| `DISABLE_COMPACT` | Desactiver tout compactage — automatique et manuel (`1` pour desactiver) |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | Activer les suggestions de prompts |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | Exiger le mode plan pour les sessions |
| `CLAUDE_CODE_TEAM_NAME` | Nom de l'equipe pour les agent teams |
| `CLAUDE_CODE_TASK_LIST_ID` | ID de la liste de taches pour l'integration des taches |
| `CLAUDE_ENV_FILE` | Chemin personnalise du fichier d'environnement |
| `FORCE_AUTOUPDATE_PLUGINS` | Forcer la mise a jour automatique des plugins (`1` pour activer) |
| `HTTP_PROXY` | URL du proxy HTTP pour les requetes reseau |
| `HTTPS_PROXY` | URL du proxy HTTPS pour les requetes reseau |
| `NO_PROXY` | Liste d'hotes separee par des virgules qui contournent le proxy |
| `MCP_TOOL_TIMEOUT` | Timeout d'execution d'outil MCP en ms |
| `MCP_CLIENT_SECRET` | Secret client OAuth MCP |
| `MCP_OAUTH_CALLBACK_PORT` | Port de callback OAuth MCP |
| `IS_DEMO` | Activer le mode demo |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | Budget de caracteres pour la sortie d'outil des slash commands |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Remplacement de la region Vertex AI pour Claude 3.5 Haiku |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Remplacement de la region Vertex AI pour Claude 3.7 Sonnet |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Remplacement de la region Vertex AI pour Claude 4.0 Opus |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Remplacement de la region Vertex AI pour Claude 4.0 Sonnet |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Remplacement de la region Vertex AI pour Claude 4.1 Opus |

---

## Commandes utiles

| Commande | Description |
|----------|-------------|
| `/model` | Changer de modele et ajuster le niveau d'effort Opus 4.6 |
| `/effort` | Definir le niveau d'effort directement : `low`, `medium`, `high` (v2.1.76+) |
| `/config` | Interface de configuration interactive |
| `/memory` | Afficher/modifier tous les fichiers memoire |
| `/agents` | Gerer les subagents |
| `/mcp` | Gerer les serveurs MCP |
| `/hooks` | Afficher les hooks configures |
| `/plugin` | Gerer les plugins |
| `/keybindings` | Configurer les raccourcis clavier personnalises |
| `/skills` | Afficher et gerer les skills |
| `/permissions` | Afficher et gerer les regles de permissions |
| `--doctor` | Diagnostiquer les problemes de configuration |
| `--debug` | Mode debogage avec details d'execution des hooks |

---

## Reference rapide : exemple complet

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "agent": "code-reviewer",
  "language": "english",
  "cleanupPeriodDays": 30,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "includeGitInstructions": true,
  "defaultShell": "bash",
  "plansDirectory": "./plans",
  "effortLevel": "medium",

  "worktree": {
    "symlinkDirectories": ["node_modules"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  },

  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0"
  },

  "autoMode": {
    "environment": [
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted internal domains: *.internal.example.com"
    ]
  },

  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*",
      "Agent(*)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../shared/"],
    "defaultMode": "acceptEdits"
  },

  "enableAllProjectMcpServers": true,

  "sandbox": {
    "enabled": true,
    "excludedCommands": ["git", "docker"],
    "filesystem": {
      "denyRead": ["./secrets/"],
      "denyWrite": ["./.env"]
    }
  },

  "attribution": {
    "commit": "Generated with Claude Code",
    "pr": ""
  },

  "statusLine": {
    "type": "command",
    "command": "git branch --show-current"
  },

  "spinnerTipsEnabled": true,
  "spinnerTipsOverride": {
    "tips": ["Custom tip 1", "Custom tip 2"],
    "excludeDefault": false
  },
  "prefersReducedMotion": false,

  "env": {
    "NODE_ENV": "development",
    "CLAUDE_CODE_EFFORT_LEVEL": "medium"
  }
}
```

---

## Sources

- [Documentation des Settings Claude Code](https://code.claude.com/docs/en/settings)
- [Schema JSON des Settings Claude Code](https://json.schemastore.org/claude-code-settings.json)
- [Changelog Claude Code](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Exemples de Settings Claude Code sur GitHub](https://github.com/feiskyer/claude-code-settings)
- [Shipyard - Aide-memoire CLI Claude Code](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [Reference des variables d'environnement Claude Code](https://code.claude.com/docs/en/env-vars)
- [Reference des permissions Claude Code](https://code.claude.com/docs/en/permissions)
