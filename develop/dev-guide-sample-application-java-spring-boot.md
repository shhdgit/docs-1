---
title: Connect to TiDB with Spring Boot
summary: Learn how to connect to TiDB using Spring Boot. This tutorial gives Java sample code snippets that work with TiDB using Spring Boot.
aliases: ['/tidb/v7.1/dev-guide-sample-application-spring-boot']
---

# Spring BootでTiDBに接続する {#connect-to-tidb-with-spring-boot}

TiDBはMySQL互換のデータベースであり、[Spring](https://spring.io/)はJava向けの人気のあるオープンソースのコンテナフレームワークです。このドキュメントでは、Springの使用方法として[Spring Boot](https://spring.io/projects/spring-boot)を使用します。

このチュートリアルでは、次のタスクを達成するために、TiDBを[Spring Data JPA](https://spring.io/projects/spring-data-jpa)および[Hibernate](https://hibernate.org/orm/)と一緒に使用する方法を学ぶことができます。

- 環境をセットアップします。
- HibernateとSpring Data JPAを使用してTiDBクラスタに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと連携します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- **Java Development Kit (JDK) 17** 以上。ビジネスおよび個人の要件に基づいて、[OpenJDK](https://openjdk.org/)または[Oracle JDK](https://www.oracle.com/hk/java/technologies/downloads/)を選択できます。
- [Maven](https://maven.apache.org/install.html) **3.8** 以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイする](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイする](/production-deployment-using-tiup.md)を参照して、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイする](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイする](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を参照して、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-java-springboot-jpa-quickstart.git
cd tidb-java-springboot-jpa-quickstart
```

### ステップ2：接続情報の構成 {#step-2-configure-connection-information}

選択したTiDBデプロイメントオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致することを確認します。

   - **Endpoint Type** が `Public` に設定されていること

   - **Branch** が `main` に設定されていること

   - **Connect With** が `General` に設定されていること

   - **Operating System** が環境に一致していること

   > **ヒント：**
   >
   > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. **Generate Password** をクリックしてランダムなパスワードを作成します。

   > **ヒント：**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**Reset Password** をクリックして新しいパスワードを生成できます。

5. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します：

   ```shell
   cp env.sh.example env.sh
   ```

6. 対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```shell
   export TIDB_HOST='{host}'  # 例：gateway01.ap-northeast-1.prod.aws.tidbcloud.com
   export TIDB_PORT='4000'
   export TIDB_USER='{user}'  # 例：xxxxxx.root
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='true'
   ```

   接続ダイアログから取得した接続パラメータで `{}` のプレースホルダを置き換えてください。

   TiDB Serverlessでは安全な接続が必要です。そのため、`USE_SSL` の値を `true` に設定する必要があります。

7. `env.sh` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します：

   ```shell
   cp env.sh.example env.sh
   ```

5. 対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```shell
   export TIDB_HOST='{host}'  # 例：tidb.xxxx.clusters.tidb-cloud.com
   export TIDB_PORT='4000'
   export TIDB_USER='{user}'  # 例：root
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='false'
   ```

   接続ダイアログから取得した接続パラメータで `{}` のプレースホルダを置き換えてください。

6. `env.sh` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します：

   ```shell
   cp env.sh.example env.sh
   ```

2. 対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```shell
   export TIDB_HOST='{host}'
   export TIDB_PORT='4000'
   export TIDB_USER='root'
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='false'
   ```

   `{}` のプレースホルダを接続パラメータで置き換え、`USE_SSL` を `false` に設定してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` で、パスワードは空です。

3. `env.sh` ファイルを保存します。

</div>
</SimpleTab>

### ステップ3：コードを実行して結果を確認する {#step-3-run-the-code-and-check-the-result}

1. 次のコマンドを実行してサンプルコードを実行します：

   ```shell
   make
   ```

2. 別のターミナルセッションでリクエストスクリプトを実行します：

   ```shell
   make request
   ```

3. 出力が一致するかどうかを確認するために、[Expected-Output.txt](https://github.com/tidb-samples/tidb-java-springboot-jpa-quickstart/blob/main/Expected-Output.txt) を確認します。

## サンプルコードスニペット {#sample-code-snippets}

独自のアプリケーション開発を完了するためのサンプルコードスニペットを参照できます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-java-springboot-jpa-quickstart](https://github.com/tidb-samples/tidb-java-springboot-jpa-quickstart) リポジトリをチェックしてください。

### TiDBに接続 {#connect-to-tidb}

構成ファイル `application.yml` を編集します：

```yaml
spring:
  datasource:
    url: ${TIDB_JDBC_URL:jdbc:mysql://localhost:4000/test}
    username: ${TIDB_USER:root}
    password: ${TIDB_PASSWORD:}
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    show-sql: true
    database-platform: org.hibernate.dialect.TiDBDialect
    hibernate:
      ddl-auto: create-drop
```

構成後、環境変数`TIDB_JDBC_URL`、`TIDB_USER`、および`TIDB_PASSWORD`をTiDBクラスターの実際の値に設定します。構成ファイルはこれらの環境変数のデフォルト設定を提供します。環境変数を構成しない場合、デフォルト値は次のとおりです。

- `TIDB_JDBC_URL`: `"jdbc:mysql://localhost:4000/test"`
- `TIDB_USER`: `"root"`
- `TIDB_PASSWORD`: `""`

### データ管理：`@Repository` {#data-management-repository}

Spring Data JPAは`@Repository`インターフェースを介してデータを管理します。`JpaRepository`が提供するCRUD操作を使用するには、`JpaRepository`インターフェースを拡張する必要があります。

```java
@Repository
public interface PlayerRepository extends JpaRepository<PlayerBean, Long> {
}
```

その後、`PlayerRepository`を必要とする任意のクラスで自動依存性注入のために`@Autowired`を使用できます。これにより、CRUD関数を直接使用できます。以下は例です：

```java
@Autowired
private PlayerRepository playerRepository;
```

### データの挿入または更新 {#insert-or-update-data}

```java
playerRepository.save(player);
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)と[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データのクエリ {#query-data}

```java
PlayerBean player = playerRepository.findById(id).orElse(null);
```

詳細については、[Query data](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの削除 {#delete-data}

```java
playerRepository.deleteById(id);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ {#next-steps}

- [Hibernateのドキュメント](https://hibernate.org/orm/documentation)からHibernateのさらなる使用法を学びます。

- このドキュメントで使用されているサードパーティ製のライブラリやフレームワークのさらなる使用法については、それぞれの公式ドキュメントを参照してください：

  - [Spring Frameworkのドキュメント](https://spring.io/projects/spring-framework)
  - [Spring Bootのドキュメント](https://spring.io/projects/spring-boot)
  - [Spring Data JPAのドキュメント](https://spring.io/projects/spring-data-jpa)
  - [Hibernateのドキュメント](https://hibernate.org/orm/documentation)

- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。

- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

- Java開発者向けのコースを通じて学びます：[JavaからTiDBと連携](https://eng.edu.pingcap.com/catalog/info/id:212)。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
