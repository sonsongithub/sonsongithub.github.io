---
layout: page
categories:
- blog
title: Swift 2 (& LLDB) シンポジウムで発表してきた
---

## 議論したかったこと
中でも発表中に言った「Rubyで実行できるCのソースコード」を書かないようにしたいというところが一番訴えたかったところです．
結局，何も考えずにSwiftでiOS/Macのコードを書くと，「Swiftで実行できるObjective-Cのソースコード」になってしまいます．
多分に```if let hoge = hoge() {}```が，そこらじゅうに散らばると思いますが，結局，「Swiftっぽく」を考えないなら，Swiftで書く意味はあまりないと考えます．

この考え方が間違っていないとすると，次に考えるべきは「Swiftっぽい」であり，コードを書くときにどんな風に書けばいいのかという点になると思います．

### Swiftっぽく書くということ
私は，[reddift](https://github.com/sonsongithub/reddift)という，[reddit.com](https://www.reddit.com)のAPIラッパーをちょこちょこ勉強がてら作っています．
このソースコードは，swiftjpでもらったフィードバックを多分に生かしながら，新しいパラダイムを覚えるたびに方向転換するという非常に効率の悪いコーディングスタイルで作っています．
その甲斐あってか，[reddit](https://www.reddit.com)で作ったものを公開したときは，"very swifty"と[コメント](https://www.reddit.com/r/swift/comments/34t3kn/swift_reddit_api_wrapper/)をもらいました．
つまるところ，[reddift](https://github.com/sonsongithub/reddift)に対して，多少はSwiftっぽいという感想を持つエンジニアもいるようです（そのあと，なんかどうでも議論になったので・・・ｇｄｇｄでしたが）．
当時の[reddift](https://github.com/sonsongithub/reddift)で，Swiftらしいといえば，そのときは，`Result<T>`を使って戻り値を返すくらいでした．
現在では，データ型をすべて`struct`に置き換え，value type programmingスタイルに，`map`と`flatMap`を使ってunwrapするように作り変えています．

<div class="github-widget" data-repo="sonsongithub/reddift"></div>

このことから安直に考えると，Swiftっぽいというのは，そういう関数型言語っぽいことを取り入れていくということ？と類推することができますが，アンケートであったとしても，データが少なすぎます．
まぁ，こういうのは多分に主観的な感想なので，データで結論を出す類のものでもないのですが・・・宗教的な議論にもなりそうですし・・・・・
それについて，シンポジウムでも多少議論がありました．

#### シンポジウムのディスカッションの中で
1. Swiftに純粋な関数型というより，強い型制約を使う思想が入っている．
2. つまり型制約を積極的に使って，コーディングすべきなのは間違いない．
3. Swiftが関数型言語かというと，Haskellのような純粋関数型言語ではないし，どう考えても副作用や状態を保持しないといけないUIの要素を多分にコントロールする必要があるので，Swiftを純粋関数型言語として実装するのはナンセンスだ．
4. WWDC2015のあるセッションでも，Appleのエンジニアが，UIKitのクラスをValue Typeでコーディングするようなことは辞めろと指摘している．
5. Swiftらしさを模索しながらコーディングしていくしかない・・・・

これらは，大方賛成です．
まぁ，純粋関数型言語として書くのは違うけど，うまいこと取り入れよう！というように私には感じられます．
プレゼンの中でも私が述べた，「Swiftっぽいコードを書くために」の勉強の流れも，これに割と適合しているんじゃないかと自画自賛します．

1. `!`でunwrapしない
2. できる限り高階関数で実装する
3. できる限り`struct`で実装する
4. `Result<A>`を使う
5. `flatMap`でunwrapする

## その他の記憶に残った話題
シンポジウムの雰囲気は，全員がPlaygroundでコーディングしながら，ディスカッションしていた点がよかった．
このスタイルだと，疑問として投げ掛けられたときにすぐに実行結果やその実装や動きについて，参加者全員が議論できる．
今後，同様のイベントをやるなら，これを定着させていくといいと思った．

#### invarianceについて勉強できた
継承関係があるときに，setとgetの対称関係が崩れるっていう話・・・・だと理解．
covarianceとcontravarianceっていうらしい．
[@soutaro](https://twitter.com/soutaro)さんが，説明用の[サンプルコード](https://gist.github.com/soutaro/4c4facc1ca7dd1528deb)を書いてくれました．
invarianceについては，こんな[資料](https://docs.google.com/presentation/d/1H-cUi2GdBmFE8kBoaf6eogJU0IjafpNDLLaxu3C9CZw/present?slide=id.g10fda602e_10)も紹介されました．

#### エラー処理について議論できた
`try-catch` vs `Result<T>`が議論されました．
私は，`try-catch`は同期時，`Result<T>`は非同期時と考えている．
今回のシンポジウムでは，おおかたの結論は同じところに行き着いたかなぁと考える．

`try-catch`を使うと，どうしても非同期時の例外処理のコードが綺麗に書けないと思う．
`Result<T>`は，同期・非同期時でも両方で綺麗にかけるが，エラーを必ず処理するという制約をコンパイラレベルで導入するように実装されていない．
`Result<T>`は，エラー処理を省くことができる点が短所．
でも，`try-catch`もどんなエラーが飛んでくるのかわからない・・・・・．

このシンポジウムの後も，私の意見は変わらず，`try-catch`は同期時，`Result<T>`は非同期時で使うと思います．
あんまり複雑なコード書くと，可読性とメンテナンス性悪くなるし，実際に，エラー処理を必ず行うことさえ，ちゃんと意識していれば問題はないですから．

## まとめ
コードの書き方としては，Swiftらしくを模索しながら，型制約を意識しながら書くのがいいという感じでしょうか．
オープンソースになったとしても，Apple主導で開発が進むのは間違いないわけで，Swiftが言語として，いろいろ模索しながら開発されていくでしょうから，我々デベロッパもその辺に振り回されながら，やっていくしかないと思われます．

今回のシンポジウムは，パネリストを割とたくさん揃えて，前方でコーディングしながら議論するという，私にとって初めてのスタイルで行われました．
これは，非常によい試みであったと思います．
Playgroundの結果をSNS的に共有できる仕組みがあったら，もっとおもしろいシンポジウムになったんじゃないかと思いました．
また，この手のシンポジウムには，しばらく時間を置いてから（ネタがないもん），参加したいものです．

## スライド
<iframe src="http://www.slideshare.net/slideshow/embed_code/key/fi4Dt1ucKcUjs7" width="640" height="480" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>