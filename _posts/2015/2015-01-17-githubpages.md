---
layout: page
categories:
- blog
title: Github Pagesで独自ドメイン+プライベートリポジトリもおｋ
---
散々既出のネタだと思いますが・・・・．いろいろハマったので．
よく知られているとは思いますが，[githubにはサイトをホスティングする機能](https://pages.github.com)がついています．

githubのアカウント名をhogeとすると，hoge.github.ioというリポジトリを作成し，そこにHTMLコンテンツを放りこんでおくと，````http://hoge.github.io````というURLでそのHTMLを公開することができます．
このサイトをUser siteと呼ぶそうです．

そのUser siteのURLのサブパスに，リポジトリごとのページを公開することもできます．
例えば，その他にbarというリポジトリを作り，そこにgh-pagesというブランチを作成しておくと，gh-pagesブランチのコンテンツをそのまま公開することができます．
そのときのURLは，````http://hoge.github.io/hoge/````になります．
このリポジトリ（プロジェクト）ごとのサイトをProject siteと呼ぶそうです．
gh-pagesブランチに置いたコンテンツはたとえ，プライベートリポジトリであってもpublicになるので，注意を要します．

プロジェクト用のサイトは，githubが用意するかっこいいテンプレートを使ったも作ることができます．

![image](https://pages.github.com/images/choose-layout@2x.png)

## Jekyll

さらにgithubでは，[Jekyll](http://jekyllrb.com)という静的なHTMLコンテンツ生成ツールを使って，サイトをdeployすることもできます．
Jekyllは，Markdown形式で書かれたコンテンツをHTMLファイルに変換するだけのシンプルなサイト生成ツールです．
私もJekyllを使ってこのブログを書いています．
Jekyllを使ってコンテンツを書いた場合，githubにそれをpushすると自動的にコンテンツがgithubによって公開されます．
Wordpress等と比べて，databaseも不要ですし，速いし，gitでコンテンツの差分管理もできるので非常に便利です．

## 独自ドメインで運用

ここまでくると，独自ドメインでgithub pagesを運用したくなってきます．
（私はDNS等の設定や中身に詳しくないので，詳細は専門書等を参考にしてください．）
githubの解説は[こちら](https://help.github.com/articles/tips-for-configuring-an-a-record-with-your-dns-provider/)です．

### サブドメインの場合

DNSのCNAMEレコードの設定を追加すればよいでしょう．

    www.yourdomain.com -> hoge.github.io
    
### サブドメインじゃない場合

Aレコードで直接，IPで変更するのがいいみたいですが．

www.yourdomain.com -> (hoge.github.ioのIP)

私は，APEX ALIASというのを変更して，対応しました．
ムームードメインとかだと，APEX ALIASを設定できないので，今回Gehirn DNSというサービスにDNSサーバを移行して，設定しました．
Gehirnは，２ドメインまでなら，無料で使えるDNSのサービスです．
無料で使えるDNSはいろいろあると思います．他には，![DOZEN](https://dozens.jp)というのもあるそうです．

DNS側の設定が完了すれば，後は，githubのリポジトリにCNAMEとうファイルを置くだけです．
これは，````hoge.github.io````リポジトリのルートにCNAMEというファイルを置きます．
このファイルは，新しいドメインをだけを書いたファイルです．

    echo www.yourdomain.com > CNAME
    
とでもして作成すればよいでしょう．

## 参考文献

1. [github pages](https://pages.github.com)
2. [Jekyll](http://jekyllrb.com)
3. [静的サイトをgithub.ioでホスティングし、独自ドメインでアクセスする](http://blog.textfile.org/20141014/github)
4. [独自ドメインのページを設置する手順まとめ](https://m.facebook.com/notes/energize-kk/github-pagesで独自ドメインのページを設置する手順まとめ/587095077973125/)
5. [Setting up a custom domain with GitHub Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/)
6. [ムームードメイン+GitHub Pagesで独自ドメインを使う方法](http://hamasyou.com/blog/2014/03/05/github-pages-custom-domain/)
