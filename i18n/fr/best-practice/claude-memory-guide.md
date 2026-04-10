🌍 [English](../../../best-practice/claude-memory-guide.md) | [中文](../../zh/best-practice/claude-memory-guide.md) | [日本語](../../ja/best-practice/claude-memory-guide.md) | **Français** | [Русский](../../ru/best-practice/claude-memory-guide.md) | [العربية](../../ar/best-practice/claude-memory-guide.md)

# Guide de la memoire et de la gestion du contexte

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Guide complet des systemes de memoire de Claude Code — fichiers CLAUDE.md, memoire automatique, memoire d'agent et rules.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Vue d'ensemble des types de memoire

Claude Code dispose de quatre systemes de memoire distincts, chacun repondant a un objectif different :

| Type de memoire | Emplacement | Portee | Persistance | Charge quand |
|-----------------|-------------|--------|-------------|--------------|
| **CLAUDE.md** | Repertoires du projet | Par projet, hierarchique | Permanente (fichier) | Demarrage de session (ancetres) ou acces aux fichiers (descendants) |
| **Memoire automatique** | `~/.claude/memory/` et `~/.claude/projects/<hash>/memory/` | Globale utilisateur ou par projet | Permanente (fichier) | Demarrage de session |
| **Memoire d'agent** | `~/.claude/agent-memory/<agent>/` ou `.claude/agent-memory/<agent>/` | Par agent, definie par le champ `memory:` | Permanente (fichier) | Demarrage de l'agent |
| **Rules** | `.claude/rules/*.md` et `~/.claude/rules/*.md` | Ciblee par pattern de fichiers | Permanente (fichier) | Quand des fichiers correspondant aux patterns glob sont accedes |

---

## Hierarchie CLAUDE.md

### Emplacements des fichiers

| Niveau | Chemin | Partage | Description |
|--------|--------|---------|-------------|
| **Global** | `~/.claude/CLAUDE.md` | Non (personnel) | S'applique a toutes les sessions Claude Code sur cette machine |
| **Racine du projet** | `./CLAUDE.md` | Oui (suivi par git) | Instructions partagees pour tout le projet |
| **Local au projet** | `./CLAUDE.local.md` | Non (ignore par git) | Surcharges personnelles specifiques au projet |
| **Composant** | `./frontend/CLAUDE.md` | Oui (suivi par git) | Instructions specifiques au composant |
| **Imbrique** | `./src/api/v2/CLAUDE.md` | Oui (suivi par git) | Instructions specifiques a un sous-repertoire |

### Mecanismes de chargement

Claude Code utilise deux mecanismes distincts pour decouvrir les fichiers CLAUDE.md :

#### Chargement ancetre (vers le haut) — Eagre

Au demarrage d'une session, Claude remonte depuis le repertoire de travail courant jusqu'a la racine du systeme de fichiers et charge chaque CLAUDE.md qu'il trouve. Ceux-ci sont charges **immediatement au demarrage**.

```
/home/user/projects/myapp/src/   ← cwd
/home/user/projects/myapp/       ← CLAUDE.md charge (ancetre)
/home/user/projects/             ← CLAUDE.md charge (ancetre)
/home/user/                      ← CLAUDE.md charge s'il existe (ancetre)
~/.claude/                       ← CLAUDE.md charge (global)
```

#### Chargement descendant (vers le bas) — Paresseux

Les fichiers CLAUDE.md dans les sous-repertoires en dessous du cwd ne sont **pas** charges au demarrage. Ils ne sont inclus que lorsque Claude lit ou modifie des fichiers dans ces sous-repertoires au cours de la session.

```
/myapp/                 ← cwd, CLAUDE.md charge au demarrage
/myapp/frontend/        ← CLAUDE.md charge quand Claude accede aux fichiers de frontend/
/myapp/backend/         ← CLAUDE.md charge quand Claude accede aux fichiers de backend/
/myapp/backend/api/     ← CLAUDE.md charge quand Claude accede aux fichiers de backend/api/
```

#### Regles cles

- **Les ancetres se chargent toujours au demarrage** — de maniere eagre, en remontant depuis le cwd
- **Les descendants se chargent paresseusement** — uniquement quand des fichiers de ce repertoire sont accedes
- **Les freres et soeurs ne se chargent jamais** — travailler dans `frontend/` ne charge pas `backend/CLAUDE.md`
- **Le global se charge toujours** — `~/.claude/CLAUDE.md` s'applique a chaque session

---

## Strategies pour les monorepos

### CLAUDE.md racine — Conventions partagees

Gardez le fichier racine concentre sur les standards communs a toute l'equipe :

