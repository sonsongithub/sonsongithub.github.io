---
layout: page
categories:
- blog
title: Let's Encryptを使って，サーバとiOSでhttpsで通信できるようにする
---

先日，無料かつ自動でSSLの証明書を導入できるLet's Encryptのパブリックベータが開始されました．
iOS9から，SSL以外の通信や指定ドメイン以外の通信をわざわざ許可するように設定する必要があったり，今後SSL以外の通信を許可しないようにしていくとAppleが豪語しているように．世界の趨勢は，httpsを使うことで一本化されていきそうです．
そんな中，お手軽・無料・自動でSSLの証明書を取得できるLet's Encryptのサービスは，趣味でアプリの開発やサービス運用をしているプログラマの光明になる可能性を秘めています．
日本語の有志の[サイト](https://letsencrypt.jp)もあります．

#### 条件
今回作業した環境は以下です．

 * マシン：AWS EC2
 * OS：Ubuntu LTS
 * サーバ：nginx

私は設定ファイルやjsonファイルの配信くらいにしかサーバを使っていません・・・．

#### 準備
opensslのアップデート，nginx, git, emacsなどをインストールします．

    > apt-get update -y
    > apt-get upgrade -y
    > apt-get install nginx, git, emacs -y
    
そして，gitで公開されているletsencryptをチェックアウトし，完了後すぐに一度実行します．
そうすると，依存関係のあるモジュールが自動的にインストールされます．

    > git clone https://github.com/letsencrypt/letsencrypt
    > cd letsencrypt
    > ./letsencrypt-auto

これで準備完了です．

#### webrootでletsencryptを使う
letsencryptは，[マニュアル](https://letsencrypt.readthedocs.org/en/latest/using.html#installation)にあるようにインストール方法が何種類かあります．
その種類に応じて，letsencryptのクライアントがサーバと通信する方法．そして取得した証明書を自動的に設定する対象が異なります．
私は，単純にファイルを置いてるだけで，nginxを使った自動設定機能がバギーだと書いてあったので，webrootを選択しました．
webrootを使う場合は，letsencryptのクライアントは，ドメインのトップレベルのパスに何かしらファイルを書き出して，それをletsencryptのサーバ再度が読み込んで認証するように見えます．

まず，letsencryptが使うルートフォルダをnginxで公開するように設定します．
これはhttpで通信します．

    # nginxの設定
    server {
	    listen 80;
	    root /var/www/example/;
	    index index.html index.htm;
	
	    server_name www.example.com;
	
	    location / {
	        try_files $uri $uri/ =404;
	    }
	}

では，早速letsencryptで証明書を取得してみましょう．

    ./letsencrypt-auto certonly --webroot -w /var/www/example/ -d www.example.com

#### 証明書を確認する
取得した証明書は，`/etc/letsencrypt`にインストールされます．
取得した証明書の内容は，opensslのコマンドで確認できます．

    openssl x509 -text -noout \
        -in /etc/letsencrypt/live/www.example.com/cert.pem 

#### nginxでSSLの証明書をセットする

    # nginxのhttpsの設定
	server {
	  listen 443;
	  server_name www.exmple.com;
	
	  root /var/www/example/;
	  index index.html index.htm;
	
	  ssl on;
	  ssl_certificate /etc/letsencrypt/live/www.exmple.com/cert.pem;
	  ssl_certificate_key /etc/letsencrypt/live/www.exmple.com/privkey.pem;
	
	  ssl_session_timeout 5m;
	
	  ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
	  ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
	  ssl_prefer_server_ciphers on;
	}
	
nginxを更新してみる．

    > nginx -t
    > nginx -s reload

これで`https://www.exmple.com`にアクセスすれば，暗号化されたhttpsで通信できるはず．

#### 更新してみる
更新は，同じコマンドを実行するだけでよい．
新旧の証明書は，`/etc/letsencrypt/archive/www.example.com/`にアーカイブされる．

#### nginxでiOSでも接続できるようにする - SSL Labsチェックをクリアする
実は，この状態でiOSのSafari等からサーバにアクセスしても，証明書を素直に受け入れてくれません．
これは，SSLの設定がiOSの信用条件を満たしていないことに起因するようです．
私は，ここで，iOSの条件を満たすついでにSSL Labsのテストもいい点が取れるように設定を調整してみることにしました．
SSL Labsのテストは，ブラウザから調べたいサーバのアドレスを入れると，SSL Labsのサーバが指定したサーバに接続し，SSLの安全性等を検証してくれるものです．
今回の作業であれば，SSL Labsでサーバをチェックしても，おそらくBかC程度の評価になるでしょう．

まずは，証明書を変更します．
Let's encryptの指定のファイルではなく，fullchainを使います．
sslの証明書のパスを変更します．

	ssl on;
	ssl_certificate /etc/letsencrypt/live/www.exmple.com/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/www.exmple.com/privkey.pem;
	
次にセキュリティの問題のあるプロトコルをオフにしたりします．
まず，SSL3は使ってはいけないそうです．

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	
サポートする暗号化方法も修正します．

    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';

SSLセッションのキャッシュもセットします．

	ssl_session_cache shared:SSL:5m;
	
これで完了．nginxをリロードして，SSL Labsからサーバを指定して，チェックしてみてください．

![a](https://sonson.s3.amazonaws.com/SSL_Server_Test__api2_sonson_jp__Powered_by_Qualys_SSL_Labs_.png)

#### まとめ
Let's encrypt，なんか使えそうです．
ドキュメントにも書いて有りますが，90日で証明書の期限は切れます．
Let's encrypt運営は，60日くらいで更新しろと言っているので2ヶ月に一度，SSLの更新を行うようにバッチスクリプトをしかけておけばよさそうです．
しかしながら，証明書の作成・更新の回数には，制限があることです．
2015/12/8現在，私は実験で証明書の更新を繰り返したために制限に達成されてしまいました．
この変は，気を家ないとハマる人が続出すると考えられますね・・・．

[![](https://sonson.s3.amazonaws.com/Public_beta_rate_limits_-_Issuance_Tech_-_Let_s_Encrypt_Community_Support.png)](https://community.letsencrypt.org/t/public-beta-rate-limits/4772/3)


