# ビルドツール`sbt` を学ぼう

`sbt`について必要最低限を解説する。  
トピックは以下の通り。順に解説していく。

* `sbt`とは
* `sbt`プロジェクトの作成
* `sbt`の起動
* JARファイルの実行
* ライブラリの依存関係追加

## 1. `sbt(Simple Build Tool)`とは

`sbt`はScala/Javaに対応したビルドツール。  
ビルドツールは、アプリケーションのビルド、テスト、パッケージング、などのタスクをコンソール上から実行できる。

Javaでのビルドツールは、`maven`や`gradle`などが一般的に使われているが、Scalaでは`sbt`がデファクトスタンダードとなっている。  
なぜなら、`sbt`はScalaでシンプルに記述できるためである。
なので、今後Scalaプロジェクトにおいては、`sbt`をビルドツールとして利用する。Scalaで開発する場合は、Scalaの慣習に従いましょう。

なお、sbtはとても奥が深いため、詳し知りたい場合には以下を参照。  
https://www.scala-sbt.org/1.x/docs/ja/index.html


**<u>`sbt`の特徴</u>**

代表的な特徴を以下に記載する。

* ビルド定義をScalaで記述できる
    * XMLやJSONを利用しない。
* JARファイルのパッケージングとパブリッシュ
    * アプリケーションを配布する際に一般的にJARファイルにパッケージングする。また、JARをMavenリポジトリにパブリッシュして作成したライブラリを外部公開する場合がある。`sbt`はこれらに対応する。
* Scala/Javaの混合プロジェクトに対応
* 代表的なテストライブラリに対応
    * Specs/ScalaTest/JUnitなどのテストライブラリに対応していて、コンソールからテストを実行できる。
* ライブラリの依存関係の解決
    * Mavenリポジトリなどの外部サーバにあるJARファイルをダウンロードし、プロジェクトの依存関係に含めることができる。さらに、そのJARファイルに依存するJARも自動的に解決してくれる。
* プラグインによる拡張
    * 独自のプラグイン機能によって、さまざまなプラグインを利用できる。 

## 2. `sbt`プロジェクトの作成

ここでは、ターミナルからsbtプロジェクトを作成する方法を記載する。

```
sbt new [テンプレート名]
```

でプロジェクトを作成する。
このプロジェクト生成機能は`giter8`といわれるツールを利用しているらしい。  
指定できるテンプレートは、以下のURLから参照する。

https://github.com/foundweekends/giter8/wiki/giter8-templates


今回は、テンプレートを`scala/scala-seed.g8`としてプロジェクトを生成してみる。
なお、プロジェクト名は、`sandbox`としてみた。

```
$ sbt new scala/scala-seed.g8
[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nagakuray/.sbt/1.0/plugins
[info] Set current project to sbt-samplepj (in build file:/Users/nagakuray/sbt-samplePj/)

A minimal Scala project.

name [Scala Seed Project]: sandbox

Template applied in ./sandbox

```

`sandbox`内に生成されたファイルを確認すると、以下のファイルが生成されていることがわかる。

```
$ tree sandbox/
sandbox/
├── build.sbt
├── project
│   ├── Dependencies.scala
│   └── build.properties
└── src
    ├── main
    │ └── scala
    │     └── example
    │         └── Hello.scala
    └── test
        └── scala
            └── example
                └── HelloSpec.scala
```

ビルドに関連するであろう、よく使うことになるファイルは以下。

|ファイル名|説明|
|:---|:-----|
|`build.sbt` |`sbt`のビルド定義ファイル。|
|`project/build.properties`|`sbt`のバージョン設定ファイル|

**`build.sbt`**

```scala
import Dependencies._

lazy val root = (project in file(".")).
  settings(
    inThisBuild(List(
      organization := "com.example",
      scalaVersion := "2.12.4",
      version      := "0.1.0-SNAPSHOT"
    )),
    name := "Hello",
    libraryDependencies += scalaTest % Test
  )
```

`organization`は、組織名などを指定する。  
`name`はこのプロジェククトの名前で、`scalaVersion`はScalaバージョン、`version`はその名の通りプロジェクトのバージョンを設定する。  

