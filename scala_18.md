# コレクション 前半
以下のコレクションの使い方と代表的なメソッドを説明します。  

* Array
* Seq
* Map
* Set

## Array
scalaの``Array``はJavaの``Array``と対応しており、``mutable``なコレクションのみを持ちます。

```
Array(要素1, 要素2, 要素3, ...)
```

例
```
scala> val arr = Array(1,2,3)
arr: Array[Int] = Array(1, 2, 3)

//(要素の順番)で要素を抽出できる。
scala> arr(1)
res31: Int = 2
```

また、``Array``から``WrappedArray``への暗黙の変換が行われるため、``Seq``のメソッドを利用することができます。  

```
scala> arr reverse
res0: Array[Int] = Array(3, 2, 1)
```

## Seq 
``Seq``はシーケンスのことで、要素の順序をもつコレクションです。

```
Seq(要素1, 要素2, 要素3, ...)
```
例

```
val seq = Seq("a","b","c")
seq: Seq[String] = List(a, b, c)

//(要素の順番)で要素を抽出できる。
scala> seq(2)
res0: String = c
```

### 代表的なメソッド   
* length
* lengthCompare
* indices
* indexOf
* lastIndexOf
* indexOfSlince
* lastIndexOfSlice
* indexWhere
* prefixLength
* +:, :+
* padTo
* patch
* updated
* sorted  
* reverse
* startsWith
* endsWith
* contains
* containsSlice
* intersect
* diff
* union (++)
* distinct

### Seqの2種類の実装について  

``Seq``は２種類のサブトレイトがあります。  
 特に指定しない場合、``LinearSeq``が呼ばれるため、最初の例では``Seq``を宣言しても``List``が返ります。 
 
|トレイト名|実装(デフォルト)|特徴|  
|:--|:--|:--|  
|IndexedSeq|Vector|要素へのランダムアクセスとlengthが速い。|  
|LinearSeq|List|先頭と末尾の要素へのアクセスが速い。|  

## Map
``map``は``key``と``value``で構成され、``key``に対応する``value``を返すコレクションです。  
``key``は重複できません。  

```
Map(key -> value, ...)
```
例

```
scala> val map= Map(1 -> "a", 2 -> "b", 3 -> "c")
map: scala.collection.immutable.Map[Int,String] = Map(1 -> a, 2 -> b, 3 -> c)
//(key)でvalueを取り出せる。
scala> map (1)
res22: String = a
```
### 代表的なメソッド
* get
* getOrElse
* isDefinedAt
* +, updated
* ++
* -
* --
* keys
* values
* filterKeys
* mapValues


### HashMapについて
HashMapは、Mapトレイトの具象クラスです。  
https://www.scala-lang.org/api/current/scala/collection/immutable/HashMap.html



## Set
``set``は集合を表すコレクションで、要素の重複ができない性質があります。  

```
Set(要素1, 要素2, 要素3, ...)
```
例  

```
scala> val set = Set(1,2,3)
set: scala.collection.immutable.Set[Int] = Set(1, 2, 3)
//(値)でその値が存在するかを返す。
scala> set (1)
res0: Boolean = true
scala> set (4)
res26: Boolean = false
//重複が削除される
scala> val set2 = Set(1, 2, 3, 3, 4)
set: scala.collection.immutable.Set[Int] = Set(1, 2, 3, 4)
```

### 代表的なメソッド
* subsetOf
* set
* +, ++
* -, --
* empty
* intersect, &
* union, |
* &~
* diff

### HashSetについて
https://www.scala-lang.org/api/current/scala/collection/immutable/HashSet.html

## 可変(mutable)と不変(immutable)

scalaのコレクションは``mutable``と``immutable``の両方が実装されています。  
そして、デフォルトではimmutableなコレクションとなります。

mutableとimmutableの両方のコレクションを使いたい場合は、``scala.collection.immutable``をインポートします。  
```
scala> import scala.collection.mutable
import scala.collection.mutable

scala> val seq = mutable.Seq(1,2,3)
seq: scala.collection.mutable.Seq[Int] = ArrayBuffer(1, 2, 3)
```

### mutableとimmutableの変換
mutableとimmutableを変換したい場合は``++=``を用います。  

```
cala> val set = Set(1,2,3)
set: scala.collection.immutable.Set[Int] = Set(1, 2, 3)

scala> val test = mutable.Set.empty ++= set
test: scala.collection.mutable.Set[Int] = Set(1, 2, 3)
```






