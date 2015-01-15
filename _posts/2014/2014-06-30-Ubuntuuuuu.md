---
layout: page
categories:
- blog
title: Ubuntuｪｪｪｪｪ
---

### UbuntuでPAC+排他リストでプロキシを設定したい

デフォルトのプロキシ設定だと，プロキシの自動設定ファイル(PAC)にすると，プロキシを使わずに接続するサーバのリストを入力できない・・・・．
みたいなので，dconf-editorをインストールし，それを使って，設定する．

    > apt-get install dconf-editor

### USB3.0＋仮想環境でカメラが動かない

Virtual Box, VMWare Fusion, VMPlayerでUbuntuの64bitを動かすと，USBカメラがちゃんと動かない・・・・．
ホスト側のマシンでUSB3.0をBIOSでオフにすると，動くという報告もあるみたいです．
はまった．