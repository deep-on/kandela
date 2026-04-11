<p align="center">
  <img src="https://kandela.ai/logo.png" width="80" alt="Kandela">
</p>

<h1 align="center">Kandela</h1>

<p align="center">
  <b>Mémoire sémantique persistante pour les agents de codage IA.</b><br>
  Votre IA retient les décisions, les échecs et le contexte — entre les sessions, projets et outils.
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.ko.md">한국어</a> | <a href="README.ja.md">日本語</a> | <a href="README.de.md">Deutsch</a> | <b>Français</b> | <a href="README.es.md">Español</a> | <a href="README.pt.md">Português</a> | <a href="README.zh.md">中文</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-AGPL--3.0-blue" alt="License">
  <img src="https://img.shields.io/badge/python-3.11+-green" alt="Python">
  <img src="https://img.shields.io/badge/docker-compose-blue" alt="Docker">
  <img src="https://img.shields.io/badge/MCP_tools-13-brightgreen" alt="MCP Tools">
  <img src="https://img.shields.io/badge/ChromaDB-vector_store-orange" alt="ChromaDB">
  <img src="https://img.shields.io/badge/embeddings-50+_languages-purple" alt="Multilingual">
</p>

<p align="center">
  <a href="https://youtu.be/XwkK06midNg?si=QhOWjoNeW_57B_FZ">▶ Voir la démo sur YouTube</a>
</p>

---

## Problème

La mémoire intégrée de Google, Claude et ChatGPT ne fonctionne que dans leur propre écosystème.
Si vous passez à Claude, les souvenirs de GPT disparaissent et ne peuvent pas être partagés avec les membres de l'équipe.

Kandela résout ce problème avec **la compatibilité cross-fournisseur + une transparence totale + un pipeline RAG à 4 niveaux**.

## Fonctionnalités principales

- **13 outils MCP** — Sauvegarde, recherche, suppression, modification, rappel automatique, recherche à la demande, Inbox, gestion de projets
- **Recherche hybride** — Sémantique + recherche par mots-clés BM25 (fusion RRF)
- **Moteur d'importance** — Score automatique 1–10 + 18 règles de taggage d'infrastructure
- **Lazy Retrieval** — Mode brief (~260 tok) + context_search à la demande
- **Continuité de session** — Détection des changements d'environnement + inclusion automatique de la mémoire d'infrastructure
- **Prompt Guard** — Prévention des mauvaises décisions basées sur des souvenirs obsolètes
- **Circuit Breaker** — Détection des schémas d'échecs répétitifs + sauvegarde automatique de Gotcha
- **Tableau de bord web** — Consultation de mémoire par projet, recherche, statistiques, surveillance des performances
- **Installation en un clic** — Installation automatique des Hooks + commandes slash
- **Embeddings multilingues** — paraphrase-multilingual-MiniLM-L12-v2 (50+ langues)

## Benchmark

Scénario de pipeline de données médicales HIPAA (8 sessions, 14 pièges décisionnels) — Kandela ACTIVÉ vs DÉSACTIVÉ :

| | Kandela ACTIVÉ | Kandela DÉSACTIVÉ | Écart |
|---|:-:|:-:|:-:|
| **Évitement des pièges décisionnels** | **100%** | 11,9% | **+88,1pp** |
| **Temps de travail** | 77,9 min | 86,6 min | **-10,1%** |
| **Code généré** | 2 152 lignes | 3 441 lignes | **-37,5%** |
| **Fichiers générés** | 40 | 62 | **-35,5%** |

> 3 exécutions (seeds=42,123,456), claude-sonnet-4-6, Groq Llama 3.3 70B (Operator).

### Enseignements clés

- **Les décisions absentes du code sont déterminantes** : Noms d'auditeurs, incidents OOM, historique de perte de données — informations invisibles à la lecture du code
- **Les décisions basées sur le code sont auto-défendues par le LLM** : Dans les scénarios centrés sur le code (InfraBot), le LLM a atteint un taux d'évitement de 95,2% même sans Kandela
- **Élimine le travail en double** : Sans Kandela, l'IA réimplémente des approches précédemment rejetées → 37,5% de code gaspillé

### Analyse de données à grande échelle

830K conversations réelles (WildChat-1M) et 1M conversations LLM (LMSYS-Chat-1M) :

