🌍 [English](../../README.md) | [中文](../zh/README.md) | [日本語](../ja/README.md) | **Français** | [Русский](../ru/README.md) | [العربية](../ar/README.md)

# claude-code-best-practice
du vibe coding a l'ingenierie agentique - la pratique rend Claude parfait

![updated with Claude Code](https://img.shields.io/badge/updated_with_Claude_Code-v2.1.96%20(Apr%2008%2C%202026%209%3A38%20PM%20PKT)-white?style=flat&labelColor=555)<br>

[![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/) [![Implemented](../../!/tags/implemented.svg)](../../implementation/) [![Orchestration Workflow](../../!/tags/orchestration-workflow.svg)](../../orchestration-workflow/orchestration-workflow.md) [![Claude](../../!/tags/claude.svg)](https://code.claude.com/docs) [![Boris](../../!/tags/boris-cherny.svg)](#-astuces-et-conseils) [![Community](../../!/tags/community.svg)](#-sabonner) ![Click on these badges below to see the actual sources](../../!/tags/click-badges.svg)<br>
<img src="../../!/tags/a.svg" height="14"> = Agents · <img src="../../!/tags/c.svg" height="14"> = Commands · <img src="../../!/tags/s.svg" height="14"> = Skills

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="Mascotte Claude Code en train de sauter" width="120" height="100"><br>

</p>


## 🧠 CONCEPTS

| Fonctionnalite | Emplacement | Description |
|---------|----------|-------------|
| <img src="../../!/tags/a.svg" height="14"> [**Subagents**](https://code.claude.com/docs/en/sub-agents) | `.claude/agents/<name>.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-subagents.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-subagents-implementation.md) Acteur autonome dans un contexte isole — outils personnalises, permissions, modele, memoire et identite persistante |
| <img src="../../!/tags/c.svg" height="14"> [**Commands**](https://code.claude.com/docs/en/slash-commands) | `.claude/commands/<name>.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-commands.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-commands-implementation.md) Connaissances injectees dans le contexte existant — modeles de prompts simples invoques par l'utilisateur pour l'orchestration des workflows |
| <img src="../../!/tags/s.svg" height="14"> [**Skills**](https://code.claude.com/docs/en/skills) | `.claude/skills/<name>/SKILL.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-skills.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-skills-implementation.md) Connaissances injectees dans le contexte existant — configurables, prechargeable, auto-decouvrables, avec bifurcation de contexte et divulgation progressive · [Skills officiels](https://github.com/anthropics/skills/tree/main/skills) |
| [**Workflows**](https://code.claude.com/docs/en/common-workflows) | [`.claude/commands/weather-orchestrator.md`](../../.claude/commands/weather-orchestrator.md) | [![Orchestration Workflow](../../!/tags/orchestration-workflow.svg)](../../orchestration-workflow/orchestration-workflow.md) |
| [**Hooks**](https://code.claude.com/docs/en/hooks) | `.claude/hooks/` | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-hooks) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/claude-code-hooks) Gestionnaires definis par l'utilisateur (scripts, HTTP, prompts, agents) qui s'executent en dehors de la boucle agentique sur des evenements specifiques · [Guide](https://code.claude.com/docs/en/hooks-guide) |
| [**MCP Servers**](https://code.claude.com/docs/en/mcp) | `.claude/settings.json`, `.mcp.json` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-mcp.md) [![Implemented](../../!/tags/implemented.svg)](../../.mcp.json) Connexions Model Context Protocol vers des outils externes, bases de donnees et API |
| [**Plugins**](https://code.claude.com/docs/en/plugins) | paquets distribuables | Ensembles de skills, subagents, hooks, MCP servers et LSP servers · [Marketplaces](https://code.claude.com/docs/en/discover-plugins) · [Creer des Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) |
| [**Settings**](https://code.claude.com/docs/en/settings) | `.claude/settings.json` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-settings.md) [![Implemented](../../!/tags/implemented.svg)](../../.claude/settings.json) Systeme de configuration hierarchique · [Permissions](https://code.claude.com/docs/en/permissions) · [Model Config](https://code.claude.com/docs/en/model-config) · [Output Styles](https://code.claude.com/docs/en/output-styles) · [Sandboxing](https://code.claude.com/docs/en/sandboxing) · [Keybindings](https://code.claude.com/docs/en/keybindings) · [Fast Mode](https://code.claude.com/docs/en/fast-mode) |
| [**Status Line**](https://code.claude.com/docs/en/statusline) | `.claude/settings.json` | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-status-line) [![Implemented](../../!/tags/implemented.svg)](../../.claude/settings.json) Barre d'etat personnalisable affichant l'utilisation du contexte, le modele, le cout et les informations de session |
| [**Memory**](https://code.claude.com/docs/en/memory) | `CLAUDE.md`, `.claude/rules/`, `~/.claude/rules/`, `~/.claude/projects/<project>/memory/` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-memory.md) [![Implemented](../../!/tags/implemented.svg)](../../CLAUDE.md) Contexte persistant via les fichiers CLAUDE.md et les imports `@path` · [Auto Memory](https://code.claude.com/docs/en/memory) · [Rules](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) |
| [**Checkpointing**](https://code.claude.com/docs/en/checkpointing) | automatique (base sur git) | Suivi automatique des modifications de fichiers avec retour en arriere (`Esc Esc` ou `/rewind`) et resume cible |
| [**CLI Startup Flags**](https://code.claude.com/docs/en/cli-reference) | `claude [flags]` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-cli-startup-flags.md) Drapeaux de ligne de commande, sous-commandes et variables d'environnement pour lancer Claude Code · [Interactive Mode](https://code.claude.com/docs/en/interactive-mode) |
| **AI Terms** | | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-codex-cursor-gemini/blob/main/reports/ai-terms.md) Ingenierie agentique · Ingenierie du contexte · Vibe Coding |
| [**Best Practices**](https://code.claude.com/docs/en/best-practices) | | Bonnes pratiques officielles · [Prompt Engineering](https://github.com/anthropics/prompt-eng-interactive-tutorial) · [Etendre Claude Code](https://code.claude.com/docs/en/features-overview) |

### 🔥 Nouveautes

| Fonctionnalite | Emplacement | Description |
|---------|----------|-------------|
| [**Power-ups**](../../best-practice/claude-power-ups.md) | `/powerup` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-power-ups.md) Lecons interactives enseignant les fonctionnalites de Claude Code avec des demos animees (v2.1.90) |
| [**Ultraplan**](https://code.claude.com/docs/en/ultraplan) ![beta](../../!/tags/beta.svg) | `/ultraplan` | Elaborez des plans dans le cloud avec revision en navigateur, commentaires en ligne et execution flexible — a distance ou teleporte vers le terminal |
| [**Claude Code Web**](https://code.claude.com/docs/en/claude-code-on-the-web) ![beta](../../!/tags/beta.svg) | `claude.ai/code` | Executez des taches sur l'infrastructure cloud — taches de longue duree, correction automatique de PR, sessions paralleles sans configuration locale · [Taches planifiees](https://code.claude.com/docs/en/web-scheduled-tasks) |
| [**No Flicker Mode**](https://code.claude.com/docs/en/fullscreen) ![beta](../../!/tags/beta.svg) | `CLAUDE_CODE_NO_FLICKER=1` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2039421575422980329) Rendu sans scintillement sur ecran alternatif avec support souris, memoire stable et defilement integre — apercu de recherche optionnel |
| [**Computer Use**](https://code.claude.com/docs/en/computer-use) ![beta](../../!/tags/beta.svg) | `computer-use` MCP server | Laissez Claude controler votre ecran — ouvrir des apps, cliquer, taper et capturer votre ecran sur macOS · [Desktop](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer) |
| [**Auto Mode**](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) ![beta](../../!/tags/beta.svg) | `claude --enable-auto-mode` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/claudeai/status/2036503582166393240) Un classificateur de securite en arriere-plan remplace les demandes de permission manuelles — Claude decide ce qui est sur tout en bloquant les injections de prompts et les escalades risquees · Lancez avec `claude --enable-auto-mode` (ou `--permission-mode auto`), ou basculez avec `Shift+Tab` pendant une session · [Blog](https://claude.com/blog/auto-mode) |
| [**Channels**](https://code.claude.com/docs/en/channels) ![beta](../../!/tags/beta.svg) | `--channels`, base sur plugin | Envoyez des evenements depuis Telegram, Discord ou des webhooks vers une session en cours — Claude reagit pendant votre absence · [Reference](https://code.claude.com/docs/en/channels-reference) |
| [**Slack**](https://code.claude.com/docs/en/slack) | `@Claude` dans Slack | Mentionnez @Claude dans le chat d'equipe avec une tache de code — redirige vers des sessions Claude Code web pour corrections de bugs, revues de code et execution de taches en parallele |
| [**Code Review**](https://code.claude.com/docs/en/code-review) ![beta](../../!/tags/beta.svg) | GitHub App (geree) | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/claudeai/status/2031088171262554195) Analyse multi-agent de PR qui detecte les bugs, vulnerabilites de securite et regressions · [Blog](https://claude.com/blog/code-review) |
| [**GitHub Actions**](https://code.claude.com/docs/en/github-actions) | `.github/workflows/` | Automatisez les revues de PR, le triage des issues et la generation de code dans les pipelines CI/CD · [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd) |
| [**Chrome**](https://code.claude.com/docs/en/chrome) ![beta](../../!/tags/beta.svg) | `--chrome`, extension | [![Best Practice](../../!/tags/best-practice.svg)](../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) Automatisation du navigateur via Claude in Chrome — testez des apps web, debuguez avec la console, automatisez des formulaires, extrayez des donnees de pages |
| [**Scheduled Tasks**](https://code.claude.com/docs/en/scheduled-tasks) | `/loop`, `/schedule`, outils cron | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2030193932404150413) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-scheduled-tasks-implementation.md) `/loop` execute des prompts localement sur un planning recurrent (jusqu'a 3 jours) · [`/schedule`](https://code.claude.com/docs/en/web-scheduled-tasks) execute des prompts dans le cloud sur l'infrastructure Anthropic — fonctionne meme quand votre machine est eteinte · [Annonce](https://x.com/noahzweben/status/2036129220959805859) |
| [**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation) ![beta](../../!/tags/beta.svg) | `/voice` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/trq212/status/2028628570692890800) Saisie vocale appuyer-pour-parler avec support de 20 langues et touche d'activation reconfigurable |
| [**Simplify & Batch**](https://code.claude.com/docs/en/skills#bundled-skills) | `/simplify`, `/batch` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2027534984534544489) Skills integres pour la qualite du code et les operations en masse — simplify refactorise pour la reutilisation et l'efficacite, batch execute des commandes sur plusieurs fichiers |
| [**Agent Teams**](https://code.claude.com/docs/en/agent-teams) ![beta](../../!/tags/beta.svg) | integre (var env) | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2019472394696683904) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-agent-teams-implementation.md) Plusieurs agents travaillant en parallele sur le meme codebase avec coordination des taches partagee |
| [**Remote Control**](https://code.claude.com/docs/en/remote-control) | `/remote-control`, `/rc` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/noahzweben/status/2032533699116355819) Continuez les sessions locales depuis n'importe quel appareil — telephone, tablette ou navigateur · [Headless Mode](https://code.claude.com/docs/en/headless) |
| [**Git Worktrees**](https://code.claude.com/docs/en/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees) | integre | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2025007393290272904) Branches git isolees pour le developpement parallele — chaque agent obtient sa propre copie de travail |
| [**Ralph Wiggum Loop**](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) | plugin | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/ghuntley/how-to-ralph-wiggum) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26) Boucle de developpement autonome pour les taches de longue duree — itere jusqu'a achevement |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

<a id="orchestration-workflow"></a>

## <a href="../../orchestration-workflow/orchestration-workflow.md"><img src="../../!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

Voir [orchestration-workflow](../../orchestration-workflow/orchestration-workflow.md) pour les details d'implementation du patron <img src="../../!/tags/c.svg" height="14"> **Command** → <img src="../../!/tags/a.svg" height="14"> **Agent** → <img src="../../!/tags/s.svg" height="14"> **Skill**.


<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.svg" alt="Flux d'architecture Command Skill Agent" width="100%">
</p>

<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.gif" alt="Demo du Workflow d'orchestration" width="600">
</p>

![How to Use](../../!/tags/how-to-use.svg)

```bash
claude
/weather-orchestrator
```

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

## ⚙️ WORKFLOWS DE DEVELOPPEMENT

Tous les workflows majeurs convergent vers le meme patron architectural : **Recherche → Plan → Execution → Revue → Livraison**

| Nom | ★ | Specificite | Plan | <img src="../../!/tags/a.svg" height="14"> | <img src="../../!/tags/c.svg" height="14"> | <img src="../../!/tags/s.svg" height="14"> |
|------|---|------------|------|---|---|---|
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | 146k | ![instinct scoring](https://img.shields.io/badge/instinct_scoring-ddf4ff) ![AgentShield](https://img.shields.io/badge/AgentShield-ddf4ff) ![multi-lang rules](https://img.shields.io/badge/multi--lang_rules-ddf4ff) | <img src="../../!/tags/a.svg" height="14"> [planner](https://github.com/affaan-m/everything-claude-code/blob/main/agents/planner.md) | 47 | 82 | 182 |
| [Superpowers](https://github.com/obra/superpowers) | 141k | ![TDD-first](https://img.shields.io/badge/TDD--first-ddf4ff) ![Iron Laws](https://img.shields.io/badge/Iron_Laws-ddf4ff) ![whole-plan review](https://img.shields.io/badge/whole--plan_review-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [writing-plans](https://github.com/obra/superpowers/tree/main/skills/writing-plans) | 5 | 3 | 14 |
| [Spec Kit](https://github.com/github/spec-kit) | 86k | ![spec-driven](https://img.shields.io/badge/spec--driven-ddf4ff) ![constitution](https://img.shields.io/badge/constitution-ddf4ff) ![22+ tools](https://img.shields.io/badge/22%2B_tools-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [speckit.plan](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md) | 0 | 9+ | 0 |
| [gstack](https://github.com/garrytan/gstack) | 67k | ![role personas](https://img.shields.io/badge/role_personas-ddf4ff) ![/codex review](https://img.shields.io/badge/%2Fcodex_review-ddf4ff) ![parallel sprints](https://img.shields.io/badge/parallel_sprints-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [autoplan](https://github.com/garrytan/gstack/tree/main/skills/autoplan) | 0 | 0 | 37 |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) | 49k | ![fresh 200K contexts](https://img.shields.io/badge/fresh_200K_contexts-ddf4ff) ![wave execution](https://img.shields.io/badge/wave_execution-ddf4ff) ![XML plans](https://img.shields.io/badge/XML_plans-ddf4ff) | <img src="../../!/tags/a.svg" height="14"> [gsd-planner](https://github.com/gsd-build/get-shit-done/blob/main/agents/gsd-planner.md) | 24 | 68 | 0 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 44k | ![full SDLC](https://img.shields.io/badge/full_SDLC-ddf4ff) ![agent personas](https://img.shields.io/badge/agent_personas-ddf4ff) ![22+ platforms](https://img.shields.io/badge/22%2B_platforms-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [bmad-create-prd](https://github.com/bmad-code-org/BMAD-METHOD/tree/main/src/bmm-skills/2-plan-workflows/bmad-create-prd) | 0 | 0 | 39 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | 38k | ![delta specs](https://img.shields.io/badge/delta_specs-ddf4ff) ![brownfield](https://img.shields.io/badge/brownfield-ddf4ff) ![artifact DAG](https://img.shields.io/badge/artifact_DAG-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [opsx:propose](https://github.com/Fission-AI/OpenSpec/blob/main/src/commands/workflow/new-change.ts) | 0 | 11 | 0 |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 26k | ![teams orchestration](https://img.shields.io/badge/teams_orchestration-ddf4ff) ![tmux workers](https://img.shields.io/badge/tmux_workers-ddf4ff) ![skill auto-inject](https://img.shields.io/badge/skill_auto--inject-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [ralplan](https://github.com/Yeachan-Heo/oh-my-claudecode/tree/main/skills/ralplan) | 19 | 0 | 37 |
| [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | 14k | ![Compound Learning](https://img.shields.io/badge/Compound_Learning-ddf4ff) ![Multi-Platform CLI](https://img.shields.io/badge/Multi--Platform_CLI-ddf4ff) ![Plugin Marketplace](https://img.shields.io/badge/Plugin_Marketplace-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [ce-plan](https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills/ce-plan) | 51 | 4 | 44 |
| [HumanLayer](https://github.com/humanlayer/humanlayer) | 10k | ![RPI](https://img.shields.io/badge/RPI-ddf4ff) ![context engineering](https://img.shields.io/badge/context_engineering-ddf4ff) ![300k+ LOC](https://img.shields.io/badge/300k%2B_LOC-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [create_plan](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/create_plan.md) | 6 | 27 | 0 |

### Autres
- [Workflow cross-modele (Claude Code + Codex)](../../development-workflows/cross-model-workflow/cross-model-workflow.md) [![Implemented](../../!/tags/implemented.svg)](../../development-workflows/cross-model-workflow/cross-model-workflow.md)
- [RPI](../../development-workflows/rpi/rpi-workflow.md) [![Implemented](../../!/tags/implemented.svg)](../../development-workflows/rpi/rpi-workflow.md)
- [Boucle Ralph Wiggum](https://www.youtube.com/watch?v=eAtvoGlpeRU) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26)
- [Workflow Andrej Karpathy (membre fondateur, OpenAI)](https://x.com/karpathy/status/2015883857489522876)
- [Workflow Peter Steinberger (createur d'OpenClaw)](https://youtu.be/8lF7HmQ_RgY?t=2582)
- Workflow Boris Cherny (createur de Claude Code) — [13 astuces](../../tips/claude-boris-13-tips-03-jan-26.md) · [10 astuces](../../tips/claude-boris-10-tips-01-feb-26.md) · [12 astuces](../../tips/claude-boris-12-tips-12-feb-26.md) · [2 astuces](../../tips/claude-boris-2-tips-25-mar-26.md) · [15 astuces](../../tips/claude-boris-15-tips-30-mar-26.md) [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny)

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

## 💡 ASTUCES ET CONSEILS (69)

🚫👶 = ne pas micro-gerer

[Prompting](#tips-prompting) · [Planification](#tips-planning) · [CLAUDE.md](#tips-claudemd) · [Agents](#tips-agents) · [Commands](#tips-commands) · [Skills](#tips-skills) · [Hooks](#tips-hooks) · [Workflows](#tips-workflows) · [Avance](#tips-workflows-advanced) · [Git / PR](#tips-git-pr) · [Debugging](#tips-debugging) · [Utilitaires](#tips-utilities) · [Quotidien](#tips-daily)

![Community](../../!/tags/community.svg)

<a id="tips-prompting"></a>■ **Prompting (3)**

| Astuce | Source |
|-----|--------|
| mettez Claude au defi — "passe-moi un oral sur ces changements et ne fais pas de PR tant que je n'ai pas reussi ton test." ou "prouve-moi que ca marche" et faites comparer a Claude entre main et votre branche 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| apres une correction mediocre — "sachant tout ce que tu sais maintenant, recommence de zero et implemente la solution elegante" 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| Claude corrige la plupart des bugs tout seul — collez le bug, dites "corrige", ne micro-gerez pas la methode 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742750473720121) |

<a id="tips-planning"></a>■ **Planification/Specs (6)**

| Astuce | Source |
|-----|--------|
| commencez toujours avec le [plan mode](https://code.claude.com/docs/en/common-workflows) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179845336527000) |
| commencez avec une spec ou un prompt minimal et demandez a Claude de vous interviewer avec l'outil [AskUserQuestion](https://code.claude.com/docs/en/cli-reference), puis creez une nouvelle session pour executer la spec | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2005315275026260309) |
| faites toujours un plan par phases avec validation, chaque phase ayant plusieurs tests (unitaires, automatisation, integration) | |
| lancez un second Claude pour reviser votre plan en tant qu'ingenieur senior, ou utilisez le [cross-model](../../development-workflows/cross-model-workflow/cross-model-workflow.md) pour la revue | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742745365057733) |
| redigez des specs detaillees et reduisez l'ambiguite avant de deleguer le travail — plus vous etes specifique, meilleur est le resultat | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| prototype > PRD — construisez 20-30 versions au lieu d'ecrire des specs, le cout de construction est faible alors tentez plusieurs essais | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3630) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3630) |

<a id="tips-claudemd"></a>■ **CLAUDE.md (7)**

| Astuce | Source |
|-----|--------|
| [CLAUDE.md](https://code.claude.com/docs/en/memory) devrait viser moins de [200 lignes](https://code.claude.com/docs/en/memory#write-effective-instructions) par fichier. [60 lignes chez humanlayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md) ([toujours pas garanti a 100%](https://www.reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/)) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179840848597422) [![Dex](../../!/tags/community-dex.svg)](https://www.humanlayer.dev/blog/writing-a-good-claude-md) |
| enveloppez les regles CLAUDE.md specifiques au domaine dans des [balises \<important if="..."\>](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md) pour empecher Claude de les ignorer lorsque les fichiers deviennent longs | [![Dex](../../!/tags/community-dex.svg)](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md) |
| utilisez [plusieurs CLAUDE.md](../../best-practice/claude-memory.md) pour les monorepos — chargement ascendant + descendant | |
| utilisez [.claude/rules/](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) pour decouper les instructions volumineuses | |
| [memory.md](https://code.claude.com/docs/en/memory), constitution.md ne garantit rien | |
| tout developpeur devrait pouvoir lancer Claude, dire "lance les tests" et que ca marche du premier coup — si ce n'est pas le cas, votre CLAUDE.md manque les commandes essentielles de setup/build/test | [![Dex](../../!/tags/community-dex.svg)](https://x.com/dexhorthy/status/2034713765401551053) |
| gardez les codebases propres et finissez les migrations — des frameworks partiellement migres perturbent les modeles qui risquent de choisir le mauvais patron | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=1112) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=1112) |
| utilisez [settings.json](../../best-practice/claude-settings.md) pour les comportements imposes par le harness (attribution, permissions, modele) — ne mettez pas "NE JAMAIS ajouter Co-Authored-By" dans CLAUDE.md quand `attribution.commit: ""` est deterministe | [![davila7](../../!/tags/community-davila7.svg)](https://x.com/dani_avila7/status/2036182734310195550) |

<a id="tips-agents"></a><img src="../../!/tags/a.svg" height="14"> **Agents (4)**

| Astuce | Source |
|-----|--------|
| creez des [sub-agents](https://code.claude.com/docs/en/sub-agents) specifiques aux fonctionnalites (contexte supplementaire) avec des [skills](https://code.claude.com/docs/en/skills) (divulgation progressive) plutot que des agents generiques qa, ingenieur backend | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179850139000872) |
| dites "utilise des subagents" pour consacrer plus de puissance de calcul a un probleme — deleguez les taches pour garder votre contexte principal propre et concentre 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| [agent teams avec tmux](https://code.claude.com/docs/en/agent-teams) et [git worktrees](https://x.com/bcherny/status/2025007393290272904) pour le developpement parallele | |
| utilisez le [test time compute](https://code.claude.com/docs/en/sub-agents) — des fenetres de contexte separees donnent de meilleurs resultats ; un agent peut causer des bugs et un autre (meme modele) peut les trouver | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031151689219321886) |

<a id="tips-commands"></a><img src="../../!/tags/c.svg" height="14"> **Commands (3)**

| Astuce | Source |
|-----|--------|
| utilisez des [commands](https://code.claude.com/docs/en/slash-commands) pour vos workflows plutot que des [sub-agents](https://code.claude.com/docs/en/sub-agents) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| utilisez des [slash commands](https://code.claude.com/docs/en/slash-commands) pour chaque workflow de "boucle interne" que vous faites plusieurs fois par jour — evite de repeter les prompts, les commands vivent dans `.claude/commands/` et sont versionnees dans git | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| si vous faites quelque chose plus d'une fois par jour, transformez-le en [skill](https://code.claude.com/docs/en/skills) ou [command](https://code.claude.com/docs/en/slash-commands) — creez des commands `/techdebt`, context-dump ou analytics | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742748984742078) |

<a id="tips-skills"></a><img src="../../!/tags/s.svg" height="14"> **Skills (9)**

| Astuce | Source |
|-----|--------|
| utilisez [context: fork](https://code.claude.com/docs/en/skills) pour executer un skill dans un subagent isole — le contexte principal ne voit que le resultat final, pas les appels d'outils intermediaires. Le champ agent permet de definir le type de subagent | [![Lydia](../../!/tags/lydia.svg)](https://x.com/lydiahallie/status/2033603164398883042) |
| utilisez des [skills dans des sous-dossiers](../../reports/claude-skills-for-larger-mono-repos.md) pour les monorepos | |
| les skills sont des dossiers, pas des fichiers — utilisez les sous-repertoires references/, scripts/, examples/ pour la [divulgation progressive](https://code.claude.com/docs/en/skills) | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| integrez une section Pieges dans chaque skill — contenu a plus forte valeur, ajoutez les points de defaillance de Claude au fil du temps | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| le champ description d'un skill est un declencheur, pas un resume — ecrivez-le pour le modele ("quand dois-je me declencher ?") | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| n'enoncez pas l'evident dans les skills — concentrez-vous sur ce qui pousse Claude hors de son comportement par defaut 🚫👶 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| ne mettez pas Claude sur des rails dans les skills — donnez des objectifs et des contraintes, pas des instructions prescriptives etape par etape 🚫👶 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| incluez des scripts et bibliotheques dans les skills pour que Claude compose plutot que de reconstruire le code repetitif | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| integrez `` !`command` `` dans SKILL.md pour injecter la sortie dynamique du shell dans le prompt — Claude l'execute a l'invocation et le modele ne voit que le resultat | [![Lydia](../../!/tags/lydia.svg)](https://x.com/lydiahallie/status/2034337963820327017) |

<a id="tips-hooks"></a>■ **Hooks (5)**

| Astuce | Source |
|-----|--------|
| utilisez des [hooks a la demande](https://code.claude.com/docs/en/skills) dans les skills — /careful bloque les commandes destructrices, /freeze bloque les modifications en dehors d'un repertoire | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| [mesurez l'utilisation des skills](https://code.claude.com/docs/en/skills) avec un hook PreToolUse pour trouver les skills populaires ou sous-utilises | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| utilisez un [hook PostToolUse](https://code.claude.com/docs/en/hooks) pour auto-formater le code — Claude genere du code bien formate, le hook gere les derniers 10% pour eviter les echecs CI | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179852047335529) |
| routez les [demandes de permission](https://code.claude.com/docs/en/hooks) vers Opus via un hook — laissez-le scanner les attaques et auto-approuver celles qui sont sures 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| utilisez un [hook Stop](https://code.claude.com/docs/en/hooks) pour inciter Claude a continuer ou verifier son travail a la fin d'un tour | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701059253874861) |

<a id="tips-workflows"></a>■ **Workflows (7)**

| Astuce | Source |
|-----|--------|
| evitez la zone morte de l'agent, faites un [/compact](https://code.claude.com/docs/en/interactive-mode) manuel a 50% maximum. Utilisez [/clear](https://code.claude.com/docs/en/cli-reference) pour reinitialiser le contexte en milieu de session si vous changez de tache | |
| Claude Code vanilla est meilleur que n'importe quel workflow pour les petites taches | |
| utilisez [/model](https://code.claude.com/docs/en/model-config) pour selectionner le modele et le raisonnement, [/context](https://code.claude.com/docs/en/interactive-mode) pour voir l'utilisation du contexte, [/usage](https://code.claude.com/docs/en/costs) pour verifier les limites du plan, [/extra-usage](https://code.claude.com/docs/en/interactive-mode) pour configurer la facturation supplementaire, [/config](https://code.claude.com/docs/en/settings) pour configurer les parametres — utilisez Opus pour le plan mode et Sonnet pour le code pour tirer le meilleur des deux | [![Cat](../../!/tags/cat-wu.svg)](https://x.com/_catwu/status/1955694117264261609) |
| activez toujours le [thinking mode](https://code.claude.com/docs/en/model-config) true (pour voir le raisonnement) et le [Output Style](https://code.claude.com/docs/en/output-styles) Explanatory (pour voir la sortie detaillee avec les encadres ★ Insight) dans [/config](https://code.claude.com/docs/en/settings) pour mieux comprendre les decisions de Claude | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179838864666847) |
| utilisez le mot-cle ultrathink dans les prompts pour un [raisonnement a effort eleve](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking#tips-and-best-practices) | |
| [/rename](https://code.claude.com/docs/en/cli-reference) les sessions importantes (ex. [TODO - tache de refactoring]) et [/resume](https://code.claude.com/docs/en/cli-reference) les plus tard — etiquetez chaque instance quand vous executez plusieurs Claudes simultanement | [![Cat](../../!/tags/cat-wu.svg)](https://every.to/podcast/how-to-use-claude-code-like-the-people-who-built-it) |
| utilisez [Esc Esc ou /rewind](https://code.claude.com/docs/en/checkpointing) pour annuler quand Claude devie au lieu d'essayer de corriger dans le meme contexte | |

<a id="tips-workflows-advanced"></a>■ **Workflows avances (6)**

| Astuce | Source |
|-----|--------|
| utilisez beaucoup les diagrammes ASCII pour comprendre votre architecture | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742759218794768) |
| utilisez [/loop](https://code.claude.com/docs/en/scheduled-tasks) pour la surveillance locale recurrente (jusqu'a 3 jours) · utilisez [/schedule](https://code.claude.com/docs/en/web-scheduled-tasks) pour les taches cloud recurrentes qui s'executent meme quand votre machine est eteinte | |
| utilisez le [plugin Ralph Wiggum](https://github.com/shanraisshan/novel-llm-26) pour les taches autonomes de longue duree | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179858435281082) |
| [/permissions](https://code.claude.com/docs/en/permissions) avec la syntaxe wildcard (Bash(npm run *), Edit(/docs/**)) au lieu de dangerously-skip-permissions | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179854077407667) |
| [/sandbox](https://code.claude.com/docs/en/sandboxing) pour reduire les demandes de permission avec l'isolation des fichiers et du reseau — 84% de reduction en interne | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700506465579443) [![Cat](../../!/tags/cat-wu.svg)](https://creatoreconomy.so/p/inside-claude-code-how-an-ai-native-actually-works-cat-wu) |
| investissez dans des skills de [verification de produit](https://code.claude.com/docs/en/skills) (signup-flow-driver, checkout-verifier) — ca vaut la peine d'y consacrer une semaine pour les perfectionner | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |

<a id="tips-git-pr"></a>■ **Git / PR (5)**

| Astuce | Source |
|-----|--------|
| gardez les PR petites et ciblees — [p50 de 118 lignes](../../tips/claude-boris-2-tips-25-mar-26.md) (141 PR, 45K lignes modifiees en un jour), une fonctionnalite par PR, plus facile a reviser et reverter | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| toujours [squash merge](../../tips/claude-boris-2-tips-25-mar-26.md) les PR — historique lineaire propre, un commit par fonctionnalite, git revert et git bisect faciles | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| committez souvent — essayez de committer au moins une fois par heure, des que la tache est terminee | ![Shayan](../../!/tags/community-shayan.svg) |
| taguez [@claude](https://github.com/apps/claude) sur la PR d'un collegue pour generer automatiquement des regles de lint pour les retours de revue recurrents — automatisez-vous hors de la revue de code 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=2715) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=2715) |
| utilisez [/code-review](https://code.claude.com/docs/en/code-review) pour l'analyse multi-agent de PR — detecte les bugs, vulnerabilites de securite et regressions avant le merge | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031089411820228645) |

<a id="tips-debugging"></a>■ **Debugging (7)**

| Astuce | Source |
|-----|--------|
| prenez l'habitude de faire des captures d'ecran et de les partager avec Claude chaque fois que vous etes bloque | ![Shayan](../../!/tags/community-shayan.svg) |
| utilisez des MCP ([Claude in Chrome](https://code.claude.com/docs/en/chrome), [Playwright](https://github.com/microsoft/playwright-mcp), [Chrome DevTools](https://developer.chrome.com/blog/chrome-devtools-mcp)) pour que Claude voie les logs de la console Chrome tout seul | |
| demandez toujours a Claude d'executer le terminal (dont vous voulez voir les logs) en tache de fond pour un meilleur debugging | |
| [/doctor](https://code.claude.com/docs/en/cli-reference) pour diagnostiquer les problemes d'installation, d'authentification et de configuration | |
| une erreur pendant la compaction peut etre resolue en utilisant [/model](https://code.claude.com/docs/en/model-config) pour selectionner un modele a 1M de tokens, puis en executant [/compact](https://code.claude.com/docs/en/interactive-mode) | |
| utilisez un [cross-model](../../development-workflows/cross-model-workflow/cross-model-workflow.md) pour le QA — par ex. [Codex](https://github.com/shanraisshan/codex-cli-best-practice) pour la revue de plan et d'implementation | |
| la recherche agentique (glob + grep) bat le RAG — Claude Code a essaye et abandonne les bases de donnees vectorielles car le code derive et les permissions sont complexes | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3095) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3095) |

<a id="tips-utilities"></a>■ **Utilitaires (5)**

| Astuce | Source |
|-----|--------|
| terminaux [iTerm](https://iterm2.com/)/[Ghostty](https://ghostty.org/)/[tmux](https://github.com/tmux/tmux) plutot que l'IDE ([VS Code](https://code.visualstudio.com/)/[Cursor](https://www.cursor.com/)) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742753971769626) |
| [/voice](https://code.claude.com/docs/en/voice-dictation) ou [Wispr Flow](https://wisprflow.ai) pour le prompting vocal (productivite x10) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038454362226467112) |
| [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks) pour le feedback de Claude | ![Shayan](../../!/tags/community-shayan.svg) |
| [status line](https://github.com/shanraisshan/claude-code-status-line) pour la conscience du contexte et une compaction rapide | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700784019452195) ![Shayan](../../!/tags/community-shayan.svg) |
| explorez les fonctionnalites de [settings.json](../../best-practice/claude-settings.md) comme [Plans Directory](../../best-practice/claude-settings.md#plans-directory), [Spinner Verbs](../../best-practice/claude-settings.md#display--ux) pour une experience personnalisee | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701145023197516) |

<a id="tips-daily"></a>■ **Quotidien (2)**

| Astuce | Source |
|-----|--------|
| [mettez a jour](https://code.claude.com/docs/en/setup) Claude Code quotidiennement | ![Shayan](../../!/tags/community-shayan.svg) |
| commencez votre journee en lisant le [changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) | ![Shayan](../../!/tags/community-shayan.svg) |

![Boris Cherny + Team](../../!/tags/claude.svg)

| Article / Tweet | Source |
|-----------------|--------|
| [15 fonctionnalites cachees et sous-utilisees dans Claude Code (Boris) \| 30/Mar/26](../../tips/claude-boris-15-tips-30-mar-26.md) | [Tweet](https://x.com/bcherny/status/2038454336355999749) |
| [Squash Merging et distribution de taille de PR (Boris) \| 25/Mar/26](../../tips/claude-boris-2-tips-25-mar-26.md) | [Tweet](https://x.com/bcherny/status/2038552880018538749) |
| [Lecons de la construction de Claude Code : comment nous utilisons les Skills (Thariq) \| 17/Mar/26](../../tips/claude-thariq-tips-17-mar-26.md) | [Article](https://x.com/trq212/status/2033949937936085378) |
| [Code Review et Test Time Compute (Boris) \| 10/Mar/26](../../tips/claude-boris-2-tips-10-mar-26.md) | [Tweet](https://x.com/bcherny/status/2031089411820228645) |
| /loop — planifier des taches recurrentes jusqu'a 3 jours (Boris) \| 07 Mar 2026 | [Tweet](https://x.com/bcherny/status/2030193932404150413) |
| AskUserQuestion + ASCII Markdowns (Thariq) \| 28 Fev 2026 | [Tweet](https://x.com/trq212/status/2027543858289250472) |
| Voir comme un Agent - lecons de la construction de Claude Code (Thariq) \| 28 Fev 2026 | [Article](https://x.com/trq212/status/2027463795355095314) |
| Git Worktrees - 5 facons dont Boris les utilise \| 21 Fev 2026 | [Tweet](https://x.com/bcherny/status/2025007393290272904) |
| Lecons de la construction de Claude Code : le Prompt Caching est tout (Thariq) \| 20 Fev 2026 | [Article](https://x.com/trq212/status/2024574133011673516) |
| [12 facons de personnaliser vos claudes (Boris) \| 12/Fev/26](../../tips/claude-boris-12-tips-12-feb-26.md) | [Tweet](https://x.com/bcherny/status/2021699851499798911) |
| [10 astuces pour utiliser Claude Code de l'equipe (Boris) \| 01/Fev/26](../../tips/claude-boris-10-tips-01-feb-26.md) | [Tweet](https://x.com/bcherny/status/2017742741636321619) |
| [Comment j'utilise Claude Code — 13 astuces de ma configuration etonnamment simple (Boris) \| 03/Jan/26](../../tips/claude-boris-13-tips-03-jan-26.md) | [Tweet](https://x.com/bcherny/status/2007179832300581177) |
| Demandez a Claude de vous interviewer avec l'outil AskUserQuestion (Thariq) \| 28/Dec/25 | [Tweet](https://x.com/trq212/status/2005315275026260309) |
| Toujours utiliser le plan mode, donner a Claude un moyen de verifier, utiliser /code-review (Boris) \| 27/Dec/25 | [Tweet](https://x.com/bcherny/status/2004711722926616680) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

## 🎬 VIDEOS / PODCASTS

| Video / Podcast | Source | YouTube |
|-----------------|--------|---------|
| Tout ce qu'on a mal fait avec Research-Plan-Implement (Dex) \| 24 Mar 2026 \| MLOps Community | [![Dex](../../!/tags/community-dex.svg)](https://x.com/daborhyde) | [YouTube](https://youtu.be/YwZR6tc7qYg) |
| Construire Claude Code avec Boris Cherny (Boris) \| 04 Mar 2026 \| The Pragmatic Engineer | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/julbw1JuAz0) |
| Responsable de Claude Code : que se passe-t-il apres que le code est resolu (Boris) \| 19 Fev 2026 \| Lenny's Podcast | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/We7BZVKbCVw) |
| Dans Claude Code avec son createur Boris Cherny (Boris) \| 17 Fev 2026 \| Y Combinator | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/PQU9o_5rHC4) |
| Boris Cherny (createur de Claude Code) sur ce qui a fait progresser sa carriere (Boris) \| 15 Dec 2025 \| Ryan Peterman | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/AmdLVWMdjOk) |
| Les secrets de Claude Code par les ingenieurs qui l'ont construit (Cat) \| 29 Oct 2025 \| Every | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/IDSAMqip6ms) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

## 🔔 S'ABONNER

| Source | Nom | Badge |
|--------|------|-------|
| ![Reddit](https://img.shields.io/badge/-FF4500?style=flat&logo=reddit&logoColor=white) | [r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/), [r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/), [r/Anthropic](https://www.reddit.com/r/Anthropic/) | ![Boris + Team](../../!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Claude](https://x.com/claudeai), [Anthropic](https://x.com/AnthropicAI), [Boris](https://x.com/bcherny), [Thariq](https://x.com/trq212), [Cat](https://x.com/_catwu), [Lydia](https://x.com/lydiahallie), [Noah](https://x.com/noahzweben), [Anthony](https://x.com/amorriscode), [Alex](https://x.com/alexalbert__), [Kenneth](https://x.com/neilhtennek) | ![Boris + Team](../../!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Jesse Kriss](https://x.com/obra) ([Superpowers](https://github.com/obra/superpowers)), [Affaan Mustafa](https://x.com/affaanmustafa) ([ECC](https://github.com/affaan-m/everything-claude-code)), [Garry Tan](https://x.com/garrytan) ([gstack](https://github.com/garrytan/gstack)), [Dex Horthy](https://x.com/dexhorthy) ([HumanLayer](https://github.com/humanlayer/humanlayer)), [Kieran Klaassen](https://x.com/kieranklaassen) ([Compound Eng](https://github.com/EveryInc/compound-engineering-plugin)), [Tabish Gilani](https://x.com/0xTab) ([OpenSpec](https://github.com/Fission-AI/OpenSpec)), [Brian McAdams](https://x.com/BMadCode) ([BMAD](https://github.com/bmad-code-org/BMAD-METHOD)), [Lex Christopherson](https://x.com/official_taches) ([GSD](https://github.com/gsd-build/get-shit-done)), [Dani Avila](https://x.com/dani_avila7) ([CC Templates](https://github.com/davila7/claude-code-templates)), [Dan Shipper](https://x.com/danshipper) ([Every](https://every.to/)), [Andrej Karpathy](https://x.com/karpathy) ([AutoResearch](https://x.com/karpathy/status/2015883857489522876)), [Peter Steinberger](https://x.com/steipete) ([OpenClaw](https://x.com/openclaw)), [Sigrid Jin](https://x.com/realsigridjin) ([claw-code](https://github.com/ultraworkers/claw-code)), [Yeachan Heo](https://x.com/bellman_ych) ([oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)) | ![Community](../../!/tags/community.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Anthropic](https://www.youtube.com/@anthropic-ai) | ![Boris + Team](../../!/tags/claude.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Lenny's Podcast](https://www.youtube.com/@LennysPodcast), [Y Combinator](https://www.youtube.com/@ycombinator), [The Pragmatic Engineer](https://www.youtube.com/@pragmaticengineer), [Ryan Peterman](https://www.youtube.com/@ryanlpeterman), [Every](https://www.youtube.com/@every_media), [MLOps Community](https://www.youtube.com/@MLOps) | ![Community](../../!/tags/community.svg) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

## ☠️ STARTUPS / ENTREPRISES

| Claude | Remplace |
|-|-|
|[**Code Review**](https://code.claude.com/docs/en/code-review)|[Greptile](https://greptile.com), [CodeRabbit](https://coderabbit.ai), [Devin Review](https://devin.ai), [OpenDiff](https://opendiff.com), [Cursor BugBot](https://bugbot.dev)|
|[**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation)|[Wispr Flow](https://wisprflow.ai), [SuperWhisper](https://superwhisper.com/)|
|[**Remote Control**](https://code.claude.com/docs/en/remote-control)|[OpenClaw](https://openclaw.ai/)
|[**Claude in Chrome**](https://code.claude.com/docs/en/chrome)|[Playwright MCP](https://github.com/microsoft/playwright-mcp), [Chrome DevTools MCP](https://developer.chrome.com/blog/chrome-devtools-mcp)|
|[**Computer Use**](https://docs.anthropic.com/en/docs/agents-and-tools/computer-use)|[OpenAI CUA](https://openai.com/index/computer-using-agent/)|
|[**Cowork**](https://claude.com/blog/cowork-research-preview)|[ChatGPT Agent](https://openai.com/chatgpt/agent/), [Perplexity Computer](https://www.perplexity.ai/computer/), [Manus](https://manus.im)|
|[**Tasks**](https://x.com/trq212/status/2014480496013803643)|[Beads](https://github.com/steveyegge/beads)
|[**Plan Mode**](https://code.claude.com/docs/en/common-workflows)|[Agent OS](https://github.com/buildermethods/agent-os)|
|[**Skills / Plugins**](https://code.claude.com/docs/en/plugins)|Startups YC de wrappers IA ([reddit](https://reddit.com/r/ClaudeAI/comments/1r6bh4d/claude_code_skills_are_basically_yc_ai_startup/))|

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

<a id="billion-dollar-questions"></a>
![Billion-Dollar Questions](../../!/tags/billion-dollar-questions.svg)

*Discutez de ces questions dans les [Issues](https://github.com/noya21th/ClaudeCode-BestPractice/issues)*

**Memoire et instructions (4)**

1. Que devez-vous exactement mettre dans votre CLAUDE.md — et que devez-vous omettre ?
2. Si vous avez deja un CLAUDE.md, un fichier separate constitution.md ou rules.md est-il vraiment necessaire ?
3. A quelle frequence devez-vous mettre a jour votre CLAUDE.md, et comment savoir quand il est devenu obsolete ?
4. Pourquoi Claude ignore-t-il encore les instructions de CLAUDE.md — meme quand elles disent OBLIGATOIRE en majuscules ? ([reddit](https://reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/))

**Agents, Skills et Workflows (6)**

1. Quand devez-vous utiliser un command vs un agent vs un skill — et quand Claude Code vanilla est-il simplement meilleur ?
2. A quelle frequence devez-vous mettre a jour vos agents, commands et workflows a mesure que les modeles s'ameliorent ?
3. Donner a votre subagent une persona detaillee ameliore-t-il la qualite ? A quoi ressemble un "persona/prompt parfait" pour un subagent de recherche/QA ?
4. Devriez-vous vous fier au plan mode integre de Claude Code — ou construire votre propre command/agent de planification qui applique le workflow de votre equipe ?
5. Si vous avez un skill personnel (ex. /implement avec votre style de code), comment integrer les skills communautaires (ex. /simplify) sans conflits — et qui gagne quand ils sont en desaccord ?
6. Y sommes-nous ? Pouvons-nous convertir un codebase existant en specs, supprimer le code, et faire regenerer par l'IA exactement le meme code a partir de ces specs seules ?

**Specs et documentation (3)**

1. Chaque fonctionnalite de votre repo devrait-elle avoir une spec sous forme de fichier markdown ?
2. A quelle frequence devez-vous mettre a jour les specs pour qu'elles ne deviennent pas obsoletes quand une nouvelle fonctionnalite est implementee ?
3. Lors de l'implementation d'une nouvelle fonctionnalite, comment gerez-vous l'effet domino sur les specs des autres fonctionnalites ?

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

## RAPPORTS

<p align="center">
  <a href="../../reports/claude-agent-sdk-vs-cli-system-prompts.md"><img src="https://img.shields.io/badge/Agent_SDK_vs_CLI-555?style=for-the-badge" alt="Agent SDK vs CLI"></a>
  <a href="../../reports/claude-in-chrome-v-chrome-devtools-mcp.md"><img src="https://img.shields.io/badge/Browser_Automation_MCP-555?style=for-the-badge" alt="Browser Automation MCP"></a>
  <a href="../../reports/claude-global-vs-project-settings.md"><img src="https://img.shields.io/badge/Global_vs_Project_Settings-555?style=for-the-badge" alt="Global vs Project Settings"></a>
  <a href="../../reports/claude-skills-for-larger-mono-repos.md"><img src="https://img.shields.io/badge/Skills_in_Monorepos-555?style=for-the-badge" alt="Skills in Monorepos"></a>
  <br>
  <a href="../../reports/claude-agent-memory.md"><img src="https://img.shields.io/badge/Agent_Memory-555?style=for-the-badge" alt="Agent Memory"></a>
  <a href="../../reports/claude-advanced-tool-use.md"><img src="https://img.shields.io/badge/Advanced_Tool_Use-555?style=for-the-badge" alt="Advanced Tool Use"></a>
  <a href="../../reports/claude-usage-and-rate-limits.md"><img src="https://img.shields.io/badge/Usage_&_Rate_Limits-555?style=for-the-badge" alt="Usage & Rate Limits"></a>
  <a href="../../reports/claude-agent-command-skill.md"><img src="https://img.shields.io/badge/Agents_vs_Commands_vs_Skills-555?style=for-the-badge" alt="Agents vs Commands vs Skills"></a>
  <br>
  <a href="../../reports/llm-day-to-day-degradation.md"><img src="https://img.shields.io/badge/LLM_Degradation-555?style=for-the-badge" alt="LLM Degradation"></a>
</p>

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

![How to Use](../../!/tags/how-to-use.svg)

```
1. Lisez le repo comme un cours, apprenez ce que sont les commands, agents, skills et hooks avant d'essayer de les utiliser.
2. Clonez ce repo et experimentez avec les exemples, essayez /weather-orchestrator, ecoutez les sons des hooks, lancez les agent teams, pour voir comment les choses fonctionnent reellement.
3. Allez dans votre propre projet et demandez a Claude de suggerer quelles bonnes pratiques de ce repo vous devriez ajouter, donnez-lui ce repo comme reference pour qu'il sache ce qui est possible.
```

<a href="https://www.youtube.com/watch?v=AkAhkalkRY4"><img src="../../!/thumbnail/video-1.png" alt="Regarder sur YouTube" width="300"></a>

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="separateur de section" width="60" height="50">
</p>

## Remerciements

Basé sur [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice). Merci à Shayan Rais pour sa contribution open source.
