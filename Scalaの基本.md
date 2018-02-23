# Scalaの基本

## Scalaとは

Scalaの基本について、勉強する。

### クラスベースのオブジェクト指向言語

基本的にはScalaでもクラスからオブジェクトを`new`でオブジェクトを生成して使う。

```
class Point(val x: Int, val y: Int)
val p = new Point(1, 2)
println(p.x) // => 1
println(p.y) // => 2
```

### オブジェクト指向と関数型の特徴を使える言語

Scalaは、オブジェクト指向に加えて関数型言語の特徴を備えた比較的新しい言語。
**オブジェクト指向に加えて**というのがポイント。

### 全ての値はオブジェクト

間接的に全ての型、クラスは`Any`を継承している。
Javaではプリミティブ型扱いされる以下の型もオブジェクト。

* Byte
* Short
* Int
* Long
* Float
* Double
* Boolean
* Char

正確に言うと、`Any`型のサブクラス`AnyVal`を継承した型になっている。  
上記以外の`String`型や`List`型などは、`Any`型のサブクラス`AnyRef`を継承した型として定義されている。  
また、自分で定義したクラスも基本的には、`Any`型のサブクラス`AnyRef`を継承した型として定義されている。

全てがオブジェクトなので、それぞれのオブジェクトでメソッドを持っている。

### 関数(メソッドではない)はオブジェクト

ここで上げておきたい大事な点として、今後、**メソッド**と**関数**という用語の定義を明確に区別すること。

Scalaでは、関数はオブジェクトになる。  
オブジェクトなので、メソッドの引数や戻り値としても定義できる。  
関数を引数や戻り値として定義したメソッドを**高階メソッド**という。
後半の勉強会で何度も出てくるので、定義だけいまのうちに覚えておく。

```scala
// 関数の定義
val sum: (Int,Int) => Int = (a,b) => a + b

// 関数を引数にとるメソッドの定義
def printSum(f: (Int,Int) => Int) = {
  println(f(1,2))
}

printSum(sum)  // => 3
```

## Scalaの代表的な特徴

いくつかScalaの代表的な特徴を紹介しておく。

**簡潔性**

同じ機能(お金)のクラスをScalaとJavaで書いたコードが以下。

```java
// Java版
public class Money {
        private final BigDecimal amount;
        private final Currency currency;
        public Money(BigDecimal amount, Currency currency) {
                this.amount = amount;
                this.currency = currency;
        }
        public BigDecimal getAmount() {
                return amount;
        }
        public Currency getCurrency() {
                return currency;
        }
}
```

```scala
// Scala版
class Money(val amount: BigDecimal, val currency: Currency)
```

**型推論**

Scalaは静的型付け言語でありながらも、スクリプト言語のような表現力を持っている。  
変数の型、メソッドの戻り値の型の宣言をコンパイラに推論させることができる。つまり、スクリプト言語のような簡潔な表現形式を取りながら、Javaのような型安全の両方を実現できる。

```scala
// 型推論しない場合の書き方
val name1: String = "Hello World"

// 型推論させるような書き方
val name2 = "Hello World"
```

**Javaとの相互運用**

ScalaはJVM上で動くので、Java APIもScalaプログラムから呼び出し可能。  
Scalaに対応したOSSがなくても、十分に枯れた実績のあるJavaの資産が使える。

```scala
import java.io.FileInputStream

val fis = new FileInputSteram("names.txt")
```

**関数型による宣言的な記述**

前述したとおり、Scalaは関数型言語としての機能を取り込んでいるので、より簡潔的な宣言が記述可能になっている。

たとえば、ある文字列に大文字が含まれるか検索するメソッドをJavaで実装する場合は以下のようになる。Javaに限らず、多くの言語では、命令型の記述になってしまう。

```java
public static boolean isNameHashUpperCase(String name) {
  boolean result = false;
  for (int i = 0; i < name.length; i ++){
    if (Character.isUpperCase(name.charAt(i))) {
      result = truel
      break;
    }
  }
  return result;
}
```

Scalaでは、以下のような簡潔な記述ができる。
`name`は`String`型の引数である。`String`型は`Char`型のコレクションとしての便利なメソッドが利用できる。