これら4つの値はJARファイルの名前に反映される。  
この場合は`hello_2.12-0.1.0-SNAPSHOT.jar`という名前でJARファイルが生成される。

`libraryDependencies`だが、ここにはライブラリの依存関係を設定する。
ここでは、`project/Dependencies.scala`で設定した値を利用している。`libraryDependencies`の詳しい設定については後述する。

また、ここで`.settings`メソッド内で`:=`が使われているが、`build.sbt`のDSLであり**セッティング式**と呼ばれている。左辺がキーで右辺が値となっている。


**`project/build.properties`**

```scala
sbt.version=1.1.0
```

## 3. `sbt`の起動　

ここまでで、プロジェクトの設定が終わっているので、sbtを起動してみる。
`sbt`の起動モードとして、以下の2つのモードがある。

* バッチモード
    * `sbt`シェルを起動せず、コンパイルなどのコマンドを実行する。コマンドが終了するとOSのシェルに戻ります。実行の都度、`sbt`を起動するため、遅い。
* インタラクティブモード
    * `sbt`シェルが起動する。シェルに対して対話的にコンパイルなどのコマンドを実行できるため、バッチモードより早い。

ローカル環境で利用する際には、インタラクティブモードの利用がオススメ。  
バッチ実行は、サーバ上でビルドの実行を自動化するなどするに利用する方針が良いと思われる。

### 3.1. インタラクティブモード

インタラクティブモードの実行方法を、簡単に解説。
OSのシェル上から`sbt`と入力して、インタラクティブモードを実行する。  
`>`が表示されれば、インタラクティブモードになっている。

```
$ sbt

[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nagakuray/.sbt/1.0/plugins
[info] Loading project definition from /Users/nagakuray/sbt-samplePj/sandbox/project
[info] Loading settings from build.sbt ...
[info] Set current project to Hello (in build file:/Users/nagakuray/sbt-samplePj/sandbox/)
[info] sbt server started at local:///Users/nagakuray/.sbt/1.0/server/f3778b98a434aa7f3638/sock
sbt:Hello>
```

バッチモードと説明がかぶるため、詳細は`help`で確認してほしい。

```
> help

  about                                          Displays basic information about sbt and the build.
  tasks                                          Lists the tasks defined for the current project.
  settings                                       Lists the settings defined for the current project.
  reload                                         (Re)loads the current project or changes to plugins project or returns from it.
  new                                            Creates a new sbt build.
  projects                                       Lists the names of available projects or temporarily adds/removes extra builds to the session.
  project                                        Displays the current project or changes to the provided `project`.
  set [every] <setting>                          Evaluates a Setting and applies it to the current project.
  session                                        Manipulates session settings.  For details, run 'help session'.
  inspect [tree|uses|definitions|actual] <key>   Prints the value for 'key', the defining scope, delegates, related definitions, and dependencies.
  <log-level>                                    Sets the logging level to 'log-level'.  Valid levels: debug, info, warn, error
  plugins                                        Lists currently available plugins.
  last                                           Displays output from a previous command or the output from a specific task.
  last-grep                                      Shows lines from the last output for 'key' that match 'pattern'.
  export <tasks>+                                Executes tasks and displays the equivalent command lines.
  show <key>                                     Displays the result of evaluating the setting or task associated with 'key'.
  all <task>+                                    Executes all of the specified tasks concurrently.
  help                                           Displays this help message or prints detailed help on requested commands (run 'help <command>').
  completions                                    Displays a list of completions for the given argument string (run 'completions <string>').
  ; <command> (; <command>)*                     Runs the provided semicolon-separated commands.
  early(<command>)                               Schedules a command to run before other commands on startup.
  exit                                           Terminates the build.
  ~ <command>                                    Executes the specified command whenever source files change.

More command help available using 'help <command>' for:
  !, +, ++, +-, <, ^, ^^, alias, append, apply, client, eval, iflast, onFailure, reboot, shell, startServer
```

アプリケーションのビルドに不可欠なsbtのタスク(`tasks`)を確認する。
実用途としては、`task`を頻繁に利用することになるであろう。

