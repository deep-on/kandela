<p align="center">
  <img src="https://kandela.ai/logo.png" width="80" alt="Kandela">
</p>

<h1 align="center">Kandela</h1>

<p align="center">
  <b>Memoria semántica persistente para agentes de codificación con IA.</b><br>
  Tu IA recuerda decisiones, fallos y contexto — entre sesiones, proyectos y herramientas.
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.ko.md">한국어</a> | <a href="README.ja.md">日本語</a> | <a href="README.de.md">Deutsch</a> | <a href="README.fr.md">Français</a> | <b>Español</b> | <a href="README.pt.md">Português</a> | <a href="README.zh.md">中文</a>
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
  <a href="https://youtu.be/XwkK06midNg?si=QhOWjoNeW_57B_FZ">▶ Ver demo en YouTube</a>
</p>

---

## Problema

La memoria integrada de Google, Claude y ChatGPT solo funciona dentro de su propio ecosistema.
Si cambias a Claude, los recuerdos de GPT desaparecen y no se pueden compartir con los miembros del equipo.

Kandela resuelve este problema con **compatibilidad cross-proveedor + transparencia total + pipeline RAG de 4 niveles**.

## Funcionalidades principales

- **13 herramientas MCP** — Guardar, buscar, eliminar, editar, recuerdo automático, búsqueda on-demand, Inbox, gestión de proyectos
- **Búsqueda híbrida** — Semántica + búsqueda por palabras clave BM25 (fusión RRF)
- **Motor de importancia** — Puntuación automática 1–10 + 18 reglas de etiquetado de infraestructura
- **Lazy Retrieval** — Modo brief (~260 tok) + context_search on-demand
- **Continuidad de sesión** — Detección de cambios de entorno + inclusión automática de memoria de infraestructura
- **Prompt Guard** — Prevención de decisiones erróneas basadas en recuerdos obsoletos
- **Circuit Breaker** — Detección de patrones de fallo repetitivos + almacenamiento automático de Gotcha
- **Panel web** — Consulta de memoria por proyecto, búsqueda, estadísticas, monitoreo de rendimiento
- **Instalación en un clic** — Instalación automática de Hooks + comandos slash
- **Embeddings multilingües** — paraphrase-multilingual-MiniLM-L12-v2 (50+ idiomas)

## Benchmark

Escenario de pipeline de datos médicos HIPAA (8 sesiones, 14 trampas de decisión) — Kandela ACTIVADO vs DESACTIVADO:

| | Kandela ACTIVADO | Kandela DESACTIVADO | Diferencia |
|---|:-:|:-:|:-:|
| **Evasión de trampas de decisión** | **100%** | 11,9% | **+88,1pp** |
| **Tiempo de trabajo** | 77,9 min | 86,6 min | **-10,1%** |
| **Código generado** | 2.152 líneas | 3.441 líneas | **-37,5%** |
| **Archivos generados** | 40 | 62 | **-35,5%** |

> 3 ejecuciones (seeds=42,123,456), claude-sonnet-4-6, Groq Llama 3.3 70B (Operator).

### Hallazgos clave

- **Las decisiones que no están en el código son las que importan**: Nombres de auditores, incidentes OOM, historial de pérdida de datos — información invisible al leer el código
- **Las decisiones basadas en código son autodefendidas por el LLM**: En escenarios centrados en código (InfraBot), el LLM logró un 95,2% de evasión incluso sin Kandela
- **Elimina el retrabajo**: Sin Kandela, la IA reimplementa enfoques previamente rechazados → 37,5% de código desperdiciado

### Análisis de datos a gran escala

830K conversaciones reales (WildChat-1M) y 1M conversaciones LLM (LMSYS-Chat-1M):

| Hallazgo | Métrica | Implicación |
|---------|:------:|-------------|
| La referencia a instrucciones iniciales cae 18%p después de 10 turnos | 66%→48% | Los LLM generales empiezan a olvidar el contexto temprano |
| Los LLM pequeños caen hasta 66%p | chatglm 84%→19% | La mejora de memoria es crítica para modelos locales/pequeños |
| El 14% de las correcciones de usuario son "olvidó la conversación anterior" | 82K conversaciones de codificación | Problema central que Kandela resuelve |
| Kandela mantiene el contexto después de 49+ turnos | Benchmark interno | Brief Recall previene el deterioro |

