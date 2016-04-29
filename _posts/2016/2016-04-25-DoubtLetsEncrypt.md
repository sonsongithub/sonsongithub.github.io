---
layout: page
categories:
- blog
title: Let's Encryptを疑え！信用はお金で買え！
---

### 背景

[Let's Encrypt](https://letsencrypt.org)を使って[LINE Bot](https://developers.line.me)からのcallbackを受けるサーバを作ったがダメで，さらに[AWS API Gateway](http://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/welcome.html)もLet's Encrypt SSLを信用してなくて，大変だったというお話．

Let's Encryptは，無料で使えるSSLの証明書取得サービスである．
自動取得/更新がサーバに実装されており，それのためのサーバ側の更新スクリプトも配布されている．
DNSをハックされる危険性や平文をネットワークに流す危険性が危惧される昨今，無料でSSLの証明書を利用できる趣味でサービスを開発したりするエンジニアの強い味方となっている．

しかしながら，SSLとは，基本的に信用を管理会社に委託する仕組みである．
SSL通信の通信相手が正しいことの根拠は，SSLの発行元が「このサイトを管理している人は確かに○○さんです．私が保証します．」と言っているだけなのである．
なので，信用度の高いSSLの発行元は，実際にクライアントのサイト管理の担当者に会いに行ったりすると聞いたことがある．
つまり，サイトの信用度は，発行元がお金をかけて構築しているものなのである．
その背景を考慮すると，無料のLet's EncryptによるSSLが，有償サービスのSSLよりも信用度が落ちるのは当然のことである．
信用はお金で買うのだ．

### 問題

LINE Botは，Botの処理を実行するサーバにLINE Botがcallbackしてくるという仕組みになっている．
Botは，httpでcallbackしてくる．
LINE側は，LINE Botとcallback先となるサーバはSSLで通信することを動作条件としている．
さっそく，node.jsでhttpのサーバを実装し，Botからのアクセスを待ち受けると，待てど暮らせど，Botからアクセスがこない．

LINE Bot側がLet's Enctyptの証明書を受け付けないことは，知られた問題のようだ．
しかし，一部情報によると最近のアップデートで使えるようになったもある．
SSL，LINE Bot，Let's Encryptについて，調べてみたら・・・・出るわ出るわ・・・・・．

[Googleで検索](https://www.google.co.jp/search?client=safari&rls=en&q=SSL+LINE+Bot+letsencrypt&ie=UTF-8&oe=UTF-8&gfe_rd=cr&ei=19sgV-u1LKyL8Qff24HYBg)

### 切り分け

callback先のサーバのログ，LINE Botのログもないため，一体どこでエラーが起きているのかわからない．

そこで，問題の切り分けを行うため，以下のテストを行う．
AWS API Gatewayは，Amazonのサービスで，Web APIのフロントエンドを簡単に作れるサービスです．
ラムダ式を使ってサーバを使わずにAPIっぽいものを作ったり，実際の処理は他のサーバにディスパッチするだけのAPIを作ったりすることができます．
このAPIは，httpsで公開されるため，今回の目的に適合します．
このテストでは，AWS API Gatewayが私のEC2のサーバのnode.jsで作ったサーバを叩くようにする

+ 自宅のネットワークからcallback URLが正しくレスポンスをかえすか？
 + 返す
+ LTEネットワーク経由でcallback URLが正しくレスポンスをかえすか？
 + 返す
+ EC2のインスタンスからcallback URLが正しくレスポンスをかえすか？
 + 返す
+ AWS API GatewayならばLINE Bot APIはPOSTしてくるのか？
 + 返さない，そもそもAWS API Gatewayのテストが動かない

これは，AWS API Gatewayの問題か・・・・？
というわけで，全面的にLet's Encryptを疑う構成で追加的にAWS API Gatewayをテストする
 
+ AWS API GatewayでLet's EncryptでSSL通信するhttpsを叩く
 + 動かない，そもそもAWS API Gatewayのテストが動かない
+ AWS API GatewayでSSL通信のhttps://github.comを叩く
 + 動く
+ AWS API Gatewayで非SSL通信のURLを叩く
 + 動く
+ AWS API Gatewayで非SSL通信にした自作のcallback URLを叩く
 + 動く

結論．
AWS API GatewayもLet's EncryptのSSLを信用していなかったというオチである．
というわけで，Let's Encryptを使ってLINE Botを作るのは基本的に避けた方が余計な作業発生しない分楽ということだろう．

### 動いた構成

```
LINE Bot -> [https(amazon)] -> AWS API Gateway -> [http] -> EC2
```

### まとめ

やはり，何事もお金が肝心である．
大切なことなのでもう一度いいますね．信用はお金で買うのだ．

### 参考文献
+ [http://qiita.com/git6_com/items/008404506836011af33b](http://qiita.com/git6_com/items/008404506836011af33b)
+ [http://shunirr.hatenablog.jp/entry/2016/04/13/164902](http://shunirr.hatenablog.jp/entry/2016/04/13/164902)