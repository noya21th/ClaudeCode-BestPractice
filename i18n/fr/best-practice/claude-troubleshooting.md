🌍 [English](../../../best-practice/claude-troubleshooting.md) | [中文](../../zh/best-practice/claude-troubleshooting.md) | [日本語](../../ja/best-practice/claude-troubleshooting.md) | **Français** | [Русский](../../ru/best-practice/claude-troubleshooting.md) | [العربية](../../ar/best-practice/claude-troubleshooting.md)

# Guide de depannage

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Guide de diagnostic pour les problemes courants de Claude Code — organise par symptome avec des solutions etape par etape.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Diagnostics rapides

Executez ces commandes en premier lorsque quelque chose ne fonctionne pas :

| Commande | Ce qu'elle verifie |
|----------|--------------------|
| `claude --version` | Version installee — comparez avec la [derniere version](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) |
| `/doctor` | Sante de l'installation, version de Node.js, authentification, validite des settings |
| `/status` | Modele actuel, compte, connectivite API, niveau d'abonnement |
| `/usage` | Limites du forfait, statut des limites de debit, quota restant |
| `/context` | Utilisation de la fenetre de contexte, fichiers memoire charges, Skills actifs |
| `/hooks` | Configurations de Hook actives et leurs sources |
| `/mcp` | Statut de connexion des serveurs MCP |

---

## Problemes d'installation

### Echec de l'installation de Claude Code

**Symptomes** : `npm install -g @anthropic-ai/claude-code` echoue avec des erreurs de permissions ou des conflits de version.

**Cause** : Version de Node.js trop ancienne, problemes de permissions npm, ou paquets globaux en conflit.

**Solution** :

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

### Commande Claude introuvable

**Symptomes** : Le terminal affiche `claude: command not found` apres une installation reussie.

**Cause** : Le repertoire global des binaires npm n'est pas dans votre PATH.

**Solution** :

```bash
# Find where npm installs global binaries
npm config get prefix

# Add to your shell profile (~/.zshrc, ~/.bashrc)
export PATH="$(npm config get prefix)/bin:$PATH"

# Reload shell
source ~/.zshrc
```

---

## Problemes d'authentification

### Cle API non reconnue

**Symptomes** : Claude Code demarre mais ne peut pas effectuer d'appels API. Les messages d'erreur mentionnent l'authentification ou une cle invalide.

**Cause** : Cle API manquante ou expiree. Mauvaise variable d'environnement.

**Solution** :

```bash
# Re-authenticate
claude /login

# Or set the API key directly
export ANTHROPIC_API_KEY="sk-ant-..."

# For Bedrock users
export CLAUDE_CODE_USE_BEDROCK=1
claude /setup-bedrock
```

### Erreurs de limite de debit

**Symptomes** : Erreurs `429 Too Many Requests`. Claude cesse de repondre en cours de tache.

**Cause** : Limites de debit du forfait depassees. Trop de sessions ou agents concurrents.

**Solution** :

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

## Problemes de performance

### Reponses lentes

**Symptomes** : Claude met plus de 30 secondes a repondre. Longues pauses entre les appels d'outils.

**Cause** : Fenetre de contexte trop grande. Trop de Skills charges. Hooks synchrones bloquant les reponses.

**Solution** :

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

### Consommation elevee de tokens

**Symptomes** : Le quota s'epuise plus vite que prevu. `/usage` montre une consommation elevee.

**Cause** : Fichiers CLAUDE.md volumineux. Trop de Skills precharges. Agents lances avec heritage complet du contexte. Pas de compaction.

**Solution** :

1. Gardez CLAUDE.md sous 200 lignes
2. Utilisez `user-invocable: false` sur les Skills qui ne doivent etre precharges que par des agents specifiques
3. Definissez `model: haiku` sur les agents qui n'ont pas besoin de toutes les capacites
4. Compactez regulierement avec `/compact`
5. Utilisez `effort: low` ou `effort: medium` pour les taches routinieres

---

## Problemes de Hook

### Le Hook ne se declenche pas

**Symptomes** : Vous avez configure un Hook dans les settings mais il ne s'execute jamais.

**Cause** : Mauvais fichier de settings. Matcher manquant ou mal defini. Hook desactive. Surcharge par la hierarchie des settings.

**Solution** :

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

### Le Hook bloque les reponses

**Symptomes** : Longues pauses apres les appels d'outils. Claude semble fige pendant plusieurs secondes.

**Cause** : Hook synchrone executant une commande lente. Appel reseau dans un Hook non asynchrone.

**Solution** :

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

### Timeout du Hook

**Symptomes** : Le Hook demarre mais le resultat n'est jamais utilise. Avertissement de timeout du Hook dans les logs.

**Cause** : La commande du Hook prend trop de temps. Service externe indisponible. Le script se bloque.

**Solution** : Testez la commande du Hook manuellement. Ajoutez un timeout dans le script lui-meme. Assurez-vous que les services externes sont accessibles. Envisagez de rendre le Hook asynchrone si le resultat n'est pas critique.

---

## Problemes MCP

### Le serveur MCP ne se connecte pas

**Symptomes** : Les outils MCP ne sont pas disponibles. `/mcp` affiche le serveur comme deconnecte.

**Cause** : Binaire du serveur introuvable. Mauvaise configuration. Conflit de port. Variables d'environnement manquantes.