```scala
def isNameHashUpperCase(name: String): Boolean = name.exists(_.isUpper)
```

これら以外にも、トレイトによる多重継承や、暗黙の型変換などJavaにはない機能がある。

Scalaが難しいのは、こうした機能を使うための十分に使うための学習コストがかかるためである。ベターJavaで書くのであれば、さほど学習コストは高くないはず。

## Scalaプログラムの実行方法

いくつかあるが、代表的なプログラム実行方法を記載しておく。

### クラスを作ってビルド・実行する
IntelliJ IDEAを使ってScalaプログラムを動かしてみる。

1. メニューから`File` → `New` → `Project`を指定
1. プロジェクト名を指定する
1. srcディレクトリで`New` → `Scala Class`を
選択し、開いたウィンドウで`HelloWorld`という`object`を作成する。すると、`HelloWorld.scala`というファイルが作成される。
1. 以下の内容を記述して実行する。

```scala
object HelloWorld {
  def main(args:Array[String]):Unit = {
    println("Hello, World!")
  }
}
```

もしくは、以下のように、`Appトレイト`を使うと、よりシンプルに記述できる。

```scala
object HelloWorld extends App {
  // プログラム引数はargs。args(0)とすると各要素にアクセスできる。
  println("Hello, World!")
}
```

プログラムについて解説しておく。

```scala
object HelloWorld { 
  def main(args: Array[String]): Unit = {
    println("Hello, World!")
  }
}
```

* Scalaではアクセス修飾子を省略した場合は`public`になる
* objectは、インスタンスを1つだけ作るキーワード。
* Scalaでは、Javaのクラスメソッド(static)はなく、すべてインスタンスメソッドの扱いになる
* Scalaのメソッド宣言は、`def`から始まる。メソッドの引数の型は、コロン(`:`)に続けて記述する。Scalaで配列を利用する場合は、`Array`を利用する。
* Scalaのメソッドの戻り値の型は、(`:`)に続けて記述する。
* `Unit`とは意味のある値を返さない型です。ここではJavaの`void`相当と理解しておけばよい。
* Scalaではメソッドの本体のブロックは、メソッドの部の後ろにイコール(`=`)をつけて記述する


### REPLによる実行方法

Scalaのコードをインタプリタで対話的に実行する方法。この実行環境のことを REPL (=Read EvalPrint Loop)と呼ぶ。

```
$ scala
 Welcome to Scala 2.12.4 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_144).
Type in expressions for evaluation. Or try :help.

scala> println("Hello, World!")
Hello, World!
```

### スクリプトモードを使った実行方法

以下のような`HelloWorldScript.scala`を作っておき、

```
println("Hello, World!")
```

このようにスクリプトファイルとして実行できる。

```
$ scala HelloWorldScript.scala
Hello, World!
```

## 変数の定義

ここからは、REPL(Read-eval-print loop)を起動して確認していく。

### var

`var`で定義された変数は、再代入可能

```
scala> var name = "Masaru Igarashi"
name: String = Masaru Igarashi

scala> name = "Naomi Kaneko"
name: String = Naomi Kaneko
```

代入する値の型に応じて、変数の型が決定される型推論もされていることにも注目。

### val

`val`は読み込み専用の変数。一度値を代入すると、再代入できない。不変（イミュータブル）な性質をもつ。

```
scala> val name = "Masaru Igaarshi"
name: String = Masaru Igaarshi

scala> name = "Naomi Kaneko"
<console>:12: error: reassignment to val
       name = "Naomi Kaneko"
```

Scalaでは、一度作成したら再度の代入ができない「不変なval」と何度も再代入可能な「可変なvar」の2種類の宣言が可能。

関数型のプログラミングスタイルでは変数を再代入できない`val`を原則的に利用する。

### lazy val

`lazy val`は、参照されるまで評価されない、遅延評価型の`val`  
`lazy val`は、必要になるまで計算コストを控える場合に使う。

```
// この時点では初期化されていない
scala> lazy val name = "Masaru Igarashi"
name: String = <lazy>

scala> println(name) // 参照される際にnameが初期化される
Masaru Igarashi
```

`lazy val`の補足。試しに打ってみるといいかもしれない。

