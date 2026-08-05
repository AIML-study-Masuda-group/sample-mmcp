# mmcp によるMCPサーバー管理サンプル

**mmcp**は、Model Context Protocol (MCP) サーバーの設定を一元管理し、複数のAIエージェント（Claude Code、Cursor等）に適用するツールです。

## 📋 概要

### 主な機能

- MCPサーバーを`~/.mmcp.json`で一元管理
- 複数のエージェント（Claude Code、Claude Desktop、GitHub Copilot CLI等）に一括適用
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

Notion等のMCPサーバーを使う場合は、API key/トークンを設定する必要があります。

### トークンの取得

詳しい取得方法は [docs/API_KEYS.md](docs/API_KEYS.md) を参照してください。

- **Notion**: Integration Token（書き込み用・読み取り専用の2種類）
- **Google Cloud**: Application Default Credentials または サービスアカウントキー

### 環境変数の設定

```bash
# 一時的に設定
export NOTION_TOKEN=ntn_xxxxx

# 永続化する場合は ~/.zshrc や ~/.bashrc に追加
echo 'export NOTION_TOKEN=ntn_xxxxx' >> ~/.zshrc
source ~/.zshrc
```

### MCPサーバーの追加

```bash
# Notion - Notion API（書き込み権限あり）
mmcp add notion npx -y @notionhq/notion-mcp-server

# Notion readonly - Notion API（読み取り専用）
mmcp add notion-readonly npx -y @notionhq/notion-mcp-server

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
| Filesystem | ファイル操作 | `@modelcontextprotocol/server-filesystem` |
| Playwright | ブラウザ自動化 | `@playwright/mcp` |
| Draw.io | ダイアグラム生成（Mermaid/CSV/XML → draw.io） | `@drawio/mcp` |

### 環境変数が必要なもの

| サーバー名 | 説明 | 必要な環境変数 |
|---|---|---|
| Notion | Notion API（書き込み権限あり） | `NOTION_TOKEN` |
| Notion readonly | Notion API（読み取り専用） | `NOTION_READONLY_TOKEN` |
| BigQuery | Google BigQuery | `BIGQUERY_PROJECT` |
| Dataplex | Dataplex Universal Catalog | `DATAPLEX_PROJECT` |

詳しい設定方法は [docs/API_KEYS.md](docs/API_KEYS.md) を参照してください。

### mmcp 管理対象外（エージェント個別設定）

以下のMCPサーバーは、特定のエージェントでのみ使用するため mmcp では管理せず、各エージェントの設定ファイルに直接記載します。

| サーバー名 | 対象エージェント | 理由 |
|---|---|---|
| GitHub | Claude Desktop のみ | Claude Code 等では `gh` CLI や `.mcp.json` で対応。MCP のトークン消費を避けるため |

#### GitHub MCP（Claude Desktop 個別設定）

mise でバイナリをインストールし、Claude Desktop の設定ファイルに直接追加します。Docker / Go / npm は不要です。

**1. github-mcp-server のインストール:**

```bash
# mise のグローバル設定にインストール（~/.config/mise/config.toml に追加される）
mise use -g ubi:github/github-mcp-server@0.32.0
```

**2. Claude Desktop の設定ファイルに追加:**

設定ファイルの場所: `~/Library/Application Support/Claude/claude_desktop_config.json`

`mcpServers` に以下を追加（`<HOME>` は各ユーザーのホームディレクトリに置き換え）:

```json
{
  "github": {
    "command": "<HOME>/.local/share/mise/installs/ubi-github-github-mcp-server/0.32.0/github-mcp-server",
    "args": ["stdio"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "<PAT>"
    }
  }
}
```

> **Note**: Claude Desktop は mise の shim を経由しないため、バイナリのフルパスを指定する必要があります。

**3. Fine-grained PAT の作成:**

1. https://github.com/settings/personal-access-tokens/new にアクセス
2. 以下の設定で作成:
   - **Resource owner**: scene-live
   - **Repository access**: 必要なリポを選択（data-analytics-platform, insight-hub 等）
   - **Permissions**:
     - Contents: Read
     - Pull requests: Read & Write
     - Issues: Read
     - Metadata: Read
   - **Expiration**: 1 year
   - **Description**: `Claude Desktop MCP - 社内リポ探索・PR作成用（github-mcp-server）`

**注意事項:**
- Copilot HTTP 版（`api.githubcopilot.com/mcp/`）は Organization のプライベートリポジトリにアクセスできないため使用しない
- Claude Desktop の GUI「カスタムコネクタ」に GitHub を追加済みの場合は削除する（Organization アクセスの問題があるため）
- `mmcp apply --mode replace` を実行すると、この手動設定が消えるため注意（merge モードでは保持される）

---

## 🎯 対応エージェント

mmcpは以下のAIエージェントに対応しています:

- **claude-code** - VS Code拡張機能版Claude Code
- **claude-desktop** - Claude Desktopアプリ
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
- **[API_KEYS.md](docs/API_KEYS.md)**: GitHub、Notion、Google Cloud等のAPI keyやトークンの取得手順を詳しく解説
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
