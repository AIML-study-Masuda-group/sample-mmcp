# API key・トークンの取得方法

このガイドでは、各MCPサーバーで必要なAPI key・トークンの取得方法を説明します。

## 🔑 設定方法

取得したAPI key/トークンは、以下のいずれかの方法で設定します。

### mmcp単体の場合

環境変数として設定:

```bash
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxx
export NOTION_TOKEN=ntn_xxxxx

# 永続化する場合は ~/.zshrc や ~/.bashrc に追加
echo 'export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxx' >> ~/.zshrc
```

### miseを使う場合

`~/.config/mise/.env.mcp` ファイルに記載:

```bash
# サンプルファイルをコピー
cp sample-mmcp/.env.mcp.sample ~/.config/mise/.env.mcp

# エディタで開き、API key/トークンを設定
vim ~/.config/mise/.env.mcp
```

---

## GitHub Personal Access Token

GitHubリポジトリの操作（コード検索、Issue/PR管理等）に必要です。

### 取得手順

1. https://github.com/settings/tokens にアクセス
2. **"Generate new token (classic)"** をクリック
3. トークンの説明を入力（例: "MCP Server for Claude Code"）
4. 必要なスコープを選択:
   - `repo` - プライベートリポジトリも含む場合
   - `public_repo` - 公開リポジトリのみの場合
5. **"Generate token"** をクリック
6. 表示されたトークンをコピー（`ghp_`で始まる文字列）

### 設定例

```bash
# 環境変数として設定
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# または .env.mcp に記載
GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 注意事項

- トークンは一度しか表示されないため、必ずコピーして保存してください
- トークンは定期的に更新することをおすすめします
- 不要になったトークンは削除してください

---

## Notion Integration Token

Notionページの読み書き、データベース操作に必要です。

### 取得手順

1. https://www.notion.so/profile/integrations にアクセス
2. **"新しいインテグレーションを作成"** をクリック
3. インテグレーション名を入力（例: "Claude Code MCP"）
4. ワークスペースを選択
5. **"送信"** をクリック
6. **Internal Integration Token** をコピー（`ntn_`で始まる文字列）
7. Notionで使用したいページを開き、**"..."** メニューから **"接続を追加"** を選択
8. 作成したインテグレーションを選択

### 設定例

```bash
# 環境変数として設定
export NOTION_TOKEN=ntn_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# または .env.mcp に記載
NOTION_TOKEN=ntn_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 注意事項

- インテグレーションを追加したページとその子ページにのみアクセスできます
- ページごとに「接続を追加」する必要があります
- 不要になった接続はNotionページから削除できます

---

## Slack Bot Token

Slackチャンネルのメッセージ読み書き、情報取得に必要です。

### 取得手順

1. https://api.slack.com/apps にアクセス
2. **"Create New App"** をクリック
3. **"From scratch"** を選択
4. アプリ名を入力（例: "Claude Code MCP"）
5. ワークスペースを選択
6. **"Create App"** をクリック
7. サイドバーの **"OAuth & Permissions"** をクリック
8. **"Scopes"** セクションで以下のBot Token Scopesを追加:
   - `channels:read` - チャンネル一覧の取得
   - `channels:history` - チャンネルメッセージの読み取り
   - `chat:write` - メッセージの投稿
9. ページ上部の **"Install to Workspace"** をクリック
10. **"許可する"** をクリック
11. 表示された **Bot User OAuth Token** をコピー（`xoxb-`で始まる文字列）

### Team IDの取得

1. Slackワークスペースを開く
2. ワークスペース名をクリック
3. **"設定と管理"** > **"ワークスペースの設定"** を選択
4. URLに表示される `https://app.slack.com/client/T01234567/...` の `T01234567` 部分がTeam ID

### 設定例

```bash
# 環境変数として設定
export SLACK_BOT_TOKEN=xoxb-xxxxxxxxxxxx-xxxxxxxxxxxx-xxxxxxxxxxxxxxxxxxxxxxxx
export SLACK_TEAM_ID=T01234567

# または .env.mcp に記載
SLACK_BOT_TOKEN=xoxb-xxxxxxxxxxxx-xxxxxxxxxxxx-xxxxxxxxxxxxxxxxxxxxxxxx
SLACK_TEAM_ID=T01234567
```

### オプション設定

特定のチャンネルのみに制限したい場合:

```bash
# カンマ区切りでチャンネルIDを指定
SLACK_CHANNEL_IDS=C01234567,C76543210
```

チャンネルIDの確認方法:
1. Slackでチャンネルを開く
2. チャンネル名をクリック
3. 画面下部に表示される「チャンネルID」をコピー

---

## Google Cloud（BigQuery & Dataplex）

Google CloudのBigQueryやDataplexにアクセスするには、認証設定が必要です。

### 前提条件

- Google Cloudプロジェクトが作成済みであること
- BigQueryまたはDataplex APIが有効化されていること

### 必要なツールのインストール

#### 1. gcloud CLI のインストール

Google Cloud の認証に必要です。

```bash
# macOS（Homebrew）
brew install google-cloud-sdk

# インストール確認
gcloud --version
```

