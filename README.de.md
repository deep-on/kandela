<p align="center">
  <img src="https://kandela.ai/logo.png" width="80" alt="Kandela">
</p>

<h1 align="center">Kandela</h1>

<p align="center">
  <b>Persistenter semantischer Speicher für KI-Coding-Agenten.</b><br>
  Ihre KI merkt sich Entscheidungen, Fehler und Kontext — über Sitzungen, Projekte und Tools hinweg.
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.ko.md">한국어</a> | <a href="README.ja.md">日本語</a> | <b>Deutsch</b> | <a href="README.fr.md">Français</a> | <a href="README.es.md">Español</a> | <a href="README.pt.md">Português</a> | <a href="README.zh.md">中文</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-0.6.0-blue" alt="Version">
  <img src="https://img.shields.io/badge/license-AGPL--3.0-blue" alt="License">
  <img src="https://img.shields.io/badge/python-3.11+-green" alt="Python">
  <img src="https://img.shields.io/badge/docker-compose-blue" alt="Docker">
  <img src="https://img.shields.io/badge/MCP_tools-13-brightgreen" alt="MCP Tools">
  <img src="https://img.shields.io/badge/ChromaDB-vector_store-orange" alt="ChromaDB">
  <img src="https://img.shields.io/badge/embeddings-50+_languages-purple" alt="Multilingual">
</p>

<p align="center">
  <a href="https://youtu.be/XwkK06midNg?si=QhOWjoNeW_57B_FZ">▶ Demo auf YouTube ansehen</a>
</p>

---

## Problem

Der eingebaute Speicher von Google, Claude und ChatGPT funktioniert nur innerhalb des jeweiligen Ökosystems.
Wenn Sie zu Claude wechseln, gehen die GPT-Erinnerungen verloren und können nicht mit Teammitgliedern geteilt werden.

Kandela löst dieses Problem mit **Cross-Provider-Kompatibilität + vollständiger Transparenz + 4-facher RAG-Pipeline**.

## Hauptfunktionen

- **13 MCP-Tools** — Speichern, Suchen, Löschen, Bearbeiten, automatischer Abruf, On-Demand-Suche, Inbox, Projektverwaltung
- **Hybridsuche** — Semantische + BM25-Schlüsselwortsuche (RRF-Fusion)
- **Wichtigkeits-Engine** — Automatische Bewertung 1–10 + 18 regelbasierte Infrastruktur-Tags
- **Lazy Retrieval** — Brief-Modus (~260 Tok) + context_search on-demand
- **Sitzungskontinuität** — Erkennung von Umgebungsänderungen + automatische Einbeziehung von Infrastruktur-Erinnerungen
- **Prompt Guard** — Verhindert fehlerhafte Entscheidungen basierend auf veralteten Erinnerungen
- **Circuit Breaker** — Erkennung wiederkehrender Fehlermuster + automatische Gotcha-Speicherung
- **Web-Dashboard** — Projektbezogene Speicheransicht, Suche, Statistiken, Leistungsüberwachung
- **Ein-Klick-Installation** — Automatische Installation von Hooks + Slash-Befehlen
- **Mehrsprachige Embeddings** — paraphrase-multilingual-MiniLM-L12-v2 (50+ Sprachen)

## Benchmark

HIPAA-Szenario für medizinische Datenpipeline (8 Sitzungen, 14 Entscheidungsfallen) — Kandela EIN vs AUS:

| | Kandela EIN | Kandela AUS | Differenz |
|---|:-:|:-:|:-:|
| **Entscheidungsfallen-Vermeidung** | **100%** | 11,9% | **+88,1pp** |
| **Arbeitszeit** | 77,9 Min | 86,6 Min | **-10,1%** |
| **Generierter Code** | 2.152 Zeilen | 3.441 Zeilen | **-37,5%** |
| **Generierte Dateien** | 40 | 62 | **-35,5%** |

> 3 Durchläufe (Seeds=42,123,456), claude-sonnet-4-6, Groq Llama 3.3 70B (Operator).

### Wichtige Erkenntnisse

- **Entscheidungen, die nicht im Code stehen, sind entscheidend**: Prüfernamen, OOM-Vorfälle, Datenverlust-Historie — Informationen, die beim Code-Lesen unsichtbar sind
- **Codebasierte Entscheidungen werden vom LLM selbst verteidigt**: In code-zentrierten Szenarien (InfraBot) erreichte das LLM auch ohne Kandela eine Vermeidungsrate von 95,2%
- **Eliminiert Nacharbeit**: Ohne Kandela implementiert die KI zuvor abgelehnte Ansätze erneut → 37,5% Code-Verschwendung

### Großdatenanalyse

830K reale Gespräche (WildChat-1M) und 1M LLM-Gespräche (LMSYS-Chat-1M):

