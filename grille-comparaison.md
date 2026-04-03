# Grille de comparaison — Claude Code vs GitHub Copilot Agent Mode

> **Protocole** : même modèle (Claude Opus 4.6), même prompt, même dataset, même conventions SOFINCO.
> La seule variable est l'outil. Les critères 1–13 sont identiques à `evaluation-baseline.md`.
> Les critères 14–22 mesurent les différences structurelles entre les deux outils.

---

## Grille synthétique

| # | Critère | Claude Code | Copilot Agent Mode |
|---|---------|-------------|-------------------|
| | **A. Exécution pipeline** | | |
| 1 | `dbt run` sans erreur | | |
| 2 | `dbt test` sans erreur | | |
| | **B. Architecture DAG** | | |
| 3 | Nb modèles staging | /11 | /11 |
| 4 | Nb modèles intermediate | /3 | /3 |
| 5 | Nb modèles marts | /3 | /3 |
| | **C. Intelligence métier** | | |
| 6 | `late_fee_days` exposée comme métrique explicite | | |
| 7 | `return_date IS NULL` géré (183 DVDs non rendus) | | |
| 8 | `original_language_id` ignoré | | |
| 9 | Relation N:1 PAYMENT→RENTAL agrégée | | |
| 10 | Scoring FM dans `fct_customers__customer_id` | | |
| | **D. Conformité entreprise** | | |
| 11 | Conventions SOFINCO respectées | | |
| | **E. Efficience** | | |
| 12 | Durée totale (min) | | |
| 13 | Nb interventions humaines (corrections) | | |
| | **F. Outillage** *(nouveau)* | | |
| 14 | Nb sessions nécessaires | | |
| 15 | Nb approbations manuelles | | |
| 16 | Compaction de contexte observée | | |
| 17 | Perte de détail métier après compaction | | |
| 18 | Thinking effort utilisé | | |
| 19 | Setup initial (temps) | | |
| 20 | Fichiers de config créés | | |
| 21 | Règles de sécurité transposables | /22 | /22 |
| 22 | Premium requests consommées | — | |

---

## Détail des nouveaux critères (section F)

### Critère 14 — Nb sessions nécessaires

**Quoi** : combien de sessions de chat distinctes sont nécessaires pour compléter le run (plan → staging → intermediate → marts → dbt build → dbt test).

**Attendu** :
- Claude Code : 1 session (contexte 1M, pas de compaction)
- Copilot : 3 sessions (plan, staging, intermediate+marts) — imposé par la fenêtre ~130K utilisable

**Pourquoi ça compte** : chaque changement de session est une friction opérationnelle — sauvegarder le progress, ouvrir un nouveau chat, ré-attacher les fichiers. Le surcoût est mesurable en minutes.

### Critère 15 — Nb approbations manuelles

**Quoi** : nombre total de fois où le développeur doit cliquer "Allow" / "Approve" / confirmer une action de l'agent.

**Attendu** :
- Claude Code : ~5 (les commandes `ask` : `dbt build`, `git push`)
- Copilot avec `autoApprove` configuré : ~10-15 (éditions de fichiers non couvertes)
- Copilot sans `autoApprove` : ~35-50

**Comment mesurer** : compteur incrémenté manuellement pendant le run. Distinguer les approbations terminal vs les approbations d'édition de fichier.

### Critère 16 — Compaction de contexte observée

**Quoi** : la compaction automatique s'est-elle déclenchée pendant le run ?

**Comment observer** : surveiller l'indicateur de contexte dans le Chat view. Si la barre atteint ~95% puis redescend, la compaction a eu lieu. Copilot affiche aussi un message dans le chat quand il compacte.

**Attendu** :
- Claude Code : non (1M tokens)
- Copilot : possible à partir de la session 2 si non fragmenté, improbable avec le workflow 3 sessions

### Critère 17 — Perte de détail métier après compaction

**Quoi** : si la compaction s'est déclenchée (critère 16), l'agent a-t-il perdu des informations critiques de l'exploration ?

