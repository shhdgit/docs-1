---
title: Manage TiDB Serverless Branches
summary: Learn How to manage TiDB Serverless branches.
---

# TiDB Serverless ブランチの管理 {#manage-tidb-serverless-branches}

このドキュメントでは、[TiDB Cloudコンソール](https://tidbcloud.com)を使用してTiDB Serverlessブランチを管理する方法について説明します。TiDB Cloud CLIを使用して管理する場合は、[`ticloud branch`](/tidb-cloud/ticloud-branch-create.md)を参照してください。

## 必要なアクセス権限 {#required-access}

- [ブランチを作成](#create-a-branch)したり、[ブランチに接続](#connect-to-a-branch)したりするには、組織の`Organization Owner`ロールまたは対象プロジェクトの`Project Owner`ロールに所属する必要があります。
- プロジェクト内のクラスタの[ブランチを表示](#create-a-branch)するには、そのプロジェクトに所属する必要があります。

アクセス権限の詳細については、[ユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles)を参照してください。

## ブランチの作成 {#create-a-branch}

> **Note:**
>
> 2023年7月5日以降に作成されたTiDB Serverlessクラスタに対してのみ、ブランチを作成できます。詳細な制限事項については、[制限事項とクォータ](/tidb-cloud/branch-overview.md#limitations-and-quotas)を参照してください。

ブランチを作成するには、次の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの概要ページを開きます。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 右上隅の**Create Branch**をクリックします。
4. ブランチ名を入力し、**Create**をクリックします。

クラスタ内のデータサイズに応じて、ブランチの作成には数分かかります。

## ブランチの表示 {#view-branches}

クラスタのブランチを表示するには、次の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの概要ページを開きます。
2. 左側のナビゲーションペインで**Branches**をクリックします。

   クラスタのブランチリストが右側のペインに表示されます。

## ブランチに接続 {#connect-to-a-branch}

ブランチに接続するには、次の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの概要ページを開きます。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 接続する対象のブランチの行で、**Action**列の\*\*...\*\*をクリックします。
4. ドロップダウンリストで**Connect**をクリックします。接続情報のダイアログが表示されます。
5. ルートパスワードを作成またはリセットするには、**Generate Password**または**Reset Password**をクリックします。
6. 接続情報を使用してブランチに接続します。

または、クラスタの概要ページから接続文字列を取得できます。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの概要ページを開きます。
2. 右上隅の**Connect**をクリックします。
3. `Branch`ドロップダウンリストで接続するブランチを選択します。
4. ルートパスワードを作成またはリセットするには、**Generate Password**または**Reset Password**をクリックします。
5. 接続情報を使用してブランチに接続します。

## ブランチの削除 {#delete-a-branch}

ブランチを削除するには、次の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDB Serverlessクラスタの概要ページを開きます。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 削除する対象のブランチの行で、**Action**列の\*\*...\*\*をクリックします。
4. ドロップダウンリストで**Delete**をクリックします。
5. 削除を確認します。

## 次のステップ {#what-s-next}

- [GitHub CI/CDパイプラインにTiDB Serverlessブランチを統合する](/tidb-cloud/branch-github-integration.md)
