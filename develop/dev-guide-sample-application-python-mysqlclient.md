---
title: Connect to TiDB with mysqlclient
summary: Learn how to connect to TiDB using mysqlclient. This tutorial gives Python sample code snippets that work with TiDB using mysqlclient.
---

# TiDBとmysqlclientを使用して接続する {#connect-to-tidb-with-mysqlclient}

TiDBはMySQL互換のデータベースであり、[mysqlclient](https://github.com/PyMySQL/mysqlclient)はPython向けの人気のあるオープンソースドライバです。

このチュートリアルでは、TiDBとmysqlclientを使用して次のタスクを実行する方法を学ぶことができます:

-   環境をセットアップする。
-   mysqlclientを使用してTiDBクラスタに接続する。
-   アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作のためのサンプルコードスニペットを見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、以下が必要です:

-   [Python **3.10** 以上](https://www.python.org/downloads/)
-   [Git](https://git-scm.com/downloads)
-   TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、以下のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、以下のように作成できます:**

-   (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
-   [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで以下のコマンドを実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart.git
cd tidb-python-mysqlclient-quickstart;
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysqlclient`を含む）をインストールしてください：

```shell
pip install -r requirements.txt
```

もしインストールの問題に遭遇した場合は、[mysqlclient公式ドキュメント](https://github.com/PyMySQL/mysqlclient#install)を参照してください。

### ステップ3：接続情報の設定 {#step-3-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDBデプロイメントオプションに応じて行います。

<SimpleTab>
<div label="TiDB Serverless">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象クラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  接続ダイアログの構成が、お使いの環境に合致していることを確認してください。

    -   **エンドポイントタイプ** が `Public` に設定されていること

    -   **Branch** が `main` に設定されていること

    -   **Connect With** が `General` に設定されていること

    -   **Operating System** がお使いの環境に合致していること

    > **ヒント:**
    >
    > もしプログラムがWindows Subsystem for Linux (WSL)で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4.  **Generate Password** をクリックしてランダムなパスワードを作成します。

    > **ヒント:**
    >
    > もし以前にパスワードを作成している場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset Password** をクリックしてください。

5.  以下のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

6.  対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例は以下の通りです:

    ```dotenv
    TIDB_HOST='{gateway-region}.aws.tidbcloud.com'
    TIDB_PORT='4000'
    TIDB_USER='{prefix}.root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH=''
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

    TiDB Serverlessは安全な接続が必要です。mysqlclientの `ssl_mode` はデフォルトで `PREFERRED` に設定されているため、`CA_PATH` を手動で指定する必要はありません。空のままにしてください。もし特別な理由で `CA_PATH` を手動で指定する必要がある場合は、異なるオペレーティングシステム用の証明書パスを取得するために [TLS connections to TiDB Serverless](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters) を参照してください。

7.  `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1.  [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象クラスタの名前をクリックして概要ページに移動します。

2.  右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3.  **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4.  以下のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

5.  対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例は以下の通りです:

    ```dotenv
    TIDB_HOST='{host}.clusters.tidb-cloud.com'
    TIDB_PORT='4000'
    TIDB_USER='{username}'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{your-downloaded-ca-path}'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換え、前の手順でダウンロードした証明書パスで `CA_PATH` を構成してください。

6.  `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1.  以下のコマンドを実行して、`.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

2.  対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例は以下の通りです:

    ```dotenv
    TIDB_HOST='{tidb_server_host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    接続パラメータでプレースホルダ `{}` を置き換えてください。また、`CA_PATH` 行を削除してください。もしTiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行して結果を確認します {#step-4-run-the-code-and-check-the-result}

1.  次のコマンドを実行して、サンプルコードを実行します：

    ```shell
    python mysqlclient_example.py
    ```

2.  出力が一致するかどうかを確認するために、[Expected-Output.txt](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart/blob/main/Expected-Output.txt) を確認してください。

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-mysqlclient-quickstart](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart) リポジトリを確認してください。

### TiDBへの接続 {#connect-to-tidb}

```python
def get_mysqlclient_connection(autocommit:bool=True) -> MySQLdb.Connection:
    db_conf = {
        "host": ${tidb_host},
        "port": ${tidb_port},
        "user": ${tidb_user},
        "password": ${tidb_password},
        "database": ${tidb_db_name},
        "autocommit": autocommit
    }

    if ${ca_path}:
        db_conf["ssl_mode"] = "VERIFY_IDENTITY"
        db_conf["ssl"] = {"ca": ${ca_path}}

    return MySQLdb.connect(**db_conf)
```

この機能を使用する際には、`${tidb_host}`, `${tidb_port}`, `${tidb_user}`, `${tidb_password}`, `${tidb_db_name}`、および`${ca_path}`をTiDBクラスターの実際の値で置き換える必要があります。

### データの挿入 {#insert-data}

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player = ("1", 1, 1)
        cursor.execute("INSERT INTO players (id, coins, goods) VALUES (%s, %s, %s)", player)
```

[データを挿入](/develop/dev-guide-insert-data.md) に関する詳細は、詳細情報を参照してください。

### データのクエリ {#query-data}

// requirement start
上記の要件に従って、元のテキストを日本語に翻訳してください。翻訳されたコンテンツから出力を開始し、コメント（例：// 翻訳されたコンテンツの開始/終了）を削除してください。

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM players")
        print(cur.fetchone()[0])
```

[クエリデータ](/develop/dev-guide-get-data-from-single-table.md)を参照して詳細情報をご覧ください。

### データを更新 {#update-data}

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player_id, amount, price="1", 10, 500
        cursor.execute(
            "UPDATE players SET goods = goods + %s, coins = coins + %s WHERE id = %s",
            (-amount, price, player_id),
        )
```

[データの更新](/develop/dev-guide-update-data.md)に関する詳細は、こちらを参照してください。

### データの削除 {#delete-data}

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player_id = "1"
        cursor.execute("DELETE FROM players WHERE id = %s", (player_id,))
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート {#useful-notes}

### ドライバーまたはORMフレームワークを使用しますか？ {#using-driver-or-orm-framework}

Pythonドライバーはデータベースへの低レベルなアクセスを提供しますが、開発者が次のことを行う必要があります：

-   データベース接続の手動の確立と解除。
-   データベーストランザクションの手動管理。
-   データ行（`mysqlclient`でタプルとして表される）をデータオブジェクトに手動でマッピングする。

複雑なSQL文を書く必要がない限り、[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワークの使用が推奨されます。例えば、[SQLAlchemy](/develop/dev-guide-sample-application-python-sqlalchemy.md)、[Peewee](/develop/dev-guide-sample-application-python-peewee.md)、Django ORMなどです。これにより、次のことができます：

-   接続とトランザクションの管理のための[定型コード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減します。
-   複数のSQL文の代わりにデータオブジェクトでデータを操作します。

## 次のステップ {#next-steps}

-   [mysqlclientのドキュメント](https://mysqlclient.readthedocs.io/)から`mysqlclient`のさらなる使用法を学びます。
-   [開発者ガイド](/develop/dev-guide-overview.md)の章でTiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
-   プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通して学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## 助けが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
