---
title: Build a TiDB Serverless Cluster
summary: Learn how to build a TiDB Serverless cluster in TiDB Cloud and connect to it.
---

# TiDBサーバーレスクラスターを構築する {#build-a-tidb-serverless-cluster}

<CustomContent platform="tidb">

このドキュメントでは、TiDBの利用を開始する最も迅速な方法について説明します。[TiDB Cloud](https://en.pingcap.com/tidb-cloud)を使用してTiDBサーバーレスクラスターを作成し、それに接続し、サンプルアプリケーションを実行します。

ローカルマシンでTiDBを実行する必要がある場合は、[TiDBのローカルでの起動](/quick-start-with-tidb.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

このドキュメントでは、TiDB Cloudの利用を開始する最も迅速な方法について説明します。TiDBクラスターを作成し、それに接続し、サンプルアプリケーションを実行します。

</CustomContent>

## ステップ1. TiDBサーバーレスクラスターを作成する {#step-1-create-a-tidb-serverless-cluster}

1.  TiDB Cloudアカウントをお持ちでない場合は、[こちら](https://tidbcloud.com/free-trial)をクリックしてアカウントを作成してください。

2.  TiDB Cloudアカウントに[ログイン](https://tidbcloud.com/)します。

3.  [**クラスター**](https://tidbcloud.com/console/clusters)ページで、**クラスターの作成**をクリックします。

4.  **クラスターの作成**ページでは、**サーバーレス**がデフォルトで選択されています。必要に応じてデフォルトのクラスター名を更新し、その後、クラスターを作成するリージョンを選択します。

5.  TiDBサーバーレスクラスターを作成するには、**作成**をクリックします。

    TiDB Cloudクラスターは約30秒で作成されます。

6.  TiDB Cloudクラスターが作成されたら、クラスター名をクリックしてクラスターの概要ページに移動し、右上隅の**接続**をクリックします。接続ダイアログボックスが表示されます。

7.  ダイアログで、希望の接続方法とオペレーティングシステムを選択して、対応する接続文字列を取得します。このドキュメントでは、MySQLクライアントを例として使用します。

8.  ランダムなパスワードを生成するには、**パスワードを生成**をクリックします。生成されたパスワードは再表示されないため、パスワードを安全な場所に保存してください。rootパスワードを設定しない場合、クラスターに接続できません。

<CustomContent platform="tidb">

> **注意:**
>
> [TiDBサーバーレス](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターの場合、クラスターの接頭辞をユーザー名に含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](https://docs.pingcap.com/tidbcloud/select-cluster-tier#user-name-prefix)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> [TiDBサーバーレス](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターの場合、クラスターに接続する際に、ユーザー名にクラスターの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[/tidb-cloud/select-cluster-tier.md#user-name-prefix](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

</CustomContent>

## ステップ2. クラスターに接続する {#step-2-connect-to-a-cluster}

1.  MySQLクライアントがインストールされていない場合は、オペレーティングシステムを選択し、以下の手順に従ってインストールしてください。

<SimpleTab>

<div label="macOS">

macOSの場合、[Homebrew](https://brew.sh/index)をインストールしていない場合は、以下のコマンドを実行してMySQLクライアントをインストールしてください:

```shell
brew install mysql-client
```

翻訳されたコンテンツは次のとおりです：

出力要件は次のとおりです：

-   元のテキストと比較し、翻訳で情報が失われていないことを確認します。
-   追加の素材やコメントを追加しないでください。
-   書き言葉では、口語を避けてください。
-   元のテキストはMarkdownの一部かもしれませんが、元のテキストの一部を失わないようにしてください。
-   記事の元のMarkdown形式を変更しないでください。

    mysql-client is keg-only, which means it was not symlinked into /opt/homebrew,
    because it conflicts with mysql (which contains client libraries).

    If you need to have mysql-client first in your PATH, run:
    echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> \~/.zshrc

    For compilers to find mysql-client you may need to set:
    export LDFLAGS="-L/opt/homebrew/opt/mysql-client/lib"
    export CPPFLAGS="-I/opt/homebrew/opt/mysql-client/include"

MySQLクライアントをPATHに追加するには、上記の出力で以下のコマンドを見つけて（出力が文書内の上記の出力と一貫していない場合は、代わりに出力の対応するコマンドを使用して）実行してください：

```shell
echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc
```

その後、`source`コマンドでグローバル環境変数を宣言し、MySQLクライアントが正常にインストールされていることを確認してください。

```shell
source ~/.zshrc
mysql --version
```

期待される出力の例：

// translated content start
原文の内容を日本語に翻訳してください。出力の要件は以下の通りです：

-   元のテキストと比較し、翻訳で情報が失われていないことを確認する。
-   追加の素材やコメントを追加しない。
-   書き言葉で、口語を避ける。
-   元のテキストはMarkdownの一部かもしれないので、元のテキストの一部を失わないようにする。
-   記事の元のMarkdown形式を変更しないでください。
    // translated content end

    mysql  Ver 8.0.28 for macos12.0 on arm64 (Homebrew)

</div>

<div label="Linux">

Linuxについて、以下はCentOS 7を例としています。

```shell
yum install mysql
```

その後、MySQLクライアントが正常にインストールされていることを確認してください。

```shell
mysql --version
```

期待される出力の例：
// original content end

原文を日本語に翻訳してください。出力の要件は以下の通りです：

-   元のテキストと比較し、翻訳で情報が失われていないことを確認してください。
-   追加の素材やコメントを追加しないでください。
-   書き言葉で、口語表現を避けてください。
-   元のテキストはMarkdownの一部かもしれませんが、元のテキストの一部を失わないでください。
-   記事の元のMarkdown形式を変更しないでください。

    mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86\_64) using readline 5.1

</div>

</SimpleTab>

2.  [Step 1](#step-1-create-a-tidb-serverless-cluster) で取得した接続文字列を実行します。

    ```shell
    mysql --connect-timeout 15 -u '<prefix>.root' -h <host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p
    ```

<CustomContent platform="tidb">

> **注意:**
>
> -   TiDB Serverless クラスタに接続する場合は、[TLS 接続を使用する必要があります](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)。
> -   TiDB Serverless クラスタに接続する際に問題が発生した場合は、[TiDB Serverless クラスタへの安全な接続](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> -   TiDB Serverless クラスタに接続する場合は、[TLS 接続を使用する必要があります](/tidb-cloud/secure-connections-to-serverless-clusters.md)。
> -   TiDB Serverless クラスタに接続する際に問題が発生した場合は、[TiDB Serverless クラスタへの安全な接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。

</CustomContent>

3.  パスワードを入力してサインインします。

## Step 3. SQL ステートメントを実行する {#step-3-execute-a-sql-statement}

TiDB Cloud 上で最初の SQL ステートメントを実行してみましょう。

```sql
SELECT 'Hello TiDB Cloud!';
```

// translated content start
期待される出力：
// translated content end

```sql
+-------------------+
| Hello TiDB Cloud! |
+-------------------+
| Hello TiDB Cloud! |
+-------------------+
```

もし実際の出力が期待される出力と類似しているなら、おめでとうございます。あなたはTiDB Cloud上でSQLステートメントを成功裏に実行しました。
