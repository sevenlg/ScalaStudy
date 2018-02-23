# これまでの復習 Part1


## 1. 基本文法

基本文法の大事なところだけおさらい。

### 1.1. 変数の定義

Scalaでは、ミュータブルな性質を持つ`var`とイミュータブルな性質を持つ`val`がある。

Scalaのお作法としては、`val`を極力使うようにして、`var`はどうしようもない時に使うといったスタンス。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

// ミュータブルな変数は、varで宣言する
var name1 = "nagakruay"

// イミュータブルな変数はvalで宣言する
val name2 = "nagakuray"

// Exiting paste mode, now interpreting.

name1: String = nagakruay
name2: String = nagakuray

scala> name1 = "igarashi"
name1: String = igarashi

scala> name2 = "igarashi"
<console>:12: error: reassignment to val
       name2 = "igarashi"
             ^
```

### 1.2. 補完子

**`s`補間子**

Scalaでは、文字列の前に`s`をつける。
さらに、対象のオブジェクトの名前に$をつけると、文字列中にそのその変数の値を文字列として展開できる。

```
scala> val name = "Yamada"
name: String = Yamada

scala> val age = 20
age: Int = 20

scala> val weight = 65.3
weight: Double = 65.3

scala> val value = s"name = $name, age = $age, weight = $weight"
value: String = name = Yamada, age = 20, weight = 65.3

scala> val value = s"nameLength = ${name.length}"
value: String = nameLength = 6
```

**f補間子**

`f`を文字列の先頭につけると、書式化文字列を扱える。

```
scala> val value = f"name = $name, age = $age%d, weight = $weight%3.2f"
value: String = name = Yamada, age = 20, weight = 65.30
```

### 1.3. 副作用を伴うメソッドの定義

**呼び出した時に返り値以外に何らかの影響を与えるメソッドを「副作用を伴うメソッド」と呼ぶ。**

**Scalaでは、なるべく「副作用がない」ように実装するのがベター。**

関数型プログラミングでは「メソッドはどんな副作用も持ってはならない」という発想があり、副作用を持たないことで以下メリットがある。

* 同じ引数で何度呼び出しても同じ結果が得られる。
* オブジェクトや関数の独立度が上がり、見通しが良くなる。
* 予期せぬ影響によるバグを減らす事ができる。

**もう少し踏み込んだ定義**

* 副作用がないとは
    * メソッドの戻り値が`Unit`型でない
    * 他のリソースに影響を与えない。

* 副作用があるとは
    * メソッドの戻り値が`Unit`
    * データベースやファイルの更新が発生する。他のリソースに影響を与えてしまう。状態を書き換えてしまう。

**副作用があるかどうかの見分け方、定義の慣習**

引数がない場合は、副作用を伴うメソッドは括弧をつけ、副作用を伴わないメソッドは括弧を付けないことで、副作用を伴うメソッドであるかないかを判別しやすくしている。

```
// サイズを確認するだけ、書き換えは発生しない。副作用がない
queue.size

// printlnメソッドは、Unit型。副作用がある
println()
```

### 1.4. 変数の遅延評価

遅延評価の定義は、以下。

* 値が必要になるまで、値の評価を後回しにする
* 値が必要になった段階で初めて値が決定する

Scalaでは、大体以下の2つで遅延評価を実現する。

1. 遅延評価した値をキャッシュしない: 引数の名前渡し
2. 遅延評価した値をキャッシュする: `lazy val`


**引数の名前渡し(`call-by-name`)**

メソッドおよび関数の引数の型を`=> 戻り値の型`と指定すると、名前渡し(`call-by-name`)となる。

まず、名前渡しを使わない場合をみてみる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

class Logger(isDebug: Boolean) {
  // 名前渡しで定義しない場合
  def debug(message: String): Unit =
    if (isDebug) println(message) 
}

// Exiting paste mode, now interpreting.

defined class Logger
```

```
scala> val debugLogger = new Logger(true)
debugLogger: Logger = Logger@35e3f3b8

scala> debugLogger.debug{ val msg = "hi"; println(s"msg = $msg"); msg}
msg = hi
hi

scala> val notDebugLogger = new Logger(false)
notDebugLogger: Logger = Logger@13da6bc9

scala> notDebugLogger.debug{ val msg = "hi"; println(s"msg = $msg"); msg}
msg = hi

```

名前渡しを使わない場合、デバッグモード時(`isDebug == true`)でも非デバッグモード時(`isDebug == false`)のどちらであっても、`message`が評価されてしまっている。


