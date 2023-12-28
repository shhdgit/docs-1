---
title: Connect to TiDB with mysqlclient
summary: Learn how to connect to TiDB using mysqlclient. This tutorial gives Python sample code snippets that work with TiDB using mysqlclient.
---

# mysqlclientを使用してTiDBに接続する {#connect-to-tidb-with-mysqlclient}

TiDBはMySQL互換のデータベースであり、[mysqlclient](https://github.com/PyMySQL/mysqlclient)はPythonで人気のあるオープンソースのドライバーです。

このチュートリアルでは、以下のタスクを実行する方法を学ぶことができます：

- 環境をセットアップする。
- mysqlclientを使用してTiDBクラスターに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作のサンプルコードを見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedで動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です：

- [Python **3.10**以上](https://www.python.org/downloads/)。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターを持っていない場合は、次のように作成できます：**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](/production-deployment-using-tiup.md)を参照して、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターを持っていない場合は、次のように作成できます：**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を参照して、ローカルクラスターを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1：サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードのリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart.git
cd tidb-python-mysqlclient-quickstart;
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

サンプルアプリケーションに必要なパッケージ（`mysqlclient`を含む）をインストールするには、次のコマンドを実行してください。

```shell
pip install -r requirements.txt
```

インストールの問題が発生した場合は、[mysqlclient公式ドキュメント](https://github.com/PyMySQL/mysqlclient#install)を参照してください。

### ステップ3：接続情報を設定する {#step-3-configure-connection-information}

選択したTiDBデプロイオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、実行環境に一致していることを確認してください。

   - **Endpoint Type**は`Public`に設定されています。

   - **Branch**は`main`に設定されています。

   - **Connect With**は`General`に設定されています。

   - **Operating System**は、実行環境に一致しています。

   > **Tip:**
   >
   > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. ランダムなパスワードを作成するには、**Generate Password**をクリックします。

   > **Tip:**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成することができます。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{gateway-region}.aws.tidbcloud.com'
   TIDB_PORT='4000'
   TIDB_USER='{prefix}.root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   CA_PATH=''
   ```

   プレースホルダ`{}`を接続ダイアログから取得した接続パラメータで置き換えてください。

   TiDB Serverlessでは、安全な接続が必要です。mysqlclientの`ssl_mode`はデフォルトで`PREFERRED`に設定されているため、`CA_PATH`を手動で指定する必要はありません。空のままにしてください。ただし、特別な理由で`CA_PATH`を手動で指定する必要がある場合は、[TiDB ServerlessへのTLS接続](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)を参照して、異なるオペレーティングシステム用の証明書パスを取得することができます。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

   接続文字列を取得する詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

5. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{host}.clusters.tidb-cloud.com'
   TIDB_PORT='4000'
   TIDB_USER='{username}'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   CA_PATH='{your-downloaded-ca-path}'
   ```

   プレースホルダ`{}`を接続ダイアログから取得した接続パラメータで置き換え、前のステップでダウンロードした証明書パスを`CA_PATH`に設定してください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{tidb_server_host}'
   TIDB_PORT='4000'
   TIDB_USER='root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   プレースホルダ`{}`を接続パラメータで置き換え、`CA_PATH`の行を削除してください。TiDBをローカルで実行している場合は、デフォルトのホストアドレスは`127.0.0.1`で、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

1. サンプルコードを実行するには、次のコマンドを実行してください：

   ```shell
   python mysqlclient_example.py
   ```

2. 出力が一致するかどうかを確認するには、[Expected-Output.txt](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart/blob/main/Expected-Output.txt)を参照してください。

## サンプルコードスニペット {#sample-code-snippets}

あなたのアプリケーション開発を完成させるために、以下のサンプルコードスニペットを参照してください。

完全なサンプルコードと実行方法については、[tidb-samples/tidb-python-mysqlclient-quickstart](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart)リポジトリをチェックしてください。

### TiDBに接続する {#connect-to-tidb}

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

この機能を使用する際には、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`、`${ca_path}`を、お使いのTiDBクラスターの実際の値で置き換える必要があります。

### データの挿入 {#insert-data}

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player = ("1", 1, 1)
        cursor.execute("INSERT INTO players (id, coins, goods) VALUES (%s, %s, %s)", player)
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM players")
        print(cur.fetchone()[0])
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

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

- 手動でデータベース接続を確立し、解放する。
- 手動でデータベーストランザクションを管理する。
- データ行（`mysqlclient`でタプルとして表される）をデータオブジェクトに手動でマップする。

複雑なSQL文を書く必要がない限り、開発には[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワークを使用することをお勧めします。例えば、[SQLAlchemy](/develop/dev-guide-sample-application-python-sqlalchemy.md)、[Peewee](/develop/dev-guide-sample-application-python-peewee.md)、Django ORMなどです。これにより、次のことができます。

- 接続とトランザクションを管理するための[ボイラープレートコード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減する。
- SQL文の数ではなく、データオブジェクトを使用してデータを操作する。

## 次のステップ {#next-steps}

- [mysqlclientのドキュメント](https://mysqlclient.readthedocs.io/)から`mysqlclient`のより詳細な使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後、[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
