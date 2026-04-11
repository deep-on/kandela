<p align="center">
  <img src="https://kandela.ai/logo.png" width="80" alt="Kandela">
</p>

<h1 align="center">Kandela</h1>

<p align="center">
  <b>Memória semântica persistente para agentes de codificação com IA.</b><br>
  Sua IA lembra decisões, falhas e contexto — entre sessões, projetos e ferramentas.
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.ko.md">한국어</a> | <a href="README.ja.md">日本語</a> | <a href="README.de.md">Deutsch</a> | <a href="README.fr.md">Français</a> | <a href="README.es.md">Español</a> | <b>Português</b> | <a href="README.zh.md">中文</a>
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
  <a href="https://youtu.be/XwkK06midNg?si=QhOWjoNeW_57B_FZ">▶ Assistir demo no YouTube</a>
</p>

---

## Problema

A memória integrada do Google, Claude e ChatGPT funciona apenas dentro do seu próprio ecossistema.
Se você muda para o Claude, as memórias do GPT desaparecem e não podem ser compartilhadas com membros da equipe.

Kandela resolve este problema com **compatibilidade cross-provedor + transparência total + pipeline RAG de 4 camadas**.

## Funcionalidades principais

- **13 ferramentas MCP** — Salvar, buscar, excluir, editar, recordação automática, busca on-demand, Inbox, gestão de projetos
- **Busca híbrida** — Semântica + busca por palavras-chave BM25 (fusão RRF)
- **Motor de importância** — Pontuação automática 1–10 + 18 regras de tagueamento de infraestrutura
- **Lazy Retrieval** — Modo brief (~260 tok) + context_search on-demand
- **Continuidade de sessão** — Detecção de mudanças de ambiente + inclusão automática de memória de infraestrutura
- **Prompt Guard** — Prevenção de decisões erradas baseadas em memórias obsoletas
- **Circuit Breaker** — Detecção de padrões de falha repetitivos + armazenamento automático de Gotcha
- **Painel web** — Consulta de memória por projeto, busca, estatísticas, monitoramento de desempenho
- **Instalação com um clique** — Instalação automática de Hooks + comandos slash
- **Embeddings multilíngues** — paraphrase-multilingual-MiniLM-L12-v2 (50+ idiomas)

## Benchmark

Cenário de pipeline de dados médicos HIPAA (8 sessões, 14 armadilhas de decisão) — Kandela ATIVADO vs DESATIVADO:

| | Kandela ATIVADO | Kandela DESATIVADO | Diferença |
|---|:-:|:-:|:-:|
| **Evasão de armadilhas de decisão** | **100%** | 11,9% | **+88,1pp** |
| **Tempo de trabalho** | 77,9 min | 86,6 min | **-10,1%** |
| **Código gerado** | 2.152 linhas | 3.441 linhas | **-37,5%** |
| **Arquivos gerados** | 40 | 62 | **-35,5%** |

> 3 execuções (seeds=42,123,456), claude-sonnet-4-6, Groq Llama 3.3 70B (Operator).

### Descobertas principais

- **Decisões que não estão no código são o que importa**: Nomes de auditores, incidentes OOM, histórico de perda de dados — informações invisíveis na leitura do código
- **Decisões baseadas em código são autodefendidas pelo LLM**: Em cenários centrados em código (InfraBot), o LLM alcançou 95,2% de evasão mesmo sem Kandela
- **Elimina retrabalho**: Sem Kandela, a IA reimplementa abordagens previamente rejeitadas → 37,5% de código desperdiçado

### Análise de dados em grande escala

830K conversas reais (WildChat-1M) e 1M conversas LLM (LMSYS-Chat-1M):

| Descoberta | Métrica | Implicação |
|---------|:------:|-------------|
| A referência às instruções iniciais cai 18%p após 10 turnos | 66%→48% | LLMs gerais começam a esquecer o contexto inicial |
| LLMs pequenos caem até 66%p | chatglm 84%→19% | Aumento de memória é crítico para modelos locais/pequenos |
| 14% das correções do usuário são "esqueceu a conversa anterior" | 82K conversas de codificação | Problema central que Kandela resolve |
| Kandela mantém o contexto após 49+ turnos | Benchmark interno | Brief Recall previne a degradação |

