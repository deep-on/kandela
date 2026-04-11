<p align="center">
  <img src="https://kandela.ai/logo.png" width="80" alt="Kandela">
</p>

<h1 align="center">Kandela</h1>

<p align="center">
  <b>AIコーディングエージェントのための永続的セマンティックメモリ。</b><br>
  AIが意思決定、失敗、コンテキストを記憶します — セッション、プロジェクト、ツールを超えて。
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.ko.md">한국어</a> | <b>日本語</b> | <a href="README.de.md">Deutsch</a> | <a href="README.fr.md">Français</a> | <a href="README.es.md">Español</a> | <a href="README.pt.md">Português</a> | <a href="README.zh.md">中文</a>
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
  <a href="https://youtu.be/XwkK06midNg?si=QhOWjoNeW_57B_FZ">▶ YouTubeでデモを見る</a>
</p>

---

## 課題

Google・Claude・ChatGPTの内蔵メモリは、各社のエコシステム内でのみ動作します。
Claudeに切り替えるとGPTの記憶は消え、チームメンバーとの共有もできません。

Kandelaは**クロスプロバイダー + 完全な透明性 + 4重RAGパイプライン**でこの問題を解決します。

## 主な機能

- **13個のMCPツール** — 保存、検索、削除、編集、自動想起、オンデマンド検索、Inbox、プロジェクト管理
- **ハイブリッド検索** — セマンティック + BM25キーワード検索（RRFフュージョン）
- **重要度エンジン** — 1〜10の自動スコア + 18ルールベースのインフラタギング
- **遅延検索** — briefモード（〜260 tok）+ context_searchオンデマンド
- **セッション継続性** — 環境変化の検出 + インフラメモリの自動包含
- **プロンプトガード** — 古い記憶に基づく誤った意思決定の防止
- **サーキットブレーカー** — 繰り返し失敗パターンの検出 + 自動Gotcha保存
- **Webダッシュボード** — プロジェクト別メモリ閲覧、検索、統計、パフォーマンス監視
- **ワンクリックインストール** — Hooks + スラッシュコマンドの自動インストール
- **多言語埋め込み** — paraphrase-multilingual-MiniLM-L12-v2（50以上の言語）

## ベンチマーク

HIPAA医療データパイプラインシナリオ（8セッション、14の意思決定トラップ）— Kandela ON vs OFF：

| | Kandela ON | Kandela OFF | 差分 |
|---|:-:|:-:|:-:|
| **意思決定トラップ回避** | **100%** | 11.9% | **+88.1pp** |
| **作業時間** | 77.9分 | 86.6分 | **-10.1%** |
| **生成コード** | 2,152行 | 3,441行 | **-37.5%** |
| **生成ファイル** | 40個 | 62個 | **-35.5%** |

> 3回実行（seeds=42,123,456）、claude-sonnet-4-6、Groq Llama 3.3 70B（Operator）。

### 重要な知見

- **コードにない意思決定が重要**: 監査人名、OOM事故、データ損失履歴 — コードリーディングでは見えない情報
- **コードベースの意思決定はLLMが自己防衛**: コード中心のシナリオ（InfraBot）では、LLMはKandelaなしでも95.2%の回避率を達成
- **手戻りを排除**: Kandelaなしでは、AIが以前却下されたアプローチを再実装 → 37.5%のコード無駄

### 大規模データ分析

830K件の実際の会話（WildChat-1M）と1M件のLLM会話（LMSYS-Chat-1M）：

| 発見 | 指標 | 示唆 |
|---------|:------:|-------------|
| 10ターン後に初期指示参照が18%p低下 | 66%→48% | 一般LLMは早期コンテキストを忘れ始める |
| 小型LLMは最大66%p低下 | chatglm 84%→19% | ローカル/小型モデルにメモリ拡張が不可欠 |
| ユーザー修正の14%が「前の会話を忘れた」 | 82K件のコーディング会話 | Kandelaが解決する核心的問題 |
| Kandelaは49ターン以降もコンテキスト維持 | 内部ベンチマーク | Brief Recallが減衰を防止 |

