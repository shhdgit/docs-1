---
title: TimeStamp Oracle (TSO) in TiDB
summary: Learn about TimeStamp Oracle (TSO) in TiDB.
---

# TiDBにおけるTimeStamp Oracle（TSO） {#timestamp-oracle-tso-in-tidb}

TiDBでは、Placement Driver（PD）がクラスタ内のさまざまなコンポーネントにタイムスタンプを割り当てることで重要な役割を果たしています。これらのタイムスタンプは、トランザクションやデータに時間的なマーカーを割り当てるための仕組みであり、TiDB内で[Percolator](https://research.google.com/pubs/pub36726.html)モデルを有効にするために重要です。Percolatorモデルは、Multi-Version Concurrency Control（MVCC）と[トランザクション管理](/transaction-overview.md)をサポートするために使用されます。

次の例は、TiDBで現在のTSOを取得する方法を示しています：

```sql
BEGIN; SET @ts := @@tidb_current_ts; ROLLBACK;
Query OK, 0 rows affected (0.0007 sec)
Query OK, 0 rows affected (0.0002 sec)
Query OK, 0 rows affected (0.0001 sec)

SELECT @ts;
+--------------------+
| @ts                |
+--------------------+
| 443852055297916932 |
+--------------------+
1 row in set (0.00 sec)
```

この例では、トランザクション内で `BEGIN; ...; ROLLBACK` を使用していることに注意してください。これは、TSOタイムスタンプがトランザクションごとに割り当てられるためです。

前の例から取得したTSOタイムスタンプは10進数です。このタイムスタンプを解析するには、SQL関数 [`TIDB_PARSE_TSO()`](/functions-and-operators/tidb-functions.md#tidb_parse_tso) を使用できます。

```sql
SELECT TIDB_PARSE_TSO(443852055297916932);
+------------------------------------+
| TIDB_PARSE_TSO(443852055297916932) |
+------------------------------------+
| 2023-08-27 20:33:41.687000         |
+------------------------------------+
1 row in set (0.00 sec)
```

以下の例は、バイナリ形式で表されたTSOタイムスタンプの見本です：

```shell
0000011000101000111000010001011110111000110111000000000000000100  ← This is 443852055297916932 in binary
0000011000101000111000010001011110111000110111                    ← The first 46 bits are the physical timestamp
                                              000000000000000100  ← The last 18 bits are the logical timestamp
```

TSOタイムスタンプには2つのパーツがあります：

- 物理タイムスタンプ：1970年1月1日以来のミリ秒単位のUNIXタイムスタンプ。
- 論理タイムスタンプ：同じミリ秒内で複数のタイムスタンプが必要な場合や、特定のイベントで時計の進行が逆転する可能性がある場合に使用される、増加するカウンター。このような場合、物理タイムスタンプは変更されず、論理タイムスタンプは着実に進み続けます。このメカニズムにより、TSOタイムスタンプの整合性が保証され、常に前進し、後退することはありません。

この知識を元に、SQLでTSOタイムスタンプをより詳しく調べることができます：

```sql
SELECT @ts, UNIX_TIMESTAMP(NOW(6)), (@ts >> 18)/1000, FROM_UNIXTIME((@ts >> 18)/1000), NOW(6), @ts & 0x3FFFF\G
*************************** 1. row ***************************
                            @ts: 443852055297916932
         UNIX_TIMESTAMP(NOW(6)): 1693161835.502954
               (@ts >> 18)/1000: 1693161221.6870
FROM_UNIXTIME((@ts >> 18)/1000): 2023-08-27 20:33:41.6870
                         NOW(6): 2023-08-27 20:43:55.502954
                  @ts & 0x3FFFF: 4
1 row in set (0.00 sec)
```

`>> 18`操作は、物理タイムスタンプを抽出するために使用される18ビットのビット[右シフト](/functions-and-operators/bit-functions-and-operators.md)を示します。物理タイムスタンプはミリ秒で表されるため、より一般的なUNIXタイムスタンプ形式（秒で測定）から逸脱しているため、[`FROM_UNIXTIME()`](/functions-and-operators/date-and-time-functions.md)と互換性のある形式に変換するために1000で割る必要があります。このプロセスは、`TIDB_PARSE_TSO()`の機能と一致します。

また、2進数で`000000000000000100`という論理タイムスタンプを抽出することもでき、これは10進数の`4`に相当します。

また、次のようにCLIツールを使用してタイムスタンプを解析することもできます：

```shell
$ tiup ctl:v7.1.0 pd tso 443852055297916932
```

    system:  2023-08-27 20:33:41.687 +0200 CEST
    logic:   4

ここでは、`system:`で始まる行に物理タイムスタンプ、`logic:`で始まる行に論理タイムスタンプが表示されます。
