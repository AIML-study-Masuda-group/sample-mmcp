# mmcp によるMCPサーバー管理サンプル

**mmcp**は、Model Context Protocol (MCP) サーバーの設定を一元管理し、複数のAIエージェント（Claude Code、Cursor等）に適用するツールです。

## 📋 概要

### 主な機能

- MCPサーバーを`~/.mmcp.json`で一元管理
- 複数のエージェント（Claude Code、Claude Desktop、Codex CLI、GitHub Copilot CLI等）に一括適用
- 環境変数を使った安全なAPI key管理
- コマンド一つで全エージェントに設定を同期

### このリポジトリについて

このリポジトリには、mmcpを使ってMCPサーバーを管理するためのサンプル設定が含まれています:

- **mmcp単体での使い方** - 基本的な使用方法（このREADME）
- **miseによる自動化** - より高度な自動化（[docs/MISE.md](docs/MISE.md)）
- **CLIツール vs MCPサーバー** - `gh`や`gcloud`などのCLIツールとMCPサーバーを効率的に使い分ける方法（[docs/CLI_VS_MCP.md](docs/CLI_VS_MCP.md)）
- **API key取得方法** - 各サービスのトークン取得手順（[docs/API_KEYS.md](docs/API_KEYS.md)）
- **トラブルシューティング** - 問題解決ガイド（[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)）

---

## 🚀 クイックスタート

### 1. 前提条件

Node.jsがインストールされている必要があります。

```bash
# Node.js のインストール確認
node --version

# 未インストールの場合（macOS）
brew install node

# または公式サイトからダウンロード
# https://nodejs.org/
```

### 2. mmcp のインストール

```bash
# npm でインストール
npm install -g mmcp@0.6.2

# バージョン確認（v0.6.0 以上が必要）
mmcp --version
```

> **Note**: v0.6.0 以降で `--mode replace` オプションが追加されました。
> 古いバージョンでは MCP サーバーの削除が正しく反映されない場合があります。

### 3. MCPサーバーの追加

まずは環境変数不要のMCPサーバーから始めましょう。

```bash
# Sequential Thinking - 構造化された思考プロセス
mmcp add sequential-thinking npx -y @modelcontextprotocol/server-sequential-thinking

# Context7 - 最新ドキュメント検索
mmcp add context7 npx -y @upstash/context7-mcp

# Filesystem - ファイル操作（デスクトップへのアクセスを許可）
mmcp add filesystem npx -y @modelcontextprotocol/server-filesystem /Users/$(whoami)/Desktop

# Playwright - ブラウザ自動化
mmcp add playwright npx -y @playwright/mcp

# Draw.io - ダイアグラム生成（Mermaid/CSV/XML → draw.ioエディタで表示）
mmcp add drawio npx -y @drawio/mcp
```

### 4. エージェントの登録

使用するAIエージェントを登録します。

```bash
# VS Code拡張のClaude Codeを使う場合
mmcp agents add claude-code

# 他のエージェントも使う場合（オプション）
mmcp agents add claude-desktop     # Claude Desktopアプリ
mmcp agents add codex-cli          # OpenAI Codex CLI
mmcp agents add github-copilot-cli # GitHub Copilot CLI
```

### 5. 設定を適用

```bash
mmcp apply
```

### 6. VS Codeを再起動

VS Codeを再起動すると、Claude Code拡張機能でMCPサーバーが使えるようになります！

```
Command Palette (Cmd+Shift+P) > Developer: Reload Window
```

---

## 🔑 環境変数が必要なMCPサーバー

GitHub、Notion、Slack等のMCPサーバーを使う場合は、API key/トークンを設定する必要があります。

### トークンの取得

詳しい取得方法は [docs/API_KEYS.md](docs/API_KEYS.md) を参照してください。

- **GitHub**: Personal Access Token
- **Notion**: Integration Token
- **Slack**: Bot Token & Team ID
- **Google Cloud**: Application Default Credentials または サービスアカウントキー

### 環境変数の設定

```bash
# 一時的に設定
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxx
export NOTION_TOKEN=ntn_xxxxx

# 永続化する場合は ~/.zshrc や ~/.bashrc に追加
echo 'export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxx' >> ~/.zshrc
source ~/.zshrc
```

### MCPサーバーの追加

```bash
# GitHub - リポジトリ操作（Docker版）
mmcp add github docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server

# Notion - Notion API
mmcp add notion npx -y @notionhq/notion-mcp-server

# Slack - Slack API
mmcp add slack npx -y @modelcontextprotocol/server-slack

# 設定を適用
mmcp apply
```

---

## 📖 基本的なコマンド

### MCPサーバーの管理

```bash
# 登録済みMCPサーバーを確認
mmcp list

# MCPサーバーを削除
mmcp remove <サーバー名>

# 例: Sequential Thinking を削除
mmcp remove sequential-thinking
```

### エージェントの管理

```bash
# 登録済みエージェントを確認
mmcp agents list

# エージェントを追加
mmcp agents add <エージェント名>

# エージェントを削除
mmcp agents remove <エージェント名>
```

### 設定の適用

```bash
# すべてのエージェントに設定を適用（マージモード：既存設定を保持）
mmcp apply --mode merge

# 置換モード：mmcp で管理していない設定を削除
mmcp apply --mode replace

# 適用後は必ずVS Codeを再起動
# Command Palette (Cmd+Shift+P) > Developer: Reload Window
```

