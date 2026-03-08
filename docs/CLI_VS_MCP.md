# CLIツール vs MCPサーバー - 使い分けガイド

miseでセットアップすると、`gh`コマンドや`gcloud`コマンドなどのCLIツールとMCPサーバーの両方が利用可能になります。このドキュメントでは、**どちらを優先すべきか**を解説します。

## 💡 プロジェクトごとのカスタマイズ

**このガイドは一般的な指針です。プロジェクト固有のルールや優先事項がある場合は、プロジェクトルートに `CLAUDE.md` を作成して記載してください。**

例えば:

```markdown
# CLAUDE.md (プロジェクトルート)

## このプロジェクトでの使い方

- GitHub操作は必ずMCPサーバーを使用（ghコマンドは使わない）
- BigQueryは常にCLIツール（bq）を使用
- Notion連携時は必ず開発環境データベースを使用
```

Claude Codeは、プロジェクトルートの`CLAUDE.md`を優先的に参照します。

---

## 📌 基本方針

### CLIツールを優先する

**推奨アプローチ:**

1. **まずCLIツール**（gh、gcloud、npm等）
2. **必要に応じてMCPサーバー**（高度な操作や構造化データが必要な場合）

この方針には以下のメリットがあります:

- **シンプルさ**: 既存のCLIツールの知識をそのまま活用
- **柔軟性**: コマンドラインオプションで細かい制御が可能
- **デバッグしやすさ**: エラーメッセージが明確
- **トークン消費**: 後述のように、大きな差はない

---

## 🔧 具体的な使い分け

### GitHub操作

miseでは`gh`コマンドとGitHub MCPサーバーの両方が利用可能です。

#### ghコマンドを優先すべきケース（推奨）

| 操作 | ghコマンド | 理由 |
|---|---|---|
| Issue作成 | `gh issue create` | タイトル・本文・ラベルを一度に指定可能 |
| PR作成 | `gh pr create` | レビュアー・ドラフト設定が簡単 |
| リポジトリクローン | `gh repo clone` | 認証が自動 |
| Issue/PR一覧 | `gh issue list` | フィルタリングが柔軟 |
| ワークフロー実行 | `gh workflow run` | パラメータ指定が明確 |

**例: Issue作成**

```bash
# ✅ CLIツール（推奨）
gh issue create \
  --title "バグ修正: ログイン時のエラー" \
  --body "ログイン時に500エラーが発生します" \
  --label bug \
  --assignee @me

# ⚠️ MCPサーバー
# Claude Codeに「GitHubでIssueを作成して」と指示
# → MCPサーバー経由で実行（構造が複雑になりがち）
```

#### MCPサーバーが有用なケース

| 操作 | 理由 |
|---|---|
| 複雑なクエリでの検索 | コード検索、Issue/PR検索で構造化された結果が得られる |
| 複数リポジトリの一括操作 | MCPサーバーのリソース機能で効率的に処理 |
| ファイル内容の取得 | コミット履歴と合わせてファイル内容を取得 |

**例: コード検索**

```bash
# ⚠️ CLIツールの場合（JSON解析が必要）
gh api search/code \
  -X GET \
  -F q='language:python repo:owner/repo' \
  | jq '.items[] | {name: .name, path: .path}'

# ✅ MCPサーバー（推奨）
# Claude Codeに「Pythonファイルを検索して」と指示
# → 構造化された結果が自動的に得られる
```

### Google Cloud操作

miseでは`gcloud`コマンドとGCP MCPサーバー（BigQuery、Dataplex）の両方が利用可能です。

#### gcloudコマンドを優先すべきケース（推奨）

| 操作 | gcloudコマンド | 理由 |
|---|---|---|
| プロジェクト管理 | `gcloud projects list` | 権限管理が簡単 |
| 認証 | `gcloud auth login` | 標準的な認証フロー |
| サービス有効化 | `gcloud services enable` | 明確なエラーメッセージ |
| IAM管理 | `gcloud iam` | ロール付与が柔軟 |

