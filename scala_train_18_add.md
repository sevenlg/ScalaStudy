# コレクションについて（前半）  

## Arrayの使い所
主にjavaとの互換性が必要なタイミングで使用する。
特にこだわりがなければseqを用いる。（基本的にイミュータブルなコレクションの利用を推奨している為）

## ArrayとSeqの互換性について
ArrayはJavaのArrayと同様の機能の他にSeqのメソッドが利用できます。 
Seq型が求められているところにArray型を代入すると暗黙の型変換によってWrappedArrayに変換されます。  

```
scala> val seq:Seq[Int] = Array(1,2,3)
seq: Seq[Int] = WrappedArray(1, 2, 3)
```

## ArrayがSeqメソッドを使える理由の補足
ArrayがSeqのメソッドを利用できるのは、ArrayからSeqのサブクラスであるWrappedArrayまたはArrayOpsへの暗黙の型変換が行われるためです。
WrappedArrayとArrayOpsの違いは、WrappedArrayにSeqのメソッドを使うとWrappedArrayが返ってきますが、ArrayOpsではArrayが返ってくることです。
これらの暗黙の型変換はPreDefで定義されています。
http://www.scala-lang.org/api/2.12.3/scala/Predef$.html  
src:  
https://github.com/scala/scala/blob/v2.12.4/src/library/scala/Predef.scala  
genericArrayOps[l.417 Predef]  
Array to WrappedArray[l.575 LowPriorityImplicits]  

## mutableとimmutableの違い  
scalaでは、mutableとimmutableの2つのコレクションが実装されています。  
mutableは要素が可変で、immutableは要素が不変なコレクションを表します。
scalaではimmutableなコレクションの使用が推奨されています。  
理由として、以下の2点があります。  
* コレクションは別の場所で編集されないためロジックの推論がしやすい。  
* 格納する要素が少ない時、mutableなコレクションよりデータをコンパクトに格納できる。(Mapを例とするとmutableの場合は、空のMapは約80バイトを消費するのに対し、immutableは16~40バイトとなります。)

### val + immutableの場合  
要素の編集と値の再代入が禁止されます。  

```
scala> val set = Set(1,2,3)
set: scala.collection.immutable.Set[Int] = Set(1, 2, 3)

scala> val set_cp = set
set_cp: scala.collection.immutable.Set[Int] = Set(1, 2, 3)

scala> set += 4
<console>:13: error: value += is not a member of scala.collection.immutable.Set[Int]
  Expression does not convert to assignment because receiver is not assignable.
       set += 4
           ^
```

### var + immutableの場合  
要素の編集が禁止されているため、新たなコレクションを作成します。  

```
scala> var set_ver = Set(1,2,3)
set_ver: scala.collection.immutable.Set[Int] = Set(1, 2, 3)

scala> var set_ver_cp = set_ver
set_ver_cp: scala.collection.immutable.Set[Int] = Set(1, 2, 3)

scala> set_ver += 4

scala> set_ver
res5: scala.collection.immutable.Set[Int] = Set(1, 2, 3, 4)

scala> set_ver_cp
res6: scala.collection.immutable.Set[Int] = Set(1, 2, 3)
```

### val + mutableの場合  
値の再代入が禁止されますが、コレクションの要素は編集可能です。  

```
scala> import scala.collection.mutable
import scala.collection.mutable

scala> val set = mutable.Set(1,2,3)
set: scala.collection.mutable.Set[Int] = Set(1, 2, 3)

scala> val set_cp = set
set_cp: scala.collection.mutable.Set[Int] = Set(1, 2, 3)

scala> set += 4
res7: set.type = Set(1, 2, 3, 4)

scala> set_cp
res8: scala.collection.mutable.Set[Int] = Set(1, 2, 3, 4)

scala> set eq set_cp
res9: Boolean = true
```