```markdown
# CLAUDE.md (root)

## Coding Standards
- TypeScript strict mode everywhere
- No `any` types without justification comments
- Prefer functional components with hooks

## Git Conventions
- Conventional commits: feat/fix/chore/docs/refactor
- Squash merge to main

## Testing
- Every PR must include tests
- Minimum 80% coverage for new code
```

### CLAUDE.md de composant — Details specifiques

Chaque composant recoit ses propres instructions ciblees :

```markdown
# frontend/CLAUDE.md

## Stack
- Next.js 15 App Router
- Tailwind CSS + shadcn/ui
- React Query for server state

## Patterns
- Server Components by default
- Client Components only for interactivity
- Use app/api/ for API routes
```

### CLAUDE.local.md — Preferences personnelles

Surcharges personnelles qui ne doivent jamais etre commitees :

```markdown
# CLAUDE.local.md (git-ignored)

- Use verbose comments — I am learning this codebase
- Run prettier after every file edit
- Always explain your reasoning before making changes
```

---

## Systeme de memoire automatique

La memoire automatique est le systeme de Claude Code pour retenir des informations d'une conversation a l'autre. Elle stocke des observations et apprentissages qui persistent entre les sessions.

### Types

| Type | Stocke dans | Description |
|------|-------------|-------------|
| `user` | `~/.claude/memory/` | Preferences utilisateur et patterns globaux — s'applique a tous les projets |
| `feedback` | `~/.claude/memory/` | Corrections et ajustements comportementaux issus des retours utilisateur |
| `project` | `~/.claude/projects/<hash>/memory/` | Apprentissages et contexte specifiques au projet |
| `reference` | `~/.claude/projects/<hash>/memory/` | Materiaux de reference que Claude a trouves utiles pour ce projet |

### Fonctionnement

1. Claude observe un pattern, une preference ou une correction au cours d'une session
2. Si l'observation est susceptible d'etre utile dans les sessions futures, Claude la stocke en memoire automatique
3. Au demarrage de la session suivante, les entrees de memoire automatique pertinentes sont chargees dans le contexte
4. Les utilisateurs peuvent consulter et gerer les entrees via `/memory`

### Quand utiliser la memoire automatique vs CLAUDE.md

| Utilisez la memoire automatique quand | Utilisez CLAUDE.md quand |
|----------------------------------------|--------------------------|
| L'instruction a emerge d'un retour pendant une session | L'instruction est planifiee a l'avance |
| C'est une preference personnelle (ton, verbosite, workflow) | C'est une convention d'equipe (standards de code, architecture) |
| Elle evolue au fil de l'utilisation | Elle est stable et revisee en code review |
| Elle s'applique a l'utilisateur a travers les projets | Elle s'applique au projet quel que soit l'utilisateur |

---

## Memoire d'agent

Les agents avec le champ `memory:` dans leur frontmatter maintiennent une memoire persistante qui survit entre les sessions. Cela permet aux agents d'apprendre et de s'ameliorer au fil du temps.

### Portees

| Portee | Stockee dans | Partagee | Description |
|--------|-------------|----------|-------------|
| `user` | `~/.claude/agent-memory/<agent>/` | Non | Memoire inter-projets pour cet agent — suit l'utilisateur |
| `project` | `.claude/agent-memory/<agent>/` | Oui | Memoire specifique au projet — partagee avec l'equipe via git |
| `local` | `.claude/agent-memory/<agent>/` (ignore par git) | Non | Memoire specifique au projet, personnelle |

### Configuration

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes"
memory: project
---
Review code changes. After each review, note patterns you see frequently
so you can flag them earlier in future reviews.
```

### Cas d'utilisation

- **Revue de code** avec `memory: project` — apprend les patterns specifiques a l'equipe, les bugs courants, les preferences de style
- **Assistant de recherche** avec `memory: user` — retient les sujets explores, les sources preferees, le style d'ecriture
- **Agent de deploiement** avec `memory: project` — suit l'historique des deploiements, les particularites connues de l'environnement

---

## Rules (.claude/rules/)

Les Rules sont des fichiers markdown a portee glob qui se chargent automatiquement lorsque Claude accede a des fichiers correspondant au pattern glob.

### Emplacement

| Chemin | Portee |
|--------|--------|
| `.claude/rules/*.md` | Rules au niveau du projet (suivies par git, partagees en equipe) |
| `~/.claude/rules/*.md` | Rules globales (personnelles, s'appliquent a tous les projets) |

### Format

Chaque fichier de rule commence par une ligne `# Glob:` qui definit quand la rule s'active :

```markdown
# Glob: **/*.tsx

## React Component Rules

- Always use functional components
- Export components as named exports, not default
- Co-locate styles in a .module.css file
```

```markdown
# Glob: **/*.test.*

