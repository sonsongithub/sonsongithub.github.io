---
layout: page
categories:
- blog
title: NSLineBreakByCharWrapping's bug
---

<a href="https://github.com/sonsongithub/InspectCoreText"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://camo.githubusercontent.com/365986a132ccd6a44c23a9169022c0b5c890c387/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f7265645f6161303030302e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_right_red_aa0000.png"></a>

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