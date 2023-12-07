---
title: Manage TiDB Serverless Branches
summary: Learn How to manage TiDB Serverless branches.
---

# TiDB Serverless ブランチの管理 {#manage-tidb-serverless-branches}

このドキュメントでは、[TiDB Cloudコンソール](https://tidbcloud.com)を使用してTiDB Serverlessブランチを管理する方法について説明します。TiDB Cloud CLIを使用して管理する場合は、[`ticloud branch`](/tidb-cloud/ticloud-branch-create.md)を参照してください。

## 必要なアクセス権限 {#required-access}

-   [ブランチを作成](#create-a-branch)したり、[ブランチに接続](#connect-to-a-branch)するには、組織の`Organization Owner`ロールまたは対象プロジェクトの`Project Owner`ロールに属している必要があります。
-   プロジェクト内のクラスタのブランチを[表示](#create-a-branch)するには、そのプロジェクトに属している必要があります。

アクセス権限に関する詳細は、[ユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles)を参照してください。

## ブランチの作成 {#create-a-branch}

> **注意:**
>
> 2023年7月5日以降に作成されたTiDB Serverlessクラスタにのみブランチを作成できます。その他の制限事項については、[制限事項とクォータ](/tidb-cloud/branch-overview.md#limitations-and-quotas)を参照してください。

ブランチを作成するには、以下の手順を実行してください:

1.  [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2.  左側のナビゲーションペインで**ブランチ**をクリックします。
3.  右上隅の**ブランチの作成**をクリックします。
4.  ブランチ名を入力し、**作成**をクリックします。

クラスタ内のデータサイズに応じて、ブランチの作成は数分で完了します。

## ブランチの表示 {#view-branches}

クラスタのブランチを表示するには、以下の手順を実行してください:

1.  [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2.  左側のナビゲーションペインで**ブランチ**をクリックします。

    クラスタのブランチリストが右側のペインに表示されます。

## ブランチへの接続 {#connect-to-a-branch}

ブランチに接続するには、以下の手順を実行してください:

1.  [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2.  左側のナビゲーションペインで**ブランチ**をクリックします。
3.  接続する対象のブランチの行で、**アクション**列の\*\*...\*\*をクリックします。
4.  ドロップダウンリストで**接続**をクリックします。接続情報のダイアログが表示されます。
5.  ルートパスワードを作成またはリセットするには、**パスワードの生成**または**パスワードのリセット**をクリックします。
6.  接続情報を使用してブランチに接続します。

または、クラスタの概要ページから接続文字列を取得することもできます:

1.  [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2.  右上隅の**接続**をクリックします。
3.  `ブランチ`のドロップダウンリストで接続したいブランチを選択します。
4.  ルートパスワードを作成またはリセットするには、**パスワードの生成**または**パスワードのリセット**をクリックします。
5.  接続情報を使用してブランチに接続します。

## ブランチの削除 {#delete-a-branch}

ブランチを削除するには、以下の手順を実行してください:

1.  [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2.  左側のナビゲーションペインで**ブランチ**をクリックします。
3.  削除する対象のブランチの行で、**アクション**列の\*\*...\*\*をクリックします。
4.  ドロップダウンリストで**削除**をクリックします。
5.  削除を確認します。

## 次のステップ {#what-s-next}

-   [TiDB ServerlessブランチをGitHubのCI/CDパイプラインに統合する](/tidb-cloud/branch-github-integration.md)
