---
title: Slow Queries Page of TiDB Dashboard
summary: Learn the Slow Queries page of TiDB Dashboard.
---

# TiDBダッシュボードの遅いクエリページ {#slow-queries-page-of-tidb-dashboard}

TiDBダッシュボードの遅いクエリページでは、クラスタ内のすべての遅いクエリを検索して表示することができます。

デフォルトでは、実行時間が300ミリ秒を超えるSQLクエリは遅いクエリと見なされます。これらのクエリは[遅いクエリログ](/identify-slow-queries.md)に記録され、TiDBダッシュボードを介して検索することができます。遅いクエリのしきい値は、[`tidb_slow_log_threshold`](/system-variables.md#tidb_slow_log_threshold)セッション変数または[`instance.tidb_slow_log_threshold`](/tidb-configuration-file.md#tidb_slow_log_threshold) TiDBパラメータを介して調整することができます。

> **Note:**
>
> 遅いクエリログが無効になっている場合、この機能は利用できません。遅いクエリログはデフォルトで有効になっており、システム変数[`tidb_enable_slow_log`](/system-variables.md#tidb_enable_slow_log)を介して有効または無効にすることができます。

## ページへのアクセス {#access-the-page}

遅いクエリページにアクセスするには、次の2つの方法のいずれかを使用できます。

- TiDBダッシュボードにログインした後、左のナビゲーションメニューで**遅いクエリ**をクリックします。

- ブラウザで<http://127.0.0.1:2379/dashboard/#/slow_query>を訪れます。`127.0.0.1:2379`を実際のPDアドレスとポートに置き換えてください。

遅いクエリページに表示されるすべてのデータは、TiDB遅いクエリシステムテーブルと遅いクエリログから取得されます。詳細については[遅いクエリログ](/identify-slow-queries.md)を参照してください。

### フィルタの変更 {#change-filters}

時間範囲、関連するデータベース、SQLキーワード、SQLタイプ、表示する遅いクエリの数に基づいて遅いクエリをフィルタリングすることができます。以下の画像では、直近30分間の100件の遅いクエリがデフォルトで表示されています。

![リストフィルタの変更](/media/dashboard/dashboard-slow-queries-list1-v620.png)

### より多くのカラムを表示 {#display-more-columns}

ページ上で**カラム**をクリックすると、より多くのカラムを表示することができます。カラム名の右側の\*\*(i)\*\*アイコンにマウスを移動して、このカラムの説明を表示することができます。

![より多くのカラムを表示](/media/dashboard/dashboard-slow-queries-list2-v620.png)

### 遅いクエリのローカルエクスポート {#export-slow-queries-locally}

ページの右上隅にある☰（**その他**）をクリックして**エクスポート**オプションを表示します。**エクスポート**をクリックすると、TiDBダッシュボードは現在のリストの遅いクエリをCSVファイルとしてエクスポートします。

![遅いクエリのローカルエクスポート](/media/dashboard/dashboard-slow-queries-export-v651.png)

### カラムでソート {#sort-by-column}

デフォルトでは、リストは**終了時間**で降順にソートされています。カラムの見出しをクリックして、そのカラムでソートするかソート順序を切り替えることができます。

![ソート基準の変更](/media/dashboard/dashboard-slow-queries-list3-v620.png)

## 実行の詳細を表示 {#view-execution-details}

リスト内の任意のアイテムをクリックすると、遅いクエリの詳細な実行情報が表示されます。これには以下が含まれます：

- クエリ：SQLステートメントのテキスト（以下の図の領域1）
- プラン：遅いクエリの実行プラン（以下の図の領域2）
- その他のソートされたSQL実行情報（以下の図の領域3）

![実行の詳細を表示](/media/dashboard/dashboard-slow-queries-detail1-v620.png)

### SQL {#sql}

> **Note:**
>
> `Query`列に記録されるクエリの最大長は、[`tidb_stmt_summary_max_sql_length`](/system-variables.md#tidb_stmt_summary_max_sql_length-new-in-v40)システム変数によって制限されています。

**展開**ボタンをクリックしてアイテムの詳細情報を表示します。**コピー**ボタンをクリックして、詳細情報をクリップボードにコピーします。

### 実行プラン {#execution-plans}

TiDBダッシュボードでは、グラフとテキストの2つの方法で実行プランを表示することができます。ビジュアル実行プランを使用すると、ステートメントの各オペレータと詳細情報を直感的に学ぶことができます。実行プランの読み方については、[クエリ実行プランの理解](/explain-overview.md)を参照してください。

#### ビジュアル実行プラン {#visual-execution-plans}

以下の図はビジュアル実行プランを示しています。

![ビジュアル実行プラン](/media/dashboard/dashboard-visual-plan-2.png)

- グラフは左から右、上から下に実行を示しています。
- 上位ノードは親オペレータであり、下位ノードは子オペレータです。
- タイトルバーの色は、オペレータが実行されるコンポーネントを示しています：黄色はTiDB、青色はTiKV、ピンク色はTiFlashです。
- タイトルバーにはオペレータ名が表示され、以下のテキストはオペレータの基本情報です。

ノード領域をクリックすると、詳細なオペレータ情報が右側のサイドバーに表示されます。

![ビジュアル実行プラン - サイドバー](/media/dashboard/dashboard-visual-plan-popup.png)

### SQL実行の詳細 {#sql-execution-details}

対応するタブのタイトルをクリックして、SQL実行の情報を切り替えます。

![異なる実行情報を表示](/media/dashboard/dashboard-slow-queries-detail2-v620.png)