## Arquitectura

```
Cliente (Claude Code / Desktop / Cursor)
  │
  ├─ MCP Protocol ──→ Servidor Kandela
  │                      │
  │                  MemoryStore (ChromaDB)
  │                      ├── Búsqueda híbrida (Semantic + BM25 RRF)
  │                      ├── Motor de importancia (18 reglas)
  │                      ├── Continuidad de sesión
  │                      └── Prompt Guard / Circuit Breaker
  │
  └─ Panel web ──→ REST API + Navegador de memoria
```

### Tipos de memoria

| Tipo | Descripción | Auto/Manual |
|------|-------------|:-----------:|
| `fact` | Preferencias, stack tecnológico, información del entorno | Ambos |
| `decision` | Decisiones de diseño, compromisos, justificaciones | Manual |
| `summary` | Resúmenes de sesión al final de la conversación | Auto |
| `snippet` | Patrones de código frecuentes, configuraciones | Manual |

### Pipeline de búsqueda

```
Consulta → Semantic (MiniLM-L12) ─┐
                                   ├─→ Fusión RRF → Reranking de importancia → Diversidad MMR → Resultados
Consulta → BM25 (Korean NLP) ─────┘
```

## Comenzar

### Servicio alojado (Recomendado)

Sin instalación necesaria. Regístrate y comienza en 2 minutos.

```bash
# 1. Regístrate en https://api.kandela.ai/dashboard
# 2. Obtén tu clave API
# 3. Conecta Claude Code:
claude mcp add memory --transport http https://api.kandela.ai/mcp \
  --header "Authorization: Bearer mcp_YOUR_KEY"
curl -sf https://api.kandela.ai/api/install | bash
```

→ [Guía de inicio](https://kandela.ai/getting-started)

### Auto-alojado

Ejecuta tu propia instancia de Kandela. Modo usuario único, control total sobre tus datos.

```bash
git clone https://github.com/deep-on/kandela-selfhost.git && cd kandela-selfhost/docker
docker compose up -d
# → http://localhost:8321/dashboard
```

→ [Repositorio Self-Host](https://github.com/deep-on/kandela-selfhost)

## Alojado vs Auto-alojado

| | Alojado (api.kandela.ai) | Auto-alojado |
|---|:-:|:-:|
| Configuración | 2 min | 5 min |
| Multiusuario | ✅ | Usuario único |
| Bot de Telegram | ✅ | — |
| Comandos remotos | ✅ | — |
| Mapa de calor de actividad | ✅ | — |
| Funciones por nivel (Pro/Max) | ✅ | Todas las funciones |
| Ubicación de datos | Nube | Tu servidor |
| Mantenimiento | Gestionado | Autogestionado |

## Stack tecnológico

- **Python 3.11+**, FastMCP (mcp[cli])
- **ChromaDB** — Base de datos vectorial (persistente)
- **sentence-transformers** — Embeddings locales (paraphrase-multilingual-MiniLM-L12-v2)
- **Pydantic v2** — Validación de entradas
- **Docker** — Despliegue

## Enlaces

- **Página principal**: [kandela.ai](https://kandela.ai)
- **Repositorio Self-Host**: [deep-on/kandela-selfhost](https://github.com/deep-on/kandela-selfhost)
- **Política de privacidad**: [kandela.ai/privacy](https://kandela.ai/privacy)
- **Términos de servicio**: [kandela.ai/terms](https://kandela.ai/terms)

## Licencia

- **Servidor**: [AGPL-3.0](LICENSE) — Copyright (c) 2025-2026 Deep-ON Inc.
- **Archivos del lado cliente** (hooks, comandos slash): [MIT](LICENSE-CLIENT)

## Aviso legal

Este software se proporciona "tal cual" sin garantía de ningún tipo.
Los usuarios son responsables de realizar copias de seguridad de sus propios datos.