## Testing Rules

- Use describe/it blocks, not test()
- Mock external dependencies, not internal modules
- Assert on behavior, not implementation
```

### Quand utiliser les Rules vs CLAUDE.md

| Utilisez les Rules quand | Utilisez CLAUDE.md quand |
|--------------------------|--------------------------|
| Les instructions s'appliquent a des types de fichiers specifiques | Les instructions s'appliquent a tout le projet |
| Vous souhaitez un chargement paresseux base sur l'acces aux fichiers | Vous souhaitez un chargement eagre au demarrage de la session |
| Les directives sont ciblees sur un pattern (`*.py`, `*.tsx`, `migrations/*.sql`) | Les directives sont universelles (conventions git, decisions d'architecture) |

---

## Tableau comparatif

| Fonctionnalite | CLAUDE.md | Memoire automatique | Memoire d'agent | Rules |
|----------------|-----------|---------------------|-----------------|-------|
| **Cree par** | Developpeur (manuellement) | Claude (automatiquement) | Agent (automatiquement) | Developpeur (manuellement) |
| **Partage avec l'equipe** | Oui (sauf `.local.md`) | Non | Oui (si portee `project`) | Oui |
| **Charge au demarrage** | Oui (ancetres) | Oui | Oui (demarrage agent) | Non (paresseux) |
| **Cible par type de fichier** | Non | Non | Non | Oui (patterns glob) |
| **Modifiable par l'utilisateur** | Oui (edition du fichier) | Oui (`/memory`) | Oui (edition du fichier) | Oui (edition du fichier) |
| **Persiste entre les sessions** | Oui | Oui | Oui | Oui |
| **Ideal pour** | Conventions de projet | Preferences apprises | Amelioration des agents | Directives specifiques aux types de fichiers |

---

## Bonnes pratiques

### Garder CLAUDE.md sous 200 lignes

Les fichiers CLAUDE.md trop longs provoquent une surcharge du contexte et Claude peut ignorer les instructions en fin de fichier. Scindez les gros fichiers en fichiers CLAUDE.md au niveau des composants et en rules.

### Utiliser les Rules pour les directives par type de fichier

Ne mettez pas « quand vous editez des fichiers Python, faites X » dans CLAUDE.md. Creez plutot `.claude/rules/python.md` avec `# Glob: **/*.py`. Cela ne se charge que quand c'est necessaire.

### Utiliser la memoire d'agent pour les agents evolutifs

Si un agent doit devenir plus intelligent au fil du temps, donnez-lui une portee `memory:`. L'agent stocke des observations qui ameliorent les executions futures — patterns de revue, odeurs de code, particularites de deploiement.

### Compacter a 50 % d'utilisation du contexte

Executez `/context` periodiquement pour verifier l'utilisation. Quand elle atteint 50 %, executez `/compact` avec des instructions de focus optionnelles : `/compact focus on the authentication refactor`. Cela preserve le contexte important tout en liberant de l'espace.

### Ignorer les fichiers personnels dans Git

Ajoutez toujours a `.gitignore` :

```
CLAUDE.local.md
.claude/settings.local.json
```

### Separer les instructions stables des instructions evolutives

- **Stables** (standards de code, architecture) → CLAUDE.md et rules
- **Evolutives** (preferences, patterns, apprentissages) → memoire automatique et memoire d'agent

---

## Obsolescence et nettoyage

### Signes de memoire obsolete

- CLAUDE.md reference des APIs depreciees ou des fonctionnalites supprimees
- Les entrees de memoire automatique contredisent les instructions actuelles de CLAUDE.md
- La memoire d'agent contient des patterns specifiques au projet qui ne sont plus d'actualite
- Les Rules referencent des patterns de fichiers qui n'existent plus dans le codebase

### Strategies de nettoyage

1. **Auditez CLAUDE.md trimestriellement** — Lisez chaque instruction et verifiez qu'elle s'applique encore
2. **Revisez la memoire automatique** — Executez `/memory` et supprimez les entrees qui ne sont plus pertinentes
3. **Elaguez la memoire d'agent** — Verifiez `.claude/agent-memory/` pour les observations obsoletes
4. **Mettez a jour les rules** — Verifiez que les patterns glob correspondent encore aux fichiers du projet
5. **Suivez avec des commentaires** — Ajoutez `<!-- Last reviewed: 2026-04-09 -->` aux sections de CLAUDE.md

---

## Sources

- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code Rules — Docs](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules)
- [Claude Code Sub-agents (Memory) — Docs](https://code.claude.com/docs/en/sub-agents)
- [CLAUDE.md in Monorepos](../../../best-practice/claude-memory.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
