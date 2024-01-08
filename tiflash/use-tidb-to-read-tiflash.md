---
title: Use TiDB to Read TiFlash Replicas
summary: Learn how to use TiDB to read TiFlash replicas.
---

# TiDBを使用してTiFlashレプリカを読む {#use-tidb-to-read-tiflash-replicas}

このドキュメントでは、TiDBを使用してTiFlashレプリカを読む方法について紹介します。

TiDBは、TiFlashレプリカを読むための3つの方法を提供します。エンジンの構成を行わずにTiFlashレプリカを追加した場合、CBO（コストベースの最適化）モードがデフォルトで使用されます。

## スマート選択 {#smart-selection}

TiFlashレプリカを持つテーブルに対して、TiDBオプティマイザはコストの見積もりに基づいて自動的にTiFlashレプリカを使用するかどうかを決定します。`desc`または`explain analyze`ステートメントを使用して、TiFlashレプリカが選択されているかどうかを確認できます。例：

```sql
desc select count(*) from test.t;
```

```sql
explain analyze select count(*) from test.t;
```

`cop[tiflash]`は、タスクが処理のためにTiFlashに送信されることを意味します。TiFlashレプリカを選択していない場合は、`analyze table`ステートメントを使用して統計情報を更新し、その後`explain analyze`ステートメントを使用して結果を確認できます。

テーブルに単一のTiFlashレプリカのみがあり、関連ノードがサービスを提供できない場合、CBOモードのクエリは繰り返しリトライされます。この状況では、エンジンを指定するか、TiKVレプリカからデータを読むために手動のヒントを使用する必要があります。

## エンジンの分離 {#engine-isolation}

エンジンの分離は、対応する変数を構成することで、すべてのクエリが指定されたエンジンのレプリカを使用するように指定することです。オプションのエンジンは、"tikv"、"tidb"（TiDBの内部メモリテーブル領域を示し、一部のTiDBシステムテーブルを格納し、ユーザーによって積極的に使用されることはできません）、"tiflash"です。

<CustomContent platform="tidb">

次の2つの構成レベルでエンジンを指定できます：

- TiDBインスタンスレベル、つまりINSTANCEレベル。TiDB構成ファイルに次の構成項目を追加します：

  ```
  [isolation-read]
  engines = ["tikv", "tidb", "tiflash"]
  ```

  **INSTANCEレベルのデフォルト構成は`["tikv", "tidb", "tiflash"]`です。**

- SESSIONレベル。次のステートメントを使用して構成します：

  ```sql
  set @@session.tidb_isolation_read_engines = "engine list separated by commas";
  ```

  または

  ```sql
  set SESSION tidb_isolation_read_engines = "engine list separated by commas";
  ```

  SESSIONレベルのデフォルト構成は、TiDB INSTANCEレベルの構成から継承されます。

最終的なエンジン構成はセッションレベルの構成であり、つまりセッションレベルの構成がインスタンスレベルの構成をオーバーライドします。たとえば、INSTANCEレベルで"tikv"を構成し、SESSIONレベルで"tiflash"を構成した場合、TiFlashレプリカが読み取られます。最終的なエンジン構成が"tikv"と"tiflash"である場合、TiKVとTiFlashレプリカの両方が読み取られ、オプティマイザは自動的により良いエンジンを選択して実行します。

> **Note:**
>
> [TiDB Dashboard](/dashboard/dashboard-intro.md)およびその他のコンポーネントは、TiDBメモリテーブル領域に格納されているシステムテーブルを読み取る必要があるため、インスタンスレベルのエンジン構成に常に"tidb"エンジンを追加することをお勧めします。

</CustomContent>

<CustomContent platform="tidb-cloud">

次のステートメントを使用してエンジンを指定できます：

```sql
set @@session.tidb_isolation_read_engines = "engine list separated by commas";
```

または

```sql
set SESSION tidb_isolation_read_engines = "engine list separated by commas";
```

</CustomContent>

指定されたテーブルに指定されたエンジンのレプリカがない場合（たとえば、エンジンが"tiflash"に構成されているが、テーブルにTiFlashレプリカがない場合）、クエリはエラーを返します。

