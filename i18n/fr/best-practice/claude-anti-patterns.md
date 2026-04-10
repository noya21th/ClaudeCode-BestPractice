🌍 [English](../../../best-practice/claude-anti-patterns.md) | [中文](../../zh/best-practice/claude-anti-patterns.md) | [日本語](../../ja/best-practice/claude-anti-patterns.md) | **Français** | [Русский](../../ru/best-practice/claude-anti-patterns.md) | [العربية](../../ar/best-practice/claude-anti-patterns.md)

# Guide des anti-patterns

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Erreurs courantes dans la configuration de Claude Code et comment les eviter — classees par categorie.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Anti-patterns d'architecture

### 1. CLAUDE.md monolithique

**Probleme** : Un seul fichier CLAUDE.md depassant 500 lignes avec des instructions d'installation, des conventions de code, de la documentation d'API, des etapes de deploiement et des conseils de debogage melanges.

**Symptomes** : Claude ignore les instructions en fin de fichier. Respect incoherent des conventions. Surcharge du contexte au demarrage de la session.

**Correction** : Scindez en fichiers cibles. Gardez le CLAUDE.md racine sous 200 lignes pour les conventions partagees. Utilisez des fichiers CLAUDE.md au niveau des composants pour les details. Utilisez `.claude/rules/*.md` avec des patterns glob pour les directives specifiques aux types de fichiers.

```
# Avant (monolithique)
CLAUDE.md              # 800 lignes couvrant tout

# Apres (scinde)
CLAUDE.md              # 150 lignes — conventions partagees, apercu du projet
frontend/CLAUDE.md     # 80 lignes — patterns React, structure des composants
backend/CLAUDE.md      # 80 lignes — conventions API, patterns de base de donnees
.claude/rules/ts.md    # Regles specifiques TypeScript (Glob: **/*.ts)
.claude/rules/test.md  # Conventions de test (Glob: **/*.test.*)
```

### 2. Agent pour tout

**Probleme** : Utiliser des agents pour des taches simples qui n'ont besoin que d'instructions injectees dans le contexte courant.

**Symptomes** : Isolation de contexte inutile. Resultats non visibles dans la conversation principale. Execution lente due au surcout de demarrage de l'agent. Gaspillage de tokens par duplication du contexte.

**Correction** : Utilisez un Skill pour les connaissances et procedures de domaine reutilisables. Utilisez un Command pour les templates de prompt declenches par l'utilisateur. Reservez les Agents aux taches qui necessitent reellement une isolation, de la memoire, des restrictions d'outils ou une execution en arriere-plan.

### 3. Absence d'isolation de contexte

**Probleme** : Executer des taches exploratoires de longue duree (recherche, revue de code, analyse de donnees) directement dans la conversation principale sans isolation.

**Symptomes** : Le contexte de la conversation se remplit rapidement. Des resultats intermediaires non pertinents polluent les reponses futures. Necessite d'utiliser `/compact` frequemment.

**Correction** : Utilisez un Agent pour les taches qui generent un volume important de resultats intermediaires. L'agent s'execute dans un contexte neuf et ne renvoie que le resume.

### 4. Lancement circulaire d'agents

**Probleme** : L'Agent A lance l'Agent B, qui lance l'Agent A, creant une boucle infinie.

**Symptomes** : Limite `maxTurns` atteinte. Budget de tokens epuise. Aucun resultat utile.

**Correction** : Concevez des workflows en DAG (graphe oriente acyclique). Utilisez le champ `tools:` pour restreindre les agents qu'un sous-agent peut lancer. Verifiez les definitions d'agents pour les references circulaires `Agent(...)`.

```yaml
# Restreindre les agents lancables
---
name: planner
tools: Read, Glob, Grep, Agent(implementer)
# planner ne peut lancer que implementer, pas lui-meme
---
```

### 5. Noms de modeles codes en dur

**Probleme** : Utiliser des identifiants complets de modeles comme `claude-sonnet-4-20250514` dans les definitions d'agents.

