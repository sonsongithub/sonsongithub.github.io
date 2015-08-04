---
layout: page
categories:
- blog
title: Swiftで最速foldrを実行するなら？
---

Haskellの勉強をちょこちょこやってるので，```foldr```と```foldl```をSwiftに実装してみたくなった．ご存知のように，Swiftの```CollectionType```には，すでに```reduce```が実装されており，```foldl```については，実装する必要性がない．
だけど，寂しいので，再帰，ループと色々実装方法で```CollectionType```の```extension```として実装してみた．
差分として，[@norio_nomura](https://twitter.com/norio_nomura)に速度について突っ込まれたので計ってみた．

	extension CollectionType {
	    func foldr_recursive<T>(accm:T, f: (Self.Generator.Element, T) -> T) -> T {
	        var g = self.generate()
	        func next() -> T {
	            return g.next().map {x in f(x, next())} ?? accm
	        }
	        return next()
	    }
	    
	    func foldr_reduce<T>(accm:T, f: (T, Self.Generator.Element) -> T) -> T {
	        return self.reverse().reduce(accm, combine:f)
	    }
	    
	    func foldr_loop<T>(accm:T, f: (Self.Generator.Element, T) -> T) -> T {
	        var result = accm
	        for temp in self.reverse() {
	            result = f(temp, result)
	        }
	        return result
	    }
	}
	
さらに```Array```を```reverse()```するとき，ただの```CollectionType```の```extension```として実装すると，```reverse()```時に愚直に逆方向の配列をコピーしてしまうため，それをさけるために，以下のように，別の```extension```も実装しておく．
	
	// following code is written by @norio_nomura
	extension CollectionType where Index : RandomAccessIndexType {
	    func foldr_loop2<T>(accm:T, @noescape f: (Self.Generator.Element, T) -> T) -> T {
	        var result = accm
	        for temp in self.reverse() {
	            result = f(temp, result)
	        }
	        return result
	    }
	}

計測テストのコードは，以下のようになる．
配列のサイズを100, 1000, 10000, 100000, 1000000個にし，それぞれ1000回ずつ計算し，その計算時間を測定した．
Playgroundで測定したところ，最適化フラグの設定ができない点，単純にコードだけを実行していないように見えたので，OSXのユニットテストで実行速度を計測した．

計測コードは[こちら](https://gist.github.com/sonsongithub/b897f516005f53bc3748)．

<img src="http://sonson.s3.amazonaws.com/foldr-1.png" width="50%"/>

```recursive```は配列のサイズが大きくなると，スタック食いつぶして実行できませんでした．
また，ダントツで遅いのが，reduce(```self.reverse().reduce(accm, combine:f)```)をラップした場合でした．これは，```SequenceType```の```reverse()```が毎回律義に実行されて，逆順の配列が毎回生成されているからだと考察します．

結果，foldrを実装するなら，loop2，つまり，```extension CollectionType where Index : RandomAccessIndexType```で実装したタイプのfoldrが速いという結果になり，これが一番イイと考える．