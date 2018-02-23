## PlayFrameworkについて
PlayFrameworkは世の中にある数多くのWebアプリフレームワークの一つであるが、  
既存のWebアプリの処理モデルと大きく異なる性質を持つ。  
- JavaEEのようなマルチスレッドでリクエストを処理するのではなく、イベント駆動モデルで非同期処理を行う。  
クライアントが大量接続して際に、既存のスレッドモデルでは対処できなくなってきたため考案された。  
(C10K問題、クライアント1万台問題を解決する）  
  
　イベント駆動モデルを利用する場合、イベントループをブロックさせないように処理を行う必要がある。  
（イベントループはデフォルトのスレッドプール上のスレッドを使って実行される。）  
特にCPUインテンシブな処理やDBアクセスのようなI/Oでブロックするような処理は  
デフォルのスレッドプールではなく、別のスレッドプール上のスレッドを使って実行する必要がある。  
既存のフレームワークと同じような感じで普通に重い処理を書いてしまうと、  
イベントループのスレッドを占有してしまい本来処理できたはずの他のリクエストが後回しにされてしまう。  
  
## テンプレートからのプロジェクト作成
今回は新規にプロジェクト作成は行わないが、通常は以下の通りPlay標準のテンプレートからプロジェクトを生成していく。

- 最初にプロジェクトのフォルダ(ここではplay-scala-sampleとする)を作成しておく。
次に以下のようにコマンドライン上でsbtコマンドを実行する。すると標準のテンプレートからPlayのプロジェクトが作成される。

```
$ sbt new playframework/play-java-seed.g8
scala-seed.g8
[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nttd/.sbt/1.0/plugins
[info] Set current project to playframeworksample (in build file:/Users/nttd/IdeaProjects/PlayFrameworkSample/)

This template generates a Play Scala project

name [play-scala-seed]: play-scala-sample
organization [com.example]: jp.co.nttdata
play_version [2.6.11]:
sbt_version [1.0.4]:
scalatestplusplay_version [3.1.2]:

Template applied in ./play-scala-sample
```
* プロジェクトが作成されたので試しにコマンドラインで sbt runを実行してみる。  
warningが表示されるが、とりあえず気にしなくて良い。
ポート番号9000版でPlayのアプリケーションサーバが起動するので、  
ブラウザで`http://lcoalhost:9000/`にアクセスしてみる。  
"Welcome to Play!"と表示されれば正しくアプリが起動している。  
アプリが起動したことを確認したら`Ctrl-C`でAPサーバをシャットダウンしておく。

```
$ sbt run
[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nttd/.sbt/1.0/plugins
[info] Updating {file:/Users/nttd/.sbt/1.0/plugins/}global-plugins...
[info] Done updating.
[info] Loading settings from plugins.sbt,scaffold.sbt ...
[info] Loading project definition from /Users/nttd/IdeaProjects/PlayFrameworkSample/play-scala-sample/project
[info] Updating {file:/Users/nttd/IdeaProjects/PlayFrameworkSample/play-scala-sample/project/}play-scala-sample-build...
[info] Done updating.
[warn] Found version conflict(s) in library dependencies; some are suspected to be binary incompatible:
[warn] 	* org.webjars:webjars-locator-core:0.33 is selected over 0.32
[warn] 	    +- com.typesafe:npm_2.12:1.2.1                        (depends on 0.32)
[warn] 	    +- com.typesafe.sbt:sbt-web:1.4.3 (scalaVersion=2.12, sbtVersion=1.0) (depends on 0.32)
[warn] 	* org.codehaus.plexus:plexus-utils:3.0.17 is selected over {2.1, 1.5.5}
[warn] 	    +- org.apache.maven:maven-settings:3.2.2              (depends on 3.0.17)
[warn] 	    +- org.apache.maven:maven-repository-metadata:3.2.2   (depends on 3.0.17)
[warn] 	    +- org.apache.maven:maven-aether-provider:3.2.2       (depends on 3.0.17)
[warn] 	    +- org.apache.maven:maven-model:3.2.2                 (depends on 3.0.17)
[warn] 	    +- org.apache.maven:maven-core:3.2.2                  (depends on 3.0.17)
[warn] 	    +- org.apache.maven:maven-artifact:3.2.2              (depends on 3.0.17)
[warn] 	    +- org.apache.maven:maven-settings-builder:3.2.2      (depends on 3.0.17)
[warn] 	    +- org.apache.maven:maven-model-builder:3.2.2         (depends on 3.0.17)
[warn] 	    +- org.sonatype.plexus:plexus-sec-dispatcher:1.3      (depends on 1.5.5)
[warn] 	    +- org.eclipse.sisu:org.eclipse.sisu.plexus:0.0.0.M5  (depends on 2.1)
[warn] 	* com.google.guava:guava:23.0 is selected over {10.0.1, 16.0, 20.0}
[warn] 	    +- io.methvin:directory-watcher:0.3.2                 (depends on 23.0)
[warn] 	    +- com.fasterxml.jackson.datatype:jackson-datatype-guava:2.8.8 (depends on 10.0.1)
[warn] 	    +- org.eclipse.sisu:org.eclipse.sisu.plexus:0.0.0.M5  (depends on 10.0.1)
[warn] 	    +- com.spotify:docker-client:8.9.0                    (depends on 10.0.1)
[warn] Run 'evicted' to see detailed eviction warnings
[info] Loading settings from build.sbt ...
[info] Set current project to play-scala-sample (in build file:/Users/nttd/IdeaProjects/PlayFrameworkSample/play-scala-sample/)

--- (Running the application, auto-reloading is enabled) ---

[info] p.c.s.AkkaHttpServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Enter to stop and go back to the console...)

```

* 次にIntellij IDEAに作成したプロジェクトをインポートする。  
インポートするときにプロジェクトのタイプを聞かれるので`SBT`を選択する。

* インポートが完了したらsbt shellを起動し`run`を実行する。  
するとAPサーバがdevelopmentモードで起動する。  
developmentモードの場合、ソースを修正したら自動的に修正内容が反映される。


## Playのプロジェクトで作成するクラス
Playのプロジェクトでは一般的（？）に以下のように、レイヤーを意識してクラスを作成する。
以下のテーブルのレイヤー欄に記載している名前は、ドメイン駆動モデル(DDD)で定義されているものである。

| クラスの種別 | 概要 |レイヤー|
|:-----------|:-----------|:-----------|
|Viewクラス、Viewテンプレート|ブラウザに表示するUIを定義する。RestAPIを作成する場合はViewは不要。|ユーザインタフェース|
|Controllerクラス     |HttpリクエストとServiceの紐づけを行う。|アプリケーション|
|ActionBuilderクラス       |PlayのActionクラスを拡張する。|アプリケーション|
|FormInputクラス|リクエストパラメータを保持しバリデーションを行う。|アプリケーション
|Serviceクラス         |ドメインのロジック（ビジネスロジック）処理を行う。|ドメイン|
|Repositoryクラス       |データアクセス処理を行う。|インフラストラクチャ|

一般的には上位レイヤーは下位レイヤーに依存するが、下位レイヤーが上位レイヤーに依存することがあってはならない。
なお、上記以外にも以下のファイルがプロジェクトで必要になる。

| ファイルの種別 | 概要 |
|:-----------|:-----------|
|routesファイル|HTTPリクエストのルーティングを定義する。|
|Routerクラス|特定のルーティングをまとめる。|
|Moduleクラス|依存性注入(DependencyInjection)の設定を行う。|
|SQLファイル(テーブル定義)|後述するSlickのモデルオブジェクトを自動生成する際に利用する。|
|application.conf|Playに関する設定ファイル。主にDBやスレッドプールに関する設定を記述する。|
|build.sbt|ビルドやテスト等で利用する。|

## サンプルプロジェクトについて
今回説明するサンプルプロジェクトは以下に置いてある。

`https://106.162.183.6:3080/toStudy/samples/PlaySample/tree/dev_wakabayashi`

プロジェクトの構成は以下の通りである。（一部省略してある）

```
play-scala-sample/
├── build.sbt
├── app
|    ├── actions
|    |   └── CommonActionBuilder.scala
|    ├── domain
|    │   └── CustomerService.scala
|    ├── generator
|    │   └── SlickModelGenerator.scala
|    ├── models
|    │   └── Tables.scala
|    ├── repositories
|    │   └── CustomerRepository.scala
|    ├── v1
|    |   └- customers
|    |      ├── CustomerRepository.scala
|    |      └-- CustomerRouter.scala   
|    └─ Module.scala
├── conf
|    ├─ application.conf
|    ├── evolutions
|    |   └ 1.sql
|    ├─ logback.xml
|    ├─ messages
|    └- routes
└-- test
    └ 各テストクラス
```

以降各クラス/ファイルの詳細についてはコメントを記載しているので、そちらを参照すること。

## Slickを使ったDBアクセス
PlayのアプリではSlickを使ってDBアクセスを行うのが一般的である。  
以下の通りDBアクセスに必要な準備を行う。

* 1.application.confにDBの接続情報を記述する。例えば今回のサンプルではMySQLを利用するため以下のような定義を行なっている。

```
slick.dbs {
  default {
    # MySQLを利用する
    profile = "slick.jdbc.MySQLProfile$"

    # MySQLの接続設定を行う
    db {
      driver = "com.mysql.jdbc.Driver"
      url = "jdbc:mysql://127.0.0.1:3306/play_sample?characterEncoding=UTF8&useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC"
      user = "root"
      password = "nttd"
    }
  }
}
```
* 2.SQLファイル(テーブル定義)を用意する。
ここでは`slick-evolutions`を使ってSQLファイルからMySQLのテーブルを生成する。
`slick-evolutions`を使う場合は、`conf/evolustions/default/`配下に、`${番号}.sql`という名前のSQLファイルを用意しておく。
今回は`1.sql`というファイルに以下のようなcreate table文を記載した。テーブルはdevelopmentモードでAPサーバを起動後、何らかのリクエストを送信することで自動的に作成される。

```
# Example table schema
# --- !Ups
CREATE TABLE IF NOT EXISTS `customers` (
    `id`         bigint(20) NOT NULL AUTO_INCREMENT,
    `first_name`    varchar(30) NOT NULL,
    `last_name`    varchar(30) NOT NULL,
    `address`    varchar(200),
    `email`    varchar(100),
    `age`    integer ,
    `gender` varchar(1),
    `created_at` datetime DEFAULT CURRENT_TIMESTAMP,
    `updated_at` datetime DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);

# --- !Downs
DROP TABLE IF EXISTS customers;
```
* 3.Slickのモデルクラスを生成するためのジェネレータクラスを作成する。  
SlickではテーブルのレコードをORマッピングするためのモデルクラスが必要となる。  
モデルクラスは手動で作成しても良いが、Slick-codegenというジェネレータを利用した方が楽である。  
今回は`/app/generator/SlickModelGenerator.scala`というサンプルを用意しているので、詳細はそちらを参照。  
なお、ジェネレータクラスは普通のScalaアプリとして実装するので、ブラウザからではなくIntellijのメニューの`Run`を使って実行する。

* 4.Repositoryクラスで生成したモデルクラスを利用してDBアクセスを行う。   
今回は`/app/repository/CustomeRepository.scala`にSlickに関する処理を記述している。  

## Slickチートシート
Slickを使っているとこのSQLはどうやって処理すれば良いのか？という疑問点がたくさん出てくる。  
以下のチートシートを用意したので適宜参照すること。  

[前提]  
例として以下のようなModelクラスがあるとする。  
```scala
type Person = (Int,String,Int,Int)
class People(tag: Tag) extends Table[Person](tag, "PERSON") {
  def id = column[Int]("ID", O.PrimaryKey, O.AutoInc)
  def name = column[String]("NAME")
  def age = column[Int]("AGE")
  def addressId = column[Int]("ADDRESS_ID")
  def * = (id,name,age,addressId)
  def address = foreignKey("ADDRESS",addressId,addresses)(_.id)
}
lazy val people = TableQuery[People]

type Address = (Int,String,String)
class Addresses(tag: Tag) extends Table[Address](tag, "ADDRESS") {
  def id = column[Int]("ID", O.PrimaryKey, O.AutoInc)
  def street = column[String]("STREET")
  def city = column[String]("CITY")
  def * = (id,street,city)
}
lazy val addresses = TableQuery[Addresses]

```
代表的な操作の例を以下に示す。

[select]
```scala
// where id = 1を実行
// Slickでは"=="ではなく”==="を使って同値判定を行う.
db.run(people.filter { _.id === 1 }.result)
 
// where id = 1 で最初のヒットしたものを取得
db.run(people.filter { _.id === 1 }.result.head)

// where id in (1,2,3)を実行
db.run(people.filter { _.id.inSet(List(1, 2, 3)) }.result) 

// where name like ‘hoge%’ でヒットしたレコードのageを取得
db.run(people.filter { _.name.startsWith("hoge") }.map { _.age }.result)
```
[order by]
```scala
// idの昇順、nameの降順でソートする.
persons.sortBy(x => (x.id.asc, x.name.desc))
```
[group by]
```scala
//以下のSQLを実行する場合
// select ADDRESS_ID, AVG(AGE) from PERSON group by ADDRESS_ID  
db.run(people.groupBy(p => p.addressId)
       .map{ case (addressId, group) => (addressId, group.map(_.age).avg) }
       .result)
```
[having]
```scala
// 以下のSQLを実行する場合
// select ADDRESS_ID from PERSON group by ADDRESS_ID having avg(AGE) > 50
db.run(people.groupBy(p => p.addressId)
       .map{ case (addressId, group) => (addressId, group.map(_.age).avg) }
       .filter{ case (addressId, avgAge) => avgAge > 50 }
       .map(_._1).result)
```

[implicit inner join]
```scala
// 以下のSQLを実行する場合
// select P.NAME, A.CITY from PERSON P, ADDRESS A where P.ADDRESS_ID = a.id
db.run(people.flatMap(p =>
  addresses.filter(a => p.addressId === a.id)
           .map(a => (p.name, a.city))
).result)

// もしくはforを使って以下のように書ける.
db.run((for(p <- people;
     a <- addresses if p.addressId === a.id
 ) yield (p.name, a.city)
).result)
```
[explicit inner join]
```scala
// 以下のSQLを実行する場合
// select P.NAME, A.CITY from PERSON P join ADDRESS A on P.ADDRESS_ID = a.id
db.run(people join addresses on (_.addressId === _.id))
  .map{ case (p, a) => (p.name, a.city) }.result)
```
[outer join(left/right/full)]
```scala
// 以下のSQLを実行する場合
// select P.NAME,A.CITY from ADDRESS A left join PERSON P on P.ADDRESS_ID = a.id
// joinLeftのDSLを利用する.right,fullの場合も同様にjoinRight, joinFullのDSLを利用する.
db.run((addresses joinLeft people on (_.id === _.addressId))
  .map{ case (a, p) => (p.map(_.name), a.city) }.result)
```
[sub query]
サブクエリーも slickで記述できる。
```scala
// 以下のSQLを実行する場合
// select * from PERSON P where P.ID in (select ID from ADDRESS where CITY = 'New York City')
val address_ids = addresses.filter(_.city === "New York City").map(_.id)
db.run(people.filter(_.id in address_ids).result)
// 上は2行に渡って処理が記述されているが、実際は1行のクエリとして処理される。
```
[insert]
```scala
// 1行insert
db.run(people += ("foo", 3))

// Seqと+==を使って複数行insertも可能
db.run(people ++= Seq("foo", 20, 3), ("bar", 35, 4)))
```
[update]
```scala
// 以下のSQLを実行する場合
// update PERSON set NAME='hoge', AGE=100 where NAME='foo'
db.run(people.filter(_.name === "foo")
              .map(p => (p.name,p.age))
              .update(("hoge",100))
```
[delete]
```scala
// 以下のSQLを実行する場合
// delete PERSON where id = 1
db.run(people.filter { _.id === 1 }.delete)
```
[upsert]
```scala
// id = 10のカラムが存在すればupdate, なければinsert.
db.run(people.insertOrUpdate(("updated name", 35 3, 10))

insertOrUpdateではキーを明示的にセットする必要がある。（上記だとタプルの第4引数の10）
```
[case - when]
```scala
// 以下のSQLを実行する場合
// select
//    case
//      when ADDRESS_ID = 1 then 'A'
//      when ADDRESS_ID = 2 then 'B'
//    end
//  from PERSON P
db.run(people.map(p =>
  Case // Case,If,ThenのDSLを利用する
    If(p.addressId === 1) Then "A"
    If(p.addressId === 2) Then "B"
).result)
```
[SQLを直接実行]  
モデルクラスを使わずに直接SQLを書いて処理することも可能である。

```scala
db.run(sql"select * from PERSON where AGE >= 18 AND NAME = 'foo'".as[Person])
```

[実行するSQLの確認]
以下の通り実行するのSQLが確認できる
```scala
val query = people.filter { _.id === 1 }
println(query.result.statements)
```

## その他のトピック
今回のサンプルプログラムで紹介仕しきれなかったトピックについて以下に示す。

### スレッドプールとコネクションプール
TBD

### RestAPIの認証
TBD

## 参考情報
[Play Framework]  
TBD

[Slick]  
TBD

[イベント駆動]  
TBD

[依存性注入(Dependency Injection)]  
TBD

[ドメイン駆動開発]  
TBD

[APIの認証, OAuth2]  
TBD
