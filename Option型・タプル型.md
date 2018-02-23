# Option型、タプル型


## 1. Option型

### 1.1. Option型とは

`Option型`は値があるかどうかわからない状態を表すための型。

以下のように記述する。

```
Option(値、変数やメソッドの返り値)
```

以下の例のように、値がある場合は`Some(value)`を、値がない場合は`None`を返す。

```
scala> val opt = Option(1)
opt: Option[Int] = Some(1)

// Option(変数)も確認
scala> val test = "テスト"
test: String = テスト

scala> val opt = Option(test)
opt: Option[String] = Some(テスト)

// 値がnullの場合はNoneを返す
scala> val opt = Option(null)
opt: Option[Null] = None
```


### 1.2. Option型の値取得

Option型の値を取得したい時は
`get`メソッドあるいは`getOrElse`メソッドを使う。

Noneの場合、`get`メソッドを使用するとExceptionが発生する。
`getOrElse`メソッドを使うことで、Noneの場合に返す値を指定できるので便利。

```
//Some(value)の場合
scala> val opt = Option("テスト")
opt: Option[String] = Some(テスト)

// 値があるため、getメソッドでもgetorElseメソッドでも値を取得できる
scala> val result = opt.get
result: String = テスト

scala> val result = opt.getOrElse("nullです")
result: String = テスト

//Noneの場合
scala> val opt = Option(null)
opt: Option[Null] = None

// getメソッドはExceptionが発生する
scala> val result = opt.get
java.util.NoSuchElementException: None.get
  at scala.None$.get(Option.scala:349)
  at scala.None$.get(Option.scala:347)
  ... 28 elided

// getorElse()メソッドの()の中身が得られる
scala> val result = opt.getOrElse("nullです")
result: String = nullです
```

Option型の値を取得する方法として、パターンマッチも可能。

以下の`something`メソッドは、引数にOption型をとり、Some(value)であれば返り値にvalueを返し、Noneであれば「nullです」の文字列を返す。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def something(x: Option[String]) = x match {
  case Some(value) => value
  case None => "nullです"
}

// Exiting paste mode, now interpreting.

something: (x: Option[String])String

scala> something(Option("テスト"))
res5: String = テスト

scala> something(Option(null))
res6: String = nullです
```

値の取得は基本的にはパターンマッチが利用されているよう。

### 1.3. Option型のnullチェック

Javaの場合、オブジェクトの参照先がない場合には変数に「`null`」を代入していた。

`null`はオブジェクトではないので、`null`に対してメソッドを呼び出そうとすると例外（`NullPointerException`）が発生する。

そのため、`null`チェックを入れる必要があった。

```
public class NullPointerExceptionSample {
    public static void main(String[] args) {
        String str = null;
        ・
        ・ 何らかの処理
        ・      
        # nullチェックが必要
		if ( str != null) {
		      System.out.println(str.toString());
		} else {
		      System.out.println("nullです");
		}
	}
}
```

Scalaでは`null`を使わず、`Option型`を使うことが推奨される。

Option型を上手く使えれば、コンパイル時にnullチェックができる。

変数やメソッドの返り値をできるだけ、`Option()`でくるむようにする。


```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val str = null

Option(str) match {
  case Some(value) => println(value.toString)
  case None => println("nullです")
}

// Exiting paste mode, now interpreting.

nullです
str: Null = null
```

「case None =>  ・・・」の記述を忘れた場合、コンパイル時にwarningを出してくれる。

コンパイル時にnullの場合の考慮漏れに気づくことができる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val str = "test"

// 「case None =>  ・・・」を書き忘れた例
Option(str) match {
  case Some(value) => println(value.toString)
}

// Exiting paste mode, now interpreting.

<pastie>:16: warning: match may not be exhaustive.
It would fail on the following input: None
Option(str) match {
      ^
test
str: String = test
```

Option型を使うことで、**nullになる可能性がある**ということをコードの読み手に伝えられるところがメリット。

### 1.4. ScalaコレクションでのOption型

Option型はScalaコレクションでもよく使用されている。

Option型の例で頻出していた`Map`を見ておく。

`Map`はキーで値を特定できる。（Mapの詳細はコレクションの章にて学習する）

`Map`の`get`メソッドはキーに対応する値が見つかったときにSome（値）を返し、キーが見つからなければNoneを返す。

