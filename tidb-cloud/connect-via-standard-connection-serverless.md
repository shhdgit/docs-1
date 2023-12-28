---
title: Connect to TiDB Serverless via Public Endpoint
summary: Learn how to connect to your TiDB Serverless cluster via public endpoint.
---

# パブリックエンドポイントを介してTiDB Serverlessに接続する {#connect-to-tidb-serverless-via-public-endpoint}

このドキュメントでは、パブリックエンドポイントを介してTiDB Serverlessクラスタに接続する方法について説明します。パブリックエンドポイントを使用すると、ラップトップからSQLクライアントを使用してTiDB Serverlessクラスタに接続することができます。

> **ヒント：**
>
> TiDB Dedicatedクラスタにパブリックエンドポイントを介して接続する方法については、[標準接続を介してTiDB Dedicatedに接続する](/tidb-cloud/connect-via-standard-connection.md)を参照してください。

パブリックエンドポイントを介してTiDB Serverlessクラスタに接続するには、次の手順を実行します。

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. ダイアログで、エンドポイントタイプのデフォルト設定を`Public`に保ち、接続方法とオペレーティングシステムを選択して、対応する接続文字列を取得します。

   > **注意：**
   >
   > - エンドポイントタイプを`Public`に保つと、接続は標準TLS接続を介して行われます。詳細については、[TiDB ServerlessへのTLS接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。
   > - **エンドポイントタイプ**のドロップダウンリストで**Private**を選択すると、接続はプライベートエンドポイントを介して行われます。詳細については、[プライベートエンドポイントを介してTiDB Serverlessに接続する](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)を参照してください。

4. TiDB Serverlessでは、クラスタに[ブランチ](/tidb-cloud/branch-overview.md)を作成することができます。ブランチが作成されると、**ブランチ**ドロップダウンリストからブランチに接続することができます。`main`はクラスタ自体を表します。

5. パスワードをまだ設定していない場合は、**パスワードを生成**をクリックしてランダムなパスワードを生成します。生成されたパスワードは再表示されないため、安全な場所にパスワードを保存してください。

6. 接続文字列を使用してクラスタに接続します。

   > **注意：**
   >
   > TiDB Serverlessクラスタに接続する際は、ユーザー名にクラスタの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

## 次のステップ {#what-s-next}

TiDBクラスタに正常に接続した後、[TiDBでSQLステートメントを探索する](/basic-sql-operations.md)ことができます。