```
def getHTML = {
  println("Requesting..")
  scala.io.Source.fromURL("https://www.yahoo.co.jp").mkString
}

// 都度HTTPリクエストが発生する
println(getHTML)
println(getHTML)
println(getHTML)


// lazy val
lazy val getHTML = {
  println("Requesting..")
  scala.io.Source.fromURL("https://www.yahoo.co.jp").mkString  
}
// HTTPリクエストは最初の一回だけ
println(getHTML)
println(getHTML)
println(getHTML)
```

## Scalaの代表的な型

### String型

おなじみの`String`。

```
scala> val value = "hello world"
value: String = hello world
```

#### String#formatメソッド

```
scala> val name = "Yamada"
name: String = Yamada

scala> val age = 20
age: Int = 20

scala> val weight = 65.3
weight: Double = 65.3

scala> val str = "name = %s, age = %d, weight = %3.2f".format(name, age, weight)
str: String = name = Yamada, age = 20, weight = 65.30

// 標準出力に出力したいだけならprintfを使う
scala> printf("name = %s, age = %d, weight = %3.2f", name, age, weight)
name = Yamada, age = 20, weight = 65.30
```

#### s補完子

Scalaでは、文字列の前に`s`をつける。
さらに、対象のオブジェクトの名前に`$`をつけると、文字列中にそのその変数の値を文字列として展開できる。

```
// 基本的な使い方1
scala> val value = s"name = $name, age = $age, weight = $weight"
value: String = name = Yamada, age = 20, weight = 65.3

// 基本的な使い方2
scala> val value = s"nameLength = ${name.length}"
value: String = nameLength = 6
```

#### f補完子

`f`を文字列の先頭につけると`String#format`と同様に書式化文字列を扱える

```
scala> val value = f"name = $name, age = $age%d, weight = $weight%3.2f"
value: String = name = Yamada, age = 20, weight = 65.30
```


#### ScalaのStringはjava.lang.String

Scalaの`String`は`java.lang.String`と同じもの

```
scala> val value = "ABC"
value: String = ABC

scala> val value2 = value.concat("DEF")
value2: String = ABCDEF

scala> val value3 = value2.toLowerCase
value3: String = abcdef
```

### Unit型

Unit型は、Javaのvoid型に相当する。
意味のある値がないことを示す型。

```
scala> def hello(): Unit = println("Hello World!")
hello: ()Unit
```

### Boolean型

`Boolean`オブジェクトは真偽値を表現するためのオブジェクト。
`true`/`false`の設定のみ。

```
scala> val value = true
value: Boolean = true

scala> val value = false
value: Boolean = false
```

### Char型

文字を表現するための型

```
scala> val value = 'a'
value: Char = a

scala> val value = "abc"
value: String = abc
```

### Int型

32ビット(4バイト)の整数を表現するための型。

```
scala> val value1 = 10
value1: Int = 10
```

### Byte型

Byteは8ビット(1バイト)の整数を表現する型。
-128 ~ 127(2^7 ~ 2^7-1)まで表現。

```
scala> val value = 1.toByte
value: Byte = 1
```

### Short型

Shortは、16ビット(4バイト)の整数を表現する型。
-32768 ~ 32767(2^15 ~ 2^15-1)まで表現できる

```
scala> val value = 1.toShort
value: Short = 1
```

### Long型

Longは、64ビット(8バイト)の整数を表現する型。
-9223372036854775808 ~ 9223372036854775807(2^63 ~ 2^63-1)まで表現できる。

```
scala> val value = 10000L
value: Long = 10000
```


## 演算子

ScalaでもJavaと同様に、一般的な演算子として以下はサポートしている。

* `==`
* `!=`
* `<`
* `<=`
* `>`
* `>=`
* `&&`
* `||`

注意が必要な演算子について解説しておく。

### ==演算子
Javaの場合は、オブジェクトどうしの同値性を確認する場合は、`equals`メソッドを使っていた。

```java
String t1 = "text1";
String t2 = "text1";
String t3 = "text2";


System.out.println(t1.equals(t2)); // true
System.out.println(t1.equals(t3)); // false
```

Scalaの場合、同値性を確認するには、`==`を使う。  
`==`は、Scalaの場合、メソッドであることに注意。

