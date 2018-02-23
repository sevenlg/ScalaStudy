## Sparkアプリのテストについて
ここではsparkアプリのテストについて解説する。  
テストコードが入ったブランチは以下の通り。  
`https://106.162.183.6:3080/reception-history/SparkProc/tree/dev_wakabayashi`
  

### 利用するファイルについて
ここで説明するファイルについては以下の通り。

| ファイル名 | 概要 | 
|:---------|:----|
|SummaryStockMartAppForTest.scala|元々あったSummaryStockMartApp.scalaにmainメソッドとprivateメソッドを導入して、テストコードからメソッドを呼び出せるように修正したバージョン|
|SummaryStockMartAppTest.scala|上のアプリのテストコード。ScalaTestとSparkTestingBaseのAPIを利用してprivateメソッドのテストを行う。|
|各テストデータのCSVファイル | 処理対象となるDFや期待値となるDFをテストコード内で作成するために利用する。 |
|build.sbt| テストに必要なライブラリの依存定義を追加している。|
  
### 依存ライブラリについて
`build.sbt`に追加しているライブラリは以下の通り。

| ライブラリ名 | 概要 | バージョン |
|:-----------|:-----------|:---------|
|Spark Testing Base| Sparkのユニットテストを支援するためのライブラリ。| 2.2.0 |
|Scalca Test | おなじみのScalaTestのライブラリ。| 3.0.1 | 
|jackson-databind| SparkTestingBaseの依存ライブラリ。| 2.4.4 |
  
### build.sbtの追記内容
以下、build.sbtの依存ライブラリに関する記述である。
```
val testDep = Seq(
  "org.scalatest" %% "scalatest" % "3.0.1" % "test",
  "com.holdenkarau" %% "spark-testing-base" % "2.2.0_0.8.0" % "test",
  "com.fasterxml.jackson.core" % "jackson-databind" % "2.4ｚ.4" % "test"
)
・・・
lazy val oss = (project in file(".")).settings(commonSettings: _*)
  .settings(
    version := ossVer,
    libraryDependencies ++= commonDep,
    libraryDependencies ++= ossDep,
    libraryDependencies ++= testDep

```

### アプリのソースの書き方
まず第一にテストコードでテストできるようにアプリのソースを書く必要がある。  
メソッドが全くない状態でObjectに処理を全て書いてしまうと、テストコードからアプリの処理を呼び出せなくなり、そもそもテストができなくなる。  
よってテストコードからメソッ以下2点を考慮する。

- 1.mainメソッドを明示的に記述する。

```scala
Object SomeApplication extends App {
    ・・・
}
```

と記述するのではなく以下のように記述する。

```scala
Object SomeApplication {
    def main(args: String[]) {
        ・・・
        // privateメソッドでDFの処理を行う
        convertReceptionHistoryDF(・・・)
        ・・・
    }
}
```

- 2.CSVファイルやDBといった外部リソースからßのデータ読み込みはmainメソッドの冒頭で行う。  
読み込んだデータはDFに変換し、DFの処理を行うprivateメソッドのパラメータで渡す。  
以下、アプリの抜粋。

```scala
・・・
  def main(args: Array[String]) {
    val props = ConfigFactory.load()
    val envProps = props.getConfig(args(0))

    // csvファイルからDFを作成する.
    val (receptionHistoryDF, categoryResultDF) = createDfFromCsv(envProps)
    ・・・
  }

  private def createDfFromCsv(envProps: Config) = {
    val rootPath = envProps.getString("rootPath")

    // HDFSから応対履歴ファイルを読み込み
    val receptionHistoryPath = envProps.getString("hdfs.receptionHistoryPath")
    val receptionHistoryDF = spark.read
      .option("header", true)
      .option("multiLine", true)
      .csv(rootPath + receptionHistoryPath + "*.csv")

    // HDFSから大区分分類ファイルを読み込み
    val categoryResultPath = envProps.getString("hdfs.categoryResultPath")
    val categoryResultDF = spark.read
      .option("header", true)
      .option("multiLine", true)
      .csv(rootPath + categoryResultPath + "*.csv")
    (receptionHistoryDF, categoryResultDF)
  }

```

