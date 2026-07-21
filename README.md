# co-dev-support-plugins

チームの共同開発を支援するClaude Codeプラグイン集(マーケットプレイス
リポジトリ)です。`.claude-plugin/marketplace.json` が公開するプラグインは
以下の通りです。

## プラグイン一覧

### [dev-dup-protect](plugins/dev-dup-protect/)

重複開発防止プラグイン。バックログ登録・開発着手の**前**に、Linearの
バックログとGitHubのブランチ/PRを横断チェックし、重複・類似があれば
アラートを出します。うっかり防止として、Linearへのissue作成を「重複
チェック済み」になるまで自動でブロックするフックも備えます。

詳しい導入手順・使い方は [plugins/dev-dup-protect/README.md](plugins/dev-dup-protect/README.md)
と [SETUP.md](SETUP.md) を参照してください。

## インストール

```
/plugin marketplace add umiji/co-dev-support-plugins
/plugin install dev-dup-protect@co-dev-tools
```

ローカルでの動作確認は clone して:

```
/plugin marketplace add /path/to/repo
/plugin install dev-dup-protect@co-dev-tools
```

## 構成

```
.claude-plugin/marketplace.json   # このリポジトリが公開するプラグインの一覧
plugins/
  dev-dup-protect/            # 重複開発防止プラグイン本体(自己完結)
SETUP.md                          # dev-dup-protect のセットアップ&使い方ガイド
```

新しいプラグインを追加する場合は `plugins/<plugin-name>/` に自己完結した
形で置き、`.claude-plugin/marketplace.json` の `plugins` 配列に登録します。
