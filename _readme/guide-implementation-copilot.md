# Guide d'implémentation — Adaptation Copilot Agent Mode

> **Contexte** : ce document référence tous les fichiers produits pendant la conversation Claude.ai
> d'adaptation du projet dbt SOFINCO pour GitHub Copilot Agent Mode.
> Chaque fichier est décrit, situé, et ordonné dans une séquence d'implémentation.

---

## Inventaire des livrables

### A. Fichiers à placer dans le repo dbt

Ces fichiers sont opérationnels — ils vont dans le repo `dbt_db_formation` et sont utilisés
par Copilot Agent Mode pendant la démo.

| # | Fichier produit | Destination | Rôle |
|---|----------------|-------------|------|
| A1 | `copilot-instructions.md` | repo: `.github/copilot-instructions.md` | Conventions SOFINCO auto-injectées dans chaque interaction Copilot |
| A2 | `implement-dbt.prompt.md` | repo: `.github/prompts/implement-dbt.prompt.md` | Prompt exécutable pour la session 1 (plan d'architecture) |
| A3 | `continue-dbt.prompt.md` | repo: `.github/prompts/continue-dbt.prompt.md` | Prompt exécutable pour les sessions 2+ (implémentation) |
| A4 | `progress.md` | repo: `progress.md` (racine) | Coordination inter-sessions — état d'avancement persisté entre les sessions |
| A5 | `load_env.ps1` | repo: `load_env.ps1` (racine) | Charge les credentials Snowflake depuis `.env` (équivalent PowerShell de `load_env.sh`) |
| A6 | `check-all.ps1` | **hors repo**: `evaluation/check-all.ps1` | Script d'évaluation automatisée — 22 critères, compatible PS 5.1 |
| A7 | `scripts/sf_query.py` | **hors repo**: `evaluation/scripts/sf_query.py` | Helper Python pour les requêtes Snowflake (remplace `snowsql`) |

### B. Documents de référence

Ces fichiers ne vont pas dans le repo. Ce sont des documents d'analyse et de cadrage
que tu consultes pour préparer et évaluer la démo.

| # | Fichier produit | Usage |
|---|----------------|-------|
| B1 | `mapping-permissions.md` | Mapping détaillé des 22 règles de sécurité Claude Code → Copilot. Contient le snippet `.vscode/settings.json` à copier. |
| B2 | `grille-comparaison.md` | Grille d'évaluation à 22 critères (13 existants + 9 nouveaux outillage). À remplir après chaque run. |
| B3 | `workflow-compare.md` | Workflow séquencé des deux démos côte à côte, avec checklists pré-démo. |

---

## Séquence d'implémentation

### Phase 1 — Structure du repo

**Objectif** : créer l'arborescence Copilot dans le repo `dbt_db_formation`.

**Prérequis** : le repo existe déjà avec la structure dbt template (`initialisation.md`).
Si tu travailles sur une branche séparée du repo Claude Code, crée-la d'abord :

```powershell
git checkout -b copilot-demo
```

**Étapes** :

```
1.1  Supprimer CLAUDE.md (pas pertinent pour le run Copilot)
1.2  Supprimer .claude/ (idem)
1.3  Créer le dossier .github/prompts/
1.4  Créer le dossier scripts/
```

**Commandes** :

```powershell
Remove-Item -Path "CLAUDE.md" -ErrorAction SilentlyContinue
Remove-Item -Path ".claude" -Recurse -ErrorAction SilentlyContinue
New-Item -ItemType Directory -Path ".github/prompts" -Force
New-Item -ItemType Directory -Path "scripts" -Force
```

**Résultat attendu** :

```
dbt_db_formation/
├── .github/
│   └── prompts/
├── scripts/
├── models/
│   ├── staging/
│   ├── intermediate/
│   └── marts/
├── exploration_sakila_standard.md   ← déjà présent
├── dbt_project.yml                  ← déjà présent
├── profiles.yml                     ← déjà présent
├── .env.example                     ← déjà présent
├── .env                             ← déjà présent, gitignored
└── .gitignore                       ← déjà présent
```

---

### Phase 2 — Fichiers de conventions et de contexte

**Objectif** : mettre en place les fichiers que Copilot lit automatiquement.

```
2.1  Copier `copilot-instructions.md` → `.github/copilot-instructions.md`
```

**Ce que ce fichier fait** : il contient les conventions SOFINCO (nommage, style SQL, tests,
matérialisations, sécurité) et un lien vers `exploration_sakila_standard.md`.
Copilot l'injecte automatiquement dans chaque interaction — c'est l'équivalent de `CLAUDE.md`.

**Vérification** : ouvre le fichier, confirme que la section "Contexte métier" contient
le lien Markdown vers `../exploration_sakila_standard.md`. Ce chemin relatif suppose que
`exploration_sakila_standard.md` est à la racine du repo et `copilot-instructions.md`
est dans `.github/`.

---

### Phase 3 — Prompt files

**Objectif** : créer les prompts exécutables via `/` dans le Chat view.

```
3.1  Copier `implement-dbt.prompt.md` → `.github/prompts/implement-dbt.prompt.md`
3.2  Copier `continue-dbt.prompt.md`  → `.github/prompts/continue-dbt.prompt.md`
```

**Ce que ces fichiers font** :

- `implement-dbt.prompt.md` — le prompt de la session 1. Il contient `#file:exploration_sakila_standard.md`
  qui attache le fichier d'exploration au contexte. Tu le lances avec `/implement-dbt` dans le Chat view.

- `continue-dbt.prompt.md` — le prompt des sessions 2 et suivantes. Il référence
  `#file:progress.md` + `#file:exploration_sakila_standard.md` et demande à l'agent
  d'implémenter la prochaine phase marquée ⬜. Tu le lances avec `/continue-dbt`.

**Vérification** : dans VS Code, ouvre le Chat view, tape `/` — les deux prompts devraient
apparaître dans la liste. Si ce n'est pas le cas, vérifier que le setting
`chat.promptFiles` est activé dans VS Code.

---

### Phase 4 — Fichier de progression

**Objectif** : initialiser le fichier de coordination inter-sessions.

```
4.1  Copier `progress.md` → `progress.md` (racine du repo)
```

**Ce que ce fichier fait** : il encode l'architecture cible (staging, intermediate, marts),
les décisions métier verrouillées, et un tableau de progression avec des ⬜ à cocher.
Entre chaque session Copilot, tu mets à jour ce fichier manuellement pour refléter
ce qui a été implémenté (⬜ → ✅).

**Pourquoi il existe** : la fenêtre de contexte Copilot est limitée à ~130K tokens.
Le travail est découpé en 3 sessions pour éviter la compaction. `progress.md` sert
de mémoire entre les sessions — l'équivalent de tes transition prompts dans Claude Code,
mais matérialisé dans un fichier.

**Vérification** : ouvre le fichier, confirme que toutes les phases sont à ⬜.

---

### Phase 5 — Permissions et sécurité

**Objectif** : configurer les auto-approbations terminal pour réduire la friction.

```
5.1  Créer `.vscode/settings.json` avec le contenu du snippet de `mapping-permissions.md`
```

**Contenu à copier** (depuis la section 1 de `mapping-permissions.md`) :

```json
{
  "chat.tools.terminal.autoApprove": {
    "/^dbt\\s+(compile|parse|ls|test)\\b/":    true,
    "/^dbt\\s+debug\\b/":                       true,
    "/^git\\s+(add|commit|status|log|diff)\\b/": true,
    "/^uv\\s/":                                  true,
    "/^python\\s/":                              true,
    "/^rm\\s+-rf\\b/":                           false,
    "/^sudo\\s/":                                false,
    "/^curl\\s.*\\|\\s*bash/":                   false,
    "/^wget\\s.*\\|\\s*bash/":                   false,
    "/^git\\s+push\\s+--force/":                 false,
    "/^git\\s+push\\s+origin\\s+main\\b/":       false,
    "/^dbt\\s+(run|build|seed)\\s+--target\\s+prod/": false
  }
}
```

**Ce que ce fichier fait** : les commandes `true` s'exécutent sans confirmation.
Les commandes `false` déclenchent une confirmation forcée. Tout le reste (non matché)
demande aussi confirmation — c'est le comportement Default Approvals.

**Transférabilité** : 12 des 22 règles de `.claude/settings.json` sont transposées ici.
Les 10 règles non transférables (lecture de fichiers sensibles + whitelist réseau)
sont documentées dans `mapping-permissions.md`.

**Vérification** : dans VS Code, `Ctrl+,` → chercher "terminal autoApprove" — les règles
doivent apparaître.

**Note** : si `.vscode/settings.json` existe déjà avec d'autres settings, fusionner
le bloc `chat.tools.terminal.autoApprove` dans le fichier existant.

---

### Phase 6 — Scripts

**Objectif** : mettre en place les scripts de développement (dans le repo)
et d'évaluation (hors du repo).

**Principe** : les scripts d'évaluation mesurent le livrable — ils ne font pas
partie du livrable. Les séparer évite de polluer le repo avec de l'outillage de test.

```
6.1  Copier `load_env.ps1`       → `load_env.ps1` (racine du repo)
6.2  Créer un dossier `evaluation/` à côté du repo (pas dedans)
6.3  Copier `check-all.ps1`      → `evaluation/check-all.ps1`
6.4  Copier `scripts/sf_query.py` → `evaluation/scripts/sf_query.py`
```

**Structure résultante** :

```
workspace/
├── dbt_db_formation/              ← le repo
│   ├── load_env.ps1               ← outil dev (dans le repo)
│   └── ...
│
└── evaluation/                    ← hors repo
    ├── check-all.ps1
    └── scripts/
        └── sf_query.py
```

**`load_env.ps1`** — charge les credentials `.env` dans la session PowerShell.
Équivalent de `source ./load_env.sh` en bash. Usage :

```powershell
. .\load_env.ps1
```

Le point initial est obligatoire (dot-sourcing) — sans lui, les variables restent
dans un sous-processus et ne sont pas visibles par dbt.

**`check-all.ps1`** — évalue les 22 critères de la grille. Compatible PS 5.1.
Accepte un paramètre `-ProjectPath` pour pointer vers le repo depuis l'extérieur.
Détecte automatiquement l'outil via la présence de `CLAUDE.md` ou
`.github/copilot-instructions.md`. Usage :

```powershell
# Depuis le dossier evaluation/
.\check-all.ps1 -ProjectPath ..\dbt_db_formation

# Forcer l'outil
.\check-all.ps1 -ProjectPath ..\dbt_db_formation -Tool copilot
```

**`scripts/sf_query.py`** — exécute les requêtes Snowflake via `snowflake-connector-python`
(déjà dans le venv dbt). Appelé par `check-all.ps1`, pas directement. Remplace `snowsql`
qui n'est pas installé sur Windows.

**Vérification** :

```powershell
# 1. Se placer dans le repo
cd dbt_db_formation

# 2. Activer le venv
.venv\Scripts\Activate.ps1

# 3. Charger les credentials
. .\load_env.ps1

# 4. Tester la connexion Snowflake
python ..\evaluation\scripts\sf_query.py "SELECT CURRENT_TIMESTAMP();"

# 5. Lancer l'évaluation (avant le run dbt — la plupart des critères échoueront)
..\evaluation\check-all.ps1 -ProjectPath . -Tool copilot
```

---

### Phase 7 — Configuration VS Code

**Objectif** : configurer le model picker et le thinking effort.

```
7.1  Ouvrir le Chat view (Ctrl+Shift+I)
7.2  Dans le model picker (en bas du chat), sélectionner Claude Opus 4.6
7.3  Cliquer la flèche à côté du modèle → Thinking Effort → High
7.4  Vérifier que le permission level est "Default Approvals"
```

Ces réglages ne sont pas versionnés — ils sont dans l'UI VS Code.
À refaire si tu changes de machine ou de profil VS Code.

---

### Phase 8 — Commit et vérification finale

```powershell
git add .github/ progress.md load_env.ps1 .vscode/settings.json
git commit -m "feat: copilot agent mode setup"
```

**Vérification finale** — l'arborescence doit être :

```
workspace/
│
├── dbt_db_formation/                        ← le repo
│   ├── .github/
│   │   ├── copilot-instructions.md          ← conventions SOFINCO
│   │   └── prompts/
│   │       ├── implement-dbt.prompt.md      ← prompt session 1
│   │       └── continue-dbt.prompt.md       ← prompt sessions 2+
│   ├── .vscode/
│   │   └── settings.json                    ← terminal autoApprove
│   ├── models/
│   │   ├── staging/
│   │   ├── intermediate/
│   │   └── marts/
│   ├── exploration_sakila_standard.md       ← contexte métier
│   ├── progress.md                          ← coordination inter-sessions
│   ├── load_env.ps1                         ← credentials PowerShell
│   ├── load_env.sh                          ← conservé pour référence bash
│   ├── dbt_project.yml
│   ├── profiles.yml
│   ├── .env.example
│   ├── .env                                 ← gitignored
│   └── .gitignore
│
└── evaluation/                              ← hors repo
    ├── check-all.ps1                        ← évaluation automatisée
    └── scripts/
        └── sf_query.py                      ← helper Snowflake
```

---

## Documents et outils hors repo

Ces éléments ne vont pas dans le repo dbt. Les documents de référence restent dans
le projet Claude.ai. Les scripts d'évaluation vivent dans un dossier `evaluation/` à côté du repo.

### Scripts d'évaluation (`evaluation/`)

| Fichier | Usage |
|---------|-------|
| `check-all.ps1` | Évaluation automatisée des 22 critères. Lancer avec `-ProjectPath ..\dbt_db_formation` |
| `scripts/sf_query.py` | Helper Snowflake appelé par `check-all.ps1` |

### Documents de référence (projet Claude.ai)

| Document | Quand le consulter |
|----------|-------------------|
| `mapping-permissions.md` | Avant la démo — comprendre les 12 règles transposées et les 10 gaps |
| `grille-comparaison.md` | Après chaque run — remplir les 22 critères côte à côte |
| `workflow-compare.md` | Avant la démo — suivre la séquence d'exécution et cocher la checklist pré-démo |

---

## Checklist récapitulative

- [ ] Phase 1 : structure du repo nettoyée (pas de `CLAUDE.md`, pas de `.claude/`)
- [ ] Phase 2 : `.github/copilot-instructions.md` en place
- [ ] Phase 3 : prompt files en place, testés avec `/` dans le Chat view
- [ ] Phase 4 : `progress.md` initialisé, toutes les phases à ⬜
- [ ] Phase 5 : `.vscode/settings.json` avec terminal autoApprove
- [ ] Phase 6 : `load_env.ps1` dans le repo, `check-all.ps1` + `sf_query.py` dans `evaluation/` hors repo, testés
- [ ] Phase 7 : Claude Opus 4.6 sélectionné, Thinking Effort → High, Default Approvals
- [ ] Phase 8 : tout committé
- [ ] Connexion Snowflake vérifiée (`dbt debug` passe)
- [ ] Prêt pour le run