まず、tasksと入力すると、  
ソースのコンパイルする`compile`やJARファイルを作成する`package`など、さまざまなタスクが、用意されていることがわかる。

なお、タスクごとのヘルプを参照したい場合は、プロンプト上から`help [タスク名]`を入力する。

```
> tasks

This is a list of tasks defined for the current project.
It does not list the scopes the tasks are defined in; use the 'inspect' command for that.
Tasks produce values.  Use the 'show' command to run the task and print the resulting value.

  bgRun            Start an application's default main class as a background job
  bgRunMain        Start a provided main class as a background job
  clean            Deletes files produced by the build, such as generated sources, compiled classes, and task caches.
  compile          Compiles sources.
  console          Starts the Scala interpreter with the project classes on the classpath.
  consoleProject   Starts the Scala interpreter with the sbt and the build definition on the classpath and useful imports.
  consoleQuick     Starts the Scala interpreter with the project dependencies on the classpath.
  copyResources    Copies resources to the output directory.
  doc              Generates API documentation.
  package          Produces the main artifact, such as a binary jar.  This is typically an alias for the task that actually does the packaging.
  packageBin       Produces a main artifact, such as a binary jar.
  packageDoc       Produces a documentation artifact, such as a jar containing API documentation.
  packageSrc       Produces a source artifact, such as a jar containing sources and resources.
  publish          Publishes artifacts to a repository.
  publishLocal     Publishes artifacts to the local Ivy repository.
  publishM2        Publishes artifacts to the local Maven repository.
  run              Runs a main class, passing along arguments provided on the command line.
  runMain          Runs the main class selected by the first argument, passing the remaining arguments to the main method.
  test             Executes all tests.
  testOnly         Executes the tests provided as arguments or all tests if no arguments are provided.
  testQuick        Executes the tests that either failed before, were not run or whose transitive dependencies changed, among those provided as arguments.
  update           Resolves and optionally retrieves dependencies, producing a report.

More tasks may be viewed by increasing verbosity.  See 'help tasks'

sbt:Hello> help package
Produces the main artifact, such as a binary jar.  This is typically an alias for the task that actually does the packaging.
```

例えば、コンパイル用のタスクを実行したい場合には、以下のように実行する。

```
> compile

[info] Updating ...
[info] Done updating.
[info] Compiling 1 Scala source to /Users/nagakuray/sbt-samplePj/sandbox/target/scala-2.12/classes ...
[info] Done compiling.
[success] Total time: 3 s, completed 2018/02/07 0:52:35
```

### 3.2. バッチモード

バッチモードで`clean`して`compile`するコマンドは以下。

```
$ sbt clean compile

[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nagakuray/.sbt/1.0/plugins
[info] Loading project definition from /Users/nagakuray/sbt-samplePj/sandbox/project
[info] Loading settings from build.sbt ...
[info] Set current project to Hello (in build file:/Users/nagakuray/sbt-samplePj/sandbox/)
[info] Executing in batch mode. For better performance use sbt's shell
[success] Total time: 0 s, completed 2018/02/07 0:54:15
```


以下、代表的な`task`の実行方法について記載する。  
なお、`task`はインタラクティブモードでも実行できるので、ローカル環境で実行する場合には、インタラクティブモードで利用するように心がける。

**`sbt compile`**

classファイルを作成する。

```
$ sbt compile

[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nagakuray/.sbt/1.0/plugins
[info] Loading project definition from /Users/nagakuray/sbt-samplePj/sandbox/project
[info] Loading settings from build.sbt ...
[info] Set current project to Hello (in build file:/Users/nagakuray/sbt-samplePj/sandbox/)
[info] Executing in batch mode. For better performance use sbt's shell
[success] Total time: 0 s, completed 2018/02/07 9:46:54
```

**`sbt clean`**

コンパイル結果として生成された`class`ファイルを削除したい場合は、以下のコマンドを実行する。  
`target/scala-2.12`ディレクトリ自体が削除される。