次に、名前渡しを使った場合をみてみる。

```
scala> :reset

scala> :paste
// Entering paste mode (ctrl-D to finish)

class Logger(isDebug: Boolean) {
  // 名前渡しで定義する場合
  def debug(message: => String): Unit =
    if (isDebug) println(message) // デバッグモード時のみmessageが評価される
}

// Exiting paste mode, now interpreting.

defined class Logger
```

デバッグモード時(`isDebug == true`)は変数`message`を表示する。`println`などで値を必要としたときに引数である`message`が評価されている。


```
scala> val debugLogger = new Logger(true)
debugLogger: Logger = Logger@73bb1337

scala> debugLogger.debug{ val msg = "hi"; println(s"msg = $msg"); msg}
msg = hi
hi
```

非デバッグモード時(`isDebug == false`)は変数`message`は表示されない。

```
scala> val notDebugLogger = new Logger(false)
notDebugLogger: Logger = Logger@30665461

scala> notDebugLogger.debug{ val msg = "hi"; println(s"msg = $msg"); msg}

```

しかし、名前渡しは、呼び出すたびに評価され、その評価結果はキャッシュされない。

```
scala> :reset
// 中略

scala> :paste
// Entering paste mode (ctrl-D to finish)

class Logger(isDebug: Boolean) {
  def debug(message: => String): Unit =
    if (isDebug) {
    // 都度評価される
    println(message)
    println(message)
  }
}

// Exiting paste mode, now interpreting.

defined class Logger

scala> val logger = new Logger(true)
logger: Logger = Logger@55448710

scala> var counter = 1
counter: Int = 1

scala> logger.debug{ println(counter); counter += 1; "Abc"}
1
Abc
2
Abc
```

評価の結果をキャッシュしておきたい場合は、`lazy val`を使う。


**lazy val**

```
scala> :reset

scala> :paste
// Entering paste mode (ctrl-D to finish)

class Logger(isDebug: Boolean) {
  def debug(message: => String): Unit = {
    lazy val msg = message // msgが評価されるまでmessageは評価されない
    if (isDebug) {
      println(msg)
      println(msg)
    }
  }
}

// Exiting paste mode, now interpreting.

defined class Logger
```

```
scala> val logger = new Logger(true)
logger: Logger = Logger@773f7880

scala> var counter = 1
counter: Int = 1

scala> logger.debug{ println(counter); counter += 1; "abc" }
1
abc
abc
```

### 1.5. `if`式と`return`文

戻り値は最後の式が評価された値。他の言語のように、`return`を使っても戻り値を返せる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def append(v1: Int, v2: Int): Int = {
  if (v1 % 2 == 0) return v1 + v2 / 2
  if (v1 % 3 == 0) return v1 + v2 / 3
  v1 + v2
}

// Exiting paste mode, now interpreting.

append: (v1: Int, v2: Int)Int
```

が、Scalaでは`return`を使わない書き方が推奨されている。  
上記の例は、以下のように書くのがベター。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

def append(v1: Int, v2: Int): Int = {
  if (v1 % 2 == 0) v1 + v2 / 2
  else if (v1 % 3 == 0) v1 + v2 / 3
  else v1 + v2
}

// Exiting paste mode, now interpreting.

append: (v1: Int, v2: Int)Int
```

### 1.6. `for`式

Scalaにおいて、`for`も式なので、値を返すことに留意する。

以下の`for`は繰り返し処理では`Unit`型を返す。

```
for (i <- 1 to 9) {
  println(i)
}
```

`Unit`型ではなく、値を返す場合には、`for-yield`を使う。  
結構よく使うらしい。

`for-yield`のベースの書き方は、以下。

```
// パターン1
for(x <- 式1) yield 式2

// パターン2
for(x <- 式1 if 式2) yield 式3
```

`for-yield`を使った例を示しておく。

```
scala> val result1 = for(n <- Seq(1, 2, 3)) yield n * 2
result1: Seq[Int] = List(2, 4, 6)

scala> val result2 = for(n <- Seq(1, 2, 3) if n % 2 != 0) yield n * 2
result2: Seq[Int] = List(2, 6)
```

### 1.7. `try-catch`式

Scalaにおいて、`try-catch`も式。値を返すことに注意。  
基本文法は以下。

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

