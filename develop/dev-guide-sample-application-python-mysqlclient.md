---
title: Connect to TiDB with mysqlclient
summary: Learn how to connect to TiDB using mysqlclient. This tutorial gives Python sample code snippets that work with TiDB using mysqlclient.
---

# mysqlclientを使用してTiDBに接続 {#connect-to-tidb-with-mysqlclient}

TiDBはMySQL互換のデータベースであり、[mysqlclient](https://github.com/PyMySQL/mysqlclient)はPython向けの人気のあるオープンソースドライバーです。

このチュートリアルでは、TiDBとmysqlclientを使用して次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップします。
- mysqlclientを使用してTiDBクラスターに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作のサンプルコードスニペットを見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Python **3.10** 以上](https://www.python.org/downloads/)。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターがない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターがない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続 {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローン {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart.git
cd tidb-python-mysqlclient-quickstart;
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysqlclient`を含む）をインストールします：

```shell
pip install -r requirements.txt
```

If you encounter installation issues, refer to the [mysqlclient official documentation](https://github.com/PyMySQL/mysqlclient#install).

### ステップ3: 接続情報を構成する {#step-3-configure-connection-information}

選択した TiDB デプロイメントオプションに応じて TiDB クラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が環境に一致することを確認します。

   - **Endpoint Type** が `Public` に設定されていること

   - **Branch** が `main` に設定されていること

   - **Connect With** が `General` に設定されていること

   - **Operating System** が環境に一致していること

   > **ヒント:**
   >
   > プログラムが Windows Subsystem for Linux (WSL) で実行されている場合は、対応する Linux ディストリビューションに切り替えてください。

4. ランダムなパスワードを作成するには、次のコマンドを実行します。

   ```shell
   cp .env.example .env
   ```

5. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{gateway-region}.aws.tidbcloud.com'
   TIDB_PORT='4000'
   TIDB_USER='{prefix}.root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   CA_PATH=''
   ```

   接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

   TiDB Serverless では安全な接続が必要です。mysqlclient の `ssl_mode` はデフォルトで `PREFERRED` に設定されているため、`CA_PATH` を手動で指定する必要はありません。空のままにしてください。ただし、特別な理由で `CA_PATH` を手動で指定する必要がある場合は、異なるオペレーティングシステム用の証明書パスを取得するために [TLS connections to TiDB Serverless](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters) を参照してください。

6. `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックして CA 証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します。

   ```shell
   cp .env.example .env
   ```

5. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{host}.clusters.tidb-cloud.com'
   TIDB_PORT='4000'
   TIDB_USER='{username}'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   CA_PATH='{your-downloaded-ca-path}'
   ```

   接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換え、前の手順でダウンロードした証明書パスで `CA_PATH` を構成してください。

6. `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します。

   ```shell
   cp .env.example .env
   ```

2. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{tidb_server_host}'
   TIDB_PORT='4000'
   TIDB_USER='root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   接続パラメータでプレースホルダ `{}` を置き換え、`CA_PATH` 行を削除してください。TiDB をローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` で、パスワードは空です。

3. `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

1. 次のコマンドを実行してサンプルコードを実行します。

   ```shell
   python mysqlclient_example.py
   ```

2. 出力が一致するかどうかを確認するには、[Expected-Output.txt](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart/blob/main/Expected-Output.txt) を確認してください。

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-mysqlclient-quickstart](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart) リポジトリを確認してください。

### TiDB に接続 {#connect-to-tidb}

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

この機能を使用する際には、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`、および`${ca_path}`をTiDBクラスターの実際の値に置き換える必要があります。

### データの挿入 {#insert-data}

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player = ("1", 1, 1)
        cursor.execute("INSERT INTO players (id, coins, goods) VALUES (%s, %s, %s)", player)
```

For more information, refer to [データの挿入](/develop/dev-guide-insert-data.md).

### データのクエリ {#query-data}

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM players")
        print(cur.fetchone()[0])
```

For more information, refer to [Query data](/develop/dev-guide-get-data-from-single-table.md).

### Update data {#update-data}

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player_id, amount, price="1", 10, 500
        cursor.execute(
            "UPDATE players SET goods = goods + %s, coins = coins + %s WHERE id = %s",
            (-amount, price, player_id),
        )
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

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

Pythonドライバーはデータベースへの低レベルなアクセスを提供しますが、開発者が次のことを行う必要があります。

- 手動でデータベース接続を確立および解除する。
- 手動でデータベーストランザクションを管理する。
- データ行（`mysqlclient`でタプルとして表される）をデータオブジェクトに手動でマッピングする。

複雑なSQLステートメントを書く必要がない限り、[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワーク（たとえば[SQLAlchemy](/develop/dev-guide-sample-application-python-sqlalchemy.md)、[Peewee](/develop/dev-guide-sample-application-python-peewee.md)、およびDjango ORM）を使用することをお勧めします。これにより、次のことができます。

- 接続とトランザクションの管理のための[定型コード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減する。
- 複数のSQLステートメントの代わりにデータオブジェクトでデータを操作する。

## 次のステップ {#next-steps}

- [mysqlclientのドキュメント](https://mysqlclient.readthedocs.io/)から`mysqlclient`のさらなる使用法を学ぶ。
- [開発者ガイド](/develop/dev-guide-overview.md)の章で、TiDBアプリケーション開発のベストプラクティスを学ぶ。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得する。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
