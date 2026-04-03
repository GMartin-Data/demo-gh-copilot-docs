# Workflow comparé — Claude Code vs GitHub Copilot Agent Mode

**Légende** : DEV = développeur / CC = Claude Code / COP = Copilot Agent Mode

---

## Préparation (une seule fois par outil)

### Tronc commun

| # | Qui | Action |
|---|-----|--------|
| 1 | DEV | Créer le repo et la structure template (`dbt init` + ajustements `initialisation.md`) |
| 2 | DEV | Copier `exploration_sakila_standard.md` dans le repo |
| 3 | DEV | `git init` + premier commit : `docs: initial project template` |
| 4 | DEV | Créer `.env` depuis `.env.example` et renseigner les trois credentials |

### Spécifique Claude Code

| # | Qui | Action |
|---|-----|--------|
| 5a | DEV | Copier `claude_md_sofinco.md` sous le nom `CLAUDE.md` à la racine |
| 6a | DEV | Vérifier `.claude/settings.json` (permissions deny/ask/allow) |

### Spécifique Copilot

| # | Qui | Action |
|---|-----|--------|
| 5b | DEV | Créer `.github/copilot-instructions.md` (transposition des conventions SOFINCO) |
| 6b | DEV | Créer `.github/prompts/implement-dbt.prompt.md` (prompt session 1) |
| 7b | DEV | Créer `.github/prompts/continue-dbt.prompt.md` (prompt sessions 2+) |
| 8b | DEV | Créer `progress.md` (template de coordination inter-sessions) |
| 9b | DEV | Configurer `.vscode/settings.json` (terminal autoApprove) |
| 10b | DEV | Dans VS Code : sélectionner Claude Opus 4.6 + Thinking Effort → High |

---

## Démo — Exécution

### Claude Code (1 session)

| # | Qui | Action | Durée estimée |
|---|-----|--------|---------------|
| 7 | DEV | `source ./load_env.sh` | 5 sec |
| 8 | DEV | Lancer Claude Code : `claude` | 5 sec |
| 9 | DEV | Soumettre le prompt avec `@exploration_sakila_standard.md` (mode plan : Shift+Tab ×2) | 30 sec |
| 10 | CC | Lire l'exploration, proposer un plan d'architecture | 2-5 min |
| 11 | DEV | Valider ou ajuster le plan | 1-3 min |
| 12 | CC | Implémenter staging (11 modèles + sources.yml + models.yml) | 5-10 min |
| 13 | DEV | Valider le staging avant passage à la suite | 1-2 min |
| 14 | CC | Implémenter intermediate (3 modèles) | 3-5 min |
| 15 | DEV | Valider | 1 min |
| 16 | CC | Implémenter marts (3 modèles + docs) | 5-10 min |
| 17 | DEV | Valider | 1-2 min |
| 18 | CC | `dbt build` → demande confirmation (règle `ask`) | — |
| 19 | DEV | Approuver | 2 sec |
| 20 | CC | Exécution `dbt build` (run + test) | 1-3 min |
| 21 | CC | Corriger si erreurs, relancer | 0-5 min |
| 22 | DEV | `git commit` : `feat: dbt project complete` | 30 sec |
| | | **Total estimé** | **20-45 min** |

**Approbations manuelles** : ~5 (plan + 3 validations couche + dbt build)
**Sessions** : 1
**Compaction** : aucune (1M tokens)

---

### Copilot Agent Mode (3 sessions)

#### Session 1 — Plan

| # | Qui | Action | Durée estimée |
|---|-----|--------|---------------|
| 11 | DEV | `source ./load_env.sh` (terminal PowerShell dans VS Code) | 5 sec |
| 12 | DEV | Ouvrir Chat view (`Ctrl+Shift+I`), sélectionner Agent mode | 5 sec |
| 13 | DEV | Taper `/implement-dbt` (charge le prompt file) | 5 sec |
| 14 | COP | Lire l'exploration (via `#file:`), proposer un plan d'architecture | 2-5 min |
| 15 | DEV | Valider ou ajuster le plan | 1-3 min |
| 16 | DEV | Copier le plan validé dans `progress.md`, marquer la phase ✅ | 2 min |
| 17 | DEV | Fermer la session (New Chat) | 5 sec |

#### Session 2 — Staging

