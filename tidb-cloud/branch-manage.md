---
title: Manage TiDB Serverless Branches
summary: Learn How to manage TiDB Serverless branches.
---

# TiDB Serverless Branchesの管理 {#manage-tidb-serverless-branches}

このドキュメントでは、[TiDB Cloudコンソール](https://tidbcloud.com)を使用してTiDB Serverlessブランチを管理する方法について説明します。TiDB Cloud CLIを使用して管理するには、[`ticloud branch`](/tidb-cloud/ticloud-branch-create.md)を参照してください。

## 必要なアクセス {#required-access}

- [ブランチを作成](#create-a-branch)したり、[ブランチに接続](#connect-to-a-branch)するには、組織の`Organization Owner`または対象プロジェクトの`Project Owner`の役割にいる必要があります。
- プロジェクト内のクラスタの[ブランチを表示](#create-a-branch)するには、そのプロジェクトに属している必要があります。

アクセス権限の詳細については、[ユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles)を参照してください。

## ブランチの作成 {#create-a-branch}

> **Note:**
>
> 2023年7月5日以降に作成されたTiDB Serverlessクラスタにのみブランチを作成できます。その他の制限事項については、[制限事項とクォータ](/tidb-cloud/branch-overview.md#limitations-and-quotas)を参照してください。

ブランチを作成するには、以下の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 右上隅にある**Create Branch**をクリックします。
4. ブランチ名を入力し、**Create**をクリックします。

クラスタ内のデータサイズに応じて、ブランチの作成は数分で完了します。

## ブランチの表示 {#view-branches}

クラスタのブランチを表示するには、以下の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2. 左側のナビゲーションペインで**Branches**をクリックします。

   クラスタのブランチリストが右側のペインに表示されます。

## ブランチに接続 {#connect-to-a-branch}

ブランチに接続するには、以下の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 接続する対象のブランチの行で、**Action**列の\*\*...\*\*をクリックします。
4. ドロップダウンリストで**Connect**をクリックします。接続情報のダイアログが表示されます。
5. **Generate Password**または**Reset Password**をクリックして、ルートパスワードを作成またはリセットします。
6. 接続情報を使用してブランチに接続します。

また、クラスタの概要ページから接続文字列を取得することもできます。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2. 右上隅にある**Connect**をクリックします。
3. `Branch`のドロップダウンリストで接続するブランチを選択します。
4. **Generate Password**または**Reset Password**をクリックして、ルートパスワードを作成またはリセットします。
5. 接続情報を使用してブランチに接続します。

## ブランチの削除 {#delete-a-branch}

ブランチを削除するには、以下の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの名前をクリックして概要ページに移動します。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 削除する対象のブランチの行で、**Action**列の\*\*...\*\*をクリックします。
4. ドロップダウンリストで**Delete**をクリックします。
5. 削除を確認します。

## 次のステップ {#what-s-next}

- [GitHub CI/CDパイプラインにTiDB Serverlessブランチを統合](/tidb-cloud/branch-github-integration.md)
