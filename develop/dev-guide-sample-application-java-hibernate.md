---
title: Connect to TiDB with Hibernate
summary: Learn how to connect to TiDB using Hibernate. This tutorial gives Java sample code snippets that work with TiDB using Hibernate.
---

# Hibernateを使用してTiDBに接続する {#connect-to-tidb-with-hibernate}

TiDBはMySQL互換のデータベースであり、[Hibernate](https://hibernate.org/orm/)は人気のあるオープンソースのJava ORMです。バージョン`6.0.0.Beta2`から、HibernateはTiDBの方言をサポートし、TiDBの機能に適しています。

このチュートリアルでは、TiDBとHibernateを使用して次のタスクを達成する方法を学ぶことができます:

-   環境をセットアップする。
-   Hibernateを使用してTiDBクラスタに接続する。
-   アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です:

-   **Java Development Kit (JDK) 17** 以上。ビジネスおよび個人の要件に基づいて、[OpenJDK](https://openjdk.java.net/)または[Oracle JDK](https://www.oracle.com/java/technologies/downloads/)を選択できます。
-   [Maven](https://maven.apache.org/install.html) **3.8** 以上。
-   [Git](https://git-scm.com/downloads)。
-   TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-java-hibernate-quickstart.git
cd tidb-java-hibernate-quickstart
```

### ステップ2：接続情報を構成する {#step-2-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDBデプロイメントオプションに応じて行います。

<SimpleTab>
<div label="TiDBサーバーレス">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象クラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成が、操作環境に一致することを確認します。

    -   **エンドポイントタイプ** が `Public` に設定されていること

    -   **Branch** が `main` に設定されていること

    -   **Connect With** が `General` に設定されていること

    -   **Operating System** が環境に一致していること

    > **ヒント:**
    >
    > もしプログラムがWindows Subsystem for Linux (WSL)で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4.  **Generate Password** をクリックしてランダムなパスワードを作成します。

    > **ヒント:**
    >
    > もし以前にパスワードを作成している場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset Password** をクリックします。

5.  以下のコマンドを実行して `env.sh.example` をコピーし、`env.sh` にリネームします：

    ```shell
    cp env.sh.example env.sh
    ```

6.  対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例として以下のようになります：

    ```shell
    export TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{user}'  # 例: xxxxxx.root
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='true'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えるようにしてください。

    TiDBサーバーレスでは安全な接続が必要です。そのため、`USE_SSL` の値を `true` に設定する必要があります。

7.  `env.sh` ファイルを保存します。

</div>
<div label="TiDB専用">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象クラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  **どこからでもアクセスを許可** をクリックし、次に **TiDBクラスタCAをダウンロード** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB専用標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4.  以下のコマンドを実行して `env.sh.example` をコピーし、`env.sh` にリネームします：

    ```shell
    cp env.sh.example env.sh
    ```

5.  対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例として以下のようになります：

    ```shell
    export TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{user}'  # 例: root
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えるようにしてください。

6.  `env.sh` ファイルを保存します。

</div>
<div label="TiDBセルフホスト">

1.  以下のコマンドを実行して `env.sh.example` をコピーし、`env.sh` にリネームします：

    ```shell
    cp env.sh.example env.sh
    ```

2.  対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例として以下のようになります：

    ```shell
    export TIDB_HOST='{host}'
    export TIDB_PORT='4000'
    export TIDB_USER='root'
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    接続パラメータでプレースホルダ `{}` を置き換え、`USE_SSL` を `false` に設定してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `env.sh` ファイルを保存します。

</div>
</SimpleTab>

### ステップ3：コードを実行して結果を確認する {#step-3-run-the-code-and-check-the-result}

1.  サンプルコードを実行するには、次のコマンドを実行します：

    ```shell
    make
    ```

2.  出力が一致するかどうかを確認するには、[Expected-Output.txt](https://github.com/tidb-samples/tidb-java-hibernate-quickstart/blob/main/Expected-Output.txt) を参照してください。

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-java-hibernate-quickstart](https://github.com/tidb-samples/tidb-java-hibernate-quickstart) リポジトリをチェックしてください。

### TiDB に接続する {#connect-to-tidb}

Hibernate 構成ファイル `hibernate.cfg.xml` を編集します：

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>

        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.dialect">org.hibernate.dialect.TiDBDialect</property>
        <property name="hibernate.connection.url">${tidb_jdbc_url}</property>
        <property name="hibernate.connection.username">${tidb_user}</property>
        <property name="hibernate.connection.password">${tidb_password}</property>
        <property name="hibernate.connection.autocommit">false</property>

        <!-- Required so a table can be created from the 'PlayerDAO' class -->
        <property name="hibernate.hbm2ddl.auto">create-drop</property>

        <!-- Optional: Show SQL output for debugging -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
    </session-factory>
</hibernate-configuration>
```

`${tidb_jdbc_url}`, `${tidb_user}`, そして `${tidb_password}` を、お使いの TiDB クラスターの実際の値に置き換えてください。その後、以下の関数を定義してください。

```java
public SessionFactory getSessionFactory() {
    return new Configuration()
            .configure("hibernate.cfg.xml")
            .addAnnotatedClass(${your_entity_class})
            .buildSessionFactory();
}
```

この機能を使用する際には、`${your_entity_class}` を自分のデータエンティティクラスに置き換える必要があります。複数のエンティティクラスを使用する場合は、それぞれのために `.addAnnotatedClass(${your_entity_class})` ステートメントを追加する必要があります。前述の機能は、Hibernateを構成するための一つの方法に過ぎません。構成で問題が発生した場合やHibernateについて詳しく学びたい場合は、[Hibernate公式ドキュメント](https://hibernate.org/orm/documentation) を参照してください。

### データの挿入または更新 {#insert-or-update-data}

```java
try (Session session = sessionFactory.openSession()) {
    session.persist(new PlayerBean("id", 1, 1));
}
```

[データの挿入](/develop/dev-guide-insert-data.md)と[データの更新](/develop/dev-guide-update-data.md)については、詳細情報を参照してください。

### データのクエリ {#query-data}

```java
try (Session session = sessionFactory.openSession()) {
    PlayerBean player = session.get(PlayerBean.class, "id");
    System.out.println(player);
}
```

[クエリデータ](/develop/dev-guide-get-data-from-single-table.md)を参照して詳細情報をご覧ください。

### データの削除 {#delete-data}

```java
try (Session session = sessionFactory.openSession()) {
    session.remove(new PlayerBean("id", 1, 1));
}
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ {#next-steps}

-   [Hibernateのドキュメント](https://hibernate.org/orm/documentation)からHibernateのさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章を使用して、TiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
-   プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を受講し、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。
-   Java開発者向けのコースを受講します：[JavaからTiDBを使用する](https://eng.edu.pingcap.com/catalog/info/id:212)。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