**Symptomes** : Les agents cessent de fonctionner quand les identifiants de modeles sont deprecies. Comportement incoherent entre les environnements.

**Correction** : Utilisez les alias : `haiku`, `sonnet`, `opus`, ou `inherit`. Ils resolvent toujours vers la derniere version.

---

## Anti-patterns de configuration

### 6. Permissions trop permissives

**Probleme** : Definir `"permissionMode": "bypassPermissions"` sur tous les agents et utiliser `--dangerously-skip-permissions` par defaut.

**Symptomes** : Suppressions de fichiers non revisees. `git push --force` involontaire. Risque de securite lie aux outils MCP non fiables. Aucune trace d'audit des approbations d'outils.

**Correction** : Utilisez `acceptEdits` pour les agents qui doivent ecrire des fichiers mais ne doivent pas executer de commandes arbitraires. Utilisez `plan` pour les agents de recherche uniquement. Reservez `bypassPermissions` aux environnements CI/CD avec bac a sable.

| Mode | Editions | Bash | Ideal pour |
|------|----------|------|------------|
| `default` | Demande | Demande | Developpement interactif |
| `acceptEdits` | Auto | Demande | Agents d'ecriture de fichiers |
| `plan` | Refuse | Refuse | Agents de recherche et planification |
| `bypassPermissions` | Auto | Auto | CI/CD avec bac a sable uniquement |

### 7. Ignorer la hierarchie des settings

**Probleme** : Definir des reglages dans le mauvais fichier et se demander pourquoi ils sont surcharges ou ignores.

**Symptomes** : Les Hooks ne se declenchent pas. Les permissions ne sont pas celles attendues. Les membres de l'equipe ont un comportement different.

**Correction** : Comprenez l'ordre de priorite (le plus eleve l'emporte) :

1. **Managed** (`managed-settings.json`) — Impose par l'organisation
2. **Options CLI** — Surcharges pour une seule session
3. **Local** (`.claude/settings.local.json`) — Reglages personnels du projet
4. **Partage** (`.claude/settings.json`) — Reglages du projet partages en equipe
5. **Global** (`~/.claude/settings.json`) — Valeurs par defaut personnelles

### 8. Tous les Hooks synchrones

**Probleme** : Chaque Hook s'execute de maniere synchrone, bloquant la reponse de Claude pendant l'execution des scripts.

**Symptomes** : Reponses lentes. Pause perceptible apres chaque appel d'outil. L'utilisateur percoit Claude comme lent.

**Correction** : Activez `"async": true` sur les Hooks qui effectuent des effets de bord (journalisation, notifications, telemetrie). Ne gardez synchrones que les Hooks qui doivent conditionner ou modifier un comportement (verifications de securite, decisions de permissions).

### 9. Pas de settings.local.json

**Probleme** : Des preferences personnelles (sons de notification, chemins personnalises, journalisation de debogage) commitees dans `.claude/settings.json` ou elles affectent toute l'equipe.

**Symptomes** : Vos coequipiers entendent vos sons de notification. Les Hooks de debogage s'executent en production. Vos serveurs MCP personnels apparaissent pour tout le monde.

**Correction** : Utilisez `.claude/settings.local.json` pour les surcharges personnelles. Assurez-vous qu'il est dans `.gitignore`. Ne partagez que les Hooks et reglages dont toute l'equipe a besoin.

---

## Anti-patterns de Skill

### 10. user-invocable: true sur des Skills reserves aux agents

**Probleme** : Un Skill destine uniquement au prechargement par un agent (via le champ `skills:`) apparait dans le menu `/` de l'utilisateur.

**Symptomes** : Les utilisateurs invoquent le Skill directement et obtiennent des resultats confus car il etait concu comme connaissance de fond pour un agent.

**Correction** : Definissez `user-invocable: false` sur les Skills destines uniquement au prechargement par les agents.

