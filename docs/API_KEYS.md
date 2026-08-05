# API key・トークンの取得方法

このガイドでは、各MCPサーバーで必要なAPI key・トークンの取得方法を説明します。

## 🔑 設定方法

取得したAPI key/トークンは、以下のいずれかの方法で設定します。

### mmcp単体の場合

環境変数として設定:

```bash
export NOTION_TOKEN=ntn_xxxxx

# 永続化する場合は ~/.zshrc や ~/.bashrc に追加
echo 'export NOTION_TOKEN=ntn_xxxxx' >> ~/.zshrc
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

Claude Desktop の GitHub MCP サーバーに必要です。

> **Note**: GitHub MCP は mmcp 管理対象外です。Claude Code では `gh` CLI を使うため不要で、
> Claude Desktop の設定ファイルに直接記載します（手順は [README](../README.md) の
> 「GitHub MCP（Claude Desktop 個別設定）」を参照）。`.env.mcp` には値を控えておく用途で記載します。

### 取得手順

1. https://github.com/settings/personal-access-tokens/new にアクセス（Fine-grained PAT）
2. 以下を設定:
   - **Resource owner**: 対象の Organization またはユーザー
   - **Repository access**: 必要なリポジトリを選択
   - **Permissions**:
     - Contents: Read
     - Pull requests: Read & Write
     - Issues: Read
     - Metadata: Read
   - **Expiration**: 1 year
3. 「Generate token」で作成し、表示されたトークンをコピー（再表示不可）

### 設定例

```bash
# .env.mcp に記載
GITHUB_PERSONAL_ACCESS_TOKEN=github_pat_xxxxxxxxxxxxxxxxxxxxxxxx
```

### 注意事項

- Fine-grained PAT は有効期限が切れると動かなくなるため、期限を管理すること
- Copilot HTTP 版（`api.githubcopilot.com/mcp/`）は Organization のプライベートリポジトリに
  アクセスできないため使用しない

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

## Notion Integration Token（読み取り専用）

Notionページの読み取り専用アクセスに使用します。書き込み用と分けることでセキュリティを向上できます。

### 取得手順

1. https://www.notion.so/profile/integrations にアクセス
2. **"新しいインテグレーションを作成"** をクリック
3. インテグレーション名を入力（例: "Claude Code MCP (readonly)"）
4. ワークスペースを選択
5. **権限を「読み取り」のみに設定**
6. **"送信"** をクリック
7. **Internal Integration Token** をコピー（`ntn_`で始まる文字列）
8. Notionで使用したいページを開き、**"..."** メニューから **"接続を追加"** を選択
9. 作成したインテグレーションを選択

### 設定例

```bash
# 環境変数として設定
export NOTION_READONLY_TOKEN=ntn_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# または .env.mcp に記載
NOTION_READONLY_TOKEN=ntn_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 注意事項

- 書き込み用トークン（`NOTION_TOKEN`）とは別のインテグレーションを作成してください
- 読み取り専用なので誤ってデータを変更するリスクがありません

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

- [Notion API Getting Started](https://developers.notion.com/docs/getting-started)
- [Google Cloud Authentication](https://cloud.google.com/docs/authentication/getting-started)
- [BigQuery API](https://cloud.google.com/bigquery/docs/reference/rest)
