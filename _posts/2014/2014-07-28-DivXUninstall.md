---
layout: page
categories:
- blog
title: DivX for Macを削除する
---

いつインストールしたのか忘れたDivX for Macが最近，変な通知を出すようになってきてうっとしい．
しかし・・・削除しようと思っても，アンインストールツールがないし，どこにあるのかもわからない．
ネットで探すと情報があった．

![image]({{ site.url }}/assets/divx_update.jpg)

私は，とりあえず，通知さえ消えればいいので，この[サイト](https://forums.divx.com/divx/topics/divx_update_notifications_still_appearing_on_mac_after_uninstalling)の情報を元にlaunchctlだけ制御した．
以下のコマンドでDivX周りの自動起動を停止できる．

    launchctl unload /Library/LaunchAgents/com.divx.update.agent.plist
    launchctl unload /Library/LaunchAgents/com.divx.dms.agent.plist

divxがlaunchctlでロードされるかは，launchctlコマンドで確認する．

    launchctl list | grep div

### 全部消したい場合は・・・

自己責任でお願いするが，すべて消したい場合は，本体，起動スクリプト，および設定ファイルを削除すればよい．
おそらく，````sudo````しないと消せない．

まず，DivX本体を削除する．

    rm -rf /Library/Application\ Support/DivX/

次に，launchctlの起動plistを削除する．

    rm /Library/LaunchAgents/com.divx.update.agent.plist
    rm /Library/LaunchAgents/com.divx.dms.agent.plist

最後に設定ファイルを削除する．ログインユーザ毎に実行しないといけないかもしれない．

    rm $HOME/Library/Preferences/com.divx.*