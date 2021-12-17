---
title: "Vimを支える技術: Alacritty, AquaSKK, tmux, Language Server… 高速ウェブ開発の世界"
emoji: "🖊️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vim, alacritty, aquaskk, tmux, languageserver]
published: false
---

[![ストックマーク Advent Calendar 2021](/images/stmk/stmk-advent-calendar2021-banner.jpg)](https://adventar.org/calendars/6612 "ストックマーク Advent Calendar 2021 - Adventar")

# はじめに

これは、[ストックマーク Advent Calendar 2021](https://adventar.org/calendars/6612 "ストックマーク Advent Calendar 2021 - Adventar") 17日目の記事です。こんにちは、ストックマークで[Astrategy](https://stockmark.co.jp/product/astrategy "Astrategy | ストックマーク株式会社")というビジネス向けSaaSについて、主にフロントエンドの開発を担当している[@tsukkee](https://twitter.com/tsukkee)です。

Astrategyの技術構成については以前に[Astrategyを支える技術: gRPC, Elasticsearch, Cloud TPU, Fargate... SaaS型AIサービスの内側の世界](https://tech.stockmark.co.jp/blog/astrategy_overview/ "Astrategyを支える技術: gRPC, Elasticsearch, Cloud TPU, Fargate... SaaS型AIサービスの内側の世界 - Stockmark Tech Blog")という弊社テックブログ記事で紹介したことがあるのですが、本記事ではその開発環境の一部を紹介したいと思います。

さて、開発環境と言えばテキストエディタですが、皆さん開発にはどのテキストエディタ(またはIDE)を使っていますでしょうか？本記事のタイトルにもあるとおり私はVimを使っています。ただ、Astrategyの開発チームでは使うテキストエディタに制限はなく、また多様性を重視する文化なので、Emacs、IntelliJ IDEA、Cloud9、Visual Studio Code、そしてVimと開発チームの5人が5人とも別のエディタを使って開発しています。

さらに、弊社では心理的安全性が担保されているため、俗に言うエディタ戦争のようなものも起こりません。その証拠に、このAdvent Calendarの9日目にも[Emacsの記事](https://qiita.com/mkt3/items/44aa25619a9591078231 "Emacsのすゝめ - Qiita")が書かれていますし、Slackのカスタム絵文字は以下のように設定されています(私を採用してくれたEmacs使いの当時の上司が設定したものです)。素晴しいですね！

![](/images/stmk/custom_emoji.png =423x)

ということで、以降では私のVim環境 on macOSについて紹介したいと思います。なお、私のこのあたりの設定ファイルはだいたい[GitHubで公開している](https://github.com/tsukkee/config "tsukkee/config: My dotfiles.")ので興味のある方はあわせてご覧ください。

# なぜVimを使うのか？
そもそもなぜVimを使っているのかは語り出すと色々あるのですが、おおまかにはプログラムのソースコード(MarkdownやLaTeXなどによるドキュメントも含む)を編集することに特化した機能が多く搭載されているとことと、全ての操作がキーボードで完結して素早く操作できるところかなと思います。

具体的な例としては、テキストオブジェクトという機能があり、様々な単位でテキストの固まりをオブジェクトとして扱って編集できる機能があります。例えば、以下のようなテキストがあり`|`にカーソルがあったときに、

```javascript 
funciton hoge(arg1, |arg2) {
```

`di(`と入力すると`d`が消す、`i`が内側、`(`で丸カッコということで、`()`内のどこにカーソルがあっても一気に引数部分を消して以下のような状態にすることができます。

```javascript
funciton hoge(|) {
```

ソースコードを書いていると、このようにテキストを固まりで扱うことが多いのでこれは大変役に立つ機能です。Vimを学ぶでもっともオススメの書籍である[Practical Vim](https://pragprog.com/titles/dnvim2/practical-vim-second-edition/)(日本語版: [実践Vim](https://www.amazon.co.jp/%E5%AE%9F%E8%B7%B5Vim-%E6%80%9D%E8%80%83%E3%81%AE%E3%82%B9%E3%83%94%E3%83%BC%E3%83%89%E3%81%A7%E7%B7%A8%E9%9B%86%E3%81%97%E3%82%88%E3%81%86-Drew-Neil/dp/4048916599))にも「Edit Text at the Speed of Thought(思考のスピードで編集しよう!)」というキャッチフレーズが付いていますが、正にこの感覚を得られるところが素晴しいところだと感じています。

# 環境構築のポイント
このように素晴しいVimをCUIで使おうと思うのですが、CUIと言えばなんだか真っ黒で味気ない画面を想像する方もいるかも知れません。しかし、昨今のターミナルはTrue color(RGBの24bit color)表示ができるので、以下で詳細を述べますが、Alacritty + tmux + VimでTrue color表示できる環境を構築します。

また、VimでもLanguage Serverを活用することでVisual Studio Codeなど他のテキストエディタと遜色ないコーディング環境を作ることができるので、ここではvim-lspというLanguage Serverクライアントプラグインを軸に、Formatter, Linter, Fuzzy Finderが連携できるように設定します。

最終的には以下のような感じになります。

![](/images/stmk/my_vim.png =800x)

# ターミナル
Vimを使う上では、CUI上のVimを使うのか、GUI版のgVim(macOSなので[MacVim](https://macvim-dev.github.io/macvim/))を使うのかという選択肢があるのですが、私は他のCUIツールとの連携を重視してCUI版のVimを使っています。なお、[Neovim](https://neovim.io/)という選択肢もありますが、私はなんやかんやで15年以上はVimを使っており、今のところ積極的に移行する理由がなく、Neovimと同様に現在も活発に開発が続いているVimを使っています。

その上で、私は日本語入力に[AquaSKK](https://github.com/codefirst/aquaskk "codefirst/aquaskk: An input method without morphological analysis.")を使っています。ところが、この記事を執筆している段階でAquaSKKで良い感じに日本語入力できるターミナルは[Alacritty](https://github.com/alacritty/alacritty "alacritty/alacritty: A cross-platform, OpenGL terminal emulator.")のmasterブランチを自力でビルドしたもの(と、iTerm2も次点でギリいける)しか存在しません(筆者調べ)。適当に最新のRustをインストールしておいてから、GitHubからcloneしたディレクトリで

```shell
$ make app
```

するとビルドできるので、それを使います。そして、AquaSKKの環境設定 → 互換性で`io.alacritty`に対して「露払後確定」を設定すれば完璧です！この記事自体もこの設定で書いていますが、特に問題は発生していません。

![](/images/stmk/aquaskk.png =600x)

ちなみに、私が試した範囲では、他のターミナルの状況は以下のとおりです。

* Terminal.app: まずTrue color表示に対応していないので却下。
* [iTerm2](https://iterm2.com/): [この記事](https://mac-ra.com/iterm2-aquqskk-lkey/ "ターミナルや iTerm2 で AquaSKK を使う場合 | 林檎コンピュータ")や[この記事](https://mzp.hatenablog.com/entry/2016/05/15/143636 "iTerm2 + AquaSKK - みずぴー日記")に記載の内容をがんばって設定するとだいたいいけるが、稀に入力切り替えができなくなることがある。
* [Kitty](https://sw.kovidgoyal.net/kitty/): AquaSKKの入力切替は概ね問題ないが、Vimで日本語を書いているとどんどんその行の表示が壊れてくる。入力自体はできていて、他の行にカーソルを移動すると正常な表示に戻りはするが結構辛い。
* [Hyper](https://hyper.is/): AquaSKKを使っていると「あ」「い」「う」「え」「お」が正しく入力できない(「a」「i」「u」「e」「o」とアルファベットになってしまう)。
* リリース版のAlacritty: [こちらのIssue](https://github.com/alacritty/alacritty/issues/1101)で言及されている依存ライブラリの日本語入力対応が入っていないので、AquaSKKに限らず日本語が入力できない。

Alacrittyは`~/.config/alacritty/alacritty.yml`を編集することで設定できます。[Alacrittyリポジトリに雛形がある](https://github.com/alacritty/alacritty/blob/master/alacritty.yml)のでこれをコピーしてきて、必要なところだけコメントを外すのが簡単です。私はあまり多くは設定しておらずだいたい以下のとおりです。フォントは[PremolJP](https://github.com/yuru7/PlemolJP)を利用しており、PremolJP Console NFの方を利用すれば[Nerd Fonts](https://www.nerdfonts.com/)も入るので見た目をかっこよくするのが簡単になります。

```yaml: ~/.config/alacritty/alacritty.yml
window:
  # バグ回避: https://github.com/alacritty/alacritty/issues/4474
  opacity: 0.99999

font:
  normal:
    family: PlemolJP Console NF
  size: 19.0

live_config_reload: true

key_bindings:
  # 日本語キーボードでバックスラッシュを入力する
  - { key: Yen,                                         chars: "\x5C"          }
  - { key: Yen,        mods: Alt,                       chars: "\xA5"          }

  # USキーボードでControl-^を認識させる
  - { key: Key6,       mods: Control,                   chars: "\x1e"          }
```

# ターミナルマルチプレクサ
Alacrittyはウィンドウ分割などの機能は搭載しない方針らしいので、[tmux](https://github.com/tmux/tmux "tmux/tmux: tmux source code")を使います。ちなみに、私はAlacrittyを使わないときでもだいたいターミナルのウィンドウ分割やタブの機能は使わずtmuxに任せています。

さて、AlacrittyはTrue color表示に対応していますが、tmuxにもそれを認識させる必要があります。まずは、Alacritty用のTerminfoをインストールします。[AlacrittyのINSTALL.md](https://github.com/alacritty/alacritty/blob/master/INSTALL.md#terminfo)に書かれているとおり、Alacrittyをcloneしてきたところで、

```shell
$ sudo tic -xe alacritty,alacritty-direct extra/alacritty.info
```

を実行すればOKです。さらに、tmux-256colorのTerminfoも必要なので[tmuxのIssueのコメント](https://github.com/tmux/tmux/issues/1257#issuecomment-581378716)にもあるとおり、

```shell
$ brew install ncurses
$ /usr/local/opt/ncurses/bin/infocmp tmux-256color > ~/tmux-256color.info
$ tic -xe tmux-256color tmux-256color.info
```

でインストールします。その上で、`.tmux.conf`に以下を追記します。

```shell
set-option -g default-terminal 'tmux-256color'
set-option -ga terminal-overrides ',$TERM:Tc'
set-option -ga terminal-overrides ',alacritty:RGB'
```

これによって、tmux側でもTrue colorであることを認識できるようになります。tmuxのその他の設定についてはここでは詳細に述べませんが、[tpm](https://github.com/tmux-plugins/tpm)というtmux用のプラグインマネージャは入れておくと便利です。

# Vim

ようやくVimまで来ました。Alacritty、tmuxとTruc color設定をしてきましたが、さらにVimにも設定が必要です。Vimの`:h xterm-true-color`にだいたい説明がありますが、以下を`.vimrc`に設定します。

```vim: ~/.vimrc
set termguicolors
let &t_8f = "\<Esc>[38;2;%lu;%lu;%lum"
let &t_8b = "\<Esc>[48;2;%lu;%lu;%lum"
```

ちなみに、undercurl(下波線)対応のため`:h undercurl`を参考に以下の設定も追加していますが、私の環境ではAlacrittyではただの下線になってしまうようです。KittyやiTerm2はちゃんと波下線になったと思うので、そこはちょっと残念なところです。

```vim: ~/.vimrc
let &t_Cs = "\e[4:3m"
let &t_Ce = "\e[4:0m"
```

# カラーテーマ

次にカラーテーマですが、せっかくなのでAlacritty、tmux、Vimと同じカラーテーマを適用していきたいと思います。私が知っている範囲では[Nord](https://www.nordtheme.com/ "Nord")や[Dracula](https://draculatheme.com/ "Dracula — Dark theme for 227+ apps")あたりが、様々なアプリ向けに統一されたカラーテーマを提供していて、ここ最近はNordがお気に入りなのでそれを使います。

## Alacritty
[ココ](https://github.com/arcticicestudio/nord-alacritty)の内容を`.alacritty.yml`にコピペします。

## tmux
[こちら](https://www.nordtheme.com/docs/ports/tmux/installation)を参考に、tpmを使ってインストールします。

```shell: ~/.tmux.conf
set-option -g @plugin "arcticicestudio/nord-tmux"
```

## Vim
[こちら](https://www.nordtheme.com/docs/ports/vim/installation)を参考にインストールします。ちなみに、Vimのプラグインマネージャーはいくつか実装があり、Nordのページではわりと広く使われている[vim-plug](https://github.com/junegunn/vim-plug)の設定例が記載されていますが、私は[minpac](https://github.com/k-takata/minpac)というシンプルなものを使っているので、設定は以下のとおりになります。

```vim: ~/.vimrc
call minpac#init()
call minpac#add('arcticicestudio/nord-vim')

let g:nord_bold_vertical_split_line = 1
let g:nord_italic = 1
let g:nord_italic_comments = 1

colorscheme nord
```

ここまでで、以下のような雰囲気で良い感じの表示になります。なお、Vimのstatuslineやtablineまわりは[lightline.vim](https://github.com/itchyny/lightline.vim)を使うと簡単にかっこよく設定することができるので試してみてください。

![](/images/stmk/vim-true-color.png =800x)

# Language Server, Formatter, Linter
ここからは、編集機能の部分について簡単に紹介します。

最近のウェブ開発においては、エディタと開発をサポートするツール群(Language Server、Formatter、Linterなど)は分離されていることがほとんで、そのおかげで開発メンバーは好きなエディタを使いつつ、同じFormatterやLinterを適用することで統制の取れたコードを書くことができます。例えば、AstrategyのフロントエンドはVueで作られているのですが、Language ServerとしてVolarを使おう、FormatterはPrettier、LinterはESLintといったところの合意が取れていれば、あとはeslintrcやprettierrcをリポジトリに入れておけば、どのエディタを使っても問題ない状態にすることができます。

まず、Language ServerのクライアントはVim向けにいくつか実装がありますが、ここではVimでは広く使われていると思われる[vim-lsp](https://github.com/prabirshrestha/vim-lsp "prabirshrestha/vim-lsp: async language server protocol plugin for vim and neovim")を使います。さらに、自動補完用に[asyncomplete.vim](https://github.com/prabirshrestha/asyncomplete.vim "prabirshrestha/asyncomplete.vim: async completion in pure vim script for vim8 and neovim")、そしてスニペット補完用に[vim-vsnip](https://github.com/hrsh7th/vim-vsnip "hrsh7th/vim-vsnip: Snippet plugin for vim/nvim that supports LSP/VSCode's snippet format.")及び[vim-vsnip-integ](https://github.com/hrsh7th/vim-vsnip-integ "hrsh7th/vim-vsnip-integ: vim-vsnip integrations to other plugins.")を導入します。ちょっと長いですが、私の設定の肝はだいたい以下のとおりになっています。だいたい`g`が頭に付いたキーバインドでLanguage Serverのリネームや定義へのジャンプなど便利な機能をすぐに使えるようにしている感じです。

```vim: ~/.vimrc
set completeopt& completeopt+=menuone,popup,noinsert,noselect
set completepopup=height:10,width:60,highlight:InfoPopup

call minpac#add('prabirshrestha/vim-lsp')
call minpac#add('prabirshrestha/asyncomplete.vim')
call minpac#add('prabirshrestha/asyncomplete-lsp.vim')
call minpac#add('mattn/vim-lsp-settings')
call minpac#add('hrsh7th/vim-vsnip')
call minpac#add('hrsh7th/vim-vsnip-integ')

function! s:on_lsp_buffer_enabled() abort
    setlocal omnifunc=lsp#complete
    setlocal tagfunc=lsp#tagfunc
    setlocal signcolumn=yes
    nmap <buffer> gd <plug>(lsp-definition)
    nmap <buffer> gr <plug>(lsp-references)
    nmap <buffer> gi <plug>(lsp-implementation)
    nmap <buffer> ge <plug>(lsp-type-definition)
    nmap <buffer> gR <plug>(lsp-rename)
    nmap <buffer> gA <Plug>(lsp-code-action)
    nmap <buffer> gs <Plug>(lsp-document-symbol-search)

    if &filetype !=# 'vim'
        nmap <buffer> K <plug>(lsp-hover)
    endif
endfunction

augroup vimrc
  autocmd User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END

let g:asyncomplete_auto_popup = 1
let g:asyncomplete_auto_completeopt = 0
let g:asyncomplete_popup_delay = 200
let g:lsp_text_edit_enabled = 1
let g:lsp_signs_enabled = 1

imap <expr> <C-l>   vsnip#available(1)  ? '<Plug>(vsnip-expand-or-jump)' : '<C-l>'
smap <expr> <C-l>   vsnip#available(1)  ? '<Plug>(vsnip-expand-or-jump)' : '<C-l>'
imap <expr> <Tab>   vsnip#jumpable(1)   ? '<Plug>(vsnip-jump-next)'      : '<Tab>'
smap <expr> <Tab>   vsnip#jumpable(1)   ? '<Plug>(vsnip-jump-next)'      : '<Tab>'
imap <expr> <S-Tab> vsnip#jumpable(-1)  ? '<Plug>(vsnip-jump-prev)'      : '<S-Tab>'
smap <expr> <S-Tab> vsnip#jumpable(-1)  ? '<Plug>(vsnip-jump-prev)'      : '<S-Tab>'
```

また、vim-lsp-settingsがインストールされているので`:LspInstallServer`とするだけで、そのとき開いているファイルのfiletypeに応じたLanguage Serverがインストールされ、すぐに使いはじめることができます。Vueについては、デフォルトが[Veturの中身のvls](https://github.com/vuejs/vetur/tree/master/server)になっていますが、最近だと[Volar](https://github.com/johnsoncodehk/volar "johnsoncodehk/volar: ⚡ Explore high-performance tooling for Vue")の方が便利なので`:LspInstallServer volar-server`とすると良いです。なお、vim-lsp-settingsのVolarの設定は私がPRを出していることが多い([これ](https://github.com/mattn/vim-lsp-settings/pull/454)とか[これ](https://github.com/mattn/vim-lsp-settings/pull/492)とか…PR作成にあたっていつもサポート頂いているVimコミュニティの方々に感謝！)ので、気付いたところがあれば教えていただくか、PRを出していただくと良いと思います。

ところで、Visual Studio Code中心に発展してきているLanguage Server界隈ですが、その出自はVimだったりします。[Language Server ProtocolのWiki](https://github.com/microsoft/language-server-protocol/wiki/Protocol-History)にも記載がありますが、C#向けに開発されていたOmniSharpというVimプラグインが使っていた方式を発展させたのがLanguage Server Protocolになっているようです。

話を戻して、FormatterとLinterの適用については、[ALE](https://github.com/dense-analysis/ale "dense-analysis/ale: Check syntax in Vim asynchronously and fix files, with Language Server Protocol (LSP) support")を使うのが簡単です。さらに、[vim-lsp-ale](https://github.com/rhysd/vim-lsp-ale "rhysd/vim-lsp-ale: Bridge between vim-lsp and ALE")と一緒に使うことで、vim-lspと良い感じに連携が取れるようになります。

私のAstrategy開発だと、Vue + TypeScript + SCSSによるフロントエンド開発をメインに、webpackなどの設定でJavaScriptを書くこともあり、バックエンドまわりでPython(Poetry利用)とGoをちょろりと触り、あと実験的なところでRustもすこーしだけ触ることがあるので、だいたい以下の設定でどの言語で対応できるようにしています。

```vim: ~/.vimrc
call minpac#add('dense-analysis/ale')
call minpac#add('rhysd/vim-lsp-ale')

let g:ale_disable_lsp = 1
let g:ale_floating_preview = 1
let g:ale_linters_explicit = 1
let g:ale_fix_on_save = 1
let g:ale_fixers = {
\   'javascript': ['prettier'],
\   'typescript': ['prettier'],
\   'vue': ['prettier', 'eslint', 'stylelint'],
\   'scss': ['prettier', 'eslint', 'stylelint'],
\   'rust': ['rustfmt'],
\   'go': ['gofmt'],
\   'python': ['black', 'isort']
\}
let g:ale_linters = {
\   'typescript': ['eslint', 'vim-lsp'],
\   'vue': ['eslint', 'stylelint', 'vim-lsp'],
\   'scss': ['stylelint'],
\   'rust': ['clippy'],
\}
let g:ale_linter_aliases = {'vue': ['vue', 'typescript', 'scss']}
let g:ale_python_auto_poetry = 1
nmap <C-K> <Plug>(ale_detail)

nmap <buffer> g] <Plug>(ale_next)
nmap <buffer> g[ <Plug>(ale_previous)
```

# Fuzzy Finder
インタラクティブにファイルや行、シンボルなどを検索できるFuzzy Finderと呼ばれる種類のプラグインも入れます。[Vim向けの実装はなんかたくさんある](https://zenn.dev/yutakatay/articles/vim-fuzzy-finder)のですが、私はvim-lspとも連携できる[vim-clap](https://github.com/liuchengxu/vim-clap "liuchengxu/vim-clap: Modern performant fuzzy picker for Vim and NeoVim")を使っています。vim-lspとの連携にはvista.vimも必要なので合わせてインストールします。

```vim
call minpac#add('liuchengxu/vim-clap')
call minpac#add('liuchengxu/vista.vim')

let g:clap_layout = {
\   'relative': 'editor',
\   'width': '70%', 'col': '15%',
\   'height': '40%', 'row': 3
\}
let g:clap_popup_move_manager = {
\ "\<C-N>": "\<Down>",
\ "\<C-P>": "\<Up>",
\ }
let g:clap_preview_direction = 'UD'
```

この状態で、`:Clap tags vim_lsp`とすると、vim-lspが認識するシンボル情報をClap上で絞り込んでジャンプすることができます。

![](/images/stmk/vim-clap.png =800x)

# おわりに
かなり駆け足でしたが、私が普段開発で使っているVim環境について紹介しました。Vimの環境構築に関する記事は世の中にたくさんありますが、Alacritty, AquaSKK, tmux, Vimの組み合わせを網羅しつつ必要十分な設定を紹介している記事はあまりないと思いますので、参考になるところがあればと思います。また、これまでVimを使ったことがないよという方(がこの記事を最後まで読んでくださっているか分かりませんが…)も、この機会にVimに興味を持ってもらえたら嬉しいです。