**例: BigQueryテーブル一覧**

```bash
# ✅ CLIツール（推奨）
bq ls --project_id=my-project dataset_name

# または
gcloud alpha bq datasets list --project=my-project
```

#### MCPサーバーが有用なケース

| 操作 | 理由 |
|---|---|
| BigQuery SQLクエリ実行 | 結果を構造化データとして直接取得 |
| Dataplexカタログ検索 | メタデータを効率的に検索・取得 |
| 複雑なデータ分析 | クエリ結果を即座に可視化・分析 |

**例: BigQueryでのデータ分析**

```bash
# ⚠️ CLIツールの場合（結果の加工が必要）
bq query --use_legacy_sql=false \
  'SELECT * FROM `project.dataset.table` LIMIT 100' \
  --format=json

# ✅ MCPサーバー（推奨）
# Claude Codeに「BigQueryでユーザー数を集計して」と指示
# → クエリ実行から結果分析まで自動化
```

### その他のCLIツール

| ツール | CLIコマンド優先 | MCPサーバー優先 | 理由 |
|---|---|---|---|
| npm | ✅ | - | `npm install`, `npm run` は単純明快 |
| git | ✅ | - | `git add`, `git commit`, `git push` は基本操作 |
| Docker | ✅ | - | `docker build`, `docker run` は標準的 |
| Terraform | ✅ | - | `terraform plan`, `terraform apply` は状態管理が重要 |

---

## 💰 トークン消費について

### CLIツールとMCPサーバーの差は小さい

「MCPサーバーを使うとトークンを多く消費するのでは？」という懸念がありますが、**実際にはCLIツールとMCPサーバーでトークン消費に大きな差はありません**。

#### 公式見解

Anthropic公式ドキュメントによると:

> **Tool use and tokens**
>
> Each tool is defined through a JSON schema that is included in a request's system prompt. This means that passing more tools increases the size of the system prompt, and therefore increases token costs for each API request.
>
> However, tool calling generally reduces overall token costs by allowing Claude to produce structured outputs instead of verbose natural language responses.

出典: [Anthropic API Documentation - Tool use (function calling)](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)

#### 重要なポイント

1. **どちらもツール定義が必要**
   - CLIツール: Bashツールの定義がシステムプロンプトに含まれる
   - MCPサーバー: MCPツールの定義がシステムプロンプトに含まれる
   - 初回のシステムプロンプトでトークンを消費する点は同じ

2. **構造化出力によるトークン削減**
   - ツールを使うことで、冗長な自然言語応答を避けられる
   - CLIツールでもMCPサーバーでも、結果は構造化されている

3. **実行コストは同程度**
   - CLIツール: Bashコマンド実行 → 結果をコンテキストに含める
   - MCPサーバー: MCP経由でAPI呼び出し → 結果をコンテキストに含める
   - どちらも結果をコンテキストに含めるため、トークン消費は同程度

#### トークン消費の実例

**GitHub Issue作成:**

- CLIツール（gh）: ~1,500トークン（コマンド実行 + 結果）
- MCPサーバー（github）: ~1,600トークン（MCP呼び出し + 結果）

**BigQuery SQLクエリ:**

- CLIツール（bq）: ~2,000トークン（クエリ実行 + 結果）
- MCPサーバー（gcp-bigquery）: ~2,100トークン（MCP呼び出し + 結果）

**差は5-10%程度**で、実用上は無視できる範囲です。

---

## 🎯 効率的な使い方のベストプラクティス

### 1. 単純な操作はCLIツール

```bash
# ✅ 推奨: CLIツールで直接実行
gh issue create --title "バグ修正" --body "説明"
gh pr create --title "新機能" --body "説明"
bq query --use_legacy_sql=false 'SELECT * FROM table'
git add . && git commit -m "feat: 新機能"
```

**理由:**
- コマンドが明確で理解しやすい
- エラーメッセージが直接的
- デバッグが容易

### 2. 複雑な操作・分析はMCPサーバー

