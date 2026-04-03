# GitHub Copilot Pro — Limites d'usage ($10/mois)

> Synthèse au 02/04/2026

## Quotas mensuels

| Ressource | Limite | Reset |
|---|---|---|
| Completions inline (autocomplétion) | **Illimitées** | — |
| Premium requests | **300/mois** | 1er du mois, 00:00 UTC |
| Overage (si activé) | $0.04/requête | Facturation en fin de cycle |

Les requêtes non utilisées **ne se cumulent pas** d'un mois à l'autre.

## Modèles et multiplicateurs

**Modèles inclus** (0 premium request sur plan payant) : GPT-5 mini, GPT-4.1, GPT-4o.

**Modèles premium** (consomment des premium requests avec multiplicateur) :

| Modèle | Multiplicateur | Requêtes effectives sur 300 |
|---|---|---|
| Claude Sonnet 4.5 | 1× | 300 |
| Claude Opus 4.5 | 3× | 100 |
| Modèles reasoning avancés | 5×–20× | 60–15 |

⚠️ Les multiplicateurs exacts varient — vérifier dans le model picker ou sur [GitHub Docs](https://docs.github.com/en/copilot/concepts/billing/copilot-requests).

## Comportement à l'épuisement du quota

1. Copilot bascule **silencieusement** sur GPT-4.1 (modèle inclus)
2. Un message discret apparaît dans le chat — facile à rater
3. La qualité des réponses chute significativement
4. L'autocomplétion inline reste fonctionnelle (illimitée)

## Suivi de la consommation

| Méthode | Accès |
|---|---|
| **Dashboard VS Code** | Clic sur l'icône Copilot (status bar) → % utilisé par catégorie |
| **GitHub.com** | `Settings > Billing > Metered usage > Copilot` |
| **Extension tierce** | "Copilot Token Tracker" (Rob Bos) — status bar temps réel + analytics |
| **Indicateur de contexte** | Barre de remplissage dans la zone de saisie du chat (ex: 15K/128K) |

## Limites de contexte (harness agent)

- Fenêtre effective : **~128K–192K tokens** (selon modèle)
- Reserved output : **~30–40%** de la fenêtre (non configurable)
- Contexte utilisable réel : **~80K–130K tokens**
- Le modèle sous-jacent peut supporter plus (ex: Opus 4.6 = 1M) — c'est une limite produit, pas modèle
- Compaction automatique quand la fenêtre se remplit ; manuelle via `/compact`

## Conseils pratiques

- **Quotidien** : rester sur GPT-4.1 (gratuit, illimité) pour les tâches courantes
- **Premium requests** : réserver aux tests comparatifs ou tâches complexes nécessitant des modèles avancés
- **Context management** : utiliser `/clear` + reprise via `#file:progress.md` plutôt que laisser compacter
- **Monitoring** : surveiller le modèle actif dans le model picker pour détecter un fallback silencieux
- **Budget** : ne pas activer l'overage sauf besoin explicite — le fallback GPT-4.1 suffit pour éviter les surprises

## Comparaison rapide des plans individuels

| Plan | Prix | Completions | Premium requests |
|---|---|---|---|
| Free | $0 | 2 000/mois | 50/mois |
| **Pro** | **$10/mois** | **Illimitées** | **300/mois** |
| Pro+ | $39/mois | Illimitées | 1 500/mois |

## Sources

- [Requests in GitHub Copilot](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)
- [Plans for GitHub Copilot](https://docs.github.com/en/copilot/get-started/plans)
- [Monitoring usage](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/monitor-premium-requests)
- [Billing for individuals](https://docs.github.com/en/copilot/concepts/billing/billing-for-individuals)
- [Context management (VS Code)](https://code.visualstudio.com/docs/copilot/chat/copilot-chat-context)
