# Workflow Copilot Agent Mode — Environnement Linux

**Légende** : DEV = développeur / COP = Copilot Agent Mode

---

## Préparation (une seule fois)

| # | Qui | Action |
|---|-----|--------|
| 1 | DEV | Créer le repo et la structure template (`dbt init` + ajustements `initialisation.md`) |
| 2 | DEV | Copier `exploration_sakila_standard.md` dans le repo |
| 3 | DEV | `git init` + premier commit : `docs: initial project template` |
| 4 | DEV | Créer `.env` depuis `.env.example` et renseigner les trois credentials |
| 5 | DEV | Créer `.github/copilot-instructions.md` (transposition des conventions SOFINCO) |
| 6 | DEV | Créer `.github/prompts/implement-dbt.prompt.md` (prompt de planification — Plan agent) |
| 7 | DEV | Créer `.github/prompts/continue-dbt.prompt.md` (prompt session 2) |
| 8 | DEV | Créer `progress.md` (template de coordination inter-sessions) |
| 9 | DEV | Configurer `.vscode/settings.json` (terminal autoApprove) |
| 10 | DEV | Dans VS Code : sélectionner Claude Opus 4.6 + Thinking Effort → High |

---

## Démo — Exécution (2 sessions)

> **Mécanisme clé — Plan agent + handoff.**
> Copilot dispose d'un agent dédié à la planification (Plan agent), séparé de l'agent
> d'implémentation (Agent Mode). Le Plan agent analyse la codebase, pose des questions
> de clarification, et produit un plan structuré — sans écrire de code.
>
> Une fois le plan validé, un bouton **"Start Implementation"** transfère automatiquement
> l'historique de conversation et le contexte vers Agent Mode. C'est le *handoff* :
> l'agent d'implémentation repart avec le plan complet en mémoire, sans copie manuelle.

### Session 1 — Plan + Staging

| # | Qui | Action | Durée estimée |
|---|-----|--------|---------------|
| 11 | DEV | `source ./load_env.sh` (terminal intégré VS Code) | 5 sec |
| 12 | DEV | Ouvrir Chat view (`Ctrl+Shift+I`), sélectionner **Plan** dans le dropdown agents | 5 sec |
| 13 | DEV | Taper `/implement-dbt` (charge le prompt file avec `#file:exploration_sakila_standard.md`) | 5 sec |
| 14 | COP | Lire l'exploration, poser des questions de clarification si nécessaire | 1-3 min |
| 15 | DEV | Répondre aux questions, itérer jusqu'au plan complet | 1-3 min |
| 16 | COP | Produire le plan d'architecture structuré | 2-5 min |
| 17 | DEV | Valider le plan | 1-2 min |
| 18 | DEV | Cliquer **"Start Implementation"** → handoff vers Agent Mode | 5 sec |
| 19 | COP | Implémenter staging (11 modèles + sources.yml + models.yml) | 5-10 min |
| 20 | DEV | Approuver les créations de fichiers (~15 approbations) | 2-3 min |
| 21 | DEV | Valider le staging, mettre à jour `progress.md` | 2 min |
| 22 | DEV | Fermer la session (New Chat) | 5 sec |

### Session 2 — Intermediate + Marts + Build

| # | Qui | Action | Durée estimée |
|---|-----|--------|---------------|
| 23 | DEV | Nouvelle session Agent, taper `/continue-dbt` | 5 sec |
| 24 | COP | Lire `progress.md`, implémenter intermediate (3 modèles) | 3-5 min |
| 25 | DEV | Approuver les créations de fichiers (~5 approbations) | 1 min |
| 26 | COP | Implémenter marts (3 modèles + docs) | 5-10 min |
| 27 | DEV | Approuver les créations de fichiers (~8 approbations) | 1-2 min |
| 28 | COP | `dbt build` → demande confirmation (non listé dans autoApprove) | — |
| 29 | DEV | Approuver | 2 sec |
| 30 | COP | Exécution `dbt build` (run + test) | 1-3 min |
| 31 | COP | Corriger si erreurs, relancer | 0-5 min |
| 32 | DEV | Mettre à jour `progress.md` | 1 min |
| 33 | DEV | `git commit` : `feat: dbt project complete` | 30 sec |

| | | **Total estimé** | **22-50 min** |

**Approbations manuelles** : ~25 (fichiers + commandes terminal non auto-approuvées)
**Sessions** : 2
**Compaction** : improbable avec le découpage en 2 sessions

> **Plan de repli — 3 sessions.** Si l'indicateur de contexte dépasse ~60% à la fin
> du staging (étape 21), fermer la session et ouvrir une session 3 pour intermediate +
> marts + build via `/continue-dbt`. Le workflow le permet sans modification —
> `progress.md` sert de pont.

---

## Notes

- **`source ./load_env.sh`** doit être exécuté par DEV avant de lancer Copilot — l'agent ne doit pas accéder à `.env` directement.
- **Plan agent + handoff** : le Plan agent est sélectionné dans le dropdown agents. Il produit le plan sans écrire de code. Le bouton "Start Implementation" transfère l'historique complet vers Agent Mode — pas de copie manuelle du plan nécessaire.
- **`progress.md` reste nécessaire** entre la session 1 (plan + staging) et la session 2 (intermediate + marts + build). Le handoff supprime la copie manuelle *du plan*, pas la coordination inter-sessions.
- **Les approbations de fichiers** restent le principal facteur de friction. Le `terminal autoApprove` réduit les approbations terminal mais ne couvre pas les éditions de fichiers.
- **Surveiller l'indicateur de contexte** dans le Chat view. Si la barre dépasse ~60% avant la fin du staging, basculer en plan de repli 3 sessions.
- **`/effort max`** n'existe pas côté Copilot. Thinking Effort plafonne à High. Documenter le niveau pour la comparaison avec Claude Code.

---

## Checklist pré-démo

- [ ] `.env` renseigné
- [ ] `source ./load_env.sh` exécuté (terminal intégré VS Code)
- [ ] `.github/copilot-instructions.md` en place
- [ ] `.github/prompts/implement-dbt.prompt.md` en place
- [ ] `.github/prompts/continue-dbt.prompt.md` en place
- [ ] `progress.md` initialisé (toutes les phases ⬜)
- [ ] `.vscode/settings.json` avec terminal autoApprove configuré
- [ ] `exploration_sakila_standard.md` dans le repo et committé
- [ ] Claude Opus 4.6 sélectionné dans le model picker
- [ ] Thinking Effort → High
- [ ] Plan agent accessible dans le dropdown agents
- [ ] `dbt debug` passe (connexion Snowflake OK)
