---
title: Keywords
summary: Keywords and Reserved Words
---

# キーワード {#keywords}

この記事では、TiDBのキーワード、予約語と非予約語の違い、およびクエリのためのすべてのキーワードを紹介します。

キーワードとは、SQL文で特別な意味を持つ単語のことで、`SELECT`、`UPDATE`、`DELETE`などがあります。そのうちのいくつかは、直接識別子として使用できるもので、**非予約キーワード**と呼ばれます。その他のものは、識別子として使用する前に特別な処理が必要なもので、**予約キーワード**と呼ばれます。ただし、特別な非予約キーワードもあり、これらは予約キーワードとして扱うことを推奨します。

予約キーワードを識別子として使用するには、バッククォート `` ` ``で囲む必要があります。

```sql
CREATE TABLE select (a INT);
```

    ERROR 1105 (HY000): line 0 column 19 near " (a INT)" (total length 27)

```sql
CREATE TABLE `select` (a INT);
```

    Query OK, 0 rows affected (0.09 sec)

非予約キーワードはバックティックを必要としません。例えば、`BEGIN`や`END`のようなキーワードは、次の文で識別子として正常に使用できます。

```sql
CREATE TABLE `select` (BEGIN int, END int);
```

    Query OK, 0 rows affected (0.09 sec)

特殊な場合では、予約語は `.` デリミターと一緒に使用される場合、バックティックは必要ありません。

```sql
CREATE TABLE test.select (BEGIN int, END int);
```

    Query OK, 0 rows affected (0.08 sec)

## キーワードリスト {#keyword-list}

以下のリストは、TiDBのキーワードを示しています。予約済みのキーワードは`(R)`でマークされています。[ウィンドウ関数](/functions-and-operators/window-functions.md)の予約済みキーワードは`(R-Window)`でマークされています。バックティック `` ` ``でエスケープする必要がある特別な予約されていないキーワードは、`(S)`でマークされています。

<TabsPanel letters="ABCDEFGHIJKLMNOPQRSTUVWXYZ" />

<a id="A" class="letter" href="#A">A</a>

- ACCOUNT
- ACTION
- ADD (R)
- ADMIN
- ADVISE
- AFTER
- AGAINST
- AGO
- ALGORITHM
- ALL (R)
- ALTER (R)
- ALWAYS
- ANALYZE (R)
- AND (R)
- ANY
- ARRAY (R)
- AS (R)
- ASC (R)
- ASCII
- ATTRIBUTE
- ATTRIBUTES
- AUTO\_ID\_CACHE
- AUTO\_INCREMENT
- AUTO\_RANDOM
- AUTO\_RANDOM\_BASE
- AVG
- AVG\_ROW\_LENGTH

<a id="B" class="letter" href="#B">B</a>

- BACKEND
- BACKUP
- BACKUPS
- BATCH
- BDR
- BEGIN
- BERNOULLI
- BETWEEN (R)
- BIGINT (R)
- BINARY (R)
- BINDING
- BINDINGS
- BINDING\_CACHE
- BINLOG
- BIT
- BLOB (R)
- BLOCK
- BOOL
- BOOLEAN
- BOTH (R)
- BTREE
- BUCKETS
- BUILTINS
- BY (R)
- BYTE

<a id="C" class="letter" href="#C">C</a>

- CACHE
- CALIBRATE
- CALL (R)
- CANCEL
- CAPTURE
- CARDINALITY
- CASCADE (R)
- CASCADED
- CASE (R)
- CAUSAL
- CHAIN
- CHANGE (R)
- CHAR (R)
- CHARACTER (R)
- CHARSET
- CHECK (R)
- CHECKPOINT
- CHECKSUM
- CIPHER
- CLEANUP
- CLIENT
- CLIENT\_ERRORS\_SUMMARY
- CLOSE
- CLUSTER
- CLUSTERED
- CMSKETCH
- COALESCE
- COLLATE (R)
- COLLATION
- COLUMN (R)
- COLUMN\_FORMAT
- COLUMN\_STATS\_USAGE
- COLUMNS
- COMMENT
- COMMIT
- COMMITTED
- COMPACT
- COMPRESSED
- COMPRESSION
- CONCURRENCY
- CONFIG
- CONNECTION
- CONSISTENCY
- CONSISTENT
- CONSTRAINT (R)
- CONTEXT
- CONTINUE (R)
- CONVERT (R)
- CORRELATION
- CPU
- CREATE (R)
- CROSS (R)
- CSV\_BACKSLASH\_ESCAPE
- CSV\_DELIMITER
- CSV\_HEADER
- CSV\_NOT\_NULL
- CSV\_NULL
- CSV\_SEPARATOR
- CSV\_TRIM\_LAST\_SEPARATORS
- CUME\_DIST (R-Window)
- CURRENT
- CURRENT\_DATE (R)
- CURRENT\_ROLE (R)
- CURRENT\_TIME (R)
- CURRENT\_TIMESTAMP (R)
- CURRENT\_USER (R)
- CURSOR (R)
- CYCLE