簡単な例を以下に示しておく。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val returnCode = try {
  // IllegalArgumentExceptionを発生させてみる
  throw new IllegalArgumentException
  //throw new IllegalStateException
  //throw new UnsupportedOperationException

} catch {
  case e: IllegalArgumentException
    => println(e); 1
  case e: IllegalStateException
    => println(e); 2
  case e: UnsupportedOperationException
    => println(e); 3

} finally {
  println("final")
}

// Exiting paste mode, now interpreting.

java.lang.IllegalArgumentException
final
returnCode: Int = 1
```


## 2. クラス

### 2.1. コンストラクタ

基本コンストラクタ、補助コンストラクタについておさらい。  

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

import java.util.{Currency, Locale}

// 基本コンストラクタ
class Money(amount: BigDecimal, currency: Currency) {

  // 補助コンストラクタ
  def this(amount: BigDecimal) = this(amount, Currency.getInstance(Locale.getDefault))

  // 補助コンストラクタ
  def this() = this(BigDecimal(0))

  // インスタンスメソッド
  def asString: String = s"${currency.getSymbol}$amount"

}

// Exiting paste mode, now interpreting.

import java.util.{Currency, Locale}
defined class Money
```


### 2.2. デフォルト引数

補助コンストラクタを利用しないで引数を省略するには、  
デフォルト引数を利用すればいい。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

import java.util.{Currency, Locale}

// 基本コンストラクタ
class Money(amount: BigDecimal, currency: Currency = Currency.getInstance(Locale.getDefault)) {
  // インスタンスメソッド
  def asString: String = s"${currency.getSymbol}$amount"

}

// Exiting paste mode, now interpreting.

import java.util.{Currency, Locale}
defined class Money
```

### 2.3. コンストラクタ引数をフィールドとして定義

Scalaでは、コンストラクタ引数をそのまま、フィールド(`var`/`val`)として定義できる。  
これを**パラメータフィールド**と呼ぶ。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

import java.util.Currency

// コンストラクタ引数をvalで宣言
class MoneyVal(val amount: BigDecimal)

// コンストラクタ引数をvarで宣言
class MoneyVar(var amount: BigDecimal)

// Exiting paste mode, now interpreting.

import java.util.Currency
defined class MoneyVal
defined class MoneyVar
```


```
scala> val moneyVal = new MoneyVal(1000)
moneyVal: MoneyVal = MoneyVal@1d0f7bcf

scala> val moneyVar = new MoneyVar(1000)
moneyVar: MoneyVar = MoneyVar@3897f9ae

scala> moneyVal.amount
res0: BigDecimal = 1000

scala> moneyVar.amount
res1: BigDecimal = 1000

scala> moneyVal.amount = 100
<console>:15: error: reassignment to val
       moneyVal.amount = 100
                       ^
scala> moneyVar.amount = 100
moneyVar.amount: BigDecimal = 100
```

## 2.4. 抽象クラス、抽象メンバ、継承

抽象クラス、抽象メンバ、継承についておさらい。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

// 抽象クラスの定義
abstract class AbstractGreeting {
  // 抽象メンバの定義
  val message: String
  // 抽象クラスには、具象メソッドを定義することもできる。
  def greet(): Unit = println(message)
}

// Exiting paste mode, now interpreting.

defined class AbstractGreeting
```

`AbstractGreeting`クラスを実装したクラスは、以下。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

class JapaneseGreeting extends AbstractGreeting {
  // overrideで具象メンバを定義
  override val message: String = "こんにちは"
}

class EnglishGreeting extends AbstractGreeting {
  // overrideで具象メンバを定義
  override val message: String = "Hello"
}

// Exiting paste mode, now interpreting.

defined class JapaneseGreeting
defined class EnglishGreeting
```

以下のように利用する。

```
scala> val japaneseGreeting: AbstractGreeting = new JapaneseGreeting
japaneseGreeting: AbstractGreeting = JapaneseGreeting@f9d7d3f

scala> val englishGreeting: AbstractGreeting = new EnglishGreeting
englishGreeting: AbstractGreeting = EnglishGreeting@26aecf31

scala> japaneseGreeting.
greet   message

scala> japaneseGreeting.greet
こんにちは

scala> englishGreeting.greet
Hello
```

### 2.5. シールドされたクラス

シールドされたクラスは、パターンマッチの考慮もれを検出する目的で使用される。

```

scala> :paste
// Entering paste mode (ctrl-D to finish)

sealed abstract class Gender
object Male extends Gender
object Female extends Gender
object Unknown extends Gender

// Exiting paste mode, now interpreting.

defined class Gender
defined object Male
defined object Female
defined object Unknown
```

実行例は、以下。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val gender: Gender = Male

