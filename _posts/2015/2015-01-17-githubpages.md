---
layout: page
categories:
- blog
title: Github Pagesで独自ドメイン+プライベートリポジトリもおｋ
---
散々既出のネタだと思いますが・・・・．いろいろハマったので．
よく知られているとは思いますが，[githubにはサイトをホスティングする機能](https://pages.github.com)がついています．

githubのアカウント名をhogeとすると，hogeというリポジトリを作成し，そこにHTMLコンテンツを放りこんでおくと，````http://hoge.github.io````というURLでそのHTMLを公開することができます．
このサイトをUser siteと呼ぶそうです．

そのUser siteのURLのサブパスに，リポジトリごとのページを公開することもできます．
例えば，その他にbarというリポジトリを作り，そこにgh-pagesというブランチを作成しておくと，gh-pagesブランチのコンテンツをそのまま公開することができます．
そのときのURLは，````http://hoge.github.io/hoge/````になります．
このリポジトリ（プロジェクト）ごとのサイトをProject siteと呼ぶそうです．
gh-pagesブランチに置いたコンテンツはたとえ，プライベートリポジトリであってもpublicになるので，注意を要します．

### Jekyll

さらにgithubでは，Jekyllという静的なHTMLコンテンツ生成ツールを使って，サイトをdeployすることもできます．
Jekyllは，Markdown形式で書かれたコンテンツをHTMLファイルに変換するだけのシンプルなサイト生成ツールです．
私もJekyllを使ってこのブログを書いています．
Wordpress等と比べて，databaseも不要ですし，速いし，gitでコンテンツの差分管理もできるので非常に便利です．

### 独自ドメイン

ここまでくると，独自ドメインをgithub