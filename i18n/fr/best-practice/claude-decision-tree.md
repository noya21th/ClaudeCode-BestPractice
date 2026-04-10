🌍 [English](../../../best-practice/claude-decision-tree.md) | [中文](../../zh/best-practice/claude-decision-tree.md) | [日本語](../../ja/best-practice/claude-decision-tree.md) | **Français** | [Русский](../../ru/best-practice/claude-decision-tree.md) | [العربية](../../ar/best-practice/claude-decision-tree.md)

# Agent, Skill ou Command — Arbre de decision

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Quand utiliser un Agent, un Skill ou un Command — un cadre de decision avec des scenarios concrets.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Comparaison rapide

| Dimension | Agent | Command | Skill |
|-----------|-------|---------|-------|
| **Emplacement** | `.claude/agents/<nom>.md` | `.claude/commands/<nom>.md` | `.claude/skills/<nom>/SKILL.md` |
| **Contexte** | Contexte isole et neuf | Injecte dans le contexte courant | Injecte dans le contexte courant (ou fork avec `context: fork`) |
| **Invocation** | Outil `Agent(...)`, decouverte auto via la description | `/slash-command` par l'utilisateur | `/slash-command`, decouverte auto, ou prechargement par un agent |
| **Memoire** | Persistante (scopes `user`/`project`/`local`) | Aucune | Aucune |
| **Outils** | Liste blanche personnalisee via le champ `tools:` | Herite des outils de la session | Herite des outils de la session (ou restreint via `allowed-tools:`) |
| **Modele** | Configurable (`haiku`/`sonnet`/`opus`/`inherit`) | Configurable | Configurable |
| **Permissions** | Configurables (`acceptEdits`/`plan`/`bypassPermissions`) | Herite des permissions de la session | Herite des permissions de la session |
| **Decouverte auto** | Oui (via `description`, utiliser `PROACTIVELY`) | Non (declenchement utilisateur uniquement) | Oui (via `description`) |
| **Prechargeable** | Non | Non | Oui (via le champ `skills:` de l'agent) |
| **Arriere-plan** | Oui (`background: true`) | Non | Non |
| **Isolation worktree** | Oui (`isolation: worktree`) | Non | Non |
| **Ideal pour** | Taches complexes multi-etapes necessitant une isolation | Workflows declenches par l'utilisateur et orchestration | Connaissances et procedures de domaine reutilisables |

---

## Diagramme de decision

```
Depart : "Je dois ajouter une capacite a Claude Code"
│
├─ A-t-il besoin de son propre contexte isole ?
│  ├─ OUI ──→ A-t-il besoin d'une memoire persistante entre les sessions ?
│  │          ├─ OUI ──→ AGENT (avec memory: user/project/local)
│  │          └─ NON ──→ AGENT (ou Skill avec context: fork)
│  │
│  └─ NON ──→ Seul l'utilisateur doit-il le declencher explicitement ?
│             ├─ OUI ──→ COMMAND (l'utilisateur tape /slash-command)
│             └─ NON ──→ Claude doit-il le decouvrir et l'invoquer automatiquement ?
│                        ├─ OUI ──→ SKILL (avec description pour la decouverte)
│                        └─ NON ──→ Est-ce une connaissance de domaine reutilisable ?
│                                   ├─ OUI ──→ SKILL (avec user-invocable: false pour prechargement seul)
│                                   └─ NON ──→ COMMAND (simple template de prompt)
```

### Regles de decision rapides

- **Besoin d'isolation de contexte ?** → Agent
- **Besoin de memoire persistante ?** → Agent
- **Besoin d'execution en arriere-plan ?** → Agent
- **Besoin d'isolation worktree ?** → Agent
- **Besoin de restrictions d'outils personnalisees ?** → Agent
- **Workflow declenche par l'utilisateur ?** → Command
- **Procedure ou connaissance de domaine reutilisable ?** → Skill
- **Decouverte automatique par Claude ?** → Skill (ou Agent avec `PROACTIVELY` dans la description)
- **Prechargeable dans un agent ?** → Skill

---

## Scenarios concrets (12)

### 1. Revue de code

**Recommandation** : Agent

**Pourquoi** : La revue de code necessite un contexte isole afin que les constats ne polluent pas la conversation principale. L'agent peut disposer de ses propres outils (lecture seule), d'un modele (haiku pour la rapidite) et d'une memoire persistante pour apprendre les habitudes de revue de l'equipe.

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes for bugs, security, and style"
tools: Read, Glob, Grep, Bash
model: sonnet
memory: project
permissionMode: plan
---
```

### 2. Recuperation meteo (chaine Command → Agent → Skill)

**Recommandation** : Command orchestrant un Agent avec un Skill precharge

**Pourquoi** : L'utilisateur le declenche explicitement (Command), la logique de recuperation s'execute en isolation (Agent), et les instructions de recuperation de donnees sont des connaissances de domaine reutilisables (Skill precharge dans l'agent).

```
/weather-orchestrator  →  Agent(weather-agent)  →  Skill(weather-svg-creator)
     Command                   Agent                      Skill
```

### 3. Scaffolding de fichiers

**Recommandation** : Skill

**Pourquoi** : Les templates de scaffolding sont des connaissances de domaine reutilisables. Claude peut decouvrir automatiquement le skill lorsque l'utilisateur demande de creer un nouveau composant, une route ou un module. Pas besoin d'isolation — les fichiers sont ecrits directement dans le projet.

```yaml
# .claude/skills/react-scaffold/SKILL.md
---
name: react-scaffold
description: "Scaffold React components with TypeScript, tests, and Storybook stories"
paths: "src/components/**"
---
```

### 4. Orchestration CI/CD

**Recommandation** : Command

**Pourquoi** : Les workflows CI/CD sont toujours declenches par l'utilisateur (`/deploy`, `/release`). Ils suivent une sequence fixe et ont besoin du contexte de la session en cours (branche, modifications recentes).

```yaml
# .claude/commands/deploy.md
---
name: deploy
description: "Build, test, and deploy to staging or production"
argument-hint: "[staging|production]"
---
```

### 5. Tests navigateur

**Recommandation** : Agent

**Pourquoi** : Les tests navigateur sont de longue duree, peuvent necessiter une execution en arriere-plan et ne doivent pas bloquer la conversation principale. L'agent peut avoir des serveurs MCP specifiques (Playwright, Chrome DevTools) et s'executer dans un worktree pour l'isolation.

```yaml
# .claude/agents/browser-tester.md
---
name: browser-tester
description: "Run end-to-end browser tests against the dev server"
tools: Bash, Read, WebFetch, mcp__playwright__*
background: true
model: sonnet
---
```

### 6. Affichage de l'heure

**Recommandation** : Skill

**Pourquoi** : Leger, pas besoin d'isolation, decouvert automatiquement. Claude invoque le skill chaque fois que quelqu'un demande « quelle heure est-il ? »

### 7. Deploiement multi-etapes

**Recommandation** : Command orchestrant des Agents

**Pourquoi** : Le command est le point d'entree utilisateur (`/full-deploy`). Il lance des agents pour chaque etape — agent de build, agent de test, agent de deploiement — chacun s'executant dans son propre contexte avec les outils appropries.

### 8. Linting / Formatage

**Recommandation** : Skill (invoque automatiquement)

**Pourquoi** : Les regles de linting sont des connaissances de domaine reutilisables. Le skill s'active automatiquement via `paths:` lorsque Claude modifie des fichiers correspondants. Aucune interaction utilisateur necessaire.

```yaml
# .claude/skills/python-linting/SKILL.md
---
name: python-linting
description: "Apply Black formatting and ruff linting to Python files"
paths: "**/*.py"
---
```

### 9. Assistant de recherche

**Recommandation** : Agent

**Pourquoi** : Les taches de recherche sont de longue duree, beneficient de la memoire (rappel des recherches passees), necessitent des outils web (`WebFetch`, `WebSearch`) et doivent s'executer dans un contexte isole pour ne pas encombrer la conversation principale.

### 10. Migration de donnees

**Recommandation** : Agent avec isolation worktree

**Pourquoi** : Les migrations de base de donnees modifient des fichiers critiques et doivent etre testees en isolation. L'execution dans un worktree git signifie que la branche principale reste intacte jusqu'a ce que la migration soit verifiee.

```yaml
# .claude/agents/data-migrator.md
---
name: data-migrator
description: "Create and test database migration scripts"
isolation: worktree
model: sonnet
---
```

### 11. Generation de documentation

**Recommandation** : Skill

**Pourquoi** : La generation de documentation est une procedure reutilisable. Claude peut la decouvrir automatiquement lorsqu'on lui demande d'« ecrire la doc » ou de « generer une reference API ». Elle s'execute dans le contexte courant et ecrit les fichiers directement dans le projet.

### 12. Initialisation de projet

**Recommandation** : Command (avec Hook Setup)

**Pourquoi** : L'initialisation de projet est toujours declenchee par l'utilisateur et ne s'execute qu'une seule fois. Combinez un command `/setup` avec un Hook `Setup` pour l'executer automatiquement au premier clone.

---

## Patterns de composition

### Command → Agent → Skill

Le pattern d'orchestration canonique. Le command est le point d'entree utilisateur, l'agent fournit l'isolation et le controle des outils, et les skills apportent les connaissances de domaine.

```
L'utilisateur tape /command
  └─ Le prompt du Command s'execute dans le contexte courant
       └─ Agent(...) lance avec un contexte isole
            └─ Le Skill precharge fournit les instructions
                 └─ L'Agent termine, le resultat remonte au contexte du command
                      └─ L'outil Skill est invoque pour la sortie finale
```

### Agent avec Skills precharges

Un agent charge le contenu des skills au demarrage via le champ `skills:`. Le contenu du skill est injecte dans le system prompt de l'agent, lui donnant une expertise de domaine sans que l'utilisateur n'ait rien a invoquer.

```yaml
# .claude/agents/api-builder.md
---
name: api-builder
skills:
  - openapi-validator
  - rest-conventions
  - error-handling-patterns
---
```

### Skill avec fork de contexte

Un skill qui a besoin d'isolation mais pas de toute la lourdeur d'une definition d'agent. Le reglage `context: fork` execute le skill dans un contexte de sous-agent tout en conservant le format simple du fichier skill.

```yaml
# .claude/skills/security-scan/SKILL.md
---
name: security-scan
description: "Scan the codebase for security vulnerabilities"
context: fork
agent: general-purpose
---
```

---

## Anti-patterns dans le choix du mecanisme

| Anti-pattern | Probleme | Meilleure approche |
|--------------|----------|-------------------|
| Agent pour des taches simples | Surcout du contexte isole alors qu'il suffit d'injecter des instructions | Utiliser un Skill |
| Command pour la decouverte auto | Les commands ne peuvent pas etre invoques automatiquement par Claude | Utiliser un Skill avec une `description` |
| Skill pour des taches de longue duree | Les skills s'executent dans le contexte principal et bloquent la conversation | Utiliser un Agent avec `background: true` |
| Agent pour des conventions partagees | Le contexte de l'agent est isole — les conventions ne se propagent pas a la session principale | Utiliser CLAUDE.md ou des Rules |
| Command pour des connaissances prechargeables | Les commands ne peuvent pas etre precharges dans les agents | Utiliser un Skill avec `user-invocable: false` |
| Skill sans `description` | Claude ne peut pas le decouvrir automatiquement | Toujours ajouter une description pertinente |

---

## Sources

- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Slash Commands — Docs](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Common Workflows — Docs](https://code.claude.com/docs/en/common-workflows)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
