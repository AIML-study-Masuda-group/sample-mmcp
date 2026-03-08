# トラブルシューティング

mmcp や MCP サーバーで問題が発生した場合の対処方法をまとめています。

---

## 目次

- [MCPサーバーが認識されない](#mcpサーバーが認識されない)
- [環境変数が読み込まれない](#環境変数が読み込まれない)
- [古いMCPサーバー設定が残っている](#古いmcpサーバー設定が残っている)
- [特定のMCPサーバーが動作しない](#特定のmcpサーバーが動作しない)
- [npxコマンドが見つからない](#npxコマンドが見つからない)
- [Dockerコマンドが見つからない](#dockerコマンドが見つからない)

---

## MCPサーバーが認識されない

Claude CodeやClaude DesktopでMCPサーバーが表示されない、または動作しない場合。

### 確認方法

**mmcp単体の場合:**

```bash
# mmcpに登録されているか確認
mmcp list

# エージェントに設定が適用されているか確認
cat ~/.claude.json | jq '.mcpServers'
```

**miseを使う場合:**

```bash
# mmcpに登録されているか確認
mise run mmcp-list

# エージェントに設定が適用されているか確認
cat ~/.claude.json | jq '.mcpServers'
```

### 解決方法

#### 1. 設定を再適用

**mmcp単体:**
```bash
mmcp apply
```

**mise:**
```bash
mise run mmcp-apply
```

#### 2. VS Codeを再起動

設定変更後は必ずVS Codeを再起動してください:

```
Command Palette (Cmd+Shift+P) > Developer: Reload Window
```

#### 3. エージェントが登録されているか確認

**mmcp単体:**
```bash
# エージェント一覧を確認
mmcp agents list

# claude-codeが登録されていない場合
mmcp agents add claude-code
mmcp apply
```

**mise:**
```bash
# エージェント一覧を確認
mise run mmcp-agents-list

# claude-codeが登録されていない場合
mise run mmcp-agents-add claude-code
mise run mmcp-apply
```

#### 4. 設定ファイルを直接確認

```bash
# Claude Code の設定ファイル
cat ~/.claude.json | jq '.'

# 構文エラーがないか確認
jq '.' ~/.claude.json
```

---

## 環境変数が読み込まれない

GitHub、Notion、Slack等のMCPサーバーで認証エラーが発生する場合。

### 確認方法

**mmcp単体の場合:**

```bash
# 環境変数が設定されているか確認
echo $GITHUB_PERSONAL_ACCESS_TOKEN
echo $NOTION_TOKEN
echo $SLACK_BOT_TOKEN

# シェル設定ファイルに記載されているか確認
cat ~/.zshrc | grep GITHUB_PERSONAL_ACCESS_TOKEN
cat ~/.bashrc | grep GITHUB_PERSONAL_ACCESS_TOKEN
```

**miseを使う場合:**

```bash
# .env.mcp ファイルが存在するか確認
ls -la ~/.config/mise/.env.mcp

# 内容を確認（トークンが実際に設定されているか）
cat ~/.config/mise/.env.mcp | grep GITHUB_PERSONAL_ACCESS_TOKEN
```

### 解決方法

#### mmcp単体の場合

1. **環境変数を設定:**

```bash
# 一時的に設定
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxx
export NOTION_TOKEN=ntn_xxxxx

# 永続化（~/.zshrc または ~/.bashrc に追加）
echo 'export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxx' >> ~/.zshrc
echo 'export NOTION_TOKEN=ntn_xxxxx' >> ~/.zshrc

# シェル設定を再読み込み
source ~/.zshrc
```

2. **MCPサーバーを再登録:**

```bash
# 既存の設定を削除
mmcp remove github
mmcp remove notion

# 環境変数付きで再登録
mmcp add github docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server
mmcp add notion npx -y @notionhq/notion-mcp-server

# 適用
mmcp apply
```

#### miseを使う場合

1. **`.env.mcp`ファイルを確認・編集:**

```bash
# ファイルが存在しない場合はコピー
cp sample-mmcp/.env.mcp.sample ~/.config/mise/.env.mcp

# エディタで開き、実際のトークンを設定
vim ~/.config/mise/.env.mcp
```

2. **環境変数を読み込んでセットアップ:**

```bash
# 環境変数を読み込む
source ~/.config/mise/.env.mcp

# MCPサーバーをセットアップ
mise run mcp-setup-with-env

# 適用
mise run mmcp-apply
```

3. **VS Codeを再起動:**

```
Command Palette (Cmd+Shift+P) > Developer: Reload Window
```

---

## 古いMCPサーバー設定が残っている

MCPサーバーを移行・更新した際、各エージェントの設定ファイルに古い設定が残ることがあります。

### 原因

mmcp v0.5.0 以前では `mmcp apply` がマージモードのみで、削除した設定が反映されませんでした。
v0.6.0 以降では `--mode replace` オプションで解決できます。

### 確認方法

```bash
# mmcp のバージョン確認
mmcp --version

# mmcpに登録されているサーバー一覧
mmcp list

# 各エージェントの設定を確認
cat ~/.claude.json | jq '.mcpServers | keys'
cat ~/.codex/config.toml | grep '^\[mcp_servers\.'
cat ~/.copilot/mcp-config.json | jq '.mcpServers | keys'
```

### 解決方法

#### 方法1: 置換モードで適用（推奨・v0.6.0+）

**mmcp単体:**
```bash
mmcp remove <古いサーバー名>
mmcp apply --mode replace
```

**mise:**
```bash
mise run mmcp-remove-and-apply <古いサーバー名>

# または手動で
mise run mmcp-remove <古いサーバー名>
mise run mmcp-apply-replace
```

#### 方法2: 各エージェントから手動削除（v0.5.0 以前または方法1で解決しない場合）

**Claude Code (JSON形式):**

```bash
# 対話的に削除
vim ~/.claude.json

# または jq で削除
cat ~/.claude.json | jq 'del(.mcpServers."古いサーバー名")' > /tmp/tmp.json
mv /tmp/tmp.json ~/.claude.json
```

**Claude Desktop (JSON形式):**

```bash
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json | jq 'del(.mcpServers."古いサーバー名")' > /tmp/tmp.json
mv /tmp/tmp.json ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**Codex CLI (TOML形式):**

```bash
# エディタで手動削除
vim ~/.codex/config.toml
# [mcp_servers.古いサーバー名] セクションと [mcp_servers.古いサーバー名.env] セクションを削除
```

**GitHub Copilot CLI (JSON形式):**

```bash
cat ~/.copilot/mcp-config.json | jq 'del(.mcpServers."古いサーバー名")' > /tmp/tmp.json
mv /tmp/tmp.json ~/.copilot/mcp-config.json
```

#### 3. VS Code / Claude Desktop を再起動

```
Command Palette (Cmd+Shift+P) > Developer: Reload Window
```

### よくあるケース

- MCPサーバーを公式版に移行した時（例: `bigquery` → `gcp-bigquery`）
- パッケージ名が変更された時
- Docker版に移行した時（例: npm版GitHub → Docker版GitHub）
- 環境変数の設定方法が変更された時
- **不要になったMCPサーバーを削除した時**（v0.6.0+ では `--mode replace` で解決）

---

## 特定のMCPサーバーが動作しない

### Serenaが動作しない

**確認方法:**

```bash
# serenaコマンドが使えるか確認
which serena

# バージョン確認
serena --version
```

**解決方法:**

```bash
# 再インストール
uv tool uninstall serena
uv tool install serena

# mmcpに再登録（mise使用時）
mise run mcp-add-serena
mise run mmcp-apply

# mmcpに再登録（mmcp単体）
mmcp add serena serena mcp
mmcp apply
```

### BigQuery/Dataplexが動作しない

**確認方法:**

```bash
# MCP Toolboxがインストールされているか確認
which mcp-bigquery
which mcp-dataplex

# Google Cloud認証が設定されているか確認
gcloud auth application-default print-access-token
```

**解決方法:**

1. **MCP Toolboxを再インストール:**

```bash
# mise使用時
mise run mcp-install-toolbox

# mmcp単体の場合
npm install -g @google-cloud/mcp-toolbox
```

2. **Google Cloud認証を設定:**

```bash
# Application Default Credentials（推奨）
gcloud auth application-default login

# または、サービスアカウントキーを設定
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

3. **プロジェクトIDが正しいか確認:**

```bash
# .env.mcp を確認
cat ~/.config/mise/.env.mcp | grep BIGQUERY_PROJECT
cat ~/.config/mise/.env.mcp | grep DATAPLEX_PROJECT
```

### GitHubサーバーが動作しない（Docker版）

**確認方法:**

```bash
# Dockerが起動しているか確認
docker ps

# トークンが設定されているか確認
echo $GITHUB_PERSONAL_ACCESS_TOKEN
```

**解決方法:**

```bash
# Dockerを起動
open -a Docker

# トークンを再設定
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxx

# mmcpに再登録
mmcp remove github
mmcp add github docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server
mmcp apply
```

---

## npxコマンドが見つからない

`npx: command not found` エラーが発生する場合。

### 確認方法

```bash
# Node.jsがインストールされているか確認
which node
node --version

# npmがインストールされているか確認
which npm
npm --version

# npxがインストールされているか確認
which npx
npx --version
```

### 解決方法

#### 1. Node.jsをインストール

**Homebrewを使う場合:**

```bash
brew install node
```

**公式インストーラーを使う場合:**

1. https://nodejs.org/ にアクセス
2. LTS版をダウンロード
3. インストーラーを実行

#### 2. PATHを確認

```bash
# PATHを確認
echo $PATH

# Node.jsのパスが含まれているか確認
which node

# 含まれていない場合、シェル設定ファイルに追加
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

#### 3. miseを使う場合

```bash
# mise でNode.jsをインストール
mise install node@latest

# または、プロジェクトのconfig.tomlに記載されているバージョンをインストール
mise install
```

---

## Dockerコマンドが見つからない

`docker: command not found` エラーが発生する場合。

### 確認方法

```bash
# Dockerがインストールされているか確認
which docker
docker --version

# Dockerが起動しているか確認
docker ps
```

### 解決方法

#### 1. Docker Desktopをインストール

1. https://www.docker.com/products/docker-desktop/ にアクセス
2. macOS版をダウンロード
3. インストーラーを実行

#### 2. Dockerを起動

```bash
# Docker Desktopを起動
open -a Docker

# 起動を確認
docker ps
```

#### 3. PATHを確認

```bash
# PATHを確認
echo $PATH

# Dockerのパスが含まれているか確認
which docker

# 含まれていない場合、シェル設定ファイルに追加
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

## その他のエラー

### JSONパースエラー

**エラーメッセージ:**
```
SyntaxError: Unexpected token } in JSON
```

**解決方法:**

```bash
# 設定ファイルの構文を確認
jq '.' ~/.claude.json

# エラーがある場合、バックアップから復元
cp ~/.claude.json.backup ~/.claude.json

# または、mmcpで再生成
mmcp apply
```

### パーミッションエラー

**エラーメッセージ:**
```
Error: EACCES: permission denied
```

**解決方法:**

```bash
# npm のグローバルディレクトリを確認
npm config get prefix

# パーミッションを修正
sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}

# または、nvm/mise を使ってNode.jsを管理（推奨）
```

### ポート競合エラー

**エラーメッセージ:**
```
Error: listen EADDRINUSE: address already in use
```

**解決方法:**

```bash
# 使用中のポートを確認
lsof -i :ポート番号

# プロセスを終了
kill -9 <PID>

# または、別のポートを使用するようMCPサーバーを設定
```

---

## サポート

上記の方法で解決しない場合は、以下を確認してください:

- [mmcp GitHub Issues](https://github.com/koki-develop/mmcp/issues)
- [Model Context Protocol 公式ドキュメント](https://modelcontextprotocol.io/)
- [Claude Code ドキュメント](https://docs.claude.com/en/docs/claude-code)

問題を報告する際は、以下の情報を含めてください:

```bash
# システム情報
uname -a
node --version
npm --version
mmcp --version

# mmcp設定
mmcp list
mmcp agents list

# エラーメッセージ全文
```
