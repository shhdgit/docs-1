---
title: Connect to TiDB with JDBC
summary: Learn how to connect to TiDB using JDBC. This tutorial gives Java sample code snippets that work with TiDB using JDBC.
---

# JDBCを使用してTiDBに接続する {#connect-to-tidb-with-jdbc}

TiDBはMySQL互換のデータベースであり、JDBC（Java Database Connectivity）はJava用のデータアクセスAPIです。[MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/)は、JDBCのMySQL実装です。

このチュートリアルでは、次のタスクを実行するために、TiDBとJDBCを使用する方法を学ぶことができます。

- 環境をセットアップする。
- JDBCを使用してTiDBクラスタに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedで動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- **Java Development Kit（JDK）17**以上。ビジネスおよび個人の要件に基づいて、[OpenJDK](https://openjdk.org/)または[Oracle JDK](https://www.oracle.com/hk/java/technologies/downloads/)を選択できます。
- [Maven](https://maven.apache.org/install.html) **3.8**以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次のように作成できます。**

- （推奨）[TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイする](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイする](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次のように作成できます。**

- （推奨）[TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイする](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイする](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1：サンプルアプリリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-java-jdbc-quickstart.git
cd tidb-java-jdbc-quickstart
```

### ステップ2：接続情報の設定 {#step-2-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDBデプロイオプションに応じて設定を行います。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、実行環境に合致していることを確認します。

   - **Endpoint Type**が`Public`に設定されていること

   - **Branch**が`main`に設定されていること

   - **Connect With**が`General`に設定されていること

   - **Operating System**が実行環境に合致していること

   > **ヒント：**
   >
   > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. **Generate Password**をクリックして、ランダムなパスワードを作成します。

   > **ヒント：**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成することができます。

5. 次のコマンドを実行して、`env.sh.example`をコピーして`env.sh`にリネームします：

   ```shell
   cp env.sh.example env.sh
   ```

6. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例は以下のとおりです：

   ```shell
   export TIDB_HOST='{host}'  # e.g. gateway01.ap-northeast-1.prod.aws.tidbcloud.com
   export TIDB_PORT='4000'
   export TIDB_USER='{user}'  # e.g. xxxxxx.root
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='true'
   ```

   プレースホルダ`{}`を接続ダイアログから取得した接続パラメータで置き換えてください。

   TiDB Serverlessでは、安全な接続が必要です。そのため、`USE_SSL`の値を`true`に設定する必要があります。

7. `env.sh`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して、`env.sh.example`をコピーして`env.sh`にリネームします：

   ```shell
   cp env.sh.example env.sh
   ```

5. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例は以下のとおりです：

   ```shell
   export TIDB_HOST='{host}'  # e.g. tidb.xxxx.clusters.tidb-cloud.com
   export TIDB_PORT='4000'
   export TIDB_USER='{user}'  # e.g. root
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='false'
   ```

   プレースホルダ`{}`を接続ダイアログから取得した接続パラメータで置き換えてください。

6. `env.sh`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`env.sh.example`をコピーして`env.sh`にリネームします：

   ```shell
   cp env.sh.example env.sh
   ```

2. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例は以下のとおりです：

   ```shell
   export TIDB_HOST='{host}'
   export TIDB_PORT='4000'
   export TIDB_USER='root'
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='false'
   ```

   プレースホルダ`{}`を接続パラメータで置き換え、`USE_SSL`を`false`に設定してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `env.sh`ファイルを保存します。

</div>
</SimpleTab>

### ステップ3：コードを実行して結果を確認する {#step-3-run-the-code-and-check-the-result}

1. サンプルコードを実行するには、次のコマンドを実行します：

   ```shell
   make
   ```

2. 出力が一致するかどうかを確認するには、[Expected-Output.txt](https://github.com/tidb-samples/tidb-java-jdbc-quickstart/blob/main/Expected-Output.txt)を確認してください。

## サンプルコードの断片 {#sample-code-snippets}

あなた自身のアプリケーション開発を完了するために、以下のサンプルコードの断片を参照することができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-java-jdbc-quickstart](https://github.com/tidb-samples/tidb-java-jdbc-quickstart)リポジトリを確認してください。

### TiDBに接続する {#connect-to-tidb}

```java
public MysqlDataSource getMysqlDataSource() throws SQLException {
    MysqlDataSource mysqlDataSource = new MysqlDataSource();

    mysqlDataSource.setServerName(${tidb_host});
    mysqlDataSource.setPortNumber(${tidb_port});
    mysqlDataSource.setUser(${tidb_user});
    mysqlDataSource.setPassword(${tidb_password});
    mysqlDataSource.setDatabaseName(${tidb_db_name});
    if (${tidb_use_ssl}) {
        mysqlDataSource.setSslMode(PropertyDefinitions.SslMode.VERIFY_IDENTITY.name());
        mysqlDataSource.setEnabledTLSProtocols("TLSv1.2,TLSv1.3");
    }

    return mysqlDataSource;
}
```

この機能を使用する際には、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`を、お使いのTiDBクラスターの実際の値で置き換える必要があります。

### データの挿入 {#insert-data}

```java
public void createPlayer(PlayerBean player) throws SQLException {
    MysqlDataSource mysqlDataSource = getMysqlDataSource();
    try (Connection connection = mysqlDataSource.getConnection()) {
        PreparedStatement preparedStatement = connection.prepareStatement("INSERT INTO player (id, coins, goods) VALUES (?, ?, ?)");
        preparedStatement.setString(1, player.getId());
        preparedStatement.setInt(2, player.getCoins());
        preparedStatement.setInt(3, player.getGoods());

        preparedStatement.execute();
    }
}
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

```java
public void getPlayer(String id) throws SQLException {
    MysqlDataSource mysqlDataSource = getMysqlDataSourceByEnv();
    try (Connection connection = mysqlDataSource.getConnection()) {
        PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM player WHERE id = ?");
        preparedStatement.setString(1, id);
        preparedStatement.execute();

        ResultSet res = preparedStatement.executeQuery();
        if(res.next()) {
            PlayerBean player = new PlayerBean(res.getString("id"), res.getInt("coins"), res.getInt("goods"));
            System.out.println(player);
        }
    }
}
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

```java
public void updatePlayer(String id, int amount, int price) throws SQLException {
    MysqlDataSource mysqlDataSource = getMysqlDataSourceByEnv();
    try (Connection connection = mysqlDataSource.getConnection()) {
        PreparedStatement transfer = connection.prepareStatement("UPDATE player SET goods = goods + ?, coins = coins + ? WHERE id=?");
        transfer.setInt(1, -amount);
        transfer.setInt(2, price);
        transfer.setString(3, id);
        transfer.execute();
    }
}
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

```java
public void deletePlayer(String id) throws SQLException {
    MysqlDataSource mysqlDataSource = getMysqlDataSourceByEnv();
    try (Connection connection = mysqlDataSource.getConnection()) {
        PreparedStatement deleteStatement = connection.prepareStatement("DELETE FROM player WHERE id=?");
        deleteStatement.setString(1, id);
        deleteStatement.execute();
    }
}
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート {#useful-notes}

### ドライバーまたはORMフレームワークを使用しますか？ {#using-driver-or-orm-framework}

Javaドライバーはデータベースへの低レベルなアクセスを提供しますが、開発者が次のことを行う必要があります。

- データベース接続の手動の確立と解放。
- データベーストランザクションの手動管理。
- データ行をデータオブジェクトに手動でマッピングする。

複雑なSQL文を書く必要がない場合は、[Hibernate](/develop/dev-guide-sample-application-java-hibernate.md)、[MyBatis](/develop/dev-guide-sample-application-java-mybatis.md)、または[Spring Data JPA](/develop/dev-guide-sample-application-java-spring-boot.md)などの[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワークを使用することをお勧めします。これにより、次のことができます。

- 接続とトランザクションの管理に関する[ボイラープレートコード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減します。
- 複数のSQL文ではなく、データオブジェクトを使用してデータを操作します。

## 次のステップ {#next-steps}

- [MySQL Connector/Jのドキュメント](https://dev.mysql.com/doc/connector-j/en/)からMySQL Connector/Jの使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章で、TiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータの取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を受講し、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。
- Java開発者向けのコースを受講します：[JavaからTiDBを使用する](https://eng.edu.pingcap.com/catalog/info/id:212)。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
