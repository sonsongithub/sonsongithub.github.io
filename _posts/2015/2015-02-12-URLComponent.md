---
layout: page
categories:
- blog
title: NSURLComponentsとNSURLQueryItem
---

先日気付いてしまった．
iOS8に，こんなに便利なクラスが追加されていたとは・・・．

iOS7は，NSStringのメソッドでパーセントエスケープを実行するとなぜかデフォルトだと&(アンバッサンド)をエスケープしてくれなくてブチ切れるという，妖怪不祥事案件で言う所のいわゆるクエリ付きのURLにきちんとアクセスできないというヤツを引きおこしていました．

この二つの使い方は，至って簡単．
まずは，URLの生成．

    NSURLQueryItem *item1 = [NSURLQueryItem queryItemWithName:@"key1" value:@"hoge"];
    NSURLQueryItem *item2 = [NSURLQueryItem queryItemWithName:@"key2" value:@"あいうえお"];
    NSURLComponents *components = [[NSURLComponents alloc] initWithString:@"https://www.hoge.com/service"];
    components.queryItems = @[item1, item2];
    [[UIApplication sharedApplication] openURL:[components URL]];

NSURLQueryItemを名前と値で生成して，NSURLComponentsのqueryItemsに配列で突っ込むだけ．
非常に簡単だ！

    NSURL *URL = nil;
    {
        NSString *value1 = @"ほげええええ";
        NSString *value2 = @"$FD#@#&";
        NSURLQueryItem *registerItem = [NSURLQueryItem queryItemWithName:@"value1" value:value1];
        NSURLQueryItem *callbackItem = [NSURLQueryItem queryItemWithName:@"value2" value:value2];
        NSURLComponents *components = [[NSURLComponents alloc] initWithString:@"https://www.hoge.com"];
        components.queryItems = @[registerItem, callbackItem];
        URL = [components URL];
    }
    {
        NSLog(@"%@", URL.absoluteString);
        NSURLComponents *components = [[NSURLComponents alloc] initWithURL:URL resolvingAgainstBaseURL:YES];
        for (NSURLQueryItem *item in components.queryItems) {
            NSLog(@"%@=>%@", item.name, item.value);
        }
    }

NSURLComponentsにNSURLを突っ込んで生成するだけ．後は，勝手にパースして、クエリのオブジェクトも生成してくれる．
このコードを実行すれば，以下のような結果になるはず．

    https://www.hoge.com?value1=%E3%81%BB%E3%81%92%E3%81%88%E3%81%88%E3%81%88%E3%81%88&value2=$FD%23@%23%26
    value1=>ほげええええ
    value2=>$FD#@#&

便利じゃ・・・・．なんでももうちょっと早く提供してくれなかったのか・・・・．
もうなんだか，色々そろってきたなぁ．