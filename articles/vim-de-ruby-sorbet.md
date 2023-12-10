---
title: "どうしてもVimでRubyの開発をしたかった話、というかvim-lsp-settingsにsorbetを追加した話"
emoji: "💎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vim", "ruby", "sorbet", "lsp"]
published: false
---

# はじめに

これは。。。




# Sorbetを使う

ALEにはsorbetを扱える仕組みがあるので参考になる。

1. 方針を考える


2. settings.jsonに情報を追加する
今回はRubyなので、`ruby`セクションに情報を追加する。
`description`は公式サイトなどから適切なキャッチコピーを持ってくると良いです。

```json
{
  "command": "sorbet",
  "url": "https://sorbet.org/",
  "description": "Sorbet is a fast, powerful type checker designed for Ruby.",
  "requires": [
    "bundle"
  ],
  "root_uri_patterns": [
    "Gemfile"
  ]
}
```

3. installerを追加する
settingsのcommandと同じ名前にinstallerを付けた名前のファイルを作ります。

Language Serverには単体で実行ファイルをダウンロードして実行すれば良いものと、プロジェクト内でインストールされたものを使うものがあります。
このあたりは各Language Serverのドキュメントを読んで想定する使い方を調べる。
Sorbetについては、Bundleでインストールされたものを使うことになっている。
vim-lsp-settingsの既存のものだとRubocopのLSP Modeがこの方式なので、コピペしてきてrubocopのところをsorbetに置換する。

READMEにその旨追記しておけると良さそう。
なお、Linux / macOSとWindows向けで2つinstallerを作成する必要があるが、もし手元に環境がない場合は他の方にお願いすると良い(Windows環境はmattnさん自身がやってくださる…ありがたいがいり)。

3. settingを作成する



4. 複数のLSPを併用するときは…
vimrcに設定を書かないとどれか1つしか選ばれない…
ruby-lspがあまり仕事していない気もする…?



ruby-lspのDocumentFormatはRuboCopがあればそれを呼ぶ実装になっているので、使うpluginを減らしたい場合はそっちにFormatやらせても良いかも
