---
title: Connect to TiDB with SQLAlchemy
summary: Learn how to connect to TiDB using SQLAlchemy. This tutorial gives Python sample code snippets that work with TiDB using SQLAlchemy.
---

# SQLAlchemyを使用してTiDBに接続 {#connect-to-tidb-with-sqlalchemy}

TiDBはMySQL互換のデータベースであり、[SQLAlchemy](https://www.sqlalchemy.org/)は人気のあるPython SQLツールキットおよびオブジェクト関係マッパー（ORM）です。

このチュートリアルでは、TiDBとSQLAlchemyを使用して次のタスクを実行する方法を学ぶことができます。

- 環境を設定します。
- SQLAlchemyを使用してTiDBクラスタに接続します。
- アプリケーションをビルドおよび実行します。オプションで、基本的なCRUD操作のサンプルコードスニペットを見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedクラスタと動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Python 3.8以上](https://www.python.org/downloads/)。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- （推奨）[TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- （推奨）[TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1：サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-python-sqlalchemy-quickstart.git
cd tidb-python-sqlalchemy-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（SQLAlchemyとPyMySQLを含む）をインストールします：

```shell
pip install -r requirements.txt
```

#### なぜPyMySQLを使用するのですか？ {#why-use-pymysql}

SQLAlchemyは複数のデータベースと連携するORMライブラリです。これにより、データベースの高レベルな抽象化が提供され、開発者はよりオブジェクト指向の方法でSQLステートメントを記述できます。ただし、SQLAlchemyにはデータベースドライバが含まれていません。データベースに接続するには、データベースドライバをインストールする必要があります。このサンプルアプリケーションでは、TiDBと互換性のある純粋なPython MySQLクライアントライブラリであるPyMySQLをデータベースドライバとして使用し、すべてのプラットフォームにインストールできます。

また、[mysqlclient](https://github.com/PyMySQL/mysqlclient)や[mysql-connector-python](https://dev.mysql.com/doc/connector-python/en/)などの他のデータベースドライバを使用することもできます。ただし、これらは純粋なPythonライブラリではなく、対応するC/C++コンパイラとMySQLクライアントが必要です。詳細については、[SQLAlchemy公式ドキュメント](https://docs.sqlalchemy.org/en/20/core/engines.html#mysql)を参照してください。

### ステップ3：接続情報を構成する {#step-3-configure-connection-information}

選択したTiDBのデプロイオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致していることを確認します。

   - **Endpoint Type**が`Public`に設定されていること

   - **Branch**が`main`に設定されていること

   - **Connect With**が`General`に設定されていること

   - **Operating System**が環境に一致していること

   > **ヒント：**
   >
   > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. **Generate Password**をクリックしてランダムなパスワードを作成します。

   > **ヒント：**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

   ```shell
   cp .env.example .env
   ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```dotenv
   TIDB_HOST='{host}'  # 例：gateway01.ap-northeast-1.prod.aws.tidbcloud.com
   TIDB_PORT='4000'
   TIDB_USER='{user}'  # 例：xxxxxx.root
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   CA_PATH='{ssl_ca}'  # 例：/etc/ssl/certs/ca-certificates.crt（Debian / Ubuntu / Arch）
   ```

   接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、次に**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

   ```shell
   cp .env.example .env
   ```

5. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```dotenv
   TIDB_HOST='{host}'  # 例：tidb.xxxx.clusters.tidb-cloud.com
   TIDB_PORT='4000'
   TIDB_USER='{user}'  # 例：root
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   CA_PATH='{your-downloaded-ca-path}'
   ```

   接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換え、前の手順でダウンロードした証明書パスで`CA_PATH`を構成してください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

   ```shell
   cp .env.example .env
   ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```dotenv
   TIDB_HOST='{tidb_server_host}'
   TIDB_PORT='4000'
   TIDB_USER='root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   接続パラメータでプレースホルダ `{}` を置き換え、`CA_PATH`行を削除してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`で、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4：コードを実行して結果を確認する {#step-4-run-the-code-and-check-the-result}

1. 次のコマンドを実行してサンプルコードを実行します：

   ```shell
   python sqlalchemy_example.py
   ```

2. 出力が一致するかどうかを確認するには、[Expected-Output.txt](https://github.com/tidb-samples/tidb-python-sqlalchemy-quickstart/blob/main/Expected-Output.txt)を確認してください。

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-sqlalchemy-quickstart](https://github.com/tidb-samples/tidb-python-sqlalchemy-quickstart)リポジトリを確認してください。

### TiDBに接続 {#connect-to-tidb}

```python
from sqlalchemy import create_engine, URL
from sqlalchemy.orm import sessionmaker

def get_db_engine():
    connect_args = {}
    if ${ca_path}:
        connect_args = {
            "ssl_verify_cert": True,
            "ssl_verify_identity": True,
            "ssl_ca": ${ca_path},
        }
    return create_engine(
        URL.create(
            drivername="mysql+pymysql",
            username=${tidb_user},
            password=${tidb_password},
            host=${tidb_host},
            port=${tidb_port},
            database=${tidb_db_name},
        ),
        connect_args=connect_args,
    )

engine = get_db_engine()
Session = sessionmaker(bind=engine)
```

この機能を使用する場合、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`、`${ca_path}`をTiDBクラスターの実際の値で置き換える必要があります。

### テーブルの定義 {#define-a-table}

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class Player(Base):
    id = Column(Integer, primary_key=True)
    name = Column(String(32), unique=True)
    coins = Column(Integer)
    goods = Column(Integer)

    __tablename__ = "players"
```

詳細については、[SQLAlchemyドキュメント：デクラレーティブを使用したクラスのマッピング](https://docs.sqlalchemy.org/en/20/orm/declarative_mapping.html)を参照してください。

### データの挿入 {#insert-data}

```python
with Session() as session:
    player = Player(name="test", coins=100, goods=100)
    session.add(player)
    session.commit()
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

```python
with Session() as session:
    player = session.query(Player).filter_by(name == "test").one()
    print(player)
```

詳細については、[Query data](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### Update data {#update-data}

```python
with Session() as session:
    player = session.query(Player).filter_by(name == "test").one()
    player.coins = 200
    session.commit()
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

```python
with Session() as session:
    player = session.query(Player).filter_by(name == "test").one()
    session.delete(player)
    session.commit()
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ {#next-steps}

- [SQLAlchemyのドキュメント](https://www.sqlalchemy.org/)からSQLAlchemyのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を使用して、TiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
