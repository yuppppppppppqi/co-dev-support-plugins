# Duplicate Development Guard

チームの共同開発向け・**重複開発防止プラグイン**です。
Linearのバックログと GitHub のブランチ/PR を横断検索し、

- これから登録しようとしているバックログの**類似Issueの有無**
- これから作ろうとしている機能が**既に別ブランチで実装されていないか**
- 類似Issueの**ステータスと担当者**(相方が既に進めていないか)

をチェックしてアラート(🔴 重複あり / 🟡 類似あり / 🟢 重複なし)を出します。

## 構成

| コンポーネント | 役割 |
| --- | --- |
| `agents/duplicate-detector.md` | 調査担当のサブエージェント。Linear MCP + git/GitHub を読み取り専用で検索し、日本語のアラートレポートを返す |
| `/dev-dup-protect:dup-check <説明>` | 事前チェックのみ。レポート提示後にチェック済みマーカーを記録 |
| `/dev-dup-protect:backlog-add <説明>` | チェック → 問題なければ(または確認の上で)Linear MCP経由でIssue登録。類似Issueは説明文に「関連」としてリンク |
| `/dev-dup-protect:start-task <Issue ID or 説明>` | 着手前チェック。類似Issueの担当者・進行状況と既存ブランチの実装有無を確認し、OKなら自分へのアサイン+In Progress化を提案 |
| `hooks/dup-guard.sh` (PreToolUse) | **機械的ガード**。Linear MCPのIssue作成ツール呼び出しを検知し、直近45分以内にこのプロジェクトで重複チェックが記録されていなければ作成を**ブロック**(プロンプト頼みにしない) |

チェック記録は `~/.claude/.dev-dup-protect/checked-<プロジェクトごとのキー>`
にマーカーとして保存され、45分で失効します。

## 前提

- Claude Code に **Linear MCP** が接続されていること:

  ```bash
  claude mcp add --transport http linear https://mcp.linear.app/mcp
  ```

  (初回はブラウザでLinear認証。チームメンバー各自の環境で実行)
- GitHub側はローカルの `git` があれば動作。`gh` CLI か GitHub MCP があれば
  PR情報も見ます。

## インストール(あなたと知人、それぞれの環境で)

このリポジトリ(main反映後)をマーケットプレイスとして追加:

```
/plugin marketplace add umiji/co-dev-support-plugins
/plugin install dev-dup-protect@co-dev-tools
```

対象プロジェクトのリポジトリに移す場合は、`plugins/dev-dup-protect/` と
ルートの `.claude-plugin/marketplace.json` をそのままコピーすれば、同様に
`/plugin marketplace add <owner>/<repo>` で入ります(プラグインは
自己完結で、このリポジトリ内の他ファイルに依存しません)。

ローカルでの動作確認は clone して:

```
/plugin marketplace add /path/to/repo
/plugin install dev-dup-protect@co-dev-tools
```

## 使い方(想定フロー)

1. **バックログ登録前** — `/dev-dup-protect:backlog-add 通知機能のメール対応`
   → 類似Issue・類似ブランチのレポート → 問題なければそのままLinearに登録。
   コマンドを使わず「これLinearに登録して」と頼んだ場合も、フックが作成を
   止めてチェックを強制します。
2. **開発着手前** — `/dev-dup-protect:start-task CTX-42`
   → 類似Issueのステータス/担当者、既存ブランチの実装有無を確認 →
   OKならアサイン+In Progress化+Issue IDを含むブランチ名を提案。
3. **単なる確認** — `/dev-dup-protect:dup-check 検索のインクリメンタル化`

## 調整ポイント

- **別ワークスペースのLinearも検索したい場合**: 環境変数
  `DUP_GUARD_LINEAR_API_KEY` に、そのワークスペースのPersonal API keyを
  セットすると、接続中のLinear MCPセッション(別ワークスペース)に加えて
  直接GraphQL APIでも検索します。詳細は [SETUP.md](../../SETUP.md) 参照
- **ガードの一時無効化**: 環境変数 `DUP_GUARD_DISABLE=1`
- **有効期限**: `hooks/dup-guard.sh` の `TTL_SECONDS`(既定 2700 = 45分)
- **フックの対象ツール**: `hooks/hooks.json` の `matcher`
  (既定は `mcp__.*[Ll]inear.*[Cc]reate.*[Ii]ssue.*` — Linear MCPの
  issue作成系ツール名にマッチ。サーバー名を変えている場合はここを調整)
- ブランチ名に Issue ID を入れる運用(`feat/CTX-42-...`)にすると
  ブランチ⇔Issueの突き合わせ精度が上がります。

## 設計メモ

- エージェントは**読み取り専用**(Issue作成・更新はコマンド側の明示フローのみ)。
- 「チェックしてから登録」はプロンプトのお願いではなく PreToolUse フックで
  **機械的に**強制。マーカーはチェック実施後にのみ記録する規約です。
- Linear MCP未接続などで確認できなかった情報源は、レポートに「未確認」と
  明記されます(暗黙のパスなし)。
