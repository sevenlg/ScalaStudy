# ScalaTestについて

## 1. ScalaTestとは
Scalaでユニットテストを行うためのフレームワークにはScalaTest、spaces2、ScalaCheckなどがある。  
その中でもScalaTestはテストスタイルを任意にカスタマイズできるなど柔軟性に優れているため良く使用されている。  
今回のレッスンでは、ScalaTestの基本的な使用方法について説明していく。

## 2. ScalaTestの環境作成

### 2.1. sbtプロジェクトの作成
ScalaTestを行うための環境作成の始めとして、IntelliJでsbtプロジェクトを作成する。  
デフォルトで下記のようなディレクトリが作成されるが、  メインのコードは**src/main/scala**配下に、  
テストコードは**src/test/scala**配下に作成する。

```
ScalaTestSample
├── build.sbt
├── project
│   ├── build.properties
│   ├── plugins.sbt
│   ├── project
│   └── target
├── src
│   ├── main
│   │   └── scala
│   └── test
│       └── scala
└── target
```

### 2.2. sbtビルドファイルの作成
ScalaTestのライブラリを使用するためにsbtビルドファイルに、  
下記のような依存関係を設定する。
※sbtビルドファイルについては、次回の勉強会で詳しく説明があるので詳細は割愛する

```
libraryDependencies += "org.scalatest"   %% "scalatest" % "3.0.1" % Test
```

## 3. ScalaTestのスタイル
ScalaTestではユニットテストのクラスを記述するためのスタイルが複数あり、任意に選択することができます。
http://www.scalatest.org/user_guide/selecting_a_style

| スタイル | クラス名 | 特徴 |
|:--|:--|:--|
| FunSuite | org.scalatest.FunSuite | xUnitからのチームにとってFunSuiteは、BDDの利点のいくつかを提供しながら、快適で親しみやすいと感じています。FunSuiteは具体的なテスト名を記述しやすく、集中的なテストを書くことができ、ステークホルダー間のコミュニケーションを促進できる仕様の出力を生成します。 |
| FlatSpec | org.scalatest.FlatSpec | FlatSpecはxUnitのようにフラット（ネストがない）で単純な構造であり、xUnitからBDDに移行したいチームにとっては最初のステップになるだろう。"X should Y"、 "A must B" " などの記述ができる。|
| FunSpec | org.scalatest.FunSpec | RubyのRSpecツールから来ているチームFunSpecにとっては、非常によく知られています。より一般的には、BDDを好むチームでは、 FunSpecテキストを構造化するためのネストと穏やかなガイド（describeとit）は、仕様スタイルのテストを書くための優れた汎用的な選択肢を提供します。|
| WordSpec | org.scalatest.WordSpec | WordSpecはSpecsやSpecs2から来ているチームにとっては、使い慣れた感じがします。SpecsNのテストをScalaTestに移植するのが最も自然な方法です。WordSpecはテキストの書き方が非常に規範的なので、高度な規律を必要とするチームには、仕様テキストが適用されます。 |
| FreeSpec | org.scalatest.FreeSpec | 仕様書の作成方法については絶対的な自由（および指針なし）が与えられるためFreeSpec、BDDの経験があり、仕様書の構成方法に同意できるチームにとっては良い選択です。 |
| PropSpec | org.scalatest.PropSpec | PropSpecは、propertyのチェックだけでテストを書くチームに最適です。別のスタイル特性がメインユニットのテストスタイルとして選択されたときに、時折のテストマトリックスを記述するのにも適しています。 |
| FeatureSpec | org.scalatest.FeatureSpec | FeatureSpecトレイトは、受け入れ要件を定義するプログラマー以外の側と一緒に働くプログラマーのプロセスを促進することを含む、受入れテストを主な目的としています。 |
| RefSpec（JVMのみ） | org.scalatest.refspec.RefSpec | RefSpecはテストをメソッドとして定義することができます。これは、テストを関数として表すスタイルクラスと比較して、テストごとに1つの関数リテラルを保存します。関数リテラルの数が少なくなると、コンパイル時間が短縮され、クラスファイルが生成される回数が減り、ビルド時間を最小限に抑えることができます。その結果、Specスタティックコードジェネレータを使用して多数のテストをプログラマチックに生成する場合と同様に、ビルド時間が問題となる大規模プロジェクトでは、usingを使用することをお勧めします。 |