- 3.アプリの処理をprivateメソッドに分割する。  
アプリの処理をprivateメソッドに分割することでテストコードから呼び出せるようにしておく。  
例えば応対履歴データフレームを変換する処理は以下のようにprivateメソッドに分割する。  
入力パラメータと戻り値は共にDataFrame型としている。  

```scala
  private def convertReceptionHistoryDF(filteredReceptionHistoryDF: DataFrame) = {
    // 応対履歴データフレーム
    val convertedReceptionHistoryDF = filteredReceptionHistoryDF.select(
      filteredReceptionHistoryDF("reception_no"),
      udfConvertSearchYM(filteredReceptionHistoryDF("search_ym")).as("search_ym"),
      udfConvertUsagePeriod(filteredReceptionHistoryDF("usage_period")).as("usage_period"),
      udfConvertTerminalForm(filteredReceptionHistoryDF("terminal_form")).as("terminal_form"),
      udfConvertIphoneFlg(filteredReceptionHistoryDF("iphone_flg")).as("iphone_flg"),
      udfConvertMemberAge(filteredReceptionHistoryDF("member_age")).as("member_age"),
      udfConvertMemberSex(filteredReceptionHistoryDF("member_sex")).as("member_sex"),
      udfConvertPrivateCorpClass(filteredReceptionHistoryDF("private_corp_class")).as("private_corp_class"))
    convertedReceptionHistoryDF
  }
```
テストコードからSparkアプリのObject経由でprivateメソッドを呼び出し  
DFが正しく作成/変換されているかをチェックする。  
なおprivateメソッドはある程度意味のある単位で分割すること。 

- 4.DBやファイルへの結果出力はmainメソッドの最後に行う。
  以下ソースの抜粋。

```scala
・・・
  def main(args: Array[String]) {
    val props = ConfigFactory.load()
    val envProps = props.getConfig(args(0))

・・・
    val targetWriteDF = createWriteDF(convertedReceptionHistoryDF, convertedCategoryResultDF)
    targetWriteDF.persist()

    // 変換したDFを出力する.
    writeDF(envProps, targetWriteDF)
  }
```

### テストコードの書き方
実際のテストコード(SummaryStockMartAppTest.scala)の抜粋を以下に示す。
詳細についてはコードのコメントを参照。なおScalaTestのスタイルはFunSpecを採用している。

```scala
package jp.co.nttdata

import com.holdenkarau.spark.testing.DataFrameSuiteBase
import com.typesafe.config.ConfigFactory
import org.apache.spark.sql.DataFrame
import org.scalatest.{FunSpec, PrivateMethodTester}

/**
  * 集約版中間マート作成アプリのテストクラス
  */
// 最初にFunSuiteをextendsする.
// DataFrameSuiteBaseはSparkTestingBaseで提供されているDataFrameのテストに関するtraitである.
// PrivateMethodTesterはScalaTestのprivateメソッドをテストするためのtrairである.
class SummaryStockMartAppTest extends FunSpec with DataFrameSuiteBase with PrivateMethodTester {
  import spark.implicits._

  describe("中間マート作成アプリのテスト") {
    describe("過去2年分のデータを取得するテスト") {
      it("1.応対履歴データが正しく取得できること") {
        // 以下のsparkオブジェクトはspark-testing-baseで提供されているものである.
        // ユニットテストではアプリのSparkSessionは利用しないので注意.
        val targetDF = spark.read
          .option("header", true)
          .option("multiLine", true)
          // 処理対象となる応対履歴データのテストデータをロードしDFに変換する
          .csv("./src/test/csv/1_target_reception_history.csv")

        // 期待値(正しい値)となるDFを作るために期待値用のテストデータをロードしDFに変換する.
        val expectedDF = spark.read
          .option("header", true)
          .option("multiLine", true)
          .csv("./src/test/csv/1_expected_reception_history.csv")

        // テスト対象となるSparkアプリのオブジェクトを取得する.
        val targetApp = SummaryStockMartAppForTest

        // privateのgetFilteredReceptionHistoryDFメソッドを呼び出す.
        // privateメソッドはアプリのオブジェクトから直接,target.getFilteredReceptionHistoryDF()と書いて
        // 呼び出すことができない.そのため以下の処理を行う必要がある.
        val getFilteredReceptionHistoryDFMethod = PrivateMethod[DataFrame]('getFilteredReceptionHistoryDF)

        // テスト対象のオブジェクトに対してinvokePrivateを実行するとprivateメソッドが呼び出される.
        // 呼び出すprivateメソッドの引数には,上で作成した応対履歴データのテストデータDFをセットする.
        val actualDF = targetApp.invokePrivate(getFilteredReceptionHistoryDFMethod(targetDF))

        // assertDataFrameEqualsで期待値となるDFと実際にprivateメソッドを呼び出して取得したDFを比較する.
        // なお,assertDataFrameEqualsで比較する場合、期待値と実際値の順序性も一致している必要があるので注意.
        assertDataFrameEquals(expectedDF, actualDF)
      }

```