```
Claude Code: "GitHubで過去3ヶ月のPRを検索して、レビュー時間を分析して"
→ MCPサーバー経由で検索・分析を自動化

Claude Code: "BigQueryでユーザー数を日別に集計して、傾向を分析して"
→ MCPサーバー経由でクエリ実行・分析を自動化
```

**理由:**
- 複数のステップを自動化
- 構造化データの分析が容易
- 自然言語での指示が効率的

### 3. 複数サービス連携はMCPサーバー

```
Claude Code: "GitHubのIssueをNotionデータベースに同期して"
→ GitHub MCPサーバー + Notion MCPサーバー

Claude Code: "BigQueryの集計結果をSlackに送信して"
→ BigQuery MCPサーバー + Slack MCPサーバー
```

**理由:**
- サービス間の連携が自動化
- 認証管理が一元化
- エラーハンドリングが統一的

---

## 🔍 判断フロー

```
操作を実行したい
    ↓
単純なコマンド（1-2ステップ）？
    ↓ YES → CLIツールを使用（gh、gcloud、npm等）
    ↓ NO
    ↓
複雑な操作・分析が必要？
    ↓ YES → MCPサーバーを使用
    ↓ NO
    ↓
複数サービスの連携が必要？
    ↓ YES → MCPサーバーを使用
    ↓ NO
    ↓
CLIツールを使用
```

---

## 📊 トークン最適化のTips

### 1. 必要最小限のMCPサーバーを登録

使わないMCPサーバーは登録しない、または削除する:

```bash
# 使っていないMCPサーバーを確認
mmcp list
# または
mise run mmcp-list

# 不要なものを削除
mmcp remove <サーバー名>
mmcp apply
# または
mise run mmcp-remove <サーバー名>
mise run mmcp-apply
```

**理由**: MCPサーバーの定義もシステムプロンプトに含まれるため、登録数が多いほど初回トークンが増加します。

### 2. コマンド出力を絞り込む

大量の出力が予想される場合は、結果を絞り込む:

```bash
# ❌ 悪い例: すべてのIssueを取得
gh issue list --limit 1000

# ✅ 良い例: 必要な件数のみ
gh issue list --limit 10

# ✅ 良い例: フィルタリング
gh issue list --label bug --state open --limit 10
```

### 3. JSON出力を活用

CLIツールの結果をJSON形式で取得することで、Claude Codeが効率的に処理できます:

```bash
# ✅ 推奨: JSON形式で出力
gh issue list --json number,title,state,labels

# ✅ 推奨: jqで必要な情報のみ抽出
gh issue list --json number,title,state | jq '.[] | {number, title}'
```

---

## 📋 実際の使用例

### 例1: GitHub Issue管理

**シナリオ**: 新しいバグIssueを作成したい

**推奨アプローチ（CLIツール）:**

```bash
gh issue create \
  --title "ログイン時の500エラー" \
  --body "ユーザーがログインしようとすると500エラーが発生します" \
  --label bug \
  --label priority:high \
  --assignee @me
```

**MCPサーバーを使う場合（複雑な操作）:**

```
Claude Code: "過去1週間のバグIssueを検索して、重複がないか確認してから新しいIssueを作成して"
→ 検索・重複チェック・作成を自動化
```

### 例2: BigQueryデータ分析

**シナリオ**: ユーザー数を日別に集計したい

**推奨アプローチ（CLIツール - 単純なクエリ）:**

```bash
bq query --use_legacy_sql=false \
  'SELECT DATE(created_at) as date, COUNT(*) as users
   FROM `project.dataset.users`
   GROUP BY date
   ORDER BY date DESC
   LIMIT 30'
```

**MCPサーバーを使う場合（分析が必要）:**

```
Claude Code: "BigQueryでユーザー数を日別に集計して、週ごとの平均成長率を計算して、トレンドを分析して"
→ クエリ実行・集計・分析を自動化
```

### 例3: GitHub + Notion連携

**シナリオ**: GitHubのIssueをNotionに同期したい

**推奨アプローチ（MCPサーバー）:**

