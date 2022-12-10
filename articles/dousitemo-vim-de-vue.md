---
title: "どうしてもVimでVueの開発をしたかった話"
emoji: "🤔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vim, vue, languageserver]
published: true
---

[![ストックマーク Advent Calendar 2022](/images/stmk/stmk-advent-calendar2022-banner.jpg)](https://adventar.org/calendars/7542 "ストックマーク Advent Calendar 2022 - Adventar")

# はじめに

これは、[ストックマーク Advent Calendar 2022](https://adventar.org/calendars/7542 "ストックマーク Advent Calendar 2022 - Adventar") 11日目の記事です。こんにちは、ストックマークで[Astrategy](https://stockmark.co.jp/product/astrategy "Astrategy | ストックマーク株式会社")というビジネス向けSaaSについて、主にフロントエンドの開発を担当している[@tsukkee](https://twitter.com/tsukkee)です。

[昨年のAdvent Calendarの私の記事](https://zenn.dev/tsukkee/articles/stmk_advent_calendar_vim)では、Vimを中心とした開発環境の記事を書いたのですが、今年はその環境をなんとか維持した話を書きたいと思います。

ちなみに、弊社はVimのほか、Emacs、IntelliJ IDEA、Cloud9、Visual Studio Codeを使うメンバーも在籍しており多様性があるのですが、最近デザイナーの方の中に[Nova](https://nova.app/jp/)使いの方がいることが発覚し、さらに混迷を極めています。そんな中最近入社された方の中にNeoVimユーザーの方がいて、Vim派が倍増(1人→2人)したという朗報もありました。

# VimでVueを使ったフロントエンド開発をする

さて、弊社ストックマークではフロントエンド開発にVueを採用しています。Vueをはじめとしたフロントエンドフレームワークを使った開発では、ESLintなどのLinter、PrettierなどのFormatterを活用することで、チーム内でコードの一貫性を保ちつつ効率的に開発を進めることができます。そして、さらに効率的なコーディングに欠かせないのがLanguage Serverの設定です。

Vueにおいては永らく[Vetur](https://vuejs.github.io/vetur/)が使われてきましたが、昨年の記事でも少し触れたように[Volar](https://github.com/johnsoncodehk/volar)が登場し、さらには今年1月に[Vue 3 as the new default](https://blog.vuejs.org/posts/vue-3-as-the-new-default.html)になった段階で、

> * Improved TypeScript IDE support for Single File Components via Volar
> * Command line type checking for SFCs via vue-tsc

と述べられており、Volarと、さらにコア部分を共有するvue-tscがVueのエコシステムの中で標準ツールとなりました。今のところ[来年末がVue 2系のEnd of Life](https://vuejs.org/about/faq.html#what-s-the-difference-between-vue-2-and-vue-3)と言われている中で、残念ながら弊社プロダクトはVue 3移行が完了していないのですが(来年のAdvent CalendarではVue 3に移行できた話をしたい…！)、Volarは[こちらの手順](https://github.com/johnsoncodehk/volar/blob/master/docs/installation.md#vue-2)でVue 2でも使うことができます。まだVue 3に移行できていないので…という理由でVolarを諦める必要はないので活用していきましょう。

# ちょいちょい動かなくなる

ということで、VimでもLinter、Formatter、Language Serverを設定できて、Vue開発も快適にできるぜ！となったのですが、[Volarが安定版となったのは今年の10月頭だった](https://blog.vuejs.org/posts/volar-1.0.html)ので、それまでというかその前後まではちょいちょい動かなくなっていました。

ここで、VimでLanguage Serverで使うということは、Language ServerであるVolar、クライアント側の[vim-lsp](https://github.com/prabirshrestha/vim-lsp)、さらにvim-lsp向けに設定を提供している[vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)がそれぞれが正しく動いている必要があります。それぞれちょいちょい私の方からPull Request(どれも数行程度のものですが…)を送り、そしてそれ以上に各プロジェクトの開発者の皆様に助けてもらって今に至ります。

なお、vim-lspを使っていて問題が発生した場合には、以下のようにvim-lspからログを吐くようにしつつ、変なメッセージが出ていないか、さらには[仕様](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/)とにらめっこして齟齬がないかを見ていると原因が特定できることが多いです。

```vim
let lsp_log_verbose=1
let lsp_log_file = expand('~/lsp.log')
```

ということで、以下ざっと私から送ったPull Requestの紹介です。

## Volar

https://github.com/johnsoncodehk/volar/pull/1046

https://github.com/johnsoncodehk/volar/pull/1403

今のところvim-lspとVolarは標準入出力で通信しているため、Volar側で`console.log()`でログが吐かれると標準出力として吐かれ、JSONで通信しているところと混ざって壊れるという問題があったので、`console.warn()`を使って標準エラーの方に出力するようにしました。

https://github.com/johnsoncodehk/volar/pull/1834

vim-lspが`workspaceFolder`という設定への対応がexperimentalなのですが、Volar側が`workspaceFolder`が有効であることが前提のコードになっている箇所があったので修正しました。


このあたりの修正は、Volar側の開発が早く、またちょくちょくリファクタリングも入っているようなので、今は跡形もない気がします笑

## vim-lsp

https://github.com/prabirshrestha/vim-lsp/pull/1379

https://github.com/prabirshrestha/vim-lsp/pull/1382

Volarが1.0になるタイミングで、[初期設定まわりに色々とリファクタリングが入った](https://github.com/johnsoncodehk/volar/pull/1916)とのことなのですが、その中でクライアントから申告されたCapability(Language Serverが提供する機能のうち、どれに対応しているかという情報)に応じて動作を変更するようになりました。クライアント、つまりvim-lspから正しくCapabilityを申告しなければならなくなったのですが、vim-lspがRename機能への対応があるにも関わらず、申告しているCapabilityにその情報が含まれていなかったので修正しました。

しかし、こちらの修正は他のLanguage Serverの挙動も変えることになり、さらにvim-lsp側の修正が必要となってしまい、最終的には[hrsh7thさんに修正していただきました](https://github.com/prabirshrestha/vim-lsp/pull/1387)。大変ありがたい限りです。


# vim-lsp-settings

https://github.com/mattn/vim-lsp-settings/pull/534

https://github.com/mattn/vim-lsp-settings/pull/551

https://github.com/mattn/vim-lsp-settings/pull/571

https://github.com/mattn/vim-lsp-settings/pull/581

https://github.com/mattn/vim-lsp-settings/pull/607

https://github.com/mattn/vim-lsp-settings/pull/619
Volar側が安定版になるまでの間、不定期に破壊的変更が入ってたので、インストールするタイミングによって意図せず壊れたりしないように、vim-lsp-settings側で常にVolarのバージョンを固定するようにしていました。そのため、自分の方でタイミングを見てたまにバージョンを上げるためのPull Requestを送らせていただいてました。

https://github.com/mattn/vim-lsp-settings/pull/627

それがついに、前述したとおり安定版となったため、minor versionまで固定するように変更しています。この記事を書いている段階で、Volarは1.0.12までバージョンが上がっているのですが、今のところ`:LspInstallServer`でVolarを入れ直すだけで適切に最新バージョンが使えるよういなっているので良い感じです。

# さいごに
上記のようにいくつかPull Requestを送っていたのですが、内容としてはVimとVolarを繋ぐためにほんの少しずれていたところを直した程度で、実際にはVolarとvim-lsp、vim-lsp-settingsがそれぞれ素晴しいソフトウェアであることに、VimでのVue開発を支えてもらっているなと感じます。また、上記の各Pull Requestにおいても色々と助けていただき感謝感謝です。

そして、VimでVue開発したい！という人が自分以外に地球に何人いるかは分かりませんが、上記の修正がそういう人たちにちょっとは役に立つと良いなと思います。何より自分が便利なことが一番嬉しいことなので、来年もできる範囲でちょいちょいコントリビュートしていけると良いなと思いました。

# おまけ
昨年のAdvent Calendarでは、AquaSKKをターミナル上で使うことに命を懸けておりました。当時は、Alacrittyのmasterを自力でビルドするしか方法がなかったのですが、その後KittyやWezTermでも日本語入力への対応が進み、AquaSKKを問題なく使えるようになっています。以下のように互換性設定をすると良いです。

![AquaSKKの互換性設定](/images/stmk/aquaskk-compatibilities.png =600x)

また、Alacrittyではundercurlが表示できないという問題もあったのですが、こちらも[対応されており](https://github.com/alacritty/alacritty/issues/1628)、今では問題なく表示できるようになっています。

各ターミナルの開発者の皆様にも感謝！！
