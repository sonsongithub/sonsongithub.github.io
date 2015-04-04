---
layout: page
categories:
- blog
title: cocoapods変わってる・・・・・．
---

### 登録

<blockquote class="twitter-tweet" lang="en"><p><a href="https://twitter.com/sonson_twit">@sonson_twit</a> podspecの追加・変更はだいぶ前に trunk という仕組みを使うように変更されましたよ。 <a href="http://t.co/Kqc2mDlMBN">http://t.co/Kqc2mDlMBN</a> PRでの変更は受け付けてもらえません。</p>&mdash; kishikawa katsumi (@k_katsumi) <a href="https://twitter.com/k_katsumi/status/584232728440152066">April 4, 2015</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### 既存リポジトリの所有権	

[claim](https://trunk.cocoapods.org/claims/new)

0. CocoaPodsが入ってない場合は入れてください。
Specのバリデーションなどで必要になるので。もしまだ入ってなければ別途手順を書きます。

1. Podspecを書く。
上記のプルリクエストで書いたようなのが標準的なものです。
Podで配布されているプロジェクトをいくつか見ると感じがわかります。僕のを参考にしてもらってもいいです。
リポジトリ自体が標準的なディレクトリ構成になってるほうが楽ですが、そこは適当でも何とかなります。

2. 配布するコミットにバージョン番号のタグを付ける
Podはタグでバージョンを識別するのでタグを付けてください。
v1.0.0か、単に1.0.0が標準的です。
タグを付けるには`git tag v1.0.0`して`git push --tags`です。

3. Podspecを検証する - CocoaPodsにはPodspecを検証するコマンドが入っています。
aa
aa

    pod spec lint UZTextView

Podspecのあるディレクトリで上記を実行してエラーがなければプルリクエストしてOKです。

4. プルリクエストを出す
https://github.com/CocoaPods/Specs をforkします。

forkしたリポジトリをcloneします。
オリジナルのリポジトリを別名で登録します。

    git remote add upstream git@github.com:CocoaPods/Specs.git

最新までrebaseします。

    git pull --rebase upstream master

プルリクエスト用のブランチを作ります。

    git checkout -b UZTextView100

Pod名+バージョンでディレクトリを作ります。
コミットします。

    git commit -m 'Add UZTextView podspec.'

「自分の」リポジトリにPushします。

    git push origin UZTextView100

Githubの自分のリポジトリからPull Requestを送ります。

### podspec変換

    pod ipc spec LibraryName.podspec