## アーキテクチャ

```
クライアント（Claude Code / Desktop / Cursor）
  │
  ├─ MCP Protocol ──→ Kandelaサーバー
  │                      │
  │                  MemoryStore (ChromaDB)
  │                      ├── ハイブリッド検索（Semantic + BM25 RRF）
  │                      ├── 重要度エンジン（18ルール）
  │                      ├── セッション継続性
  │                      └── プロンプトガード / サーキットブレーカー
  │
  └─ Webダッシュボード ──→ REST API + メモリブラウザ
```

### メモリタイプ

| タイプ | 説明 | 自動/手動 |
|------|-------------|:-----------:|
| `fact` | 好み、技術スタック、環境情報 | 両方 |
| `decision` | 設計判断、トレードオフ、根拠 | 手動 |
| `summary` | 会話終了時のセッション要約 | 自動 |
| `snippet` | よく使うコードパターン、設定 | 手動 |

### 検索パイプライン

```
クエリ → Semantic (MiniLM-L12) ─┐
                                ├─→ RRFフュージョン → 重要度再ランク → MMR多様性 → 結果
クエリ → BM25 (Korean NLP) ────┘
```

## はじめに

### ホスティングサービス（推奨）

インストール不要。登録して2分で開始できます。

```bash
# 1. https://api.kandela.ai/dashboard でサインアップ
# 2. APIキーを取得
# 3. Claude Codeを接続:
claude mcp add memory --transport http https://api.kandela.ai/mcp \
  --header "Authorization: Bearer mcp_YOUR_KEY"
curl -sf https://api.kandela.ai/api/install | bash
```

→ [スタートガイド](https://kandela.ai/getting-started)

### セルフホスト

自分のKandelaインスタンスを運用。シングルユーザーモード、データの完全な制御。

```bash
git clone https://github.com/deep-on/kandela-selfhost.git && cd kandela-selfhost/docker
docker compose up -d
# → http://localhost:8321/dashboard
```

→ [セルフホストリポジトリ](https://github.com/deep-on/kandela-selfhost)

## ホスティング vs セルフホスト

| | ホスティング (api.kandela.ai) | セルフホスト |
|---|:-:|:-:|
| セットアップ | 2分 | 5分 |
| マルチユーザー | ✅ | シングルユーザー |
| Telegramボット | ✅ | — |
| リモートコマンド | ✅ | — |
| アクティビティヒートマップ | ✅ | — |
| ティア機能（Pro/Max） | ✅ | 全機能 |
| データの場所 | クラウド | 自サーバー |
| メンテナンス | マネージド | セルフマネージド |

## 技術スタック

- **Python 3.11+**、FastMCP (mcp[cli])
- **ChromaDB** — ベクトルデータベース（永続化）
- **sentence-transformers** — ローカル埋め込み（paraphrase-multilingual-MiniLM-L12-v2）
- **Pydantic v2** — 入力バリデーション
- **Docker** — デプロイ

## リンク

- **ホームページ**: [kandela.ai](https://kandela.ai)
- **セルフホストリポジトリ**: [deep-on/kandela-selfhost](https://github.com/deep-on/kandela-selfhost)
- **プライバシーポリシー**: [kandela.ai/privacy](https://kandela.ai/privacy)
- **利用規約**: [kandela.ai/terms](https://kandela.ai/terms)

## ライセンス

- **サーバー**: [AGPL-3.0](LICENSE) — Copyright (c) 2025-2026 Deep-ON Inc.
- **クライアント側ファイル**（hooks、スラッシュコマンド）: [MIT](LICENSE-CLIENT)

## 免責事項

本ソフトウェアは、いかなる種類の保証もなく「現状のまま」提供されます。
ユーザーは自身のデータのバックアップに責任を負います。