| Erkenntnis | Metrik | Implikation |
|---------|:------:|-------------|
| Referenz auf anfängliche Anweisungen sinkt nach 10 Runden um 18%p | 66%→48% | Allgemeine LLMs beginnen frühen Kontext zu vergessen |
| Kleine LLMs fallen um bis zu 66%p | chatglm 84%→19% | Speichererweiterung für lokale/kleine Modelle unerlässlich |
| 14% der Nutzerkorrekturen sind „vorheriges Gespräch vergessen" | 82K Coding-Gespräche | Kernproblem, das Kandela löst |
| Kandela hält den Kontext nach 49+ Runden aufrecht | Interner Benchmark | Brief Recall verhindert Verfall |

## Architektur

```
Client (Claude Code / Desktop / Cursor)
  │
  ├─ MCP Protocol ──→ Kandela-Server
  │                      │
  │                  MemoryStore (ChromaDB)
  │                      ├── Hybridsuche (Semantic + BM25 RRF)
  │                      ├── Wichtigkeits-Engine (18 Regeln)
  │                      ├── Sitzungskontinuität
  │                      └── Prompt Guard / Circuit Breaker
  │
  └─ Web-Dashboard ──→ REST API + Speicher-Browser
```

### Speichertypen

| Typ | Beschreibung | Auto/Manuell |
|------|-------------|:-----------:|
| `fact` | Präferenzen, Tech-Stack, Umgebungsinfos | Beides |
| `decision` | Designentscheidungen, Kompromisse, Begründungen | Manuell |
| `summary` | Sitzungszusammenfassungen am Gesprächsende | Auto |
| `snippet` | Häufig genutzte Codemuster, Konfigurationen | Manuell |

### Such-Pipeline

```
Abfrage → Semantic (MiniLM-L12) ─┐
                                  ├─→ RRF-Fusion → Wichtigkeits-Reranking → MMR-Diversität → Ergebnisse
Abfrage → BM25 (Korean NLP) ─────┘
```

## Erste Schritte

### Gehosteter Dienst (Empfohlen)

Keine Installation erforderlich. Registrieren Sie sich und starten Sie in 2 Minuten.

```bash
# 1. Registrierung unter https://api.kandela.ai/dashboard
# 2. API-Schlüssel erhalten
# 3. Claude Code verbinden:
claude mcp add memory --transport http https://api.kandela.ai/mcp \
  --header "Authorization: Bearer mcp_YOUR_KEY"
curl -sf https://api.kandela.ai/api/install | bash
```

→ [Erste-Schritte-Anleitung](https://kandela.ai/getting-started)

### Selbst gehostet

Betreiben Sie Ihre eigene Kandela-Instanz. Einzelbenutzer-Modus, volle Kontrolle über Ihre Daten.

```bash
git clone https://github.com/deep-on/kandela-selfhost.git && cd kandela-selfhost/docker
docker compose up -d
# → http://localhost:8321/dashboard
```

→ [Self-Host-Repository](https://github.com/deep-on/kandela-selfhost)

## Gehostet vs Selbst gehostet

| | Gehostet (api.kandela.ai) | Selbst gehostet |
|---|:-:|:-:|
| Einrichtung | 2 Min | 5 Min |
| Mehrbenutzer | ✅ | Einzelbenutzer |
| Telegram-Bot | ✅ | — |
| Fernbefehle | ✅ | — |
| Aktivitäts-Heatmap | ✅ | — |
| Tier-Funktionen (Pro/Max) | ✅ | Alle Funktionen |
| Datenspeicherort | Cloud | Eigener Server |
| Wartung | Verwaltet | Selbstverwaltet |

## Technologie-Stack

- **Python 3.11+**, FastMCP (mcp[cli])
- **ChromaDB** — Vektordatenbank (persistent)
- **sentence-transformers** — Lokale Embeddings (paraphrase-multilingual-MiniLM-L12-v2)
- **Pydantic v2** — Eingabevalidierung
- **Docker** — Bereitstellung

## Links

- **Homepage**: [kandela.ai](https://kandela.ai)
- **Self-Host-Repository**: [deep-on/kandela-selfhost](https://github.com/deep-on/kandela-selfhost)
- **Datenschutzrichtlinie**: [kandela.ai/privacy](https://kandela.ai/privacy)
- **Nutzungsbedingungen**: [kandela.ai/terms](https://kandela.ai/terms)

## Lizenz

- **Server**: [AGPL-3.0](LICENSE) — Copyright (c) 2025-2026 Deep-ON Inc.
- **Client-seitige Dateien** (Hooks, Slash-Befehle): [MIT](LICENSE-CLIENT)

## Haftungsausschluss

Diese Software wird ohne jegliche Gewährleistung „wie besehen" bereitgestellt.
Benutzer sind für die Sicherung ihrer eigenen Daten verantwortlich.
