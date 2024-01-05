---
title: Connect to TiDB with Sequelize
summary: Learn how to connect to TiDB using Sequelize. This tutorial gives Node.js sample code snippets that work with TiDB using Sequelize.
---

# Sequelizeを使用してTiDBに接続 {#connect-to-tidb-with-sequelize}

TiDBはMySQL互換のデータベースであり、[Sequelize](https://sequelize.org/)はNode.js向けの人気のあるORMフレームワークです。

このチュートリアルでは、TiDBとSequelizeを使用して次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップします。
- Sequelizeを使用してTiDBクラスタに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- [Node.js **18**](https://nodejs.org/en/download/) またはそれ以降。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続 {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

> **注意**
>
> 完全なコードスニペットと実行手順については、[tidb-samples/tidb-nodejs-sequelize-quickstart](https://github.com/tidb-samples/tidb-nodejs-sequelize-quickstart) GitHubリポジトリを参照してください。

### ステップ1：サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```bash
git clone git@github.com:tidb-samples/tidb-nodejs-sequelize-quickstart.git
cd tidb-nodejs-sequelize-quickstart
```

### ステップ2：依存関係のインストール {#step-2-install-dependencies}

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`sequelize`を含む）をインストールします：

```bash
npm install
```

### ステップ3：接続情報の設定 {#step-3-configure-connection-information}

選択したTiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>

<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致していることを確認します。

   - **エンドポイントタイプ** が `Public` に設定されていること

   - **Branch** が `main` に設定されていること

   - **Connect With** が `General` に設定されていること

   - **Operating System** が環境に一致していること

   > **注意**
   >
   > Node.jsアプリケーションでは、TLS（SSL）接続を確立する際にデフォルトでNode.jsが組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、SSL CA証明書を提供する必要はありません。

4. **Generate Password** をクリックしてランダムなパスワードを作成します。

   > **ヒント**
   >
   > 以前にパスワードを生成した場合、元のパスワードを使用するか、**Reset Password** をクリックして新しいパスワードを生成できます。

5. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します：

   ```shell
   cp .env.example .env
   ```

6. `.env` ファイルを編集し、接続ダイアログの対応するプレースホルダ `{}` を接続パラメータで置き換えて、環境変数を次のように設定します：

   ```dotenv
   TIDB_HOST='{host}'
   TIDB_PORT='4000'
   TIDB_USER='{user}'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   TIDB_ENABLE_SSL='true'
   ```

7. `.env` ファイルを保存します。

</div>

<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **どこからでもアクセスを許可** をクリックし、次に **TiDBクラスタCAをダウンロード** をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します：

   ```shell
   cp .env.example .env
   ```

5. `.env` ファイルを編集し、接続ダイアログの対応するプレースホルダ `{}` を接続パラメータで置き換えて、環境変数を次のように設定します：

   ```shell
   TIDB_HOST='{host}'
   TIDB_PORT='4000'
   TIDB_USER='{user}'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   TIDB_ENABLE_SSL='true'
   TIDB_CA_PATH='{path/to/ca}'
   ```

6. `.env` ファイルを保存します。

</div>

<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `.env.example` をコピーし、`.env` に名前を変更します：

   ```shell
   cp .env.example .env
   ```

2. `.env` ファイルを編集し、接続ダイアログの対応するプレースホルダ `{}` を接続パラメータで置き換えて、環境変数を次のように設定します：

   ```shell
   TIDB_HOST='{host}'
   TIDB_PORT='4000'
   TIDB_USER='root'
   TIDB_PASSWORD='{password}'
   TIDB_DB_NAME='test'
   ```

   TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3. `.env` ファイルを保存します。

</div>

</SimpleTab>

### ステップ4：サンプルアプリを実行 {#step-4-run-the-sample-app}

次のコマンドを実行してサンプルコードを実行します：

```shell
npm start
```

<details>
<summary>**期待される出力（一部）：**</summary>

```shell
INFO (app/10117): Getting sequelize instance...
Executing (default): SELECT 1+1 AS result
Executing (default): DROP TABLE IF EXISTS `players`;
Executing (default): CREATE TABLE IF NOT EXISTS `players` (`id` INTEGER NOT NULL auto_increment  COMMENT 'The unique ID of the player.', `coins` INTEGER NOT NULL COMMENT 'The number of coins that the player had.', `goods` INTEGER NOT NULL COMMENT 'The number of goods that the player had.', `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB;
Executing (default): SHOW INDEX FROM `players`
Executing (default): INSERT INTO `players` (`id`,`coins`,`goods`,`createdAt`,`updatedAt`) VALUES (1,100,100,'2023-08-31 09:10:11','2023-08-31 09:10:11'),(2,200,200,'2023-08-31 09:10:11','2023-08-31 09:10:11'),(3,300,300,'2023-08-31 09:10:11','2023-08-31 09:10:11'),(4,400,400,'2023-08-31 09:10:11','2023-08-31 09:10:11'),(5,500,500,'2023-08-31 09:10:11','2023-08-31 09:10:11');
Executing (default): SELECT `id`, `coins`, `goods`, `createdAt`, `updatedAt` FROM `players` AS `players` WHERE `players`.`coins` > 300;
Executing (default): UPDATE `players` SET `coins`=?,`goods`=?,`updatedAt`=? WHERE `id` = ?
Executing (default): DELETE FROM `players` WHERE `id` = 6
```

</details>

## サンプルコードスニペット {#sample-code-snippets}

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-sequelize-quickstart](https://github.com/tidb-samples/tidb-nodejs-sequelize-quickstart) リポジトリをチェックしてください。

### TiDBへの接続 {#connect-to-tidb}

次のコードは、環境変数で定義されたオプションを使用して、TiDBへの接続を確立します。

```typescript
// src/lib/tidb.ts
import { Sequelize } from 'sequelize';

export function initSequelize() {
  return new Sequelize({
    dialect: 'mysql',
    host: process.env.TIDB_HOST || 'localhost',     // TiDB host, for example: {gateway-region}.aws.tidbcloud.com
    port: Number(process.env.TIDB_PORT) || 4000,    // TiDB port, default: 4000
    username: process.env.TIDB_USER || 'root',      // TiDB user, for example: {prefix}.root
    password: process.env.TIDB_PASSWORD || 'root',  // TiDB password
    database: process.env.TIDB_DB_NAME || 'test',   // TiDB database name, default: test
    dialectOptions: {
      ssl:
        process.env?.TIDB_ENABLE_SSL === 'true'     // (Optional) Enable SSL
          ? {
              minVersion: 'TLSv1.2',
              rejectUnauthorized: true,
              ca: process.env.TIDB_CA_PATH          // (Optional) Path to the custom CA certificate
                ? readFileSync(process.env.TIDB_CA_PATH)
                : undefined,
            }
          : null,
    },
}

export async function getSequelize() {
  if (!sequelize) {
    sequelize = initSequelize();
    try {
      await sequelize.authenticate();
      logger.info('Connection has been established successfully.');
    } catch (error) {
      logger.error('Unable to connect to the database:');
      logger.error(error);
      throw error;
    }
  }
  return sequelize;
}
```

### データの挿入 {#insert-data}

次のクエリは単一の `Players` レコードを作成し、`Players` オブジェクトを返します：

```typescript
logger.info('Creating a new player...');
const newPlayer = await playersModel.create({
  id: 6,
  coins: 600,
  goods: 600,
});
logger.info('Created a new player.');
logger.info(newPlayer.toJSON());
```

For more information, refer to [データの挿入](/develop/dev-guide-insert-data.md).

### データのクエリ {#query-data}

次のクエリは、コインが `300` よりも多い単一の `Players` レコードを返します。

```typescript
logger.info('Reading all players with coins > 300...');
const allPlayersWithCoinsGreaterThan300 = await playersModel.findAll({
  where: {
    coins: {
      [Op.gt]: 300,
    },
  },
});
logger.info('Read all players with coins > 300.');
logger.info(allPlayersWithCoinsGreaterThan300.map((p) => p.toJSON()));
```

For more information, refer to [Query data](/develop/dev-guide-get-data-from-single-table.md).

### データの更新 {#update-data}

次のクエリセットは、IDが「6」の「Players」に700コインと700個の商品を設定します。これは[データの挿入](#insert-data)セクションで作成されました。

```typescript
logger.info('Updating the new player...');
await newPlayer.update({ coins: 700, goods: 700 });
logger.info('Updated the new player.');
logger.info(newPlayer.toJSON());
```

For more information, refer to [データの更新](/develop/dev-guide-update-data.md).

### データの削除 {#delete-data}

次のクエリは、[データの挿入](#insert-data)セクションで作成されたID `6`の`Player`レコードを削除します。

```typescript
logger.info('Deleting the new player...');
await newPlayer.destroy();
const deletedNewPlayer = await playersModel.findByPk(6);
logger.info('Deleted the new player.');
logger.info(deletedNewPlayer?.toJSON());
```

For more information, refer to [Delete data](/develop/dev-guide-delete-data.md).

## Next steps {#next-steps}

- Learn more usage of the ORM framework Sequelize driver from [the documentation of Sequelize](https://sequelize.org/).
- Learn the best practices for TiDB application development with the chapters in the [Developer guide](/develop/dev-guide-overview.md), such as [Insert data](/develop/dev-guide-insert-data.md), [Update data](/develop/dev-guide-update-data.md), [Delete data](/develop/dev-guide-delete-data.md), [Single table reading](/develop/dev-guide-get-data-from-single-table.md), [Transactions](/develop/dev-guide-transaction-overview.md), and [SQL performance optimization](/develop/dev-guide-optimize-sql-overview.md).
- Learn through the professional [TiDB developer courses](https://www.pingcap.com/education/) and earn [TiDB certifications](https://www.pingcap.com/education/certification/) after passing the exam.

## Need help? {#need-help}

Ask questions on the [Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc), or [create a support ticket](https://support.pingcap.com/).