```yaml
---
name: internal-api-conventions
user-invocable: false
description: "REST API conventions for the internal platform"
---
```

### 11. Description manquante

**Probleme** : Un Skill n'a pas de champ `description` dans son frontmatter.

**Symptomes** : Claude ne peut pas decouvrir automatiquement le Skill. Il ne se declenche que lorsque l'utilisateur tape explicitement le `/slash-command`. Le Skill est invisible pour la logique d'invocation automatique.

**Correction** : Incluez toujours une `description` qui explique **quand** utiliser le Skill. Utilisez des mots-cles que Claude rencontrerait naturellement : « Utiliser lors de la creation de composants React » ou « Se declenche sur les fichiers de test Python. »

### 12. Fichiers Skill geants

**Probleme** : Un seul fichier SKILL.md de plus de 1000 lignes couvrant chaque cas limite, exemple et reference.

**Symptomes** : Surcharge du contexte au chargement du Skill. Claude peut ignorer les instructions en fin de fichier. Chargement lent lors du prechargement dans les agents.

**Correction** : Gardez SKILL.md concentre sur la procedure principale (moins de 200 lignes). Deplacez les materiaux de reference dans des fichiers `references/` a l'interieur du repertoire du Skill. Utilisez les imports `@path` pour le contenu supplementaire que Claude charge a la demande.

### 13. disable-model-invocation sur des Skills frequemment necessaires

**Probleme** : Definir `disable-model-invocation: true` sur un Skill que Claude devrait decouvrir et invoquer automatiquement.

**Symptomes** : Claude n'invoque jamais le Skill automatiquement. Les utilisateurs doivent toujours taper le `/slash-command` manuellement meme quand le contexte appelle clairement le Skill.

**Correction** : N'utilisez `disable-model-invocation: true` que sur les Skills purement declenches par l'utilisateur (operations dangereuses, deploiement, etc.). Pour les Skills que Claude doit decouvrir de lui-meme, ne le definissez pas ou mettez-le a `false`.

---

## Anti-patterns operationnels

### 14. Ne jamais compacter

**Probleme** : Executer une longue session sans compacter, laissant le contexte se remplir a 100 %.

**Symptomes** : Claude commence a oublier les instructions precedentes. Les reponses deviennent moins precises. La session finit par echouer avec un debordement de contexte.

**Correction** : Executez `/compact` manuellement a environ 50 % d'utilisation du contexte. Utilisez `/context` pour visualiser l'utilisation actuelle. Pour les taches longues, decoupez le travail en sous-taches qui se completent chacune dans 50 % de la capacite du contexte.

### 15. Pas de boucles de verification

**Probleme** : Accepter la sortie de Claude sans verifier qu'elle fonctionne — pas d'execution de tests, pas de verification de types, pas d'inspection visuelle.

**Symptomes** : Le code genere contient des erreurs de syntaxe. Les integrations d'API utilisent les mauvais endpoints. Les fichiers de configuration contiennent des fautes de frappe.

**Correction** : Verifiez toujours la sortie. Executez les tests apres la generation de code. Utilisez l'automatisation navigateur (Chrome, Playwright) pour la verification visuelle. Incluez des etapes de verification dans vos commands et definitions d'agents.

### 16. Ignorer les limites de debit

**Probleme** : Planifier de grosses taches multi-agents sans tenir compte du budget de tokens et des limites de debit.

**Symptomes** : Les agents atteignent les limites de debit en pleine tache. Le travail est perdu ou incomplet. Surprises de facturation.

**Correction** : Utilisez `/usage` pour verifier votre statut actuel de limites de debit. Planifiez les taches pour rester dans votre budget. Utilisez `model: haiku` pour les sous-taches a fort volume et faible complexite. Decoupez les grosses taches en sessions plus petites.

---

## Sources

- [Claude Code Best Practices — Docs](https://code.claude.com/docs/en/best-practices)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Permissions — Docs](https://code.claude.com/docs/en/permissions)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
