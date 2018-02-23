# 演習問題Part2

## パターンマッチ

パターンマッチの演習問題。`TODO`をひたすら実装する。

### 1. ワイルドカードパターン

```
// String型の場合はtrue、それ以外の場合は、falseを返すメソッド
def isString(x: Any): Boolean = x match {
 // TODO
}
```

```
scala> isString("This is String type")
res0: Boolean = true

scala> isString(Seq(1,2,3,4))
res2: Boolean = false

scala> isString(true)
res3: Boolean = false
```

### 2. 定数パターン

```
// xが1の場合に"one",falseの場合に"false",それ以外の場合は"others"を返すメソッド
def describe(x: Any) = x match {
  // TODO
}
```

### 3. パターンガード

パターンにマッチするときに、さらにifで条件を指定して絞り込む

```
// nがIntで0以上の場合、nがIntで0より小さい場合とそれ以外の場合でパターンマッチ
def numberCheck(n: AnyVal): String = n match {
  // TODO
}
```

```
scala> numberCheck(100)
res5: String = n is integer and bigger than zero

scala> numberCheck(-2)
res6: String = n is integer and less than zero

scala> numberCheck(println(4))
res7: String = n is other
```

### 4. ケースオブジェクトパターンマッチ

```
sealed trait Employee
case object Parttimer extends Employee
case object RegularEmployee extends Employee
case object ContractEmployee extends Employee


def returnWedge(x: Employee) = x match {
  // TODO
}
```

```
scala> returnWedge(Parttimer)
res0: Int = 1000

scala> returnWedge(RegularEmployee)
res1: Int = 2000

scala> returnWedge(ContractEmployee)
res2: Int = 1500
```

### 5. ケースクラスパターンマッチ

```
sealed trait Name
case class FirstName(name: String) extends Name
case class LastName(name: String) extends Name

def printName(name:Name): Unit = name match {
    // TODO
}
```

```
scala> printName(FirstName("Masaru"))
Masaru

scala> printName(LastName("Igarashi"))
IGARASHI
```


### 6. タプルパターンマッチ

```
scala> val t = ("Monkey","Masaru", 4)
t: (String, String, Int) = (Monkey,Masaru,4)

scala> :paste
// Entering paste mode (ctrl-D to finish)

t match {
  // TODO
}

// Exiting paste mode, now interpreting.

res9: String = Masaru is Monkey. Thak you 4 you!
```


### 7. for文

#### 7.1.

```
scala> case class Man(name: String)
scala> val men = Seq(Man("Ozaki"), Man("Tanaka"), Man("Yoshida"))

scala> val names = for ( /* TODO */ ) yield name
names: Seq[String] = List(Ozaki, Tanaka, Yoshida)
```

#### 7.2.

`Map`の個々の要素は`(key, value)`という`Tuple`。Tupleパターンを使って`for`式で受けられる。

```
scala> val map = Map("a" -> 1, "b" -> 2, "c" -> 3)
map: scala.collection.immutable.Map[String,Int] = Map(a -> 1, b -> 2, c -> 3

scala> for ( /* TODO */ ) println(s"$k : $v")
a : 1
b : 2
c : 3
```



