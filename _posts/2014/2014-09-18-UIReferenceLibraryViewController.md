---
layout: page
categories:
- blog
title: UIReferenceLibraryViewController
---

### 単語を調べる

iOS8では，単語を引数として渡して，その単語の意味を調べられるAPIが提供される．
使い方はいたって簡単．

	UIReferenceLibraryViewController *vc
	     = [[UIReferenceLibraryViewController alloc] initWithTerm:@"test"];
	UIPopoverController *popover = [[UIPopoverController alloc] initWithContentViewController:vc];
	
	UIButton *button = sender;
	[popover presentPopoverFromRect:button.frame
                             inView:button.superview
           permittedArrowDirections:UIPopoverArrowDirectionAny
                           animated:YES];
                           
以上のコードだけで，選択したテキストの意味を辞書を使って調べたビューコントローラの結果を下図のように表示できる．

![image]({{ site.url }}/assets/dict01.png)

辞書を登録していない場合は，辞書を選択するビューが表示される．

![image]({{ site.url }}/assets/dict02.png)
