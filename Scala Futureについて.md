### 概要

`Future`とは、非同期に処理される結果が入ったOption型のようなものです。  
mapやflatMapやfilter、for式の適用といったようなOptionやListでも利用できる性質を持っています。  
ライブラリやフレームワークの処理が非同期主体となっている場合、このFutureは基本的で重要な役割を果たすクラスとなります。    

### 特徴

1. `Future`が値とともに完了した場合、Future はその値とともに成功したという。   
   `Future`が例外とともに完了した場合、Future はその例外とともに失敗したという。   
2. `Future`には 1回だけ代入することができるという重要な特性がある。  
    一度Futureオブジェクトが値もしくは例外を持つと、実質不変となり、それが上書きされることは絶対に無い。   
3. `Future`オブジェクトを作る最も簡単な方法は、非同期の計算を始めてその結果を持つ、Futureを返す futureメソッドを呼び出すことだ。  
	計算結果は Future が完了すると利用可能になる。  

ScalaのFutureは、共有データとロックについてあまり考えずに並行処理を実行できる。   
Scalaのメッソドは、起動されると決算を行って結果を返す。  
その結果がFutureオブジェクトであれば、そのFutureはまったく異なるスレッドによって非同期に実行される別の計算であることを表す。  
そのため、Futureに対する多くの操作は、暗黙の実行コンテキスト(Execution context)を必要とする。

ドキュメント: http://www.scala-lang.org/api/2.12.4/scala/concurrent/index.html
```
scala> import scala.concurrent.Future
import scala.concurrent.Future
```
エラー例
```
scala> val fut = Future{Thread.sleep(10000); 21 + 21 }
<console>:12: error: Cannot find an implicit ExecutionContext. You might pass
an (implicit ec: ExecutionContext) parameter to your method
or import scala.concurrent.ExecutionContext.Implicits.global.
       val fut = Future{Thread.sleep(10000); 21 + 21 }
```
このエラーメッセージは、Scala自身が提供するグローバル実行コンテキストをインポットするという問題です。  
JVM上は、グローバル実行コンテキストはスレッドプールを使う。  
スコープに暗黙の実行コンテキストを入れれば、Futureを作ることができる。
```
scala> import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.ExecutionContext.Implicits.global
```
```
scala> val fut = Future{Thread.sleep(10000); 21 + 21 }
fut: scala.concurrent.Future[Int] = Future(<not completed>)
```

前の例でFutureは、グローバル実行コンテキストを使ってコードブロックを非同期に実行し、42という値を返して終了する。  
スレッドは実行を開始すると10秒間スリープする。  
そのため、このfutureは、完了するまで少なくとも10秒かかる。  

```
scala> fut.isCompleted
res2: Boolean = false
```
```
scala> fut.value
res3: Option[scala.util.Try[Int]] = None
```
Futureは完了すると、（この場合、１０秒以上の時間が過ぎると）、`isCompleted`はtrue, `value`はSomeを返す。  
```
scala> fut.isCompleted
res4: Boolean = true

scala> fut.value
res5: Option[scala.util.Try[Int]] = Some(Success(42))
```

###### Try
`value`メソッドが返すオプションにはTryが含まれている。  
tryの目的は、tryが同期計算で提供しているものを非同期計算で提供することです。   
つまり、計算が値を返さずに、例外を投げて異常終了した場合にも対処できるようにしているのである。    

同期計算では、try/catchを使ってメソッドが投げた例外は、メソッドを呼び出ししたスレッドがキャッチして処理することを保障している。   
非同期計算では、計算を始めたスレッドは、ほかのタスクに切り替えられていることが多い、その後、その非同期計算が例外を投げて失敗しても、  
元のスレッドはcatch節では例外を処理することはできなくなっている。  
そこで、非同期計算を表すFutureを操作するときに、Tryを使って、 
非同期計算が値を生成せず、例外を投げて異常終了している場合の後始末ができるようにしている。  

```
scala> val fut = Future { Thread.sleep(10000); 21/ 0}
fut: scala.concurrent.Future[Int] = Future(<not completed>)

scala> fut.value
res6: Option[scala.util.Try[Int]] = None

scala> fut.value
res8: Option[scala.util.Try[Int]] = Some(Failure(java.lang.ArithmeticException: / by zero))
```

###### ファルタリング: `filter`と`collect`  
scalaのFutureは、その値がある性質を満たすことをチェックする`filter`と`collect`の二つのメソッドを提供している。  
`filter`メソッドは、Futureの結果をチェックし、有効ならそのままにする。 
```
scala> val fut = Future { 41 }
fut: scala.concurrent.Future[Int] = Future(<not completed>)

scala> fut.isCompleted
res17: Boolean = true

scala> val valid = fut.filter(res => res > 0)
valid: scala.concurrent.Future[Int] = Future(Success(41))

```
futureの値が有効でなければ、`filter`が返すFutureの値はNoSuchElementExceptionを含む失敗になる。
```
scala> val valid = fut.filter(res => res < 0)
valid: scala.concurrent.Future[Int] = Future(<not completed>)

scala> valid.value
res18: Option[scala.util.Try[Int]] = Some(Failure(java.util.NoSuchElementException: Future.filter predicate is not satisfied))
```  