```
scala> val t1 = "text1"
t1: String = text1

scala> val t2 = "text1"
t2: String = text1

scala> val t3 = "text2"
t3: String = text2

scala> println(t1 == t2)
true

scala> println(t1 == t3)
false
```

少し、細かい話になるが、Scalaの`==`メソッドは、`equals`メソッドを呼び出している。String型の場合は、`==`メソッドを使うと、`java.lang.String#equals`メソッドを呼び出すことができるが、自分で作ったクラスを呼び出すときは、`==`メソッド、`equals`メソッドを使っても同値性の確認はそのままだとできない。

```
scala> class Hoge(a: String)
defined class Hoge

scala> val h1 = new Hoge("hello")
h1: Hoge = Hoge@608f79a8

scala> val h2 = new Hoge("hello")
h2: Hoge = Hoge@4df9df06

scala> h1 == h2
res5: Boolean = false
```

自分で作ったクラスの同値性確認を`==`メソッドで可能にするには、`equals`メソッドをオーバーライドするか、ケースクラスで定義する必要がある。


### インクリメント(`++`)とデクリメント(`--`)

Javaでは`++`演算子、`--`演算子をサポートしていたが、Scalaはサポートされていない。

```
scala> var a = 1
a: Int = 1

scala> a ++
<console>:13: error: value ++ is not a member of Int
       a ++
         ^

scala> a --
<console>:13: error: value -- is not a member of Int
       a --
         ^
```

Scalaでは、`+=`メソッド、`-=`メソッドを利用する。

```
scala> var a = 1
a: Int = 1

scala> a += 1

scala> a
res13: Int = 2

scala> var b = 1
b: Int = 1

scala> b -= 1

scala> b
res15: Int = 0
```


## 制御構文

**「構文」と「式」と「文」という用語について**

「**構文（Syntax）**」は、そのプログラミング言語内でプログラムが構造を持つためのルール。  
多くの場合、プログラミング言語内で特別扱いされるキーワード、たとえば`class`や`val`、`if`などが含まれ、そして正しいプログラムを構成するためのルールがある。  
`class`の場合であれば、`class`の後にはクラス名が続き、クラスの中身は`{`と`}`で括られる、など。


「**式（Expression）**」は、プログラムを構成する部分のうち、評価が成功すると値になるもの。  
たとえば`1`や`1 + 2`、`"hoge"`など。これらは評価することにより、数値や文字列の値になる。  
もうひとつ例をだすと、`if (条件) { 式 } else { 式 }`のように、`if`は評価されると値になる。  
評価が成功、という表現を使ったが、評価の結果として例外が投げられた場合等が、評価が失敗した場合に当たる。

「**文（Statement）**」は、式とは対照的にプログラムを構成する部分のうち、評価しても**値にならないもの**。  
たとえば変数の定義である`val i = 1`は評価しても変数`i`が定義され、`i`の値が`1`になるが、この定義全体としては値を持たない。よって、これは文になる。


### if式

#### else if/else句がないif式

以下の例は、``式を使って、現在時刻を表す秒数(エポック秒) が2で割り切れる偶数か判定する。偶数ならその時間を表示、奇数なら何もしない。を実装した例。

```
scala> import java.time._
import java.time._

scala> :paste
// Entering paste mode (ctrl-D to finish)

def printTime = {
  val now = ZonedDateTime.now
  if (now.toEpochSecond % 2 == 0) println(now)
}

// Exiting paste mode, now interpreting.

printTime: Unit
```

`printTime`の戻り値からわかるように、`if`式は`Unit`型を返します。

実行結果は以下の通り。

```
scala> printTime
2017-11-10T09:28:40.760+09:00[Asia/Tokyo]

scala> printTime

scala> printTime
2017-11-10T09:28:42.196+09:00[Asia/Tokyo]
```

#### else/else if句があるif式

```
scala> import java.time._
import java.time._

scala> :paste
// Entering paste mode (ctrl-D to finish)

def getValue = {
  if (ZonedDateTime.now.toEpochSecond % 2 == 0)
    1
  else
    0
}

// Exiting paste mode, now interpreting.

getValue: Int

scala> getValue
res51: Int = 1

scala> :type getValue
Int
```

