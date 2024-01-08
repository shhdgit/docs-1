---
title: TiDB Cloud Release Notes in 2023
summary: Learn about the release notes of TiDB Cloud in 2023.
---

# 2023年のTiDB Cloudリリースノート {#tidb-cloud-release-notes-in-2023}

このページでは、[TiDB Cloud](https://www.pingcap.com/tidb-cloud/)の2023年のリリースノートをリストアップしています。

## 2023年12月5日 {#december-5-2023}

**一般的な変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)は、失敗したchangefeedを再開することができ、新しいchangefeedを作成する手間を省くことができます。

  詳細については、[Changefeed states](/tidb-cloud/changefeed-overview.md#changefeed-states)を参照してください。

**コンソールの変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)の接続体験を向上させます。

  **Connect**ダイアログインターフェースを改善し、TiDB Serverlessユーザーによりスムーズで効率的な接続体験を提供します。さらに、TiDB Serverlessはより多くのクライアントタイプを導入し、接続のための希望するブランチを選択できるようになりました。

  詳細については、[Connect to TiDB Serverless](/tidb-cloud/connect-via-standard-connection-serverless.md)を参照してください。

## 2023年11月28日 {#november-28-2023}

**一般的な変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)は、バックアップからSQLバインディングを復元することをサポートします。

  TiDB Dedicatedは、バックアップから復元する際に、ユーザーアカウントとSQLバインディングをデフォルトで復元します。この強化機能は、v6.2.0以降のバージョンのクラスタで利用可能であり、データの復元プロセスを効率化します。SQLバインディングの復元により、クエリ関連の構成と最適化のスムーズな再統合が保証され、より包括的で効率的なリカバリ体験が提供されます。

  詳細については、[Back up and restore TiDB Dedicated data](/tidb-cloud/backup-and-restore.md)を参照してください。

