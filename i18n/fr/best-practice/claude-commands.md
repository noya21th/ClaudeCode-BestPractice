🌍 [English](../../../best-practice/claude-commands.md) | [中文](../../zh/best-practice/claude-commands.md) | [日本語](../../ja/best-practice/claude-commands.md) | **Français** | [Русский](../../ru/best-practice/claude-commands.md) | [العربية](../../ar/best-practice/claude-commands.md)

# Bonnes pratiques des Commands

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A35%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-commands-implementation.md)

Commands Claude Code — champs frontmatter et slash commands officielles integrees.

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
| `description` | string | Recommande | Ce que fait la command. Affiche dans l'autocompletion et utilise par Claude pour la decouverte automatique |
| `argument-hint` | string | Non | Indication affichee pendant l'autocompletion (ex. `[issue-number]`, `[filename]`) |
| `disable-model-invocation` | boolean | Non | Definir a `true` pour empecher Claude d'invoquer automatiquement cette command |
| `user-invocable` | boolean | Non | Definir a `false` pour masquer du menu `/` — la command devient uniquement une connaissance de fond |
| `paths` | string/list | Non | Patterns glob qui limitent l'activation de ce skill. Accepte une chaine separee par des virgules ou une liste YAML. Quand defini, Claude charge le skill automatiquement uniquement lorsqu'il travaille avec des fichiers correspondant aux patterns |
| `allowed-tools` | string | Non | Outils autorises sans demande de permission lorsque cette command est active |
| `model` | string | Non | Modele a utiliser lorsque cette command s'execute (ex. `haiku`, `sonnet`, `opus`) |
| `effort` | string | Non | Remplace le niveau d'effort du modele lors de l'invocation (`low`, `medium`, `high`, `max`) |
| `context` | string | Non | Definir a `fork` pour executer la command dans un contexte de subagent isole |
| `agent` | string | Non | Type de subagent quand `context: fork` est defini (defaut : `general-purpose`) |
| `shell` | string | Non | Shell pour les blocs `` !`command` `` — accepte `bash` (defaut) ou `powershell`. Necessite `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | Non | Hooks de cycle de vie limites a cette command |

---

## ![Official](../../../!/tags/official.svg) **(65)**

| # | Command | Tag | Description |
|---|---------|-----|-------------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Connectez-vous a votre compte Anthropic |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Deconnectez-vous de votre compte Anthropic |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Configurez l'authentification Amazon Bedrock, la region et les modeles via un assistant interactif. Visible uniquement quand `CLAUDE_CODE_USE_BEDROCK=1` est defini. Les nouveaux utilisateurs Bedrock peuvent aussi acceder a cet assistant depuis l'ecran de connexion |
| 4 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Ouvrez la page de mise a niveau pour passer a un plan superieur |
| 5 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Definissez la couleur de la barre de prompt pour la session en cours. Couleurs disponibles : `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`. Utilisez `default` pour reinitialiser |
| 6 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Ouvrez l'interface Settings pour ajuster le theme, le modele, le style de sortie et d'autres preferences. Alias : `/settings` |
| 7 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Ouvrez ou creez votre fichier de configuration des raccourcis clavier |
| 8 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Gerez les regles allow, ask et deny pour les permissions d'outils. Ouvre un dialogue interactif ou vous pouvez voir les regles par portee, ajouter ou supprimer des regles, gerer les repertoires de travail et consulter les refus recents en auto mode. Alias : `/allowed-tools` |
| 9 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Consultez et mettez a jour vos parametres de confidentialite. Disponible uniquement pour les abonnes Pro et Max |
| 10 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Activez/desactivez le mode sandbox. Disponible uniquement sur les plateformes supportees |
| 11 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Configurez la status line de Claude Code. Decrivez ce que vous voulez, ou executez sans arguments pour auto-configurer depuis votre prompt shell |
| 12 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Commandez des stickers Claude Code |
| 13 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Configurez les raccourcis clavier du terminal pour Shift+Enter et d'autres raccourcis. Visible uniquement dans les terminaux qui en ont besoin, comme VS Code, Alacritty ou Warp |
| 14 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Changez le theme de couleurs. Inclut des variantes claires et sombres, des themes accessibles aux daltoniens (daltonises) et des themes ANSI utilisant la palette de couleurs de votre terminal |
| 15 | `/voice` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Activez/desactivez la dictee vocale appuyer-pour-parler. Necessite un compte Claude.ai |
| 16 | `/context` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Visualisez l'utilisation actuelle du contexte sous forme de grille coloree. Affiche des suggestions d'optimisation pour les outils gourmands en contexte, le gonflement de la memoire et les avertissements de capacite |
| 17 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Affichez les statistiques d'utilisation des tokens. Consultez le guide de suivi des couts pour les details specifiques a l'abonnement |
| 18 | `/extra-usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Configurez l'utilisation supplementaire pour continuer a travailler quand les limites de taux sont atteintes |
| 19 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Generez un rapport analysant vos sessions Claude Code, incluant les zones de projet, les patterns d'interaction et les points de friction |
| 20 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Visualisez l'utilisation quotidienne, l'historique des sessions, les series et les preferences de modele |
| 21 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Ouvrez l'interface Settings (onglet Status) affichant la version, le modele, le compte et la connectivite. Fonctionne pendant que Claude repond, sans attendre la fin de la reponse en cours |
| 22 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Affichez les limites d'utilisation du plan et le statut des limites de taux |
| 23 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Diagnostiquez et verifiez votre installation et configuration de Claude Code |
| 24 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Soumettez des commentaires sur Claude Code. Alias : `/bug` |
| 25 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Affichez l'aide et les commandes disponibles |
| 26 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Decouvrez les fonctionnalites de Claude Code a travers des lecons interactives rapides avec des demos animees |
| 27 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Consultez le changelog dans un selecteur de version interactif. Selectionnez une version specifique pour voir ses notes de publication, ou choisissez d'afficher toutes les versions |
| 28 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Listez et gerez les taches en arriere-plan. Alias : `/bashes` |
| 29 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | Copiez la derniere reponse de l'assistant dans le presse-papiers. Passez un nombre `N` pour copier la N-ieme reponse la plus recente : `/copy 2` copie l'avant-derniere. Quand des blocs de code sont presents, affiche un selecteur interactif pour choisir des blocs individuels ou la reponse complete. Appuyez sur `w` dans le selecteur pour ecrire la selection dans un fichier au lieu du presse-papiers, utile en SSH |
| 30 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | Exportez la conversation en cours en texte brut. Avec un nom de fichier, ecrit directement dans ce fichier. Sans, ouvre un dialogue pour copier dans le presse-papiers ou sauvegarder dans un fichier |
| 31 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Gerez les configurations d'agents |
| 32 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Configurez les parametres de Claude in Chrome |
| 33 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Consultez les configurations de hooks pour les evenements d'outils |
| 34 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Gerez les integrations IDE et affichez le statut |
| 35 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Gerez les connexions MCP server et l'authentification OAuth |
| 36 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Gerez les plugins Claude Code |
| 37 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Rechargez tous les plugins actifs pour appliquer les changements en attente sans redemarrer. Rapporte les comptes pour chaque composant recharge et signale les erreurs de chargement |
| 38 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Listez les skills disponibles |
| 39 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | Editez les fichiers memoire `CLAUDE.md`, activez ou desactivez l'auto-memory, et consultez les entrees d'auto-memory |
| 40 | `/effort [low\|medium\|high\|max\|auto]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Definissez le niveau d'effort du modele. `low`, `medium` et `high` persistent entre les sessions. `max` s'applique uniquement a la session en cours et necessite Opus 4.6. `auto` reinitialise au defaut du modele. Sans argument, affiche le niveau actuel. Prend effet immediatement sans attendre la fin de la reponse en cours |
| 41 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Activez/desactivez le fast mode |
| 42 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Selectionnez ou changez le modele IA. Pour les modeles qui le supportent, utilisez les fleches gauche/droite pour ajuster le niveau d'effort. Le changement prend effet immediatement sans attendre la fin de la reponse en cours |
| 43 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Partagez une semaine gratuite de Claude Code avec des amis. Visible uniquement si votre compte est eligible |
| 44 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Entrez en plan mode directement depuis le prompt. Passez une description optionnelle pour entrer en plan mode et commencer immediatement avec cette tache, par exemple `/plan corriger le bug d'auth` |
| 45 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Elaborez un plan dans une session ultraplan, revisez-le dans votre navigateur, puis executez a distance ou renvoyez-le a votre terminal |
| 46 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Ajoutez un nouveau repertoire de travail a la session en cours |
| 47 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Ouvrez un visualiseur de diff interactif montrant les changements non commites et les diffs par tour. Utilisez les fleches gauche/droite pour basculer entre le diff git actuel et les tours individuels de Claude, et haut/bas pour parcourir les fichiers |
| 48 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Initialisez le projet avec un guide `CLAUDE.md`. Definissez `CLAUDE_CODE_NEW_INIT=1` pour un flux interactif qui guide aussi a travers les skills, hooks et fichiers de memoire personnelle |
| 49 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Obsolete. Installez plutot le plugin `code-review` : `claude plugin install code-review@claude-plugins-official` |
| 50 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Analysez les changements en attente sur la branche actuelle pour les vulnerabilites de securite. Examine le diff git et identifie les risques comme l'injection, les problemes d'auth et l'exposition de donnees |
| 51 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Continuez la session en cours dans l'app Claude Code Desktop. macOS et Windows uniquement. Alias : `/app` |
| 52 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Configurez l'app Claude GitHub Actions pour un depot. Vous guide dans la selection d'un repo et la configuration de l'integration |
| 53 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Installez l'app Claude Slack. Ouvre un navigateur pour completer le flux OAuth |
| 54 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Affichez le QR code pour telecharger l'app mobile Claude. Alias : `/ios`, `/android` |
| 55 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Rendez cette session disponible pour le controle a distance depuis claude.ai. Alias : `/rc` |
| 56 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Configurez l'environnement distant par defaut pour les sessions web demarrees avec `--remote` |
| 57 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Creez, mettez a jour, listez ou executez des taches planifiees Cloud. Claude vous guide dans la configuration de maniere conversationnelle |
| 58 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Creez une branche de la conversation actuelle a ce point. Alias : `/fork` |
| 59 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Posez une question rapide sans l'ajouter a la conversation |
| 60 | `/clear` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Effacez l'historique de conversation et liberez du contexte. Alias : `/reset`, `/new` |
| 61 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Compactez la conversation avec des instructions de focus optionnelles |
| 62 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Quittez le CLI. Alias : `/quit` |
| 63 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Renommez la session en cours et affichez le nom dans la barre de prompt. Sans nom, en genere un automatiquement a partir de l'historique de conversation |
| 64 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Reprenez une conversation par ID ou nom, ou ouvrez le selecteur de session. Alias : `/continue` |
| 65 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Rembobinez la conversation et/ou le code a un point precedent, ou resumez a partir d'un message selectionne. Voir checkpointing. Alias : `/checkpoint` |

Les skills integres comme `/debug` peuvent aussi apparaitre dans le menu slash-command, mais ce ne sont pas des commands integrees.

---

## Sources

- [Slash Commands Claude Code](https://code.claude.com/docs/en/slash-commands)
- [Mode interactif Claude Code](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