`else if`句が追加された場合も式の結果型は統一されているため、結果型は同様に`Int`型になる。

### match式

Javaは`if`式以外の条件分岐としては`switch`文が有名だが、Scalaでは`match`式を利用する。
`match`式の構文規則は、以下。

```
val 変数名 = match {
  case パターン1 => 式1
  case パターン2 => 式2
  case パターン3 => 式3
  case _ => 式4
}
```

#### Unit型を返すmatch式

`Unit`型を返す`match`式の例。  
`println`は`Unit`型を返す。

```
scala> val word:String = "uno"
word: String = uno

scala> :paste
// Entering paste mode (ctrl-D to finish)

word match {
  case "uno" => println("1です")
  case "dos" => println("2です")
  case "tres" => println("3です")
  case _ => println(word)
}

// Exiting paste mode, now interpreting.

1です
```

`match`式は`case 条件 => 評価する式`を複数持つ。  
`case`は記述した順序で、上から順番に評価するが、`case _ =>`は上位のいずれの`case`にもマッチしなかった場合に利用される。

```
scala> val word:String = "いち"
word: String = いち

scala> :paste
// Entering paste mode (ctrl-D to finish)

word match {
  case "uno" => println("1です")
  case "dos" => println("2です")
  case "tres" => println("3です")
  case _ => println(word)
}

// Exiting paste mode, now interpreting.

いち
```

#### Unit以外の型を返すmatch式

`Unit`以外の返す方法をみていく。  
以下の例は、`Int`型の値を返す`match`式を使った例。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def convertToNumber(word: String): Int = word match {
  case "uno" => 1
  case "dos" => 2
  case "tres" => 3
  case _ => 0
}

// Exiting paste mode, now interpreting.

convertToNumber: (word: String)Int

scala> convertToNumber("uno")
res7: Int = 1

scala> convertToNumber("dos")
res8: Int = 2

scala> convertToNumber("tres")
res9: Int = 3

scala> convertToNumber("one")
res10: Int = 0
```

`match`式で`Int`型を返しているのがわかると思う。

#### 値判定以外のmatch式

**型を判定する場合**

`case 変数名: 型名`とすると型を判定できる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def convertToString(value: Any): String = value match {
  case s: String => s
  case n: Int => n.toString
  case d: Double => d.toString
  case dt: ZonedDateTime => dt.toString
  case _ => ""
}

// Exiting paste mode, now interpreting.

convertToString: (value: Any)String

scala> convertToString("abc")
res0: String = abc

scala> convertToString(100)
res1: String = 100

scala> convertToString(1.00)
res2: String = 1.0

scala> convertToString(ZonedDateTime.now)
res3: String = 2017-05-02T16:10:39.919+09:00[Asia/Tokyo]
```

**コレクションを判定する場合**

型以外にコレクションを判定する場合にも`match`式が使える。  
以下は、`Array`コレクションの要素をパターンマッチングした例。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def convertToString(value: Array[Int]): String = value match {
  case Array(1) => "1"
  case Array(1, 2) => "1, 2"
  case _ => ""
}

// Exiting paste mode, now interpreting.

convertToString: (value: Array[Int])String

scala> convertToString(Array(1))
res0: String = 1

scala> convertToString(Array(1,2))
res1: String = 1, 2

scala> convertToString(Array())
res2: String = ""
```

#### case句に後置形式のif式を組み合わせる

`match`式の`case`に後置形式の`if`式の条件を記述できる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def getValue(str: String): Any = str match {
  case value if value.startsWith("number") && value.split("=").length >= 2 =>
    val params = value.split("=")
    val numberText = params(1)
    numberText.trim.toInt
  case value if value.startsWith("text") && value.split("=").length >= 2 =>
    val params = value.split("=")
    val numberText = params(1)
    numberText.trim
  case _ => None
}

// Exiting paste mode, now interpreting.

getValue: (str: String)Any

scala> getValue("number = 1")
res0: Any = 1

scala> getValue("text = a")
res1: Any = a

scala> getValue("number aaa")
res2: Any = None
```

### for式

```
for (i <- 1 to 9) {
  println(i)
}
```

単一の式の場合は、`{}`を省略できる。

```
for (x <- 1 to 9) println(x)
```

