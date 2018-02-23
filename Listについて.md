# Listについて

`List`について学ぶ。

## 1. Listとは

リストはScalaプログラムでもっともよく使われるであろうデータ構造。
順序を持っているコレクションであり、イミュータブルな性質をもつ。  
リスト(`List`)と配列(`Array`)は似ているが、配列はミュータブルである。

```
scala> val colors = List("red", "yellow", "green")
colors: List[String] = List(red, yellow, green)
```

### 1.1. Listの特徴

ちなみに、すこしややこしいが、ListはSeqのサブ型のLinearSeqのサブ型である。

* `Seq`型のサブ型
  * `LinearSeq`型
        * `List`型、`Stream`型
            * `head`/`tail`が早い
  * `IndexedSeq`型
        * `Vector`型など  
            * 要素へのランダムアクセスが早い
            * `length`が早い

`scala/collection/immutable/List.scala`

```
sealed abstract class List[+A] extends AbstractSeq[A]
                                  with LinearSeq[A]
                                  with Product
                                  with GenericTraversableTemplate[A, List]
                                  with LinearSeqOptimized[A, List[A]]
                                  with scala.Serializable {
  override def companion: GenericCompanion[List] = List

  def isEmpty: Boolean
  def head: A
  def tail: List[A]
```

上記のコードをみてわかる通り、`List`は共変(covariant)である。

### 1.2. Listに対する基本操作

`List`型は、`head`と`tail`で構成される。

リストに対する操作は、以下の3つのメソッドを基本的に使う。

* `head` リストの先頭要素を返す
* `tail` 先頭要素を覗く全ての要素から構成されるリストを返す
* `isEmpty` リストが空ならば`true`を返す


要素には、`head`/`tail`でアクセスする。  
以下の例で確認。ついでに`isEmpty`も確認してみる。

```
scala> val colors = List("red", "yellow")
colors: List[String] = List(red, yellow)

scala> colors.isEmpty
res1: Boolean = false

scala> val head1 = colors.head
head1: String = red

scala> val tail1 = colors.tail
tail1: List[String] = List(yellow)

scala> val head2 = tail1.head
head2: String = yellow

scala> val tail2 = tail1.tail
tail2: List[String] = List()

scala> tail3.isEmpty
res2: Boolean = true

// 要素が空の場合のhead, tailアクセスはエラーになる
scala> val head3 = tail2.head
java.util.NoSuchElementException: head of empty list
  at scala.collection.immutable.Nil$.head(List.scala:428)
  at scala.collection.immutable.Nil$.head(List.scala:425)
  ... 28 elided

scala> val tail3 = tail2.tail
java.lang.UnsupportedOperationException: tail of empty list
  at scala.collection.immutable.Nil$.tail(List.scala:430)
  at scala.collection.immutable.Nil$.tail(List.scala:425)
  ... 28 elided
```

上の実行結果をみてわかる通り、`head`は先頭の要素側を表し、`tail`は残りの要素の`List`型のインスタンスとなっている。
さらに、`tail`は`head`と`tail`で構成されており、**再帰的なデータ型となっている**。  
また、最後の要素は`Nil`になっている。`Nil`は空のリストを表す。

図で表すと以下のようになる。

![リストの構成](List_chain.png)


再帰的な構造となっているため、先頭から順にアクセスする。そのため、以下のような要素を指定したランダムアクセス的な使い方は`List`では基本的にしない。

```
scala> val colors = List("red", "yellow", "green")
colors: List[String] = List(red, yellow, green)

scala> colors(2)
res1: String = green
```

また、`List`の長さの計算は、リスト全体を辿って末尾まで走査するので、比較的時間がかかる。

```
scala> val colors = List("red", "yellow", "green")
colors: List[String] = List(red, yellow, green)

scala> colors.length
res1: Int = 3
```

### (補足) `Nil`を`List`に代入可能な理由について

`Nil`は要素が空の`List`を表すが、要素型は`Nothing`になる。  
`Nothing`は、すべての型のサブ型である。

`scala/collection/immutable/List.scala`

```
case object Nil extends List[Nothing] {
  override def isEmpty = true
  override def head: Nothing =
    throw new NoSuchElementException("head of empty list")
  override def tail: List[Nothing] =
    throw new UnsupportedOperationException("tail of empty list")
  // Removal of equals method here might lead to an infinite recursion similar to IntMap.equals.
  override def equals(that: Any) = that match {
    case that1: scala.collection.GenSeq[_] => that1.isEmpty
    case _ => false
  }
}
```

「`List`は共変(covariant)であること」、「`Nil`は要素が空のListで要素型が`Nothing`」であることより、`Nil`は`List`型に代入することが可能になっている。

## 2. Listの生成

`List`の生成方法について確認する。

## 2.1. 基本的な生成方法

基本的な生成方法は、いままででてきたとおりだが、
`apply`メソッドを使っても`List`を生成できる。

