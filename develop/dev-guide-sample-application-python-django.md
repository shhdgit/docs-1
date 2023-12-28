---
title: Connect to TiDB with Django
summary: Learn how to connect to TiDB using Django. This tutorial gives Python sample code snippets that work with TiDB using Django.
aliases: ['/tidb/v7.1/dev-guide-outdated-for-django','/tidb/stable/dev-guide-outdated-for-django']
---

# Djangoを使用してTiDBに接続する {#connect-to-tidb-with-django}

TiDBはMySQL互換のデータベースであり、[Django](https://www.djangoproject.com)はPythonの人気のあるWebフレームワークであり、強力なオブジェクト関係マッパー（ORM）ライブラリを含んでいます。

このチュートリアルでは、次のタスクを実行するために、TiDBとDjangoを使用する方法を学ぶことができます。

- 環境をセットアップする。
- Djangoを使用してTiDBクラスタに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作のためのサンプルコードスニペットを見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedクラスタで動作します。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Python 3.8以上](https://www.python.org/downloads/)。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次のように作成することができます。**

- （推奨）[TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイする](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイする](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次のように作成することができます。**

- （推奨）[TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイする](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイする](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1：サンプルアプリリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-python-django-quickstart.git
cd tidb-python-django-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

サンプルアプリケーションに必要なパッケージ（Django、django-tidb、およびmysqlclientを含む）をインストールするには、次のコマンドを実行してください：

```shell
pip install -r requirements.txt
```

mysqlclientのインストールに問題が発生した場合は、[mysqlclient公式ドキュメント](https://github.com/PyMySQL/mysqlclient#install)を参照してください。

#### `django-tidb`とは？ {#what-is-django-tidb}

`django-tidb`は、TiDBとDjangoの互換性の問題を解決するためのDjango用のTiDB方言です。

`django-tidb`をインストールするには、Djangoのバージョンに合わせたバージョンを選択してください。例えば、`django==4.2.*`を使用している場合は、`django-tidb==4.2.*`をインストールしてください。マイナーバージョンは同じである必要はありません。最新のマイナーバージョンを使用することをお勧めします。

詳細については、[django-tidbリポジトリ](https://github.com/pingcap/django-tidb)を参照してください。

### ステップ3：接続情報を設定する {#step-3-configure-connection-information}

TiDBクラスタに接続するには、選択したTiDBデプロイメントオプションに応じて接続してください。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が操作環境と一致することを確認してください。

   - **Endpoint Type**は`Public`に設定されています。

   - **Branch**は`main`に設定されています。

   - **Connect With**は`General`に設定されています。

   - **Operating System**は、環境に合わせて設定してください。

   > **ヒント：**
   >
   > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. ランダムなパスワードを作成するには、**Generate Password**をクリックしてください。

   > **ヒント：**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを作成することができます。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{host}'  # e.g. gateway01.ap-northeast-1.prod.aws.tidbcloud.com
   TIDB_PORT='4000'
   TIDB_USER='{user}'  # e.g. xxxxxx.root
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   CA_PATH='{ssl_ca}'  # e.g. /etc/ssl/certs/ca-certificates.crt (Debian / Ubuntu / Arch)
   ```

   プレースホルダ`{}`を接続ダイアログから取得した接続パラメータで置き換えてください。

   TiDB Serverlessでは、安全な接続が必要です。mysqlclientの`ssl_mode`は`PREFERRED`にデフォルトで設定されているため、`CA_PATH`を手動で指定する必要はありません。空のままにしてください。ただし、特別な理由で`CA_PATH`を手動で指定する必要がある場合は、[TiDB ServerlessへのTLS接続](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)を参照して、異なるオペレーティングシステム用の証明書パスを取得してください。

7. `.env`ファイルを保存してください。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

   接続文字列を取得する詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

5. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{host}'  # e.g. tidb.xxxx.clusters.tidb-cloud.com
   TIDB_PORT='4000'
   TIDB_USER='{user}'  # e.g. root
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   CA_PATH='{your-downloaded-ca-path}'
   ```

   プレースホルダ`{}`を接続ダイアログから取得した接続パラメータで置き換えてください。また、前のステップでダウンロードした証明書パスを`CA_PATH`に設定してください。

6. `.env`ファイルを保存してください。

</div>
<div label="TiDB Self-Hosted">

1. `.env.example`をコピーして`.env`にリネームするために、次のコマンドを実行してください：

   ```shell
   cp .env.example .env
   ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けてください。例の結果は以下の通りです：

   ```dotenv
   TIDB_HOST='{tidb_server_host}'
   TIDB_PORT='4000'
   TIDB_USER='root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   接続パラメーターで`{}`をプレースホルダーに置き換え、`CA_PATH`の行を削除してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`で、パスワードは空です。

3. `.env`ファイルを保存してください。

</div>
</SimpleTab>

### ステップ4：データベースを初期化する {#step-4-initialize-the-database}

プロジェクトのルートディレクトリで、次のコマンドを実行してデータベースを初期化してください：

```shell
python manage.py migrate
```

### ステップ5：サンプルアプリケーションを実行する {#step-5-run-the-sample-application}

1. 開発モードでアプリケーションを実行します：

   ```shell
   python manage.py runserver
   ```

   アプリケーションはデフォルトでポート`8000`で実行されます。別のポートを使用するには、コマンドにポート番号を追加することができます。以下は例です：

   ```shell
   python manage.py runserver 8080
   ```

2. アプリケーションにアクセスするには、ブラウザを開き、`http://localhost:8000/`に移動します。サンプルアプリケーションでは、次のことができます：

   - 新しいプレイヤーを作成する。
   - プレイヤーを一括作成する。
   - すべてのプレイヤーを表示する。
   - プレイヤーを更新する。
   - プレイヤーを削除する。
   - 2人のプレイヤー間で商品を取引する。

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-django-quickstart](https://github.com/tidb-samples/tidb-python-django-quickstart)リポジトリをチェックしてください。

### TiDBに接続する {#connect-to-tidb}

ファイル`sample_project/settings.py`に、次の設定を追加します：

```python
DATABASES = {
    "default": {
        "ENGINE": "django_tidb",
        "HOST": ${tidb_host},
        "PORT": ${tidb_port},
        "USER": ${tidb_user},
        "PASSWORD": ${tidb_password},
        "NAME": ${tidb_db_name},
        "OPTIONS": {
            "charset": "utf8mb4",
        },
    }
}

TIDB_CA_PATH = ${ca_path}
if TIDB_CA_PATH:
    DATABASES["default"]["OPTIONS"]["ssl_mode"] = "VERIFY_IDENTITY"
    DATABASES["default"]["OPTIONS"]["ssl"] = {
        "ca": TIDB_CA_PATH,
    }
```

`${tidb_host}`, `${tidb_port}`, `${tidb_user}`, `${tidb_password}`, `${tidb_db_name}`, および `${ca_path}` を、お使いの TiDB クラスターの実際の値で置き換える必要があります。

### データモデルの定義 {#define-the-data-model}

```python
from django.db import models

class Player(models.Model):
    name = models.CharField(max_length=32, blank=False, null=False)
    coins = models.IntegerField(default=100)
    goods = models.IntegerField(default=1)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

詳細については、[Djangoモデル](https://docs.djangoproject.com/en/dev/topics/db/models/)を参照してください。

### データの挿入 {#insert-data}

```python
# insert a single object
player = Player.objects.create(name="player1", coins=100, goods=1)

# bulk insert multiple objects
Player.objects.bulk_create([
    Player(name="player1", coins=100, goods=1),
    Player(name="player2", coins=200, goods=2),
    Player(name="player3", coins=300, goods=3),
])
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

```python
# get a single object
player = Player.objects.get(name="player1")

# get multiple objects
filtered_players = Player.objects.filter(name="player1")

# get all objects
all_players = Player.objects.all()
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

```python
# update a single object
player = Player.objects.get(name="player1")
player.coins = 200
player.save()

# update multiple objects
Player.objects.filter(coins=100).update(coins=200)
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

```python
# delete a single object
player = Player.objects.get(name="player1")
player.delete()

# delete multiple objects
Player.objects.filter(coins=100).delete()
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ {#next-steps}

- [Djangoのドキュメント](https://www.djangoproject.com/)からDjangoの使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を参照して、TiDBアプリケーション開発のベストプラクティスを学びます。例えば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータの取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通して学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
