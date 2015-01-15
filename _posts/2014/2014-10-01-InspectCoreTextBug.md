---
layout: page
categories:
- blog
title: NSLineBreakByCharWrapping's bug
---

![image](https://raw.githubusercontent.com/sonsongithub/InspectCoreText/master/sample.png)

### 原因は

前回のエントリで書いたCoreTextの謎の挙動だが，大方，原因が特定できた．
どうやら，NSAttributedStringにスタイルとして，lineBreakModeにNSLineBreakByCharWrappingを指定したときに発生するバグのようだ．
図は，lineBreakModeに3つの値を入れたときのレンダリング結果だ．
NSLineBreakByCharWrappingを指定したときのみ，空白が変な場所にレンダリングされて，最後の文字"i"までレンダリングされていないことがわかる．
正しくレンダリングしているlineBreakModeのときも黄色で示しているレンダリングサイズは似たようなエリアを返している．
このことから，このバグは，CTFramesetterSuggestFrameSizeWithConstraintsにはバグがなく，それが原因というより，レンダリングにつかっているCTFrameDrawに起因すると推察することもできる．

とりあえず，Appleにバグレポートを書いたので，8.1とかで修正されるといいのだが・・・・．

### サンプルコード

[https://github.com/sonsongithub/InspectCoreText](https://github.com/sonsongithub/InspectCoreText)