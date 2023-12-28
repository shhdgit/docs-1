---
title: Connect to TiDB Serverless with WordPress
summary: Learn how to use TiDB Serverless to run WordPress. This tutorial gives step-by-step guidance to run WordPress + TiDB Serverless in a few minutes.
---

# WordPressを使用してTiDB Serverlessに接続する {#connect-to-tidb-serverless-with-wordpress}

TiDBはMySQL互換のデータベースであり、TiDB Serverlessは完全に管理されたTiDBのオファリングです。[WordPress](https://github.com/WordPress)は、ユーザーがウェブサイトを作成および管理できるフリーでオープンソースのコンテンツ管理システム（CMS）です。WordPressはPHPで書かれ、MySQLデータベースを使用しています。

このチュートリアルでは、TiDB Serverlessを使用して無料でWordPressを実行する方法を学ぶことができます。

> **Note:**
>
> TiDB Serverlessに加えて、このチュートリアルはTiDB DedicatedおよびTiDB Self-Hostedクラスタでも動作します。ただし、コスト効率の観点から、WordPressをTiDB Serverlessで実行することを強くお勧めします。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- TiDB Serverlessクラスタ。[TiDB Serverlessクラスタを作成する](/develop/dev-guide-build-cluster-in-cloud.md)に従って、自分のTiDB Cloudクラスタを作成します。

## TiDB ServerlessでWordPressを実行する {#run-wordpress-with-tidb-serverless}

このセクションでは、TiDB ServerlessでWordPressを実行する方法を示します。

### ステップ1：WordPressサンプルリポジトリをクローンする {#step-1-clone-the-wordpress-sample-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/Icemap/wordpress-tidb-docker.git
cd wordpress-tidb-docker
```

### ステップ2：依存関係をインストールする {#step-2-install-dependencies}

1. サンプルリポジトリは、WordPressを起動するために[Docker](https://www.docker.com/)と[Docker Compose](https://docs.docker.com/compose/)が必要です。これらがインストールされている場合は、このステップをスキップできます。WordPressをLinux環境（Ubuntuなど）で実行することを強くお勧めします。次のコマンドを実行して、これらをインストールします。

   ```shell
   sudo sh install.sh
   ```

2. サンプルリポジトリには、[TiDB互換プラグイン](https://github.com/pingcap/wordpress-tidb-plugin)がサブモジュールとして含まれています。次のコマンドを実行して、サブモジュールを更新します。

   ```shell
   git submodule update --init --recursive
   ```

### ステップ3：接続情報を設定する {#step-3-configure-connection-information}

WordPressデータベース接続をTiDB Serverlessに設定します。

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が操作環境に一致することを確認します。

   - **エンドポイントタイプ**が`Public`に設定されていること。
   - **接続先**が`WordPress`に設定されていること。
   - **オペレーティングシステム**が`Debian/Ubuntu/Arch`に設定されていること。

4. ランダムなパスワードを作成するには、**パスワードを生成**をクリックします。

   > **Tip:**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**パスワードをリセット**をクリックして新しいパスワードを生成できます。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{HOST}'  # e.g. gateway01.ap-northeast-1.prod.aws.tidbcloud.com
   TIDB_PORT='4000'
   TIDB_USER='{USERNAME}'  # e.g. xxxxxx.root
   TIDB_PASSWORD='{PASSWORD}'
   TIDB_DB_NAME='test'
   ```

   プレースホルダ`{}`を接続ダイアログから取得した接続パラメータで置き換えてください。デフォルトでは、TiDB Serverlessには`test`データベースが付属しています。TiDB Serverlessクラスタで既に別のデータベースを作成している場合は、`test`をデータベース名に置き換えることができます。

7. `.env`ファイルを保存します。

### ステップ4：TiDB ServerlessでWordPressを起動する {#step-4-start-wordpress-with-tidb-serverless}

1. 次のコマンドを実行して、WordPressをDockerコンテナとして実行します。

   ```shell
   docker compose up -d
   ```

2. WordPressを設定して、[localhost](http://localhost/)を訪問します（コンテナをローカルマシンで起動した場合）またはリモートマシンでWordPressを実行している場合は`http://<your_instance_ip>`を訪問します。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](/tidb-cloud/tidb-cloud-support.md)してください。
