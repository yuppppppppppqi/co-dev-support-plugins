# co-dev-support-plugins

CortexLabの共同開発を支援するClaude Codeプラグインのマーケットプレイスです。

## 収録プラグイン

| プラグイン | 概要 |
| --- | --- |
| [`cortexlab-dup-guard`](plugins/cortexlab-dup-guard/) | 重複開発ガード。バックログ登録・着手の前に、Linearバックログと GitHub のブランチ/PR を横断チェックし、重複・類似作業をアラートする |

## インストール

各自のClaude Code環境で:

```
/plugin marketplace add umiji/co-dev-support-plugins
/plugin install cortexlab-dup-guard@cortexlab-tools
```

セットアップと使い方の詳細は
[`plugins/cortexlab-dup-guard/SETUP.md`](plugins/cortexlab-dup-guard/SETUP.md)
を参照してください。