| Découverte | Métrique | Implication |
|---------|:------:|-------------|
| La référence aux instructions initiales chute de 18%p après 10 tours | 66%→48% | Les LLM généraux commencent à oublier le contexte précoce |
| Les petits LLM chutent jusqu'à 66%p | chatglm 84%→19% | L'augmentation de mémoire est critique pour les modèles locaux/petits |
| 14% des corrections utilisateur sont « a oublié la conversation précédente » | 82K conversations de codage | Problème fondamental que Kandela résout |
| Kandela maintient le contexte après 49+ tours | Benchmark interne | Le Brief Recall empêche la dégradation |

## Architecture

```
Client (Claude Code / Desktop / Cursor)
  │
  ├─ MCP Protocol ──→ Serveur Kandela
  │                      │
  │                  MemoryStore (ChromaDB)
  │                      ├── Recherche hybride (Semantic + BM25 RRF)
  │                      ├── Moteur d'importance (18 règles)
  │                      ├── Continuité de session
  │                      └── Prompt Guard / Circuit Breaker
  │
  └─ Tableau de bord web ──→ REST API + Navigateur de mémoire
```

### Types de mémoire

| Type | Description | Auto/Manuel |
|------|-------------|:-----------:|
| `fact` | Préférences, stack technique, informations d'environnement | Les deux |
| `decision` | Décisions de conception, compromis, justifications | Manuel |
| `summary` | Résumés de session en fin de conversation | Auto |
| `snippet` | Modèles de code fréquemment utilisés, configurations | Manuel |

### Pipeline de recherche

```
Requête → Semantic (MiniLM-L12) ─┐
                                  ├─→ Fusion RRF → Reranking d'importance → Diversité MMR → Résultats
Requête → BM25 (Korean NLP) ─────┘
```

## Démarrage

### Service hébergé (Recommandé)

Aucune installation nécessaire. Inscrivez-vous et commencez en 2 minutes.

```bash
# 1. Inscription sur https://api.kandela.ai/dashboard
# 2. Obtenez votre clé API
# 3. Connectez Claude Code :
claude mcp add memory --transport http https://api.kandela.ai/mcp \
  --header "Authorization: Bearer mcp_YOUR_KEY"
curl -sf https://api.kandela.ai/api/install | bash
```

→ [Guide de démarrage](https://kandela.ai/getting-started)

### Auto-hébergé

Exécutez votre propre instance Kandela. Mode mono-utilisateur, contrôle total de vos données.

```bash
git clone https://github.com/deep-on/kandela-selfhost.git && cd kandela-selfhost/docker
docker compose up -d
# → http://localhost:8321/dashboard
```

→ [Dépôt Self-Host](https://github.com/deep-on/kandela-selfhost)

## Hébergé vs Auto-hébergé

| | Hébergé (api.kandela.ai) | Auto-hébergé |
|---|:-:|:-:|
| Configuration | 2 min | 5 min |
| Multi-utilisateur | ✅ | Mono-utilisateur |
| Bot Telegram | ✅ | — |
| Commandes à distance | ✅ | — |
| Heatmap d'activité | ✅ | — |
| Fonctionnalités par palier (Pro/Max) | ✅ | Toutes les fonctionnalités |
| Emplacement des données | Cloud | Votre serveur |
| Maintenance | Gérée | Auto-gérée |

## Stack technique

- **Python 3.11+**, FastMCP (mcp[cli])
- **ChromaDB** — Base de données vectorielle (persistante)
- **sentence-transformers** — Embeddings locaux (paraphrase-multilingual-MiniLM-L12-v2)
- **Pydantic v2** — Validation des entrées
- **Docker** — Déploiement

## Liens

- **Page d'accueil** : [kandela.ai](https://kandela.ai)
- **Dépôt Self-Host** : [deep-on/kandela-selfhost](https://github.com/deep-on/kandela-selfhost)
- **Politique de confidentialité** : [kandela.ai/privacy](https://kandela.ai/privacy)
- **Conditions d'utilisation** : [kandela.ai/terms](https://kandela.ai/terms)

## Licence

- **Serveur** : [AGPL-3.0](LICENSE) — Copyright (c) 2025-2026 Deep-ON Inc.
- **Fichiers côté client** (hooks, commandes slash) : [MIT](LICENSE-CLIENT)

## Avertissement

Ce logiciel est fourni « en l'état » sans garantie d'aucune sorte.
Les utilisateurs sont responsables de la sauvegarde de leurs propres données.