`Array`コレクションをジェネレータとしてみた例。

```
scala> val numbers = Array(1, 2, 3)
numbers: Array[Int] = Array(1, 2, 3)

scala> for (n <- numbers) println(n)
1
2
3
```

`Seq`コレクションをジェネレータとしてみた例。

```
scala> val numbers = Seq(1, 2, 3)
numbers: Seq[Int] = List(1, 2, 3)

scala> for (n <- numbers) println(n)
1
2
3
```

`Set`コレクションをジェネレータとしてみた例。

```
scala> val numbers = Set(1, 2, 3)
numbers: scala.collection.immutable.Set[Int] = Set(1, 2, 3)

scala> for (n <- numbers) println(n)
1
2
3
```

`Hash`コレクションをジェネレータとしてみた例。

```
scala> val numbers = Map(1 -> "one", 2 -> "two", 3 -> "three")
numbers: scala.collection.immutable.Map[Int,String] = Map(1 -> one, 2 -> two, 3 -> three)

scala> for ((key,value) <- numbers) println(s"key = $key, value = $value")
key = 1, value = one
key = 2, value = two
key = 3, value = three
```

`Iteretor`をジェネレータとして見た例。

```
$ echo -e "ABC\nDEF\nGHI" > /tmp/abc.txt;
$ cat /tmp/abc.txt
ABC
DEF
GHI
```

```
scala> import scala.io.Source
import scala.io.Source

scala> val source = Source.fromFile("/tmp/abc.txt")
source: scala.io.BufferedSource = non-empty iterator

scala> val lines: Iterator[String] = source.getLines
lines: Iterator[String] = non-empty iterator

scala> for (line <- lines) println(line)
ABC
DEF
GHI
```

#### for式の入れ子

入れ子の`for`文も実装できる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

for (x <- 1 to 3; y <- 10 to 30 by 10)
  println(x + ":" + y)

// Exiting paste mode, now interpreting.

1:10
1:20
1:30
2:10
2:20
2:30
3:10
3:20
3:30
```

#### for式内でのフィルタリング

`for`式の中に`if`式も記述できる。

```
scala> val names = Seq("Java","Scala","VBA","PHP","Ruby")
names: Seq[String] = List(Java, Scala, VBA, PHP, Ruby)

scala> for (lang <- names if !lang.startsWith("J")) println(lang)
Scala
VBA
PHP
Ruby
```

上記のような書き方もできるが、Scalaでは、コレクション操作のメソッドを使って以下のようにも記述できる。

```
scala> names.filter(!_.startsWith("J")).foreach(println)
Scala
VBA
PHP
Ruby
```


### while式

`while`式を使った書き方。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

var num: Int = 3
while(num >=0) {
  println(num)
  num -= 1
}

// Exiting paste mode, now interpreting.

3
2
1
0
num: Int = -1
```

上の例では、変数`num`は`var`で宣言されている。  
しかし、Scalaでは、`var`はどうしてもやむを得ない時以外は使用しない習慣があるので、上記の書き方を見ることは少ない。


### try-catch式

`try-catch-finally`の書き方として、基本構文は、以下のようになる。

```
val 変数名 = try {
  // 式
} catch {
  case ex: 例外型 => // 例外ケース1
    // 式1
  case ex: 例外型 => // 例外ケース2
    // 式2
} finally { // 省略可能
  // 終了処理
}
```

実装例は、以下のようになる。

```
scala> import java.io._
import java.io._

scala> import java.nio.charset.StandardCharsets
import java.nio.charset.StandardCharsets

scala> :paste
// Entering paste mode (ctrl-D to finish)

def getFileBody(file: String): String = {
  var in: FileInputStream = null
  try {
    in = new FileInputStream(file)
    val buf = new Array[Byte](in.available())
    in.read(buf)
    new String(buf, StandardCharsets.UTF_8)
  } catch {
    case ex: IOException =>
      ex.printStackTrace()
      ""
  } finally {
    if (in != null) {
      println("in.close")
      in.close()
    }
  }
}

// Exiting paste mode, now interpreting.

getFileBody: (file: String)String

scala> getFileBody("/tmp/abc.txt")
in.close
res6: String =
"ABC
DEF
GHI
"
```
