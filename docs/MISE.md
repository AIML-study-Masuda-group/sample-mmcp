# mise による自動化ガイド

mmcp単体での使い方でも十分ですが、**miseタスク**を使うことで、さらに便利に自動化できます。

## miseによる自動化のメリット

- **環境変数の一元管理**: `.env.mcp`ファイルで環境変数を管理
- **セットアップの自動化**: 複数のMCPサーバーを一括でセットアップ
- **再現性の向上**: `config.toml`で設定を管理し、チーム内で共有可能
- **バージョン管理**: Node.jsやツールのバージョンを固定

---

## セットアップ手順

### 1. 前提条件（mise自動インストール）

```bash
# mise のインストール（手動）
curl https://mise.run | sh

# Node.js と mmcp のインストール（mise が自動でインストール）
mise install
```

この手順では、`mise install` コマンドが `config.toml` の定義に基づいて必要なツールを自動的にインストールします。

### 2. 設定ファイルのセットアップ（手動）

この手順は **手動で一度だけ** 実行する必要があります。

#### config.toml のマージ

sample-mmcp/config.toml の内容を、グローバル設定にマージします：

```bash
# 方法1: 自動マージ
cat sample-mmcp/config.toml >> ~/.config/mise/config.toml

# 方法2: 手動でコピー
vim ~/.config/mise/config.toml
# sample-mmcp/config.toml の [tools] と [tasks.*] セクションをコピー
```

#### 環境変数ファイルの作成

```bash
# サンプルファイルをコピー
cp sample-mmcp/.env.mcp.sample ~/.config/mise/.env.mcp

# エディタで開き、API key/トークンを設定
vim ~/.config/mise/.env.mcp
```

**重要**: `.env.mcp` には実際の API key やトークンを設定してください。設定方法は[API_KEYS.md](API_KEYS.md)を参照。

### 3. MCPサーバーのセットアップ（miseタスクで自動化）

#### 3-1. すぐに使えるMCPサーバー（環境変数不要）

```bash
mise run mcp-setup-quick
```

登録されるサーバー:
- **Sequential Thinking** - 構造化された思考プロセス
- **Context7** - 最新ドキュメント検索
- **Filesystem** - ファイル操作
- **Playwright** - ブラウザ自動化

#### 3-2. 環境変数が必要なMCPサーバー

前提: `.env.mcp` にAPI key/トークンが設定されていること（手順2参照）

```bash
mise run mcp-setup-with-env
```

登録されるサーバー（環境変数が設定されているもののみ）:
- **GitHub** - GitHubリポジトリ操作
- **Notion** - Notion API
- **Slack** - Slack API

#### 3-3. Google Cloud（BigQuery & Dataplex）のセットアップ

Google公式のMCP Toolboxを使用してBigQueryとDataplexに接続します。

**前提条件（手動）**: Google Cloudへの認証設定が完了していること
- **推奨**: `gcloud auth application-default login` を実行
- **または**: `.env.mcp` に `GOOGLE_APPLICATION_CREDENTIALS` を設定

```bash
# MCP Toolbox バイナリをインストール（miseタスク）
mise run mcp-install-toolbox

# BigQuery & Dataplex を mmcp に追加（miseタスク）
mise run mcp-setup-gcp-toolbox
```

#### 3-4. Serenaのセットアップ（オプション）

Serenaはセマンティックコード検索ツールです。大規模なコードベースでの開発に便利です。

```bash
# Serenaをインストール（miseタスク）
mise run mcp-install-serena

# mmcpに追加（miseタスク）
mise run mcp-add-serena
```

対応言語: Python, TypeScript/JavaScript, PHP, Go, Rust, C/C++, Java

### 4. エージェントの登録と適用（miseタスクで自動化）

```bash
# エージェントを登録
mise run mmcp-agents-add claude-code
# または複数同時に
mise run mmcp-agents-add claude-code claude-desktop codex-cli

# 設定を適用
mise run mmcp-apply
```

### 5. VS Codeを再起動（手動）

VS Codeを再起動すると、Claude Code拡張機能でMCPサーバーが使えるようになります！

```
Command Palette (Cmd+Shift+P) > Developer: Reload Window
```

---

## miseタスク一覧

### MCPサーバーのセットアップ

```bash
# すぐに使えるMCPサーバーを追加
mise run mcp-setup-quick

# 環境変数が必要なMCPサーバーを追加
mise run mcp-setup-with-env

# Google MCP Toolboxをインストール
mise run mcp-install-toolbox

# BigQuery & Dataplexを追加
mise run mcp-setup-gcp-toolbox

# Serenaをインストール
mise run mcp-install-serena

# Serenaをmmcpに追加
mise run mcp-add-serena
```

### エージェントの管理

```bash
# エージェントを追加
mise run mmcp-agents-add <agent-name>

# 登録済みエージェント一覧
mise run mmcp-agents-list

# 設定を適用
mise run mmcp-apply
```

### MCPサーバーの管理

```bash
# 登録済みサーバー一覧
mise run mmcp-list

# MCPサーバーを削除
mise run mmcp-remove <server-name>

# MCPサーバーカタログを開く
mise run mmcp-browse-catalog
```

---

## トラブルシューティング

### MCPサーバーが認識されない

```bash
# miseタスクで設定を確認
mise run mmcp-list

# 設定を再適用
mise run mmcp-apply

# VS Codeを再起動
# Command Palette (Cmd+Shift+P) > Reload Window
```

### 環境変数が読み込まれない

```bash
# 環境変数ファイルが存在するか確認
ls -la ~/.config/mise/.env.mcp

# 環境変数を再読み込み
source ~/.config/mise/.env.mcp

# 設定を再適用
mise run mcp-setup-with-env
mise run mmcp-apply
```

### Serenaが動作しない

```bash
# serenaコマンドが使えるか確認
which serena

# 再インストール
uv tool uninstall serena
mise run mcp-install-serena
mise run mcp-add-serena
mise run mmcp-apply
```