```
scala> val list = List.apply("red","blue","yellow")
list: List[String] = List(red, blue, yellow)

scala> val list = List("red","blue","yellow")
list: List[String] = List(red, blue, yellow)
```

## 2.2. コンスを使った生成

リストは、`::`(コンス)を使っても生成できる。
最後の要素は`Nil`(空のリスト)とすることに注意する。

```
scala> val numbers = 1 :: 2 :: 3 :: 4 :: 5 :: Nil
numbers: List[Int] = List(1, 2, 3, 4, 5)

// 以下のように書くと、(head,tail)の構成がわかりやすいかもしれない。
scala> val numbers = 1 :: (2 :: (3 :: (4 ::(5 :: Nil))))
numbers: List[Int] = List(1, 2, 3, 4, 5)
```

## 3. パターンマッチング

リストはパターンマッチで分解できる。

```
scala> val colors = List("red","blue","yellow")
colors: List[String] = List(red, blue, yellow)

scala> val List(a,b,c) = colors
a: String = red
b: String = blue
c: String = yellow
```

`::`(コンス)をつかっても分解できる。

```
scala> val colors = List("red","blue","yellow")
colors: List[String] = List(red, blue, yellow)

scala> val a :: b :: c :: Nil  = colors
a: String = red
b: String = blue
c: String = blue

// colorsの長さが不定の場合は、cをListとして取得することもできる。
scala>  val a :: b :: c  = colors
a: String = red
b: String = blue
c: List[String] = List(yellow)
```

`match`を使ったパターンマッチは、以下の例のように書くこともできる。
`head`、`tail`、`isEmpty`の基本メソッドを使わずにリストを分解できる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def matchList(list: List[String]) = list match {
  case x :: xs => xs
  case List() => List()  // List()はNilに置き換えてもいい。
}

// Exiting paste mode, now interpreting.

matchList: (list: List[String])List[String]

scala> val fooBarBaz = "Foo" :: "Bar" :: "Baz" :: Nil
fooBarBaz: List[String] = List(Foo, Bar, Baz)

scala> matchList(fooBarBaz)
res5: List[String] = List(Bar, Baz)
```

## 4. Listの代表的なメソッド

ここでは、`List`の代表的なメソッドを記載する。  
コレクション共通のメソッドについては、後のレッスンで記載されると思う。

**`contains`メソッド**

指定した要素がリストの中に存在するかをチェックするメソッド。

```
scala> val numberList = List(1,2,3,4,5)
numberList: List[Int] = List(1, 2, 3, 4, 5)

scala> numberList.contains(1)
res0: Boolean = true

scala> numberList.contains(6)
res1: Boolean = false
```

**`reverse`メソッド**

要素を逆順に並べ変えたリストを返却したい場合に使うメソッド。

```
scala> val numberList = List(1,2,3,4,5)
numberList: List[Int] = List(1, 2, 3, 4, 5)

scala> numberList.reverse
res2: List[Int] = List(5, 4, 3, 2, 1)
```

**`indices`メソッド**

要素のインデックスを`Range`で取得したい場合に使うメソッド。

```
scala> val fruits = List("Banana","Orange","Apple")
fruits: List[String] = List(Banana, Orange, Apple)

scala> fruits.indices
res5: scala.collection.immutable.Range = Range 0 until 3
```

**`flatten`メソッド**

要素としての`List`を展開(フラット)して新しいリストを返却するメソッド。

```
scala> val nestedList = List(List(1,2,3), List(), Nil, List(5,6,8))
nestedList: List[List[Int]] = List(List(1, 2, 3), List(), List(), List(5, 6, 8))

scala> nestedList.flatten
res7: List[Int] = List(1, 2, 3, 5, 6, 8)
```

**`zip`メソッド**

2つのリストの各要素を組みにした`List`を要素とした`List`を返却するメソッド。

```
scala> val fruits = List("Grape","Apple","Melon")
fruits: List[String] = List(Grape, Apple, Melon)

scala> val colors = List("purple","red","green")
colors: List[String] = List(purple, red, green)

scala> fruits.zip(colors)
res16: List[(String, String)] = List((Grape,purple), (Apple,red), (Melon,green))
```

**`+:`メソッド**

要素をリストの先頭に追加して新しいリストを返却したい場合は、`:+`メソッド。

```
scala> val numbers = List(1,2,3,4,5)
numbers: List[Int] = List(1, 2, 3, 4, 5)

scala> 0 +: numbers
res23: List[Int] = List(0, 1, 2, 3, 4, 5)
```


**`:+`メソッド**

要素を末尾に追加して新しいリストを返却したい場合は、`+:`メソッド。

```
scala> val numbers = List(1,2,3,4,5)
numbers: List[Int] = List(1, 2, 3, 4, 5)

scala> numbers :+ 6
res24: List[Int] = List(1, 2, 3, 4, 5, 6)
```

**`:::`メソッド**

リスト同士の結合には、`:::`メソッドを使う。

```
scala> List(1,2,3) ::: List(4,5,6) ::: List(7,8,9)
res30: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9)
```