## 手動ヒント {#manual-hint}

手動ヒントは、エンジンの分離の条件を満たす前提で、特定のテーブルの指定されたレプリカを使用するようにTiDBに強制することができます。以下は手動ヒントを使用した例です：

```sql
select /*+ read_from_storage(tiflash[table_name]) */ ... from table_name;
```

クエリステートメントでテーブルにエイリアスを設定した場合、ヒントを含むステートメントでヒントが有効になるように、ステートメントでエイリアスを使用する必要があります。例：

```sql
select /*+ read_from_storage(tiflash[alias_a,alias_b]) */ ... from table_name_1 as alias_a, table_name_2 as alias_b where alias_a.column_1 = alias_b.column_2;
```

上記のステートメントでは、`tiflash[]`はオプティマイザにTiFlashレプリカを読み取るように促します。必要に応じて、`tikv[]`を使用してオプティマイザにTiKVレプリカを読み取るように促すこともできます。ヒントの構文の詳細については、[READ\_FROM\_STORAGE](/optimizer-hints.md#read_from_storagetiflasht1_name--tl_name--tikvt2_name--tl_name-)を参照してください。

ヒントで指定されたテーブルに指定されたエンジンのレプリカがない場合、ヒントは無視され、警告が報告されます。また、ヒントはエンジンの分離の条件を満たす前提でのみ有効です。ヒントで指定されたエンジンがエンジン分離リストに含まれていない場合、ヒントは無視され、警告が報告されます。

> **Note:**
>
> 5.7.7以前のバージョンのMySQLクライアントは、デフォルトでオプティマイザヒントをクリアします。これらの古いバージョンでヒント構文を使用するには、`--comments`オプションを指定してクライアントを起動する必要があります。たとえば、`mysql -h 127.0.0.1 -P 4000 -uroot --comments`です。

## スマート選択、エンジンの分離、および手動ヒントの関係 {#the-relationship-of-smart-selection-engine-isolation-and-manual-hint}

上記の3つのTiFlashレプリカの読み取り方法では、エンジンの分離は利用可能なエンジンのレプリカの全体範囲を指定します。この範囲内で、手動ヒントはより細かい粒度のステートメントレベルおよびテーブルレベルのエンジン選択を提供します。最終的にCBOは決定を行い、指定されたエンジンリスト内でコストの見積もりに基づいてエンジンのレプリカを選択します。

> **Note:**
>
> - v4.0.3以前では、読み取り専用でないSQLステートメント（たとえば、`INSERT INTO ... SELECT`、`SELECT ... FOR UPDATE`、`UPDATE ...`、`DELETE ...`）でTiFlashレプリカから読み取る動作は未定義です。
> - v4.0.3からv6.2.0までのバージョンでは、内部的にTiDBはデータの正確性を保証するために、読み取り専用でないSQLステートメントのTiFlashレプリカを無視します。つまり、[スマート選択](#smart-selection)では、TiDBは自動的にTiFlashレプリカ以外を選択します。TiFlashレプリカのみを指定する[エンジンの分離](#engine-isolation)の場合、TiDBはエラーを報告します。また、[手動ヒント](#manual-hint)の場合、TiDBはヒントを無視します。
> - v6.3.0からv7.0.0までのバージョンでは、TiFlashレプリカが有効な場合、[`tidb_enable_tiflash_read_for_write_stmt`](/system-variables.md#tidb_enable_tiflash_read_for_write_stmt-new-in-v630)変数を使用して、TiDBが読み取り専用でないSQLステートメントでTiFlashレプリカを使用するかどうかを制御できます。
> - v7.1.0以降、TiFlashレプリカが有効であり、現在のセッションの[SQLモード](/sql-mode.md)が厳密でない場合（つまり、`sql_mode`値に`STRICT_TRANS_TABLES`または`STRICT_ALL_TABLES`が含まれていない場合）、TiDBはコストの見積もりに基づいて読み取り専用でないSQLステートメントでTiFlashレプリカを使用するかどうかを自動的に決定します。"