```
$ sbt clean

[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nagakuray/.sbt/1.0/plugins
[info] Loading project definition from /Users/nagakuray/sbt-samplePj/sandbox/project
[info] Loading settings from build.sbt ...
[info] Set current project to Hello (in build file:/Users/nagakuray/sbt-samplePj/sandbox/)
[success] Total time: 0 s, completed 2018/02/07 0:58:27
```

**`sbt run`**

プログラムの実行は、以下。

```
$ sbt run

[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nagakuray/.sbt/1.0/plugins
[info] Loading project definition from /Users/nagakuray/sbt-samplePj/sandbox/project
[info] Loading settings from build.sbt ...
[info] Set current project to Hello (in build file:/Users/nagakuray/sbt-samplePj/sandbox/)
[info] Updating ...
[info] Done updating.
[info] Compiling 1 Scala source to /Users/nagakuray/sbt-samplePj/sandbox/target/scala-2.12/classes ...
[info] Done compiling.
[info] Packaging /Users/nagakuray/sbt-samplePj/sandbox/target/scala-2.12/hello_2.12-0.1.0-SNAPSHOT.jar ...
[info] Done packaging.
[info] Running example.Hello
hello
[success] Total time: 3 s, completed 2018/02/07 1:00:07
```

**`sbt test`**

```
$ sbt test

[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nagakuray/.sbt/1.0/plugins
[info] Loading project definition from /Users/nagakuray/sbt-samplePj/sandbox/project
[info] Loading settings from build.sbt ...
[info] Set current project to Hello (in build file:/Users/nagakuray/sbt-samplePj/sandbox/)
[info] Compiling 1 Scala source to /Users/nagakuray/sbt-samplePj/sandbox/target/scala-2.12/test-classes ...
[info] Done compiling.
[info] HelloSpec:
[info] The Hello object
[info] - should say hello
[info] Run completed in 411 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 1, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[success] Total time: 4 s, completed 2018/02/07 1:00:48
```

**`sbt package`**

次は以下のコマンドを使って、JARファイルを作成する。
JARファイルは、`target/scala-2.12`ディレクトリに出力される。

```
$ sbt package

[info] Loading settings from idea.sbt ...
[info] Loading global plugins from /Users/nagakuray/.sbt/1.0/plugins
[info] Loading project definition from /Users/nagakuray/sbt-samplePj/sandbox/project
[info] Loading settings from build.sbt ...
[info] Set current project to Hello (in build file:/Users/nagakuray/sbt-samplePj/sandbox/)
[info] Packaging /Users/nagakuray/sbt-samplePj/sandbox/target/scala-2.12/hello_2.12-0.1.0-SNAPSHOT.jar ...
[info] Done packaging.
[success] Total time: 0 s, completed 2018/02/07 1:02:48
```


## 4. JARファイルの起動

作成したJARファイルは、Javaと同様にコマンドラインから単体で実行できる。


```
$ scala target/scala-2.12/hello_2.12-0.1.0-SNAPSHOT.jar
hello
```

## 5. ライブラリの依存関係の追加

開発を効率的に進めるため、車輪の発明をしないために、外部のライブラリを利用したい場合がある(DBのアクセスライブラリやFTPなどのクライアントライブラリ等)。

その場合は、`build.sbt`の`libraryDependencies`に設定を追加するとできるようになる。  
書き方についてはコメントを参照してほしい。

**`build.sbt`**

```scala
// project/Dependencies.scalaはここではインポートを無効化する。
// もちろん、project/Dependencies.scalaに依存関係をコーディングしてもOK
// import Dependencies._

lazy val root = (project in file(".")).
  settings(
    inThisBuild(List(
      organization := "com.example",
      scalaVersion := "2.12.4",
      version      := "0.1.0-SNAPSHOT"
    )),
    name := "Hello",

    // +=メソッドを使った書き方。
    libraryDependencies += "org.apache.derby" % "derby" % "10.4.1.3"
    
    // ++=メソッドを使った書き方。
    // 依存ライブラリのリストを一度に追加する。
    // 普段利用する際には、++=メソッドを使った書き方が一般的。
    libraryDependencies ++= Seq(
      "org.scalatest"   %% "scalatest" % "3.0.4" % Test,
      "org.scalikejdbc" %% "scalikejdbc"         % "2.5.2",
      "com.h2database"  %  "h2"                  % "1.4.195",
      "ch.qos.logback"  %  "logback-classic"     % "1.2.3"
    )
  )
```

