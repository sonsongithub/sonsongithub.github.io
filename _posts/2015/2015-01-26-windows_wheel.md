---
layout: page
categories:
- blog
title: Windowsのスクロールホイールの向きを逆にする
---

仕事でWindowsを使うことがあるのですが（使いたくないけど），一番気持ち悪いのがスクロールホイールの向きがMacと違うところなのである．
今まで，抜本的な解決方法がわからなかったのですが，レジストリをいじる方法を知ったので，拡散のためにも書き留めておきます．
詳しくは，以下のサイトで知りました．

[丸井綜研ーWindows 7のマウスホイールを上下逆に](http://marui.hatenablog.com/entry/20120505/1336196908)

Windowsでレジストリエディタを起動し，キー，FlipFlopWheelを検索し，その値を1にセットするだけ．
そうすると，マウスでのホイール操作の向きがすべて反転します．
特定のマウスデバイスのみを反転させたい場合は，そのデバイスIDを調べて，キーを設定する必要があるようです．
私は，このブログの著者と同じく，全部反転したかったので，FlipFlopWheelの値を頭からすべて検索し，すべて値を1にセットしました．
快適です．