### テストコード作成のポイント
Sparkアプリのテストコード作成に関していくつかポイントがある。

- 1.テストコードを使う場合spark-submitコマンドを利用できない。  
そのためSparkSessionのオブジェクトはテスト対象となるSparkアプリで作成するのではなく,  
SparkTesingBaseで提供されているいものを利用する。(implicitで定義されている)
テストクラスで`with DataFramesSuite Base`と記述するとsparkオブジェクトが利用できる。  
sparkオブジェクトを使って、テストデータのCSVファイルからDFを作ることが可能となる。

- 2.テストデータは処理対象となるデータ(Target Data),期待値となるデータ(Expected data)を用意し、以下の流れでテストを行う。  
    - 2-1.テストしたい処理（メソッド）に対して処理対象データ(Target Data)から作成したDFをパラメータで渡す。  
    - 2-2.テストしたい処理が終わったら実際の結果(Actural Data)のDFが返却される。  
    - 2-3.期待値のDF(Expected Data)と実際の結果のDF(Actual Data)を比較して一致すればOK、一致しなければNGとする。

- 3.DataFrameの比較を行うassertDataFrameEqualsは、行(row)の順序性が一致している必要があるので、実際の結果データと一致するように期待値データを作成する必要がある。  
また、DFに対してgroupByを行うと結果の順序がわからなくなることがある。  
順序性が不明な場合,実際の結果データと期待値データのDFを文字列のリストに変換し、  
各リストをソートしてから比較すると良い。  
以下、テストクラスの末尾に`convertDfToSortedList`というメソッドを定義しており、これを呼び出す。

```
  /**
    * DFの行を文字列に変換しソート済みリストとして返却する
    * @param df 変換対象のDF
    * @return ソート済みリスト
    */
  private def convertDfToSortedList(df: DataFrame): List[String] = {
    import spark.implicits._
    df.map(_.mkString).take(df.count.toInt).toList.sorted
  }
```

以下のように`convertDfToSortedList`を利用する。

```scala
      it("5.書き込み用DFが正しく作成されること") {
        ・・・
        val actualList = convertDfToSortedList(actualDF)
        val expectedList = convertDfToSortedList(expectedDF)
        // ソート済みのリストを比較して一致するかをチェックする.
        assert(expectedList == actualList)
      }
```

### 副作用のテスト
中間マート作成アプリではDBやHiveのテーブルへ書き込み処理（副作用）が発生する。
副作用のテストでは、テストコードの中でDBやORCファイルからデータを取得して、  
DFが一致するかをチェックすれば良い。詳細は以下の通り。