ここで、`"org.scalatest" %% "scalatest" % "3.0.4" % Test`といった文法について解説する。

これは、

```
グループID %% アーティファクトID % リビジョン % コンフィグレーション
```
となっている。

`グループID %% アーティファクトID`の`%%`は、`グループID % アーティファクトID_Scalaバージョン`と同義である。  
この部分は、`%`で記述してもコンパイルできるが、Scalaでビルドされたライブラリでは`%%`を利用するのが慣習的らしい。

なお、ライブラリについては、Mavenリポジトリを参照するか、Google検索エンジンから情報を拾ってくるのが一般的。

[Maven repository]  
https://mvnrepository.com/

以下、グループID、アーティファクトリID、リビジョン、コンフィギュレーションについて簡単に解説する。

**グループID**  
そのライブラリが所属するグループです。一般的に、この文字列はドメイン名を逆順した形式になっている。

**アーティファクトID**  
ライブラリが持つユニークな識別子。

**リビジョン**  
ライブラリのバージョン。リビジョンには以下の規則がある。  
普段利用するときには、リビジョンを指定することが多い。

* `1.1.1`	: 指定したリビジョンを利用
* `1.1.+`	: 1.1.x系で最新のバージョンを利用
* `[1.1.1, 2.0.0]` : 1.1.1以上2.0.0以下を利用
* `[1.1.1, 2.0.0[` : 1.1.1以上2.0.0未満
* `]1.1.1, 2.0.0]` : 1.1.1を超えて2.0.0以下
* `]1.1.1, 2.0.0[` : 1.1.1を超えて2.0.0未満

**コンフィグレーション**  
コンフィギュレーションに指定できる代表的な値には以下がある。
普段使いとしては、以下だけ覚えておけば特に問題ない。

|スコープ|役割|
|:-------|:---| 
|Compile|**コンフィグレーションの指定を省略した場合のデフォルト値**。すべての状況でクラスパスに追加される。|
|Test|テストのときのみ必要な場合に指定。テストのコンパイルと実行のときにクラスパスに追加される。|

上記以外のコンフィギュレーションに指定できる値は、  
`LibraryManagementSyntax.class`を参照。

`LibraryManagementSyntax.class`

```scala
package sbt.librarymanagement
trait LibraryManagementSyntax extends scala.AnyRef with sbt.librarymanagement.LibraryManagementSyntax0 with sbt.librarymanagement.DependencyBuilders with sbt.librarymanagement.DependencyFilterExtra {
  type ExclusionRule = sbt.librarymanagement.InclExclRule
  final val ExclusionRule : sbt.librarymanagement.InclExclRule.type = { /* compiled code */ }
  type InclusionRule = sbt.librarymanagement.InclExclRule
  final val InclusionRule : sbt.librarymanagement.InclExclRule.type = { /* compiled code */ }
  final val Compile : sbt.librarymanagement.Configuration = { /* compiled code */ }
  final val Test : sbt.librarymanagement.Configuration = { /* compiled code */ }
  final val Runtime : sbt.librarymanagement.Configuration = { /* compiled code */ }
  final val IntegrationTest : sbt.librarymanagement.Configuration = { /* compiled code */ }
  final val Default : sbt.librarymanagement.Configuration = { /* compiled code */ }
  final val Provided : sbt.librarymanagement.Configuration = { /* compiled code */ }
  final val Optional : sbt.librarymanagement.Configuration = { /* compiled code */ }
}
```

依存ライブラリがダウンロードされるのは、`sbt compile`時である。  
ダウンロードされたライブラリは、ホームディレクトリ直下の`.ivy2`というディレクトリの`cache`ディレクトリに各ライブラリが保存される。  
`sbt`やIntelliJ IDEAでは、この`.ivy2`を参照してビルドを実行している。