| # | Qui | Action | Durée estimée |
|---|-----|--------|---------------|
| 18 | DEV | Nouvelle session Agent, taper `/continue-dbt` | 5 sec |
| 19 | COP | Lire `progress.md` + `exploration_sakila_standard.md`, implémenter staging | 5-10 min |
| 20 | DEV | Approuver les créations de fichiers (~15 approbations) | 2-3 min |
| 21 | DEV | Valider le staging, mettre à jour `progress.md` | 2 min |
| 22 | DEV | Fermer la session | 5 sec |

#### Session 3 — Intermediate + Marts + Build

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

| | | **Total estimé** | **25-55 min** |

**Approbations manuelles** : ~30 (fichiers + commandes terminal non auto-approuvées)
**Sessions** : 3
**Compaction** : improbable avec le découpage en 3 sessions

---

## Comparaison directe

| Aspect | Claude Code | Copilot Agent Mode |
|--------|------------|-------------------|
| **Étapes de préparation** | 6 | 10 |
| **Étapes d'exécution** | 16 | 23 |
| **Sessions** | 1 | 3 |
| **Approbations manuelles** | ~5 | ~30 |
| **Overhead inter-sessions** | 0 | ~6 min (sauvegarder progress, ouvrir nouveau chat, ré-attacher contexte) |
| **Durée estimée totale** | 20-45 min | 25-55 min |
| **Surcoût Copilot** | — | +5-10 min (friction approbations + changements de session) |
| **Fichiers spécifiques outil** | 2 | 5 |
| **Risque de perte de contexte** | Nul | Faible (si 3 sessions respectées) |

---

## Notes

### Les deux outils

- **`source ./load_env.sh`** doit être exécuté par DEV avant de lancer l'outil — ni Claude Code ni Copilot ne doivent accéder à `.env` directement.
- **Validation humaine aux étapes clés** — le rôle de DEV dans la boucle est identique : plan, couches, exécution. C'est le message fort de la démo, quel que soit l'outil.

### Spécifique Copilot

- **Le prompt file `/implement-dbt`** inclut `#file:exploration_sakila_standard.md` — c'est ce qui injecte le contexte métier. Si le fichier n'est pas dans le workspace, le `#file:` échoue silencieusement.
- **`progress.md` doit être mis à jour manuellement** par le DEV entre chaque session. C'est une discipline opérationnelle absente côté Claude Code.
- **Les approbations de fichiers** sont le principal facteur de friction. Le `terminal autoApprove` réduit les approbations terminal mais ne couvre pas les éditions de fichiers.
- **Surveiller l'indicateur de contexte** dans le Chat view. Si la barre dépasse 70% avant la fin d'une session, envisager de fragmenter plus tôt.

### Spécifique Claude Code

- **Mode plan** : Shift+Tab ×2 (ou `/plan`) avant de soumettre le prompt initial. Pas d'équivalent exact côté Copilot — le Plan agent est un agent séparé, pas un mode du même agent.
- **`/effort max`** : disponible uniquement côté Claude Code. Copilot plafonne à High. Documenter le niveau utilisé pour la comparaison.

---

## Checklist pré-démo

### Claude Code

- [ ] `.env` renseigné
- [ ] `source ./load_env.sh` exécuté
- [ ] `CLAUDE.md` à la racine
- [ ] `.claude/settings.json` en place
- [ ] `exploration_sakila_standard.md` dans le repo
- [ ] `dbt debug` passe (connexion Snowflake OK)
- [ ] `/effort max` configuré

### Copilot Agent Mode

- [ ] `.env` renseigné
- [ ] `source ./load_env.sh` exécuté (terminal PowerShell VS Code)
- [ ] `.github/copilot-instructions.md` en place
- [ ] `.github/prompts/implement-dbt.prompt.md` en place
- [ ] `.github/prompts/continue-dbt.prompt.md` en place
- [ ] `progress.md` initialisé (toutes les phases ⬜)
- [ ] `.vscode/settings.json` avec terminal autoApprove configuré
- [ ] `exploration_sakila_standard.md` dans le repo et committé
- [ ] Claude Opus 4.6 sélectionné dans le model picker
- [ ] Thinking Effort → High
- [ ] `dbt debug` passe (connexion Snowflake OK)