または[公式サイト](https://cloud.google.com/sdk/docs/install)からダウンロードしてインストールできます。

#### 2. toolbox のインストール

GCP MCP サーバー（BigQuery、Dataplex）を動かすために必要な [MCP Toolbox for Databases](https://github.com/googleapis/genai-toolbox) です。gcloud CLI とは別のツールです。

```bash
# ディレクトリ作成（存在しない場合）
mkdir -p ~/.local/bin

# macOS (Apple Silicon / M1, M2, M3, M4)
curl -o ~/.local/bin/toolbox https://storage.googleapis.com/genai-toolbox/v0.21.0/darwin/arm64/toolbox
chmod +x ~/.local/bin/toolbox

# macOS (Intel)
curl -o ~/.local/bin/toolbox https://storage.googleapis.com/genai-toolbox/v0.21.0/darwin/amd64/toolbox
chmod +x ~/.local/bin/toolbox

# インストール確認
~/.local/bin/toolbox --version
```

> **Note:** 最新バージョンは [GitHub Releases](https://github.com/googleapis/genai-toolbox/releases) で確認できます。

PATHに `~/.local/bin` を追加していない場合は、以下を `~/.zshrc` に追加してください:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### 認証方法（2つから選択）

#### 方法1: Application Default Credentials（推奨）

最も簡単な方法です。以下のコマンドを実行するだけで認証が完了します:

```bash
gcloud auth application-default login
```

この方法を使う場合、`.env.mcp`で`GOOGLE_APPLICATION_CREDENTIALS`を設定する必要はありません。

**設定例:**

```bash
# .env.mcp に記載（GOOGLE_APPLICATION_CREDENTIALSは空でOK）
BIGQUERY_PROJECT=my-project-12345
DATAPLEX_PROJECT=my-project-12345
GOOGLE_APPLICATION_CREDENTIALS=
```

#### 方法2: サービスアカウントキー

より厳密な権限管理が必要な場合や、本番環境で使用する場合に適しています。

**取得手順:**

1. https://console.cloud.google.com/ にアクセス
2. プロジェクトを選択
3. サイドバーから **"IAM & Admin"** > **"Service Accounts"** を選択
4. **"Create Service Account"** をクリック
5. サービスアカウント名を入力（例: "mcp-bigquery-access"）
6. 必要なロールを付与:

   **BigQuery用:**
   - `BigQuery User` (`roles/bigquery.user`)
   - `BigQuery Metadata Viewer` (`roles/bigquery.metadataViewer`)
   - `BigQuery Data Editor` (`roles/bigquery.dataEditor`)

   **Dataplex用:**
   - `Dataplex Reader` (`roles/dataplex.viewer`)
   - `Dataplex Editor` (`roles/dataplex.editor`)

7. **"Done"** をクリック
8. 作成したサービスアカウントをクリック
9. **"Keys"** タブを選択
10. **"Add Key"** > **"Create new key"** をクリック
11. **"JSON"** を選択して **"Create"** をクリック
12. ダウンロードされたJSONファイルを保存

**ファイル配置:**

```bash
# ディレクトリ作成
mkdir -p ~/.config/gcp

# ダウンロードしたキーファイルを移動
mv ~/Downloads/your-project-xxxxx.json ~/.config/gcp/service-account.json

# パーミッション設定（重要）
chmod 600 ~/.config/gcp/service-account.json
```

**設定例:**

```bash
# 環境変数として設定
export BIGQUERY_PROJECT=my-project-12345
export DATAPLEX_PROJECT=my-project-12345
export GOOGLE_APPLICATION_CREDENTIALS=/Users/$(whoami)/.config/gcp/service-account.json

# または .env.mcp に記載
BIGQUERY_PROJECT=my-project-12345
DATAPLEX_PROJECT=my-project-12345
GOOGLE_APPLICATION_CREDENTIALS=/Users/$(whoami)/.config/gcp/service-account.json
```

### プロジェクトIDの確認

1. https://console.cloud.google.com/ にアクセス
2. 画面上部のプロジェクト選択メニューをクリック
3. 「プロジェクトID」列に表示される値をコピー

### 注意事項

- サービスアカウントキーは秘密情報です。gitリポジトリにコミットしないでください
- 不要になったサービスアカウントキーは削除してください
- 本番環境では、最小限の権限のみを付与することをおすすめします

---

## トラブルシューティング

### トークンが無効と表示される

- トークンの有効期限が切れていないか確認してください
- トークンのスコープ/権限が正しく設定されているか確認してください
- トークンを再生成してみてください

### 環境変数が読み込まれない

**mmcp単体の場合:**
```bash
# 環境変数が設定されているか確認
echo $GITHUB_PERSONAL_ACCESS_TOKEN

# シェル設定ファイルに記載されているか確認
cat ~/.zshrc | grep GITHUB_PERSONAL_ACCESS_TOKEN
```

**miseの場合:**
```bash
# .env.mcp ファイルが存在するか確認
ls -la ~/.config/mise/.env.mcp

# 内容を確認
cat ~/.config/mise/.env.mcp
```

### Google Cloud認証エラー

```bash
# ADCが設定されているか確認
gcloud auth application-default print-access-token

# プロジェクトが正しく設定されているか確認
gcloud config get-value project

# APIが有効化されているか確認
gcloud services list --enabled | grep bigquery
```

---

## 参考リンク

- [GitHub Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [Notion API Getting Started](https://developers.notion.com/docs/getting-started)
- [Slack API Documentation](https://api.slack.com/start)
- [Google Cloud Authentication](https://cloud.google.com/docs/authentication/getting-started)
- [BigQuery API](https://cloud.google.com/bigquery/docs/reference/rest)
