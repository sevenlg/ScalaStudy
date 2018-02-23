
# 演習問題Part1

## 1. 九九の計算と表示

* 9段の81個の計算結果を表示する
* 空白やタブで計算結果を整形する


`TODO`を実装する

```scala
object MultiTable {

  def main(args: Array[String]): Unit = {
    println(multiTable())
  }

  def makeRowSeq(row: Int) = ??? // TODO 一行分の計算を作成する Vector(  1,  2,  3, ...)
  def makeRow(row: Int) = makeRowSeq(row).mkString
  def multiTable() = {
    val tableSeq = for (row <- 1 to 9) yield makeRow(row)
    tableSeq.mkString("\n")
  }
}
```

結果が以下のようになればOK

```
  1  2  3  4  5  6  7  8  9
  2  4  6  8 10 12 14 16 18
  3  6  9 12 15 18 21 24 27
  4  8 12 16 20 24 28 32 36
  5 10 15 20 25 30 35 40 45
  6 12 18 24 30 36 42 48 54
  7 14 21 28 35 42 49 56 63
  8 16 24 32 40 48 56 64 72
  9 18 27 36 45 54 63 72 81
```


## 2. 副作用のないプログラミング

以下のクラス定義、メソッド定義は副作用をともなう。  
これを副作用のない状態にしてみる。ケースクラスは利用せずに実装すること。

```
scala> :paste
// Entering paste mode (ctrl-D to finish)

import java.util.{ Currency, Locale }

class Money(var amount: Int, val currency: Currency = Currency.getInstance(Locale.getDefault)) {

  def add(money: Money): Unit = {
    amount += money.amount
  }

  def substract(money: Money): Unit = {
    amount -= money.amount
  }

  def +(money: Money): Unit = add(money)
  def -(money: Money): Unit = substract(money)

  def asString: String = s"${currency.getSymbol}$amount"
}

// Exiting paste mode, now interpreting.

import java.util.{Currency, Locale}
defined class Money
```

上記の実装だと、`money1`が書き換わってしまっている。

```
scala> val money1 = new Money(200)
money1: Money = Money@2fd2bbb1

scala> val money2 = new Money(100)
money2: Money = Money@2e273fd4

scala> money1 - money2

scala> money1.asString
res18: String = ￥100
```

(ヒント)   
* ミュータブル変数ををイミュータブル変数に置き換える
* `substract`メソッド、`add`メソッドで新しい`Money`インスタンスを生成する

## 3. ケースクラスへの置き換え

前問で定義したクラスをケースクラスに置き換える。
