---
title: Connect to TiDB Serverless with WordPress
summary: Learn how to use TiDB Serverless to run WordPress. This tutorial gives step-by-step guidance to run WordPress + TiDB Serverless in a few minutes.
---

# WordPressでTiDB Serverlessに接続する {#connect-to-tidb-serverless-with-wordpress}

TiDBはMySQL互換のデータベースであり、TiDB Serverlessは完全に管理されたTiDBオファリングです。[WordPress](https://github.com/WordPress)は、ユーザーがウェブサイトを作成および管理できる無料のオープンソースのコンテンツ管理システム（CMS）です。WordPressはPHPで書かれており、MySQLデータベースを使用しています。

このチュートリアルでは、TiDB Serverlessを使用して無料でWordPressを実行する方法を学ぶことができます。

> **Note:**
>
> TiDB Serverlessに加えて、このチュートリアルはTiDB DedicatedおよびTiDB Self-Hostedクラスターでも機能します。ただし、コスト効率のためにWordPressをTiDB Serverlessで実行することを強くお勧めします。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- TiDB Serverlessクラスター。[TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、自分自身のTiDB Cloudクラスターを作成していない場合は、作成してください。

## TiDB ServerlessでWordPressを実行する {#run-wordpress-with-tidb-serverless}

このセクションでは、TiDB ServerlessでWordPressを実行する方法を示します。

### ステップ1：WordPressのサンプルリポジトリをクローンする {#step-1-clone-the-wordpress-sample-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/Icemap/wordpress-tidb-docker.git
cd wordpress-tidb-docker
```

### ステップ2：依存関係をインストールする {#step-2-install-dependencies}

1. サンプルリポジトリは、WordPressを起動するために[Docker](https://www.docker.com/)および[Docker Compose](https://docs.docker.com/compose/)が必要です。これらがインストールされている場合は、このステップをスキップできます。WordPressをLinux環境（Ubuntuなど）で実行することを強くお勧めします。次のコマンドを実行して、これらをインストールします。

   ```shell
   sudo sh install.sh
   ```

2. サンプルリポジトリには、[TiDB互換プラグイン](https://github.com/pingcap/wordpress-tidb-plugin)がサブモジュールとして含まれています。次のコマンドを実行して、サブモジュールを更新します。

   ```shell
   git submodule update --init --recursive
   ```

### ステップ3：接続情報を構成する {#step-3-configure-connection-information}

WordPressのデータベース接続をTiDB Serverlessに構成します。

1. [**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスターの名前をクリックして、その概要ページに移動します。

2. 右上隅にある**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致することを確認します。

   - **エンドポイントタイプ**が`Public`に設定されています。
   - **Connect With**が`WordPress`に設定されています。
   - **Operating System**が`Debian/Ubuntu/Arch`に設定されています。

4. ランダムなパスワードを作成するには、**Generate Password**をクリックします。

   > **Tip:**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成できます。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`に名前を変更します。

   ```shell
   cp .env.example .env
   ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例の結果は次のとおりです。

   ```dotenv
   TIDB_HOST='{HOST}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
   TIDB_PORT='4000'
   TIDB_USER='{USERNAME}'  # 例: xxxxxx.root
   TIDB_PASSWORD='{PASSWORD}'
   TIDB_DB_NAME='test'
   ```

   プレースホルダ `{}`を接続ダイアログから取得した接続パラメータで置き換えてください。デフォルトでは、TiDB Serverlessには`test`データベースが付属しています。TiDB Serverlessクラスターで別のデータベースを既に作成している場合は、`test`をデータベース名に置き換えることができます。

7. `.env`ファイルを保存します。

### ステップ4：TiDB ServerlessでWordPressを起動する {#step-4-start-wordpress-with-tidb-serverless}

1. 次のコマンドを実行して、WordPressをDockerコンテナーとして実行します。

   ```shell
   docker compose up -d
   ```

2. WordPressを設定して、WordPressサイトを設定します。ローカルマシンでコンテナーを起動した場合は[localhost](http://localhost/)を訪れ、WordPressがリモートマシンで実行されている場合は`http://<your_instance_ip>`を訪れます。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](/tidb-cloud/tidb-cloud-support.md)してください。"