**Solution** :

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

### Permission MCP refusee

**Symptomes** : Les outils MCP apparaissent mais echouent avec des erreurs de permission lors de l'invocation.

**Cause** : Outil absent de la liste autorisee. Mode de permission trop restrictif.

**Solution** : Ajoutez l'outil MCP a la liste `allowedTools` dans les settings, ou approuvez-le lorsque demande. Pour les agents, incluez `mcp__<server>__<tool>` dans le champ `tools:`.

---

## Problemes d'Agent

### L'Agent ne s'invoque pas automatiquement

**Symptomes** : Vous avez cree un Agent avec une description contenant `PROACTIVELY` mais Claude ne le lance jamais automatiquement.

**Cause** : La description ne decrit pas clairement quand l'invoquer. L'outil Agent n'est pas disponible dans le contexte actuel. Un autre mecanisme gere la tache.

**Solution** :

1. Assurez-vous que le champ `description` utilise `PROACTIVELY` et indique clairement la condition de declenchement
2. Verifiez que le fichier de l'agent est dans `.claude/agents/` avec une extension `.md`
3. Verifiez que Claude a acces a l'outil `Agent` (non restreint par le champ `tools:` de l'agent parent)

### L'Agent lance Bash au lieu de l'outil Agent

**Symptomes** : Claude execute `claude --agent my-agent` via Bash au lieu d'utiliser l'outil `Agent(...)`.

**Cause** : La definition de l'agent utilise un langage vague comme « lancer » ou « executer » que Claude interprete comme une commande shell.

**Solution** : Dans votre CLAUDE.md ou les instructions de l'agent, indiquez explicitement : « Utilisez l'outil Agent, pas des commandes bash, pour lancer les sous-agents. » Evitez les termes comme « lancer » ou « executer » dans les descriptions d'agents.

### L'Agent depasse maxTurns

**Symptomes** : L'Agent s'arrete en cours de tache avec un resultat incomplet.

**Cause** : `maxTurns` defini trop bas pour la complexite de la tache. L'Agent boucle ou reessaie.

**Solution** : Augmentez `maxTurns` dans le frontmatter de l'agent. Examinez le comportement de l'agent pour reperer les boucles inutiles. Decoupez les taches complexes en sous-taches plus petites.

---

## Problemes de Skill

### Le Skill n'apparait pas dans le menu /

**Symptomes** : Vous avez cree un Skill mais il n'apparait pas quand vous tapez `/`.

**Cause** : `user-invocable: false` est defini. Le fichier du Skill n'est pas au bon emplacement. Erreur de syntaxe dans le frontmatter YAML.

**Solution** :

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

### Le Skill n'est pas decouvert automatiquement

**Symptomes** : Le Skill existe et a une description mais Claude ne l'invoque jamais automatiquement.

**Cause** : `disable-model-invocation: true` est defini. La description ne correspond pas au contexte rencontre par Claude. Trop de Skills en concurrence pour le meme declencheur.

**Solution** :

1. Supprimez `disable-model-invocation: true` si la decouverte automatique est souhaitee
2. Redigez une `description` claire utilisant des mots-cles que Claude rencontrerait naturellement
3. Reduisez le nombre de Skills — trop de Skills auto-decouvrables cree de l'ambiguite

---

## Problemes de memoire

### CLAUDE.md ne se charge pas

**Symptomes** : Les instructions dans CLAUDE.md sont ignorees. Claude ne respecte pas les conventions du projet.

**Cause** : Le fichier est dans un repertoire descendant (chargement paresseux, pas eagre). Le nom du fichier contient une faute de frappe. Surcharge via une politique managed.

**Solution** :

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

### CLAUDE.local.md commite par erreur

**Symptomes** : Des preferences personnelles apparaissent chez les coequipiers. `.claude/settings.local.json` dans l'historique git.

**Cause** : Les fichiers locaux ne sont pas dans `.gitignore`.

**Solution** :

```bash
# Add to .gitignore
echo "CLAUDE.local.md" >> .gitignore
echo ".claude/settings.local.json" >> .gitignore

# Remove from tracking if already committed
git rm --cached CLAUDE.local.md
git rm --cached .claude/settings.local.json
```

---

## Problemes Git

### Nettoyage des worktrees

**Symptomes** : Des worktrees git obsoletes restent apres l'execution d'un agent. L'espace disque augmente.

**Cause** : Un Agent avec `isolation: worktree` a plante ou a ete interrompu avant le nettoyage.

**Solution** :

```bash
# List worktrees
git worktree list

# Remove stale worktrees
git worktree prune

# Force remove a specific worktree
git worktree remove /path/to/worktree --force
```

### Auteur des commits

**Symptomes** : Les commits git sont attribues au mauvais auteur quand Claude Code les cree.

**Cause** : L'auteur git n'est pas configure pour les sessions Claude Code.

**Solution** :

```bash
# Set git author for the project
git config user.name "Your Name"
git config user.email "your@email.com"

# Or use --author flag in hooks/scripts
git commit --author="Your Name <your@email.com>" -m "message"
```

---

## Sources

- [Claude Code Troubleshooting — Docs](https://code.claude.com/docs/en/troubleshooting)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Hooks — Docs](https://code.claude.com/docs/en/hooks)
- [Claude Code MCP — Docs](https://code.claude.com/docs/en/mcp)
- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
