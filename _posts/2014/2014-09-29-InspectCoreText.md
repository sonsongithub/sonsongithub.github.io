---
layout: page
categories:
- blog
title: Can CoreText estimate text frame? 
---

<a href="https://github.com/sonsongithub/InspectCoreText"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://camo.githubusercontent.com/365986a132ccd6a44c23a9169022c0b5c890c387/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f7265645f6161303030302e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_right_red_aa0000.png"></a>

![image](https://raw.githubusercontent.com/sonsongithub/InspectCoreText/master/sample.png)

### おかしい？

CoreTextでテキストをレンダリングするとき，典型的な例として，以下のような流れがあると考える．

1. テキストを用意する．
2. NSAttributedStringを用意する．
3. CTFramesetterを作成する．
4. CTFramesetterSuggestFrameSizeWithConstraintsでテキストのレンダリングサイズを計算する．
5. （レンダリングサイズに基づいてビューをリサイズする）
6. レンダリングサイズに基づいてCTFrameを作成する．

例えば，

    NSString *string = @"abcdef  \ngh \n ";
    
という文字列をレンダリングすることを考えよう．
本来ならば，図の一番上のgoodの例のようにレンダリングされるはずです．
まずは，どうでもいいメソッドは，以下のように実装する．

    - (void)setAttributedString:(NSAttributedString *)attributedString {
        _attributedString = attributedString;
        [self update];
    }
    
    - (void)drawRect:(CGRect)rect {
        CGContextRef context = UIGraphicsGetCurrentContext();
        
        // attribute
        [[UIColor yellowColor] setFill];
        CGContextFillRect(context, _contentRect);
        [self drawStringRectForDebug];
        
        // draw text
        CGContextSaveGState(context);
        CGContextTranslateCTM(context, 0, _contentRect.size.height);
        CGContextScaleCTM(context, 1.0, -1.0);
        CGContextSetTextMatrix(context, CGAffineTransformIdentity);
        CTFrameDraw(_frame, context);
        CGContextRestoreGState(context);
    }

そして，この````update````というメソッドが肝となる．
````update````メソッドでは，文字列が入力されると，````CTFrame````と````CTFrameSetter````を作成する．
この````update````メソッドの中では，````CTFramesetterSuggestFrameSizeWithConstraints````が返すサイズに基づき，````CTFrame````を作成する．．
これが・・・・罠・・・・．
下のコードのように````CTFramesetterSuggestFrameSizeWithConstraints````が返すサイズをそのまま使ってしまうと，図の真ん中のbadのようなレンダリング結果が得られてしまうのだ・・・．

    - (void)update {
        SAFE_CFRELEASE(_framesetter);
        SAFE_CFRELEASE(_frame);
        
        CFAttributedStringRef p
             = (__bridge CFAttributedStringRef)_attributedString;
        if (p) {
            _framesetter = CTFramesetterCreateWithAttributedString(p);
        }
        else {
            p = CFAttributedStringCreate(NULL, CFSTR(""), NULL);
            _framesetter = CTFramesetterCreateWithAttributedString(p);
            CFRelease(p);
        }
        
        CGFloat constrainedWidth = self.frame.size.width;
        CGSize frameSize
             = CTFramesetterSuggestFrameSizeWithConstraints(
                _framesetter,
                CFRangeMake(0, _attributedString.length),
                NULL,
                CGSizeMake(constrainedWidth, CGFLOAT_MAX),
                NULL
            );
        _contentRect = CGRectZero;
        _contentRect.size = frameSize;
        
        CGMutablePathRef path = CGPathCreateMutable();
        CGPathAddRect(path, NULL, _contentRect);
        _frame
            = CTFramesetterCreateFrame(_framesetter, CFRangeMake(0, 0), path, NULL);
        CGPathRelease(path);
        
        [self setNeedsDisplay];
    }

図の一番上のような結果を得るには，````CTFramesetterSuggestFrameSizeWithConstraints````に指定した幅をサイズとして入力する必要がある．

        CGFloat constrainedWidth = self.frame.size.width;
        CGSize frameSize
             = CTFramesetterSuggestFrameSizeWithConstraints(
                _framesetter,
                CFRangeMake(0, _attributedString.length),
                NULL,
                CGSizeMake(constrainedWidth, CGFLOAT_MAX),
                NULL
            );
        _contentRect = CGRectZero;
        _contentRect.size = frameSize;
        _contentRect.size.width = constrainedWidth;

これは，実は，````CTFramesetterSuggestFrameSizeWithConstraints````が返すサイズが改行の前にある半角，全角スペースをないものとして処理してしまうためである．
CTFrameレンダリングするときには当然空白をレンダリングするためのスペースが必要なのだが，````CTFramesetterSuggestFrameSizeWithConstraints````は，横幅として行の最後にある空白を無視したサイズを返すため，空白が入り切らず，図の真ん中のbadのようなレンダリング結果が得られるのである．

本末転倒だが，````CTFramesetterSuggestFrameSizeWithConstraints````が返す横幅ですべてのコンテンツをレンダリングしたいなら，以下のように二度サイズを計算すればよい．不毛だが．

    CFAttributedStringRef p
        = (__bridge CFAttributedStringRef)_attributedString;
    if (p) {
        _framesetter = CTFramesetterCreateWithAttributedString(p);
    }
    else {
        p = CFAttributedStringCreate(NULL, CFSTR(""), NULL);
        _framesetter = CTFramesetterCreateWithAttributedString(p);
        CFRelease(p);
    }
    
    CGFloat width = self.frame.size.width;
    
    CGSize frameSize = CGRectZero;

    frameSize = CTFramesetterSuggestFrameSizeWithConstraints(
        _framesetter,
        CFRangeMake(0, _attributedString.length),
        NULL,
        CGSizeMake(width, CGFLOAT_MAX),
        NULL
    );
    
    _contentRect = CGRectZero;
    _contentRect.size = frameSize;
    CGFloat temp = frameSize.width;
    
    frameSize = CTFramesetterSuggestFrameSizeWithConstraints(
        _framesetter,
        CFRangeMake(0, _attributedString.length),
        NULL,
        CGSizeMake(temp, CGFLOAT_MAX),
        NULL
    );
    
    _contentRect = CGRectZero;
    _contentRect.size = frameSize;
    frameSize.width = temp;
    
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathAddRect(path, NULL, _contentRect);
    _frame
        = CTFramesetterCreateFrame(_framesetter, CFRangeMake(0, 0), path, NULL);
    CGPathRelease(path);
    
さて，この動作は多分仕様なんだろうが・・・・・．
どう受け止めればいいのやら．

### サンプルコード

[https://github.com/sonsongithub/InspectCoreText](https://github.com/sonsongithub/InspectCoreText)