<a id="D" class="letter" href="#D">D</a>

- DATA
- DATABASE (R)
- DATABASES (R)
- DATE
- DATETIME
- DAY
- DAY\_HOUR (R)
- DAY\_MICROSECOND (R)
- DAY\_MINUTE (R)
- DAY\_SECOND (R)
- DDL
- DEALLOCATE
- DECIMAL (R)
- DECLARE
- DEFAULT (R)
- DEFINER
- DELAY\_KEY\_WRITE
- DELAYED (R)
- DELETE (R)
- DENSE\_RANK (R-Window)
- DEPENDENCY
- DEPTH
- DESC (R)
- DESCRIBE (R)
- DIGEST
- DIRECTORY
- DISABLE
- DISABLED
- DISCARD
- DISK
- DISTINCT (R)
- DISTINCTROW (R)
- DIV (R)
- DO
- DOUBLE (R)
- DRAINER
- DROP (R)
- DRY
- DUAL (R)
- DUPLICATE
- DYNAMIC

<a id="E" class="letter" href="#E">E</a>

- ELSE (R)

- ELSEIF (R)

- ENABLE

- ENABLED

- ENCLOSED (R)

- ENCRYPTION

- END

- ENFORCED

- ENGINE

- ENGINES

- ENUM

- ERROR

- ERRORS

- ESCAPE

- ESCAPED (R)

- EVENT

- EVENTS

- EVOLVE

- EXCEPT (R)

- EXCHANGE

- EXCLUSIVE

- EXECUTE

- EXISTS (R)

- EXIT (R)

- EXPANSION

- EXPIRE

- EXPLAIN (R)

- EXTENDED

- ELSE (R)

- ELSEIF (R)

- 有効にする

- 有効

- 囲まれた (R)

- 暗号化

- 終わり

- 強制

- エンジン

- エンジン

- 列挙型

- エラー

- エラー

- エスケープ

- エスケープ (R)

- イベント

- イベント

- 進化する

- 除外 (R)

- 交換

- 排他的

- 実行する

- 存在する (R)

- 終了 (R)

- 拡張

- 期限切れ

- 説明する (R)

- 拡張された

<a id="F" class="letter" href="#F">F</a>

- ログイン失敗試行回数
- 偽 (R)
- フォールト
- 取得 (R)
- フィールド
- ファイル
- 最初
- 最初の値 (R-Window)
- 固定
- 浮動小数点 (R)
- フラッシュ
- 後続
- ため (R)
- 強制 (R)
- 外部 (R)
- フォーマット
- 見つかった
- から (R)
- 完全
- 全文検索 (R)
- 関数

<a id="G" class="letter" href="#G">G</a>

- 一般
- 生成された (R)
- グローバル
- 許可する (R)
- 許可
- グループ (R)
- グループ (R-Window)

<a id="H" class="letter" href="#H">H</a>

- ハンドラ
- ハッシュ
- 持つ (R)
- ヘルプ
- 高い優先度 (R)
- ヒストグラム
- フライト中のヒストグラム
- 履歴
- ホスト
- 時間
- 時間マイクロ秒 (R)
- 時間分 (R)
- 時間秒 (R)

<a id="I" class="letter" href="#I">I</a>

- 識別された
- もし (R)
- 無視する (R)
- 類似 (R)
- インポート
- インポート
- に (R)
- 増加
- 増分
- インデックス (R)
- インデックス
- ファイル (R)
- 内部 (R)
- INOUT (R)
- 挿入 (R)
- 挿入方法
- インスタンス
- INT (R)
- INT1 (R)
- INT2 (R)
- INT3 (R)
- INT4 (R)
- INT8 (R)
- INTEGER (R)
- 交差 (R)
- インターバル (R)
- に (R)
- 不可視
- 呼び出し元
- 入出力
- IPC
- である (R)
- 分離
- 発行者
- 繰り返す (R)

<a id="J" class="letter" href="#J">J</a>

- ジョブ
- ジョブ
- 結合 (R)
- JSON

<a id="K" class="letter" href="#K">K</a>

- キー (R)
- キー (R)
- キーのブロックサイズ
- 終了する (R)

<a id="L" class="letter" href="#L">L</a>

