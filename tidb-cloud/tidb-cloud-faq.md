---
title: TiDB Cloud FAQs
summary: Learn about the most frequently asked questions (FAQs) relating to TiDB Cloud.
---

# TiDB Cloud FAQs {#tidb-cloud-faqs}

<!-- markdownlint-disable MD026 -->

このドキュメントは、TiDB Cloudに関する最もよくある質問をリストしています。

## 一般的なFAQ {#general-faqs}

### TiDB Cloudとは何ですか？ {#what-is-tidb-cloud}

TiDB Cloudは、直感的なコンソールを介して制御できる完全に管理されたクラウドインスタンスで、TiDBクラスタの展開、管理、および保守をさらに簡単にします。 Amazon Web ServicesまたはGoogle Cloudで簡単に展開し、ミッションクリティカルなアプリケーションを迅速に構築できます。

TiDB Cloudを使用すると、インフラストラクチャの管理やクラスタの展開など、かつては複雑だったタスクを、ほとんどまたはまったくトレーニングを受けていない開発者やDBAが簡単に処理できるため、アプリケーションに集中し、データベースの複雑さには集中しなくても済みます。また、ボタンをクリックするだけでTiDBクラスタをスケーリングインまたはスケーリングアウトすることで、コストのかかるリソースを無駄にすることなく、データベースを必要なだけの容量と期間で提供できます。

### TiDBとTiDB Cloudの関係は何ですか？ {#what-is-the-relationship-between-tidb-and-tidb-cloud}

TiDBはオープンソースのデータベースであり、TiDB Self-Hostedを自社のデータセンター、自己管理クラウド環境、またはその両方で実行したい組織にとって最良の選択肢です。

TiDB Cloudは、TiDBの完全に管理されたクラウドデータベースサービスです。使いやすいWebベースの管理コンソールを備えており、ミッションクリティカルな本番環境のためにTiDBクラスタを管理できます。

### TiDB CloudはMySQLと互換性がありますか？ {#is-tidb-cloud-compatible-with-mysql}

