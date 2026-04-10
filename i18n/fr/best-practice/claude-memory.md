🌍 [English](../../../best-practice/claude-memory.md) | [中文](../../zh/best-practice/claude-memory.md) | [日本語](../../ja/best-practice/claude-memory.md) | **Français** | [Русский](../../ru/best-practice/claude-memory.md) | [العربية](../../ar/best-practice/claude-memory.md)

# Memoire Claude

Contexte persistant via les fichiers CLAUDE.md — comment les rediger et comment ils se chargent dans les monorepos.

<table width="100%">
<tr>
<td><a href="../README.md">← Retour aux bonnes pratiques Claude Code</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1. Rediger un bon CLAUDE.md

Un CLAUDE.md bien structure est le moyen le plus impactant d'ameliorer la sortie de Claude Code pour votre projet. Humanlayer a un excellent guide couvrant ce qu'il faut inclure, comment le structurer et les pieges courants.

- [Humanlayer - Rediger un bon Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## 2. CLAUDE.md dans les grands monorepos

Lorsque vous travaillez avec Claude Code dans un monorepo, comprendre comment les fichiers CLAUDE.md sont charges en contexte est crucial pour organiser efficacement les instructions de votre projet.

<p align="center">
  <a href="https://x.com/bcherny/status/2016339448863355206"><img src="../../../best-practice/assets/claude-memory/claude-memory-monorepo.jpg" alt="Chargement CLAUDE.md dans les monorepos" width="600"></a>
</p>

### Les deux mecanismes de chargement

Claude Code utilise deux mecanismes distincts pour charger les fichiers CLAUDE.md :

#### Chargement ascendant (VERS LE HAUT de l'arborescence)

Quand vous demarrez Claude Code, il remonte **vers le haut** depuis votre repertoire de travail actuel jusqu'a la racine du systeme de fichiers et charge chaque CLAUDE.md qu'il trouve en chemin. Ces fichiers sont charges **immediatement au demarrage**.

#### Chargement descendant (VERS LE BAS de l'arborescence)

Les fichiers CLAUDE.md dans les sous-repertoires en dessous de votre repertoire de travail actuel ne sont **PAS charges au lancement**. Ils ne sont inclus que lorsque Claude lit des fichiers dans ces sous-repertoires pendant votre session. C'est ce qu'on appelle le **chargement paresseux (lazy loading)**.

### Exemple de structure monorepo

Considerez un monorepo typique avec des repertoires separes pour differents composants :

```
/mymonorepo/
├── CLAUDE.md          # Instructions de niveau racine (partagees entre tous les composants)
├── frontend/
│   └── CLAUDE.md      # Instructions specifiques au frontend
├── backend/
│   └── CLAUDE.md      # Instructions specifiques au backend
└── api/
    └── CLAUDE.md      # Instructions specifiques a l'API
```

### Scenario 1 : Executer Claude Code depuis le repertoire racine

Quand vous executez Claude Code depuis `/mymonorepo/` :

```bash
cd /mymonorepo
claude
```

| Fichier | Charge au lancement ? | Raison |
|------|-------------------|--------|
| `/mymonorepo/CLAUDE.md` | Oui | C'est votre repertoire de travail actuel |
| `/mymonorepo/frontend/CLAUDE.md` | Non | Charge uniquement quand vous lisez/editez des fichiers dans `frontend/` |
| `/mymonorepo/backend/CLAUDE.md` | Non | Charge uniquement quand vous lisez/editez des fichiers dans `backend/` |
| `/mymonorepo/api/CLAUDE.md` | Non | Charge uniquement quand vous lisez/editez des fichiers dans `api/` |

### Scenario 2 : Executer Claude Code depuis un repertoire de composant

Quand vous executez Claude Code depuis `/mymonorepo/frontend/` :

```bash
cd /mymonorepo/frontend
claude
```

| Fichier | Charge au lancement ? | Raison |
|------|-------------------|--------|
| `/mymonorepo/CLAUDE.md` | Oui | C'est un repertoire ancetre |
| `/mymonorepo/frontend/CLAUDE.md` | Oui | C'est votre repertoire de travail actuel |
| `/mymonorepo/backend/CLAUDE.md` | Non | Branche differente de l'arborescence |
| `/mymonorepo/api/CLAUDE.md` | Non | Branche differente de l'arborescence |

### Points cles

1. **Les ancetres se chargent toujours au demarrage** — Claude remonte l'arborescence et charge tous les fichiers CLAUDE.md qu'il trouve. Cela garantit que vous avez toujours acces aux instructions de niveau racine, valables pour tout le depot.

2. **Les descendants se chargent paresseusement** — Les fichiers CLAUDE.md des sous-repertoires ne se chargent que lorsque vous interagissez avec des fichiers dans ces sous-repertoires. Cela evite que du contexte non pertinent n'encombre votre session.

3. **Les freres ne se chargent jamais** — Si vous travaillez dans `frontend/`, vous n'obtiendrez pas `backend/CLAUDE.md` ni `api/CLAUDE.md` en contexte.

4. **CLAUDE.md global** — Vous pouvez aussi placer un CLAUDE.md dans `~/.claude/CLAUDE.md` dans votre dossier personnel, qui s'applique a TOUTES les sessions Claude Code quel que soit le projet.

### Pourquoi cette conception fonctionne pour les monorepos

- **Les instructions partagees se propagent vers le bas** — Le CLAUDE.md de niveau racine contient les conventions valables pour tout le depot, les standards de code et les patterns communs qui s'appliquent partout.

- **Les instructions specifiques aux composants restent isolees** — Les developpeurs frontend n'ont pas besoin des instructions specifiques au backend encombrant leur contexte, et vice versa.

- **Le contexte est optimise** — En chargeant paresseusement les fichiers CLAUDE.md descendants, Claude Code evite de charger potentiellement des centaines de kilo-octets d'instructions non pertinentes au demarrage.

### Bonnes pratiques

1. **Mettez les conventions partagees dans le CLAUDE.md racine** — Standards de code, formats de messages de commit, modeles de PR et autres directives valables pour tout le depot.

2. **Mettez les instructions specifiques au composant dans le CLAUDE.md du composant** — Patterns specifiques au framework, architecture des composants, conventions de test propres a ce composant.

3. **Utilisez CLAUDE.local.md pour les preferences personnelles** — Ajoutez-le au `.gitignore` pour les instructions qui ne devraient pas etre partagees avec l'equipe.

---

## Sources

- [Documentation Claude Code - Comment Claude recherche les memoires](https://code.claude.com/docs/en/memory#how-claude-looks-up-memories)
- [Boris Cherny sur X - Clarification sur le chargement de CLAUDE.md](https://x.com/bcherny/status/2016339448863355206)
- [Humanlayer - Rediger un bon Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