`collect`メソッドを使えば、Future値の有効性をチェックした上で、変換するということを一つの呼び出しで実行できる。  
`collect`に渡された部分関数がFutureの結果に対して定義されている場合、collectが返すfutureは、collectによって変換された値を含む成功である。  
```
scala> val valid = fut.collect{ case res if res > 0 => res + 46 }
valid: scala.concurrent.Future[Int] = Future(<not completed>)

scala> valid.value
res19: Option[scala.util.Try[Int]] = Some(Success(87))
```
そうでなければ、futureはNoSuchElementExceptionを含む失敗である。
```
scala> val valid = fut.collect{ case res if res < 0 => res + 46 }
valid: scala.concurrent.Future[Int] = Future(<not completed>)

scala> valid.value
res20: Option[scala.util.Try[Int]] = Some(Failure(java.util.NoSuchElementException: Future.collect partial function is not defined at: 41))
```

##### 失敗の処理: `failed`、`fallbackTo`
ScalaのFutureは、失敗したFutureを操作する方法として、`failed`、`fallbackTo`を提供している。  
`failed`メソッドは、任意の型の失敗したFutureから、成功したFuture[Throwable]に変更する。  
このFuture[Throwable]は、エラーの原因となった例外を含んでいる。

```
scala> val fut = Future { 42 / 0 }
fut: scala.concurrent.Future[Int] = Future(<not completed>)

scala> fut.value
res21: Option[scala.util.Try[Int]] = Some(Failure(java.lang.ArithmeticException: / by zero))

scala> val expectedFailure = fut.failed
expectedFailure: scala.concurrent.Future[Throwable] = Future(Success(java.lang.ArithmeticException: / by zero))

scala> expectedFailure.value
res22: Option[scala.util.Try[Throwable]] = Some(Success(java.lang.ArithmeticException: / by zero))
```

`failed`メソッドの呼び出しに使ったFutureが最終的に成功すると、`failed`が返したfuture自体がNoSuchElementExceptionを起こして失敗する。  
そこで、`failed`メソッドは、futureが失敗すると予想されているとき以外は適切に動作しない。

```
scala> val success = Future { 42 / 1}
success: scala.concurrent.Future[Int] = Future(<not completed>)

scala> success.value
res23: Option[scala.util.Try[Int]] = Some(Success(42))

scala> val unexpectedSuccess = success.failed
unexpectedSuccess: scala.concurrent.Future[Throwable] = Future(Failure(java.util.NoSuchElementException: Future.failed not completed with a throwable.))

scala> unexpectedSuccess.value
res24: Option[scala.util.Try[Throwable]] = Some(Failure(java.util.NoSuchElementException: Future.failed not completed with a throwable.))
```

`fallbackTo`は、`fallbackTo`を呼び出したFutureが失敗したとき、代わりに使えFutureを提供できるようにする。  
```
scala> val fallback = fut.fallbackTo(success)
fallback: scala.concurrent.Future[Int] = Future(Success(42))

scala> fallback.value
res25: Option[scala.util.Try[Int]] = Some(Success(42))
```

##### 副作用の実行: `foreach`、`onComplete`、`andThen`
Futureが完了した後、副作用を実行しなければならないときがある。  
Futureは、そのためのメソッドをいくつか提供している。

もっとも基本的なメソッドは、Futureが成功して完了した時に副作用を実行する`Foreach`である。
```
scala> val failure = Future { 42/0 }
failure: scala.concurrent.Future[Int] = Future(<not completed>)

scala> val success = Future { 42/1 }
success: scala.concurrent.Future[Int] = Future(Success(42))
```
```
scala> failure.foreach(ex => println(ex))

scala> success.foreach(rex => println(rex))
42
```

Futureは、`コールバック`関数を登録するメソッドも二つ提供している。  
`onComplete`メソッドは、Futureが最終的に成功したか失敗したかにかわらず実行される。  
関数にはTryが渡される。  渡されるTryは、 
1.Futureが成功した場合は、値を格納するSuccessであり。  
2.Futureが失敗した場合は、失敗の原因となった例外を格納するFailureである。  
```
scala> import scala.util.{Success, Failure}
import scala.util.{Success, Failure}
```
`success`:
```
scala> :paste
// Entering paste mode (ctrl-D to finish)

success.onComplete {
  case Success(res) => println(res)
  case Failure(ex) => println(ex)
}


// Exiting paste mode, now interpreting.

42
```
`failure`:
```
scala> :paste
// Entering paste mode (ctrl-D to finish)


failure.onComplete {
  case Success(res) => println(res)
  case Failure(ex) => println(ex)
}

// Exiting paste mode, now interpreting.

java.lang.ArithmeticException: / by zero

```

