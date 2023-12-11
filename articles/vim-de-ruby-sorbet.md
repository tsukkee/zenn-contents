---
title: "VimにLanguage Serverの設定を追加する方法(RubyのSorbetを例に)"
emoji: "💎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vim", "languageserver", "ruby", "sorbet"]
published: true
---

# はじめに

これは、[ストックマーク Advent Calendar 2023](https://adventar.org/calendars/9363 "ストックマーク Advent Calendar 2023 - Adventar") 11日目の記事です。こんにちは、ストックマークで[Anews](https://stockmark.co.jp/product/anews "Anews | ストックマーク株式会社")と[Astrategy](https://stockmark.co.jp/product/astrategy "Astrategy | ストックマーク株式会社")というビジネス向けSaaSについて、プロダクト開発チームのチームリードをしている[@tsukkee](https://twitter.com/tsukkee)です。[一昨年](https://zenn.dev/tsukkee/articles/stmk_advent_calendar_vim)と[昨年](https://zenn.dev/tsukkee/articles/dousitemo-vim-de-vue)に引き続き、全く空気を読まずにVimの話をしたいと思います。

ちなみに、昨年弊社のVim派が倍増(1人→2人)になったとお伝えしたのですが、その後Emacs派閥の人数が増えついに倍の勢力(4人)を誇るまでになり、Vim派の肩身が狭くなってきているのですが(？)、ちょうど昨日[Yokohama.vim #13 〜飲茶でもつまみながらの会〜](https://yokohamavim.connpass.com/event/302231/ "Yokohama.vim #13 〜飲茶でもつまみながらの会〜 - connpass")に参加することができ、Vim成分を補給できたので私は元気です。

# VimからSorbetのLanguage Serverを使いたい
さて、昨年末ぐらいから私はAnewsというプロダクトの開発に参画しました。そのため、それまではあまり触れてこなかったRailsを使った開発をすることになり、加えて、最近は[Sorbet](https://sorbet.org/)というRubyに型を付けられる仕組みの導入を試験的にはじめています。このようにあらたな言語やフレームワークでの開発が必要になったとき、皆さんは「とにかく対応するLanguage ServerをVimから使えるようにしないとな」と考えるのではないでしょうか。

私の場合は、[vim-lsp](https://github.com/prabirshrestha/vim-lsp)を活用しており、多くの場合は[vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)で各Language Server向けの設定が用意されています。しかし、残念ながらSorbet向けの設定はまだ用意されていなかったので、[自分で用意してみました](https://github.com/mattn/vim-lsp-settings/pull/710)。

この記事では、今後また別のLanguage Serverの設定を追加したくなったときの自分向けの備忘録と、他にも同じことをしたい人の一助になればと思い、以下、Sorbetを例にどのようにvim-lsp-settingsに設定を追加すれば良いかを説明したいと思います。

なお、SorbetそのものやSorbetをどう活用していくかについては、またいずれ[弊社テックブログ](https://tech.stockmark.co.jp/)で他のメンバーが紹介してくれると思います！

# vim-lsp-settingsにLanguage Serverの設定を追加する
おおよそ以下の手順で進めると良いかと思います。
1. 方針を考える
2. settings.jsonに情報を追加する
3. installerを用意する
4. settingを追加する
5. 動作確認する
6. READMEを更新する

以下、それぞれ説明していきます。

## 1. 方針を考える
[昨年の記事](https://zenn.dev/tsukkee/articles/dousitemo-vim-de-vue)では、VueのLanguage ServerであるVolar[^1]を追加する話をしました。当時Volarまだ実装が不安定だったので、vim-lsp-settingsの作者であるmattnさんのアドバイスもあり、後述するinstallerの作成時にバージョン固定してインストールすることとし、Volar側のバージョンが上がるたびにvim-lsp-settings側の設定も都度見直して変更することにしていました。このように、扱いたいLanguage Serverの実装がまだ安定していないと思われる場合には、バージョンを固定して使うかどうか、どのぐらいの頻度で設定を見直すかを考えておくと良いです。

既に実装が安定しているLanguage Serverの場合は、特にバージョン固定しないようにすることで、いつでもその時点で最新のLanguage Serverを使うようにすることができます。

Sorbetの場合は、installerでも触れますが、各プロジェクトにBundleでインストールされたコマンドを使うので、ここの考慮は不要でした。

[^1]: 最近では[Volar](https://volarjs.dev/)はLanguage Serverを作るためのフレームワークという位置付けで、[Vue Language Tools](https://github.com/vuejs/language-tools)と呼ぶのが正しそうです。

## 2. settings.jsonに情報を追加する
方針が決まったら具体的に設定を追加していきます。

vim-lsp-settingsでは、[settings.json](https://github.com/mattn/vim-lsp-settings/blob/master/settings.json)にFileTypeごとに使えるLanguage Serverの設定が列挙されており、まずはここに情報を追加する必要があります。

今回はRubyなので、`ruby`セクションに情報を追加します。Sorbetの場合は以下のような情報を追加しました。`command`はLanguage Serverの実行ファイルの名称で、後述するsettingやinstallerの名前の関連付けにも使われます。`url`はLanguage Serverの公式サイトのURLを記述し、`description`は公式サイトなどから適切なキャッチコピーを持ってくると良いです。`requires`はそのLanguage Serverの実行に必要なコマンドを記載します。例えば、JavaScript系のツールならnpmが入ることが多いと思います。`root_uri_patterns`には、プロジェクトのルートディレクトリの手掛かりになるファイル名を記載します。多くの場合、その言語のパッケージマネージャーの設定ファイルを記載することになるかと思います。それ以外にも設定できる項目もあるので、他のLanguage Serverの設定も参考にすると良いです。

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

## 3. installerを追加する
installerは、Linux / macOS向けとWindows向けの2種類を用意する必要があります。それぞれ、installerディレクトリ以下に、`install-{{command名}}.sh`と`install-{{command名}}.cmd`というファイルを作成します。Sorbetの場合はinstall-sorbet.shとinstall-sorbet.cmdです。

ここで、Language Serverには単体で実行ファイルをダウンロードして実行すれば良いものと、プロジェクト内でパッケージマネージャなどを通じてインストールされたものを使うものがあります。このあたりは各Language Serverのドキュメントを読んで想定する使い方を調べておく必要があります。Sorbetについては、[公式ドキュメント](https://sorbet.org/docs/vscode)にLanguage Serverが`bundle exec srb typecheck --lsp`を実行するという記載があり、これは各プロジェクト内でBundleでインストールされたものを使うことを意味します。

installerはイチから自分で書いても良いとは思うのですが、多くの場合同じFileTypeの他のLanguage Serverの設定をコピペしてくると簡単です。Sorbetの場合は、RubocopのLSP Modeがそのまま流用できると判断したので、コピペしてきてrubocopのところをsorbetに置換することにしました。

Linux / macOS向けの場合は以下のようになり、installerと言いながら実際には、各プロジェクトにインストールされたSorbetのコマンド(srbコマンド)をBundleから叩くコマンドを、「sorbet」という名前で定義しています。

```shell
#!/bin/sh

set -e

cat <<EOF >sorbet
#!/bin/sh
TARGET_DIR=\$1
shift
cd \${TARGET_DIR}
bundle exec srb typecheck \$*
EOF

chmod +x sorbet

echo 'Install Done.'
echo '**You need add sorbet dependencies in Gemfile.**'
```

なお、単体で実行ファイルをインストールすることで動かせるLanguage Serverの場合、ここでパッケージマネージャーなどと通じてコマンドをダウンロードさせます。場合によっては、settings.jsonで定義したcommand名に合うように、シンボリックリンクを張ったりリネームしたりする必要があるかも知れません。

## 4. settingを追加する
さらに、setttingsディレクトリ以下に、`{{コマンド名}}.vim`という名前でLanguage Serverをvim-lspに登録する設定を記載します。こちらも、他のLanguage Serverの設定をマネするのが簡単で、基本的には`:LspRegisterServer`というvim-lspに設定を追加するためのユーティリティコマンドを使って、設定を記載します。

また、vim-lsp-settingsでは、vimrcに`g:lsp_settings`という変数を用意することで、ここの設定を各ユーザーで上書きできるようにする必要があるため、`lsp_settings#get()`関数を使います。この関数は、`g:lsp_settings`の値→settings.jsonの値→第3引数のフォールバックの値の優先度で値を取得してくれます。

Sorbet特有の話としては、LSPモードで動かすために`--lsp`オプションが必要なほか、Sorbetではファイル変更検知に[watchman](https://facebook.github.io/watchman/)というコマンドを使っており、それが存在しない場合には`--disable-watchman`オプションを指定するという部分を追加しています。

```vim
function! Vim_lsp_get_watchman_flag()
  if executable('watchman')
    return ''
  endif

  " cf. https://sorbet.org/docs/vscode#installing-and-enabling-the-sorbet-extension
  echo "To watch file changes, watchman is required for sorbet-lsp"
  return '--disable-watchman'
endfunction

augroup vim_lsp_settings_sorbet
  au!
  LspRegisterServer {
      \ 'name': 'sorbet',
      \ 'cmd': {server_info->lsp_settings#get('sorbet', 'cmd', [lsp_settings#exec_path('sorbet'), lsp#utils#uri_to_path(lsp_settings#root_uri('sorbet')), '--lsp', Vim_lsp_get_watchman_flag()])+lsp_settings#get('sorbet', 'args', [])},
      \ 'root_uri':{server_info->lsp_settings#get('sorbet', 'root_uri', lsp_settings#root_uri('sorbet'))},
      \ 'initialization_options': lsp_settings#get('sorbet', 'initialization_options', v:null),
      \ 'allowlist': lsp_settings#get('sorbet', 'allowlist', ['ruby']),
      \ 'blocklist': lsp_settings#get('sorbet', 'blocklist', []),
      \ 'config': lsp_settings#get('sorbet', 'config', lsp_settings#server_config('sorbet')),
      \ 'workspace_config': lsp_settings#get('sorbet', 'workspace_config', {}),
      \ 'semantic_highlight': lsp_settings#get('sorbet', 'semantic_highlight', {}),
      \ }
augroup END
```

なお、上記項目の中で、`initialization_options`というのは、各Language Serverが持つ個別の設定項目です。Sorbetの場合だと、たとえば https://sorbet.org/docs/highlight-untyped#in-other-lsp-clients に言及があり、`"highlightUntyped": true`というオプションが与えられるようになっています。他のLanguage Serverでもドキュメントやあるいはソースコード中に言及があったりするので、見てみると良いかも知れません。

## 5. 動作確認する
ここまで、できれば設定したLanguage Serverが動くようになっているはずです。対応するFileTypeのファイルを開いてから、`:LspInstallServer {{command名}}`でインストール→Language Serverの動作まで問題ないか確認してみましょう。

ここでうまく動作していない場合は、`:LspStatus`コマンドでLanguage Serverが動いているかどうかを確認すると良いです。`starting`で止まっている場合はそもそもうまく起動していないかも知れないので、settingに書いたコマンドや引数に間違いがないかを確認してみてください。

`stopped`になっている場合は何か実行時エラーが発生している可能性があります。vim-lspは、`lsp_log_verbose`と`lsp_log_file`を設定することで、ログをファイルに吐かせることができます。私は[mattnさんの記事](https://mattn.kaoriya.net/software/vim/20191231213507.htm)を参考に`:LspDebug`コマンドを定義し、いつでもデバッグできるようにしています。

```vim
command! LspDebug let lsp_log_verbose=1 | let lsp_log_file = expand('~/lsp.log')
```

Sorbetの場合だと、CodeAction(`:LspCodeAction`)を実行したときにエラーが発生してしまったので、上記手順でログを確認したところ、以下のようなログが確認できました。

```
日 12/10 16:29:27 2023:["<---",1,"sorbet",{"response":{"id":3,"jsonrpc":"2.0","requestMethod":"sorbet/error","error":{"code":-32602,"message":"Unable to deserialize LSP request: Error deserializing JSON message: Invalid value for enum type `CodeActionKind`: "}},"request":{"method":"textDocument/codeAction","jsonrpc":"2.0","id":3,"params":{"context":{"diagnostics":[],"only":["","quickfix","refactor","refactor.extract","refactor.inline","refactor.rewrite"]},"range":{"end":{"character":28,"line":57},"start":{"character":26,"line":57}},"textDocument":{"uri":"file:///path/to/file.rb"}}}}]
```

これは、``Error deserializing JSON message: Invalid value for enum type `CodeActionKind`: ``の部分がポイントで、色々調べた結果、
1. vim-lspは https://github.com/prabirshrestha/vim-lsp/blob/3af8f3b38effc4a631a15bb283a4b701c251275d/autoload/lsp/ui/vim/code_action.vim#L55 でCodeActionのrequest時に`only`に`""`(Empty)というCodeActionKindを含めている
2. 一方で、Sorbetは https://github.com/sorbet/sorbet/blob/4a47295e560d9f7f9f32261bb4b6793793931065/main/lsp/tools/make_lsp_types.cc#L418-L421 でCodeActionKindの定義をしているが、`""`(Empty)が含まれておらず、vim-lspから送信されてきた`""`(Empty)を未知の値としてパースエラーにしてしまっている
3. Language Server Protocolの仕様的には https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#codeActionKind にもあるとおりSorbetが受け取ってくれると嬉しい

ということが分かりました。とりいそぎ https://github.com/sorbet/sorbet/compare/master...tsukkee:sorbet:handle-empty-codeactionkind?expand=1 のようにSorbetを直すとCodeActionが動くようになるのですが、この修正が妥当かどうかはが分からず、SorbetのSlackで質問しているところです(回答来るかドキドキ…)。

それでもなんだか良く分かないこともよくあり、私はvim-jp Slackの#tech-lspチャネルなどで相談させていただくことも多く、今回も色々とアドバイスをいただくことができました。ほんとうにありがたい限りです。

## 6. READMEを更新する
少なくとも、READMEにある[Support Languagesの表](https://github.com/mattn/vim-lsp-settings#supported-languages)は更新する必要があります。

また、Sorbetの場合は、RuboCopのLSPモードと同様、各プロジェクトでのsorbetが必要なほか、watchmanへの依存もあるのでその旨を記載してみました。
```
### [sorbet (Ruby)](https://sorbet.org/docs/vscode)

To use sorbet, you need to install rubocop in your Ruby project using bundler.
Also, [Watchman](https://facebook.github.io/watchman/) is required to watch file changes.
For more details, please see [Sorbet](https://sorbet.org/docs/vscode#installing-and-enabling-the-sorbet-extension) and [Watchman](https://facebook.github.io/watchman/docs/install.html) documentations.
```

ここまでの内容でvim-lsp-settingsにPull Request(PR)を出せば良いはず！SorbetのPRもこの記事執筆時にはまだApprove & Mergeされていないので、上記内容に変更が入るかも知れません。

# おまけ: 複数Language Serverの併用
VSCodeでRubyの開発をする場合、最近は[Ruby Extension Pack](https://marketplace.visualstudio.com/items?itemName=Shopify.ruby-extensions-pack)を使うらしいのですが、この中身を見るとruby-lspとsorbetを併用するかたちになっています。

vim-lsp + vim-lsp-settingsでこれを再現する場合、vimrcに以下のように利用するLanguage Serverを列挙する必要があります。これを記述しておかないと、各FileTypeごとにsettings.jsonではじめに見つかったLanguage Serverしか有効になりません(はじめこれに気付かずちょっと時間を溶かしました…)。

```vim
let g:lsp_settings_filetype_ruby = ['sorbet', 'ruby-lsp']
```

また、vim-lspが提供するコマンドの一部は`--server`オプションでどのLanguage Serverから情報を受け取るかを選ぶことができます。たとえば、`:LspHover --server=sorbet`とすることで、Sorbetから得られた型情報のみを表示することができます。

# おわりに
ちょっと長めの記事になってしまいましたが、Vimに限らずLanguage Serverを活用した開発は今後も広がっていくと思うので、新しいLanguage Serverが登場したときに素早くVimでの開発にも取り込むために、この記事が役に立つと良いなと思っています。

最後に、StockmarkではVimmerでもそうでない方でも、一緒にプロダクトを開発してくれるメンバーを募集中ですので、気になった方は[Stockmark採用ポータル](https://stockmark.wraptas.site/)からお声掛けいただけると嬉しいです！

