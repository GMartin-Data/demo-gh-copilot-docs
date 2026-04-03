# Mapping des permissions — Claude Code → GitHub Copilot

> **Objectif** : transposer les règles de sécurité de `.claude/settings.json` vers Copilot Agent Mode.
> Identifier ce qui est transférable, ce qui nécessite une adaptation, et ce qui n'a pas d'équivalent.

---

## 1. Implémentation Copilot : snippet prêt à l'emploi

À ajouter dans `.vscode/settings.json` (versionnable dans le repo) :

```json
{
  "chat.tools.terminal.autoApprove": {

    // ── ALLOW (auto-approuvé, pas de confirmation) ──────────────

    "/^dbt\\s+(compile|parse|ls|test)\\b/":    true,
    "/^dbt\\s+debug\\b/":                       true,
    "/^git\\s+(add|commit|status|log|diff)\\b/": true,
    "/^uv\\s/":                                  true,
    "/^python\\s/":                              true,

    // ── DENY (bloqué, confirmation forcée) ──────────────────────

    "/^rm\\s+-rf\\b/":                           false,
    "/^sudo\\s/":                                false,
    "/^curl\\s.*\\|\\s*bash/":                   false,
    "/^wget\\s.*\\|\\s*bash/":                   false,
    "/^git\\s+push\\s+--force/":                 false,
    "/^git\\s+push\\s+origin\\s+main\\b/":       false,
    "/^dbt\\s+(run|build|seed)\\s+--target\\s+prod/": false

    // ── ASK (non matchées → Default Approvals = confirmation) ───
    // git push, dbt run, dbt build, dbt seed, snowsql
    // → pas dans la liste = tombent sur le comportement par défaut
    //   (confirmation manuelle requise)
  }
}
```

**Logique** : les commandes `true` s'exécutent sans interruption. Les commandes `false` sont explicitement bloquées. Tout le reste (non matché) déclenche une confirmation manuelle — c'est l'équivalent du `ask` de Claude Code.

---

## 2. Mapping règle par règle

### Section `deny` de Claude Code

| Règle Claude Code | Transférable ? | Équivalent Copilot | Notes |
|---|---|---|---|
| `Read(.env)` | ❌ Non | — | Copilot n'a pas de mécanisme pour interdire la lecture d'un fichier. Protection indirecte via `.gitignore` (fichier non indexé) mais l'agent peut toujours faire `cat .env` en terminal. |
| `Read(.env.*)` | ❌ Non | — | Idem. |
| `Read(~/.dbt/**)` | ❌ Non | — | Idem. Fichier hors workspace, non indexé par Copilot. Risque résiduel via commande terminal. |
| `Read(~/.ssh/**)` | ❌ Non | — | Idem. |
| `Read(secrets/**)` | ❌ Non | — | Idem. |
| `Bash(rm -rf *)` | ✅ Oui | `/^rm\\s+-rf\\b/`: false | Confirmation forcée. |
| `Bash(sudo *)` | ✅ Oui | `/^sudo\\s/`: false | Confirmation forcée. |
| `Bash(curl * \| bash)` | ✅ Oui | `/^curl\\s.*\\|\\s*bash/`: false | Confirmation forcée. |
| `Bash(wget * \| bash)` | ✅ Oui | `/^wget\\s.*\\|\\s*bash/`: false | Confirmation forcée. |
| `Bash(git push --force*)` | ✅ Oui | `/^git\\s+push\\s+--force/`: false | Confirmation forcée. |
| `Bash(git push origin main*)` | ✅ Oui | `/^git\\s+push\\s+origin\\s+main\\b/`: false | Confirmation forcée. |
| `Bash(dbt run --target prod*)` | ✅ Oui | `/^dbt\\s+(run\|build\|seed)\\s+--target\\s+prod/`: false | Élargi à `build` et `seed` par précaution. |

### Section `ask` de Claude Code

| Règle Claude Code | Transférable ? | Équivalent Copilot | Notes |
|---|---|---|---|
| `Bash(git push *)` | ✅ Implicite | Non listé → Default Approvals | Toute commande non matchée demande confirmation. `git push` (hors `--force` et `origin main` qui sont deny) tombe dans ce cas. |
| `Bash(dbt run *)` | ✅ Implicite | Non listé → Default Approvals | Idem. `dbt run` sans `--target prod` demande confirmation. |
| `Bash(dbt build *)` | ✅ Implicite | Non listé → Default Approvals | Idem. |
| `Bash(dbt seed *)` | ✅ Implicite | Non listé → Default Approvals | Idem. |
| `Bash(snowsql *)` | ✅ Implicite | Non listé → Default Approvals | Idem. |

### Section `allow` de Claude Code