> **重要**: `mmcp remove` で MCP サーバーを削除した後は、`--mode replace` で適用してください。
> マージモード（デフォルト）では、削除した設定がエージェント側に残ったままになります。

---

## 📦 登録可能なMCPサーバー

### すぐに使えるもの（環境変数不要）

| サーバー名 | 説明 | パッケージ |
|---|---|---|
| Sequential Thinking | 構造化された思考プロセス | `@modelcontextprotocol/server-sequential-thinking` |
| Context7 | 最新ドキュメント検索 | `@upstash/context7-mcp` |
| Filesystem | ファイル操作 | `@modelcontextprotocol/server-filesystem` |
| Playwright | ブラウザ自動化 | `@playwright/mcp` |
| Draw.io | ダイアグラム生成（Mermaid/CSV/XML → draw.io） | `@drawio/mcp` |

### 環境変数が必要なもの

| サーバー名 | 説明 | 必要な環境変数 |
|---|---|---|
| GitHub | GitHubリポジトリ操作 | `GITHUB_PERSONAL_ACCESS_TOKEN` |
| Notion | Notion API | `NOTION_TOKEN` |
| Slack | Slack API | `SLACK_BOT_TOKEN`, `SLACK_TEAM_ID` |
| BigQuery | Google BigQuery | `BIGQUERY_PROJECT` |
| Dataplex | Dataplex Universal Catalog | `DATAPLEX_PROJECT` |

詳しい設定方法は [docs/API_KEYS.md](docs/API_KEYS.md) を参照してください。

---

## 🎯 対応エージェント

mmcpは以下のAIエージェントに対応しています:

- **claude-code** - VS Code拡張機能版Claude Code
- **claude-desktop** - Claude Desktopアプリ
- **codex-cli** - OpenAI Codex CLI
- **cursor** - Cursor IDE
- **gemini-cli** - Google Gemini CLI
- **github-copilot-cli** - GitHub Copilot CLI

---

## 🔧 より高度な使い方

### miseによる自動化（おすすめ）

**mise**を使うと、以下のようなメリットがあります:

- 環境変数の一元管理（`.env.mcp`ファイル）
- セットアップの自動化（複数のMCPサーバーを一括で設定）
- 再現性の向上（`config.toml`で設定を管理）
- バージョン管理（Node.jsやツールのバージョンを固定）

詳しくは [docs/MISE.md](docs/MISE.md) を参照してください。

---

## 🔍 トラブルシューティング

問題が発生した場合は [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) を参照してください。

よくある問題:

- [MCPサーバーが認識されない](docs/TROUBLESHOOTING.md#mcpサーバーが認識されない)
- [環境変数が読み込まれない](docs/TROUBLESHOOTING.md#環境変数が読み込まれない)
- [古いMCPサーバー設定が残っている](docs/TROUBLESHOOTING.md#古いmcpサーバー設定が残っている)

---

## 📁 ファイル構成

```
sample-mmcp/
├── README.md                # このファイル（mmcp基本ガイド）
├── config.toml              # mise設定（タスク定義とツールバージョン管理）
├── .env.mcp.sample          # 環境変数のテンプレート（実際の設定は.env.mcpにコピー）
├── .gitignore               # Git管理対象外ファイルの設定（.env.mcpなど）
└── docs/
    ├── MISE.md              # miseによる自動化ガイド（環境変数管理とセットアップ自動化）
    ├── CLI_VS_MCP.md        # CLIツールとMCPサーバーの使い分けガイド
    ├── API_KEYS.md          # 各サービスのAPI key/トークン取得方法
    └── TROUBLESHOOTING.md   # よくある問題と解決方法
```

### 各ファイルの詳細

#### ルートディレクトリ

- **[README.md](README.md)**: mmcpの基本的な使い方とクイックスタートガイド（このファイル）
- **[config.toml](config.toml)**: mise設定ファイル。MCPサーバーのセットアップタスク、Node.jsバージョン管理、環境変数読み込みなどを定義
- **[.env.mcp.sample](.env.mcp.sample)**: 環境変数のテンプレート。実際に使用する際は `.env.mcp` としてコピーし、実際のトークン値を設定
- **[.gitignore](.gitignore)**: Gitで管理しないファイルを定義（`.env.mcp`、`.DS_Store`など）

#### docsディレクトリ

- **[MISE.md](docs/MISE.md)**: miseを使った高度な自動化ガイド。環境変数の一元管理やセットアップの自動化方法を解説
- **[CLI_VS_MCP.md](docs/CLI_VS_MCP.md)**: CLIツール（`gh`、`gcloud`など）とMCPサーバーの使い分けガイド。どちらを優先すべきか、具体的なユースケース別に解説
- **[API_KEYS.md](docs/API_KEYS.md)**: GitHub、Notion、Slack、Google Cloud等のAPI keyやトークンの取得手順を詳しく解説
- **[TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)**: MCPサーバーが認識されない、環境変数が読み込まれないなど、よくある問題と解決方法

---

## 📚 参考リンク

- [mmcp GitHub](https://github.com/koki-develop/mmcp)
- [MCP Market](https://mcpmarket.com/server) - MCPサーバーカタログ
- [Model Context Protocol 公式](https://modelcontextprotocol.io/)
- [Claude Code ドキュメント](https://docs.claude.com/en/docs/claude-code)
- [mise 公式ドキュメント](https://mise.jdx.dev/)

---

## 📝 ライセンス

このサンプルはMITライセンスの下で公開されています。
