---
title: Import Data into TiDB Cloud via MySQL CLI
summary: Learn how to import Data into TiDB Cloud via MySQL CLI.
---

# TiDB CloudをMySQL CLIを使ってデータをインポートする {#import-data-into-tidb-cloud-via-mysql-cli}

このドキュメントでは、[MySQLコマンドラインクライアント](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)を使用して、TiDB Cloudにデータをインポートする方法について説明します。SQLファイルまたはCSVファイルからデータをインポートすることができます。次のセクションでは、それぞれのファイルタイプからデータをインポートするためのステップバイステップの手順を提供します。

## 前提条件 {#prerequisites}

MySQL CLIを使用してTiDB Cloudにデータをインポートする前に、次の前提条件を満たす必要があります。

- TiDB Cloudクラスタにアクセスできること。TiDBクラスタを持っていない場合は、[TiDB Serverlessクラスタを構築する](/develop/dev-guide-build-cluster-in-cloud.md)の手順に従って作成してください。
- ローカルコンピュータにMySQL CLIをインストールしてください。

## ステップ1. TiDB Cloudクラスタに接続する {#step-1-connect-to-your-tidb-cloud-cluster}

TiDBのデプロイオプションに応じて、TiDBクラスタに接続してください。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が操作環境と一致することを確認してください。

   - **Endpoint Type**が`Public`に設定されていること。
   - **Connect With**が`MySQL CLI`に設定されていること。
   - **Operating System**が操作環境と一致すること。

4. **Generate Password**をクリックして、ランダムなパスワードを作成します。

   > **ヒント:**
   >
   > 以前にパスワードを作成したことがある場合は、元のパスワードを使用するか、**Reset Password**をクリックして新しいパスワードを生成してください。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして、概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックします。

   接続文字列を取得する詳細については、[標準接続を介してTiDB Dedicatedに接続する](/tidb-cloud/connect-via-standard-connection.md)を参照してください。

</div>
</SimpleTab>

## ステップ2. テーブルを定義し、サンプルデータを挿入する {#step-2-define-the-table-and-insert-sample-data}

データをインポートする前に、テーブル構造を準備し、実際のサンプルデータを挿入する必要があります。次の例は、テーブルを作成し、サンプルデータを挿入するために使用できるSQLファイル（`product_data.sql`）です：

```sql
-- Create a table in your TiDB database
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(255),
    price DECIMAL(10, 2)
);

-- Insert sample data into the table
INSERT INTO products (product_id, product_name, price) VALUES
    (1, 'Laptop', 999.99),
    (2, 'Smartphone', 499.99),
    (3, 'Tablet', 299.99);
```

## ステップ3. SQLファイルまたはCSVファイルからデータをインポートする {#step-3-import-data-from-a-sql-or-csv-file}

SQLファイルまたはCSVファイルからデータをインポートすることができます。以下のセクションでは、それぞれのタイプからデータをインポートするためのステップバイステップの手順を提供します。

<SimpleTab>
<div label="SQLファイルから">

SQLファイルからデータをインポートするには、以下の手順を実行してください。

1. インポートしたいデータが含まれる実際のSQLファイル（例：`product_data.sql`）を用意します。このSQLファイルには、実際のデータを含む`INSERT`ステートメントが含まれている必要があります。

2. 以下のコマンドを使用して、SQLファイルからデータをインポートします。

   ```bash
   mysql --comments --connect-timeout 150 -u '<your_username>' -h <your_cluster_host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=<your_ca_path> -p <your_password> < product_data.sql
   ```

> **Note:**
>
> ここで使用されているデフォルトのデータベース名は`test`です。手動でデータベースを作成するか、SQLファイル内の`CREATE DATABASE`コマンドを使用することができます。

</div>
<div label="CSVファイルから">

CSVファイルからデータをインポートするには、以下の手順を実行してください。

1. TiDBにデータをインポートするために、データベースとスキーマを作成します。

2. インポートしたいデータが含まれるサンプルのCSVファイル（例：`product_data.csv`）を用意します。以下はCSVファイルの例です。

   **product\_data.csv:**

   ```csv
   product_id,product_name,price
   4,Laptop,999.99
   5,Smartphone,499.99
   6,Tablet,299.99
   ```

3. 以下のコマンドを使用して、CSVファイルからデータをインポートします。

   ```bash
   mysql --comments --connect-timeout 150 -u '<your_username>' -h <your_host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=<your_ca_path> -p<your_password> -e "LOAD DATA LOCAL INFILE '<your_csv_path>' INTO TABLE products
   FIELDS TERMINATED BY ','
   LINES TERMINATED BY '\n'
   IGNORE 1 LINES (product_id, product_name, price);"
   ```

4. パス、テーブル名（この例では`products`）、`<your_username>`、`<your_host>`、`<your_password>`、`<your_csv_path>`、`<your_ca_path>`、およびその他のプレースホルダーを実際の情報に置き換え、必要に応じてサンプルのCSVデータを実際のデータセットに置き換えてください。

> **Note:**
>
> `LOAD DATA LOCAL INFILE`の構文の詳細については、[`LOAD DATA`](/sql-statements/sql-statement-load-data.md)を参照してください。

</div>
</SimpleTab>