**Signaux de perte** :
- L'agent oublie le filtre `rental_date < '2006-01-01'` pour le scoring FM
- L'agent ne gère pas les 183 DVDs non rendus (`return_date IS NULL`)
- L'agent ne fait pas l'agrégation N:1 PAYMENT→RENTAL
- L'agent inclut `original_language_id`

**Attendu** :
- Claude Code : non applicable
- Copilot : non si workflow 3 sessions respecté, possible si tout est fait en 1 session

### Critère 18 — Thinking effort utilisé

**Quoi** : le niveau de réflexion configuré pour le modèle.

**Valeurs possibles** :
- Claude Code : `low`, `medium`, `high`, `max` (via `/effort`)
- Copilot : Low, Medium, High (via model picker submenu)

**Impact sur la comparaison** : si Claude Code est en `max` et Copilot en High, une partie de l'écart de qualité pourrait venir du budget de réflexion, pas de l'outil. Documenter le niveau exact utilisé de chaque côté.

### Critère 19 — Setup initial (temps)

**Quoi** : temps entre "je veux utiliser cet outil" et "je peux soumettre le premier prompt".

**Claude Code** :
- `CLAUDE.md` existe déjà (conventions SOFINCO)
- `.claude/settings.json` existe déjà (permissions)
- `source ./load_env.sh` + lancer `claude`
- **Estimé : 1-2 minutes**

**Copilot** :
- Créer `.github/copilot-instructions.md` (transposition de `CLAUDE.md`)
- Créer `.github/prompts/implement-dbt.prompt.md`
- Créer `.github/prompts/continue-dbt.prompt.md`
- Créer `progress.md`
- Configurer `.vscode/settings.json` (terminal autoApprove)
- Sélectionner Opus 4.6 + Thinking Effort High dans le model picker
- `source ./load_env.sh` + ouvrir VS Code
- **Estimé : 10-15 minutes** (première fois) / **2-3 minutes** (runs suivants)

### Critère 20 — Fichiers de config créés

**Quoi** : inventaire des fichiers spécifiques à l'outil dans le repo.

| Claude Code | Copilot |
|---|---|
| `CLAUDE.md` | `.github/copilot-instructions.md` |
| `.claude/settings.json` | `.vscode/settings.json` |
| — | `.github/prompts/implement-dbt.prompt.md` |
| — | `.github/prompts/continue-dbt.prompt.md` |
| — | `progress.md` |

**Observation** : Copilot nécessite plus de fichiers d'infrastructure pour atteindre un résultat comparable. Le surcoût vient de la fragmentation en sessions (prompt files + progress) et de l'absence de permissions intégrées (settings VS Code séparé).

### Critère 21 — Règles de sécurité transposables

**Quoi** : sur les 22 règles de `.claude/settings.json`, combien trouvent un équivalent côté Copilot.

**Résultat** : 12/22. Détail dans `mapping-permissions.md`.

**Gaps** :
- 5 règles de lecture de fichiers sensibles (`Read(...)`) — aucun équivalent
- 5 règles de whitelist réseau (`network.allow`) — aucun équivalent
- Le `deny` de Claude Code est un blocage dur. Le `false` de Copilot est une confirmation forcée contournable.

### Critère 22 — Premium requests consommées

**Quoi** : nombre de premium requests Copilot consommées pendant le run. Opus 4.6 coûte ×3 par requête.

**Pourquoi c'est un critère enterprise** : un plan Business offre 300 premium requests/mois. Un run complet peut consommer 30-50 requêtes agent × 3 = 90-150 premium requests. C'est 30-50% du budget mensuel pour une seule démo.

**Claude Code** : non applicable (facturation différente — abonnement Max/Pro, pas de compteur par requête sur le même modèle).

---

## Message de la comparaison

Les critères 1–13 mesurent **la qualité du livrable**. Si les deux outils produisent le même résultat sur ces critères (attendu, puisque le modèle est identique), la différenciation se fait sur les critères 14–22 : **le coût opérationnel pour y arriver**.

Le message enterprise n'est pas "quel outil produit le meilleur dbt" — c'est :

> **À qualité de livrable égale, quel outil impose le moins de friction opérationnelle et offre les meilleures garanties de sécurité ?**
