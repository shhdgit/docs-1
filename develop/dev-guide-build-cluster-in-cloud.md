---
title: Build a TiDB Serverless Cluster
summary: Learn how to build a TiDB Serverless cluster in TiDB Cloud and connect to it.
---

# TiDB Serverlessクラスタを構築する {#build-a-tidb-serverless-cluster}

<CustomContent platform="tidb">

このドキュメントでは、TiDBを最速で始める方法を説明します。[TiDB Cloud](https://en.pingcap.com/tidb-cloud)を使用して、TiDB Serverlessクラスタを作成し、接続し、サンプルアプリケーションを実行します。

ローカルマシンでTiDBを実行する必要がある場合は、[TiDBをローカルで起動する](/quick-start-with-tidb.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

このドキュメントでは、TiDB Cloudを最速で始める方法を説明します。TiDBクラスタを作成し、接続し、サンプルアプリケーションを実行します。

</CustomContent>

## ステップ1. TiDB Serverlessクラスタを作成する {#step-1-create-a-tidb-serverless-cluster}

1. TiDB Cloudアカウントをお持ちでない場合は、[ここをクリック](https://tidbcloud.com/free-trial)してアカウントを登録してください。

2. [ログイン](https://tidbcloud.com/)してTiDB Cloudアカウントにアクセスします。

3. [**クラスタ**](https://tidbcloud.com/console/clusters)ページで、**クラスタを作成**をクリックします。

4. **クラスタを作成**ページで、**Serverless**がデフォルトで選択されています。必要に応じてデフォルトのクラスタ名を更新し、クラスタを作成するリージョンを選択します。

5. TiDB Serverlessクラスタを作成するには、**作成**をクリックします。

   TiDB Cloudクラスタは約30秒で作成されます。

6. TiDB Cloudクラスタが作成されたら、クラスタの概要ページに移動し、右上隅の**接続**をクリックします。接続ダイアログボックスが表示されます。

7. ダイアログで、接続方法とオペレーティングシステムを選択して、対応する接続文字列を取得します。このドキュメントでは、MySQLクライアントを例に使用します。

8. **パスワードを生成**をクリックして、ランダムなパスワードを生成します。生成されたパスワードは再表示されないため、安全な場所にパスワードを保存してください。ルートパスワードを設定しない場合、クラスタに接続できません。

<CustomContent platform="tidb">

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタの場合、クラスタに接続する際には、ユーザー名にクラスタの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](https://docs.pingcap.com/tidbcloud/select-cluster-tier#user-name-prefix)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタの場合、クラスタに接続する際には、ユーザー名にクラスタの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

</CustomContent>

## ステップ2. クラスタに接続する {#step-2-connect-to-a-cluster}

1. MySQLクライアントがインストールされていない場合は、オペレーティングシステムを選択し、以下の手順に従ってインストールします。

<SimpleTab>

<div label="macOS">

macOSの場合、[Homebrew](https://brew.sh/index)をインストールしていない場合は、次のコマンドを実行してMySQLクライアントをインストールします：

```shell
brew install mysql-client
```

The outputは以下の通りです：

    mysql-client is keg-only, which means it was not symlinked into /opt/homebrew,
    because it conflicts with mysql (which contains client libraries).

    If you need to have mysql-client first in your PATH, run:
      echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc

    For compilers to find mysql-client you may need to set:
      export LDFLAGS="-L/opt/homebrew/opt/mysql-client/lib"
      export CPPFLAGS="-I/opt/homebrew/opt/mysql-client/include"

上記の出力で、MySQLクライアントをPATHに追加するには、上記の出力と一致しない場合は、出力の対応するコマンドを使用して、次のコマンドを実行します。

```shell
echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc
```

次に、`source`コマンドを使用してグローバル環境変数を宣言し、MySQLクライアントが正常にインストールされたことを確認します。

```shell
source ~/.zshrc
mysql --version
```

// 以下は、入力内容の例です：

**Note:** この記事では、TiDB Serverless クラスタをセットアップする方法について説明します。

**Note:** この記事では、TiDB Serverless クラスタをセットアップする方法について説明します。

    mysql  Ver 8.0.28 for macos12.0 on arm64 (Homebrew)

</div>

</div>

<div label="Linux">

Linuxの場合、次のようにCentOS 7を例にしてください：

```shell
yum install mysql
```

その後、MySQLクライアントが正常にインストールされたことを確認してください。

```shell
mysql --version
```

// 以下は、入力内容の例です：

**Note:** この記事では、TiDB Serverless クラスタを作成し、TiDB Cloud でセキュリティを確保する方法について説明します。

**Note:** この記事では、TiDB Serverless クラスタを作成し、TiDB Cloud でセキュリティを確保する方法について説明します。

    mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1

</div>

</SimpleTab>

2. [ステップ1](#ステップ1-tidb-serverless-クラスタを作成する)で取得した接続文字列を実行します。

   ```shell
   mysql --connect-timeout 15 -u '<prefix>.root' -h <host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p
   ```

<CustomContent platform="tidb">

> **Note:**
>
> - TiDB Serverlessクラスタに接続する場合は、[TLS接続を使用する必要があります](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)。
> - TiDB Serverlessクラスタに接続する際に問題が発生した場合は、詳細については[Secure Connections to TiDB Serverless Clusters](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> - TiDB Serverlessクラスタに接続する場合は、[TLS接続を使用する必要があります](/tidb-cloud/secure-connections-to-serverless-clusters.md)。
> - TiDB Serverlessクラスタに接続する際に問題が発生した場合は、詳細については[Secure Connections to TiDB Serverless Clusters](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。

</CustomContent>

3. パスワードを入力してサインインします。

## ステップ3. SQLステートメントを実行する {#step-3-execute-a-sql-statement}

TiDB Cloudで最初のSQLステートメントを実行してみましょう。

// The input content end

```sql
SELECT 'Hello TiDB Cloud!';
```

ここには翻訳してはいけないK-Vマップがあります。コンテンツのキーを翻訳せず、値で置き換える必要があります：
{"TiDB Serverless":"TiDB Serverless","Cluster":"クラスタ","Y":"Y","TiDB Cloud":"TiDB Cloud","I":"I","**Note:**":"**Note:**","server":"サーバー","Homebrew":"Homebrew","Secure":"セキュリティ"}

// 入力コンテンツの開始

期待される出力：

// 入力コンテンツの終了

```sql
+-------------------+
| Hello TiDB Cloud! |
+-------------------+
| Hello TiDB Cloud! |
+-------------------+
```

TiDB Cloudを使用してSQLステートメントを正常に実行できた場合、おめでとうございます。
