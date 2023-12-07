---
title: Connect to TiDB Serverless via Public Endpoint
summary: Learn how to connect to your TiDB Serverless cluster via public endpoint.
---

# TiDB Serverlessへのパブリックエンドポイント経由での接続 {#connect-to-tidb-serverless-via-public-endpoint}

このドキュメントでは、TiDB Serverlessクラスターにパブリックエンドポイント経由で接続する方法について説明します。パブリックエンドポイントを使用すると、ラップトップからSQLクライアントを使用してTiDB Serverlessクラスターに接続できます。

> **ヒント:**
>
> TiDB Dedicatedクラスターに標準接続経由で接続する方法については、[標準接続経由でTiDB Dedicatedに接続する](/tidb-cloud/connect-via-standard-connection.md)を参照してください。

TiDB Serverlessクラスターにパブリックエンドポイント経由で接続するには、以下の手順を実行してください:

1.  [**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスターの名前をクリックして概要ページに移動します。

2.  右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3.  ダイアログで、エンドポイントタイプをデフォルト設定のまま`Public`とし、希望の接続方法とオペレーティングシステムを選択して、対応する接続文字列を取得します。

    > **注意:**
    >
    > -   エンドポイントタイプを`Public`のままにすると、接続が標準TLS接続経由となります。詳細については、[TiDB ServerlessへのTLS接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。
    > -   **エンドポイントタイプ**のドロップダウンリストで**Private**を選択すると、接続がプライベートエンドポイント経由となります。詳細については、[TiDB Serverlessへのプライベートエンドポイント経由での接続](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)を参照してください。

4.  TiDB Serverlessでは、クラスターのために[ブランチ](/tidb-cloud/branch-overview.md)を作成できます。ブランチが作成された後、**ブランチ**のドロップダウンリストからブランチに接続することができます。`main`はクラスター自体を表します。

5.  パスワードをまだ設定していない場合は、**パスワードを生成**をクリックしてランダムなパスワードを生成します。生成されたパスワードは再表示されないため、安全な場所に保存してください。

6.  接続文字列を使用してクラスターに接続します。

    > **注意:**
    >
    > TiDB Serverlessクラスターに接続する際は、ユーザー名にクラスターの接頭辞を含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名の接頭辞](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

## 次のステップ {#what-s-next}

TiDBクラスターに正常に接続した後、[TiDBでSQLステートメントを探索](/basic-sql-operations.md)することができます。
