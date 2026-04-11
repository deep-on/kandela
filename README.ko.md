<p align="center">
  <img src="https://kandela.ai/logo.png" width="80" alt="Kandela">
</p>

<h1 align="center">Kandela</h1>

<p align="center">
  <b>AI 코딩 에이전트를 위한 영구 시맨틱 메모리.</b><br>
  AI가 결정, 실패, 컨텍스트를 기억합니다 — 세션, 프로젝트, 도구를 넘어서.
</p>

<p align="center">
  <a href="README.md">English</a> | <b>한국어</b> | <a href="README.ja.md">日本語</a> | <a href="README.de.md">Deutsch</a> | <a href="README.fr.md">Français</a> | <a href="README.es.md">Español</a> | <a href="README.pt.md">Português</a> | <a href="README.zh.md">中文</a>
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
  <a href="https://youtu.be/KlsbUJmQqIA?si=Jh4_Nc-OR-UBvUQj">▶ YouTube에서 데모 보기</a>
</p>

---

## 문제

Google · Claude · ChatGPT의 자체 메모리는 자사 생태계에서만 작동합니다.
Claude로 바꾸면 GPT 기억이 사라지고, 팀원과 공유할 수 없습니다.

Kandela는 **크로스 프로바이더 + 완전한 투명성 + 4중 RAG 파이프라인**으로 이 문제를 해결합니다.

## 주요 기능

- **13개 MCP 도구** — 저장, 검색, 삭제, 수정, 자동 회상, on-demand 검색, Inbox, 프로젝트 관리
- **하이브리드 검색** — 시맨틱 + BM25 키워드 검색 (RRF 퓨전)
- **중요도 엔진** — 1~10 자동 점수 + 18개 규칙 기반 인프라 태깅
- **지연 검색** — brief 모드 (~260 tok) + context_search on-demand
- **세션 연속성** — 환경 변화 감지 + 인프라 메모리 자동 포함
- **프롬프트 가드** — 오래된 기억 기반의 잘못된 결정 방지
- **서킷 브레이커** — 반복 실패 패턴 감지 + 자동 Gotcha 저장
- **웹 대시보드** — 프로젝트별 메모리 조회, 검색, 통계, 성능 모니터링
- **원클릭 설치** — Hooks + 슬래시 명령 자동 설치
- **다국어 임베딩** — paraphrase-multilingual-MiniLM-L12-v2 (50개 이상 언어)

## 벤치마크

HIPAA 의료 데이터 파이프라인 시나리오 (8세션, 14개 의사결정 트랩) — Kandela ON vs OFF:

| | Kandela ON | Kandela OFF | 차이 |
|---|:-:|:-:|:-:|
| **의사결정 트랩 회피** | **100%** | 11.9% | **+88.1pp** |
| **작업 시간** | 77.9분 | 86.6분 | **-10.1%** |
| **생성 코드** | 2,152줄 | 3,441줄 | **-37.5%** |
| **생성 파일** | 40개 | 62개 | **-35.5%** |

> 3회 실행 (seeds=42,123,456), claude-sonnet-4-6, Groq Llama 3.3 70B (Operator).

### 핵심 인사이트

- **코드에 없는 결정이 중요합니다**: 감사관 이름, OOM 사고, 데이터 손실 이력 — 코드 읽기로는 보이지 않는 정보
- **코드 기반 결정은 LLM이 스스로 방어합니다**: 코드 중심 시나리오 (InfraBot)에서 LLM은 Kandela 없이도 95.2% 회피율 달성
- **재작업을 제거합니다**: Kandela 없이는, AI가 이전에 거부된 접근 방식을 다시 구현 → 37.5% 코드 낭비

### 대규모 데이터 분석

830K개 실제 대화 (WildChat-1M)와 1M개 LLM 대화 (LMSYS-Chat-1M):

| 발견 | 지표 | 시사점 |
|---------|:------:|-------------|
| 10턴 후 초기 지시 참조가 18%p 하락 | 66%→48% | 일반 LLM은 이른 컨텍스트를 잊기 시작 |
| 소형 LLM은 최대 66%p 하락 | chatglm 84%→19% | 로컬/소형 모델에 메모리 증강 필수 |
| 사용자 수정의 14%가 "이전 대화를 잊었다" | 82K개 코딩 대화 | Kandela가 해결하는 핵심 문제 |
| Kandela는 49턴 이후에도 컨텍스트 유지 | 내부 벤치마크 | Brief Recall이 감쇠 방지 |

