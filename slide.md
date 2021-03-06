Domala Hands-on
------
Doma勉強会 2017

[bakenezumi](https://github.com/bakenezumi)



Change Log
-----
- v1.0 - 2017-12-16
  - Doma勉強会 2017 in Tokyo用に新規作成<!-- .element: style="font-size:80%" -->
- v1.1 - 2017-12-25
  - play.api.libs.json.Json.{Reads, Writes}を使ってJsonマッピング定義をしていた箇所をplay.api.libs.json.Json.Format + inmapを使用するように修正<!-- .element: style="font-size:80%" -->



このHands-onでは[Domala](https://github.com/bakenezumi/domala)を利用した基本的な開発方法と、

<!-- .element: style="font-size:80%" -->

[Play Framework](https://www.playframework.com/)での利用方法をガイドします

<!-- .element: style="font-size:80%" -->



Domalaについて
-----

DomalaはScala用のDBアクセスライブラリです<!-- .element: style="font-size:80%" -->

[Doma2](https://github.com/domaframework/doma)のラッパーであり

<!-- .element: style="font-size:80%" -->

Scalaから利用しやすくしています<!-- .element: style="font-size:80%" -->

https://github.com/bakenezumi/domala

<!-- .element: style="font-size:80%; margin-top: 70px" -->



事前準備
-----
次のソフトウェアをインストール<!-- .element: style="font-size:80%" -->
* [JDK8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* [Git](https://git-scm.com/downloads)



事前準備
-----
[Mac]ターミナルから<!-- .element: style="font-size:80%; text-align:left; margin-left: 30px" -->

```sh
$ git clone https://github.com/bakenezumi/domala-handson
$ cd domala-handson
$ chmod 755 bin/sbt
$ bin/sbt run
```
[Win]コマンドプロンプトから<!-- .element: style="font-size:80%; text-align:left; margin-left: 30px" -->

```sh
> git clone https://github.com/bakenezumi/domala-handson
> cd domala-handson
> bin\sbt run
```



事前準備
-----

実行後<!-- .element: style="font-size:80%; text-align:left; margin-left: 30px" -->
```sh
Domala hands-on Setup is complete.
[success] Total time: ...
```
と出力されたら準備は完了です<!-- .element: style="font-size:80%; text-align:left; margin-left: 30px" -->



Hands-on全容
-----

1. コンソールアプリを作る
  1. Holder, Entity, Daoを作る
  1. Daoを使う
  1. 全てHolderにする
1. REST API Serverを作る<!-- .element: style="margin-top: 30px" -->
  1. Playアプリへの切り替え
  1. Jsonマッピングを修正
  1. 項目追加
  1. PUTの実装



project構成
-----

```
domala-handson/
 - app/             # Playアプリのソース（コンソールアプリでは使用しません）
 - bin/             # sbtの実行ファイル
 - conf/            # Playアプリの設定ファイル（コンソールアプリでは使用しません）
 - project/         # sbtの設定ファイル
 - src/             # コンソールアプリのソース（Playアプリでは使用しません）
 - repository/      # Domala関連ソース（双方のアプリで使用します）
 - build.sbt
```



1. コンソールアプリを作る
-----
  1. Holder, Entity, Daoを作る
  1. Daoを使う
  1. 全てHolderにする



1.1.  Holder, Entity, Daoを作る
-----

まずはDomala関連のソースを作ります<!-- .element: style="font-size:80%" -->

```diff
  domala-handson/
  - src/main/scala/sample
                    - AppConfig.scala
                    - SampleApp.scala
+ - repository/src/main/scala/sample/
+                              - ID.scala
+                              - Emp.scala
+                              - EmpDao.scala
  - build.sbt
```



1.1.  Holder, Entity, Daoを作る - 1

directoryが無いので作成します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

コンソールからcloneしたディレクトリにdomala-handsonに移動しmkdir<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

[Mac]<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```sh
$ mkdir -p repository/src/main/scala/sample
```

[Win]<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```sh
domala-handson> mkdir repository\src\main\scala\sample
```
作ったdirectoryにソースを作っていきます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



1.1.  Holder, Entity, Daoを作る - 2

[Holder](https://github.com/bakenezumi/domala/blob/master/notes/specification.md#holder-class)は[Doma2のDomain](https://doma.readthedocs.io/ja/stable/domain/)にあたるものです

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

永続化される実際の値を独自型にラップすることでコンパイラチェックやパターンマッチ等、静的型付き言語のメリットを得られやすくなります<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

case classに@Holderを注釈するか、AnyValを継承することで定義できます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*repository/src/main/scala/sample/ID.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
package sample

case class ID[ENTITY](value: Long) extends AnyVal
```



1.1.  Holder, Entity, Daoを作る - 3

[Entity](https://github.com/bakenezumi/domala/blob/master/notes/specification.md#entity-class)はcase classに@Entityを注釈して定義します

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

@Idや@Versionといった[Doma2のEntity](https://doma.readthedocs.io/ja/stable/entity/)と同じアノテーションをフィールドに付与できます

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*repository/src/main/scala/sample/Emp.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
package sample

import domala._

@Entity
case class Emp(
  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  @SequenceGenerator(sequence = "emp_id_seq")
  id: ID[Emp],
  name: String,
  age: Int,
  @Version
  version: Int) {
    def growOld: Emp =
      this.copy(age = this.age + 1)
}
```



1.1.  Holder, Entity, Daoを作る - 4

[Dao](https://github.com/bakenezumi/domala/blob/master/notes/specification.md#dao-trait)はtraitに@Daoを注釈して定義します

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

SQLは[Doma2のDao](https://doma.readthedocs.io/ja/stable/dao/)のように外部ファイルに記述することもできますが、アノテーションパラメータに記述することもできます。

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*repository/src/main/scala/sample/EmpDao.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
package sample

import domala._
import domala.jdbc.Result

@Dao
trait EmpDao {
  @Select("select * from emp where id = /* idd */''")
  def selectById(id: ID[Emp]): Option[Emp]

  @Select("select * from emp")
  def selectAll: Seq[Emp]

  @Insert
  def insert(emp: Emp): Result[Emp]

  @Update
  def update(emp: Emp): Result[Emp]

  @Delete
  def delete(emp: Emp): Result[Emp]

  @Script("""
  create table emp(
      id int not null primary key,
      name varchar(20),
      age int,
      version int not null
  );
  create sequence emp_id_seq start with 1;
  """)
  def create(): Unit

}
```



1.1.  Holder, Entity, Daoを作る - 5

プロジェクトディレクトリからsbtを起動します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

[Mac]<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```sh
$ bin/sbt
```
[Win]<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```sh
> bin\sbt
```

このコマンドでsbtコンソールが立ち上がります<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

起動に若干時間がかかるため上げっぱなしでOKです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

sbtコンソールからコンパイルします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
sbt:domala-handson> compile
```



1.1.  Holder, Entity, Daoを作る - 6

```sh
[error] <macro>:3:9: [DOMALA4092] [sample.EmpDao.selectById]のSQLの妥当検査に失敗しました
（[1]行目[38] 番目の文字付近）。詳細は次のものです。[DOMALA4067] 
SQL内の変数[idd]に対応するパラメータがメソッドに存在しません
（[3]番目の文字付近）。 
SQL[select * from emp where id = /* idd */'']。
[error]     def selectById(id: ID[Emp]): Option[Emp]
[error]         ^
[error] one error found
[error] (repository/compile:compileIncremental) Compilation failed
```
変数名が間違っていたのでコンパイルが失敗しました<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

直します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*repository/src/main/scala/sample/EmpDao.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```diff
-    "select * from emp where id = /* idd */''"
+    "select * from emp where id = /* id */''"
```



1.1.  Holder, Entity, Daoを作る - 7
```sh
sbt:domala-handson> compile
...
[success] Total time: ...
```

[success]が表示されたらOKです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



1.2.  Daoを使う
-----

Domalaで作ったDaoを使ってDBにアクセスします<!-- .element: style="font-size:80%" -->

```diff
  domala-handson/
  - src/main/scala/sample
                    - AppConfig.scala
-                   - SampleApp.scala
+                   - SampleApp.scala
  - repository/
     - src/main/scala/sample/
                       - ID.scala
                       - Emp.scala
                       - EmpDao.scala
  - build.sbt
```



1.2.  Daoを使う - 1

DBにアクセスする準備として[Config](https://github.com/bakenezumi/domala/blob/master/notes/specification.md#config-class)が必要ですが、このHands-onではprojectに内包しているH2DBを使うように設定済みです

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

実際にはAppConfig.scalaを接続先に合わせて変更してください<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

また別途JDBCドライバも必要です（sbtではproject-root/libにjarを置けばクラスパスが通ります）<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*src/main/scala/sample/AppConfig.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
package sample

import domala.jdbc.LocalTransactionConfig
import org.seasar.doma.jdbc.Naming
import org.seasar.doma.jdbc.dialect.H2Dialect
import org.seasar.doma.jdbc.tx.LocalTransactionDataSource

object AppConfig extends LocalTransactionConfig(
  dataSource = new LocalTransactionDataSource(
    "jdbc:h2:mem:sample", "", ""
  ),
  dialect = new H2Dialect(),
  naming = Naming.SNAKE_LOWER_CASE
) {
  Class.forName("org.h2.Driver")
}
```



1.2.  Daoを使う - 2

projectに予め用意してあったSampleApp.scalaからDaoを呼び出すようにします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*src/main/scala/sample/SampleApp.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```diff
object SampleApp extends App {
-   println("""
-  /$$$$$$$                                    /$$
- | $$__  $$                                  | $$
- | $$  \ $$  /$$$$$$  /$$$$$$/$$$$   /$$$$$$ | $$  /$$$$$$ 
- | $$  | $$ /$$__  $$| $$_  $$_  $$ |____  $$| $$ |____  $$
- | $$  | $$| $$  \ $$| $$ \ $$ \ $$  /$$$$$$$| $$  /$$$$$$$
- | $$  | $$| $$  | $$| $$ | $$ | $$ /$$__  $$| $$ /$$__  $$
- | $$$$$$$/|  $$$$$$/| $$ | $$ | $$|  $$$$$$$| $$|  $$$$$$$
- |_______/  \______/ |__/ |__/ |__/ \_______/|__/ \_______/
-  """)
-  println("""
-  /$$   /$$                           /$$
- | $$  | $$                          | $$
- | $$  | $$  /$$$$$$  /$$$$$$$   /$$$$$$$  /$$$$$$$        /$$$$$$  - /$$$$$$$ 
- | $$$$$$$$ |____  $$| $$__  $$ /$$__  $$ /$$_____//$$$$$$/$$__  $$| $$__  - $$
- | $$__  $$  /$$$$$$$| $$  \ $$| $$  | $$|  $$$$$$|______/ $$  \ $$| $$  \ - $$
- | $$  | $$ /$$__  $$| $$  | $$| $$  | $$ \____  $$      | $$  | $$| $$  | - $$
- | $$  | $$|  $$$$$$$| $$  | $$|  $$$$$$$ /$$$$$$$/      |  $$$$$$/| $$  | - $$
- |__/  |__/ \_______/|__/  |__/ \_______/|_______/        \______/ |__/  |- __/
-  """)
-  println("Domala hands-on Setup is complete.")
}
```



1.2.  Daoを使う - 3

App {}の中を全て消し、下記内容に書き換えます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*src/main/scala/sample/SampleApp.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
package sample

import domala._
import domala.jdbc.Config

object SampleApp extends App {
  private implicit lazy val config: Config = AppConfig
  private lazy val dao: EmpDao = EmpDao.impl

  Required {
    // create table & insert
    dao.create()
    val inserted = Seq(
      Emp(ID(-1), "scott", 10, -1),
      Emp(ID(-1), "allen", 20, -1)
    ).map(dao.insert)
    println(inserted)

    // idが2のEmpのageを +1 にupdate
    val updated =
      dao
        .selectById(ID(2))
        .map(_.growOld)
        .map(dao.update)
    println(updated)

    dao.selectAll.foreach(println)
    // =>
    //   Emp(ID(1),scott,10,1)
    //   Emp(ID(2),allen,21,2)
  }
} 
 ```



1.2.  Daoを使う - 3

sbtコンソールから先ほどのアプリを実行します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```sh
sbt:domala-handson> run
```

 ```sh
 ...
 select * from emp
12 13, 2017 12:00:45 午後 sample.EmpDao selectAll
情報: [DOMA2221] EXIT   : クラス=[sample.EmpDao], メソッド=[selectAll]
Emp(ID(1),scott,10,1)
Emp(ID(2),allen,21,2)
12 13, 2017 12:00:46 午後 org.seasar.doma.jdbc.tx.LocalTransaction commit
情報: [DOMA2067] ローカルトランザクション[693049242]をコミットしました。
12 13, 2017 12:00:46 午後 org.seasar.doma.jdbc.tx.LocalTransaction commit
情報: [DOMA2064] ローカルトランザクション[693049242]を終了しました。
[success] Total time: ...
 ```

上のような検索結果と共に[success]が表示されたらOKです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



1.3.  全てHolderにする
-----
Domalaでは特別な理由がない限りEntityのメンバーは<!-- .element: style="font-size:60%;" -->

全てHolderにすることを推奨します<!-- .element: style="font-size:60%;" -->

[Doma本家のガイド](http://doma.readthedocs.io/ja/stable/domain/)<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" --><span style="font-size:60%">より抜粋 </span><span style="font-size:40%">(ドメインをホルダーに変更しています)</span>

> ホルダークラスを利用することで、データベース上のカラムの型が同じあっても アプリケーション上意味が異なるものを別の型で表現できます。 これにより意味を明確にしプログラミングミスを事前に防ぎやすくなります。 また、ホルダークラスに振る舞いを持たせることでよりわかりやすいプログラミングが可能です。<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



1.3.  全てHolderにする - 1

追加するファイル<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
  domala-handson/
  - src/main/scala/sample
                    - AppConfig.scala
                    - SampleApp.scala
  - repository/
     - src/main/scala/sample/
                       - ID.scala
                       - Emp.scala
                       - EmpDao.scala
+                      - Name.scala
+                      - Age.scala
  - build.sbt
```



1.3.  全てHolderにする - 2

まずHolderを追加します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*repository/src/main/scala/sample/Name.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
package sample

case class Name(value: String) extends AnyVal
```

*repository/src/main/scala/sample/Age.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
package sample

case class Age(value: Int) extends AnyVal {
  def grow = Age(value + 1)
}
```



1.3.  全てHolderにする - 3

作ったHolderを使うようEntityを変更します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*repository/src/main/scala/sample/Emp.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```diff
 @Entity
 case class Emp(
   @Id
   @GeneratedValue(strategy = GenerationType.SEQUENCE)
   @SequenceGenerator(sequence = "emp_id_seq")
   id: ID[Emp],
-  name: String,
+  name: Name,
-  age: Int,
+  age: Age,
   @Version
   version: Int) {
     def growOld: Emp =
-      this.copy(age = this.age + 1)
+      this.copy(age = age.grow)
 }
```



1.3.  全てHolderにする - 4

アプリのファクトリ部分を変更します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*src/main/scala/sample/SampleApp.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
     val inserted = Seq(
-      Emp(ID(-1), "scott", 10, -1),
-      Emp(ID(-1), "allen", 20, -1)
+      Emp(ID(-1), Name("scott"), Age(10), -1),
+      Emp(ID(-1), Name("allen"), Age(20), -1)
     ).map(dao.insert)
```



1.3.  全てHolderにする - 5

実行します。<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
 sbt:domala-handson> run
 ...
 select * from emp
 12 13, 2017 12:44:57 午後 sample.EmpDao selectAll
 情報: [DOMA2221] EXIT   : クラス=[sample.EmpDao], メソッド=[selectAll]
+Emp(ID(1),Name(scott),Age(10),1)
+Emp(ID(2),Name(allen),Age(21),2)
 12 13, 2017 12:44:57 午後 org.seasar.doma.jdbc.tx.LocalTransaction commit
 情報: [DOMA2067] ローカルトランザクション[1156254207]をコミットしました。
 12 13, 2017 12:44:57 午後 org.seasar.doma.jdbc.tx.LocalTransaction commit
 情報: [DOMA2064] ローカルトランザクション[1156254207]を終了しました。
[success] ...
```

HolderにしたことでtoStringの内容が変わりました<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

コンソールアプリのHands-onは以上になります<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2. REST API Serverを作る
-----

  1. Playアプリへの切り替え
  1. Jsonマッピングを修正
  1. 項目追加
  1. PUTの実装



2.1. Playアプリへの切り替え
-----

アプリをコンソールからPlayに切り替えます<!-- .element: style="font-size:80%" -->

```diff
  domala-handson/
+  - app/sample/
      - AppConfig.scala
      - SampleModule.scala
      - SampleJsonConverter.scala
      - SampleController.scala
+  - conf/
      - evolutions/default/
         - 1.sql
      - application.conf
      - routes
-  - src/main/scala/sample
-                    - AppConfig.scala
-                    - SampleApp.scala
  - repository/
     - src/main/scala/sample/
                       - ID.scala
                       - Emp.scala
                       - EmpDao.scala
  - build.sbt
```



2.1.  Playアプリへの切り替え - 1

build.sbtの下記のコメント部を切り替えます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```diff
- //lazy val root = (project in file(".")).enablePlugins(PlayScala).settings(
+ lazy val root = (project in file(".")).enablePlugins(PlayScala).settings(
- lazy val root = (project in file(".")).settings(
+ //lazy val root = (project in file(".")).settings(
    inThisBuild(List(
      scalaVersion := "2.12.4",
      version := "0.1.0"
    )),
    name := "domala-handson",
    libraryDependencies ++= Seq(
      "com.h2database" % "h2" % "1.4.196",
      "com.typesafe.play" %% "play" % "2.6.9"
-     // , guice
-     // , jdbc
-     // , evolutions
+     , guice
+     , jdbc
+     , evolutions
    )
  ) dependsOn repository aggregate repository
```



2.1.  Playアプリへの切り替え - 2

修正後のbuild.sbt

```scala
lazy val root = (project in file(".")).enablePlugins(PlayScala).settings(
//lazy val root = (project in file(".")).settings(
  inThisBuild(List(
    scalaVersion := "2.12.4",
    version := "0.1.0"
  )),
  name := "domala-handson",
  libraryDependencies ++= Seq(
    "com.h2database" % "h2" % "1.4.196",
    "com.typesafe.play" %% "play" % "2.6.9"
    , guice
    , jdbc
    , evolutions
  )
) dependsOn repository aggregate repository

lazy val repository = (project in file("repository")).settings(
  inThisBuild(List(
    scalaVersion := "2.12.4",
    version := "0.1.0"
  )),
  metaMacroSettings,
  libraryDependencies ++= Seq(
    "org.scalameta" %% "scalameta" % "1.8.0" % Provided,    
    "com.github.domala" %% "domala" % "0.1.0-beta.8"
  )
)

lazy val metaMacroSettings: Seq[Def.Setting[_]] = Seq(
  addCompilerPlugin("org.scalameta" % "paradise" % "3.0.0-M10" cross CrossVersion.full),
  scalacOptions += "-Xplugin-require:macroparadise",
  scalacOptions in (Compile, console) ~= (_ filterNot (_ contains "paradise")) // macroparadise plugin doesn't work in repl yet.
)
```



2.1.  Playアプリへの切り替え - 3

build.sbtを編集した場合、sbtに再度読み込ませる必要があるため再起動します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
sbt:domala-handson> exit
```

```
$ bin/sbt 
...
[domala-handson] $
```

このようなプロンプトが出力されたらOKです。続いてPlayを実行します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
 [domala-handson] $ run
```

```sh
--- (Running the application, auto-reloading is enabled) ---

[info] p.c.s.AkkaHttpServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Enter to stop and go back to the console...)
```

9000番ポートでPlayが起動します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->




2.1.  Playアプリへの切り替え - 4

任意のブラウザから下記URLににアクセスします<!-- .element: style="font-size:60%; text-align:left; margin-left: 90px" -->

http://localhost:9000/@evolutions/apply/default?redirect=/employees

<!-- .element: style="font-size:60%; text-align:left; margin-left: 90px; margin-bottom: 50px" -->

ブラウザ上に以下JSONが表示されたらOKです<!-- .element: style="font-size:60%; text-align:left; margin-left: 90px" -->

>[{"id":{"value":1},"name":{"value":"SMITH"},"age":{"value":10},"version":1},{"id":{"value":2},"name":{"value":"ALLEN"},"age":{"value":20},"version":1}]

<!-- .element: style="font-size:60%; text-align:left" -->



2.1.  Playアプリへの切り替え - 5

Hands-on用projectにはいくつかPlay用に事前に準備しているファイルがあるためその説明をします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```
  domala-handson/
  - conf/
     - evolutions/default/
        - 1.sql
     - application.conf
     - routes
  - app/sample/
     - AppConfig.scala
     - SampleModule.scala
     - SampleJsonConverter.scala
     - SampleController.scala
```



2.1.  Playアプリへの切り替え - 6

*conf/evolutions/default/1.sql*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

PlayにはDBのマイグレーションをアプリケーション起動時に自動的に行う機能があり、その設定となります<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

このHands-onでは以下のscriptを流しています<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sql
# Users schema

# --- !Ups
create table emp(
    id int not null primary key,
    name varchar(20),
    age int,
    version int not null
);
create sequence emp_id_seq start with 3;
insert into emp (id, name, age, version) values(1, 'SMITH', 10, 1);
insert into emp (id, name, age, version) values(2, 'ALLEN', 20, 1);
```

https://www.playframework.com/documentation/2.6.x/Evolutions

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.1.  Playアプリへの切り替え - 7

*conf/application.conf*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

Playの環境情報の設定ファイルです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
play.http.secret.key = "changeme"

play.modules {
  enabled += sample.SampleModule   #DI設定Module
}

play.evolutions {
  db.default.enabled = true        #DBのマイグレーション行うか
}

db {
  default{
    url="jdbc:h2:mem:sample"
    doma.dialect=org.seasar.doma.jdbc.dialect.H2Dialect # DomaのDialect
    hikaricp.minimumIdle = 10      # 最小DBコネクションプールサイズ
    hikaricp.maximumPoolSize = 10  # 最大DBコネクションプールサイズ
  }
}

# DB接続用スレッドプール設定（Webのスレッドプールとは分ける）
jdbc.executor {
  executor = "thread-pool-executor"
  thread-pool-executor {
    fixed-pool-size = 10           
  }
  throughput = 1
}
```

https://www.playframework.com/documentation/2.6.x/Configuration

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.1.  Playアプリへの切り替え - 8

*conf/routes*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

PlayのHTTPルーティングの設定ファイルです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

全取得以外のAPIはコメントにしています<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
GET     /employees                  sample.SampleController.selectAll
# GET     /employees/:id              sample.SampleController.selectById(id: Int)
# POST    /employees                  sample.SampleController.insert
# PUT     /employees/:id              sample.SampleController.update(id: Int)
# DELETE  /employees/:id              sample.SampleController.delete(id: Int)

GET     /assets/*file               controllers.Assets.versioned(path="/public", file: Asset)

```
https://www.playframework.com/documentation/2.6.x/ScalaRouting

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.1.  Playアプリへの切り替え - 9

*app/sample/AppConfig.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

Playアプリ用DomalaのConfigです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

PlayのDIコンテナが管理するDataSource、Dialectを取得して設定する場合はこのようにします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
@Singleton
class AppConfig @Inject() (db: play.api.db.Database, dialect: Dialect) extends LocalTransactionConfig(
  dataSource = db.dataSource,
  dialect = dialect,
  naming = Naming.SNAKE_LOWER_CASE
)
```
https://github.com/bakenezumi/domala/blob/master/notes/specification.md#config-class

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.1.  Playアプリへの切り替え - 10

*app/sample/SampleModule.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

Dialect、Config、スレッドプールのDI設定をしています<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```scala
class SampleModule extends Module {
  override def bindings(env: Environment, conf: Configuration) = {
    Seq(
      bind[Dialect].toInstance(getDialect(env, conf, "default")),
      bind[domala.jdbc.Config].to(classOf[AppConfig]),
      bind[JdbcExecutionContext].to(classOf[JdbcExecutionContextImpl])
    )
  }
  private def getDialect(env: Environment, conf: Configuration, dbName: String): Dialect = {
    val key = s"db.$dbName.doma.dialect"
    val dialectClassName = conf.get[String](key)
    Class
      .forName(dialectClassName, false, env.classLoader)
      .newInstance().asInstanceOf[Dialect]
  }
}

// jdbc execution thread pool
trait JdbcExecutionContext extends ExecutionContext

@Singleton
class JdbcExecutionContextImpl @Inject()(system: ActorSystem)
  extends CustomExecutionContext(system, "jdbc.executor") with JdbcExecutionContext
```
https://www.playframework.com/documentation/2.6.x/Modules

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.1.  Playアプリへの切り替え - 11

*app/sample/SampleJsonConverter.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

Entity、及びHolderクラスをJsonへマッピングするルール定義を行っています<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

Json.formatはPlayが提供するマクロでcase classのフィールド名をキーにしたJson変換を自動的に行ってくれます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```scala
object EmpConverter {

  implicit def idFormat[T] = Json.format[ID[T]]

  implicit val nameFormat = Json.format[Name]

  implicit val ageFormat = Json.format[Age]

  implicit val empFormat = Json.format[Emp]

  implicit val writesResult = Json.writes[Result[Emp]]

}
```
https://www.playframework.com/documentation/2.6.x/ScalaJson

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.1.  Playアプリへの切り替え - 12

*app/sample/SampleController.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

routesから指定されるAction関数を生成するクラスです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

DIコンテナからConfig、スレッドプールを受け取りインスタンス化されます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```
@Singleton
class SampleController @Inject()
(val controllerComponents: ControllerComponents)
(implicit config: Config, ec: JdbcExecutionContext) extends BaseController {

  lazy val dao: EmpDao = DaoProvider.get[EmpDao](config)

  def selectAll = Action.async {
    Future { Required {
      dao.selectAll
    }} map {
      case Nil => NotFound("not found.")
      case emps => Ok(Json.toJson(emps))
    }
  }

}
```
https://www.playframework.com/documentation/2.6.x/ScalaActions

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.2. Jsonマッピングを修正
-----

Jsonは表示されましたが、Holderのフィールドが露呈しているのはあまりよろしくありません<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

この項では<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

>"id":{"value":1}, "name":{"value":"SMITH"}

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

これを<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

>"id":1", name":"SMITH"

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

こうします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.2. Jsonマッピングを修正 - 1

*app/sample/SampleJsonConverter.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

下記のように修正します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
object EmpConverter {
-  implicit def idFormat[T] = Json.format[ID[T]]
+  implicit def idFormat[T]: Format[ID[T]] =
+    implicitly[Format[Long]].inmap(v => ID[T](v), { case ID(v) => v })

-  implicit val nameFormat = Json.format[Name]
+  implicit val nameFormat: Format[Name] =
+    implicitly[Format[String]].inmap(v => Name(v), { case Name(v) => v })

-  implicit val ageFormat = Json.format[Age]
+  implicit val ageFormat: Format[Age] = 
+    implicitly[Format[Int]].inmap(v => Age(v), { case Age(v) => v })
...
```



2.2. Jsonマッピングを修正 - 2

Playはrunで実行した場合ホットリロードが働くため、sbtはそのままでブラウザをリロードします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

>[{"id":1,"name":"SMITH","age":10,"version":1},{"id":2,"name":"ALLEN","age":20,"version":1}]

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

いい感じになりました<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.3. 項目追加
-----

Empに項目を追加します<!-- .element: style="font-size:80%; margin-left: 30px; margin-bottom: 50px" -->

 性別 = 男 | 女 | その他
```diff
  - repository/
     - src/main/scala/sample/
                       - ID.scala
                       - Emp.scala
                       - EmpDao.scala
                       - Name.scala
                       - Age.scala
+                      - Sex.scala
```



2.3.  項目追加 - 1

DB定義の変更<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*conf/evolutions/default/1.sql*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```diff
# Users schema

# --- !Ups
 create table emp(
    id int not null primary key,
    name varchar(20),
    age int,
+   sex char(1),
    version int not null
 );
 create sequence emp_id_seq start with 3;
-insert into emp (id, name, age, version) values(1, 'SMITH', 10, 1);
-insert into emp (id, name, age, version) values(2, 'ALLEN', 20, 1);
+insert into emp (id, name, age, sex, version) values(1, 'SMITH', 10, 'M', 1);
+insert into emp (id, name, age, sex, version) values(2, 'ALLEN', 20, 'F', 1);
```



2.3.  項目追加 - 2

Holderを追加<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

AnyValは抽象型にできないため@Holderを注釈してHolderを定義します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*repository/src/main/scala/sample/Sex.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```scala
package sample

import domala.Holder

@Holder
sealed abstract class Sex(val value: String) 
object Sex {
  object Male extends Sex("M")
  object Female extends Sex("F")
  object Other extends Sex("O")
}
```



2.3.  項目追加 - 3

作ったHolderを持つようEntityを変更します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*repository/src/main/scala/sample/Emp.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->
```diff
 @Entity
 case class Emp(
   @Id
   @GeneratedValue(strategy = GenerationType.SEQUENCE)
   @SequenceGenerator(sequence = "emp_id_seq")
   id: ID[Emp],
   name: Name,
   age: Age,
+  sex: Sex,
   @Version
   version: Int) {
     def growOld: Emp =
       this.copy(age = age.grow)
 }
```



2.3. 項目追加 - 4

Jsonへのマッピングにも追加します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*app/sample/SampleJsonConverter.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
object EmpConverter {
  ...
  implicit val ageFormat: Format[Age] = implicitly[Format[Int]].inmap(v => Age(v), { case Age(v) => v })

+ implicit val sexFormat: Format[Sex] =
+   implicitly[Format[String]].inmap({
+     case "Male" => Sex.Male
+     case "Female" => Sex.Female
+     case _ => Sex.Other      
+   }, {
+     case Sex.Male => "Male"
+     case Sex.Female => "Female"
+     case Sex.Other => "Other"
+   })

  implicit val empFormat = Json.format[Emp]
  ...  
}
```



2.3. 項目追加 - 5

DB定義を変更したのでPlayを再起動します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
EnterでPlayを終了後
[domala-handson] $ exit
```

```sh
$ bin/sbt 
```

```
[domala-handson] $ run
```

Play再起動完了後ブラウザをリロードします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

>[{"id":1,"name":"SMITH","age":10,"sex":"Male","version":1},{"id":2,"name":"ALLEN","age":20,"sex":"Female","version":1}]

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

SMITHにMale、ALLENにFameleが追加されていればOKです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.4. PUTの実装
-----
RESTサーバーとしてWebでデータの変更ができるようにします<!-- .element: style="font-size:80%" -->

Hands-onではPUTを実装します<!-- .element: style="font-size:80%" -->



2.4. PUTの実装 - 1

まずはroutesのコメントを外します<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*conf/routes*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
 GET     /employees                  sample.SampleController.selectAll
 # GET     /employees/:id              sample.SampleController.selectById(id: Int)
 # POST    /employees                  sample.SampleController.insert
-# PUT     /employees/:id              sample.SampleController.update(id: Int)
+PUT     /employees/:id              sample.SampleController.update(id: Int)
 # DELETE  /employees/:id              sample.SampleController.delete(id: Int)
```



2.4. PUTの実装 - 2

Controllerに下のように実装を加えます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

*app/sample/SampleController.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
 @Singleton
 class SampleController @Inject()
 (val controllerComponents: ControllerComponents)
 (implicit config: Config, ec: JdbcExecutionContext) extends BaseController {

   lazy val dao: EmpDao = DaoProvider.get[EmpDao](config)
   ...
+  // Json request body to Emp
+  private def toEmp(request: Request[AnyContent]) =
+    request.body.asJson.map(_.as[Emp]) // use EmpConverter.readsEmp
+      .getOrElse(throw new RuntimeException("Request body colud not parse"))

+  def update(id: Int) = Action.async { request =>
+    Future { Required {
+      dao.update(toEmp(request).copy(id = ID(id)))
+    }} map { result =>
+      Ok(Json.toJson(result))
+    }
+  }

 }
```



2.4. PUTの実装 - 3

sbtコンソールとは別のターミナルからcurlで確認してみます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
$ curl -X PUT -H "Content-Type: application/json" \
 -d '{"id": 2, "name": "Allen", "age": 25, "sex": "Female", "version": 1}'\
 http://localhost:9000/employees/2
```

```js
{"count":1,"entity":{"id":2,"name":"Allen","age":25,"sex":"Female","version":2}}
```

ageが25、versionが2のレスポンスが返ってくればOKです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

GETでも確認してみます<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
$ curl http://localhost:9000/employees
[{"id":1,"name":"SMITH","age":10,"sex":"Male","version":1},{"id":2,"name":"Allen","age":25,"sex":"Female","version":2}]
```



2.4. PUTの実装 - 4

ただ先ほどのリクエストには難点があり、Jsonリクエストボディのidが無駄です<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
{"id": 2, "name": "Allen", "age": 25, "sex": "Female", "version": 1}
 ^^^^^^^^
```

試しにリクエストボディからidを外してリクエストします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
$ curl -X PUT -H "Content-Type: application/json" \
 -d '{"name": "Allen", "age": 30, "sex": "Female", "version": 2}'\
 http://localhost:9000/employees/2
```

```sh
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Execution exception</title>
        <link rel="shortcut icon" href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAGXRFWHRTb2Z"
...
```

エラーがHTMLで返ってきました<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.4. PUTの実装 - 5

domala-handson/logs/application.logをみてみると<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
...
Caused by: play.api.libs.json.JsResultException: JsResultException(errors:List((/id,List(JsonValidationError(List(error.path.missing),WrappedArray())))))
at play.api.libs.json.JsReadable.$anonfun$as$2(JsReadable.scala:25)
	at play.api.libs.json.JsError.fold(JsResult.scala:56)
...
```

idがないとだめなようです<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->



2.4. PUTの実装 - 6

回避策としてPlayのJson.WithDefaultValuesマクロを使います<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

https://www.playframework.com/documentation/2.6.x/api/scala/index.html#play.api.libs.json.Json$$DefaultValues

<!-- .element: style="font-size:50%; text-align:left; margin-left: 30px" -->

*app/sample/SampleJsonConverter.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
 object EmpConverter {
   ...
-  implicit val empFormat = Json.format[Emp]
+  implicit val empFormat = Json.using[Json.WithDefaultValues].format[Emp]
   ...  
 }
```

このようにすることでJsonのフィールドにない値にはEmpのコンストラクタに設定されたデフォルト値がセットされるようになります<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

\* Scala では case class Foo(value: Int = -1) のようにパラメータにデフォルト値を設定することができます<!-- .element: style="font-size:40%; text-align:left; margin-left: 30px" -->



2.4. PUTの実装 - 7

*repository/src/main/scala/sample/Emp.scala*

<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```diff
 @Entity
 case class Emp(
   @Id
   @GeneratedValue(strategy = GenerationType.SEQUENCE)
   @SequenceGenerator(sequence = "emp_id_seq")
-  id: ID[Emp],
+  id: ID[Emp] = ID(-1),
   name: Name,
   age: Age,
   sex: Sex,
   @Version
   version: Int) {
   def growOld: Emp =
     this.copy(age = age.grow)
 }
```



2.4. PUTの実装 - 8

再度リクエストします<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px" -->

```sh
$ curl -X PUT -H "Content-Type: application/json" \
 -d '{"name": "Allen", "age": 30, "sex": "Female", "version": 2}'\
 http://localhost:9000/employees/2
```

```
{"count":1,"entity":{"id":2,"name":"Allen","age":30,"sex":"Female","version":3}}
```

Hands-onの説明はここまでです。POST, DELETEはよかったら各自でやってみてください<!-- .element: style="font-size:60%; text-align:left; margin-left: 30px; margin-top: 70px" -->



ありがとうございました