| Règle Claude Code | Transférable ? | Équivalent Copilot | Notes |
|---|---|---|---|
| `Bash(dbt compile *)` | ✅ Oui | `/^dbt\\s+(compile\|parse\|ls\|test)\\b/`: true | Auto-approuvé. |
| `Bash(dbt parse *)` | ✅ Oui | Idem (groupé) | |
| `Bash(dbt ls *)` | ✅ Oui | Idem (groupé) | |
| `Bash(dbt debug --no-version-check)` | ✅ Oui | `/^dbt\\s+debug\\b/`: true | Simplifié (tout `dbt debug`, pas que `--no-version-check`). |
| `Bash(dbt test *)` | ✅ Oui | Groupé avec compile/parse/ls | |
| `Bash(git add *)` | ✅ Oui | `/^git\\s+(add\|commit\|status\|log\|diff)\\b/`: true | Auto-approuvé. |
| `Bash(git commit *)` | ✅ Oui | Idem (groupé) | |
| `Bash(git status)` | ✅ Oui | Idem (groupé) | |
| `Bash(git log *)` | ✅ Oui | Idem (groupé) | |
| `Bash(git diff *)` | ✅ Oui | Idem (groupé) | |
| `Bash(uv *)` | ✅ Oui | `/^uv\\s/`: true | Auto-approuvé. |
| `Bash(python *)` | ✅ Oui | `/^python\\s/`: true | Auto-approuvé. |

### Section `network` de Claude Code

| Règle Claude Code | Transférable ? | Équivalent Copilot | Notes |
|---|---|---|---|
| `*.snowflakecomputing.com` | ❌ Non | — | Copilot Agent Mode n'a pas de mécanisme de whitelist réseau. L'agent accède au réseau via les commandes terminal (dbt, snowsql) qui elles-mêmes utilisent les credentials du shell. Pas de couche réseau intermédiaire contrôlable. |
| `pypi.org` | ❌ Non | — | Idem. |
| `files.pythonhosted.org` | ❌ Non | — | Idem. |
| `hub.getdbt.com` | ❌ Non | — | Idem. |
| `github.com` | ❌ Non | — | Idem. |

### Section `sandbox` de Claude Code

| Règle Claude Code | Transférable ? | Équivalent Copilot | Notes |
|---|---|---|---|
| `sandbox.enabled: false` | ⚠️ Partiel | Terminal sandboxing (macOS/Linux) | Copilot propose un sandboxing terminal expérimental qui restreint l'accès au filesystem et au réseau pour les commandes agent. Pas un équivalent exact mais même intention. |

---

## 3. Synthèse des gaps

### Transféré intégralement (12 règles)

Toutes les règles `Bash(...)` de `deny` et `allow` se transposent en regex dans `chat.tools.terminal.autoApprove`. La sémantique est préservée : deny → `false`, allow → `true`, ask → non listé (fallback sur confirmation manuelle).

### Non transférable (10 règles)

| Catégorie | Nombre | Raison |
|-----------|--------|--------|
| Interdiction de lecture de fichiers (`Read(...)`) | 5 règles | Copilot n'a pas de mécanisme de contrôle de lecture fichier par l'agent. La seule protection est le `.gitignore` + ne pas mettre les fichiers sensibles dans le workspace. |
| Whitelist réseau (`network.allow`) | 5 règles | Copilot n'a pas de couche réseau contrôlable. Les commandes terminal accèdent au réseau directement. |

### Différences structurelles

| Aspect | Claude Code | Copilot |
|--------|------------|---------|
| **Fichier** | `.claude/settings.json` (dans le repo) | `.vscode/settings.json` (dans le repo si `.vscode/` committé) |
| **Portée** | Fichiers + terminal + réseau | Terminal uniquement |
| **Sémantique du deny** | L'agent ne peut pas tenter l'action | L'action demande confirmation (pas un blocage dur) |
| **Éditions de fichiers** | Contrôlable (`Read`, `Write`) | Pas de contrôle — toute édition est soit auto-approuvée soit soumise à confirmation globale |
| **Contrôle enterprise** | Aucun natif | Policies organisation (`ChatToolsAutoApprove`, etc.) |

---

## 4. Friction estimée pendant la démo

| Scénario | Approbations manuelles estimées |
|----------|-------------------------------|
| Claude Code (avec `settings.json` actuel) | ~5 (les `ask` : `dbt build`, `git push`) |
| Copilot sans `autoApprove` (Default Approvals pur) | ~35-50 (chaque fichier créé/édité + chaque commande terminal) |
| Copilot avec le snippet ci-dessus | ~10-15 (les éditions de fichiers + `dbt build` + `git push`) |

Le snippet réduit la friction d'un facteur 3 environ, mais un écart résiduel persiste parce que les éditions de fichiers ne sont pas couvertes par `chat.tools.terminal.autoApprove` — elles relèvent du système d'approbation global des outils, pas du terminal.