## 아키텍처

```
클라이언트 (Claude Code / Desktop / Cursor)
  │
  ├─ MCP Protocol ──→ Kandela 서버
  │                      │
  │                  MemoryStore (ChromaDB)
  │                      ├── 하이브리드 검색 (Semantic + BM25 RRF)
  │                      ├── 중요도 엔진 (18개 규칙)
  │                      ├── 세션 연속성
  │                      └── 프롬프트 가드 / 서킷 브레이커
  │
  └─ 웹 대시보드 ──→ REST API + 메모리 브라우저
```

### 메모리 유형

| 유형 | 설명 | 자동/수동 |
|------|-------------|:-----------:|
| `fact` | 선호도, 기술 스택, 환경 정보 | 둘 다 |
| `decision` | 설계 결정, 트레이드오프, 근거 | 수동 |
| `summary` | 대화 종료 시 세션 요약 | 자동 |
| `snippet` | 자주 사용하는 코드 패턴, 설정 | 수동 |

### 검색 파이프라인

```
쿼리 → Semantic (MiniLM-L12) ─┐
                               ├─→ RRF 퓨전 → 중요도 재순위 → MMR 다양성 → 결과
쿼리 → BM25 (Korean NLP) ─────┘
```

## 시작하기

### 호스팅 서비스 (권장)

설치 불필요. 가입 후 2분 만에 시작하세요.

```bash
# 1. https://api.kandela.ai/dashboard 에서 가입
# 2. API 키 발급
# 3. Claude Code 연결:
claude mcp add memory --transport http https://api.kandela.ai/mcp \
  --header "Authorization: Bearer mcp_YOUR_KEY"
curl -sf https://api.kandela.ai/api/install | bash
```

→ [시작 가이드](https://kandela.ai/getting-started)

### 셀프호스팅

자체 Kandela 인스턴스를 운영하세요. 싱글유저 모드, 데이터 완전 제어.

```bash
git clone https://github.com/deep-on/kandela-selfhost.git && cd kandela-selfhost/docker
docker compose up -d
# → http://localhost:8321/dashboard
```

→ [셀프호스팅 저장소](https://github.com/deep-on/kandela-selfhost)

## 호스팅 vs 셀프호스팅

| | 호스팅 (api.kandela.ai) | 셀프호스팅 |
|---|:-:|:-:|
| 설치 | 2분 | 5분 |
| 다중 사용자 | ✅ | 싱글유저 |
| 텔레그램 봇 | ✅ | — |
| 원격 명령 | ✅ | — |
| 활동 히트맵 | ✅ | — |
| 티어 기능 (Pro/Max) | ✅ | 모든 기능 |
| 데이터 위치 | 클라우드 | 자체 서버 |
| 유지보수 | 관리형 | 자체 관리 |

## 기술 스택

- **Python 3.11+**, FastMCP (mcp[cli])
- **ChromaDB** — 벡터 데이터베이스 (영구 저장)
- **sentence-transformers** — 로컬 임베딩 (paraphrase-multilingual-MiniLM-L12-v2)
- **Pydantic v2** — 입력 검증
- **Docker** — 배포

## 링크

- **홈페이지**: [kandela.ai](https://kandela.ai)
- **셀프호스팅 저장소**: [deep-on/kandela-selfhost](https://github.com/deep-on/kandela-selfhost)
- **개인정보처리방침**: [kandela.ai/privacy](https://kandela.ai/privacy)
- **이용약관**: [kandela.ai/terms](https://kandela.ai/terms)

## 라이선스

- **서버**: [AGPL-3.0](LICENSE) — Copyright (c) 2025-2026 Deep-ON Inc.
- **클라이언트 파일** (hooks, 슬래시 명령): [MIT](LICENSE-CLIENT)

## 면책 조항

이 소프트웨어는 어떠한 종류의 보증 없이 "있는 그대로" 제공됩니다.
사용자는 자체 데이터 백업에 대한 책임이 있습니다.
