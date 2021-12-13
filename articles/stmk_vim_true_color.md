---
title: "Vimで"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vim]
published: false
---

この記事はStockmarkアドベントカレンダー2021の17日目の記事です。

Stockmarkでは開発に使うエディタに制限はなく、各々好きなエディタを使って開発しています。その結果、筆者が所属するAstartegyの開発チームでは、5人が5人とも別のエディタを使って開発しています。

また、エディタ戦争のようなものも起こらず心理的安全性が担保されています。その証拠にSlackのカスタムリアクションは以下のように設定されています。



さて、その中で筆者はVimを使って開発しています。

最近のウェブ開発においては、エディタと開発をサポートするツール群(Language Server、Formatter、Linterなど)は分離されていることがほとんで、そのおかげで開発メンバーは好きなエディタを使いつつ、同じFormatterやLinterを適用することで統制の取れたコードを書くことができます。

例えば、AstrategyのフロントエンドはVueで作られているのですが、Language ServerとしてVolarを使おう、FormatterはPrettier、LinterはESLintといったところの合意が取れていれば、あとはeslintrcやprettierrcをリポジトリに入れておけば、あとはどのエディタを使っても問題ない状態にすることができます。