**コンソールの変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)は、SQLステートメントのRUコストの監視をサポートします。

  TiDB Serverlessは、各SQLステートメントの[Request Units (RUs)](/tidb-cloud/tidb-cloud-glossary.md#request-unit)に関する詳細な情報を提供します。各SQLステートメントの**Total RU**および**Mean RU**のコストを表示できます。この機能により、RUコストを特定し分析することができ、運用上の潜在的なコスト削減の機会を提供します。

  SQLステートメントのRUの詳細を確認するには、[TiDB Serverlessクラスタ](https://tidbcloud.com/console/clusters)の**Diagnosis**ページに移動し、**SQL Statement**タブをクリックしてください。

## 2023年11月21日 {#november-21-2023}

**一般的な変更**

- [Data Migration](/tidb-cloud/migrate-from-mysql-using-data-migration.md)は、Google Cloudに展開されたTiDBクラスタに対して高速な物理モードをサポートします。

  これにより、AWSおよびGoogle Cloudに展開されたTiDBクラスタで物理モードを使用できるようになりました。物理モードの移行速度は最大110 MiB/sに達し、論理モードよりも2.4倍速くなります。この向上したパフォーマンスは、大規模なデータセットを迅速にTiDB Cloudに移行するのに適しています。

  詳細については、[Migrate existing data and incremental data](/tidb-cloud/migrate-from-mysql-using-data-migration.md#migrate-existing-data-and-incremental-data)を参照してください。

## 2023年11月14日 {#november-14-2023}

**一般的な変更**

- TiDB Dedicatedクラスタからデータを復元する際、デフォルトの動作がユーザーアカウントを含むすべてのユーザーアカウントを復元するように変更されました。

  詳細については、[BackUp and Restore TiDB Dedicated Data](/tidb-cloud/backup-and-restore.md)を参照してください。

- changefeedのイベントフィルターを導入します。

  この強化機能により、changefeedのイベントフィルターを簡単に[ティDB Cloudコンソール](https://tidbcloud.com/)を介して直接管理できるようになり、特定のイベントをchangefeedから除外し、データレプリケーションをより効率的に制御できるようになります。

  詳細については、[Changefeed](/tidb-cloud/changefeed-overview.md#edit-a-changefeed)を参照してください。

## 2023年11月7日 {#november-7-2023}

**一般的な変更**

- 次のリソース使用量アラートを追加しました。新しいアラートはデフォルトで無効になっています。必要に応じて有効にできます。

  - TiDBノード全体の最大メモリ使用率が10分間で70%を超えました
  - TiKVノード全体の最大メモリ使用率が10分間で70%を超えました
  - TiDBノード全体の最大CPU使用率が10分間で80%を超えました
  - TiKVノード全体の最大CPU使用率が10分間で80%を超えました

  詳細については、[TiDB Cloud Built-in Alerting](/tidb-cloud/monitor-built-in-alerting.md#resource-usage-alerts)を参照してください。

## 2023年10月31日 {#october-31-2023}

**一般的な変更**

- エンタープライズサポートプランに直接アップグレードすることをTiDB Cloudコンソールでサポートします。営業に連絡する必要はありません。

  詳細については、[TiDB Cloud Support](/tidb-cloud/tidb-cloud-support.md)を参照してください。

## 2023年10月25日 {#october-25-2023}

**一般的な変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)は、Google Cloudでのデュアルリージョンバックアップ（ベータ版）をサポートします。

  Google CloudでホストされているTiDB Dedicatedクラスタは、Google Cloud Storageとシームレスに連携します。Google Cloud Storageの[デュアルリージョン](https://cloud.google.com/storage/docs/locations#location-dr)機能と同様に、TiDB Dedicatedのデュアルリージョンで使用するリージョンのペアは同じマルチリージョン内にある必要があります。たとえば、東京と大阪は同じマルチリージョン`ASIA`にあるため、デュアルリージョンストレージに共に使用できます。

  詳細については、[Back Up and Restore TiDB Dedicated Data](/tidb-cloud/backup-and-restore.md#turn-on-dual-region-backup)を参照してください。

- [Apache Kafkaにデータ変更ログをストリーミング](/tidb-cloud/changefeed-sink-to-apache-kafka.md)する機能が一般提供（GA）になりました。

  10か月のベータトライアルの成功後、TiDB CloudからApache Kafkaへのデータ変更ログのストリーミング機能が一般提供になりました。TiDBからメッセージキューへのデータストリーミングは、データ統合シナリオで一般的なニーズです。Kafkaシンクを使用して、他のデータ処理システム（例：Snowflake）と統合したり、ビジネスの消費をサポートしたりできます。

  詳細については、[Changefeed overview](/tidb-cloud/changefeed-overview.md)を参照してください。

## 2023年10月11日 {#october-11-2023}

**一般的な変更**

- AWSに展開された[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタで[デュアルリージョンバックアップ（ベータ版）](/tidb-cloud/backup-and-restore.md#turn-on-dual-region-backup)をサポートします。

  クラウドプロバイダ内で地理的なリージョン間でバックアップをレプリケートできるようになりました。この機能により、データ保護とディザスタリカバリの機能が追加されます。

  詳細については、[Back up and restore TiDB Dedicated data](/tidb-cloud/backup-and-restore.md)を参照してください。

- データ移行は、既存のデータの物理モードと論理モードの両方をサポートします。

  物理モードでは、移行速度が最大110 MiB/sに達します。論理モードの45 MiB/sと比較して、移行パフォーマンスが大幅に向上しました。

  詳細については、[Migrate existing data and incremental data](/tidb-cloud/migrate-from-mysql-using-data-migration.md#migrate-existing-data-and-incremental-data)を参照してください。

## 2023年10月10日 {#october-10-2023}

**一般的な変更**

- [Vercel Preview Deployments](https://vercel.com/docs/deployments/preview-deployments)でTiDB Cloud Vercel統合を使用して、TiDB Serverlessブランチをサポートします。

  詳細については、[Connect with TiDB Serverless branching](/tidb-cloud/integrate-tidbcloud-with-vercel.md#connect-with-tidb-serverless-branching)を参照してください。

## 2023年9月28日 {#september-28-2023}

**APIの変更**

- 特定の組織の指定された月の請求書を取得するためのTiDB Cloud Billing APIエンドポイントを導入します。

  このBilling APIエンドポイントは、TiDB Cloud API v1beta1でリリースされており、TiDB Cloudの最新のAPIバージョンです。詳細については、[API documentation (v1beta1)](https://docs.pingcap.com/tidbcloud/api/v1beta1#tag/Billing)を参照してください。

## 2023年9月19日 {#september-19-2023}

**一般的な変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタから2つのvCPU TiDBおよびTiKVノードを削除します。

  2つのvCPUオプションは、**Create Cluster**ページまたは**Modify Cluster**ページで利用できなくなりました。

- [JavaScript用のTiDB Cloudサーバーレスドライバー（ベータ版）](/tidb-cloud/serverless-driver.md)をリリースします。

  JavaScript用のTiDB Cloudサーバーレスドライバーを使用すると、HTTPS経由で[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタに接続できます。これは、TCP接続が制限されているエッジ環境（例：[Vercel Edge Function](https://vercel.com/docs/functions/edge-functions)および[Cloudflare Workers](https://workers.cloudflare.com/)）で特に有用です。

  詳細については、[TiDB Cloud serverless driver (beta)](/tidb-cloud/serverless-driver.md)を参照してください。

**コンソールの変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタでは、**Usage This Month**パネルでのコストの見積もりを取得したり、支出限度額を設定する際に見積もりを取得したりできます。"

## 2023年9月5日 {#september-5-2023}

**一般的な変更**

- [データサービス（ベータ版）](https://tidbcloud.com/console/data-service)は、異なる状況での特定のレート制限要件を満たすために、各APIキーのレート制限をカスタマイズすることをサポートしています。

  [APIキーを作成](/tidb-cloud/data-service-api-key.md#create-an-api-key)または[編集](/tidb-cloud/data-service-api-key.md#edit-an-api-key)する際に、APIキーのレート制限を調整できます。

  詳細については、[レート制限](/tidb-cloud/data-service-api-key.md#rate-limiting)を参照してください。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターに新しいAWSリージョンをサポート: São Paulo (sa-east-1)。

- 各[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターに最大100個のIPアドレスをIPアクセスリストに追加することをサポートしています。

  詳細については、[IPアクセスリストの構成](/tidb-cloud/configure-ip-access-list.md)を参照してください。

**コンソールの変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターの**イベント**ページを導入し、クラスターの主な変更の記録を提供します。

  このページでは、過去7日間のイベント履歴を表示し、トリガー時刻やアクションを開始したユーザーなどの重要な詳細を追跡できます。

  詳細については、[TiDB Cloudクラスターのイベント](/tidb-cloud/tidb-cloud-events.md)を参照してください。

**APIの変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターの[APIエンドポイント](https://aws.amazon.com/privatelink/?privatelink-blogs.sort-by=item.additionalFields.createdDate\&privatelink-blogs.sort-order=desc)または[Google Cloud Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect)を管理するためのいくつかのTiDB Cloud APIエンドポイントをリリースしました:

  - クラスターのプライベートエンドポイントサービスを作成する
  - クラスターのプライベートエンドポイントサービス情報を取得する
  - クラスターのプライベートエンドポイントを作成する
  - クラスターのすべてのプライベートエンドポイントをリストする
  - プロジェクト内のすべてのプライベートエンドポイントをリストする
  - クラスターのプライベートエンドポイントを削除する

  詳細については、[APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster)を参照してください。

## 2023年8月23日 {#august-23-2023}

**一般的な変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターでGoogle Cloud [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect)をサポートしています。

  これにより、Google CloudでホストされているTiDB Dedicatedクラスターにプライベートエンドポイントを作成し、安全な接続を確立できます。

  主な利点:

  - 直感的な操作: 数ステップでプライベートエンドポイントを作成できます。
  - 強化されたセキュリティ: データを保護するための安全な接続を確立します。
  - 性能の向上: 低レイテンシーと高帯域幅の接続を提供します。

  詳細については、[Google Cloudでプライベートエンドポイントを介して接続](/tidb-cloud/set-up-private-endpoint-connections-on-google-cloud.md)を参照してください。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターから[Google Cloud Storage（GCS）](https://cloud.google.com/storage)にデータをストリーム配信するためのチェンジフィードの使用をサポートしています。

  これにより、自分のアカウントのバケットを使用し、厳密に適合した権限を提供することで、TiDB CloudからGCSにデータを複製し、データの変更を自由に分析できます。

  詳細については、[Cloud Storageへのシンク](/tidb-cloud/changefeed-sink-to-cloud-storage.md)を参照してください。

## 2023年8月15日 {#august-15-2023}

**一般的な変更**

- [データサービス（ベータ版）](https://tidbcloud.com/console/data-service)は、`GET`リクエストのページネーションをサポートして、開発体験を向上させます。

  `GET`リクエストでは、**Advance Properties**で**Pagination**を有効にし、エンドポイントを呼び出す際にクエリパラメータとして`page`と`page_size`を指定することで、結果をページ分割できます。たとえば、10個のアイテムを含む2番目のページを取得するには、次のコマンドを使用できます:

  ```bash
  curl --digest --user '<Public Key>:<Private Key>' \
    --request GET 'https://<region>.data.tidbcloud.com/api/v1beta/app/<App ID>/endpoint/<Endpoint Path>?page=2&page_size=10'
  ```

  この機能は、最後のクエリが`SELECT`ステートメントである`GET`リクエストにのみ適用されます。

  詳細については、[エンドポイントの呼び出し](/tidb-cloud/data-service-manage-endpoint.md#call-an-endpoint)を参照してください。

- [データサービス（ベータ版）](https://tidbcloud.com/console/data-service)は、`GET`リクエストのエンドポイント応答を指定された有効期間（TTL）のキャッシュにサポートしています。

  この機能により、データベースの負荷を減らし、エンドポイントのレイテンシーを最適化できます。

  `GET`リクエストメソッドを使用するエンドポイントでは、**Advance Properties**で**Cache Response**を有効にし、キャッシュのTTL期間を構成できます。

  詳細については、[Advanced properties](/tidb-cloud/data-service-manage-endpoint.md#advanced-properties)を参照してください。

- 2023年8月15日以降にAWSでホストされ、作成された[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターのロードバランシングの改善を無効にします:

  - AWSでホストされているTiDBノードをスケールアウトする際に、既存の接続を新しいTiDBノードに自動的に移行する機能を無効にします。
  - AWSでホストされているTiDBノードをスケールインする際に、既存の接続を利用可能なTiDBノードに自動的に移行する機能を無効にします。

  この変更により、ハイブリッド展開のリソース競合を回避し、この改善が有効になっている既存のクラスターには影響しません。新しいクラスターでこのロードバランシングの改善を有効にしたい場合は、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

## 2023年8月8日 {#august-8-2023}

**一般的な変更**

- [データサービス（ベータ版）](https://tidbcloud.com/console/data-service)は、Basic認証をサポートしています。

  リクエストで公開鍵をユーザー名として、秘密鍵をパスワードとして提供できます。['Basic' HTTP認証](https://datatracker.ietf.org/doc/html/rfc7617)を使用することで、Digest認証と比較して、より簡単にデータサービスエンドポイントを呼び出すことができます。

  詳細については、[エンドポイントの呼び出し](/tidb-cloud/data-service-manage-endpoint.md#call-an-endpoint)を参照してください。

## 2023年8月1日 {#august-1-2023}

**一般的な変更**

- TiDB Cloud [データサービス](https://tidbcloud.com/console/data-service)のデータアプリに対するOpenAPI仕様をサポートしています。

  TiDB Cloudデータサービスは、各データアプリの自動生成されたOpenAPIドキュメントを提供します。このドキュメントでは、エンドポイント、パラメータ、および応答を表示し、エンドポイントを試すことができます。

  データアプリとその展開されたエンドポイントのOpenAPI仕様（OAS）をYAMLまたはJSON形式でダウンロードできます。OASは標準化されたAPIドキュメント、簡素化された統合、および簡単なコード生成を提供し、より迅速な開発と改善されたコラボレーションを可能にします。

  詳細については、[OpenAPI仕様の使用](/tidb-cloud/data-service-manage-data-app.md#use-the-openapi-specification)および[Next.jsでのOpenAPI仕様の使用](/tidb-cloud/data-service-oas-with-nextjs.md)を参照してください。

- [Postman](https://www.postman.com/)でのデータアプリの実行をサポートしています。

  Postmanの統合により、データアプリのエンドポイントをワークスペースにインポートし、PostmanのWebアプリとデスクトップアプリの両方をサポートすることで、強化されたコラボレーションとシームレスなAPIテストを利用できます。

  詳細については、[Postmanでデータアプリを実行](/tidb-cloud/data-service-postman-integration.md)を参照してください。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターの新しい**一時停止**ステータスを導入し、この期間中に料金を請求しないコスト効果のある一時停止を可能にします。

  TiDB Dedicatedクラスターの**一時停止**をクリックすると、クラスターはまず**一時停止**ステータスに入ります。一時停止操作が完了すると、クラスターステータスが**一時停止**に変わります。

  クラスターは**一時停止**ステータスに変わった後にのみ再開でき、**一時停止**と**再開**のクリックが急速に行われることによる異常な再開の問題が解決されます。

  詳細については、[TiDB Dedicatedクラスターの一時停止または再開](/tidb-cloud/pause-or-resume-tidb-cluster.md)を参照してください。

## 2023年7月26日 {#july-26-2023}

**一般的な変更**

- [Data Service](https://tidbcloud.com/console/data-service)でTiDB Cloudに強力な機能を導入しました：自動エンドポイント生成。

  開発者は今、最小限のクリックと設定でHTTPエンドポイントを簡単に作成できます。繰り返しの雛形コードを排除し、エンドポイントの作成を簡素化し、加速し、潜在的なエラーを減らすことができます。

  この機能の使用方法についての詳細は、[エンドポイントを自動的に生成する](/tidb-cloud/data-service-manage-endpoint.md#generate-an-endpoint-automatically)を参照してください。

- [Data Service](https://tidbcloud.com/console/data-service)でエンドポイントの`PUT`および`DELETE`リクエストメソッドをサポートしました。

  - `PUT`メソッドを使用してデータを更新または変更します。これは`UPDATE`ステートメントと類似しています。
  - `DELETE`メソッドを使用してデータを削除します。これは`DELETE`ステートメントと類似しています。

  詳細については、[プロパティを構成する](/tidb-cloud/data-service-manage-endpoint.md#configure-properties)を参照してください。

- [Data Service](https://tidbcloud.com/console/data-service)で`POST`、`PUT`、および`DELETE`リクエストメソッドの**バッチ操作**をサポートしました。

  エンドポイントで**バッチ操作**が有効になっていると、1つのリクエストで複数の行に操作を実行できます。たとえば、1つの`POST`リクエストで複数のデータ行を挿入できます。

  詳細については、[高度なプロパティ](/tidb-cloud/data-service-manage-endpoint.md#advanced-properties)を参照してください。

## 2023年7月25日 {#july-25-2023}

**一般的な変更**

- 新しい[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタのデフォルトのTiDBバージョンを[v6.5.3](https://docs.pingcap.com/tidb/v6.5/release-6.5.3)から[v7.1.1](https://docs.pingcap.com/tidb/v7.1/release-7.1.1)にアップグレードしました。

**コンソールの変更**

- TiDB CloudユーザーのPingCAPサポートへのアクセスを最適化することで、サポートエントリを簡素化しました。改善点は以下の通りです：

  - <MDSvgIcon name="icon-top-organization" />の**サポート**に入口を追加しました。
  - [TiDB Cloudコンソール](https://tidbcloud.com/)の右下隅の\*\*?\*\*アイコンのメニューを直感的にするように改良しました。

  詳細については、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)を参照してください。

## 2023年7月18日 {#july-18-2023}

**一般的な変更**

- 組織レベルとプロジェクトレベルの役割ベースのアクセス制御を改良し、ユーザに最小限の権限で役割を付与できるようにしました。これにより、セキュリティ、コンプライアンス、生産性が向上します。

  - 組織の役割には`組織所有者`、`組織請求管理者`、`組織コンソール監査管理者`、`組織メンバー`が含まれます。
  - プロジェクトの役割には`プロジェクト所有者`、`プロジェクトデータアクセス読み書き`、`プロジェクトデータアクセス読み取り専用`が含まれます。
  - プロジェクト内のクラスタ（クラスタの作成、変更、削除など）を管理するには、`組織所有者`または`プロジェクト所有者`の役割である必要があります。

  異なる役割の権限についての詳細については、[ユーザの役割](/tidb-cloud/manage-user-access.md#user-roles)を参照してください。

- AWS上でホストされている[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタ向けに、顧客管理型暗号化キー（CMEK）機能（ベータ版）をサポートしました。

  AWS KMSに基づいたCMEKを作成して、TiDB CloudコンソールからEBSとS3に保存されているデータを暗号化できます。これにより、顧客が管理するキーでデータを暗号化することができ、セキュリティが向上します。

  この機能にはまだ制限があり、リクエストによってのみ利用可能です。この機能を申し込むには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

- TiDB Cloudのインポート機能を最適化し、データのインポート体験を向上させました。以下の改善点があります：

  - TiDB Serverlessの統一されたインポートエントリ：ローカルファイルのインポートとAmazon S3からのファイルのインポートのエントリを統合し、シームレスに切り替えることができます。
  - ストリームラインされた構成：Amazon S3からのデータのインポートは、1つのステップのみで行うことができるようになり、時間と労力を節約できます。
  - 強化されたCSV構成：CSV構成設定は、ファイルタイプオプションの下にあり、必要なパラメータを素早く構成できるようになりました。
  - 強化されたターゲットテーブル選択：チェックボックスをクリックしてデータのインポートのための目標テーブルを選択できるようになりました。この改善により、複雑な式が不要になり、ターゲットテーブルの選択が簡素化されます。
  - 精緻な表示情報：インポートプロセス中に不正確な情報が表示される問題を解決しました。また、不完全なデータ表示や誤解を避けるために、プレビュー機能を削除しました。
  - ソースファイルのマッピングの改善：ソースファイル名を変更して特定の命名要件を満たすためのマッピング関係を定義することができるようになりました。

## 2023年7月11日 {#july-11-2023}

**一般的な変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)が一般提供されるようになりました。

- ベータ版のTiDB Botを導入しました。これはOpenAIパワードのチャットボットで、多言語サポート、24時間365日のリアルタイム応答、統合されたドキュメントアクセスを提供します。

  TiDB Botには以下の利点があります：

  - 継続的なサポート：強化されたサポート体験のために常に利用可能です。
  - 効率の向上：自動応答によりレイテンシーが低減し、全体的な運用が向上します。
  - シームレスなドキュメントアクセス：簡単な情報取得と迅速な問題解決のためにTiDB Cloudドキュメントに直接アクセスできます。

  TiDB Botを使用するには、[TiDB Cloudコンソール](https://tidbcloud.com)の右下隅の\*\*?\*\*をクリックし、**Ask TiDB Bot**を選択してチャットを開始します。

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタ向けに、ベータ版のブランチ機能をサポートしました。

  TiDB Cloudでは、TiDB Serverlessクラスタのためのブランチを作成できます。クラスタのブランチは、元のクラスタから分岐したデータの複製を含む別個のインスタンスです。これにより、影響を気にすることなく接続して自由に実験できる独立した環境を提供します。

  2023年7月5日以降に作成されたTiDB Serverlessクラスタについては、[TiDB Cloudコンソール](/tidb-cloud/branch-manage.md)または[TiDB Cloud CLI](/tidb-cloud/ticloud-branch-create.md)を使用してブランチを作成できます。

  アプリケーション開発にGitHubを使用している場合、TiDB ServerlessブランチングをGitHub CI/CDパイプラインに統合することで、本番データベースに影響を与えることなくプルリクエストを自動的にテストできます。詳細については、[GitHubとTiDB Serverless Branching（ベータ版）の統合](/tidb-cloud/branch-github-integration.md)を参照してください。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタ向けに週次バックアップをサポートしました。詳細については、[TiDB Dedicatedデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md#turn-on-auto-backup)を参照してください。

## 2023年7月4日 {#july-4-2023}

**一般的な変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターのポイントインタイムリカバリ（PITR）（ベータ版）をサポートします。

  これにより、過去90日間の任意の時点に TiDB Serverless クラスターを復元できるようになります。この機能により、TiDB Serverless クラスターのデータリカバリ能力が向上します。たとえば、データ書き込みエラーが発生し、データを以前の状態に戻したい場合に PITR を使用できます。

  詳細については、[TiDB Serverless データのバックアップとリストア](/tidb-cloud/backup-and-restore-serverless.md#restore)を参照してください。

**コンソールの変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターのクラスター概要ページの **今月の使用状況** パネルを強化し、現在のリソース使用状況をより明確に表示します。

- 次の変更を行うことで、全体的なナビゲーション体験を向上させます。

  - 右上隅の <MDSvgIcon name="icon-top-organization" /> **組織** と <MDSvgIcon name="icon-top-account-settings" /> **アカウント** を左側のナビゲーションバーに統合します。
  - 左側のナビゲーションバーの <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke-width="1.5" xmlns="http://www.w3.org/2000/svg"><path d="M12 14.5H7.5C6.10444 14.5 5.40665 14.5 4.83886 14.6722C3.56045 15.06 2.56004 16.0605 2.17224 17.3389C2 17.9067 2 18.6044 2 20M14.5 6.5C14.5 8.98528 12.4853 11 10 11C7.51472 11 5.5 8.98528 5.5 6.5C5.5 4.01472 7.51472 2 10 2C12.4853 2 14.5 4.01472 14.5 6.5ZM22 16.516C22 18.7478 19.6576 20.3711 18.8054 20.8878C18.7085 20.9465 18.6601 20.9759 18.5917 20.9911C18.5387 21.003 18.4613 21.003 18.4083 20.9911C18.3399 20.9759 18.2915 20.9465 18.1946 20.8878C17.3424 20.3711 15 18.7478 15 16.516V14.3415C15 13.978 15 13.7962 15.0572 13.6399C15.1077 13.5019 15.1899 13.3788 15.2965 13.2811C15.4172 13.1706 15.5809 13.1068 15.9084 12.9791L18.2542 12C18.3452 11.9646 18.4374 11.8 18.4374 11.8H18.5626C18.5626 11.8 18.6548 11.9646 18.7458 12L21.0916 12.9791C21.4191 13.1068 21.5828 13.1706 21.7035 13.2811C21.8101 13.3788 21.8923 13.5019 21.9428 13.6399C22 13.7962 22 13.978 22 14.3415V16.516Z" stroke="currentColor" stroke-width="inherit" stroke-linecap="round" stroke-linejoin="round"></path></svg> **管理** を左側のナビゲーションバーに統合し、左上隅の ☰ ホバーメニューを削除します。これで、<MDSvgIcon name="icon-left-projects" /> をクリックしてプロジェクト間を切り替えたり、プロジェクト設定を変更したりできるようになります。
  - TiDB Cloud のすべてのヘルプとサポート情報を、ドキュメント、インタラクティブなチュートリアル、自習トレーニング、サポートエントリなどを含む、右下隅の **?** アイコンのメニューに統合します。

- TiDB Cloud コンソールは、より快適で目に優しい体験を提供するダークモードをサポートします。左側のナビゲーションバーの下部から、ライトモードとダークモードを切り替えることができます。

## 2023年6月27日 {#june-27-2023}

**一般的な変更**

- 新しく作成された [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターの事前構築済みサンプルデータセットを削除します。

## 2023年6月20日 {#june-20-2023}

**一般的な変更**

- 新しい [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのデフォルトの TiDB バージョンを [v6.5.2](https://docs.pingcap.com/tidb/v6.5/release-6.5.2) から [v6.5.3](https://docs.pingcap.com/tidb/v6.5/release-6.5.3) にアップグレードします。

## 2023年6月13日 {#june-13-2023}

**一般的な変更**

- Amazon S3 にデータをストリームするためのチェンジフィードの使用をサポートします。

  これにより、TiDB Cloud と Amazon S3 のシームレスな統合が可能になります。これにより、[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターから Amazon S3 にリアルタイムのデータキャプチャとレプリケーションが可能になり、下流のアプリケーションや分析が最新のデータにアクセスできるようになります。

  詳細については、[クラウドストレージへのシンク](/tidb-cloud/changefeed-sink-to-cloud-storage.md)を参照してください。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの 16 vCPU TiKV の最大ノードストレージを 4 TiB から 6 TiB に増加します。

  この強化により、TiDB Dedicated クラスターのデータストレージ容量が増加し、ワークロードのスケーリング効率が向上し、データ要件の増加に対応します。

  詳細については、[クラスターのサイズ設定](/tidb-cloud/size-your-cluster.md)を参照してください。

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターの [監視メトリクスの保持期間](/tidb-cloud/built-in-monitoring.md#metrics-retention-policy) を 3日から7日に延長します。

  メトリクスの保持期間を延長することで、より多くの過去のデータにアクセスできるようになります。これにより、より良い意思決定と迅速なトラブルシューティングのために、クラスターのトレンドやパターンを特定できるようになります。

**コンソールの変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの [**Key Visualizer**](/tidb-cloud/tune-performance.md#key-visualizer) ページの新しいネイティブウェブインフラストラクチャをリリースします。

  新しいインフラストラクチャにより、**Key Visualizer** ページを簡単にナビゲートし、より直感的かつ効率的に必要な情報にアクセスできるようになります。新しいインフラストラクチャにより、UXの多くの問題が解決され、SQL診断プロセスがユーザーフレンドリーになります。

## 2023年6月6日 {#june-6-2023}

**一般的な変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタ向けの [Index Insight (beta)](/tidb-cloud/index-insight.md) を導入し、これにより遅いクエリに対するインデックスの推奨事項を提供することでクエリのパフォーマンスを最適化します。

  Index Insight を使用すると、次のような方法で全体的なアプリケーションのパフォーマンスとデータベース操作の効率を向上させることができます。

  - 強化されたクエリのパフォーマンス: Index Insight は遅いクエリを特定し、それに適したインデックスを提案することで、クエリの実行を高速化し、応答時間を短縮し、ユーザーエクスペリエンスを向上させます。
  - コスト効率: Index Insight を使用してクエリのパフォーマンスを最適化することで、余分な計算リソースの必要性が低減され、既存のインフラを効果的に活用することができます。これにより、運用コストの節約につながる可能性があります。
  - 簡素化された最適化プロセス: Index Insight はインデックスの改善を特定し実装するプロセスを簡素化し、手動の分析や推測の必要性を排除します。その結果、正確なインデックスの推奨事項により時間と労力を節約することができます。
  - アプリケーションの効率向上: Index Insight を使用してデータベースのパフォーマンスを最適化することで、TiDB Cloud 上で実行されるアプリケーションはより大きなワークロードを処理し、より多くのユーザーに同時にサービスを提供することができます。これにより、アプリケーションのスケーリング操作がより効率的になります。

  Index Insight を使用するには、TiDB Dedicated クラスタの **Diagnosis** ページに移動し、**Index Insight BETA** タブをクリックします。

  詳細については、[Use Index Insight (beta)](/tidb-cloud/index-insight.md) を参照してください。

- [TiDB Playground](https://play.tidbcloud.com/?utm_source=docs\&utm_medium=tidb_cloud_release_notes) を導入し、登録やインストールなしで TiDB の全機能を体験できるインタラクティブなプラットフォームです。

  TiDB Playground は、スケーラビリティ、MySQL 互換性、リアルタイムアナリティクスなど、TiDB の機能を探索するためのワンストップショップ体験を提供するインタラクティブなプラットフォームです。

  TiDB Playground を使用すると、リアルタイムで複雑な構成から解放された制御された環境で TiDB の機能を試すことができます。これにより、TiDB の機能を理解するのに最適です。

  TiDB Playground を始めるには、[**TiDB Playground**](https://play.tidbcloud.com/?utm_source=docs\&utm_medium=tidb_cloud_release_notes) ページにアクセスし、探索したい機能を選択して探索を開始します。

## 2023年6月5日 {#june-5-2023}

**一般的な変更**

- [Data App](/tidb-cloud/tidb-cloud-glossary.md#data-app) を GitHub に接続するサポートを追加しました。

  [Data App を GitHub に接続](/tidb-cloud/data-service-manage-github-connection.md) することで、Data App のすべての構成を GitHub 上のコードファイルとして管理できます。これにより、TiDB Cloud Data Service がシステムアーキテクチャと DevOps プロセスにシームレスに統合されます。

  この機能により、次のタスクを簡単に実行できるようになり、Data App の開発の CI/CD エクスペリエンスが向上します。

  - GitHub で Data App の変更を自動的にデプロイする。
  - GitHub 上で Data App の変更の CI/CD パイプラインを構成する。
  - 接続された GitHub リポジトリから切断する。
  - デプロイ前にエンドポイントの変更を確認する。
  - デプロイ履歴を表示し、失敗した場合に必要なアクションを実行する。
  - コミットを再デプロイして以前のデプロイに戻す。

  詳細については、[Deploy Data App automatically with GitHub](/tidb-cloud/data-service-manage-github-connection.md) を参照してください。

## 2023年6月2日 {#june-2-2023}

**一般的な変更**

- 製品名を簡素化し明確にするために、製品名を更新しました。

  - "TiDB Cloud Serverless Tier" は "TiDB Serverless" となりました。
  - "TiDB Cloud Dedicated Tier" は "TiDB Dedicated" となりました。
  - "TiDB On-Premises" は "TiDB Self-Hosted" となりました。

  これらのリフレッシュされた名前のもとで同じ素晴らしいパフォーマンスをお楽しみください。お客様のエクスペリエンスが私たちの優先事項です。

## 2023年5月30日 {#may-30-2023}

**一般的な変更**

- TiDB Cloud のデータ移行機能の増分データ移行のサポートを強化しました。

  これにより、指定された位置以降に生成された増分データのみを TiDB Cloud にレプリケートするために、binlog ポジションまたはグローバルトランザクション識別子（GTID）を指定できるようになります。この強化により、特定の要件に合わせて必要なデータを選択してレプリケートする柔軟性が向上します。

  詳細については、[Migrate Only Incremental Data from MySQL-Compatible Databases to TiDB Cloud Using Data Migration](/tidb-cloud/migrate-incremental-data-from-mysql-using-data-migration.md) を参照してください。

- [**Events**](/tidb-cloud/tidb-cloud-events.md) ページに新しいイベントタイプ（`ImportData`）を追加しました。

- TiDB Cloud コンソールから **Playground** を削除しました。

  最適化されたエクスペリエンスを提供する新しいスタンドアロンの Playground にご期待ください。

## 2023年5月23日 {#may-23-2023}

**一般的な変更**

- TiDB に CSV ファイルをアップロードする際、英字と数字だけでなく、中国語や日本語などの文字を使用して列名を定義することができます。ただし、特殊文字についてはアンダースコア（`_`）のみがサポートされます。

  詳細については、[Import Local Files to TiDB Cloud](/tidb-cloud/tidb-cloud-import-local-files.md) を参照してください。

## 2023年5月16日 {#may-16-2023}

**コンソールの変更**

- Dedicated および Serverless ティア向けに機能カテゴリによって整理された左側のナビゲーションエントリを導入しました。

  新しいナビゲーションにより、機能エントリを発見することがより簡単で直感的になります。新しいナビゲーションを表示するには、クラスタの概要ページにアクセスしてください。

- Dedicated ティアクラスタの **Diagnosis** ページにおいて、以下の2つのタブについて新しいネイティブウェブインフラストラクチャをリリースしました。

  - [Slow Query](/tidb-cloud/tune-performance.md#slow-query)
  - [SQL Statement](/tidb-cloud/tune-performance.md#statement-analysis)

  新しいインフラストラクチャにより、2つのタブを簡単に移動し、必要な情報により直感的かつ効率的にアクセスできるようになります。新しいインフラストラクチャはユーザーエクスペリエンスを向上させ、SQL 診断プロセスをよりユーザーフレンドリーにします。

## 2023年5月9日 {#may-9-2023}

**一般的な変更**

- 2023年4月26日以降に作成された GCP ホストのクラスタのノードサイズを変更するサポートを追加しました。

  この機能により、需要の増加に対応するためにパフォーマンスの高いノードにアップグレードしたり、コストを節約するためにパフォーマンスの低いノードにダウングレードしたりすることができます。この柔軟性により、クラスタの容量をワークロードに合わせて調整し、コストを最適化することができます。

  詳細な手順については、[Change node size](/tidb-cloud/scale-tidb-cluster.md#change-vcpu-and-ram) を参照してください。

- 圧縮ファイルのインポートをサポートしました。`.gzip`、`.gz`、`.zstd`、`.zst`、`.snappy` の形式の CSV ファイルおよび SQL ファイルをインポートできます。この機能により、データのインポートが効率的かつコスト効果的に行えるようになり、データ転送コストが削減されます。

  詳細については、[Import CSV Files from Amazon S3 or GCS into TiDB Cloud](/tidb-cloud/import-csv-files.md) および [Import Sample Data](/tidb-cloud/import-sample-data.md) を参照してください。

- TiDB Cloud [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタ向けに、AWS PrivateLink パワードのエンドポイント接続を新しいネットワークアクセス管理オプションとしてサポートしました。

  プライベートエンドポイント接続により、データを公共インターネットに公開することなく、CIDR オーバーラップをサポートし、ネットワーク管理が容易になります。

  詳細については、[Set Up Private Endpoint Connections](/tidb-cloud/set-up-private-endpoint-connections.md) を参照してください。

**コンソールの変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタ向けに、バックアップ、リストア、および changefeed アクションを記録するための新しいイベントタイプを [**Event**](/tidb-cloud/tidb-cloud-events.md) ページに追加しました。

  記録できるイベントの完全なリストについては、[Logged events](/tidb-cloud/tidb-cloud-events.md#logged-events) を参照してください。

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタ向けに、[**SQL Diagnosis**](/tidb-cloud/tune-performance.md) ページに **SQL Statement** タブを導入しました。

  **SQL Statement** タブには、次の機能があります。

  - TiDB データベースで実行されたすべての SQL ステートメントの包括的な概要が表示され、遅いクエリを簡単に特定および診断できます。
  - クエリ時間、実行計画、データベースサーバーの応答など、各 SQL ステートメントの詳細情報が表示され、データベースのパフォーマンスを最適化できます。
  - 大量のデータをソート、フィルタ、検索することが容易なユーザーフレンドリーなインターフェースが提供され、最も重要なクエリに焦点を当てることができます。

  詳細については、[Statement Analysis](/tidb-cloud/tune-performance.md#statement-analysis) を参照してください。

## 2023年5月6日 {#may-6-2023}

**一般的な変更**

- [Data Service endpoint](/tidb-cloud/tidb-cloud-glossary.md#endpoint) に直接アクセスすることをサポートし、TiDB [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタが配置されている地域のエンドポイントにアクセスできます。

  新しく作成されたServerless Tierクラスタでは、エンドポイントURLにクラスタの地域情報が含まれます。 `<region>.data.tidbcloud.com` をリクエストすることで、TiDBクラスタが配置されている地域のエンドポイントに直接アクセスできます。

  また、地域を指定せずにグローバルドメイン `data.tidbcloud.com` をリクエストすることもできます。この方法では、TiDB Cloudはリクエストを対象の地域に内部的にリダイレクトしますが、これにより追加のレイテンシーが発生する可能性があります。この方法を選択する場合は、curlコマンドを呼び出す際に `--location-trusted` オプションを追加してください。

  詳細については、[エンドポイントを呼び出す](/tidb-cloud/data-service-manage-endpoint.md#call-an-endpoint) を参照してください。

## 2023年4月25日 {#april-25-2023}

**一般的な変更**

- 組織内の最初の5つの[Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタについて、以下のように各クラスタに無料の使用クォータが提供されます。

  - Row storage: 5 GiB
  - [Request Units (RUs)](/tidb-cloud/tidb-cloud-glossary.md#request-unit): 1か月あたり5000万RUs

  2023年5月31日まで、Serverless Tierクラスタは無料であり、100%の割引が適用されます。その後、無料クォータを超える使用料金が発生します。

  クラスタの **Overview** ページの **Usage This Month** エリアで、クラスタの使用状況を簡単に[モニタリングしたり、使用クォータを増やしたり](/tidb-cloud/manage-serverless-spend-limit.md#manage-spending-limit-for-tidb-serverless-clusters)することができます。クラスタの無料クォータに達すると、このクラスタでの読み取りおよび書き込み操作が制限され、クォータを増やすか、新しい月の開始時に使用量がリセットされるまで、操作が実行されません。

  異なるリソース（読み取り、書き込み、SQL CPU、ネットワーク出力など）のRU消費、価格の詳細、および制限情報については、[TiDB Cloud Serverless Tier Pricing Details](https://www.pingcap.com/tidb-cloud-serverless-pricing-details) を参照してください。

- TiDB Cloud [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタのバックアップとリストアをサポートします。

  詳細については、[Back up and Restore TiDB Cluster Data](/tidb-cloud/backup-and-restore-serverless.md) を参照してください。

- 新しい[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタのデフォルトのTiDBバージョンを、[v6.5.1](https://docs.pingcap.com/tidb/v6.5/release-6.5.1) から [v6.5.2](https://docs.pingcap.com/tidb/v6.5/release-6.5.2) にアップグレードします。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタの計画されたメンテナンスアクティビティを簡単にスケジュールして管理するためのメンテナンスウィンドウ機能を提供します。

  メンテナンスウィンドウは、オペレーティングシステムの更新、セキュリティパッチ、およびインフラストラクチャのアップグレードなどの計画されたメンテナンスアクティビティが自動的に実行される指定された時間枠です。これにより、TiDB Cloudサービスの信頼性、セキュリティ、およびパフォーマンスが確保されます。

  メンテナンスウィンドウ中に一時的な接続の中断やQPSの変動が発生する可能性がありますが、クラスタは利用可能であり、SQL操作、既存のデータのインポート、バックアップ、リストア、移行、およびレプリケーションタスクは通常通り実行されます。メンテナンス中に[許可および不許可の操作のリスト](/tidb-cloud/configure-maintenance-window.md#allowed-and-disallowed-operations-during-a-maintenance-window)を参照してください。

  メンテナンスの頻度を最小限に抑えるよう努めます。メンテナンスウィンドウが計画されている場合、デフォルトの開始時刻は、ターゲット週の水曜日の03:00です（TiDB Cloud組織のタイムゾーンに基づく）。

  - TiDB Cloudは、メンテナンスウィンドウごとに3つの電子メール通知を送信します：メンテナンス前、メンテナンス開始、メンテナンスタスク完了後の通知。
  - メンテナンスの影響を最小限に抑えるために、**Maintenance** ページでメンテナンスの開始時刻を好みの時間に変更したり、メンテナンスアクティビティを延期したりすることができます。

  詳細については、[Configure maintenance window](/tidb-cloud/configure-maintenance-window.md) を参照してください。

- TiDBの負荷分散を改善し、2023年4月25日以降に作成されたAWS上の[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタのTiDBノードをスケールアウトする際の接続の切断を減らします。

  - TiDBノードをスケールアウトする際に、既存の接続を新しいTiDBノードに自動的に移行することをサポートします。
  - TiDBノードをスケールインする際に、既存の接続を利用可能なTiDBノードに自動的に移行することをサポートします。

  現在、この機能はAWS上にホストされているすべてのDedicated Tierクラスタに提供されています。

**コンソールの変更**

- [Monitoring](/tidb-cloud/built-in-monitoring.md#view-the-metrics-page) ページの[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタの新しいネイティブWebインフラストラクチャをリリースします。

  新しいインフラストラクチャでは、[Monitoring](/tidb-cloud/built-in-monitoring.md#view-the-metrics-page) ページを簡単にナビゲートし、より直感的かつ効率的な方法で必要な情報にアクセスできます。新しいインフラストラクチャは、UXの多くの問題を解決し、モニタリングプロセスをユーザーフレンドリーにします。

## 2023年4月18日 {#april-18-2023}

**一般的な変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタの[Data Migration job specifications](/tidb-cloud/tidb-cloud-billing-dm.md#specifications-for-data-migration) のスケーリングアップまたはダウンをサポートします。

  この機能により、仕様をスケーリングアップすることで移行のパフォーマンスを向上させたり、仕様をスケーリングダウンすることでコストを削減したりすることができます。

  詳細については、[Migrate MySQL-Compatible Databases to TiDB Cloud Using Data Migration](/tidb-cloud/migrate-from-mysql-using-data-migration.md#scale-a-migration-job-specification) を参照してください。

**コンソールの変更**

- [クラスタの作成](https://tidbcloud.com/console/clusters/create-cluster) のエクスペリエンスをよりユーザーフレンドリーにするためにUIを刷新し、わずか数回のクリックでクラスタを作成および構成できるようにします。

  新しいデザインは、シンプルさに焦点を当て、視覚的な混乱を減らし、明確な指示を提供します。クラスタ作成ページで **Create** をクリックした後、クラスタ作成が完了するのを待たずに、クラスタの概要ページに移動します。

  詳細については、[Create a cluster](/tidb-cloud/create-tidb-cluster.md) を参照してください。

- **Billing** ページに **Discounts** タブを導入し、組織の所有者および請求管理者向けの割引情報を表示します。

  詳細については、[Discounts](/tidb-cloud/tidb-cloud-billing.md#discounts) を参照してください。

## 2023年4月11日 {#april-11-2023}

**一般的な変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの TiDB ノードをスケールする際の負荷分散を改善し、AWS でホストされている [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの TiDB ノードをスケールする際の接続ドロップを減らしました。

  - TiDB ノードをスケールアウトする際に既存の接続を新しい TiDB ノードに自動的に移行するサポートを追加しました。
  - TiDB ノードをスケールインする際に既存の接続を利用可能な TiDB ノードに自動的に移行するサポートを追加しました。

  現在、この機能は AWS の `Oregon (us-west-2)` リージョンでホストされている Dedicated Tier クラスターにのみ提供されています。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの [New Relic](https://newrelic.com/) 統合をサポートしました。

  New Relic 統合により、TiDB クラスターのメトリックデータを [New Relic](https://newrelic.com/) に送信するように TiDB Cloud を構成できます。その後、[New Relic](https://newrelic.com/) でアプリケーションのパフォーマンスと TiDB データベースのパフォーマンスの両方を監視および分析できます。この機能により、潜在的な問題を迅速に特定し、解決時間を短縮することができます。

  統合手順と利用可能なメトリックについては、[New Relic との TiDB Cloud の統合](/tidb-cloud/monitor-new-relic-integration.md) を参照してください。

- Dedicated Tier クラスターの Prometheus 統合に [changefeed](/tidb-cloud/changefeed-overview.md) メトリックを追加しました。

  - `tidbcloud_changefeed_latency`
  - `tidbcloud_changefeed_replica_rows`

  [TiDB Cloud を Prometheus と統合](/tidb-cloud/monitor-prometheus-and-grafana-integration.md) している場合、これらのメトリックを使用して changefeed のパフォーマンスと健康状態をリアルタイムで監視できます。さらに、Prometheus を使用してメトリックを監視するためのアラートを簡単に作成できます。

**コンソールの変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの [Monitoring](/tidb-cloud/built-in-monitoring.md#view-the-metrics-page) ページを更新し、[node-level resource metrics](/tidb-cloud/built-in-monitoring.md#server) を使用するようにしました。

  ノードレベルのリソースメトリックを使用すると、購入したサービスの実際の使用状況をより正確に把握できます。

  これらのメトリックにアクセスするには、クラスターの [Monitoring](/tidb-cloud/built-in-monitoring.md#view-the-metrics-page) ページに移動し、**Metrics** タブの下の **Server** カテゴリを確認してください。

- [Billing](/tidb-cloud/tidb-cloud-billing.md#billing-details) ページを最適化し、**Summary by Project** と **Summary by Service** に請求情報を再編成し、請求情報をより明確にしました。

## 2023年4月4日 {#april-4-2023}

**一般的な変更**

- [TiDB Cloud built-in alerts](/tidb-cloud/monitor-built-in-alerting.md#tidb-cloud-built-in-alert-conditions) から以下の 2 つのアラートを削除し、誤検知を防ぎました。これは、クラスターの一時的なオフラインやメモリ不足 (OOM) の問題が全体的なクラスターの健全性にほとんど影響を与えないためです。

  - クラスター内の少なくとも 1 つの TiDB ノードのメモリが不足しています。
  - 1 つ以上のクラスターノードがオフラインです。

**コンソールの変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスター用の [Alerts](/tidb-cloud/monitor-built-in-alerting.md) ページを導入し、各 Dedicated Tier クラスターのアクティブおよびクローズドなアラートを一覧表示します。

  **Alerts** ページには以下が提供されます：

  - 直感的で使いやすいユーザーインターフェース。このページでクラスターのアラートを表示できますが、アラート通知メールに登録していなくても大丈夫です。
  - 重要度、ステータス、およびその他の属性に基づいてアラートを迅速に検索およびソートするための高度なフィルターオプション。また、過去 7 日間の履歴データを表示できるため、アラートの履歴追跡が容易になります。
  - **Edit Rule** 機能。アラートルール設定をカスタマイズしてクラスター固有のニーズに合わせることができます。

  詳細については、[TiDB Cloud built-in alerts](/tidb-cloud/monitor-built-in-alerting.md) を参照してください。

- TiDB Cloud 関連のヘルプ情報とアクションを1か所にまとめました。

  これで、[TiDB Cloud help information](/tidb-cloud/tidb-cloud-support.md) をすべてまとめて、[TiDB Cloud console](https://tidbcloud.com/) の右下の **?** をクリックすることでサポートに連絡できるようになりました。

- [Getting Started](https://tidbcloud.com/console/getting-started) ページを導入し、TiDB Cloud の学習をサポートしました。

  **Getting Started** ページでは、インタラクティブなチュートリアル、必須ガイド、および有用なリンクが提供されます。インタラクティブなチュートリアルに従うことで、TiDB Cloud の機能と HTAP 機能を簡単に探索し、事前に構築された業界固有のデータセット (Steam Game Dataset および S\&P 500 Dataset) を使用できます。

  **Getting Started** ページにアクセスするには、[TiDB Cloud console](https://tidbcloud.com/) の左側のナビゲーションバーで <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 14.9998L9 11.9998M12 14.9998C13.3968 14.4685 14.7369 13.7985 16 12.9998M12 14.9998V19.9998C12 19.9998 15.03 19.4498 16 17.9998C17.08 16.3798 16 12.9998 16 12.9998M9 11.9998C9.53214 10.6192 10.2022 9.29582 11 8.04976C12.1652 6.18675 13.7876 4.65281 15.713 3.59385C17.6384 2.53489 19.8027 1.98613 22 1.99976C22 4.71976 21.22 9.49976 16 12.9998M9 11.9998H4C4 11.9998 4.55 8.96976 6 7.99976C7.62 6.91976 11 7.99976 11 7.99976M4.5 16.4998C3 17.7598 2.5 21.4998 2.5 21.4998C2.5 21.4998 6.24 20.9998 7.5 19.4998C8.21 18.6598 8.2 17.3698 7.41 16.5898C7.02131 16.2188 6.50929 16.0044 5.97223 15.9878C5.43516 15.9712 4.91088 16.1535 4.5 16.4998Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg> **Getting Started** をクリックしてください。このページでは、**Query Sample Dataset** をクリックしてインタラクティブなチュートリアルを開くか、他のリンクをクリックして TiDB Cloud を探索することができます。または、右下の **?** をクリックして **Interactive Tutorials** をクリックしてください。

## 2023年3月29日 {#march-29-2023}

**一般的な変更**

- [Data Service (beta)](/tidb-cloud/data-service-overview.md) は Data Apps のより細かいアクセス制御をサポートしました。

  Data App の詳細ページで、クラスターを Data App にリンクし、各 API キーの役割を指定できるようになりました。役割は、API キーがリンクされたクラスターに対してデータを読み取るか書き込むかを制御し、`ReadOnly` または `ReadAndWrite` に設定できます。この機能により、Data Apps のためにクラスターレベルと権限レベルのアクセス制御が提供され、ビジネスニーズに応じてアクセス範囲を柔軟に制御できます。

  詳細については、[Manage linked clusters](/tidb-cloud/data-service-manage-data-app.md#manage-linked-data-sources) および [Manage API keys](/tidb-cloud/data-service-api-key.md) を参照してください。

## 2023年3月28日 {#march-28-2023}

**一般的な変更**

- [changefeeds](/tidb-cloud/changefeed-overview.md)の2 RCUs、4 RCUs、および8 RCUsの仕様を追加し、[changefeed](/tidb-cloud/changefeed-overview.md#create-a-changefeed)を作成する際に希望の仕様を選択できるようにサポートします。

  これらの新しい仕様を使用すると、以前に16 RCUsが必要だったシナリオと比較して、データレプリケーションコストを最大87.5%削減できます。

- 2023年3月28日以降に作成された[changefeeds](/tidb-cloud/changefeed-overview.md)の仕様を拡大または縮小するサポートを追加します。

  より高い仕様を選択することでレプリケーションパフォーマンスを向上させるか、より低い仕様を選択することでレプリケーションコストを削減できます。

  詳細については、[changefeedのスケール](/tidb-cloud/changefeed-overview.md#scale-a-changefeed)を参照してください。

- AWSの[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタから、同じプロジェクトおよび同じリージョンの[Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタにリアルタイムで増分データをレプリケートするサポートを追加します。

  詳細については、[TiDB Cloudへのシンク](/tidb-cloud/changefeed-sink-to-tidb-cloud.md)を参照してください。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタの[Data Migration](/tidb-cloud/migrate-from-mysql-using-data-migration.md)機能のための2つの新しいGCPリージョンをサポートします：`Singapore (asia-southeast1)`および`Oregon (us-west1)`。

  これらの新しいリージョンにより、TiDB Cloudへのデータの移行オプションが増えます。上流データがこれらのリージョンに保存されている場合、GCPからTiDB Cloudへのより速く信頼性の高いデータ移行を利用できるようになります。

  詳細については、[Data Migrationを使用したMySQL互換データベースのTiDB Cloudへの移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

**コンソールの変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタの[Slow Query](/tidb-cloud/tune-performance.md#slow-query)ページのための新しいネイティブウェブインフラストラクチャをリリースします。

  この新しいインフラストラクチャを使用すると、[Slow Query](/tidb-cloud/tune-performance.md#slow-query)ページを簡単にナビゲートし、より直感的かつ効率的な方法で必要な情報にアクセスできます。新しいインフラストラクチャは、UXに関する多くの問題を解決し、SQL診断プロセスをユーザーフレンドリーにします。

## 2023年3月21日 {#march-21-2023}

**一般的な変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタの[Data Service (beta)](https://tidbcloud.com/console/data-service)を導入し、カスタムAPIエンドポイントを使用してHTTPSリクエストを介してデータにアクセスできるようにします。

  Data Serviceを使用すると、HTTPSに対応した任意のアプリケーションやサービスとTiDB Cloudをシームレスに統合できます。以下は一般的なシナリオです：

  - モバイルアプリケーションやWebアプリケーションからTiDBクラスタのデータに直接アクセスする。
  - サーバーレスエッジ関数を使用してエンドポイントを呼び出し、データベース接続プールによるスケーラビリティの問題を回避する。
  - Data Serviceをデータソースとして使用してデータ可視化プロジェクトにTiDB Cloudを統合する。
  - MySQLインターフェースがサポートしていない環境からデータベースに接続する。

  さらに、TiDB Cloudは、AIを使用してSQLステートメントを生成および実行するRESTfulインターフェースである[Chat2Query API](/tidb-cloud/use-chat2query-api.md)を提供します。

  Data Serviceにアクセスするには、左側のナビゲーションペインの[**Data Service**](https://tidbcloud.com/console/data-service)ページに移動します。詳細については、次のドキュメントを参照してください：

  - [Data Service概要](/tidb-cloud/data-service-overview.md)
  - [Data Serviceのはじめ方](/tidb-cloud/data-service-get-started.md)
  - [Chat2Query APIのはじめ方](/tidb-cloud/use-chat2query-api.md)

- 2022年12月31日以降に作成されたAWS上の[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタのTiDB、TiKV、およびTiFlashノードのサイズを縮小するサポートを追加します。

  [TiDB Cloudコンソール](/tidb-cloud/scale-tidb-cluster.md#change-vcpu-and-ram)または[TiDB Cloud API (beta)](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster)を介してノードのサイズを縮小できます。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタの[Data Migration](/tidb-cloud/migrate-from-mysql-using-data-migration.md)機能のための新しいGCPリージョンをサポートします：`Tokyo (asia-northeast1)`。

  この機能を使用すると、Google Cloud Platform（GCP）のMySQL互換データベースからTiDBクラスタにデータを簡単かつ効率的に移行できます。

  詳細については、[Data Migrationを使用したMySQL互換データベースのTiDB Cloudへの移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

**コンソールの変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタの**Events**ページを導入し、クラスタへの主な変更の記録を提供します。

  このページでは、過去7日間のイベント履歴を表示し、トリガー時刻やアクションを開始したユーザーなどの重要な詳細を追跡できます。たとえば、クラスタが一時停止されたときやクラスタサイズを変更したユーザーなどのイベントを表示できます。

  詳細については、[TiDB Cloudクラスタのイベント](/tidb-cloud/tidb-cloud-events.md)を参照してください。

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタの**Monitoring**ページに**Database Status**タブを追加し、次のデータベースレベルのメトリクスを表示します：

  - データベースごとのQPS
  - データベースごとの平均クエリ時間
  - データベースごとの失敗したクエリ

  これらのメトリクスを使用すると、個々のデータベースのパフォーマンスを監視し、データに基づいた意思決定を行い、アプリケーションのパフォーマンスを改善するためのアクションを実行できます。

  詳細については、[Serverless Tierクラスタの監視メトリクス](/tidb-cloud/built-in-monitoring.md)を参照してください。

## 2023年3月14日 {#march-14-2023}

**一般的な変更**

- 新しい[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタのデフォルトのTiDBバージョンを[v6.5.0](https://docs.pingcap.com/tidb/v6.5/release-6.5.0)から[v6.5.1](https://docs.pingcap.com/tidb/v6.5/release-6.5.1)にアップグレードします。

- ローカルCSVファイルをヘッダ行付きでアップロードし、TiDB Cloudによって作成されるターゲットテーブルの列名を変更するサポートを追加します。

  [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタにローカルCSVファイルをインポートする際、ヘッダ行の列名がTiDB Cloudの列命名規則に従っていない場合、対応する列名の横に警告アイコンが表示されます。警告を解消するには、アイコンにカーソルを移動し、メッセージに従って既存の列名を編集するか新しい列名を入力できます。

  列命名規則についての詳細については、[ローカルファイルのインポート](/tidb-cloud/tidb-cloud-import-local-files.md#import-local-files)を参照してください。

## 2023年3月7日 {#march-7-2023}

**一般的な変更**

- すべての[Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタのデフォルトのTiDBバージョンを[v6.4.0](https://docs.pingcap.com/tidb/v6.4/release-6.4.0)から[v6.6.0](https://docs.pingcap.com/tidb/v6.6/release-6.6.0)にアップグレードします。

## 2023年2月28日 {#february-28-2023}

**一般的な変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタの**SQL Diagnosis**機能を追加します。

  SQL Diagnosisを使用すると、SQLに関連するランタイムステータスに深い洞察を得ることができ、SQLパフォーマンスチューニングがより効率的になります。現在、Serverless TierのSQL Diagnosis機能は遅いクエリデータのみを提供しています。

  SQL Diagnosisを使用するには、Serverless Tierクラスタページの左側のナビゲーションバーで**SQL Diagnosis**をクリックします。

**コンソールの変更**

- 左側のナビゲーションを最適化します。

  たとえば、次のような方法でページを効率的に移動できます：

  - マウスを左上隅に置いて、クラスタやプロジェクトを素早く切り替えることができます。
  - **Clusters**ページと**Admin**ページを切り替えることができます。

**APIの変更**

- データインポートのためのいくつかのTiDB Cloud APIエンドポイントをリリースします：

  - すべてのインポートタスクをリストする
  - インポートタスクを取得する
  - インポートタスクを作成する
  - インポートタスクを更新する
  - インポートタスク用のローカルファイルをアップロードする
  - インポートタスクを開始する前にデータをプレビューする
  - インポートタスクのロール情報を取得する

  詳細については、[APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Import)を参照してください。

## 2023年2月22日 {#february-22-2023}

**一般的な変更**

- [コンソール監査ログ](/tidb-cloud/tidb-cloud-console-auditing.md)機能をサポートし、[TiDB Cloudコンソール](https://tidbcloud.com/)で組織内のメンバーによって実行されたさまざまなアクティビティを追跡する機能をサポートします。

  コンソール監査ログ機能は、`Owner`または`Audit Admin`ロールを持つユーザーにのみ表示され、デフォルトでは無効になっています。有効にするには、[TiDB Cloudコンソール](https://tidbcloud.com/)の右上隅にある<MDSvgIcon name="icon-top-organization" /> **組織** > **コンソール監査ログ**をクリックします。

  コンソール監査ログを分析することで、組織内で実行された不審な操作を特定し、組織のリソースとデータのセキュリティを向上させることができます。

  詳細については、[コンソール監査ログ](/tidb-cloud/tidb-cloud-console-auditing.md)を参照してください。

**CLIの変更**

- [TiDB Cloud CLI](/tidb-cloud/cli-reference.md)に新しいコマンド[`ticloud cluster connect-info`](/tidb-cloud/ticloud-cluster-connect-info.md)を追加します。

  `ticloud cluster connect-info`は、クラスターの接続文字列を取得するコマンドです。このコマンドを使用するには、[`ticloud`](/tidb-cloud/ticloud-update.md)をv0.3.2またはそれ以降のバージョンに更新する必要があります。

## 2023年2月21日 {#february-21-2023}

**一般的な変更**

- TiDB Cloudにデータをインポートする際に、IAMユーザーのAWSアクセスキーを使用してAmazon S3バケットにアクセスする機能をサポートします。

  この方法はRole ARNを使用するよりも簡単です。詳細については、[Amazon S3アクセスの構成](/tidb-cloud/config-s3-and-gcs-access.md#configure-amazon-s3-access)を参照してください。

- [監視メトリクスの保持期間](/tidb-cloud/built-in-monitoring.md#metrics-retention-policy)を2日から長い期間に延長します：

  - Dedicated Tierクラスターの場合、過去7日間のメトリクスデータを表示できます。
  - Serverless Tierクラスターの場合、過去3日間のメトリクスデータを表示できます。

  メトリクスの保持期間を延長することで、より多くの過去データにアクセスできるようになります。これにより、より良い意思決定と迅速なトラブルシューティングのためにクラスターの傾向とパターンを特定できます。

**コンソールの変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターの監視ページに新しいネイティブWebインフラストラクチャをリリースします。

  新しいインフラストラクチャを使用すると、監視ページを簡単にナビゲートし、より直感的かつ効率的な方法で必要な情報にアクセスできます。新しいインフラストラクチャは、UXの多くの問題を解決し、監視プロセスをはるかにユーザーフレンドリーにします。

## 2023年2月17日 {#february-17-2023}

**CLIの変更**

- [TiDB Cloud CLI](/tidb-cloud/cli-reference.md)に新しいコマンド[`ticloud connect`](/tidb-cloud/ticloud-connect.md)を追加します。

  `ticloud connect`は、ローカルマシンからSQLクライアントをインストールせずにTiDB Cloudクラスターに接続するコマンドです。TiDB Cloudクラスターに接続した後、TiDB Cloud CLIでSQLステートメントを実行できます。

## 2023年2月14日 {#february-14-2023}

**一般的な変更**

- TiDB [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターのTiKVおよびTiFlashノードの数を減らしてスケールインする機能をサポートします。

  ノード数を減らすことができます。[TiDB Cloudコンソール](/tidb-cloud/scale-tidb-cluster.md#change-node-number)または[TiDB Cloud API (beta)](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster)を介して行うことができます。

**コンソールの変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターのための**監視**ページを導入します。

  **監視**ページでは、1秒あたりに実行されるSQLステートメントの数、クエリの平均時間、失敗したクエリの数など、さまざまなメトリクスとデータを提供し、Serverless Tierクラスター内のSQLステートメントの全体的なパフォーマンスを理解するのに役立ちます。

  詳細については、[TiDB Cloud組み込み監視](/tidb-cloud/built-in-monitoring.md)を参照してください。

## 2023年2月2日 {#february-2-2023}

**CLIの変更**

- TiDB Cloud CLIクライアント[`ticloud`](/tidb-cloud/cli-reference.md)を導入します。

  `ticloud`を使用すると、数行のコマンドでターミナルやその他の自動ワークフローからTiDB Cloudリソースを簡単に管理できます。特にGitHub Actionsでは、[`setup-tidbcloud-cli`](https://github.com/marketplace/actions/set-up-tidbcloud-cli)を提供しており、`ticloud`を簡単にセットアップできます。

  詳細については、[TiDB Cloud CLIクイックスタート](/tidb-cloud/get-started-with-cli.md)および[TiDB Cloud CLIリファレンス](/tidb-cloud/cli-reference.md)を参照してください。

## 2023年1月18日 {#january-18-2023}

**一般的な変更**

- Microsoftアカウントで[TiDB Cloudにサインアップ](https://tidbcloud.com/free-trial)する機能をサポートします。

## 2023年1月17日 {#january-17-2023}

**一般的な変更**

- 新しい[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターのデフォルトのTiDBバージョンを[v6.1.3](https://docs.pingcap.com/tidb/stable/release-6.1.3)から[v6.5.0](https://docs.pingcap.com/tidb/stable/release-6.5.0)にアップグレードします。

- 新規登録ユーザーに対して、TiDB Cloudは無料の[Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターを自動的に作成し、TiDB Cloudでデータ探索の旅を迅速に開始できるようにします。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターのための新しいAWSリージョン`Seoul (ap-northeast-2)`をサポートします。

  このリージョンでは、次の機能が有効になっています：

  - [データ移行を使用してMySQL互換データベースをTiDB Cloudに移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)
  - [changefeedを使用してTiDB Cloudから他のデータサービスにデータをストリーム配信](/tidb-cloud/changefeed-overview.md)
  - [TiDBクラスターデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md)

## 2023年1月10日 {#january-10-2023}

**一般的な変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターのローカルCSVファイルからのデータインポート機能を最適化し、ユーザーエクスペリエンスを向上させます。

  - CSVファイルをアップロードするには、アップロードエリアに簡単にドラッグアンドドロップすることができます。
  - インポートタスクを作成する際、ターゲットデータベースまたはテーブルが存在しない場合、名前を入力してTiDB Cloudに自動的に作成させることができます。ターゲットテーブルを作成するために、プライマリキーを指定するか、複合プライマリキーを形成するために複数のフィールドを選択できます。
  - インポートが完了したら、[AIパワードChat2Query](/tidb-cloud/explore-data-with-chat2query.md)を使用してデータを探索できます。**Chat2Queryでデータを探索**をクリックするか、タスクリスト内のターゲットテーブル名をクリックします。

  詳細については、[ローカルファイルをTiDB Cloudにインポート](/tidb-cloud/tidb-cloud-import-local-files.md)を参照してください。

**コンソールの変更**

- 各クラスターに**サポートを取得**オプションを追加し、特定のクラスターのサポートをリクエストするプロセスを簡素化します。

  次のいずれかの方法でクラスターのサポートをリクエストできます：

  - プロジェクトの[**クラスター**](https://tidbcloud.com/console/clusters)ページで、クラスターの行の\*\*...\*\*をクリックし、**サポートを取得**を選択します。
  - クラスターの概要ページで、右上隅の\*\*...\*\*をクリックし、**サポートを取得**を選択します。

## 2023年1月5日 {#january-5-2023}

**コンソールの変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターのSQLエディタ（ベータ版）をChat2Query（ベータ版）に名前変更し、AIを使用してSQLクエリを生成する機能をサポートします。

  Chat2Queryでは、AIにSQLクエリを自動生成させるか、SQLクエリを手動で記述し、ターミナルなしでデータベースに対してSQLクエリを実行できます。

  Chat2Queryにアクセスするには、[**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、クラスター名をクリックして、左側のナビゲーションペインで**Chat2Query**をクリックします。

## 2023年1月4日 {#january-4-2023}

**一般的な変更**

- 2022年12月31日以降に作成されたAWS上のTiDB Dedicatedクラスタで、TiDB、TiKV、およびTiFlashノードのサイズ（vCPU + RAM）を増やしてスケーリングするサポートを追加しました。

  [TiDB Cloudコンソール](/tidb-cloud/scale-tidb-cluster.md#change-vcpu-and-ram)または[TiDB Cloud API（ベータ版）](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster)を使用してノードサイズを増やすことができます。

- [**モニタリング**](/tidb-cloud/built-in-monitoring.md)ページでメトリクスの保持期間を2日間に延長しました。

  これにより、過去2日間のメトリクスデータにアクセスできるようになり、クラスタのパフォーマンスとトレンドに対する柔軟性と可視性が向上します。

  この改善には追加費用はかかりません。クラスタの[**モニタリング**](/tidb-cloud/built-in-monitoring.md)ページの**診断**タブからアクセスできます。これにより、パフォーマンスの問題を特定しトラブルシューティングしたり、クラスタ全体の健全性を効果的にモニタリングしたりできます。

- Prometheusの統合のためにGrafanaダッシュボードJSONをカスタマイズするサポートを追加しました。

  [TiDB CloudをPrometheusと統合](/tidb-cloud/monitor-prometheus-and-grafana-integration.md)している場合、事前に作成されたGrafanaダッシュボードをインポートしてTiDB Cloudクラスタをモニタリングし、ダッシュボードをカスタマイズすることができます。この機能により、TiDB Cloudクラスタを簡単かつ迅速にモニタリングし、パフォーマンスの問題を素早く特定することができます。

  詳細については、[メトリクスを視覚化するためのGrafana GUIダッシュボードの使用](/tidb-cloud/monitor-prometheus-and-grafana-integration.md#step-3-use-grafana-gui-dashboards-to-visualize-the-metrics)を参照してください。

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタのデフォルトのTiDBバージョンを[v6.3.0](https://docs.pingcap.com/tidb/v6.3/release-6.3.0)から[v6.4.0](https://docs.pingcap.com/tidb/v6.4/release-6.4.0)にアップグレードしました。Serverless TierクラスタのデフォルトのTiDBバージョンをv6.4.0にアップグレードした後のコールドスタートの問題が解決されました。

**コンソールの変更**

- [**クラスタ**](https://tidbcloud.com/console/clusters)ページとクラスタ概要ページの表示をシンプルにしました。

  - [**クラスタ**](https://tidbcloud.com/console/clusters)ページでクラスタ名をクリックしてクラスタ概要ページに移動し、クラスタの操作を開始できます。
  - クラスタ概要ページから**接続**と**インポート**パネルを削除しました。右上隅の**接続**をクリックして接続情報を取得し、左側のナビゲーションパネルの**インポート**をクリックしてデータをインポートできます。