- ラベル
- LAG (R-Window)
- 言語
- 最後
- 最後のバックアップ
- 最後の値 (R-Window)
- LASTVAL
- LEAD (R-Window)
- 先頭 (R)
- 離れる (R)
- 左 (R)
- より小さい
- レベル
- 類似 (R)
- 制限 (R)
- 線形 (R)
- 行 (R)
- リスト
- ロード (R)
- ローカル
- ローカルのみ
- ローカル時間 (R)
- ローカルタイムスタンプ (R)
- 場所
- ロック (R)
- ロックされた
- ログ
- 長い (R)
- 長いバイナリデータ (R)
- 長いテキストデータ (R)
- 低い優先度 (R)

<a id="M" class="letter" href="#M">M</a>

- マスター

- 一致 (R)

- 最大値 (R)

- 時間あたりの最大接続数

- 最大インデックス数

- 最大分

- 時間あたりの最大クエリ数

- 最大行数

- 時間あたりの最大更新数

- 最大ユーザー接続数

- MB

- 中型バイナリデータ (R)

- 中型整数 (R)

- 中型テキストデータ (R)

- メンバー

- メモリ

- マージ

- マイクロ秒

- 分

- 分間マイクロ秒 (R)

- 分間秒 (R)

- 最小値

- 最小行数

- MOD (R)

- モード

- 変更する

- 月

- 名前

- 国籍

- 自然 (R)

- NCHAR

- 決して

- 次の

- NEXTVAL

- いいえ

- NOCACHE

- NOCYCLE

- NODEGROUP

- NODE\_ID

- NODE\_STATE

- NOMAXVALUE

- NOMINVALUE

- NONCLUSTERED

- なし

- NOT (R)

- NOWAIT

- NO\_WRITE\_TO\_BINLOG (R)

- NTH\_VALUE (R-Window)

- NTILE (R-Window)

- NULL (R)

- NULLS

- 数値 (R)

- NVARCHAR

<a id="O" class="letter" href="#O">O</a>

- OF (R)
- OFF
- OFFSET
- OLTP\_READ\_ONLY
- OLTP\_READ\_WRITE
- OLTP\_WRITE\_ONLY
- ON (R)
- ON\_DUPLICATE
- オンライン
- ONLY
- OPEN
- OPTIMISTIC
- OPTIMIZE (R)
- OPTION (R)
- OPTIONAL
- OPTIONALLY (R)
- OR (R)
- ORDER (R)
- OUT (R)
- OUTER (R)
- OUTFILE (R)
- OVER (R-Window)

<a id="P" class="letter" href="#P">P</a>

- PACK\_KEYS
- PAGE
- PARSER
- 部分的
- PARTITION (R)
- PARTITIONING
- PARTITIONS
- パスワード
- PASSWORD\_LOCK\_TIME
- PAUSE
- パーセント
- PERCENT\_RANK (R-Window)
- PER\_DB
- PER\_TABLE
- PESSIMISTIC
- PLACEMENT (S)
- PLUGINS
- POINT
- ポリシー
- PRECEDING
- 精度 (R)
- PREPARE
- PRESERVE
- PRE\_SPLIT\_REGIONS
- PRIMARY (R)
- PRIVILEGES
- PROCEDURE (R)
- PROCESS
- PROCESSLIST
- PROFILE
- PROFILES
- PROXY
- PUMP
- PURGE

<a id="Q" class="letter" href="#Q">Q</a>

- QUARTER
- QUERIES
- QUERY
- QUICK

<a id="R" class="letter" href="#R">R</a>

- RANGE (R)

- RANK (R-Window)

- RATE\_LIMIT

- READ (R)

- REAL (R)

- REBUILD

- RECOVER

- RECURSIVE (R)

- REDUNDANT

- REFERENCES (R)

- REGEXP (R)

- REGION

- REGIONS

- RELEASE (R)

- RELOAD

- REMOVE

- RENAME (R)

- REORGANIZE

- REPAIR

- REPEAT (R)

- REPEATABLE

- REPLACE (R)

- REPLICA

- REPLICAS

- REPLICATION

- REQUIRE (R)

- REQUIRED

- RESET

- RESOURCE

- RESPECT

- RESTART

- RESTORE

- RESTORES

- RESTRICT (R)

- RESUME

- REUSE

- REVERSE

- REVOKE (R)

- RIGHT (R)

- RLIKE (R)

- ROLE

- ROLLBACK

- ROUTINE

- ROW (R)

- ROW\_COUNT

- ROW\_FORMAT

- ROW\_NUMBER (R-Window)

- ROWS (R-Window)

- RTREE

- RUN

- サンプルレート

- サンプル数

- SAN

