---
layout: page
categories:
- blog
title: TestFlight経由のインストールのワナ
---

先日，ついにAppleからサービス提供されるようになったTestFlightですが，早速，面倒くさい小さいワナが発見されました．
テスター側から見たTestFlightの作業は以下です．

1. TestFlightをインストール
2. TestFlightで使うAppleIDを開発者に通知
3. 招待状メールを受け取る
4. 招待状メールにかかれたリンクを踏む．
5. TestFlightアプリが起動する．
6. テストのためにアプリをインストール．

ここで！招待状メールをGmailアプリで受け取ってしまうと，その**メールに書かれたリンクをChromeで開いて**しまい，TestFlightにテストを登録できないのです・・・・・・．
なぜか，Chromeでリンクを開くと，AppStoreのTestFlightが開かれてしまうのです・・・・．
なので，Gmailアプリ + Chromeを使っている人は，一度AppStoreを開いてから，Chromeに切り替え，アドレスバーのURLをコピーしてSafariに貼り付けて，URLを開き，TestFlightを開くようにしましょう．

面倒くさいですね．