gender match {
  case Male => println("男の子")
  case Female => println("女の子")
  case Unknown => println("不明")
}

// Exiting paste mode, now interpreting.

男の子
gender: Gender = Male$@49ace3b2
```

以下の場合は、`Unkonwn`のケースが漏れているため、warningとなる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

gender match {
  case Male => println("男の子")
  case Female => println("女の子")
}

// Exiting paste mode, now interpreting.

<pastie>:15: warning: match may not be exhaustive.
It would fail on the following input: Unknow
gender match {
^
男の子
```


## 3. オブジェクト

### 3.1. `object`キーワード

* Scalaには、Javaの`static`キーワードがない。Scalaでは、`object`を使って`static`メソッド相当を表現する。
* `object`キーワードで生成されるものは、**シングルトンなオブジェクト**であり、インスタンス(オブジェクト)が1つしか生成されないことを保証する。


```
scala> :paste
// Entering paste mode (ctrl-D to finish)

// objectキーワード
object HelloWorld {
  // Javaのstaticメソッド相当
  def run : Unit = {
    println("hello world!");
  }
}

// Exiting paste mode, now interpreting.

defined object HelloWorld
```

```
scala> HelloWorld.run
hello world!
```

### 3.2. コンパニオンクラスとコンパニオンオブジェクト

Scalaではオブジェクトとクラスに同じ名前を付けることが可能。  
このようなオブジェクトをコンパニオンオブジェクトと呼ぶ。

* **コンパニオンオブジェクト**とは、同一ファイル内や同一パッケージ内でクラスと同じ名前で定義されたオブジェクトのこと。
* コンパニオンオブジェクトが対になっているクラスを**コンパニオンクラス**とよぶ。
* コンパニオンクラスとコンパニオンオブジェクトのお互いは、DefaultCurrencyのようにメンバや、メソッドが`private`であったとしても、特権アクセスをもつため、アクセス可能。


```
scala> :paste
// Entering paste mode (ctrl-D to finish)

import java.util.{ Currency, Locale }

// コンパニオンクラス
// コンパニオンオブジェクトのprivateなdefaultCurrencyにアクセスできている。
class Money(var amount: BigDecimal, val currency: Currency = Money.defaultCurrency) {

  def add(money: Money): Unit = {
    amount += money.amount
  }

  def substract(money: Money): Unit = {
    amount -= money.amount
  }

  def asString: String = s"${currency.getSymbol}$amount"
}

// コンパニオンオブジェクト
object Money {
  // USドルの通貨単位
  val USD = Currency.getInstance("USD")
  // 日本円の通貨単位
  val JPY = Currency.getInstance("JPY")
  // デフォルトのCurrency
  private val defaultCurrency = Currency.getInstance(Locale.getDefault)
}

// Exiting paste mode, now interpreting.

import java.util.{Currency, Locale}
defined class Money
defined object Money
```

### 3.3. シールドされたオブジェクト

`sealed`キーワードは、`object`のパターンマッチの検出漏れを防ぐ用途で使われる。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

sealed trait Gender
object Male extends Gender
object Female extends Gender

// Exiting paste mode, now interpreting.

defined trait Gender
defined object Male
defined object Female


scala> :paste
// Entering paste mode (ctrl-D to finish)

val gender: Gender = Male

gender match {
  case Male => println("オトコ")
  case Female => println("オンナ")
}

// Exiting paste mode, now interpreting.

オトコ
```

## 4. ケースクラス

ケースクラスの特徴は、以下の通り。

1. コンストラクタの引数パラメータに、`val`を付けなくてもフィールドが自動生成される。
2. クラスに対して、以下のメソッドが自動生成される。
    * `equals`メソッド
    * `hashCode`メソッド
    * `toString`メソッド
    * `canEqual`メソッド
    * `copy`メソッド
3. コンパニオンオブジェクトに対して、以下のメソッドが自動生成される。
    * `apply`メソッド（`new`演算子を使わずにインスタンス生成できる）
    * `unapply`メソッド(クラスの属性を分解したり、抽出したりができる)

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

case class Money(amount: BigDecimal) {
  // メソッドやメンバの定義
}

// Exiting paste mode, now interpreting.

defined class Money
```

```
// newがなくてもインスタンスを生成できる
scala> val money = Money(100)
money: Money = Money(100)

// unapplyメソッドで要素の分解が可能
scala> val Money(amount) = money
amount: BigDecimal = 100

scala> amount
res0: BigDecimal = 100
```
