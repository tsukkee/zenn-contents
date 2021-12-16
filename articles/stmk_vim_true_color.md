---
title: "Vimを支える技術: LSP, tmux, AquaSKK, Alacritty… 楽しいモノレポ開発の世界"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vim, tmux, aquaskk, alacritty]
published: false
---

# はじめに

[![ストックマーク Advent Calendar 2021](/images/stmk/stmk-advent-calendar2021-banner.jpg)](https://adventar.org/calendars/6612 "ストックマーク Advent Calendar 2021 - Adventar")

これは、[ストックマーク Advent Calendar 2021](https://adventar.org/calendars/6612 "ストックマーク Advent Calendar 2021 - Adventar") 17日目の記事です。

こんにちは、ストックマークで[Astrategy](https://stockmark.co.jp/product/astrategy "Astrategy | ストックマーク株式会社")というビジネス向けSaaSの開発を担当している[@tsukkee](https://twitter.com/tsukkee)です。

私が開発を担当しているAstrategyについて、以前に[Astrategyを支える技術: gRPC, Elasticsearch, Cloud TPU, Fargate... SaaS型AIサービスの内側の世界 - Stockmark Tech Blog](https://tech.stockmark.co.jp/blog/astrategy_overview/ "Astrategyを支える技術: gRPC, Elasticsearch, Cloud TPU, Fargate... SaaS型AIサービスの内側の世界 - Stockmark Tech Blog")という弊社テックブログ記事で全体構成は紹介したことがあるのですが、本記事ではその開発環境の一部を紹介したいと思います。

さて、開発環境と言えばテキストエディタですが、皆さん開発にはどのテキストエディタ(またはIDE)を使っていますでしょうか？本記事のタイトルにもあるとおり私はVimを使っているのですが、Astrategyの開発チームでは使うテキストエディタに制限はなく、また多様性を重視する文化なので、Emacs、IntelliJ IDEA、Cloud9、Visual Studio Code、そしてVimと開発チームの5人が5人とも別のエディタを使って開発しています。ちなみに、最近は[JetBrains Fleet](https://www.jetbrains.com/fleet/ "JetBrains Fleet: The Next-Generation IDE by JetBrains")や[Zed](https://zed.dev/ "Zed")といった新しいエディタも登場しつつあり、楽しみにしています。

また、弊社では心理的安全性が担保されているため、俗に言うエディタ戦争のようなものも起こりません。その証拠にSlackのカスタム絵文字は以下のように設定されています。私を採用してくれたEmacs使いの当時の上司が設定したものです。素晴しいですね！

![](/images/stmk/custom_emoji.png =423x)

ということで、以降では私のVim環境について紹介したいと思います。なお、私のこのあたりの設定ファイルはだいたい[GitHubで公開している](https://github.com/tsukkee/config "tsukkee/config: My dotfiles.")ので興味のある方はあわせてご覧ください。

# ターミナル
突然ですが、筆者は日本語入力にSKK、macOS環境ではAquaSKKを使っています。ところが、AquaSKKは




最近のウェブ開発においては、エディタと開発をサポートするツール群(Language Server、Formatter、Linterなど)は分離されていることがほとんで、そのおかげで開発メンバーは好きなエディタを使いつつ、同じFormatterやLinterを適用することで統制の取れたコードを書くことができます。

ちなみに、VSCode中心に発展してきているLanguage Server界隈ですが、その出自はVimだったりします。https://github.com/microsoft/language-server-protocol/wiki/Protocol-History にも記載がありますが、

例えば、AstrategyのフロントエンドはVueで作られているのですが、Language ServerとしてVolarを使おう、FormatterはPrettier、LinterはESLintといったところの合意が取れていれば、あとはeslintrcやprettierrcをリポジトリに入れておけば、あとはどのエディタを使っても問題ない状態にすることができます。

そのほか、バックエンドではPythonやGolangが使われているのですが、今のところ以下のようなツールを使うようにしています。

Python

Golang


さらに

ちなみに、macOSで開発しているため、以下その前提になっていますが他のプラットフォームでも応用可能なところもあるかと思います。