Futureは、`onComplete`で登録されたこーるバック関数の実行順序に関して一切の保証をしない。  
コールバック関数の順序を強制したい場合は、`onComplete`ではなく、`andThen`を使わなければならない。  
`andThen`メソッドは、新しいFutureを返す。このFutureには、`andThen`を呼び出した元のFutureの状態が反映されている（成功なら成功、失敗なら失敗）。  
ただし、コールバック関数が完全に実行されるまで、andThenは完了しない。  
`success`:
```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val newFuture = success.andThen {
  case Success(res) => println(res)
  case Failure(ex) => println(ex)
}

// Exiting paste mode, now interpreting.

newFuture: scala.concurrent.Future[Int] = Future(<not completed>)
42
```
```
scala> newFuture.value
res31: Option[scala.util.Try[Int]] = Some(Success(42))
```

`failure`:
```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val newFuture = failure.andThen {
  case Success(res) => println(res)
  case Failure(ex) => println(ex)
}

// Exiting paste mode, now interpreting.

java.lang.ArithmeticException: / by zero
newFuture: scala.concurrent.Future[Int] = Future(<not completed>)
```
```
scala> newFuture.value
res32: Option[scala.util.Try[Int]] = Some(Failure(java.lang.ArithmeticException: / by zero))
```
`andThen`に渡されたコールバック関数が実行時に例外を投げた場合、その例外は後続のコールバックに伝播されたり、結果のFutureを介して報告されたりしないので、  
注意が必要である。
```
scala> :paste
// Entering paste mode (ctrl-D to finish)

val f = Future { 5 }
val f2 = f.andThen {
  case r => sys.error("runtime exception")
} andThen {
  case Failure(t) => println(t)
  case Success(v) => println(v)
}

// Exiting paste mode, now interpreting.

f: scala.concurrent.Future[Int] = Future(<not completed>)
f2: scala.concurrent.Future[Int] = Future(<not completed>)
java.lang.RuntimeException: runtime exception
  at scala.sys.package$.error(package.scala:27)
  at $line73.$read$$iw$$iw$$anonfun$1.applyOrElse(<pastie>:17)
  at $line73.$read$$iw$$iw$$anonfun$1.applyOrElse(<pastie>:16)
  at scala.concurrent.Future.$anonfun$andThen$1(Future.scala:533)
  at scala.concurrent.impl.Promise.liftedTree1$1(Promise.scala:29)
  at scala.concurrent.impl.Promise.$anonfun$transform$1(Promise.scala:29)
  at scala.concurrent.impl.CallbackRunnable.run(Promise.scala:60)
  at scala.concurrent.impl.ExecutionContextImpl$AdaptedForkJoinTask.exec(ExecutionContextImpl.scala:140)
  at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289)
  at java.util.concurrent.ForkJoinPool$WorkQueue.pollAndExecAll(ForkJoinPool.java:1021)
  at java.util.concurrent.ForkJoinPool$WorkQueue.execLocalTasks(ForkJoinPool.java:1046)
  at java.util.concurrent.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1058)
  at java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1692)
  at java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:157)
5
```
```
scala> f2.value
res37: Option[scala.util.Try[Int]] = Some(Success(5))
```

Futureの作成：Future.`fialed`、Future.`successful`、Future.`fromTry`   
Futureコンパニオンオブジェクトは、今までの例でFutureを作るために使ってきたapplyメソッドのほか、     
すでに完了したFutureを作るためのファクトリメソッドとして`successful`、`failed`、`fromTry`の三つを提供している。   
`注`:これらのファクトリメソッドは、ExecutionContextを必要としない。   

```
scala> import scala.concurrent.Future
import scala.concurrent.Future
```

`successful`ファクトリメソッドは、すでに成功したFutureを作る。
```
scala> Future.successful { 21 + 21 }
res1: scala.concurrent.Future[Int] = Future(Success(42))
```

`failed`ファクトリメソッドは、すでに失敗したFutureを作る。
```
scala> Future.failed(new Exception("test"))
res3: scala.concurrent.Future[Nothing] = Future(Failure(java.lang.Exception: test))
```

`fromTry`メソッドは、Tryからすでに完了したFutureを作る。
```
scala> import scala.util.{Success, Failure}
import scala.util.{Success, Failure}
```

```
scala> Future.fromTry(Success{21 + 21})
res4: scala.concurrent.Future[Int] = Future(Success(42))

scala> Future.fromTry(Failure(new Exception("test")))
res5: scala.concurrent.Future[Nothing] = Future(Failure(java.lang.Exception: test))
```

Future.`successful` と Future.`apply`区別:
```
scala> :paste
// Entering paste mode (ctrl-D to finish)

println("a")
val f1 = Future.successful { Thread.sleep(1000); println("b") }
println("c")

// Exiting paste mode, now interpreting.

a
b
c
f1: scala.concurrent.Future[Unit] = Future(Success(()))
```
```
scala> :paste
// Entering paste mode (ctrl-D to finish)

println("a")
import scala.concurrent.ExecutionContext.Implicits.global
val f2 = Future {
  Thread.sleep(1000)
  println("b")
}
println("c")

// Exiting paste mode, now interpreting.

a
c
import scala.concurrent.ExecutionContext.Implicits.global
f2: scala.concurrent.Future[Unit] = Future(<not completed>)

scala> b
```