```
scala> val fruitsColor = Map("apple" -> "red", "banana" -> "yellow", "mango" -> "orange")
fruitsColor: scala.collection.immutable.Map[String,String] = Map(apple -> red, banana -> yellow, mango -> orange)

scala> fruitsColor.get("apple")
res1: Option[String] = Some(red)

scala> fruitsColor.get("melon")
res2: Option[String] = None
```

### （参考）Option型でも使用できるメソッド（mapやforeachメソッド）

Option型はListでよく利用される`map`や`foreach`などのメソッドが用意されている。

それぞれのメソッドの詳細は後の章で学習する。ここでは使用できることを確認する。

map使用例

```
scala> val test = Option("test")
test: Option[String] = Some(test)

// map内にSome(test)の文字列の数を出し、Option[文字数の数]で返す処理を記載
scala> test.map(t => t.size)
res35: Option[Int] = Some(4)

scala> val test = Option(1)
test: Option[Int] = Some(1)

// map内にSome(1)の1に対して演算を行い、Option[演算結果]で返す処理を記載
scala> test.map(t => t*2*3)
res33: Option[Int] = Some(6)
```

foreach使用例

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val test = Option("test")

// Some(value)である場合はforeachの{}内の処理を実行する
// Noneの場合は処理しない
test.foreach { x =>
  println(x)
  println(x)
}

// Exiting paste mode, now interpreting.

test
test
test: Option[String] = Some(test)

scala> :paste
// Entering paste mode (ctrl-D to finish)

val test = Option(null)

// Some(value)である場合はforeachの{}内の処理を実行する
// Noneの場合は処理しない
test.foreach { x =>
  println(x)
  println(x)
}

// Exiting paste mode, now interpreting.

test: Option[Null] = None
```


## 2. タプル

### 2.1. タプルとは

`タプル`は、複数の異なる型を格納できるコンテナオブジェクト。

タプルを使って、メソッドから複数の異なる型を一度に返すことができるので便利。

### 2.2. タプルの使用方法

値をカンマ区切りで並べ、丸括弧でくくるとタプルになる。

タプルは22個まで要素を格納できる。

```
scala> val test = (1, "test", 'a')
test: (Int, String, Char) = (1,test,a)
```

タプルから個々の要素を取り出す場合は以下の例のように取得する。

```
scala> val a = test._1
a: Int = 1

scala> val b = test._2
b: String = test

scala> val c = test._3
c: Char = a
```

パターンマッチでの取得も可能。

```
scala> val test = (1, "test", 'a')
test: (Int, String, Char) = (1,test,a)

scala> test match { case (a,b,c) => a}
res25: Int = 1

scala> test match { case (a,b,c) => b}
res26: String = test

scala> test match { case (a,b,c) => c}
res27: Char = a
```

タプルの個々の要素は変更不可。

```
scala> test._1 = 2
<console>:12: error: reassignment to val
       test._1 = 2
               ^
```

タプルがよく使われるのは、メソッドから複数の値を返すとき。

以下の`getMaxValue`メソッドは引数に数字のリストをもらい、そのリストの中での、最大値と最大値のインデックスを返すメソッド。返り値が(Int, Int)の`タプル`になる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def getMaxValue(numbers: List[Int]) = {
  val max = numbers.max  //numbersのリストの中から最大値を取得する
  val index = numbers.indexOf(max)  //numbersのリストの最大値のインデックスを取得する
  (max, index)  // numbersのリストの中の最大値とそのインデックスをタプルで返す
}

// Exiting paste mode, now interpreting.

getMaxValue: (numbers: List[Int])(Int, Int)

scala> val numbers = List(1, 2, 3, 4, 5, 10, 6)
numbers: List[Int] = List(1, 2, 3, 4, 5, 10, 6)

scala> getMaxValue(numbers)
res19: (Int, Int) = (10,5)  //numbersのリストの最大値10とそのインデックス5を返す
```


（補足）

以下のような、「1 -> 2」のような記載でもタプルを表す。

```
scala> val test = 1 -> 2
test: (Int, Int) = (1,2)
```

タプルは要素数に応じてTuple1, Tuple2, …Tuple22というクラスがあるので、「Tuple2[String, Int]」の様に表すこともできる。

```
scala> val test = Tuple2("test",2)
test: (String, Int) = (test,2)
```
