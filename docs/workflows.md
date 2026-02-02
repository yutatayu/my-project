# GitHub Workflows ドキュメント

このドキュメントでは、`.github/workflows`ディレクトリ内のワークフロー設定について説明します。

## 目次

1. [Claude Code ワークフロー](#claude-code-ワークフロー)
2. [Claude Code Review ワークフロー](#claude-code-review-ワークフロー)

---

## Claude Code ワークフロー

**ファイル:** `.github/workflows/claude.yml`

### 概要

このワークフローは、Issue や Pull Request のコメントで `@claude` がメンションされた際に、Claude Code を実行します。Claude は、コードレビュー、質問への回答、実装リクエストなどに対応できます。

### トリガー条件

以下のイベントで自動的に起動します:

- **issue_comment** (created): Issue にコメントが作成されたとき
- **pull_request_review_comment** (created): Pull Request のレビューコメントが作成されたとき
- **issues** (opened, assigned): Issue が作成または割り当てられたとき
- **pull_request_review** (submitted): Pull Request レビューが送信されたとき

### 実行条件

ワークフローは、以下の条件のいずれかが満たされた場合にのみ実行されます:

- Issue コメントに `@claude` が含まれている
- Pull Request のレビューコメントに `@claude` が含まれている
- Pull Request レビューに `@claude` が含まれている
- Issue のタイトルまたは本文に `@claude` が含まれている

### 実行環境

- **ランナー:** `ubuntu-latest`
- **権限:**
  - `contents: read` - リポジトリの内容を読み取り
  - `pull-requests: read` - Pull Request を読み取り
  - `issues: read` - Issue を読み取り
  - `id-token: write` - ID トークンの書き込み
  - `actions: read` - PR の CI 結果を読み取り (Claude が CI 結果を確認するために必要)

### ステップ

1. **Checkout repository**
   - `actions/checkout@v4` を使用してリポジトリをチェックアウト
   - `fetch-depth: 1` で最新のコミットのみを取得

2. **Run Claude Code**
   - `anthropics/claude-code-action@v1` を使用して Claude Code を実行
   - シークレット `CLAUDE_CODE_OAUTH_TOKEN` を使用して認証
   - `actions: read` 権限を追加で付与 (CI 結果の読み取りのため)

### カスタマイズオプション (コメントアウト済み)

以下のオプションを有効化することで、ワークフローの動作をカスタマイズできます:

```yaml
# カスタムプロンプトを指定
prompt: 'Update the pull request description to include a summary of changes.'

# Claude の動作と設定をカスタマイズ
claude_args: '--allowed-tools Bash(gh pr:*)'
```

詳細は以下のドキュメントを参照してください:
- https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md
- https://code.claude.com/docs/en/cli-reference

---

## Claude Code Review ワークフロー

**ファイル:** `.github/workflows/claude-code-review.yml`

### 概要

このワークフローは、Pull Request が作成または更新された際に、自動的に Claude によるコードレビューを実行します。

### トリガー条件

以下の Pull Request イベントで自動的に起動します:

- **opened**: Pull Request が作成されたとき
- **synchronize**: Pull Request に新しいコミットがプッシュされたとき
- **ready_for_review**: Pull Request がレビュー準備完了になったとき
- **reopened**: Pull Request が再オープンされたとき

### オプション設定 (コメントアウト済み)

#### 特定のファイル変更時のみ実行

特定のファイルパターンに変更があった場合のみワークフローを実行できます:

```yaml
paths:
  - "src/**/*.ts"
  - "src/**/*.tsx"
  - "src/**/*.js"
  - "src/**/*.jsx"
```

#### 特定の PR 作成者のみ対象

特定のユーザーや初回コントリビューターの PR のみを対象にできます:

```yaml
if: |
  github.event.pull_request.user.login == 'external-contributor' ||
  github.event.pull_request.user.login == 'new-developer' ||
  github.event.pull_request.author_association == 'FIRST_TIME_CONTRIBUTOR'
```

### 実行環境

- **ランナー:** `ubuntu-latest`
- **権限:**
  - `contents: read` - リポジトリの内容を読み取り
  - `pull-requests: read` - Pull Request を読み取り
  - `issues: read` - Issue を読み取り
  - `id-token: write` - ID トークンの書き込み

### ステップ

1. **Checkout repository**
   - `actions/checkout@v4` を使用してリポジトリをチェックアウト
   - `fetch-depth: 1` で最新のコミットのみを取得

2. **Run Claude Code Review**
   - `anthropics/claude-code-action@v1` を使用して Claude Code Review を実行
   - シークレット `CLAUDE_CODE_OAUTH_TOKEN` を使用して認証
   - `code-review` プラグインを使用して自動コードレビューを実行
   - プロンプトで対象の Pull Request を指定

### プラグイン設定

- **plugin_marketplaces:** `https://github.com/anthropics/claude-code.git`
- **plugins:** `code-review@claude-code-plugins`
- **prompt:** `/code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}`

---

## セットアップ方法

これらのワークフローを使用するには、以下のシークレットを設定する必要があります:

1. GitHub リポジトリの Settings → Secrets and variables → Actions に移動
2. `CLAUDE_CODE_OAUTH_TOKEN` という名前で新しいシークレットを追加
3. Claude Code の OAuth トークンを値として設定

詳細なセットアップ手順については、[Claude Code Action のドキュメント](https://github.com/anthropics/claude-code-action)を参照してください。

---

## 使用例

### Claude Code ワークフローの使用例

1. **Issue でのメンション:**
   ```
   @claude このバグを修正してください
   ```

2. **Pull Request のコメントでのメンション:**
   ```
   @claude このコードをレビューしてください
   ```

### Claude Code Review ワークフローの使用例

Pull Request を作成すると、自動的に Claude がコードレビューを実行します。レビュー結果はコメントとして追加されます。

---

## 参考リンク

- [Claude Code Action リポジトリ](https://github.com/anthropics/claude-code-action)
- [使用方法ドキュメント](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
- [CLI リファレンス](https://code.claude.com/docs/en/cli-reference)