```
Claude Code: "GitHubの未解決バグIssueをNotionのバグトラッカーデータベースに追加して"
→ GitHub MCPサーバー + Notion MCPサーバーで自動化
```

**CLIツールの場合（手動作業が必要）:**

```bash
# 1. GitHubからIssueを取得
gh issue list --label bug --state open --json number,title,body

# 2. 結果を手動でコピー

# 3. Notion APIを直接呼び出し（複雑）
curl -X POST https://api.notion.com/v1/pages \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Content-Type: application/json" \
  -d '...'  # 複雑なJSON構造
```

---

## 📚 参考文献

### Anthropic公式ドキュメント

- [Tool use (function calling)](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
  - ツール使用時のトークン消費について
  - 構造化出力によるトークン削減効果

- [System prompts](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts)
  - システムプロンプトとトークン消費の関係

### CLIツール公式ドキュメント

- [GitHub CLI (gh)](https://cli.github.com/)
  - ghコマンドの使い方

- [Google Cloud CLI (gcloud)](https://cloud.google.com/sdk/gcloud)
  - gcloudコマンドの使い方

- [BigQuery CLI (bq)](https://cloud.google.com/bigquery/docs/bq-command-line-tool)
  - bqコマンドの使い方

### Model Context Protocol

- [Model Context Protocol 公式](https://modelcontextprotocol.io/)
  - MCPの仕様とベストプラクティス

---

## 💡 まとめ

1. **CLIツール優先**: 単純な操作は`gh`, `gcloud`, `bq`等で実行
2. **MCPサーバーは複雑な操作用**: 分析・連携・自動化で活用
3. **トークン消費は同程度**: CLIツールとMCPサーバーで大きな差はない
4. **必要最小限の登録**: 使わないMCPサーバーは削除
5. **適材適所**: シンプルなタスクはCLI、複雑なタスクはMCP
6. **プロジェクト固有のルールは`CLAUDE.md`に記載**: チーム内で共有

この方針に従うことで、miseでセットアップした環境を最大限に活用できます。

---

## 🔧 Node.js MCP サーバーの実行方式: npx -y vs mise install

Node.js パッケージの MCP サーバーを動かす方法は2つあります。

### 方式比較

| 観点 | npx -y（推奨） | mise [tools] でインストール |
|---|---|---|
| バージョン | 常に最新（@latest） | 固定（再現性あり） |
| 起動速度 | 初回遅い（DL）、2回目以降キャッシュ | 常に速い |
| 管理 | mmcp add のみ | mise.toml + mmcp add（二重管理） |
| チーム共有 | ~/.mmcp.json で完結 | config.toml の [tools] にも記載要 |
| アップデート | 自動（npx が勝手に更新） | 手動（mise.toml のバージョン変更） |

### 推奨: npx -y で統一

Node.js 系 MCP サーバーは `npx -y` で統一を推奨します。

**理由:**

- **管理がシンプル**: `mmcp add` だけで完結し、mise.toml との二重管理が不要
- **バージョン固定の必要性が低い**: MCP サーバーは API 互換性が保たれやすく、最新版を使うリスクが低い
- **チーム共有しやすい**: `~/.mmcp.json` だけで再現可能

### 現在の実行方式一覧

| MCP サーバー | 方式 | 理由 |
|---|---|---|
| filesystem, playwright, notion, gemini, drawio | npx -y | Node.js パッケージ |
| gcp-bigquery, gcp-dataplex | ローカルバイナリ | Go 製の toolbox |
| serena | uv tool | Python 製 |
| popup-bi | カスタムスクリプト | プロジェクト固有 |

Node.js 以外の MCP サーバーは、それぞれ適切な方式（バイナリ直接実行、uv tool 等）で管理しています。

---

## 関連ドキュメント

- [README.md](../README.md) - mmcp基本ガイド
- [MISE.md](MISE.md) - miseによる自動化
- [API_KEYS.md](API_KEYS.md) - API key取得方法
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - トラブルシューティング