```scala
    describe("書き込み用DFを出力するテスト") {
     ・・・
      it("6.書き込み用DFがHiveとMySQLのテーブルに正しく出力されること") {
        // DFの出力先をセットするためにenvPropsを作成している.
        // テストコード実行時はsrc/main/resourcesではなくsrc/test/resources配下のconfファイルを参照する.
        val props = ConfigFactory.load("summaryStockMart")
        // ここではサンプルとして"test"の設定を使っている
        val envProps = props.getConfig("test")

        // 書き込み用DFを作成
        val targetWriteDF = spark.read
          .option("header", true)
          .option("multiLine", true)
          .csv("./src/test/csv/6_target_write.csv")

        val targetApp = SummaryStockMartAppForTest
        // privateのwriteDFメソッドを呼び出す
        val writeDFMethod = PrivateMethod[DataFrame]('writeDF)
        // envPropsと書き込み用DFをパラメータで渡してwriteDFメソッドを呼び出す
        targetApp.invokePrivate(writeDFMethod(envProps, targetWriteDF))

        // Hiveのテーブルに正しく出力されたかをチェックする.
        // ここでは,ORCファイルから読み込んだDFが書き込み用DFと一致するかをチェックする.
        val actualDFFromORC = spark.read.orc(envProps.getString("rootPath") + envProps.getString("hive.summaryPath"))
        val actualListFromORC = convertDfToSortedList(actualDFFromORC)
        val expectedList = convertDfToSortedList(targetWriteDF)
        assert(expectedList == actualListFromORC)

        // MySQLのテーブルに正しく出力されたかをチェックする.
        // ここでは,JDBCを使って出力先テーブルからデータをDFにロードし,書き込み用DFと一致するかをチェックする.
        val actualDFFromDB = spark.read
          .format("jdbc")
          .option("url", envProps.getString("db.url"))
          .option("user"
            , envProps.getString("db.user"))
          .option("password", envProps.getString("db.password"))
          .option("dbtable", envProps.getString("db.tableName"))
          .load()
        val actualListFromDB = convertDfToSortedList(actualDFFromDB)
        assert(expectedList == actualListFromDB)
      }
```
## テストコードの実行
コンソールからプロジェクトフォルダのディレクトリ上で`sbt test`と実行すれば以下のようにテストコードが実行される。

```
nttdnoMacBook-Pro-2:SparkProc nttd$ sbt test
[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nttd/.sbt/1.0/plugins
[info] Loading settings from plugins.sbt ...
[info] Loading project definition from /Users/nttd/IdeaProjects/SparkProc/project
[info] Loading settings from build.sbt ...
[info] Set current project to SparkProc (in build file:/Users/nttd/IdeaProjects/SparkProc/)
[info] Compiling 1 Scala source to /Users/nttd/IdeaProjects/SparkProc/target/scala-2.11/test-classes ...
[info] Done compiling.
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=2048M; support was removed in 8.0
[info] SummaryStockMartAppTest:
18/02/19 16:36:41 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
18/02/19 16:36:43 WARN SparkContext: Using an existing SparkContext; some configuration may not take effect.
18/02/19 16:36:50 WARN ObjectStore: Version information not found in metastore. hive.metastore.schema.verification is not enabled so recording the schema version 1.2.0
18/02/19 16:36:50 WARN ObjectStore: Failed to get database default, returning NoSuchObjectException
18/02/19 16:36:51 WARN ObjectStore: Failed to get database global_temp, returning NoSuchObjectException
[info] 中間マート作成アプリのテスト
[info]   過去2年分のデータを取得するテスト
18/02/19 16:36:55 WARN SparkSession$Builder: Using an existing SparkSession; some configuration may not take effect.
[info]   - 1.応対履歴データが正しく取得できること
[info]   - 2.大区分分類のデータが正しく取得できること
[info]   DFの変換を行うテスト
[info]   - 3.応対履歴データのDFが正しく変換できること
[info]   - 4.大区分分類データのDFが正しく変換できること
[info]   書き込み用DFを出力するテスト
[info]   - 5.書き込み用DFが正しく作成されること
[info]   - 6.書き込み用DFがHiveとMySQLのテーブルに正しく出力されること
[info] ScalaTest
[info] Run completed in 31 seconds, 19 milliseconds.
[info] Total number of tests run: 6
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 6, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[info] Passed: Total 6, Failed 0, Errors 0, Passed 6
[success] Total time: 42 s, completed 2018/02/19 16:37:10
```

## テストの粒度について
今回サンプルで提示したテストコードは全体の処理を個別のprivateメソッドに分割してそれぞれテストを行っており、テストの粒度としてはかなり細かい。  
（ユニットテストレベルの粒度である。）  
実際の開発ではそこまで細かくテストすることは難しいと思われるので
メソッドの粒度を大きくしてアプリを作っても良い。