## Arquitetura

```
Cliente (Claude Code / Desktop / Cursor)
  │
  ├─ MCP Protocol ──→ Servidor Kandela
  │                      │
  │                  MemoryStore (ChromaDB)
  │                      ├── Busca híbrida (Semantic + BM25 RRF)
  │                      ├── Motor de importância (18 regras)
  │                      ├── Continuidade de sessão
  │                      └── Prompt Guard / Circuit Breaker
  │
  └─ Painel web ──→ REST API + Navegador de memória
```

### Tipos de memória

| Tipo | Descrição | Auto/Manual |
|------|-------------|:-----------:|
| `fact` | Preferências, stack tecnológico, informações de ambiente | Ambos |
| `decision` | Decisões de design, trade-offs, justificativas | Manual |
| `summary` | Resumos de sessão no final da conversa | Auto |
| `snippet` | Padrões de código frequentes, configurações | Manual |

### Pipeline de busca

```
Consulta → Semantic (MiniLM-L12) ─┐
                                   ├─→ Fusão RRF → Reranking de importância → Diversidade MMR → Resultados
Consulta → BM25 (Korean NLP) ─────┘
```

## Começar

### Serviço hospedado (Recomendado)

Sem instalação necessária. Cadastre-se e comece em 2 minutos.

```bash
# 1. Cadastre-se em https://api.kandela.ai/dashboard
# 2. Obtenha sua chave API
# 3. Conecte o Claude Code:
claude mcp add memory --transport http https://api.kandela.ai/mcp \
  --header "Authorization: Bearer mcp_YOUR_KEY"
curl -sf https://api.kandela.ai/api/install | bash
```

→ [Guia de início](https://kandela.ai/getting-started)

### Auto-hospedado

Execute sua própria instância Kandela. Modo usuário único, controle total sobre seus dados.

```bash
git clone https://github.com/deep-on/kandela-selfhost.git && cd kandela-selfhost/docker
docker compose up -d
# → http://localhost:8321/dashboard
```

→ [Repositório Self-Host](https://github.com/deep-on/kandela-selfhost)

## Hospedado vs Auto-hospedado

| | Hospedado (api.kandela.ai) | Auto-hospedado |
|---|:-:|:-:|
| Configuração | 2 min | 5 min |
| Multiusuário | ✅ | Usuário único |
| Bot do Telegram | ✅ | — |
| Comandos remotos | ✅ | — |
| Mapa de calor de atividade | ✅ | — |
| Funcionalidades por nível (Pro/Max) | ✅ | Todas as funcionalidades |
| Localização dos dados | Nuvem | Seu servidor |
| Manutenção | Gerenciada | Autogerenciada |

## Stack tecnológico

- **Python 3.11+**, FastMCP (mcp[cli])
- **ChromaDB** — Banco de dados vetorial (persistente)
- **sentence-transformers** — Embeddings locais (paraphrase-multilingual-MiniLM-L12-v2)
- **Pydantic v2** — Validação de entrada
- **Docker** — Implantação

## Links

- **Página inicial**: [kandela.ai](https://kandela.ai)
- **Repositório Self-Host**: [deep-on/kandela-selfhost](https://github.com/deep-on/kandela-selfhost)
- **Política de privacidade**: [kandela.ai/privacy](https://kandela.ai/privacy)
- **Termos de serviço**: [kandela.ai/terms](https://kandela.ai/terms)

## Licença

- **Servidor**: [AGPL-3.0](LICENSE) — Copyright (c) 2025-2026 Deep-ON Inc.
- **Arquivos do lado cliente** (hooks, comandos slash): [MIT](LICENSE-CLIENT)

## Aviso legal

Este software é fornecido "como está" sem garantia de qualquer tipo.
Os usuários são responsáveis pelo backup de seus próprios dados.