- セーブポイント

- 秒

- SECOND\_MICROSECOND (R)

- セカンダリ

- SECONDARY\_ENGINE

- SECONDARY\_LOAD

- SECONDARY\_UNLOAD

- セキュリティ

- 選択 (R)

- SEND\_CREDENTIALS\_TO\_TIKV

- セパレーター

- シーケンス

- シリアル

- シリアライズ可能

- セッション

- SESSION\_STATES

- 設定 (R)

- SETVAL

- シャード\_ROW\_ID\_BITS

- 共有

- 共有済み

- 表示 (R)

- シャットダウン

- 符号付き

- シンプル

- スキップ

- スキーマファイルをスキップ

- スレーブ

- 遅い

- SMALLINT (R)

- スナップショット

- いくつか

- ソース

- 空間 (R)

- 分割

- SQL (R)

- SQL\_BIG\_RESULT (R)

- SQL\_BUFFER\_RESULT

- SQL\_CACHE

- SQL\_CALC\_FOUND\_ROWS (R)

- SQL\_NO\_CACHE

- SQL\_SMALL\_RESULT (R)

- SQL\_TSI\_DAY

- SQL\_TSI\_HOUR

- SQL\_TSI\_MINUTE

- SQL\_TSI\_MONTH

- SQL\_TSI\_QUARTER

- SQL\_TSI\_SECOND

- SQL\_TSI\_WEEK

- SQL\_TSI\_YEAR

- SQLEXCEPTION (R)

- SQLSTATE (R)

- SQLWARNING (R)

- SSL (R)

- 開始

- 開始 (R)

- 統計

- 統計

- 統計\_自動\_再計算

- 統計\_バケツ

- 統計\_列\_選択

- 統計\_列\_リスト

- 統計\_拡張 (R)

- 統計\_健全

- 統計\_ヒストグラム

- 統計\_ロック済み

- 統計\_メタ

- 統計\_オプション

- 統計\_永続

- 統計\_サンプル\_ページ

- 統計\_サンプル\_レート

- 統計\_トップN

- ステータス

- ストレージ

- 格納済み (R)

- ストレート\_ジョイン (R)

- 厳密なフォーマット

- 件名

- サブパーティション

- サブパーティション

- スーパー

- スワップ

- スイッチ

- システム

- システム\_タイム

<a id="T" class="letter" href="#T">T</a>

- テーブル (R)
- テーブル
- TABLESAMPLE (R)
- テーブルスペース
- テーブル\_チェックサム
- テレメトリー
- テレメトリー\_ID
- 一時的
- 一時テーブル
- 終了 (R)
- テキスト
- よりも
- その後 (R)
- TIDB
- TiDB\_CURRENT\_TSO (R)
- TIFLASH
- TIKV\_IMPORTER
- 時間
- タイムスタンプ
- TINYBLOB (R)
- TINYINT (R)
- TINYTEXT (R)
- TO (R)
- TOKEN\_ISSUER
- TOPN
- TPCC
- トレース
- 伝統的
- 後ろに (R)
- トランザクション
- トリガー (R)
- トリガー
- 真 (R)
- 切り捨て
- TSO
- TTL
- TTL\_ENABLE
- TTL\_JOB\_INTERVAL
- タイプ

<a id="U" class="letter" href="#U">U</a>

- UNBOUNDED
- UNCOMMITTED
- UNDEFINED
- UNICODE
- UNION (R)
- UNIQUE (R)
- UNKNOWN
- UNLOCK (R)
- UNSIGNED (R)
- UNTIL (R)
- UPDATE (R)
- USAGE (R)
- USE (R)
- ユーザー
- USING (R)
- UTC\_DATE (R)
- UTC\_TIME (R)
- UTC\_TIMESTAMP (R)

<a id="V" class="letter" href="#V">V</a>

- 検証
- 値
- VALUES (R)
- VARBINARY (R)
- VARCHAR (R)
- VARCHARACTER (R)
- 変数
- VARYING (R)
- ビュー
- VIRTUAL (R)
- 表示可能

<a id="W" class="letter" href="#W">W</a>

- 待つ
- 警告
- 週
- WEIGHT\_STRING
- WHEN (R)
- WHERE (R)
- WHILE (R)
- 幅
- ウィンドウ (R-Window)
- WITH (R)
- WITHOUT
- ワークロード
- 書き込み (R)

<a id="X" class="letter" href="#X">X</a>

- X509
- XOR (R)

<a id="Y" class="letter" href="#Y">Y</a>

- 年
- 年\_月 (R)

<a id="Z" class="letter" href="#Z">Z</a>

- ゼロ埋め (R)