ｘUnitとは様々なプログラミング言語用のユニットテストのフレームーワークであり、  
Java用はJUnit、C用はCUnit、.NET対応用はNUnitと言うようにxの部分に言語の頭文字が入る。  

JUnitを使用していた人にとってはFunSuiteが記述方法が近いが、BDDの書き方に切り替えたい場合はFlatSpecが最適である（公式サイトでおすすめしている）  
今回の勉強会では、RubyのRSpecのような記述ができるFunSpecを使用することにする。

## 4. テストコードの作成

メインのコードを下記のように定義する。
```
class Calc(i: Int, j: Int) {
  /**
    * 足し算
    */
  def add = i + j

  /**
    * 引き算
    */
  def sub = i - j

  /**
    * 掛け算
    */
  def mul = i * j

  /**
    * 割り算
    */
  def div = i / j
}
```

テストコードは下記の通りになる。  
`FunSpec`を継承することにより、スタイルを`FunSpec` にすることができる。  
`describe`でテストコードの説明を記述し、`it`配下に実際のテストコードを記述する。  
また、`describe`は入れ子にすることができる。

```
import org.scalatest.FunSpec

class CalcTest extends FunSpec {

  describe("CalcTest") {

    describe("足し算のテスト") {
      it("足し算が正常に動作すること") {
        val calc = new Calc(1, 2)
        assert(calc.add == 3)
      }
    }

    describe("引き算のテスト") {
      it("引き算が正常に動作すること") {
        val calc = new Calc(2, 1)
        assert(calc.sub == 1)
      }
    }

    describe("掛け算のテスト") {
      it("掛け算が正常に動作すること") {
        val calc = new Calc(3, 6)
        assert(calc.mul == 18)
      }
    }

    describe("割り算のテスト") {
      it("割り算が正常に動作すること") {
        val calc = new Calc(18, 3)
        assert(calc.div == 6)
      }
    }
  }
}

```


テストコードの実行はIntelliJから`RUN`で実行するか、  
sbtからtestコマンドで実行できる。

```
[12:56:14] nttd $ sbt
[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nttd/.sbt/1.0/plugins
[info] Updating ProjectRef(uri("file:/Users/nttd/.sbt/1.0/plugins/"), "global-plugins")...
[info] Done updating.
[info] Loading settings from plugins.sbt ...
[info] Loading project definition from /Users/nttd/Scala/ScalaTestSample/project
[info] Updating ProjectRef(uri("file:/Users/nttd/Scala/ScalaTestSample/project/"), "scalatestsample-build")...
[info] Done updating.
[info] Loading settings from build.sbt ...
[info] Set current project to ScalaTestSample (in build file:/Users/nttd/Scala/ScalaTestSample/)
[info] sbt server started at local:///Users/nttd/.sbt/1.0/server/fefd5fbb37fcc610a4ed/sock
sbt:ScalaTestSample> test
[info] Compiling 1 Scala source to /Users/nttd/Scala/ScalaTestSample/target/scala-2.12/test-classes ...
[info] Done compiling.
[info] CalcTest:
[info] CalcTest
[info]   足し算のテスト
[info]   - 足し算が正常に動作すること
[info]   引き算のテスト
[info]   - 引き算が正常に動作すること
[info]   掛け算のテスト
[info]   - 掛け算が正常に動作すること
[info]   割り算のテスト
[info]   - 割り算が正常に動作すること
[info] Run completed in 383 milliseconds.
[info] Total number of tests run: 4
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 4, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[success] Total time: 2 s, completed 2018/02/02 12:56:42
```
















