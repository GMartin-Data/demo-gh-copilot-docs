# Comparaison GPT-5.2-Codex / GPT-5.3-Codex / GPT-5.4 / GPT-5.4 mini

> Synthèse au 02/04/2026 — Sources : OpenAI, NxCode, Morphllm, TokenCost, OpenRouter

## Vue d'ensemble

| | GPT-5.2-Codex | GPT-5.3-Codex | GPT-5.4 | GPT-5.4 mini |
|---|---|---|---|---|
| **Sortie** | 14 jan. 2026 | 5 fév. 2026 | 5 mars 2026 | 17 mars 2026 |
| **Positionnement** | Coding agent spécialisé | Coding agent spécialisé (successeur) | Modèle frontier unifié | Mini efficient polyvalent |
| **Statut** | Précédé par 5.3-Codex | Actif, intégré dans 5.4 | **Modèle principal recommandé** | Actif |
| **Knowledge cutoff** | — | — | Août 2025 | Août 2025 |

## Context window

| | GPT-5.2-Codex | GPT-5.3-Codex | GPT-5.4 | GPT-5.4 mini |
|---|---|---|---|---|
| **Context window** | 400K | 1M | 1.05M (272K standard, 1M expérimental) | 400K |
| **Max output** | — | — | 128K | — |
| **Surcharge >272K** | Non | Non | Oui (2× input, 1.5× output) | Non |

## Pricing API (par million de tokens)

| | GPT-5.2-Codex | GPT-5.3-Codex | GPT-5.4 | GPT-5.4 mini |
|---|---|---|---|---|
| **Input** | — | $1.25 | $2.50 | $0.75 |
| **Output** | — | $10.00 | $15.00 | $4.50 |
| **Cached input** | — | — | $1.25 | — |
| **Ratio vs 5.4** | — | ~50% du coût | Référence | ~30% du coût |

Note : GPT-5.4 Pro existe aussi à $30/$180 per MTok pour le raisonnement profond enterprise.

## Benchmarks clés

| Benchmark | GPT-5.2-Codex | GPT-5.3-Codex | GPT-5.4 | GPT-5.4 mini |
|---|---|---|---|---|
| **SWE-bench Verified** | — | ~80% | ~80% | — |
| **SWE-bench Pro** | — | 55.6% | 57.7% | Proche de 5.4 |
| **Terminal-Bench 2.0** | — | 77.3% | 75.1% | — |
| **OSWorld (computer use)** | — | 64% | **75%** (> humain 72.4%) | Bon (multimodal) |
| **GDPval (knowledge work)** | — | N/A | **83%** | — |
| **ARC-AGI-2** | — | — | 73.3% (vs 52.9% pour 5.2) | — |

## Capacités différenciantes

### GPT-5.2-Codex
- Premier modèle Codex agentic autonome d'OpenAI
- Context compaction intelligente pour les sessions longues
- Focus cybersécurité (analyse de vulnérabilités)
- Précédé par GPT-5.3-Codex — en voie de dépréciation

### GPT-5.3-Codex
- **Midtask steering** : redirection d'un build en cours d'exécution
- **Skills** : routines d'automatisation portables en fichiers
- Sessions longues (heures, pas minutes)
- ~25% plus rapide que 5.2-Codex
- Throughput élevé : 61.9 tokens/sec
- Avantage sur Terminal-Bench 2.0 (+2.2 pts vs 5.4) — reste pertinent pour le travail terminal-first

### GPT-5.4
- **Modèle unifié** : absorbe les capacités Codex + reasoning + computer use
- **Computer use natif** : contrôle autonome du bureau (navigateur, formulaires, apps), premier modèle à dépasser le baseline humain (75% vs 72.4%)
- **Tool Search** : sélection intelligente d'outils → 47% de réduction de tokens
- **Reasoning effort configurable** : de `none` à `xhigh` par requête
- **Midtask steering en ChatGPT** : plan de réflexion visible, ajustable en cours de génération
- Token-efficient : 47% de tokens en moins sur les tâches complexes (peut compenser le prix unitaire plus élevé)
- 33% moins d'hallucinations factuelles vs 5.2

### GPT-5.4 mini
- **30% du quota Copilot** de GPT-5.4 → ~3× moins cher en usage Copilot
- Conçu pour les **subagents** : Codex peut déléguer les sous-tâches au mini
- 2× plus rapide que GPT-5 mini
- Supporte : text, image, tool use, function calling, web search, computer use, skills
- ~94% de la qualité de GPT-5.4 pour les tâches simples
- Idéal pour : itérations rapides, navigation codebase, front-end, debugging loops

## Arbre de décision

```
Quel est ton besoin principal ?
│
├─ Tâche complexe, multi-fichiers, raisonnement profond
│  └─ GPT-5.4 (ou Claude Opus 4.6 pour SWE-bench max)
│
├─ Workflow terminal-first, volume élevé, coût maîtrisé
│  └─ GPT-5.3-Codex (encore pertinent, ~50% moins cher en input)
│
├─ Tâches rapides, subagents, itérations légères
│  └─ GPT-5.4 mini (~30% du coût, 2× plus rapide)
│
├─ Computer use / automatisation desktop
│  └─ GPT-5.4 (seul à 75% OSWorld)
│
└─ Budget serré / usage quotidien sans premium
   └─ GPT-4.1 (inclus sans consommation dans Copilot Pro)
```

## Contexte dans GitHub Copilot

Dans Copilot, ces modèles sont accessibles via le model picker mais consomment des **premium requests** avec des **multiplicateurs variables**. GPT-5.4 mini est particulièrement intéressant en Copilot car il ne consomme que 30% du quota de GPT-5.4, permettant de tripler le nombre d'interactions premium.

Les modèles inclus sans consommation (GPT-4.1, GPT-4o, GPT-5 mini) restent les plus économiques pour le quotidien.

## Trajectoire produit

- GPT-5.2 Thinking sera **retiré le 5 juin 2026** — migration nécessaire
- GPT-5.3-Codex reste actif mais OpenAI positionne GPT-5.4 comme successeur de toute la lignée
- La tendance est à la **consolidation** : un modèle frontier unifié (5.4) + des variantes mini/nano pour le volume
- Les capacités Codex spécialisées sont désormais absorbées dans le modèle principal

## Sources

- [Introducing GPT-5.4 — OpenAI](https://openai.com/index/introducing-gpt-5-4/)
- [Introducing GPT-5.4 mini and nano — OpenAI](https://openai.com/index/introducing-gpt-5-4-mini-and-nano/)
- [Models — Codex (OpenAI Developers)](https://developers.openai.com/codex/models)
- [GPT-5.4 vs GPT-5.3 Codex — NxCode](https://www.nxcode.io/resources/news/gpt-5-4-vs-gpt-5-3-codex-upgrade-comparison-2026)
- [GPT-5.4 Complete Guide — NxCode](https://www.nxcode.io/resources/news/gpt-5-4-complete-guide-features-pricing-models-2026)
- [OpenAI GPT-5 Model Guide — NxCode](https://www.nxcode.io/resources/news/openai-gpt-5-model-guide-which-to-use-2026)
- [GPT-5.4 Pricing — TokenCost](https://tokencost.app/blog/openai-gpt-5-4-pricing-benchmarks-review)
- [GPT-5.4 mini — OpenRouter](https://openrouter.ai/openai/gpt-5.4-mini)
