---
layout: page
categories:
- blog
title: Swiftでfoldrを作ってみた．reduceが速かった．
---

Haskellの勉強をちょこちょこやってるので，```foldr```と```foldl```をSwiftに実装してみたくなった．
再帰，ループと色々実装方法が有るが，[@norio_nomura](https://twitter.com/norio_nomura)に速度について突っ込まれたので計ってみた．
```CollectionType```の```extension```として実装してみた．

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

ご存知のように，Swiftの```CollectionType```には，すでに```reduce```が実装されており，```foldl```については，実装する必要性がない．
だけど，寂しいので実装してみた．
計測テストのコードは，以下のようになる．
配列のサイズを10, 100, 1000個にし，それぞれ4回ずつ計測し，計算時間を平均した．


	func gett() -> Double {
	    var time:timeval = timeval(tv_sec: 0, tv_usec: 0)
	    gettimeofday(&time, nil)
	    return Double(time.tv_sec) + Double(time.tv_usec) / 1000000.0
	}
	
    let t1 = gett();
    data.foldr_recursive(1) { (x, accm) -> Int in
        return accm + 2 * x
    }
    print(gett() - t1)

結果，以下に示すように```reduce```が一番高速という結果になった．
```foldl```や```foldr```は実装せずに，```reduce```を使いましょう・・・・．

![](http://sonson.s3.amazonaws.com/%E5%90%8D%E7%A7%B0%E6%9C%AA%E8%A8%AD%E5%AE%9A_3.jpg)

計測コードは[こちら](https://gist.github.com/sonsongithub/b897f516005f53bc3748)．