現在、TiDB Cloudはトリガー、ストアドプロシージャ、ユーザー定義関数、および外部キーを除く、MySQL 5.7の構文の大部分をサポートしています。詳細については、[MySQLとの互換性](https://docs.pingcap.com/tidb/stable/mysql-compatibility)を参照してください。

### TiDB Cloudで使用できるプログラミング言語は何ですか？ {#what-programming-languages-can-i-use-to-work-with-tidb-cloud}

MySQLクライアントまたはドライバでサポートされている任意の言語を使用できます。

### TiDB Cloudを実行できる場所はどこですか？ {#where-can-i-run-tidb-cloud}

TiDB Cloudは現在、Amazon Web ServicesとGoogle Cloudで利用できます。

### TiDB Cloudは異なるクラウドサービスプロバイダ間でのVPCピアリングをサポートしていますか？ {#does-tidb-cloud-support-vpc-peering-between-different-cloud-service-providers}

いいえ。

### TiDB CloudでサポートされるTiDBのバージョンは何ですか？ {#what-versions-of-tidb-are-supported-on-tidb-cloud}

- 2024年1月3日から、新しいTiDB DedicatedクラスタのデフォルトのTiDBバージョンはv7.5.0です。
- 2023年3月7日から、新しいTiDB ServerlessクラスタのデフォルトのTiDBバージョンはv6.6.0です。

詳細については、[TiDB Cloudリリースノート](/tidb-cloud/tidb-cloud-release-notes.md)を参照してください。

### どの企業がTiDBまたはTiDB Cloudを本番で使用していますか？ {#what-companies-are-using-tidb-or-tidb-cloud-in-production}

TiDBは、金融サービス、ゲーム、および電子商取引など、さまざまな業界の1500以上のグローバル企業に信頼されています。ユーザーには、Square（米国）、Shopee（シンガポール）、および中国銀聯（中国）などが含まれます。詳細については、[事例研究](https://en.pingcap.com/customers/)を参照してください。

### SLAはどのように見えますか？ {#what-does-the-sla-look-like}

TiDB Cloudは99.99%のSLAを提供します。詳細については、[TiDB Cloudサービスのサービスレベル契約](https://en.pingcap.com/legal/service-level-agreement-for-tidb-cloud-services/)を参照してください。

### TiDB Cloudについてもっと詳しく知りたいですか？ {#how-can-i-learn-more-about-tidb-cloud}

TiDB Cloudについて詳しく知る最良の方法は、ステップバイステップのチュートリアルに従うことです。以下のトピックをチェックして、始めてください。

- [TiDB Cloudの紹介](/tidb-cloud/tidb-cloud-intro.md)
- [はじめに](/tidb-cloud/tidb-cloud-quickstart.md)
- [TiDB Serverlessクラスタの作成](/tidb-cloud/create-tidb-cluster-serverless.md)

### クラスタを削除する際の `XXX's Org/default project/Cluster0` とは何を指しますか？ {#what-does-xxx-s-org-default-project-cluster0-refer-to-when-deleting-a-cluster}

TiDB Cloudでは、クラスタは組織名、プロジェクト名、およびクラスタ名の組み合わせで一意に識別されます。意図したクラスタを削除していることを確認するためには、そのクラスタの完全修飾名を指定する必要があります。例：`XXX's Org/default project/Cluster0`。

## アーキテクチャのFAQ {#architecture-faqs}

### TiDBクラスタには異なるコンポーネントがあります。TiDB、TiKV、TiFlashノードとは何ですか？ {#there-are-different-components-in-my-tidb-cluster-what-are-tidb-tikv-and-tiflash-nodes}

TiDBは、TiKVまたはTiFlashストアから返されたクエリからデータを集約するSQLコンピューティングレイヤーです。 TiDBは水平スケーラブルであり、TiDBノードの数を増やすと、クラスタが処理できる同時クエリの数が増加します。

TiKVは、OLTPデータを格納するために使用されるトランザクショナルストアです。 TiKVのすべてのデータは複数のレプリカ（デフォルトで3つのレプリカ）で自動的に維持されるため、TiKVはネイティブの高可用性を持ち、自動フェイルオーバーをサポートします。 TiKVは水平スケーラブルであり、トランザクショナルストアの数を増やすとOLTPスループットが増加します。

TiFlashは、トランザクショナルストア（TiKV）からデータをリアルタイムで複製し、リアルタイムのOLAPワークロードをサポートする解析ストレージです。 TiFlashはデータを列に格納して解析処理を高速化します。 TiFlashも水平スケーラブルであり、TiFlashノードを増やすとOLAPストレージとコンピューティング能力が増加します。

PD（Placement Driver）は、クラスタのメタデータを格納することで、TiDBクラスタ全体の「脳」となります。 PDは、TiKVノードがリアルタイムで報告するデータ分布状態に応じて、特定のTiKVノードにデータスケジューリングコマンドを送信します。 TiDB Cloudでは、各クラスタのPDはPingCAPによって管理され、ユーザーはそれを見たり維持したりすることはできません。

### TiDBは、TiKVノード間でデータをどのように複製しますか？ {#how-does-tidb-replicate-data-between-the-tikv-nodes}

TiKVは、キー値スペースをキーレンジに分割し、各キーレンジを「リージョン」として扱います。 TiKVでは、データはクラスタ内のすべてのノードに分散され、リージョンを基本単位として使用します。 PDは、リージョンをクラスタ内のすべてのノードにできるだけ均等に広げる（スケジューリングする）責任があります。

TiDBは、リージョンごとにRaftコンセンサスアルゴリズムを使用してデータを複製します。異なるノードに格納されているリージョンの複数のレプリカがRaftグループを形成します。

各データ変更はRaftログとして記録されます。 Raftログの複製を通じて、データは安全かつ信頼性の高い方法でRaftグループの複数のノードに複製されます。

## 高可用性のFAQ {#high-availability-faq}

### TiDB Cloudはどのように高可用性を確保していますか？ {#how-does-tidb-cloud-ensure-high-availability}

TiDBは、Raftコンセンサスアルゴリズムを使用してデータが高可用性であり、Raftグループ内のストレージ全体に安全に複製されることを確実にします。データはTiKVノード間で冗長にコピーされ、異なる可用性ゾーンに配置され、マシンまたはデータセンターの障害に対して保護されます。自動フェイルオーバーにより、TiDBは常にサービスを提供します。

SaaSプロバイダーとして、データセキュリティを重視しています。 [Service Organization Control（SOC）2 Type 1準拠](https://en.pingcap.com/press-release/pingcap-successfully-completes-soc-2-type-1-examination-for-tidb-cloud/)で必要とされる厳格な情報セキュリティポリシーと手順を確立しています。これにより、データが安全で利用可能で機密性が保たれます。

## 移行のFAQ {#migration-faq}

### 他のRDBMSからTiDB Cloudへの簡単な移行パスはありますか？ {#is-there-an-easy-migration-path-from-another-rdbms-to-tidb-cloud}

TiDBはMySQLと高い互換性があります。自己ホスト型のMySQLインスタンスまたはパブリッククラウドが提供するRDSサービスから、MySQL互換のデータをスムーズにTiDBに移行できます。詳細については、[データ移行を使用したMySQL互換データベースからTiDB Cloudへの移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

## バックアップとリストアのFAQ {#backup-and-restore-faq}

### TiDB Cloudは増分バックアップをサポートしていますか？ {#does-tidb-cloud-support-incremental-backups}

いいえ。クラスタのバックアップ保持期間内の任意の時点にデータをリストアする必要がある場合は、PITR（Point-in-time Recovery）を使用できます。詳細については、[TiDB DedicatedクラスタでPITRを使用する](/tidb-cloud/backup-and-restore.md#turn-on-auto-backup)または[TiDB ServerlessクラスタでPITRを使用する](/tidb-cloud/backup-and-restore-serverless.md#restore)を参照してください。

## HTAPのFAQ {#htap-faqs}

### TiDB CloudのHTAP機能をどのように利用できますか？ {#how-do-i-make-use-of-tidb-cloud-s-htap-capabilities}

従来、オンライントランザクション処理（OLTP）データベースとオンライン解析処理（OLAP）データベースの2種類のデータベースがありました。 OLTPとOLAPのリクエストは通常、異なるデータベースで処理されていました。この従来のアーキテクチャでは、OLTPデータベースからデータをデータウェアハウスやデータレイクに移行してOLAPを行うプロセスは、時間がかかりエラーが発生しやすいものでした。

Hybrid Transactional Analytical Processing（HTAP）データベースとして、TiDB Cloudは、OLTP（TiKV）ストアとOLAP（TiFlash）ストアの間でデータを信頼性の高い方法で自動的に複製することで、システムアーキテクチャを簡素化し、メンテナンスの複雑さを減らし、トランザクションデータのリアルタイム分析をサポートします。典型的なHTAPのユースケースには、ユーザー個別化、AI推奨、不正検出、ビジネスインテリジェンス、リアルタイムレポートなどがあります。

さらなるHTAPシナリオについては、[データプラットフォームを簡素化するHTAPデータベースの構築方法](https://pingcap.com/blog/how-we-build-an-htap-database-that-simplifies-your-data-platform)を参照してください。

### データを直接TiFlashにインポートできますか？ {#can-i-import-my-data-directly-to-tiflash}

いいえ。 TiDB Cloudにデータをインポートすると、データはTiKVにインポートされます。インポートが完了した後、SQLステートメントを使用して、指定したテーブルをTiFlashにレプリケートするように指定できます。その後、TiDBは指定したテーブルのレプリカをTiFlashに作成します。詳細については、[TiFlashレプリカの作成](/tiflash/create-tiflash-replicas.md)を参照してください。

### TiFlashデータをCSV形式でエクスポートできますか？ {#can-i-export-tiflash-data-in-the-csv-format}

いいえ。 TiFlashデータはエクスポートできません。

## セキュリティのFAQ {#security-faqs}

### TiDB Cloudは安全ですか？ {#is-tidb-cloud-secure}

TiDB Cloudでは、データの安全性を確保するために、データの休止中のすべてのデータが暗号化され、クライアントとクラスタ間のネットワークトラフィックがTransport Layer Security（TLS）を使用して暗号化されます。

- データの休止中の暗号化は、暗号化されたストレージボリュームを使用して自動化されます。
- クライアントとクラスタ間の伝送中のデータの暗号化は、TiDB CloudウェブサーバーTLSとTiDBクラスタTLSを使用して自動化されます。

### TiDB Cloudはビジネスデータをどのように暗号化しますか？ {#how-does-tidb-cloud-encrypt-my-business-data}

TiDB Cloudは、ビジネスデータ（データベースデータとバックアップデータを含む）の静止状態でのストレージボリューム暗号化をデフォルトで使用します。 TiDB Cloudは、データのトランジットに対してTLS暗号化を必要とし、また、TiDB、PD、TiKV、およびTiFlash間のデータのコンポーネントレベルのTLS暗号化も必要とします。

TiDB Cloudでのビジネスデータの暗号化に関する詳細情報を入手するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

### TiDB CloudはどのTLSバージョンをサポートしていますか？ {#what-versions-of-tls-does-tidb-cloud-support}

TiDB CloudはTLS 1.2またはTLS 1.3をサポートしています。

### TiDB CloudをVPCで実行できますか？ {#can-i-run-tidb-cloud-in-my-vpc}

いいえ。 TiDB CloudはDatabase-as-a-Service（DBaaS）であり、TiDB Cloud VPCでのみ実行されます。クラウドコンピューティングの管理サービスとして、TiDB Cloudは物理ハードウェアのセットアップやソフトウェアのインストールを必要とせずにデータベースへのアクセスを提供します。

### 私のTiDBクラスタは安全ですか？ {#is-my-tidb-cluster-secure}

TiDB Cloudでは、必要に応じてTiDB DedicatedクラスタまたはTiDB Serverlessクラスタを使用できます。

TiDB Dedicatedクラスタの場合、TiDB Cloudは次の対策でクラスタのセキュリティを確保します。

- 各クラスタに独立したサブアカウントとVPCを作成します。
- 外部接続を分離するためのファイアウォールルールを設定します。
- 各クラスタにサーバーサイドTLS証明書とコンポーネントレベルのTLS証明書を作成して、クラスタデータのトランジットを暗号化します。
- 各クラスタにIPアクセスルールを提供して、許可されたソースIPアドレスのみがクラスタにアクセスできるようにします。

TiDB Serverlessクラスタの場合、TiDB Cloudは次の対策でクラスタのセキュリティを確保します。

- 各クラスタに独立したサブアカウントを作成します。
- 外部接続を分離するためのファイアウォールルールを設定します。
- クラスタサーバTLS証明書を提供して、クラスタデータのトランジットを暗号化します。

### TiDBクラスタに接続する方法は？ {#how-do-i-connect-to-my-database-in-a-tidb-cluster}

\<TiDBクラスタに接続する方法は以下の通りです。

1. ネットワークを承認します。
2. データベースユーザーとログイン資格情報を設定します。
3. クラスタサーバのTLSをダウンロードして構成します。
4. SQLクライアントを選択し、TiDB Cloud UIに表示される自動生成された接続文字列を取得し、その文字列を使用してSQLクライアントを介してクラスタに接続します。

詳細については、[TiDB Dedicatedクラスタに接続](/tidb-cloud/connect-to-tidb-cluster.md)を参照してください。

</div>

<div label="TiDB Serverless">

TiDB Serverlessクラスタの場合、クラスタに接続する手順は以下の通りです。

1. データベースユーザーとログイン資格情報を設定します。
2. SQLクライアントを選択し、TiDB Cloud UIに表示される自動生成された接続文字列を取得し、その文字列を使用してSQLクライアントを介してクラスタに接続します。

詳細については、[TiDB Serverlessクラスタに接続](/tidb-cloud/connect-to-tidb-cluster-serverless.md)を参照してください。

</div>
</SimpleTab>

## サポートFAQ {#support-faq}

### 顧客向けのサポートは何が利用可能ですか？ {#what-support-is-available-for-customers}

TiDB Cloudは、金融サービス、eコマース、エンタープライズアプリケーション、ゲームなどのさまざまな業界で1500以上のグローバル企業でミッションクリティカルなユースケースを実行してきたTiDBの背後にあるチームによってサポートされています。 TiDB Cloudは、各ユーザーに無料の基本サポートプランを提供し、有料プランにアップグレードして拡張サービスを利用することができます。詳細については、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)を参照してください。"
