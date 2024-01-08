---
title: System Variables
summary: Use system variables to optimize performance or alter running behavior.
---

# システム変数 {#system-variables}

TiDBシステム変数は、設定が`SESSION`または`GLOBAL`スコープで適用される点で、MySQLと類似しています。

- `SESSION`スコープでの変更は、現在のセッションのみに影響します。
- `GLOBAL`スコープでの変更はすぐに適用されます。この変数が`SESSION`スコープにも適用される場合、すべてのセッション（あなたのセッションも含む）は現在のセッション値を引き続き使用します。
- 変更は[`SET`ステートメント](/sql-statements/sql-statement-set-variable.md)を使用して行われます。

```sql
# These two identical statements change a session variable
SET tidb_distsql_scan_concurrency = 10;
SET SESSION tidb_distsql_scan_concurrency = 10;

# These two identical statements change a global variable
SET @@global.tidb_distsql_scan_concurrency = 10;
SET GLOBAL tidb_distsql_scan_concurrency = 10;
```

> **Note:**
>
> TiDBクラスターにはいくつかの`GLOBAL`変数があります。このドキュメントのいくつかの変数には、`クラスターに永続化`設定があり、`Yes`または`No`に構成できます。

> - `クラスターに永続化: Yes`設定の変数の場合、グローバル変数が変更されると、すべてのTiDBサーバーに通知が送信され、システム変数キャッシュが更新されます。追加のTiDBサーバーを追加するか、既存のTiDBサーバーを再起動すると、永続化された構成値が自動的に使用されます。
> - `クラスターに永続化: No`設定の変数の場合、変更は接続されているローカルTiDBインスタンスにのみ適用されます。設定した値を保持するには、`tidb.toml`構成ファイルで変数を指定する必要があります。

> さらに、TiDBはいくつかのMySQL変数を読み取り可能および設定可能として示します。これは互換性のために必要です。アプリケーションとコネクタの両方がMySQL変数を読み取ることが一般的です。たとえば、JDBCコネクタはクエリキャッシュ設定を読み取りおよび設定しますが、その動作に依存しません。

> **Note:**
>
> 大きな値が常により良いパフォーマンスをもたらすわけではありません。また、ステートメントを実行している同時接続数を考慮することが重要です。なぜなら、ほとんどの設定は各接続に適用されるからです。

> 安全な値を決定する際に変数の単位を考慮してください：
>
> - スレッドの場合、安全な値は通常CPUコア数までです。
> - バイトの場合、安全な値は通常システムメモリ量より少ないです。
> - 時間の場合、単位が秒またはミリ秒であることに注意してください。

> 同じ単位を使用する変数は、同じリソースのセットを競合する可能性があります。

## Variable Reference {#variable-reference}

### allow\_auto\_random\_explicit\_insert <span class="version-mark">v4.0.3で新規</span> {#allow-auto-random-explicit-insert-span-class-version-mark-new-in-v4-0-3-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: Yes
- タイプ: ブール値
- デフォルト値: `OFF`
- `AUTO_RANDOM`属性を持つ列の値を`INSERT`ステートメントで明示的に指定するかどうかを決定します。

### authentication\_ldap\_sasl\_auth\_method\_name <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-auth-method-name-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 列挙型
- デフォルト値: `SCRAM-SHA-1`
- 可能な値: `SCRAM-SHA-1`、`SCRAM-SHA-256`、`GSSAPI`
- LDAP SASL認証の場合、この変数は認証メソッド名を指定します。

### authentication\_ldap\_sasl\_bind\_base\_dn <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-bind-base-dn-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証の場合、この変数は検索ツリー内の検索範囲を制限します。`AS ...`句なしでユーザーが作成されると、TiDBはユーザー名に応じてLDAPサーバー内の`dn`を自動的に検索します。

### authentication\_ldap\_sasl\_bind\_root\_dn <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-bind-root-dn-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証の場合、この変数はLDAPサーバーにログインするために使用される`dn`を指定します。

### authentication\_ldap\_sasl\_bind\_root\_pwd <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-bind-root-pwd-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証の場合、この変数はLDAPサーバーにログインするために使用されるパスワードを指定します。

### authentication\_ldap\_sasl\_ca\_path <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-ca-path-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証の場合、この変数はStartTLS接続用の証明機関ファイルの絶対パスを指定します。

### authentication\_ldap\_sasl\_init\_pool\_size <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-init-pool-size-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 整数
- デフォルト値: `10`
- 範囲: `[1, 32767]`
- LDAP SASL認証の場合、この変数はLDAPサーバーへの接続プール内の初期接続を指定します。

### authentication\_ldap\_sasl\_max\_pool\_size <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-max-pool-size-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 整数
- デフォルト値: `1000`
- 範囲: `[1, 32767]`
- LDAP SASL認証の場合、この変数はLDAPサーバーへの接続プール内の最大接続数を指定します。

### authentication\_ldap\_sasl\_server\_host <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-server-host-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAP SASL認証の場合、この変数はLDAPサーバーのホスト名またはIPアドレスを指定します。

### authentication\_ldap\_sasl\_server\_port <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-server-port-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 整数
- デフォルト値: `389`
- 範囲: `[1, 65535]`
- LDAP SASL認証の場合、この変数はLDAPサーバーのTCP/IPポート番号を指定します。

### authentication\_ldap\_sasl\_tls <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-sasl-tls-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: ブール値
- デフォルト値: `OFF`
- LDAP SASL認証の場合、この変数はプラグインによるLDAPサーバーへの接続がStartTLSで保護されているかどうかを制御します。

### authentication\_ldap\_simple\_auth\_method\_name <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-auth-method-name-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 列挙型
- デフォルト値: `SIMPLE`
- 可能な値: `SIMPLE`
- LDAPシンプル認証の場合、この変数は認証メソッド名を指定します。サポートされる値は`SIMPLE`のみです。

### authentication\_ldap\_simple\_bind\_base\_dn <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-bind-base-dn-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAPシンプル認証の場合、この変数は検索ツリー内の検索範囲を制限します。`AS ...`句なしでユーザーが作成されると、TiDBはユーザー名に応じてLDAPサーバー内の`dn`を自動的に検索します。

### authentication\_ldap\_simple\_bind\_root\_dn <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-bind-root-dn-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAPシンプル認証の場合、この変数はLDAPサーバーにログインするために使用される`dn`を指定します。

### authentication\_ldap\_simple\_bind\_root\_pwd <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-bind-root-pwd-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAPシンプル認証の場合、この変数はLDAPサーバーにログインするために使用されるパスワードを指定します。

### authentication\_ldap\_simple\_ca\_path <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-ca-path-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAPシンプル認証の場合、この変数はStartTLS接続用の証明機関ファイルの絶対パスを指定します。

### authentication\_ldap\_simple\_init\_pool\_size <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-init-pool-size-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 整数
- デフォルト値: `10`
- 範囲: `[1, 32767]`
- LDAPシンプル認証の場合、この変数はLDAPサーバーへの接続プール内の初期接続を指定します。

### authentication\_ldap\_simple\_max\_pool\_size <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-max-pool-size-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 整数
- デフォルト値: `1000`
- 範囲: `[1, 32767]`
- LDAPシンプル認証の場合、この変数はLDAPサーバーへの接続プール内の最大接続数を指定します。

### authentication\_ldap\_simple\_server\_host <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-server-host-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 文字列
- デフォルト値: ""
- LDAPシンプル認証の場合、この変数はLDAPサーバーのホスト名またはIPアドレスを指定します。

### authentication\_ldap\_simple\_server\_port <span class="version-mark">v7.1.0で新規</span> {#authentication-ldap-simple-server-port-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: Yes
- タイプ: 整数
- デフォルト値: `389`
- 範囲: `[1, 65535]`
- LDAPシンプル認証の場合、この変数はLDAPサーバーのTCP/IPポート番号を指定します。

### authentication\_ldap\_simple\_tls <span class="version-mark">v7.1.0 で新規追加</span> {#authentication-ldap-simple-tls-span-class-version-mark-new-in-v7-1-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- LDAPシンプル認証の場合、この変数はプラグインによるLDAPサーバーへの接続がStartTLSで保護されるかどうかを制御します。

### auto\_increment\_increment {#auto-increment-increment}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[1, 65535]`
- `AUTO_INCREMENT` 値のステップサイズを列に割り当てるための制御。通常、`auto_increment_offset` と組み合わせて使用されます。

### auto\_increment\_offset {#auto-increment-offset}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[1, 65535]`
- `AUTO_INCREMENT` 値の列に割り当てる初期オフセットを制御します。この設定は通常、`auto_increment_increment` と組み合わせて使用されます。例:

```sql
mysql> CREATE TABLE t1 (a int not null primary key auto_increment);
Query OK, 0 rows affected (0.10 sec)

mysql> set auto_increment_offset=1;
Query OK, 0 rows affected (0.00 sec)

mysql> set auto_increment_increment=3;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 VALUES (),(),(),();
Query OK, 4 rows affected (0.04 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+
| a  |
+----+
|  1 |
|  4 |
|  7 |
| 10 |
+----+
4 rows in set (0.00 sec)
```

### autocommit {#autocommit}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- Controls whether statements should automatically commit when not in an explicit transaction. See [Transaction Overview](/transaction-overview.md#autocommit) for more information.

### block\_encryption\_mode {#block-encryption-mode}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Enumeration
- Default value: `aes-128-ecb`
- Value options: `aes-128-ecb`, `aes-192-ecb`, `aes-256-ecb`, `aes-128-cbc`, `aes-192-cbc`, `aes-256-cbc`, `aes-128-ofb`, `aes-192-ofb`, `aes-256-ofb`, `aes-128-cfb`, `aes-192-cfb`, `aes-256-cfb`
- This variable sets the encryption mode for the built-in functions `AES_ENCRYPT()` and `AES_DECRYPT()`.

### character\_set\_client {#character-set-client}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `utf8mb4`
- The character set for data sent from the client. See [Character Set and Collation](/character-set-and-collation.md) for details on the use of character sets and collations in TiDB. It is recommended to use [`SET NAMES`](/sql-statements/sql-statement-set-names.md) to change the character set when needed.

### character\_set\_connection {#character-set-connection}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `utf8mb4`
- The character set for string literals that do not have a specified character set.

### character\_set\_database {#character-set-database}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `utf8mb4`
- This variable indicates the character set of the default database in use. **It is NOT recommended to set this variable**. When a new default database is selected, the server changes the variable value.

### character\_set\_results {#character-set-results}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `utf8mb4`
- The character set that is used when data is sent to the client.

### character\_set\_server {#character-set-server}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `utf8mb4`
- The default character set for the server.

### collation\_connection {#collation-connection}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `utf8mb4_bin`
- This variable indicates the collation used in the current connection. It is consistent with the MySQL variable `collation_connection`.

### collation\_database {#collation-database}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `utf8mb4_bin`
- This variable indicates the default collation of the database in use. **It is NOT recommended to set this variable**. When a new database is selected, TiDB changes this variable value.

### collation\_server {#collation-server}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `utf8mb4_bin`
- The default collation used when the database is created.

### cte\_max\_recursion\_depth {#cte-max-recursion-depth}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `1000`
- Range: `[0, 4294967295]`
- Controls the maximum recursion depth in Common Table Expressions.

### datadir {#datadir}

> **Note:**
>
> This variable is not supported on [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless).

<CustomContent platform="tidb">

- Scope: NONE
- Default value: it depends on the component and the deployment method.
  - `/tmp/tidb`: when you set `"unistore"` for [`--store`](/command-line-flags-for-tidb-configuration.md#--store) or if you don't set `--store`.
  - `${pd-ip}:${pd-port}`: when you use TiKV, which is the default storage engine for TiUP and TiDB Operator for Kubernetes deployments.
- This variable indicates the location where data is stored. This location can be a local path `/tmp/tidb`, or point to a PD server if the data is stored on TiKV. A value in the format of `${pd-ip}:${pd-port}` indicates the PD server that TiDB connects to on startup.

</CustomContent>

<CustomContent platform="tidb-cloud">

- Scope: NONE
- Default value: it depends on the component and the deployment method.
  - `/tmp/tidb`: when you set `"unistore"` for [`--store`](https://docs.pingcap.com/tidb/stable/command-line-flags-for-tidb-configuration#--store) or if you don't set `--store`.
  - `${pd-ip}:${pd-port}`: when you use TiKV, which is the default storage engine for TiUP and TiDB Operator for Kubernetes deployments.
- This variable indicates the location where data is stored. This location can be a local path `/tmp/tidb`, or point to a PD server if the data is stored on TiKV. A value in the format of `${pd-ip}:${pd-port}` indicates the PD server that TiDB connects to on startup.

</CustomContent>

### ddl\_slow\_threshold {#ddl-slow-threshold}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> This TiDB variable is not applicable to TiDB Cloud.

</CustomContent>

- Scope: GLOBAL
- Persists to cluster: No, only applicable to the current TiDB instance that you are connecting to.
- Type: Integer
- Default value: `300`
- Range: `[0, 2147483647]`
- Unit: Milliseconds
- Log DDL operations whose execution time exceeds the threshold value.

### default\_authentication\_plugin {#default-authentication-plugin}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Enumeration
- Default value: `mysql_native_password`
- Possible values: `mysql_native_password`, `caching_sha2_password`, `tidb_sm3_password`, `tidb_auth_token`, `authentication_ldap_sasl`, and `authentication_ldap_simple`.
- The `tidb_auth_token` authentication method is used only for the internal operation of TiDB Cloud. **DO NOT** set the variable to this value.
- This variable sets the authentication method that the server advertises when the server-client connection is being established.
- To authenticate using the `tidb_sm3_password` method, you can connect to TiDB using [TiDB-JDBC](https://github.com/pingcap/mysql-connector-j/tree/release/8.0-sm3).

<CustomContent platform="tidb">

For more possible values of this variable, see [Authentication plugin status](/security-compatibility-with-mysql.md#authentication-plugin-status).

</CustomContent>

### default\_password\_lifetime <span class="version-mark">New in v6.5.0</span> {#default-password-lifetime-span-class-version-mark-new-in-v6-5-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `0`
- Range: `[0, 65535]`
- Sets the global policy for automatic password expiration. The default value `0` indicates that the password never expires. If this system variable is set to a positive integer `N`, it means that the password lifetime is `N` days, and you must change your password within `N` days.

### default\_week\_format {#default-week-format}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `0`
- Range: `[0, 7]`
- Sets the week format used by the `WEEK()` function.

### disconnect\_on\_expired\_password <span class="version-mark">New in v6.5.0</span> {#disconnect-on-expired-password-span-class-version-mark-new-in-v6-5-0-span}

- Scope: GLOBAL
- Type: Boolean
- Default value: `ON`
- This variable is read-only. It indicates whether TiDB disconnects the client connection when the password is expired. If the variable is set to `ON`, the client connection is disconnected when the password is expired. If the variable is set to `OFF`, the client connection is restricted to the "sandbox mode" and the user can only execute the password reset operation.

<CustomContent platform="tidb">

- If you need to change the behavior of the client connection for the expired password, modify the [`security.disconnect-on-expired-password`](/tidb-configuration-file.md#disconnect-on-expired-password-new-in-v650) configuration item in the configuration file.

</CustomContent>

<CustomContent platform="tidb-cloud">

- If you need to change the default behavior of the client connection for the expired password, contact [TiDB Cloud Support](/tidb-cloud/tidb-cloud-support.md).

</CustomContent>

### error\_count {#error-count}

- Scope: SESSION
- Type: Integer
- Default value: `0`
- A read-only variable that indicates the number of errors that resulted from the last statement that generated messages.

### foreign\_key\_checks {#foreign-key-checks}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: Before v6.6.0, the default value is `OFF`. Starting from v6.6.0, the default value is `ON`.
- This variable controls whether to enable foreign key constraint checking.

### group\_concat\_max\_len {#group-concat-max-len}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `1024`
- Range: `[4, 18446744073709551615]`
- The maximum buffer size for items in the `GROUP_CONCAT()` function.

### have\_openssl {#have-openssl}

- Scope: NONE
- Type: Boolean
- Default value: `DISABLED`
- A read-only variable for MySQL compatibility. Set to `YES` by the server when the server has TLS enabled.

### have\_ssl {#have-ssl}

- Scope: NONE
- Type: Boolean
- Default value: `DISABLED`
- A read-only variable for MySQL compatibility. Set to `YES` by the server when the server has TLS enabled.

### hostname {#hostname}

- Scope: NONE
- Default value: (system hostname)
- The hostname of the TiDB server as a read-only variable.

### identity <span class="version-mark">New in v5.3.0</span> {#identity-span-class-version-mark-new-in-v5-3-0-span}

This variable is an alias for [`last_insert_id`](#last_insert_id).

### init\_connect {#init-connect}

- Scope: GLOBAL
- Persists to cluster: Yes
- Default value: ""
- The `init_connect` feature permits a SQL statement to be automatically executed when you first connect to a TiDB server. If you have the `CONNECTION_ADMIN` or `SUPER` privileges, this `init_connect` statement will not be executed. If the `init_connect` statement results in an error, your user connection will be terminated.

### innodb\_lock\_wait\_timeout {#innodb-lock-wait-timeout}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `50`
- Range: `[1, 3600]`
- Unit: Seconds
- The lock wait timeout for pessimistic transactions (default).

### interactive\_timeout {#interactive-timeout}

> **Note:**
>
> This variable is read-only for [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless).

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `28800`
- Range: `[1, 31536000]`
- Unit: Seconds
- This variable represents the idle timeout of the interactive user session. Interactive user session refers to the session established by calling [`mysql_real_connect()`](https://dev.mysql.com/doc/c-api/5.7/en/mysql-real-connect.html) API using the `CLIENT_INTERACTIVE` option (for example, MySQL Shell and MySQL Client). This variable is fully compatible with MySQL.

### last\_insert\_id {#last-insert-id}

- Scope: SESSION
- Type: Integer
- Default value: `0`
- Range: `[0, 9223372036854775807]`
- This variable returns the last `AUTO_INCREMENT` or `AUTO_RANDOM` value generated by an insert statement.
- The value of `last_insert_id` is the same as the value returned by the function `LAST_INSERT_ID()`.

### last\_plan\_from\_binding <span class="version-mark">New in v4.0</span> {#last-plan-from-binding-span-class-version-mark-new-in-v4-0-span}

- Scope: SESSION
- Type: Boolean
- Default value: `OFF`
- This variable is used to show whether the execution plan used in the previous statement was influenced by a [plan binding](/sql-plan-management.md)

### last\_plan\_from\_cache <span class="version-mark">New in v4.0</span> {#last-plan-from-cache-span-class-version-mark-new-in-v4-0-span}

- Scope: SESSION
- Type: Boolean
- Default value: `OFF`
- This variable is used to show whether the execution plan used in the previous `execute` statement is taken directly from the plan cache.

### last\_sql\_use\_alloc <span class="version-mark">New in v6.4.0</span> {#last-sql-use-alloc-span-class-version-mark-new-in-v6-4-0-span}

- Scope: SESSION
- Default value: `OFF`
- This variable is read-only. It is used to show whether the previous statement uses a cached chunk object (chunk allocation).

### license {#license}

- Scope: NONE
- Default value: `Apache License 2.0`
- This variable indicates the license of your TiDB server installation.

### log\_bin {#log-bin}

- Scope: NONE
- Type: Boolean
- Default value: `OFF`
- This variable indicates whether [TiDB Binlog](https://docs.pingcap.com/tidb/stable/tidb-binlog-overview) is used.

### max\_allowed\_packet <span class="version-mark">New in v6.1.0</span> {#max-allowed-packet-span-class-version-mark-new-in-v6-1-0-span}

> **Note:**
>
> This variable is read-only for [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless).

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `67108864`
- Range: `[1024, 1073741824]`
- The value should be an integer multiple of 1024. If the value is not divisible by 1024, a warning will be prompted and the value will be rounded down. For example, when the value is set to 1025, the actual value in TiDB is 1024.
- The maximum packet size allowed by the server and the client in one transmission of packets.
- In the `SESSION` scope, this variable is read-only.
- This variable is compatible with MySQL.

### password\_history <span class="version-mark">New in v6.5.0</span> {#password-history-span-class-version-mark-new-in-v6-5-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `0`
- Range: `[0, 4294967295]`
- This variable is used to establish a password reuse policy that allows TiDB to limit password reuse based on the number of password changes. The default value `0` means disabling the password reuse policy based on the number of password changes. When this variable is set to a positive integer `N`, the reuse of the last `N` passwords is not allowed.

### mpp\_exchange\_compression\_mode <span class="version-mark">New in v6.6.0</span> {#mpp-exchange-compression-mode-span-class-version-mark-new-in-v6-6-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `UNSPECIFIED`
- Value options: `NONE`, `FAST`, `HIGH_COMPRESSION`, `UNSPECIFIED`
- This variable is used to specify the data compression mode of the MPP Exchange operator. This variable takes effect when TiDB selects the MPP execution plan with the version number `1`. The meanings of the variable values are as follows:
  - `UNSPECIFIED`: means unspecified. TiDB will automatically select the compression mode. Currently, TiDB automatically selects the `FAST` mode.
  - `NONE`: no data compression is used.
  - `FAST`: fast mode. The overall performance is good and the compression ratio is less than `HIGH_COMPRESSION`.
  - `HIGH_COMPRESSION`: the high compression ratio mode.

### mpp\_version <span class="version-mark">New in v6.6.0</span> {#mpp-version-span-class-version-mark-new-in-v6-6-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `UNSPECIFIED`
- Value options: `UNSPECIFIED`, `0`, `1`
- This variable is used to specify different versions of the MPP execution plan. After a version is specified, TiDB selects the specified version of the MPP execution plan. The meanings of the variable values are as follows:
  - `UNSPECIFIED`: means unspecified. TiDB automatically selects the latest version `1`.
  - `0`: compatible with all TiDB cluster versions. Features with the MPP version greater than `0` do not take effect in this mode.
  - `1`: new in v6.6.0, used to enable data exchange with compression on TiFlash. For details, see [MPP version and exchange data compression](/explain-mpp.md#mpp-version-and-exchange-data-compression).

### password\_reuse\_interval <span class="version-mark">New in v6.5.0</span> {#password-reuse-interval-span-class-version-mark-new-in-v6-5-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `0`
- Range: `[0, 4294967295]`
- This variable is used to establish a password reuse policy that allows TiDB to limit password reuse based on time elapsed. The default value `0` means disabling the password reuse policy based on time elapsed. When this variable is set to a positive integer `N`, the reuse of any password used in the last `N` days is not allowed.

### max\_connections {#max-connections}

- Scope: GLOBAL
- Persists to cluster: No, only applicable to the current TiDB instance that you are connecting to.
- Type: Integer
- Default value: `0`
- Range: `[0, 100000]`
- The maximum number of concurrent connections permitted for a single TiDB instance. This variable can be used for resources control.
- The default value `0` means no limit. When the value of this variable is larger than `0`, and the number of connections reaches the value, the TiDB server rejects new connections from clients.

### max\_execution\_time {#max-execution-time}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `0`
- Range: `[0, 2147483647]`
- Unit: Milliseconds
- The maximum execution time of a statement. The default value is unlimited (zero).

> **Note:**
>
> The `max_execution_time` system variable currently only controls the maximum execution time for read-only SQL statements. The precision of the timeout value is roughly 100ms. This means the statement might not be terminated in accurate milliseconds as you specify.

<CustomContent platform="tidb">

For a SQL statement with the [`MAX_EXECUTION_TIME`](/optimizer-hints.md#max_execution_timen) hint, the maximum execution time of this statement is limited by the hint instead of this variable. The hint can also be used with SQL bindings as described [in the SQL FAQ](/faq/sql-faq.md#how-to-prevent-the-execution-of-a-particular-sql-statement).

</CustomContent>

<CustomContent platform="tidb-cloud">

For a SQL statement with the [`MAX_EXECUTION_TIME`](/optimizer-hints.md#max_execution_timen) hint, the maximum execution time of this statement is limited by the hint instead of this variable. The hint can also be used with SQL bindings as described [in the SQL FAQ](https://docs.pingcap.com/tidb/stable/sql-faq).

</CustomContent>

### max\_prepared\_stmt\_count {#max-prepared-stmt-count}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `-1`
- Range: `[-1, 1048576]`
- Specifies the maximum number of [`PREPARE`](/sql-statements/sql-statement-prepare.md) statements in the current TiDB instance.
- The value of `-1` means no limit on the maximum number of `PREPARE` statements in the current TiDB instance.
- If you set the variable to a value that exceeds the upper limit `1048576`, `1048576` is used instead:

```sql
mysql> SET GLOBAL max_prepared_stmt_count = 1048577;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> SHOW WARNINGS;
+---------+------+--------------------------------------------------------------+
| Level   | Code | Message                                                      |
+---------+------+--------------------------------------------------------------+
| Warning | 1292 | Truncated incorrect max_prepared_stmt_count value: '1048577' |
+---------+------+--------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'max_prepared_stmt_count';
+-------------------------+---------+
| Variable_name           | Value   |
+-------------------------+---------+
| max_prepared_stmt_count | 1048576 |
+-------------------------+---------+
1 row in set (0.00 sec)
```

### plugin\_dir {#plugin-dir}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)ではサポートされていません。

- Scope: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- デフォルト値: ""
- コマンドラインフラグで指定されたプラグインをロードするディレクトリを示します。

### plugin\_load {#plugin-load}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)ではサポートされていません。

- Scope: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- デフォルト値: ""
- TiDBの起動時にロードするプラグインを示します。これらのプラグインはコマンドラインフラグで指定され、カンマで区切られます。

### port {#port}

- Scope: NONE
- タイプ: 整数
- デフォルト値: `4000`
- 範囲: `[0, 65535]`
- `tidb-server`がMySQLプロトコルを使用してリッスンしているポート。

### rand\_seed1 {#rand-seed1}

- Scope: SESSION
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- この変数は`RAND()` SQL関数で使用されるランダム値ジェネレーターのシードとして使用されます。
- この変数の動作はMySQL互換です。

### rand\_seed2 {#rand-seed2}

- Scope: SESSION
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- この変数は`RAND()` SQL関数で使用されるランダム値ジェネレーターのシードとして使用されます。
- この変数の動作はMySQL互換です。

### require\_secure\_transport <span class="version-mark">v6.1.0で新規</span> {#require-secure-transport-span-class-version-mark-new-in-v6-1-0-span}

> **Note:**
>
> 現在、この変数は[TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated)ではサポートされていません。TiDB Dedicatedクラスターでこの変数を有効にしないでください。そうしないと、SQLクライアント接続の失敗が発生する可能性があります。この制限は一時的な制御措置であり、将来のリリースで解決されます。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `TiDB Self-Hosted`および[TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated)では`OFF`、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では`ON`

<CustomContent platform="tidb">

- この変数は、TiDBへのすべての接続がローカルソケットまたはTLSを使用していることを保証します。詳細については、[TiDBクライアントとサーバー間のTLSを有効にする](/enable-tls-between-clients-and-servers.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、TiDBへのすべての接続がローカルソケットまたはTLSを使用していることを保証します。

</CustomContent>

- この変数を`ON`に設定すると、TLSが有効になっているセッションからTiDBに接続する必要があります。これにより、TLSが正しく構成されていない場合のロックアウトシナリオを防ぎます。
- この設定は以前は`tidb.toml`オプション(`security.require-secure-transport`)でしたが、TiDB v6.1.0からシステム変数に変更されました。
- v7.1.2以降のv7.1パッチバージョンでは、セキュリティ強化モード（SEM）が有効になっている場合、この変数を`ON`に設定することは禁止されており、ユーザーに潜在的な接続の問題を避けるためです。

### skip\_name\_resolve <span class="version-mark">v5.2.0で新規</span> {#skip-name-resolve-span-class-version-mark-new-in-v5-2-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`tidb-server`インスタンスが接続ハンドシェイクの一部としてホスト名を解決するかどうかを制御します。
- DNSが信頼できない場合、このオプションを有効にしてネットワークパフォーマンスを向上させることができます。

> **Note:**
>
> `skip_name_resolve=ON`の場合、アイデンティティにホスト名が含まれるユーザーはサーバーにログインできなくなります。例：
>
> ```sql
> CREATE USER 'appuser'@'apphost' IDENTIFIED BY 'app-password';
> ```
>
> この例では、`apphost`をIPアドレスまたはワイルドカード（`%`）に置き換えることをお勧めします。

### socket {#socket}

- Scope: NONE
- デフォルト値: ""
- `tidb-server`がMySQLプロトコルを使用してリッスンしているローカルUNIXソケットファイル。

### sql\_log\_bin {#sql-log-bin}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)では読み取り専用です。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- [TiDB Binlog](https://docs.pingcap.com/tidb/stable/tidb-binlog-overview)に変更を書き込むかどうかを示します。

> **Note:**
>
> この変数をグローバル変数として設定することは推奨されません。なぜなら、将来のTiDBのバージョンでは、この変数をセッション変数としてのみ設定できる場合があるからです。

### sql\_mode {#sql-mode}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`
- この変数は、多くのMySQL互換の動作を制御します。詳細については、[SQLモード](/sql-mode.md)を参照してください。

### sql\_require\_primary\_key <span class="version-mark">v6.3.0で新規</span> {#sql-require-primary-key-span-class-version-mark-new-in-v6-3-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、テーブルにプライマリキーがあることを強制するかどうかを制御します。この変数が有効になった後、プライマリキーのないテーブルを作成または変更しようとするとエラーが発生します。
- この機能は、MySQL 8.0の同様に名前のついた[`sql_require_primary_key`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sql_require_primary_key)に基づいています。
- TiCDCを使用する場合は、この変数を有効にすることを強くお勧めします。これは、MySQLシンクへの変更のレプリケーションにはテーブルにプライマリキーが必要です。

### sql\_select\_limit <span class="version-mark">v4.0.2で新規</span> {#sql-select-limit-span-class-version-mark-new-in-v4-0-2-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `18446744073709551615`
- 範囲: `[0, 18446744073709551615]`
- 単位: 行
- `SELECT`ステートメントによって返される行の最大数。

### ssl\_ca {#ssl-ca}

<CustomContent platform="tidb">

- Scope: NONE
- デフォルト値: ""
- 証明機関ファイルの場所（ある場合）。この変数の値は、TiDB構成項目[`ssl-ca`](/tidb-configuration-file.md#ssl-ca)によって定義されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- Scope: NONE
- デフォルト値: ""
- 証明機関ファイルの場所（ある場合）。この変数の値は、TiDB構成項目[`ssl-ca`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-ca)によって定義されます。

</CustomContent>

### ssl\_cert {#ssl-cert}

<CustomContent platform="tidb">

- Scope: NONE
- デフォルト値: ""
- SSL/TLS接続に使用される証明書ファイル（ある場合）の場所。この変数の値は、TiDB構成項目[`ssl-cert`](/tidb-configuration-file.md#ssl-cert)によって定義されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- Scope: NONE
- デフォルト値: ""
- SSL/TLS接続に使用される証明書ファイル（ある場合）の場所。この変数の値は、TiDB構成項目[`ssl-cert`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-cert)によって定義されます。

</CustomContent>

### ssl\_key {#ssl-key}

<CustomContent platform="tidb">

- Scope: NONE
- デフォルト値: ""
- SSL/TLS接続に使用される秘密鍵ファイル（ある場合）の場所。この変数の値は、TiDB構成項目[`ssl-key`](/tidb-configuration-file.md#ssl-cert)によって定義されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- Scope: NONE
- デフォルト値: ""
- SSL/TLS接続に使用される秘密鍵ファイル（ある場合）の場所。この変数の値は、TiDB構成項目[`ssl-key`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-key)によって定義されます。

</CustomContent>

### system\_time\_zone {#system-time-zone}

- Scope: NONE
- デフォルト値: (システムに依存)
- この変数は、TiDBが最初にブートストラップされたときのシステムタイムゾーンを示します。また、[`time_zone`](#time_zone)も参照してください。

### tidb\_adaptive\_closest\_read\_threshold <span class="version-mark">v6.3.0で新規</span> {#tidb-adaptive-closest-read-threshold-span-class-version-mark-new-in-v6-3-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `4096`
- 範囲: `[0, 9223372036854775807]`
- 単位: バイト
- この変数は、TiDBサーバーが`tidb_replica_read`が`closest-adaptive`に設定されているときに、TiDBサーバーと同じ可用性ゾーンのレプリカに読み取りリクエストを送信する閾値を制御するために使用されます。推定結果がこの閾値以上またはこれに等しい場合、TiDBは同じ可用性ゾーンのレプリカに読み取りリクエストを送信します。それ以外の場合、TiDBはリーダーレプリカに読み取りリクエストを送信します。

### tidb\_allow\_batch\_cop <span class="version-mark">v4.0で新規</span> {#tidb-allow-batch-cop-span-class-version-mark-new-in-v4-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[0, 2]`
- この変数は、TiDBがTiFlashにコプロセッサリクエストを送信する方法を制御するために使用されます。次の値があります。

  - `0`: リクエストをバッチで送信しない
  - `1`: 集計および結合リクエストをバッチで送信する
  - `2`: すべてのコプロセッサリクエストをバッチで送信する

### tidb\_allow\_fallback\_to\_tikv <span class="version-mark">v5.0で新規</span> {#tidb-allow-fallback-to-tikv-span-class-version-mark-new-in-v5-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: ""
- この変数は、指定されたストレージエンジンの実行に失敗した場合、TiDBがこのSQLステートメントをTiKVで再試行することを制御するために使用されます。この変数は""または"tiflash"に設定できます。この変数が"tiflash"に設定されている場合、TiFlashがタイムアウトエラー（エラーコード：ErrTiFlashServerTimeout）を返すと、TiDBはこのSQLステートメントをTiKVで再試行します。

### tidb\_allow\_function\_for\_expression\_index <span class="version-mark">v5.2.0で新規</span> {#tidb-allow-function-for-expression-index-span-class-version-mark-new-in-v5-2-0-span}

- スコープ: NONE
- デフォルト値: `json_array`, `json_array_append`, `json_array_insert`, `json_contains`, `json_contains_path`, `json_depth`, `json_extract`, `json_insert`, `json_keys`, `json_length`, `json_merge_patch`, `json_merge_preserve`, `json_object`, `json_pretty`, `json_quote`, `json_remove`, `json_replace`, `json_search`, `json_set`, `json_storage_size`, `json_type`, `json_unquote`, `json_valid`, `lower`, `md5`, `reverse`, `tidb_shard`, `upper`, `vitess_hash`
- この変数は、式インデックスの作成に使用できる関数を表示するために使用されます。

### tidb\_allow\_mpp <span class="version-mark">v5.0で新規</span> {#tidb-allow-mpp-span-class-version-mark-new-in-v5-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- クエリを実行するためにTiFlashのMPPモードを使用するかどうかを制御します。値のオプションは次のとおりです。
  - `0`または`OFF`：MPPモードは使用されません。
  - `1`または`ON`：オプティマイザは、コスト推定に基づいてMPPモードを使用するかどうかを決定します（デフォルト）。

MPPは、ノード間のデータ交換を可能にし、高性能で高スループットなSQLアルゴリズムを提供するTiFlashエンジンによって提供される分散コンピューティングフレームワークです。MPPモードの選択の詳細については、[MPPモードの選択を制御する](/tiflash/use-tiflash-mpp-mode.md#control-whether-to-select-the-mpp-mode)を参照してください。

### tidb\_allow\_remove\_auto\_inc <span class="version-mark">v2.1.18およびv3.0.4で新規</span> {#tidb-allow-remove-auto-inc-span-class-version-mark-new-in-v2-1-18-and-v3-0-4-span}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

</CustomContent>

- スコープ: SESSION
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`ALTER TABLE MODIFY`または`ALTER TABLE CHANGE`ステートメントを実行して列の`AUTO_INCREMENT`プロパティを削除することができるかどうかを設定するために使用されます。デフォルトでは許可されていません。

### tidb\_analyze\_partition\_concurrency {#tidb-analyze-partition-concurrency}

> **Warning:**
>
> この変数によって制御される機能は、現在のTiDBバージョンでは完全に機能していません。デフォルト値を変更しないでください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `1`
- この変数は、TiDBがパーティションテーブルを分析するときの統計の読み取りと書き込みの並行性を指定します。

### tidb\_analyze\_version <span class="version-mark">v5.1.0で新規</span> {#tidb-analyze-version-span-class-version-mark-new-in-v5-1-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `2`
- 範囲: `[1, 2]`
- TiDBが統計を収集する方法を制御します。
  - TiDB Self-Hostedの場合、この変数のデフォルト値はv5.3.0から`1`から`2`に変更されます。
  - TiDB Cloudの場合、この変数のデフォルト値はv6.5.0から`1`から`2`に変更されます。
  - クラスターが以前のバージョンからアップグレードされた場合、`tidb_analyze_version`のデフォルト値はアップグレード後も変更されません。
- この変数の詳細については、[統計の紹介](/statistics.md)を参照してください。

### tidb\_auto\_analyze\_end\_time {#tidb-auto-analyze-end-time}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 時間
- デフォルト値: `23:59 +0000`
- この変数は、統計の自動更新が許可される時間枠を制限するために使用されます。たとえば、UTC時間で午前1時から3時までの自動統計更新のみを許可するには、`tidb_auto_analyze_start_time='01:00 +0000'`および`tidb_auto_analyze_end_time='03:00 +0000'`を設定します。

### tidb\_auto\_analyze\_partition\_batch\_size <span class="version-mark">v6.4.0で新規</span> {#tidb-auto-analyze-partition-batch-size-span-class-version-mark-new-in-v6-4-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `1`
- 範囲: `[1, 1024]`
- この変数は、パーティションテーブルを[自動的に分析](/statistics.md#automatic-update)するときにTiDBが分析するパーティションの数を指定します（つまり、パーティションテーブルの統計を自動的に収集することを意味します）。
- この変数の値がパーティションの数よりも小さい場合、TiDBはパーティションテーブルのすべてのパーティションを複数のバッチで自動的に分析します。この変数の値がパーティションの数以上の場合、TiDBはパーティションテーブルのすべてのパーティションを同時に分析します。
- パーティションテーブルのパーティション数がこの変数の値よりもはるかに大きく、自動分析に時間がかかる場合は、この変数の値を増やして時間を短縮することができます。

### tidb\_auto\_analyze\_ratio {#tidb-auto-analyze-ratio}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- デフォルト値: `0.5`
- 範囲: `[0, 18446744073709551615]`
- この変数は、TiDBが表の統計を自動的に更新するためのバックグラウンドスレッドで[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)を自動的に実行する閾値を設定するために使用されます。たとえば、値が0.5の場合、表の行の50%以上が変更されたときに自動分析がトリガーされます。自動分析は、`tidb_auto_analyze_start_time`および`tidb_auto_analyze_end_time`を指定して特定の時間帯に制限することができます。

> **Note:**
>
> この機能には、システム変数`tidb_enable_auto_analyze`が`ON`に設定されている必要があります。

### tidb\_auto\_analyze\_start\_time {#tidb-auto-analyze-start-time}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 時間
- デフォルト値: `00:00 +0000`
- この変数は、統計の自動更新が許可される時間枠を制限するために使用されます。たとえば、UTC時間で午前1時から3時までの自動統計更新のみを許可するには、`tidb_auto_analyze_start_time='01:00 +0000'`および`tidb_auto_analyze_end_time='03:00 +0000'`を設定します。

### tidb\_auto\_build\_stats\_concurrency <span class="version-mark">v6.5.0で新規</span> {#tidb-auto-build-stats-concurrency-span-class-version-mark-new-in-v6-5-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[1, 256]`
- この変数は、統計の自動更新の実行の並行性を設定するために使用されます。

### tidb\_backoff\_lock\_fast {#tidb-backoff-lock-fast}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `10`
- 範囲: `[1, 2147483647]`
- この変数は、読み取りリクエストがロックに遭遇したときの`backoff`時間を設定するために使用されます。

### tidb\_backoff\_weight {#tidb-backoff-weight}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `2`
- Range: `[0, 2147483647]`
- この変数は、TiDBの`backoff`の最大リトライ時間、つまり、内部ネットワークや他のコンポーネント（TiKV、PD）の障害が発生したときのリトライリクエストの最大リトライ時間を増やすために使用されます。この変数を使用して、最大リトライ時間を調整することができ、最小値は1です。

  例えば、TiDBがPDからTSOを取得するための基本タイムアウトは15秒です。`tidb_backoff_weight = 2`の場合、TSOを取得するための最大タイムアウトは次のようになります：*基本タイム \* 2 = 30秒*。

  ネットワーク環境が悪い場合、この変数の値を適切に増やすことで、タイムアウトによるアプリケーションエンドへのエラー報告を効果的に緩和することができます。アプリケーションエンドがエラー情報をより速く受け取りたい場合は、この変数の値を最小限に抑えることが推奨されます。

### tidb\_batch\_commit {#tidb-batch-commit}

> **Warning:**
>
> この変数を有効にすることは**推奨されません**。

- Scope: SESSION
- Type: Boolean
- Default value: `OFF`
- この変数は、非推奨のバッチコミット機能を有効にするかどうかを制御するために使用されます。この変数が有効になっていると、トランザクションはいくつかのステートメントをグループ化して非アトミックにコミットすることがありますが、これは推奨されません。

### tidb\_batch\_delete {#tidb-batch-delete}

> **Warning:**
>
> この変数は、非推奨のバッチDML機能に関連しており、データの破損を引き起こす可能性があります。そのため、バッチDMLのためにこの変数を有効にすることは推奨されません。代わりに、[非トランザクショナルDML](/non-transactional-dml.md)を使用してください。

- Scope: SESSION
- Type: Boolean
- Default value: `OFF`
- この変数は、非推奨のバッチDML機能の一部であるバッチ削除機能を有効にするかどうかを制御するために使用されます。この変数が有効になっていると、`DELETE`ステートメントは複数のトランザクションに分割されて非アトミックにコミットされることがあります。これを機能させるためには、`tidb_enable_batch_dml`を有効にし、`tidb_dml_batch_size`に正の値を設定する必要がありますが、これは推奨されません。

### tidb\_batch\_insert {#tidb-batch-insert}

> **Warning:**
>
> この変数は、非推奨のバッチDML機能に関連しており、データの破損を引き起こす可能性があります。そのため、バッチDMLのためにこの変数を有効にすることは推奨されません。代わりに、[非トランザクショナルDML](/non-transactional-dml.md)を使用してください。

- Scope: SESSION
- Type: Boolean
- Default value: `OFF`
- この変数は、非推奨のバッチDML機能の一部であるバッチ挿入機能を有効にするかどうかを制御するために使用されます。この変数が有効になっていると、`INSERT`ステートメントは複数のトランザクションに分割されて非アトミックにコミットされることがあります。これを機能させるためには、`tidb_enable_batch_dml`を有効にし、`tidb_dml_batch_size`に正の値を設定する必要がありますが、これは推奨されません。

### tidb\_batch\_pending\_tiflash\_count <span class="version-mark">v6.0で新規</span> {#tidb-batch-pending-tiflash-count-span-class-version-mark-new-in-v6-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `4000`
- Range: `[0, 4294967295]`
- `ALTER DATABASE SET TIFLASH REPLICA`を使用してTiFlashレプリカを追加する際に許可される利用できないテーブルの最大数を指定します。利用できないテーブルの数がこの制限を超えると、操作は停止されるか、残りのテーブルに対するTiFlashレプリカの設定が非常に遅くなります。

### tidb\_broadcast\_join\_threshold\_count <span class="version-mark">v5.0で新規</span> {#tidb-broadcast-join-threshold-count-span-class-version-mark-new-in-v5-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `10240`
- Range: `[0, 9223372036854775807]`
- 単位: 行
- 結合操作のオブジェクトがサブクエリに属する場合、オプティマイザはサブクエリ結果セットのサイズを推定できません。この状況では、サイズは結果セットの行数によって決定されます。サブクエリの推定行数がこの変数の値よりも少ない場合、ブロードキャストハッシュ結合アルゴリズムが使用されます。それ以外の場合、シャッフルハッシュ結合アルゴリズムが使用されます。
- この変数は、[`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710)が有効になった後には効果を発揮しません。

### tidb\_broadcast\_join\_threshold\_size <span class="version-mark">v5.0で新規</span> {#tidb-broadcast-join-threshold-size-span-class-version-mark-new-in-v5-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `104857600` (100 MiB)
- Range: `[0, 9223372036854775807]`
- 単位: バイト
- テーブルサイズがこの変数の値よりも小さい場合、ブロードキャストハッシュ結合アルゴリズムが使用されます。それ以外の場合、シャッフルハッシュ結合アルゴリズムが使用されます。
- この変数は、[`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710)が有効になった後には効果を発揮しません。

### tidb\_build\_stats\_concurrency {#tidb-build-stats-concurrency}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `4`
- Range: `[1, 256]`
- 単位: スレッド
- この変数は、`ANALYZE`ステートメントの実行並列度を設定するために使用されます。
- 変数を大きな値に設定すると、他のクエリの実行パフォーマンスに影響を与えます。

### tidb\_capture\_plan\_baselines <span class="version-mark">v4.0で新規</span> {#tidb-capture-plan-baselines-span-class-version-mark-new-in-v4-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は、[ベースラインキャプチャ](/sql-plan-management.md#baseline-capturing)機能を有効にするかどうかを制御するために使用されます。この機能はステートメントサマリーに依存しているため、ベースラインキャプチャを使用する前にステートメントサマリーを有効にする必要があります。
- この機能が有効になると、ステートメントサマリー内の過去のSQLステートメントが定期的に走査され、少なくとも2回現れるSQLステートメントに対して自動的にバインディングが作成されます。

### tidb\_cdc\_write\_source <span class="version-mark">v6.5.0で新規</span> {#tidb-cdc-write-source-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: SESSION
- Persists to cluster: No
- Type: Integer
- Default value: `0`
- Range: `[0, 15]`
- この変数が0以外の値に設定されている場合、このセッションで書き込まれたデータはTiCDCによって書き込まれたものと見なされます。この変数はTiCDCによってのみ変更できます。絶対にこの変数を手動で変更しないでください。

### tidb\_check\_mb4\_value\_in\_utf8 {#tidb-check-mb4-value-in-utf8}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- Scope: GLOBAL
- Persists to cluster: No、接続している現在のTiDBインスタンスにのみ適用されます。
- Type: Boolean
- Default value: `ON`
- この変数は、`utf8`文字セットが[基本多言語面（BMP）](https://en.wikipedia.org/wiki/Plane_\(Unicode\)#Basic_Multilingual_Plane)からの値のみを保存するように強制するために使用されます。BMPの外の文字を保存するには、`utf8mb4`文字セットを使用することを推奨します。
- このオプションは、以前のTiDBバージョンからクラスターをアップグレードする際に、`utf8`のチェックがより緩和されていた場合に無効にする必要があるかもしれません。詳細については、[アップグレード後のFAQ](https://docs.pingcap.com/tidb/stable/upgrade-faq)を参照してください。

### tidb\_checksum\_table\_concurrency {#tidb-checksum-table-concurrency}

- Scope: SESSION
- Type: Integer
- Default value: `4`
- Range: `[1, 256]`
- 単位: スレッド
- この変数は、[`ADMIN CHECKSUM TABLE`](/sql-statements/sql-statement-admin-checksum-table.md)ステートメントのスキャンインデックス並列度を設定するために使用されます。
- 変数を大きな値に設定すると、他のクエリの実行パフォーマンスに影響を与えます。

### tidb\_committer\_concurrency <span class="version-mark">v6.1.0で新規</span> {#tidb-committer-concurrency-span-class-version-mark-new-in-v6-1-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `128`
- Range: `[1, 10000]`
- 単一トランザクションのコミットフェーズで実行されるリクエストに対するゴルーチンの数。
- コミットするトランザクションが大きすぎる場合、トランザクションがコミットされる際のフローコントロールキューの待機時間が長すぎるかもしれません。この状況では、構成値を増やしてコミットを高速化することができます。
- この設定は以前は`tidb.toml`オプション（`performance.committer-concurrency`）でしたが、TiDB v6.1.0からシステム変数に変更されました。"

### tidb\_config {#tidb-config}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- Scope: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- デフォルト値: ""
- この変数は読み取り専用です。現在のTiDBサーバーの構成情報を取得するために使用されます。

### tidb\_constraint\_check\_in\_place {#tidb-constraint-check-in-place}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は楽観的トランザクションにのみ適用されます。悲観的トランザクションの場合は、代わりに[`tidb_constraint_check_in_place_pessimistic`](#tidb_constraint_check_in_place_pessimistic-new-in-v630)を使用してください。
- この変数が`OFF`に設定されている場合、ユニークインデックスの重複値のチェックはトランザクションがコミットされるまで延期されます。これによりパフォーマンスが向上しますが、一部のアプリケーションにとっては予期しない動作になるかもしれません。詳細については[Constraints](/constraints.md#optimistic-transactions)を参照してください。

  - `tidb_constraint_check_in_place`を`OFF`に設定して楽観的トランザクションを使用する場合:

    ```sql
    tidb> create table t (i int key);
    tidb> insert into t values (1);
    tidb> begin optimistic;
    tidb> insert into t values (1);
    Query OK, 1 row affected
    tidb> commit; -- トランザクションがコミットされるときにのみチェックされます。
    ERROR 1062 : Duplicate entry '1' for key 't.PRIMARY'
    ```

  - `tidb_constraint_check_in_place`を`ON`に設定して楽観的トランザクションを使用する場合:

    ```sql
    tidb> set @@tidb_constraint_check_in_place=ON;
    tidb> begin optimistic;
    tidb> insert into t values (1);
    ERROR 1062 : Duplicate entry '1' for key 't.PRIMARY'
    ```

### tidb\_constraint\_check\_in\_place\_pessimistic <span class="version-mark">v6.3.0で新規</span> {#tidb-constraint-check-in-place-pessimistic-span-class-version-mark-new-in-v6-3-0-span}

- Scope: SESSION
- タイプ: ブール値

<CustomContent platform="tidb">

- デフォルト値: デフォルトでは、[`pessimistic-txn.constraint-check-in-place-pessimistic`](/tidb-configuration-file.md#constraint-check-in-place-pessimistic-new-in-v640)構成項目は`true`なので、この変数のデフォルト値は`ON`です。[`pessimistic-txn.constraint-check-in-place-pessimistic`](/tidb-configuration-file.md#constraint-check-in-place-pessimistic-new-in-v640)が`false`に設定されている場合、この変数のデフォルト値は`OFF`です。

</CustomContent>

<CustomContent platform="tidb-cloud">

- デフォルト値: `ON`

</CustomContent>

- この変数は悲観的トランザクションにのみ適用されます。楽観的トランザクションの場合は、代わりに[`tidb_constraint_check_in_place`](#tidb_constraint_check_in_place)を使用してください。
- この変数が`OFF`に設定されている場合、TiDBはユニークインデックスの一意制約チェックを延期します（次にインデックスにロックが必要なステートメントを実行するとき、またはトランザクションをコミットするとき）。これによりパフォーマンスが向上しますが、一部のアプリケーションにとっては予期しない動作になるかもしれません。詳細については[Constraints](/constraints.md#pessimistic-transactions)を参照してください。
- この変数を無効にすると、TiDBは悲観的トランザクションで`LazyUniquenessCheckFailure`エラーを返す可能性があります。このエラーが発生すると、TiDBは現在のトランザクションをロールバックします。
- この変数を無効にすると、悲観的トランザクションのコミットが`Write conflict`または`Duplicate entry`エラーを返す可能性があります。このようなエラーが発生すると、TiDBは現在のトランザクションをロールバックします。

  - `tidb_constraint_check_in_place_pessimistic`を`OFF`に設定して悲観的トランザクションを使用する場合:

    ```sql
    set @@tidb_constraint_check_in_place_pessimistic=OFF;
    create table t (i int key);
    insert into t values (1);
    begin pessimistic;
    insert into t values (1);
    ```

    ```
    Query OK, 1 row affected
    ```

    ```sql
    tidb> commit; -- トランザクションがコミットされるときにのみチェックされます。
    ```

    ```
    ERROR 1062 : Duplicate entry '1' for key 't.PRIMARY'
    ```

  - `tidb_constraint_check_in_place_pessimistic`を`ON`に設定して悲観的トランザクションを使用する場合:

    ```sql
    set @@tidb_constraint_check_in_place_pessimistic=ON;
    begin pessimistic;
    insert into t values (1);
    ```

    ```
    ERROR 1062 : Duplicate entry '1' for key 't.PRIMARY'
    ```

### tidb\_cost\_model\_version <span class="version-mark">v6.2.0で新規</span> {#tidb-cost-model-version-span-class-version-mark-new-in-v6-2-0-span}

> **Note:**
>
> - TiDB v6.5.0以降、新しく作成されたクラスターはデフォルトでCost Model Version 2を使用します。v6.5.0以前のTiDBバージョンからv6.5.0以降にアップグレードする場合、`tidb_cost_model_version`の値は変更されません。
> - コストモデルのバージョンを切り替えると、クエリプランに変更が生じる可能性があります。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `2`
- 値のオプション:
  - `1`: デフォルトでTiDB v6.4.0およびそれ以前のバージョンで使用されているCost Model Version 1を有効にします。
  - `2`: [Cost Model Version 2](/cost-model.md#cost-model-version-2)を有効にします。これはTiDB v6.5.0以降で一般利用可能であり、内部テストではバージョン1よりも正確です。
- コストモデルのバージョンはオプティマイザのプラン決定に影響を与えます。詳細については[Cost Model](/cost-model.md)を参照してください。

### tidb\_current\_ts {#tidb-current-ts}

- Scope: SESSION
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 9223372036854775807]`
- この変数は読み取り専用です。現在のトランザクションのタイムスタンプを取得するために使用されます。

### tidb\_ddl\_disk\_quota <span class="version-mark">v6.3.0で新規</span> {#tidb-ddl-disk-quota-span-class-version-mark-new-in-v6-3-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `107374182400` (100 GiB)
- 範囲: `[107374182400, 1125899906842624]` (\[100 GiB, 1 PiB])
- 単位: バイト
- この変数は、[`tidb_ddl_enable_fast_reorg`](#tidb_ddl_enable_fast_reorg-new-in-v630)が有効になっている場合にのみ効果があります。これは、インデックスを作成する際のバックフィリング中のローカルストレージの使用制限を設定します。"

### tidb\_ddl\_enable\_fast\_reorg <span class="version-mark">v6.3.0で新規</span> {#tidb-ddl-enable-fast-reorg-span-class-version-mark-new-in-v6-3-0-span}

> **Note:**
>
> - [TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated) クラスターを使用している場合、この変数を使用してインデックス作成の速度を向上させる場合は、TiDBクラスターがAWSにホストされ、TiDBノードのサイズが少なくとも8 vCPUであることを確認してください。
> - [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターの場合、この変数は読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、`ADD INDEX`および`CREATE INDEX`の加速を有効にするかどうかを制御します。この変数の値を`ON`に設定すると、大量のデータを持つテーブルのインデックス作成のパフォーマンスが向上します。
- v7.1.0から、インデックスの加速操作はチェックポイントをサポートしています。TiDBオーナーノードが再起動されたり、障害が発生した場合でも、TiDBは定期的に自動更新されるチェックポイントから進行状況を回復できます。
- 完了した`ADD INDEX`操作が加速されているかどうかを確認するには、[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md#admin-show-ddl-jobs) ステートメントを実行して、`JOB_TYPE`列に`ingest`が表示されるかどうかを確認できます。

<CustomContent platform="tidb">

> **Warning:**
>
> 現在、PITRリカバリは、ログバックアップ中にインデックスの加速によって作成されたインデックスを互換性を実現するために追加処理を行います。詳細については、[Why is the acceleration of adding indexes feature incompatible with PITR?](/faq/backup-and-restore-faq.md#why-is-the-acceleration-of-adding-indexes-feature-incompatible-with-pitr)を参照してください。

> **Note:**
>
> - インデックスの加速には、書き込み可能で十分な空き容量のある[`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)が必要です。`temp-dir`が使用できない場合、TiDBは非加速のインデックス構築にフォールバックします。SSDディスクに`temp-dir`を配置することをお勧めします。
>
> - TiDBをv6.5.0以降にアップグレードする前に、TiDBの[`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)パスが正しくSSDディスクにマウントされているかどうかを確認することをお勧めします。TiDBを実行するオペレーティングシステムユーザーがこのディレクトリに対して読み取りおよび書き込み権限を持っていることを確認してください。そうでない場合、DDL操作に予測不可能な問題が発生する可能性があります。このパスはTiDBの構成項目であり、TiDBが再起動した後に有効になります。したがって、アップグレード前にこの構成項目を設定することで、別の再起動を回避できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Warning:**
>
> 現在、この機能は[ALTER TABLEステートメントで複数の列またはインデックスを変更](/sql-statements/sql-statement-alter-table.md)することと完全に互換性がありません。インデックスの加速でユニークインデックスを追加する場合は、同じステートメントで他の列またはインデックスを変更しないようにする必要があります。
>
> 現在、PITRリカバリは、ログバックアップ中にインデックスの加速によって作成されたインデックスを互換性を実現するために追加処理を行います。詳細については、[Why is the acceleration of adding indexes feature incompatible with PITR?](https://docs.pingcap.com/tidb/v7.0/backup-and-restore-faq#why-is-the-acceleration-of-adding-indexes-feature-incompatible-with-pitr)を参照してください。

</CustomContent>

### tidb\_enable\_dist\_task <span class="version-mark">v7.1.0で新規</span> {#tidb-enable-dist-task-span-class-version-mark-new-in-v7-1-0-span}

> **Warning:**
>
> この機能はまだ実験段階です。本番環境でこの機能を有効にすることはお勧めしません。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `OFF`
- この変数は、[TiDBバックエンドタスク分散実行フレームワーク](/tidb-distributed-execution-framework.md)を有効にするかどうかを制御します。フレームワークが有効になると、DDLやインポートなどのバックエンドタスクは、クラスター内の複数のTiDBノードによって分散的に実行および完了されます。
- TiDB v7.1.0では、フレームワークはパーティションテーブルの`ADD INDEX`ステートメントのみを分散的に実行することができます。
- この変数は`tidb_ddl_distribute_reorg`から名前が変更されました。

### tidb\_ddl\_error\_count\_limit {#tidb-ddl-error-count-limit}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `512`
- 範囲: `[0, 9223372036854775807]`
- この変数は、DDL操作が失敗した場合のリトライ回数を設定するために使用されます。リトライ回数がパラメータ値を超えると、誤ったDDL操作はキャンセルされます。

### tidb\_ddl\_flashback\_concurrency <span class="version-mark">v6.3.0で新規</span> {#tidb-ddl-flashback-concurrency-span-class-version-mark-new-in-v6-3-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `64`
- 範囲: `[1, 256]`
- この変数は、[`FLASHBACK CLUSTER`](/sql-statements/sql-statement-flashback-cluster.md)の並列実行を制御します。

### tidb\_ddl\_reorg\_batch\_size {#tidb-ddl-reorg-batch-size}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `256`
- 範囲: `[32, 10240]`
- 単位: 行
- この変数は、DDL操作の`re-organize`フェーズ中のバッチサイズを設定するために使用されます。たとえば、TiDBが`ADD INDEX`操作を実行すると、インデックスデータは`tidb_ddl_reorg_worker_cnt`（数）の並列ワーカーによってバッチでバックフィルされます。
  - `ADD INDEX`操作中に`UPDATE`や`REPLACE`などの多くの更新操作が存在する場合、大きなバッチサイズはトランザクションの競合の可能性が高くなります。この場合、バッチサイズを小さな値に調整する必要があります。最小値は32です。
  - トランザクションの競合が存在しない場合、バッチサイズを大きな値に設定できます（ワーカーカウントを考慮してください。参照のために[Interaction Test on Online Workloads and `ADD INDEX` Operations](https://docs.pingcap.com/tidb/stable/online-workloads-and-add-index-operations)を参照してください）。これにより、データのバックフィル速度が向上しますが、TiKVへの書き込み圧力も高くなります。

### tidb\_ddl\_reorg\_priority {#tidb-ddl-reorg-priority}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION
- タイプ: 列挙型
- デフォルト値: `PRIORITY_LOW`
- 値のオプション: `PRIORITY_LOW`, `PRIORITY_NORMAL`, `PRIORITY_HIGH`
- この変数は、`re-organize`フェーズでの`ADD INDEX`操作の優先度を設定するために使用されます。
- この変数の値を`PRIORITY_LOW`、`PRIORITY_NORMAL`、または`PRIORITY_HIGH`に設定できます。

### tidb\_ddl\_reorg\_worker\_cnt {#tidb-ddl-reorg-worker-cnt}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `4`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`re-organize`フェーズでのDDL操作の並列性を設定するために使用されます。

### tidb\_default\_string\_match\_selectivity <span class="version-mark">v6.2.0で新規</span> {#tidb-default-string-match-selectivity-span-class-version-mark-new-in-v6-2-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- デフォルト値: `0.8`
- 範囲: `[0, 1]`
- この変数は、行数を推定する際にフィルタ条件で`like`、`rlike`、および`regexp`関数のデフォルトの選択率を設定するために使用されます。この変数はまた、これらの関数の推定を支援するためにTopNを有効にするかどうかも制御します。
- TiDBは`like`を統計情報を使用してフィルタ条件で推定しようとします。しかし、`like`が複雑な文字列に一致する場合、または`rlike`または`regexp`を使用する場合、TiDBは統計情報を完全に使用できないことがよくあり、その結果、選択率`0.8`が代わりに設定され、推定が不正確になります。
- この変数は、前述の動作を変更するために使用されます。変数が`0`以外の値に設定されている場合、選択率は指定された変数値になります。
- 変数が`0`に設定されている場合、TiDBは統計情報でTopNを使用して評価しようとします。これにより、前述の3つの関数を推定する際に統計情報内のNULL数も考慮されます。前提条件は、[`tidb_analyze_version`](#tidb_analyze_version-new-in-v510)が`2`に設定されているときに統計情報が収集されていることです。この評価はわずかにパフォーマンスに影響する可能性があります。
- 変数が`0.8`以外の値に設定されている場合、TiDBは`not like`、`not rlike`、および`not regexp`の推定を適切に調整します。

### tidb\_disable\_txn\_auto\_retry {#tidb-disable-txn-auto-retry}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、明示的な楽観的トランザクションの自動リトライを無効にするかどうかを設定するために使用されます。`ON`のデフォルト値は、トランザクションがTiDBで自動的にリトライされず、`COMMIT`ステートメントがアプリケーションレイヤーで処理する必要があることを意味します。

  値を`OFF`に設定すると、TiDBはトランザクションを自動的にリトライし、`COMMIT`ステートメントからのエラーが少なくなります。この変更を行う際は注意してください。なぜなら、更新が失われる可能性があるからです。

  この変数は、自動的にコミットされる暗黙的なトランザクションやTiDBで内部的に実行されるトランザクションには影響しません。これらのトランザクションの最大リトライ回数は、`tidb_retry_limit`の値によって決定されます。

  詳細については、[リトライの制限](/optimistic-transaction.md#limits-of-retry)を参照してください。

    <CustomContent platform="tidb">

  この変数は楽観的トランザクションにのみ適用され、悲観的トランザクションには適用されません。悲観的トランザクションのリトライ回数は[`max_retry_count`](/tidb-configuration-file.md#max-retry-count)で制御されます。

    </CustomContent>

    <CustomContent platform="tidb-cloud">

  この変数は楽観的トランザクションにのみ適用され、悲観的トランザクションには適用されません。悲観的トランザクションのリトライ回数は256です。

    </CustomContent>

### tidb\_distsql\_scan\_concurrency {#tidb-distsql-scan-concurrency}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `15`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`scan`操作の並列度を設定するために使用されます。
- OLAPシナリオでは大きな値を使用し、OLTPシナリオでは小さな値を使用します。
- OLAPシナリオでは、最大値はすべてのTiKVノードのCPUコア数を超えてはいけません。
- テーブルに多くのパーティションがある場合、変数値を適切に減らすことができます（スキャンするデータのサイズとスキャンの頻度によって決まります）、これによりTiKVがメモリ不足（OOM）になるのを避けることができます。

### tidb\_dml\_batch\_size {#tidb-dml-batch-size}

> **Warning:**
>
> この変数は非推奨のバッチDML機能に関連しており、データの破損を引き起こす可能性があります。そのため、バッチDMLのためにこの変数を有効にすることはお勧めしません。代わりに、[非トランザクショナルDML](/non-transactional-dml.md)を使用してください。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- 単位: 行
- この値が`0`より大きい場合、TiDBは`INSERT`などのステートメントを小さなトランザクションにバッチ処理します。これによりメモリ使用量が削減され、`txn-total-size-limit`が大容量の変更によって達成されないようになります。
- 値が`0`の場合のみ、ACID準拠が提供されます。これを他の値に設定すると、TiDBの原子性と分離性の保証が壊れます。
- この変数を機能させるためには、`tidb_enable_batch_dml`と`tidb_batch_insert`の少なくとも1つを有効にする必要があります。

> **Note:**
>
> v7.0.0から、`tidb_dml_batch_size`は[`LOAD DATA`ステートメント](/sql-statements/sql-statement-load-data.md)にはもはや影響しません。

### tidb\_enable\_1pc <span class="version-mark">v5.0で新規</span> {#tidb-enable-1pc-span-class-version-mark-new-in-v5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、1つのリージョンに影響するトランザクションのための1段階コミット機能を有効にするかどうかを指定するために使用されます。よく使用される2段階コミットと比較して、1段階コミットはトランザクションのコミットのレイテンシを大幅に減少させ、スループットを増加させることができます。

> **Note:**
>
> - `ON`のデフォルト値は新しいクラスターにのみ適用されます。クラスターが以前のバージョンのTiDBからアップグレードされた場合、代わりに`OFF`の値が使用されます。
> - TiDB Binlogを有効にしている場合、この変数を有効にしてもパフォーマンスが向上しません。パフォーマンスを向上させるためには、[TiCDC](https://docs.pingcap.com/tidb/stable/ticdc-overview)を使用することをお勧めします。
> - このパラメータを有効にすることは、1段階コミットがトランザクションのコミットの最適なモードになることを意味します。実際には、トランザクションのコミットの最適なモードはTiDBによって決定されます。

### tidb\_enable\_analyze\_snapshot <span class="version-mark">v6.2.0で新規</span> {#tidb-enable-analyze-snapshot-span-class-version-mark-new-in-v6-2-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`ANALYZE`を実行する際に歴史データまたは最新データを読み取るかどうかを制御します。この変数が`ON`に設定されている場合、`ANALYZE`は`ANALYZE`実行時に利用可能な歴史データを読み取ります。この変数が`OFF`に設定されている場合、`ANALYZE`は最新データを読み取ります。
- v5.2以前では、`ANALYZE`は最新データを読み取ります。v5.2からv6.1まで、`ANALYZE`は`ANALYZE`実行時に利用可能な歴史データを読み取ります。

> **Warning:**
>
> `ANALYZE`が`ANALYZE`実行時に利用可能な歴史データを読み取る場合、`AUTO ANALYZE`の長い期間が`GC life time is shorter than transaction duration`エラーを引き起こす可能性があります。なぜなら、歴史データがガベージコレクションされるからです。

### tidb\_enable\_async\_commit <span class="version-mark">v5.0で新規</span> {#tidb-enable-async-commit-span-class-version-mark-new-in-v5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、2段階トランザクションコミットの第2フェーズをバックグラウンドで非同期に実行するかどうかを制御します。この機能を有効にすると、トランザクションのコミットのレイテンシを減少させることができます。

> **Note:**
>
> - `ON`のデフォルト値は新しいクラスターにのみ適用されます。クラスターが以前のバージョンのTiDBからアップグレードされた場合、代わりに`OFF`の値が使用されます。
> - TiDB Binlogを有効にしている場合、この変数を有効にしてもパフォーマンスが向上しません。パフォーマンスを向上させるためには、[TiCDC](https://docs.pingcap.com/tidb/stable/ticdc-overview)を使用することをお勧めします。
> - このパラメータを有効にすることは、非同期コミットがトランザクションのコミットの最適なモードになることを意味します。実際には、トランザクションのコミットの最適なモードはTiDBによって決定されます。

### tidb\_enable\_auto\_analyze <span class="version-mark">v6.1.0で新規</span> {#tidb-enable-auto-analyze-span-class-version-mark-new-in-v6-1-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- TiDBがバックグラウンドでテーブル統計情報を自動的に更新するかどうかを決定します。
- この設定は以前は`tidb.toml`オプション（`performance.run-auto-analyze`）でしたが、TiDB v6.1.0からシステム変数に変更されました。"

### tidb\_enable\_auto\_increment\_in\_generated {#tidb-enable-auto-increment-in-generated}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は、生成された列または式インデックスを作成する際に`AUTO_INCREMENT`列を含めるかどうかを決定するために使用されます。

### tidb\_enable\_batch\_dml {#tidb-enable-batch-dml}

> **Warning:**
>
> この変数は非推奨のバッチDML機能に関連しており、データの破損を引き起こす可能性があります。そのため、バッチDMLのためにこの変数を有効にすることは推奨されていません。代わりに、[非トランザクショナルDML](/non-transactional-dml.md)を使用してください。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は、非推奨のバッチDML機能を有効にするかどうかを制御します。有効にすると、特定のステートメントが複数のトランザクションに分割される場合があり、これは非アトミックであり注意して使用する必要があります。バッチDMLを使用する場合、操作対象のデータに同時操作がないことを確認する必要があります。また、`tidb_batch_dml_size`の正の値を指定し、`tidb_batch_insert`および`tidb_batch_delete`の少なくとも1つを有効にする必要があります。

### tidb\_enable\_cascades\_planner {#tidb-enable-cascades-planner}

> **Warning:**
>
> 現在、カスケードプランナーは実験的な機能です。本番環境で使用することは推奨されていません。

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は、カスケードプランナーを有効にするかどうかを制御するために使用されます。

### tidb\_enable\_chunk\_rpc <span class="version-mark">v4.0で新規</span> {#tidb-enable-chunk-rpc-span-class-version-mark-new-in-v4-0-span}

- Scope: SESSION
- Type: Boolean
- Default value: `ON`
- この変数は、Coprocessorで`Chunk`データエンコーディング形式を有効にするかどうかを制御するために使用されます。

### tidb\_enable\_clustered\_index <span class="version-mark">v5.0で新規</span> {#tidb-enable-clustered-index-span-class-version-mark-new-in-v5-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Enumeration
- Default value: `ON`
- Possible values: `OFF`, `ON`, `INT_ONLY`
- この変数は、デフォルトでプライマリキーを[クラスター化インデックス](/clustered-indexes.md)として作成するかどうかを制御するために使用されます。ここでの「デフォルト」とは、ステートメントが`CLUSTERED`/`NONCLUSTERED`キーワードを明示的に指定しない場合を指します。サポートされる値は`OFF`、`ON`、`INT_ONLY`です。
  - `OFF`は、デフォルトでプライマリキーが非クラスター化インデックスとして作成されることを示します。
  - `ON`は、デフォルトでプライマリキーがクラスター化インデックスとして作成されることを示します。
  - `INT_ONLY`は、動作が`alter-primary-key`構成項目によって制御されることを示します。`alter-primary-key`が`true`に設定されている場合、すべてのプライマリキーがデフォルトで非クラスター化インデックスとして作成されます。`false`に設定されている場合、整数列で構成されるプライマリキーのみがデフォルトでクラスター化インデックスとして作成されます。

### tidb\_enable\_ddl <span class="version-mark">v6.3.0で新規</span> {#tidb-enable-ddl-span-class-version-mark-new-in-v6-3-0-span}

> **Note:**
>
> この変数は[Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- Persists to cluster: No, only applicable to the current TiDB instance that you are connecting to.
- Default value: `ON`
- Possible values: `OFF`, `ON`
- この変数は、対応するTiDBインスタンスがDDLオーナーになれるかどうかを制御します。現在のTiDBクラスターに1つのTiDBインスタンスしかない場合、それをDDLオーナーにすることを防ぐことはできません。つまり、`OFF`に設定することはできません。

### tidb\_enable\_collect\_execution\_info {#tidb-enable-collect-execution-info}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- Scope: GLOBAL
- Persists to cluster: No, only applicable to the current TiDB instance that you are connecting to.
- Type: Boolean
- Default value: `ON`
- この変数は、遅いクエリログの各オペレータの実行情報を記録するかどうかを制御します。

### tidb\_enable\_column\_tracking <span class="version-mark">v5.4.0で新規</span> {#tidb-enable-column-tracking-span-class-version-mark-new-in-v5-4-0-span}

> **Warning:**
>
> 現在、`PREDICATE COLUMNS`の統計情報を収集することは実験的な機能です。本番環境で使用することは推奨されていません。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は、TiDBが`PREDICATE COLUMNS`を収集するかどうかを制御します。収集を有効にした後、無効にすると以前に収集された`PREDICATE COLUMNS`の情報がクリアされます。詳細については、[特定の列の統計情報を収集](/statistics.md#collect-statistics-on-some-columns)を参照してください。

### tidb\_enable\_enhanced\_security {#tidb-enable-enhanced-security}

- Scope: NONE
- Type: Boolean

<CustomContent platform="tidb">

- Default value: `OFF`
- この変数は、接続しているTiDBサーバーがセキュリティ強化モード（SEM）が有効かどうかを示します。値を変更するには、TiDBサーバーの構成ファイルで`enable-sem`の値を変更し、TiDBサーバーを再起動する必要があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- Default value: `ON`
- この変数は読み取り専用です。TiDB Cloudでは、セキュリティ強化モード（SEM）がデフォルトで有効になっています。

</CustomContent>

- SEMは、[Security-Enhanced Linux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux)などのシステムの設計に触発されています。MySQLの`SUPER`権限を持つユーザーの権限を制限し、代わりに`RESTRICTED`細かい権限を付与することを求めます。これらの細かい権限には次のものが含まれます。
  - `RESTRICTED_TABLES_ADMIN`: `mysql`スキーマのシステムテーブルにデータを書き込み、`information_schema`テーブルの機密カラムを表示する権限。
  - `RESTRICTED_STATUS_ADMIN`: `SHOW STATUS`で機密変数を表示する権限。
  - `RESTRICTED_VARIABLES_ADMIN`: `SHOW [GLOBAL] VARIABLES`および`SET`で機密変数を表示および設定する権限。
  - `RESTRICTED_USER_ADMIN`: 他のユーザーが変更を加えたりユーザーアカウントを削除することを防ぐ権限。

### tidb\_enable\_exchange\_partition {#tidb-enable-exchange-partition}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は、[`テーブルとのパーティションの交換`](/partitioned-table.md#partition-management)機能を有効にするかどうかを制御します。デフォルト値は`ON`であり、`テーブルとのパーティションの交換`はデフォルトで有効になっています。
- この変数はv6.3.0以降非推奨となりました。その値はデフォルト値`ON`に固定されます。つまり、`テーブルとのパーティションの交換`はデフォルトで有効になっています。

### tidb\_enable\_extended\_stats {#tidb-enable-extended-stats}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は、TiDBが拡張統計情報を収集してオプティマイザをガイドできるかどうかを示します。詳細については、[拡張統計情報の概要](/extended-statistics.md)を参照してください。

### tidb\_enable\_external\_ts\_read <span class="version-mark">v6.4.0で新規</span> {#tidb-enable-external-ts-read-span-class-version-mark-new-in-v6-4-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数が`ON`に設定されている場合、TiDBは[`tidb_external_ts`](#tidb_external_ts-new-in-v640)で指定されたタイムスタンプでデータを読み取ります。

### tidb\_external\_ts <span class="version-mark">v6.4.0で新規</span> {#tidb-external-ts-span-class-version-mark-new-in-v6-4-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `0`
- [`tidb_enable_external_ts_read`](#tidb_enable_external_ts_read-new-in-v640)が`ON`に設定されている場合、TiDBはこの変数で指定されたタイムスタンプでデータを読み取ります。

### tidb\_enable\_fast\_analyze {#tidb-enable-fast-analyze}

> **Warning:**
>
> 現在、`Fast Analyze`は実験的な機能です。本番環境で使用することは推奨されていません。

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は、統計`Fast Analyze`機能を有効にするかどうかを設定するために使用されます。
- 統計`Fast Analyze`機能が有効になっている場合、TiDBはデータの約10,000行をランダムにサンプリングして統計を取得します。データが均等に分布していないか、データサイズが小さい場合、統計の精度が低くなります。これにより、誤ったインデックスの選択など、最適でない実行計画が生じる可能性があります。通常の`Analyze`ステートメントの実行時間が許容できる場合、`Fast Analyze`機能を無効にすることをお勧めします。

### tidb\_enable\_foreign\_key <span class="version-mark">v6.3.0で新規</span> {#tidb-enable-foreign-key-span-class-version-mark-new-in-v6-3-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: v6.6.0以前は`OFF`、v6.6.0以降はデフォルト値は`ON`
- この変数は、`FOREIGN KEY`機能を有効にするかどうかを制御します。"

### tidb\_enable\_gc\_aware\_memory\_track {#tidb-enable-gc-aware-memory-track}

> **Warning:**
>
> この変数は、TiDBのデバッグ用の内部変数です。将来のリリースで削除される可能性があります。この変数を設定しないでください。

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数はGC-Awareメモリトラックを有効にするかどうかを制御します。

### tidb\_enable\_non\_prepared\_plan\_cache {#tidb-enable-non-prepared-plan-cache}

> **Warning:**
>
> 非プリペアド実行プランキャッシュは実験的な機能です。本番環境で使用しないことをお勧めします。この機能は、事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は[Non-prepared plan cache](/sql-non-prepared-plan-cache.md)機能を有効にするかどうかを制御します。

### tidb\_enable\_non\_prepared\_plan\_cache\_for\_dml <span class="version-mark">v7.1.0で新規</span> {#tidb-enable-non-prepared-plan-cache-for-dml-span-class-version-mark-new-in-v7-1-0-span}

> **Warning:**
>
> 非プリペアド実行プランキャッシュは実験的な機能です。本番環境で使用しないことをお勧めします。この機能は、事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`.
- この変数はDMLステートメントのために[Non-prepared plan cache](/sql-non-prepared-plan-cache.md)機能を有効にするかどうかを制御します。

### tidb\_enable\_gogc\_tuner <span class="version-mark">v6.4.0で新規</span> {#tidb-enable-gogc-tuner-span-class-version-mark-new-in-v6-4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数はGOGC Tunerを有効にするかどうかを制御します。

### tidb\_enable\_historical\_stats {#tidb-enable-historical-stats}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は歴史統計情報を有効にするかどうかを制御します。デフォルト値は`OFF`から`ON`に変更され、つまり、歴史統計情報はデフォルトで有効になります。

### tidb\_enable\_historical\_stats\_for\_capture {#tidb-enable-historical-stats-for-capture}

> **Warning:**
>
> この変数で制御される機能は、現在のTiDBバージョンでは完全に機能していません。デフォルト値を変更しないでください。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は`PLAN REPLAYER CAPTURE`によってキャプチャされる情報に、デフォルトで歴史統計情報を含めるかどうかを制御します。デフォルト値`OFF`は、デフォルトで歴史統計情報が含まれないことを意味します。

### tidb\_enable\_index\_merge <span class="version-mark">v4.0で新規</span> {#tidb-enable-index-merge-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> - TiDBクラスターをv4.0.0より前のバージョンからv5.4.0またはそれ以降にアップグレードした後、この変数は実行プランの変更によるパフォーマンスの低下を防ぐためにデフォルトで無効になります。
>
> - TiDBクラスターをv4.0.0またはそれ以降からv5.4.0またはそれ以降にアップグレードした後、この変数はアップグレード前の設定のままです。
>
> - v5.4.0以降、新しくデプロイされたTiDBクラスターでは、この変数はデフォルトで有効になります。

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数はインデックスマージ機能を有効にするかどうかを制御します。

### tidb\_enable\_index\_merge\_join {#tidb-enable-index-merge-join}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- `IndexMergeJoin`オペレーターを有効にするかどうかを指定します。
- この変数はTiDBの内部操作にのみ使用されます。調整することは**お勧めしません**。さもなければ、データの正確性に影響を与える可能性があります。

### tidb\_enable\_legacy\_instance\_scope <span class="version-mark">v6.0.0で新規</span> {#tidb-enable-legacy-instance-scope-span-class-version-mark-new-in-v6-0-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は`INSTANCE`スコープの変数を`SET SESSION`および`SET GLOBAL`構文を使用して設定することを許可します。
- このオプションは、以前のTiDBバージョンとの互換性のためにデフォルトで有効になっています。

### tidb\_enable\_list\_partition <span class="version-mark">v5.0で新規</span> {#tidb-enable-list-partition-span-class-version-mark-new-in-v5-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は`LIST (COLUMNS) TABLE PARTITION`機能を有効にするかどうかを設定します。

### tidb\_enable\_local\_txn {#tidb-enable-local-txn}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数はリリースされていない機能に使用されます。**変数値を変更しないでください**。

### tidb\_enable\_metadata\_lock <span class="version-mark">v6.3.0で新規</span> {#tidb-enable-metadata-lock-span-class-version-mark-new-in-v6-3-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は[Metadata lock](/metadata-lock.md)機能を有効にするかどうかを設定します。この変数を設定する際には、クラスターで実行中のDDLステートメントがないことを確認する必要があります。さもなければ、データが不正確または一貫性がない可能性があります。

### tidb\_enable\_mutation\_checker <span class="version-mark">v6.0.0で新規</span> {#tidb-enable-mutation-checker-span-class-version-mark-new-in-v6-0-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数はTiDBミューテーションチェッカーを有効にするかどうかを制御します。これはDMLステートメントの実行中にデータとインデックスの整合性をチェックするツールです。ステートメントに対してチェッカーがエラーを返すと、TiDBはステートメントの実行をロールバックします。この変数を有効にすると、CPU使用率がわずかに増加します。詳細については、[データとインデックスの不整合エラーのトラブルシューティング](/troubleshoot-data-inconsistency-errors.md)を参照してください。
- v6.0.0以降の新しいクラスターの場合、デフォルト値は`ON`です。v6.0.0より前のバージョンからアップグレードされた既存のクラスターの場合、デフォルト値は`OFF`です。

### tidb\_enable\_new\_cost\_interface <span class="version-mark">v6.2.0で新規</span> {#tidb-enable-new-cost-interface-span-class-version-mark-new-in-v6-2-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- TiDB v6.2.0は以前のコストモデルの実装をリファクタリングしました。この変数はリファクタリングされたコストモデルの実装を有効にするかどうかを制御します。
- この変数はデフォルトで有効になっています。リファクタリングされたコストモデルは以前と同じコスト計算式を使用するため、プランの決定が変わることはありません。
- クラスターがv6.1からv6.2にアップグレードされる場合、この変数は引き続き`OFF`のままであり、手動で有効にすることをお勧めします。クラスターがv6.1より前のバージョンからアップグレードされる場合、この変数はデフォルトで`ON`に設定されます。

### tidb\_enable\_new\_only\_full\_group\_by\_check <span class="version-mark">v6.1.0で新規</span> {#tidb-enable-new-only-full-group-by-check-span-class-version-mark-new-in-v6-1-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数はTiDBが`ONLY_FULL_GROUP_BY`チェックを実行する際の動作を制御します。`ONLY_FULL_GROUP_BY`の詳細については、[MySQLドキュメント](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by)を参照してください。v6.1.0では、TiDBはこのチェックをより厳密かつ正確に処理します。
- バージョンアップによる潜在的な互換性の問題を回避するため、この変数のデフォルト値はv6.1.0では`OFF`です。

### tidb\_enable\_noop\_functions <span class="version-mark">v4.0で新規</span> {#tidb-enable-noop-functions-span-class-version-mark-new-in-v4-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙
- デフォルト値: `OFF`
- 可能な値: `OFF`, `ON`, `WARN`
- デフォルトでは、TiDBはまだ実装されていない機能の構文を使用しようとするとエラーが返されます。変数値を`ON`に設定すると、TiDBは利用できない機能の場合に無視します。これはSQLコードを変更できない場合に役立ちます。
- `noop`関数を有効にすると、次の動作が制御されます:
  - `LOCK IN SHARE MODE`構文
  - `SQL_CALC_FOUND_ROWS`構文
  - `START TRANSACTION READ ONLY`および`SET TRANSACTION READ ONLY`構文
  - `tx_read_only`、`transaction_read_only`、`offline_mode`、`super_read_only`、`read_only`および`sql_auto_is_null`システム変数
  - `GROUP BY <expr> ASC|DESC`構文

> **警告:**
>
> `OFF`のデフォルト値のみが安全と見なされます。`tidb_enable_noop_functions=1`を設定すると、TiDBがエラーを提供せずに特定の構文を無視する可能性があるため、アプリケーションで予期しない動作が発生する可能性があります。たとえば、構文`START TRANSACTION READ ONLY`は許可されますが、トランザクションは読み取り専用モードのままです。

### tidb\_enable\_noop\_variables <span class="version-mark">v6.2.0で新規</span> {#tidb-enable-noop-variables-span-class-version-mark-new-in-v6-2-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ON`
- 変数値を`OFF`に設定すると、TiDBの動作は次のようになります:
  - `SET`を使用して`noop`変数を設定すると、TiDBは`"setting *variable_name* has no effect in TiDB"`警告を返します。
  - `SHOW [SESSION | GLOBAL] VARIABLES`の結果に`noop`変数が含まれなくなります。
  - `SELECT`を使用して`noop`変数を読み取ると、TiDBは`"variable *variable_name* has no effect in TiDB"`警告を返します。
- TiDBインスタンスが`noop`変数を設定および読み取ったかどうかを確認するには、`SELECT * FROM INFORMATION_SCHEMA.CLIENT_ERRORS_SUMMARY_GLOBAL;`ステートメントを使用できます。

### tidb\_enable\_null\_aware\_anti\_join <span class="version-mark">v6.3.0で新規</span> {#tidb-enable-null-aware-anti-join-span-class-version-mark-new-in-v6-3-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: v7.0.0より前では`OFF`、v7.0.0以降は`ON`
- タイプ: ブール値
- この変数は、特別なセット演算子`NOT IN`および`!= ALL`によって生成されたANTI JOINでNull Aware Hash Joinを適用するかどうかを制御します。
- 以前のバージョンからv7.0.0またはそれ以降のクラスターにアップグレードすると、この機能は自動的に有効になり、つまりこの変数は`ON`に設定されます。

### tidb\_enable\_outer\_join\_reorder <span class="version-mark">v6.1.0で新規</span> {#tidb-enable-outer-join-reorder-span-class-version-mark-new-in-v6-1-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- v6.1.0以降、TiDBの[Join Reorder](/join-reorder.md)アルゴリズムはOuter Joinをサポートしています。この変数は、TiDBがJoin ReorderのOuter Joinサポートを有効にするかどうかを制御します。
- TiDBの以前のバージョンからクラスターをアップグレードする場合は、次の点に注意してください:

  - アップグレード前のTiDBバージョンがv6.1.0より前の場合、アップグレード後のこの変数のデフォルト値は`ON`になります。
  - アップグレード前のTiDBバージョンがv6.1.0またはそれ以降の場合、アップグレード後のこの変数のデフォルト値はアップグレード前の値に従います。

### `tidb_enable_inl_join_inner_multi_pattern` <span class="version-mark">v7.0.0で新規</span> {#tidb-enable-inl-join-inner-multi-pattern-span-class-version-mark-new-in-v7-0-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、内部テーブルに`Selection`または`Projection`演算子がある場合にIndex Joinがサポートされるかどうかを制御します。デフォルト値`OFF`は、このシナリオでIndex Joinがサポートされないことを意味します。

### tidb\_enable\_ordered\_result\_mode {#tidb-enable-ordered-result-mode}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- 最終出力結果を自動的に並べ替えるかどうかを指定します。
- たとえば、この変数を有効にすると、TiDBは`SELECT a, MAX(b) FROM t GROUP BY a`を`SELECT a, MAX(b) FROM t GROUP BY a ORDER BY a, MAX(b)`として処理します。

### tidb\_enable\_paging <span class="version-mark">v5.4.0で新規</span> {#tidb-enable-paging-span-class-version-mark-new-in-v5-4-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、ページングの方法を使用してコプロセッサリクエストを送信するかどうかを制御します。v5.4.0からv6.2.0までのTiDBバージョンでは、この変数は`IndexLookup`演算子にのみ影響します。v6.2.0以降では、この変数はグローバルに影響します。v6.4.0から、この変数のデフォルト値は`OFF`から`ON`に変更されます。
- ユーザーシナリオ:

  - すべてのOLTPシナリオでは、ページングの方法を使用することをお勧めします。
  - `IndexLookup`および`Limit`を使用し、`Limit`を`IndexScan`にプッシュダウンできない読み取りクエリの場合、読み取りクエリのレイテンシが高く、TiKVの`Unified read pool CPU`の使用率が高くなる場合があります。このような場合、[`tidb_enable_paging`](#tidb_enable_paging-new-in-v540)を`ON`に設定すると、TiDBは少ないデータを処理するため、クエリのレイテンシとリソース消費が低減されます。
  - [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview)を使用したデータエクスポートやフルテーブルスキャンなどのシナリオでは、ページングを有効にすると、TiDBプロセスのメモリ消費が効果的に削減されます。

> **Note:**
>
> TiFlashの代わりにストレージエンジンとしてTiKVを使用するOLAPシナリオでは、ページングを有効にすると、一部の場合でパフォーマンスが低下する可能性があります。低下が発生した場合は、この変数を使用してページングを無効にするか、[`tidb_min_paging_size`](/system-variables.md#tidb_min_paging_size-new-in-v620)および[`tidb_max_paging_size`](/system-variables.md#tidb_max_paging_size-new-in-v630)変数を使用してページングサイズの範囲を調整することを検討してください。

### tidb\_enable\_parallel\_apply <span class="version-mark">v5.0で新規</span> {#tidb-enable-parallel-apply-span-class-version-mark-new-in-v5-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`Apply`演算子の並行性を有効にするかどうかを制御します。並行性の数は`tidb_executor_concurrency`変数で制御されます。`Apply`演算子は相関サブクエリを処理し、デフォルトでは並行性がないため、実行速度が遅くなります。この変数値を`1`に設定すると、並行性が増し、実行速度が速くなります。現在、`Apply`の並行性はデフォルトで無効になっています。

### tidb\_enable\_pipelined\_window\_function {#tidb-enable-pipelined-window-function}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、ウィンドウ関数のパイプライン実行アルゴリズムを使用するかどうかを指定します。

### tidb\_enable\_plan\_cache\_for\_param\_limit <span class="version-mark">v6.6.0で新規</span> {#tidb-enable-plan-cache-for-param-limit-span-class-version-mark-new-in-v6-6-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、Prepared Plan Cacheが`LIMIT`パラメータ(`LIMIT ?`)を持つ実行計画をキャッシュするかどうかを制御します。デフォルト値は`ON`であり、Prepared Plan Cacheは10000を超える変数を持つ実行計画をキャッシュしません。

### tidb\_enable\_plan\_cache\_for\_subquery <span class="version-mark">v7.0.0で新規</span> {#tidb-enable-plan-cache-for-subquery-span-class-version-mark-new-in-v7-0-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、サブクエリを含むクエリをPrepared Plan Cacheがキャッシュするかどうかを制御します。

### tidb\_enable\_plan\_replayer\_capture {#tidb-enable-plan-replayer-capture}

<CustomContent platform="tidb-cloud">

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、`PLAN REPLAYER CAPTURE`機能を有効にするかどうかを制御します。デフォルト値`OFF`は、`PLAN REPLAYER CAPTURE`機能を無効にします。

</CustomContent>

<CustomContent platform="tidb">

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、[`PLAN REPLAYER CAPTURE`機能](/sql-plan-replayer.md#use-plan-replayer-capture-to-capture-target-plans)を有効にするかどうかを制御します。デフォルト値`ON`は、`PLAN REPLAYER CAPTURE`機能を有効にします。

</CustomContent>

### tidb\_enable\_plan\_replayer\_continuous\_capture <span class="version-mark">v7.0.0で新規</span> {#tidb-enable-plan-replayer-continuous-capture-span-class-version-mark-new-in-v7-0-0-span}

<CustomContent platform="tidb-cloud">

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール
- デフォルト値: `OFF`
- この変数は、`PLAN REPLAYER CONTINUOUS CAPTURE` 機能を有効にするかどうかを制御します。デフォルト値 `OFF` は、機能を無効にします。

</CustomContent>

<CustomContent platform="tidb">

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール
- デフォルト値: `OFF`
- この変数は、[`PLAN REPLAYER CONTINUOUS CAPTURE` 機能](/sql-plan-replayer.md#use-plan-replayer-continuous-capture)を有効にするかどうかを制御します。デフォルト値 `OFF` は、機能を無効にします。

</CustomContent>

### tidb\_enable\_prepared\_plan\_cache <span class="version-mark">v6.1.0で新規</span> {#tidb-enable-prepared-plan-cache-span-class-version-mark-new-in-v6-1-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール
- デフォルト値: `ON`
- [Prepared Plan Cache](/sql-prepared-plan-cache.md)を有効にするかどうかを決定します。有効にすると、`Prepare` および `Execute` の実行計画がキャッシュされ、その後の実行は実行計画の最適化をスキップするため、パフォーマンスが向上します。
- この設定は以前は `tidb.toml` オプション (`prepared-plan-cache.enabled`) でしたが、TiDB v6.1.0からシステム変数に変更されました。

### tidb\_enable\_prepared\_plan\_cache\_memory\_monitor <span class="version-mark">v6.4.0で新規</span> {#tidb-enable-prepared-plan-cache-memory-monitor-span-class-version-mark-new-in-v6-4-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ON`
- この変数は、Prepared Plan Cache にキャッシュされた実行計画によって消費されるメモリをカウントするかどうかを制御します。詳細については、[Prepared Plan Cache のメモリ管理](/sql-prepared-plan-cache.md#memory-management-of-prepared-plan-cache)を参照してください。

### tidb\_enable\_pseudo\_for\_outdated\_stats <span class="version-mark">v5.3.0で新規</span> {#tidb-enable-pseudo-for-outdated-stats-span-class-version-mark-new-in-v5-3-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール
- デフォルト値: `OFF`
- この変数は、統計情報が古い場合にオプティマイザーの挙動を制御します。

<CustomContent platform="tidb">

- オプティマイザーは、次のようにしてテーブルの統計情報が古いかどうかを決定します: テーブルの統計情報を取得するために `ANALYZE` が最後に実行されてから、テーブル行の80%が変更された場合（変更された行数を総行数で割ったもの）、オプティマイザーはこのテーブルの統計情報が古いと判断します。この比率は、[`pseudo-estimate-ratio`](/tidb-configuration-file.md#pseudo-estimate-ratio) 構成を使用して変更できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- オプティマイザーは、次のようにしてテーブルの統計情報が古いかどうかを決定します: テーブルの統計情報を取得するために `ANALYZE` が最後に実行されてから、テーブル行の80%が変更された場合（変更された行数を総行数で割ったもの）、オプティマイザーはこのテーブルの統計情報が古いと判断します。

</CustomContent>

- デフォルト値（変数値が `OFF` の場合）、テーブルの統計情報が古い場合でも、オプティマイザーはテーブルの統計情報を引き続き使用します。変数値を `ON` に設定すると、オプティマイザーはテーブルの統計情報が総行数を除いて信頼できなくなったと判断します。その後、オプティマイザーは擬似統計情報を使用します。
- テーブルのデータが`ANALYZE`を実行せずに頻繁に変更される場合、実行計画を安定させるために、変数値を `OFF` に設定することをお勧めします。

### tidb\_enable\_rate\_limit\_action {#tidb-enable-rate-limit-action}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール
- デフォルト値: `OFF`
- この変数は、データを読み取る演算子の動的メモリ制御機能を有効にするかどうかを制御します。デフォルトでは、この演算子は、[`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency) が許可するデータの読み取りスレッドの最大数を有効にします。単一のSQLステートメントのメモリ使用量が毎回[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を超えると、データを読み取る演算子は1つのスレッドを停止します。

<CustomContent platform="tidb">

- データを読み取る演算子が1つのスレッドしか残っておらず、単一のSQLステートメントのメモリ使用量が常に[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を超えると、このSQLステートメントは他のメモリ制御動作をトリガーします。例えば、[ディスクにデータをスピル](/system-variables.md#tidb_enable_tmp_storage_on_oom)します。
- この変数は、SQLステートメントがデータを読み取る場合にメモリ使用量を効果的に制御します。計算操作（結合や集計操作など）が必要な場合、メモリ使用量は`tidb_mem_quota_query`の制御下にない可能性があり、OOMのリスクが高まります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- データを読み取る演算子が1つのスレッドしか残っておらず、単一のSQLステートメントのメモリ使用量が常に[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を超えると、このSQLステートメントはデータをディスクにスピルします。

</CustomContent>

### tidb\_enable\_resource\_control <span class="version-mark">v6.6.0で新規</span> {#tidb-enable-resource-control-span-class-version-mark-new-in-v6-6-0-span}

> **Note:**
>
> この変数は、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ON`
- タイプ: ブール
- この変数は、[リソース制御機能](/tidb-resource-control.md)のスイッチです。この変数が `ON` に設定されていると、TiDBクラスターはリソースグループに基づいてアプリケーションリソースを分離できます。

### tidb\_enable\_reuse\_chunk <span class="version-mark">v6.4.0で新規</span> {#tidb-enable-reuse-chunk-span-class-version-mark-new-in-v6-4-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ON`
- 値のオプション: `OFF`, `ON`
- この変数は、TiDBがチャンクオブジェクトキャッシュを有効にするかどうかを制御します。値が `ON` の場合、TiDBはキャッシュされたチャンクオブジェクトを優先し、キャッシュされていない場合のみシステムから要求します。値が `OFF` の場合、TiDBは直接システムからチャンクオブジェクトを要求します。

### tidb\_enable\_slow\_log {#tidb-enable-slow-log}

> **Note:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- タイプ: ブール
- デフォルト値: `ON`
- この変数は、スローログ機能を有効にするかどうかを制御します。

### tidb\_enable\_tmp\_storage\_on\_oom {#tidb-enable-tmp-storage-on-oom}

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ON`
- 値のオプション: `OFF`, `ON`
- 単一のSQLステートメントがシステム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)で指定されたメモリクォータを超えると、一部の演算子の一時ストレージを有効にするかどうかを制御します。
- v6.3.0以前では、TiDB構成項目 `oom-use-tmp-storage` を使用してこの機能を有効または無効にできます。v6.3.0以降のクラスターにアップグレードした後は、TiDBクラスターはこの変数を自動的に `oom-use-tmp-storage` の値で初期化します。その後、`oom-use-tmp-storage`の値を変更しても**もはや**効果がありません。

### tidb\_enable\_stmt\_summary <span class="version-mark">v3.0.4で新規</span> {#tidb-enable-stmt-summary-span-class-version-mark-new-in-v3-0-4-span}

> **Note:**
>
> この変数は、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール
- デフォルト値: `ON`
- この変数は、ステートメントサマリー機能を有効にするかどうかを制御します。有効にすると、SQL実行情報（時間消費など）が`information_schema.STATEMENTS_SUMMARY`システムテーブルに記録され、SQLパフォーマンスの問題を特定およびトラブルシューティングできます。

### tidb\_enable\_strict\_double\_type\_check <span class="version-mark">v5.0で新規</span> {#tidb-enable-strict-double-type-check-span-class-version-mark-new-in-v5-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール
- デフォルト値: `ON`
- この変数は、テーブルが無効な`DOUBLE`型の定義で作成できるかどうかを制御します。この設定は、以前のバージョンのTiDBが型を検証する際に厳格でなかったため、より古いバージョンからのアップグレードパスを提供することを意図しています。
- `ON`のデフォルト値はMySQLと互換性があります。

たとえば、型`DOUBLE(10)`は、浮動小数点型の精度が保証されていないため、無効と見なされます。`tidb_enable_strict_double_type_check`を`OFF`に変更した後、テーブルは作成されます。

```sql
mysql> CREATE TABLE t1 (id int, c double(10));
ERROR 1149 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use

mysql> SET tidb_enable_strict_double_type_check = 'OFF';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE t1 (id int, c double(10));
Query OK, 0 rows affected (0.09 sec)
```

> **Note:**
>
> この設定は、MySQLが`FLOAT`タイプの精度を許可しているため、`DOUBLE`タイプにのみ適用されます。この動作はMySQL 8.0.17から非推奨となり、`FLOAT`または`DOUBLE`タイプの精度を指定することは推奨されません。

### tidb\_enable\_table\_partition {#tidb-enable-table-partition}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙型
- デフォルト値: `ON`
- 可能な値: `OFF`, `ON`, `AUTO`
- この変数は`TABLE PARTITION`機能を有効にするかどうかを設定するために使用されます：
  - `ON`は、Rangeパーティショニング、Hashパーティショニング、および単一の列でのRange列パーティショニングを有効にします。
  - `AUTO`は`ON`と同じように機能します。
  - `OFF`は`TABLE PARTITION`機能を無効にします。この場合、パーティションテーブルを作成する構文は実行できますが、作成されるテーブルはパーティション化されたものではありません。

### tidb\_enable\_telemetry <span class="version-mark">v4.0.2で新規</span> {#tidb-enable-telemetry-span-class-version-mark-new-in-v4-0-2-span}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`

<CustomContent platform="tidb">

- この変数は、TiDBでのテレメトリ収集が有効になっているかどうかを動的に制御するために使用されます。現在のバージョンでは、テレメトリはデフォルトで無効になっています。すべてのTiDBインスタンスで[`enable-telemetry`](/tidb-configuration-file.md#enable-telemetry-new-in-v402) TiDB構成項目が`false`に設定されている場合、テレメトリ収集は常に無効になり、このシステム変数は効果を発揮しません。詳細については、[Telemetry](/telemetry.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、TiDBでのテレメトリ収集が有効になっているかどうかを動的に制御するために使用されます。

</CustomContent>

### tidb\_enable\_tiflash\_read\_for\_write\_stmt <span class="version-mark">v6.3.0で新規</span> {#tidb-enable-tiflash-read-for-write-stmt-span-class-version-mark-new-in-v6-3-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、`INSERT`、`DELETE`、および`UPDATE`を含むSQLステートメントの読み取り操作をTiFlashにプッシュダウンできるかどうかを制御します。例：

  - `INSERT INTO SELECT`ステートメントでの`SELECT`クエリ（典型的な使用シナリオ：[TiFlashクエリ結果のマテリアリゼーション](/tiflash/tiflash-results-materialization.md)）
  - `UPDATE`および`DELETE`ステートメントでの`WHERE`条件フィルタリング
- v7.1.0から、この変数は非推奨となりました。[`tidb_allow_mpp = ON`](/system-variables.md#tidb_allow_mpp-new-in-v50)の場合、オプティマイザは[SQLモード](/sql-mode.md)とTiFlashレプリカのコスト見積もりに基づいてクエリをTiFlashにプッシュダウンするかどうかを知能的に決定します。TiDBは、現在のセッションの[SQLモード](/sql-mode.md)が厳格でない場合にのみ（つまり、`sql_mode`の値に`STRICT_TRANS_TABLES`および`STRICT_ALL_TABLES`が含まれない場合）`INSERT INTO SELECT`などの`INSERT`、`DELETE`、および`UPDATE`を含むSQLステートメントの読み取り操作をTiFlashにプッシュダウンすることを許可します。

### tidb\_enable\_top\_sql <span class="version-mark">v5.4.0で新規</span> {#tidb-enable-top-sql-span-class-version-mark-new-in-v5-4-0-span}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`

<CustomContent platform="tidb">

- この変数は、[Top SQL](/dashboard/top-sql.md)機能を有効にするかどうかを制御するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、[Top SQL](https://docs.pingcap.com/tidb/stable/top-sql)機能を有効にするかどうかを制御するために使用されます。

</CustomContent>

### tidb\_enable\_tso\_follower\_proxy <span class="version-mark">v5.3.0で新規</span> {#tidb-enable-tso-follower-proxy-span-class-version-mark-new-in-v5-3-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数はTSO Follower Proxy機能を有効にします。値が`OFF`の場合、TiDBはPDリーダーからのTSOのみを取得します。この機能が有効になった後、TiDBはすべてのPDノードに均等にリクエストを送信し、PDフォロワーを介してTSOリクエストを転送します。これにより、PDリーダーのCPU負荷が軽減されます。
- TSO Follower Proxyを有効にするシナリオ：
  - TSOリクエストの圧力が高いため、PDリーダーのCPUがボトルネックに達し、TSO RPCリクエストの遅延が発生する場合。
  - TiDBクラスターに多くのTiDBインスタンスがあり、[`tidb_tso_client_batch_max_wait_time`](#tidb_tso_client_batch_max_wait_time-new-in-v530)の値を増やしてもTSO RPCリクエストの遅延が緩和されない場合。

> **Note:**
>
> PDリーダーのCPU使用率のボトルネック（ネットワークの問題などのCPU使用率のボトルネック以外の理由によるTSO RPCの遅延が増加する場合）の場合、TSO Follower Proxyを有効にすると、TiDBの実行遅延が増加し、クラスターのQPSパフォーマンスに影響を与える可能性があります。

### tidb\_enable\_unsafe\_substitute <span class="version-mark">v6.3.0で新規</span> {#tidb-enable-unsafe-substitute-span-class-version-mark-new-in-v6-3-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、生成された列の式を安全ではない方法で置き換えるかどうかを制御します。デフォルト値は`OFF`で、つまり、安全でない置換はデフォルトで無効になっています。詳細については、[Generated Columns](/generated-columns.md)を参照してください。

### tidb\_enable\_vectorized\_expression <span class="version-mark">v4.0で新規</span> {#tidb-enable-vectorized-expression-span-class-version-mark-new-in-v4-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、ベクトル化された実行を有効にするかどうかを制御するために使用されます。

### tidb\_enable\_window\_function {#tidb-enable-window-function}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、ウィンドウ関数のサポートを有効にするかどうかを制御します。ウィンドウ関数は予約語を使用する場合があります。これにより、通常実行できるSQLステートメントがTiDBのアップグレード後に解析できなくなる可能性があります。この場合、`tidb_enable_window_function`を`OFF`に設定できます。

### `tidb_enable_row_level_checksum` <span class="version-mark">v7.1.0で新規</span> {#tidb-enable-row-level-checksum-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`

<CustomContent platform="tidb">

- この変数は、[TiCDCデータの単一行データの整合性検証](/ticdc/ticdc-integrity-check.md)機能を有効にするかどうかを制御するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、[TiCDCデータの単一行データの整合性検証](https://docs.pingcap.com/tidb/stable/ticdc-integrity-check)機能を有効にするかどうかを制御するために使用されます。

</CustomContent>

### tidb\_enforce\_mpp <span class="version-mark">v5.1で新規</span> {#tidb-enforce-mpp-span-class-version-mark-new-in-v5-1-span}

- スコープ: SESSION
- タイプ: ブール値
- デフォルト値: `OFF`

<CustomContent platform="tidb">

- このデフォルト値を変更するには、[`performance.enforce-mpp`](/tidb-configuration-file.md#enforce-mpp)構成値を変更してください。

</CustomContent>

- オプティマイザのコスト見積もりを無視し、クエリ実行にTiFlashのMPPモードを強制的に使用するかどうかを制御します。値のオプションは次のとおりです：
  - `0`または`OFF`：MPPモードは強制的に使用されません（デフォルト）。
  - `1`または`ON`：コスト見積もりが無視され、MPPモードが強制的に使用されます。この設定は、`tidb_allow_mpp=true`の場合にのみ有効です。

MPPはTiFlashエンジンが提供する分散コンピューティングフレームワークであり、ノード間でデータの交換を可能にし、高性能で高スループットなSQLアルゴリズムを提供します。MPPモードの選択についての詳細については、[MPPモードを選択するかどうかを制御する](/tiflash/use-tiflash-mpp-mode.md#control-whether-to-select-the-mpp-mode)を参照してください。

### tidb\_evolve\_plan\_baselines <span class="version-mark">v4.0で新規</span> {#tidb-evolve-plan-baselines-span-class-version-mark-new-in-v4-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、ベースライン進化機能を有効にするかどうかを制御するために使用されます。詳細な紹介や使用法については、[ベースライン進化](/sql-plan-management.md#baseline-evolution)を参照してください。
- ベースライン進化がクラスターに与える影響を減らすために、次の構成を使用します:
  - `tidb_evolve_plan_task_max_time`を設定して、各実行計画の最大実行時間を制限します。デフォルト値は600秒です。
  - `tidb_evolve_plan_task_start_time`および`tidb_evolve_plan_task_end_time`を設定して、時間枠を制限します。デフォルト値はそれぞれ`00:00 +0000`および`23:59 +0000`です。

### tidb\_evolve\_plan\_task\_end\_time <span class="version-mark">v4.0で新規</span> {#tidb-evolve-plan-task-end-time-span-class-version-mark-new-in-v4-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 時間
- デフォルト値: `23:59 +0000`
- この変数は、ベースライン進化の終了時刻を設定するために使用されます。

### tidb\_evolve\_plan\_task\_max\_time <span class="version-mark">v4.0で新規</span> {#tidb-evolve-plan-task-max-time-span-class-version-mark-new-in-v4-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `600`
- 範囲: `[-1, 9223372036854775807]`
- 単位: 秒
- この変数は、ベースライン進化機能の各実行計画の最大実行時間を制限するために使用されます。

### tidb\_evolve\_plan\_task\_start\_time <span class="version-mark">v4.0で新規</span> {#tidb-evolve-plan-task-start-time-span-class-version-mark-new-in-v4-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 時間
- デフォルト値: `00:00 +0000`
- この変数は、ベースライン進化の開始時刻を設定するために使用されます。

### tidb\_executor\_concurrency <span class="version-mark">v5.0で新規</span> {#tidb-executor-concurrency-span-class-version-mark-new-in-v5-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `5`
- 範囲: `[1, 256]`
- 単位: スレッド

この変数は、次のSQL演算子の並行性を1つの値に設定するために使用されます:

- `index lookup`
- `index lookup join`
- `hash join`
- `hash aggregation`（`partial`および`final`フェーズ）
- `window`
- `projection`

`tidb_executor_concurrency`は、より簡単な管理のために、次の既存のシステム変数をまとめたものです:

- `tidb_index_lookup_concurrency`
- `tidb_index_lookup_join_concurrency`
- `tidb_hash_join_concurrency`
- `tidb_hashagg_partial_concurrency`
- `tidb_hashagg_final_concurrency`
- `tidb_projection_concurrency`
- `tidb_window_concurrency`

v5.0以降、引き続き上記のシステム変数を別々に変更できます（非推奨の警告が返されます）、および変更は対応する単一の演算子にのみ影響します。その後、`tidb_executor_concurrency`を使用して演算子の並行性を変更する場合、別々に変更された演算子には影響しません。すべての演算子の並行性を`tidb_executor_concurrency`で変更する場合は、上記のすべての変数の値を`-1`に設定できます。

以前のバージョンからv5.0にアップグレードされたシステムの場合、上記の変数の値を変更していない場合（つまり、`tidb_hash_join_concurrency`の値が`5`で、残りの値が`4`であることを意味します）、これらの変数によって管理されていた演算子の並行性は自動的に`tidb_executor_concurrency`によって管理されます。これらの変数のいずれかを変更した場合、対応する演算子の並行性は引き続き変更された変数によって制御されます。

### tidb\_expensive\_query\_time\_threshold {#tidb-expensive-query-time-threshold}

> **Note:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- Scope: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- タイプ: 整数
- デフォルト値: `60`
- 範囲: `[10, 2147483647]`
- 単位: 秒
- この変数は、高コストなクエリログを出力するかどうかを決定する閾値値を設定するために使用されます。高コストなクエリログと遅いクエリログの違いは次のとおりです:
  - 遅いログは、ステートメントが実行された後に出力されます。
  - 高コストなクエリログは、閾値値を超える実行時間を持つ実行中のステートメントとそれらに関連する情報を出力します。

### tidb\_force\_priority {#tidb-force-priority}

> **Note:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- Scope: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- タイプ: 列挙型
- デフォルト値: `NO_PRIORITY`
- 可能な値: `NO_PRIORITY`、`LOW_PRIORITY`、`HIGH_PRIORITY`、`DELAYED`
- この変数は、TiDBサーバーで実行されるステートメントのデフォルトの優先度を変更するために使用されます。使用例としては、OLAPクエリを実行している特定のユーザーが、OLTPクエリを実行しているユーザーよりも低い優先度を受け取ることを確認することがあります。
- デフォルト値の`NO_PRIORITY`は、ステートメントの優先度が強制的に変更されないことを意味します。

### tidb\_gc\_concurrency <span class="version-mark">v5.0で新規</span> {#tidb-gc-concurrency-span-class-version-mark-new-in-v5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- GCの[ロック解決](/garbage-collection-overview.md#resolve-locks)ステップで使用するスレッド数を指定します。値が`-1`の場合、TiDBは使用するガベージコレクションスレッドの数を自動的に決定します。

### tidb\_gc\_enable <span class="version-mark">v5.0で新規</span> {#tidb-gc-enable-span-class-version-mark-new-in-v5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- TiKVのガベージコレクションを有効にします。ガベージコレクションを無効にすると、古い行の古いバージョンが削除されなくなり、システムのパフォーマンスが低下します。

### tidb\_gc\_life\_time <span class="version-mark">v5.0で新規</span> {#tidb-gc-life-time-span-class-version-mark-new-in-v5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 持続時間
- デフォルト値: `10m0s`
- 範囲: `[10m0s, 8760h0m0s]`
- データが各GCで保持される時間制限を、Go Durationの形式で設定します。GCが発生すると、現在の時刻からこの値を引いたものがセーフポイントです。

> **Note:**
>
> - 頻繁な更新のシナリオでは、`tidb_gc_life_time`の大きな値（日数、あるいは数ヶ月）は、次のような潜在的な問題を引き起こす可能性があります:
>   - より大きなストレージ使用
>   - 大量の履歴データが、特に`select count(*) from t`のような範囲クエリに対して、ある程度のパフォーマンスに影響を与える可能性があります。
> - `tidb_gc_life_time`よりも長く実行されているトランザクションがある場合、GC中に`start_ts`以降のデータが保持され、このトランザクションの実行を継続できます。たとえば、`tidb_gc_life_time`が10分に設定されている場合、実行されているすべてのトランザクションの中で、最も早く開始されたトランザクションが15分間実行されている場合、GCは最近の15分間のデータを保持します。

### tidb\_gc\_max\_wait\_time <span class="version-mark">v6.1.0で新規</span> {#tidb-gc-max-wait-time-span-class-version-mark-new-in-v6-1-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `86400`
- 範囲: `[600, 31536000]`
- 単位: 秒
- この変数は、アクティブなトランザクションがGCセーフポイントをブロックする最大時間を設定するために使用されます。各GC時に、デフォルトではセーフポイントは実行中のトランザクションの開始時刻を超えません。アクティブなトランザクションの実行時間がこの変数の値を超えない場合、GCセーフポイントは、実行時間がこの値を超えるまでブロックされます。

### tidb\_gc\_run\_interval <span class="version-mark">v5.0で新規</span> {#tidb-gc-run-interval-span-class-version-mark-new-in-v5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: Duration
- デフォルト値: `10m0s`
- 範囲: `[10m0s, 8760h0m0s]`
- GC間隔を指定します。Go Durationの形式で、例えば、`"1h30m"`、`"15m"`のようになります。

### tidb\_gc\_scan\_lock\_mode <span class="version-mark">v5.0で新規</span> {#tidb-gc-scan-lock-mode-span-class-version-mark-new-in-v5-0-span}

> **Warning:**
>
> 現在、Green GCは実験的な機能です。本番環境で使用することはお勧めしません。

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙型
- デフォルト値: `LEGACY`
- 可能な値: `PHYSICAL`, `LEGACY`
  - `LEGACY`: 古いスキャン方法を使用し、つまり、Green GCを無効にします。
  - `PHYSICAL`: 物理的なスキャン方法を使用し、つまり、Green GCを有効にします。

<CustomContent platform="tidb">

- この変数は、GCのResolve Locksステップでロックをスキャンする方法を指定します。変数値が`LEGACY`に設定されている場合、TiDBはリージョンごとにロックをスキャンします。`PHYSICAL`の値を使用すると、各TiKVノードがRaftレイヤーをバイパスしてデータを直接スキャンできるようになり、[Hibernate Region](/tikv-configuration-file.md#hibernate-regions)機能が有効になっている場合に、すべてのリージョンを起こすGCの影響を効果的に緩和し、Resolve Locksステップの実行速度を向上させることができます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、GCのResolve Locksステップでロックをスキャンする方法を指定します。変数値が`LEGACY`に設定されている場合、TiDBはリージョンごとにロックをスキャンします。`PHYSICAL`の値を使用すると、各TiKVノードがRaftレイヤーをバイパスしてデータを直接スキャンできるようになり、すべてのリージョンを起こすGCの影響を効果的に緩和し、Resolve Locksステップの実行速度を向上させることができます。

</CustomContent>

### tidb\_general\_log {#tidb-general-log}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- タイプ: ブール値
- デフォルト値: `OFF`

<CustomContent platform="tidb-cloud">

- この変数は、すべてのSQLステートメントをログに記録するかどうかを設定するために使用されます。この機能はデフォルトで無効になっています。問題を特定する際にすべてのSQLステートメントをトレースする必要がある場合は、この機能を有効にしてください。

</CustomContent>

<CustomContent platform="tidb">

- この変数は、すべてのSQLステートメントを[ログ](/tidb-configuration-file.md#logfile)に記録するかどうかを設定するために使用されます。この機能はデフォルトで無効になっています。問題を特定する際にすべてのSQLステートメントをトレースする必要がある場合は、この機能を有効にしてください。

- この機能のすべての記録をログで表示するには、TiDB構成項目[`log.level`](/tidb-configuration-file.md#level)を`"info"`または`"debug"`に設定し、その後に`"GENERAL_LOG"`文字列をクエリする必要があります。次の情報が記録されます:
  - `conn`: 現在のセッションのID。
  - `user`: 現在のセッションユーザー。
  - `schemaVersion`: 現在のスキーマバージョン。
  - `txnStartTS`: 現在のトランザクションが開始したタイムスタンプ。
  - `forUpdateTS`: 悲観的トランザクションモードでは、`forUpdateTS`はSQLステートメントの現在のタイムスタンプです。悲観的トランザクションで書き込み競合が発生した場合、TiDBは現在実行中のSQLステートメントをリトライし、このタイムスタンプを更新します。リトライ回数は[`max-retry-count`](/tidb-configuration-file.md#max-retry-count)を介して設定できます。楽観的トランザクションモデルでは、`forUpdateTS`は`txnStartTS`と同等です。
  - `isReadConsistency`: 現在のトランザクション分離レベルがRead Committed（RC）であるかどうかを示します。
  - `current_db`: 現在のデータベース名。
  - `txn_mode`: トランザクションモード。値のオプションは`OPTIMISTIC`と`PESSIMISTIC`です。
  - `sql`: 現在のクエリに対応するSQLステートメント。

</CustomContent>

### tidb\_non\_prepared\_plan\_cache\_size {#tidb-non-prepared-plan-cache-size}

> **Warning:**
>
> v7.1.0から、この変数は非推奨となりました。代わりに[`tidb_session_plan_cache_size`](#tidb_session_plan_cache_size-new-in-v710)を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[1, 100000]`
- この変数は[Non-prepared plan cache](/sql-non-prepared-plan-cache.md)によってキャッシュされる実行計画の最大数を制御します。

### tidb\_generate\_binary\_plan <span class="version-mark">v6.2.0で新規</span> {#tidb-generate-binary-plan-span-class-version-mark-new-in-v6-2-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、遅いログとステートメントの要約でバイナリエンコードされた実行計画を生成するかどうかを制御します。
- この変数が`ON`に設定されている場合、TiDBダッシュボードで視覚的な実行計画を表示できます。ただし、TiDBダッシュボードは、この変数が有効になってから生成された実行計画の視覚的表示のみを提供します。
- `SELECT tidb_decode_binary_plan('xxx...')`ステートメントを実行して、バイナリプランから特定のプランを解析できます。

### tidb\_gogc\_tuner\_threshold <span class="version-mark">v6.4.0で新規</span> {#tidb-gogc-tuner-threshold-span-class-version-mark-new-in-v6-4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `0.6`
- 範囲: `[0, 0.9)`
- この変数は、GOGCのチューニングのための最大メモリ閾値を指定します。メモリがこの閾値を超えると、GOGC Tunerは動作を停止します。

### tidb\_guarantee\_linearizability <span class="version-mark">v5.0で新規</span> {#tidb-guarantee-linearizability-span-class-version-mark-new-in-v5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、非同期コミットのためにコミットTSが計算される方法を制御します。デフォルト値（`ON`）の場合、2相コミットはPDサーバーから新しいTSを要求し、そのTSを使用して最終的なコミットTSを計算します。この状況では、すべての並行トランザクションに対して線形性が保証されます。
- この変数を`OFF`に設定すると、PDサーバーからTSを取得するプロセスがスキップされ、因果一貫性のみが保証され、線形性は保証されません。詳細については、ブログ記事[Async Commit, the Accelerator for Transaction Commit in TiDB 5.0](https://en.pingcap.com/blog/async-commit-the-accelerator-for-transaction-commit-in-tidb-5-0/)を参照してください。
- 因果一貫性のみが必要なシナリオでは、パフォーマンスを向上させるためにこの変数を`OFF`に設定できます。

### tidb\_hash\_exchange\_with\_new\_collation {#tidb-hash-exchange-with-new-collation}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、新しい照合順序が有効になっているクラスターでMPPハッシュパーティション交換演算子が生成されるかどうかを制御します。`true`は演算子を生成し、`false`は生成しないことを意味します。
- この変数はTiDBの内部操作に使用されます。この変数を設定することは**推奨されません**。

### tidb\_hash\_join\_concurrency {#tidb-hash-join-concurrency}

> **Warning:**
>
> v5.0以降、この変数は非推奨となりました。代わりに[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`hash join`アルゴリズムの並行性を設定するために使用されます。
- `-1`の値は、`tidb_executor_concurrency`の値が代わりに使用されることを意味します。

### tidb\_hashagg\_final\_concurrency {#tidb-hashagg-final-concurrency}

> **Warning:**
>
> v5.0以降、この変数は非推奨です。代わりに[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`final`フェーズで並行して`hash aggregation`アルゴリズムを実行する並行性を設定するために使用されます。
- 集約関数のパラメータが一意でない場合、`HashAgg`は2つのフェーズでそれぞれ並行して実行されます - `partial`フェーズと`final`フェーズ。
- `-1`の値は、`tidb_executor_concurrency`の値が代わりに使用されることを意味します。

### tidb\_hashagg\_partial\_concurrency {#tidb-hashagg-partial-concurrency}

> **Warning:**
>
> v5.0以降、この変数は非推奨です。代わりに[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`partial`フェーズで並行して`hash aggregation`アルゴリズムを実行する並行性を設定するために使用されます。
- 集約関数のパラメータが一意でない場合、`HashAgg`は2つのフェーズでそれぞれ並行して実行されます - `partial`フェーズと`final`フェーズ。
- `-1`の値は、`tidb_executor_concurrency`の値が代わりに使用されることを意味します。

### tidb\_historical\_stats\_duration <span class="version-mark">v6.6.0で新規</span> {#tidb-historical-stats-duration-span-class-version-mark-new-in-v6-6-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 持続時間
- デフォルト値: `168h`、つまり7日間
- この変数は、履歴統計情報がストレージに保持される期間を制御します。

### tidb\_ignore\_prepared\_cache\_close\_stmt <span class="version-mark">v6.0.0で新規</span> {#tidb-ignore-prepared-cache-close-stmt-span-class-version-mark-new-in-v6-0-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、プリペアドステートメントキャッシュを閉じるコマンドを無視するかどうかを設定するために使用されます。
- この変数が`ON`に設定されている場合、バイナリプロトコルの`COM_STMT_CLOSE`コマンドとテキストプロトコルの[`DEALLOCATE PREPARE`](/sql-statements/sql-statement-deallocate.md)ステートメントが無視されます。詳細については、[COM\_STMT\_CLOSE`コマンドと`DEALLOCATE PREPARE\`ステートメントを無視する](/sql-prepared-plan-cache.md#ignore-the-com_stmt_close-command-and-the-deallocate-prepare-statement)を参照してください。

### tidb\_index\_join\_batch\_size {#tidb-index-join-batch-size}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `25000`
- 範囲: `[1, 2147483647]`
- 単位: 行
- この変数は、`index lookup join`操作のバッチサイズを設定するために使用されます。
- OLAPシナリオでは大きな値を使用し、OLTPシナリオでは小さな値を使用します。

### tidb\_index\_join\_double\_read\_penalty\_cost\_rate <span class="version-mark">v6.6.0で新規</span> {#tidb-index-join-double-read-penalty-cost-rate-span-class-version-mark-new-in-v6-6-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- デフォルト値: `0`
- 範囲: `[0, 18446744073709551615]`
- この変数は、インデックス結合の選択にペナルティコストを適用するかどうかを決定し、オプティマイザがインデックス結合を選択する可能性を減らし、代替の結合方法（ハッシュ結合やtiflash結合など）を選択する可能性を高めます。
- インデックス結合が選択されると、多くのテーブルルックアップリクエストがトリガーされ、多くのリソースを消費します。この変数を使用して、オプティマイザがインデックス結合を選択する可能性を減らすことができます。
- この変数は、[`tidb_cost_model_version`](/system-variables.md#tidb_cost_model_version-new-in-v620)変数が`2`に設定されている場合にのみ有効です。

### tidb\_index\_lookup\_concurrency {#tidb-index-lookup-concurrency}

> **Warning:**
>
> v5.0以降、この変数は非推奨です。代わりに[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`index lookup`操作の並行性を設定するために使用されます。
- OLAPシナリオでは大きな値を使用し、OLTPシナリオでは小さな値を使用します。
- `-1`の値は、`tidb_executor_concurrency`の値が代わりに使用されることを意味します。

### tidb\_index\_lookup\_join\_concurrency {#tidb-index-lookup-join-concurrency}

> **Warning:**
>
> v5.0以降、この変数は非推奨です。代わりに[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`index lookup join`アルゴリズムの並行性を設定するために使用されます。
- `-1`の値は、`tidb_executor_concurrency`の値が代わりに使用されることを意味します。

### tidb\_index\_merge\_intersection\_concurrency <span class="version-mark">v6.5.0で新規</span> {#tidb-index-merge-intersection-concurrency-span-class-version-mark-new-in-v6-5-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- この変数は、インデックスマージが実行する交差操作の最大並行性を設定します。これは、TiDBが動的剪定モードでパーティションテーブルにアクセスする場合にのみ有効です。実際の並行性は、`tidb_index_merge_intersection_concurrency`の値とパーティションテーブルのパーティション数の小さい方の値です。
- デフォルト値の`-1`は、`tidb_executor_concurrency`の値が使用されることを意味します。

### tidb\_index\_lookup\_size {#tidb-index-lookup-size}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `20000`
- 範囲: `[1, 2147483647]`
- 単位: 行
- この変数は、`index lookup`操作のバッチサイズを設定するために使用されます。
- OLAPシナリオでは大きな値を使用し、OLTPシナリオでは小さな値を使用します。

### tidb\_index\_serial\_scan\_concurrency {#tidb-index-serial-scan-concurrency}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、`serial scan`操作の並行性を設定するために使用されます。
- OLAPシナリオでは大きな値を使用し、OLTPシナリオでは小さな値を使用します。

### tidb\_init\_chunk\_size {#tidb-init-chunk-size}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `32`
- 範囲: `[1, 32]`
- 単位: 行
- この変数は、実行プロセス中の初期チャンクの行数を設定するために使用されます。

### tidb\_isolation\_read\_engines <span class="version-mark">v4.0で新規</span> {#tidb-isolation-read-engines-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: SESSION
- デフォルト値: `tikv,tiflash,tidb`
- この変数は、TiDBがデータを読み取る際に使用できるストレージエンジンのリストを設定するために使用されます。

### tidb\_last\_ddl\_info <span class="version-mark">v6.0.0で新規</span> {#tidb-last-ddl-info-span-class-version-mark-new-in-v6-0-0-span}

- Scope: SESSION
- デフォルト値: ""
- タイプ: 文字列
- この変数は読み取り専用です。現在のセッション内で最後のDDL操作の情報を取得するためにTiDB内部で使用されます。
  - "query": 最後のDDLクエリ文字列。
  - "seq\_num": 各DDL操作のシーケンス番号。DDL操作の順序を識別するために使用されます。

### tidb\_last\_query\_info <span class="version-mark">v4.0.14で新規</span> {#tidb-last-query-info-span-class-version-mark-new-in-v4-0-14-span}

- Scope: SESSION
- デフォルト値: ""
- この変数は読み取り専用です。TiDB内部で使用され、最後のDMLステートメントのトランザクション情報をクエリするために使用されます。情報には次のものが含まれます:
  - `txn_scope`: トランザクションのスコープ。`global`または`local`であることができます。
  - `start_ts`: トランザクションの開始タイムスタンプ。
  - `for_update_ts`: 以前に実行されたDMLステートメントの`for_update_ts`。これはTiDBの内部用語で、通常はこの情報を無視できます。
  - `error`: エラーメッセージ（あれば）。"

### tidb\_last\_txn\_info <span class="version-mark">v4.0.9で新規</span> {#tidb-last-txn-info-span-class-version-mark-new-in-v4-0-9-span}

- Scope: SESSION
- Type: String
- この変数は、現在のセッション内で最後のトランザクション情報を取得するために使用されます。読み取り専用変数です。トランザクション情報には次のものが含まれます。
  - トランザクションのスコープ。
  - 開始およびコミットTS。
  - トランザクションのコミットモード（2フェーズ、1フェーズ、または非同期コミットのいずれか）。
  - 非同期コミットまたは1フェーズコミットから2フェーズコミットへのトランザクションのフォールバック情報。
  - 発生したエラー。

### tidb\_last\_plan\_replayer\_token <span class="version-mark">v6.3.0で新規</span> {#tidb-last-plan-replayer-token-span-class-version-mark-new-in-v6-3-0-span}

- Scope: SESSION
- Type: String
- この変数は読み取り専用であり、現在のセッションでの最後の `PLAN REPLAYER DUMP` 実行の結果を取得するために使用されます。

### tidb\_load\_based\_replica\_read\_threshold <span class="version-mark">v7.0.0で新規</span> {#tidb-load-based-replica-read-threshold-span-class-version-mark-new-in-v7-0-0-span}

<CustomContent platform="tidb">

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `"1s"`
- 範囲: `[0s, 1h]`
- Type: String
- この変数は、負荷ベースのレプリカ読み取りをトリガーするためのしきい値を設定するために使用されます。リーダーノードの推定キュー時間がしきい値を超えると、TiDBはフォロワーノードからデータを読み取ることを優先します。フォーマットは、`"100ms"` や `"1s"` などの時間の長さです。詳細については、[Hotspot Issuesのトラブルシューティング](/troubleshoot-hot-spot-issues.md#scatter-read-hotspots)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `"1s"`
- 範囲: `[0s, 1h]`
- Type: String
- この変数は、負荷ベースのレプリカ読み取りをトリガーするためのしきい値を設定するために使用されます。リーダーノードの推定キュー時間がしきい値を超えると、TiDBはフォロワーノードからデータを読み取ることを優先します。フォーマットは、`"100ms"` や `"1s"` などの時間の長さです。詳細については、[Hotspot Issuesのトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues#scatter-read-hotspots)を参照してください。

</CustomContent>

### `tidb_lock_unchanged_keys` <span class="version-mark">v7.1.1で新規</span> {#tidb-lock-unchanged-keys-span-class-version-mark-new-in-v7-1-1-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- Type: Boolean
- デフォルト値: `ON`
- この変数は、次のシナリオで特定のキーをロックするかどうかを制御するために使用されます。`ON` に設定されている場合、これらのキーはロックされます。`OFF` に設定されている場合、これらのキーはロックされません。
  - `INSERT IGNORE` および `REPLACE` ステートメントの重複キー。v6.1.6より前では、これらのキーはロックされませんでした。この問題は [#42121](https://github.com/pingcap/tidb/issues/42121) で修正されました。
  - `UPDATE` ステートメントの一意キーで、キーの値が変更されていない場合。v6.5.2より前では、これらのキーはロックされませんでした。この問題は [#36438](https://github.com/pingcap/tidb/issues/36438) で修正されました。
- トランザクションの一貫性と合理性を維持するために、この値を変更することはお勧めしません。これら2つの修正によってTiDBのアップグレードが深刻なパフォーマンスの問題を引き起こし、ロックなしの動作が許容できる場合（前述の問題を参照）、この変数を `OFF` に設定することができます。

### tidb\_log\_file\_max\_days <span class="version-mark">v5.3.0で新規</span> {#tidb-log-file-max-days-span-class-version-mark-new-in-v5-3-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- Type: Integer
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`

<CustomContent platform="tidb">

- この変数は、現在のTiDBインスタンスでログを保持する最大日数を設定するために使用されます。その値は、構成ファイルの [`max-days`](/tidb-configuration-file.md#max-days) 設定の値にデフォルトで設定されます。変数値を変更すると、現在のTiDBインスタンスにのみ影響します。TiDBが再起動されると、変数値がリセットされ、構成値には影響しません。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、現在のTiDBインスタンスでログを保持する最大日数を設定するために使用されます。

</CustomContent>

### tidb\_low\_resolution\_tso {#tidb-low-resolution-tso}

- Scope: SESSION
- Type: Boolean
- デフォルト値: `OFF`
- この変数は、低精度TSO機能を有効にするかどうかを設定するために使用されます。この機能を有効にした後、新しいトランザクションはデータを読み取るために2秒ごとに更新されるタイムスタンプを使用します。
- 主な適用シナリオは、古いデータを読み取ることが許容される場合に、小規模な読み取り専用トランザクションのTSOの取得オーバーヘッドを削減することです。

### tidb\_max\_auto\_analyze\_time <span class="version-mark">v6.1.0で新規</span> {#tidb-max-auto-analyze-time-span-class-version-mark-new-in-v6-1-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- Type: Integer
- デフォルト値: `43200`
- 範囲: `[0, 2147483647]`
- 単位: 秒
- この変数は、自動 `ANALYZE` タスクの最大実行時間を指定するために使用されます。自動 `ANALYZE` タスクの実行時間が指定された時間を超えると、タスクは中止されます。この変数の値が `0` の場合、自動 `ANALYZE` タスクの最大実行時間に制限はありません。

### tidb\_max\_bytes\_before\_tiflash\_external\_group\_by <span class="version-mark">v7.0.0で新規</span> {#tidb-max-bytes-before-tiflash-external-group-by-span-class-version-mark-new-in-v7-0-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- Type: Integer
- デフォルト値: `-1`
- 範囲: `[-1, 9223372036854775807]`
- この変数は、TiFlashの `GROUP BY` でのハッシュ集約演算子の最大メモリ使用量をバイト単位で指定するために使用されます。メモリ使用量が指定された値を超えると、TiFlashはハッシュ集約演算子をディスクにスピルさせます。この変数の値が `-1` の場合、TiDBはこの変数をTiFlashに渡しません。この変数の値が `0` の場合、メモリ使用量に制限はありません。つまり、TiFlashのハッシュ集約演算子はスピルをトリガーしません。詳細については、[TiFlash Spill to Disk](/tiflash/tiflash-spill-disk.md)を参照してください。

<CustomContent platform="tidb">

> **Note:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、集約は通常、複数のTiFlashノードで分散して実行されます。この変数は、単一のTiFlashノードでの集約演算子の最大メモリ使用量を制御します。
> - この変数が `-1` に設定されている場合、TiFlashは、自身の構成項目 [`max_bytes_before_external_group_by`](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters) の値に基づいて集約演算子の最大メモリ使用量を決定します。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、集約は通常、複数のTiFlashノードで分散して実行されます。この変数は、単一のTiFlashノードでの集約演算子の最大メモリ使用量を制御します。
> - この変数が `-1` に設定されている場合、TiFlashは、自身の構成項目 `max_bytes_before_external_group_by` の値に基づいて集約演算子の最大メモリ使用量を決定します。

</CustomContent>

### tidb\_max\_bytes\_before\_tiflash\_external\_join <span class="version-mark">v7.0.0で新規</span> {#tidb-max-bytes-before-tiflash-external-join-span-class-version-mark-new-in-v7-0-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 9223372036854775807]`
- この変数は、TiFlashの`JOIN`におけるHash Join演算子の最大メモリ使用量をバイト単位で指定するために使用されます。メモリ使用量が指定された値を超えると、TiFlashはHash Join演算子をディスクにスピルします。この変数の値が`-1`の場合、TiDBはこの変数をTiFlashに渡しません。この変数の値が`0`以上の場合のみ、TiDBはこの変数をTiFlashに渡します。この変数の値が`0`の場合、メモリ使用量は無制限であり、つまり、TiFlash Hash Join演算子はスピルをトリガーしません。詳細については、[TiFlash Spill to Disk](/tiflash/tiflash-spill-disk.md)を参照してください。

<CustomContent platform="tidb">

> **Note:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、JOINは通常、複数のTiFlashノードで分散して実行されます。この変数は、単一のTiFlashノード上でJOIN演算子の最大メモリ使用量を制御します。
> - この変数が`-1`に設定されている場合、TiFlashは、単一のTiFlashノード上でのJOIN演算子の最大メモリ使用量を、独自の構成項目[`max_bytes_before_external_join`](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters)の値に基づいて決定します。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、JOINは通常、複数のTiFlashノードで分散して実行されます。この変数は、単一のTiFlashノード上でJOIN演算子の最大メモリ使用量を制御します。
> - この変数が`-1`に設定されている場合、TiFlashは、単一のTiFlashノード上でのJOIN演算子の最大メモリ使用量を、独自の構成項目`max_bytes_before_external_join`の値に基づいて決定します。

</CustomContent>

### tidb\_max\_bytes\_before\_tiflash\_external\_sort <span class="version-mark">v7.0.0で新規</span> {#tidb-max-bytes-before-tiflash-external-sort-span-class-version-mark-new-in-v7-0-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 9223372036854775807]`
- この変数は、TiFlashのTopNおよびSort演算子の最大メモリ使用量をバイト単位で指定するために使用されます。メモリ使用量が指定された値を超えると、TiFlashはTopNおよびSort演算子をディスクにスピルします。この変数の値が`-1`の場合、TiDBはこの変数をTiFlashに渡しません。この変数の値が`0`以上の場合のみ、TiDBはこの変数をTiFlashに渡します。この変数の値が`0`の場合、メモリ使用量は無制限であり、つまり、TiFlash TopNおよびSort演算子はスピルをトリガーしません。詳細については、[TiFlash Spill to Disk](/tiflash/tiflash-spill-disk.md)を参照してください。

<CustomContent platform="tidb">

> **Note:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、TopNおよびSortは通常、複数のTiFlashノードで分散して実行されます。この変数は、単一のTiFlashノード上でTopNおよびSort演算子の最大メモリ使用量を制御します。
> - この変数が`-1`に設定されている場合、TiFlashは、単一のTiFlashノード上でのTopNおよびSort演算子の最大メモリ使用量を、独自の構成項目[`max_bytes_before_external_sort`](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters)の値に基づいて決定します。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> - TiDBクラスターに複数のTiFlashノードがある場合、TopNおよびSortは通常、複数のTiFlashノードで分散して実行されます。この変数は、単一のTiFlashノード上でTopNおよびSort演算子の最大メモリ使用量を制御します。
> - この変数が`-1`に設定されている場合、TiFlashは、単一のTiFlashノード上でのTopNおよびSort演算子の最大メモリ使用量を、独自の構成項目`max_bytes_before_external_sort`の値に基づいて決定します。

</CustomContent>

### tidb\_max\_chunk\_size {#tidb-max-chunk-size}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1024`
- 範囲: `[32, 2147483647]`
- 単位: 行
- この変数は、実行プロセス中のチャンク内の最大行数を設定するために使用されます。大きすぎる値を設定すると、キャッシュの局所性の問題が発生する可能性があります。

### tidb\_max\_delta\_schema\_count <span class="version-mark">v2.1.18およびv3.0.5で新規</span> {#tidb-max-delta-schema-count-span-class-version-mark-new-in-v2-1-18-and-v3-0-5-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1024`
- 範囲: `[100, 16384]`
- この変数は、キャッシュされるスキーマバージョン（対応するバージョンの変更されたテーブルID）の最大数を設定するために使用されます。値の範囲は100〜16384です。

### tidb\_max\_paging\_size <span class="version-mark">v6.3.0で新規</span> {#tidb-max-paging-size-span-class-version-mark-new-in-v6-3-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `50000`
- 範囲: `[1, 9223372036854775807]`
- 単位: 行
- この変数は、コプロセッサーページングリクエストプロセス中の最大行数を設定するために使用されます。値を小さく設定すると、TiDBとTiKV間のRPC数が増加し、値を大きく設定すると、データの読み込みやフルテーブルスキャンなどの場合に過剰なメモリ使用量が発生する可能性があります。この変数のデフォルト値は、OLTPシナリオでのパフォーマンスを向上させるものです。アプリケーションがストレージエンジンとしてTiKVのみを使用する場合は、OLAPワークロードクエリを実行する際にこの変数の値を増やすことを検討してください。これにより、より良いパフォーマンスが得られる可能性があります。

### tidb\_max\_tiflash\_threads <span class="version-mark">v6.1.0で新規</span> {#tidb-max-tiflash-threads-span-class-version-mark-new-in-v6-1-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 256]`
- 単位: スレッド
- この変数は、TiFlashがリクエストを実行する最大同時実行数を設定するために使用されます。デフォルト値は`-1`であり、これはこのシステム変数が無効であり、最大同時実行数はTiFlash構成`profiles.default.max_threads`の設定に依存することを示します。値が`0`の場合、スレッドの最大数はTiFlashによって自動的に構成されます。

### tidb\_mem\_oom\_action <span class="version-mark">v6.1.0で新規</span> {#tidb-mem-oom-action-span-class-version-mark-new-in-v6-1-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙型
- デフォルト値: `CANCEL`
- 可能な値: `CANCEL`, `LOG`

<CustomContent platform="tidb">

- `tidb_mem_quota_query`で指定されたメモリクォータを超え、ディスクにスピルできない単一のSQLステートメントが発生した場合、TiDBが実行する操作を指定します。詳細については、[TiDB Memory Control](/configure-memory-usage.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- [`tidb_mem_quota_query`](#tidb_mem_quota_query)で指定されたメモリクォータを超え、ディスクにスピルできない単一のSQLステートメントが発生した場合、TiDBが実行する操作を指定します。

</CustomContent>

- デフォルト値は`CANCEL`ですが、TiDB v4.0.2およびそれ以前のバージョンでは、デフォルト値は`LOG`です。
- この設定は以前は`tidb.toml`オプション（`oom-action`）でしたが、TiDB v6.1.0からシステム変数に変更されました。

### tidb\_mem\_quota\_analyze <span class="version-mark">v6.1.0で新規</span> {#tidb-mem-quota-analyze-span-class-version-mark-new-in-v6-1-0-span}

> **Warning:**
>
> 現在、`ANALYZE`メモリクォータは実験的な機能であり、本番環境でメモリ統計が不正確になる可能性があります。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 9223372036854775807]`
- 単位: バイト
- この変数は、TiDBが統計情報を更新する際の最大メモリ使用量を制御します。このようなメモリ使用量は、手動で[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)を実行した場合や、TiDBがバックグラウンドで自動的に解析タスクを実行した場合に発生します。合計メモリ使用量がこの閾値を超えると、ユーザーが実行した`ANALYZE`は終了し、より低いサンプリング率を試すか、後で再試行するように促すエラーメッセージが報告されます。TiDBバックグラウンドでの自動タスクがメモリ閾値を超えて終了し、使用されたサンプリング率がデフォルト値よりも高い場合、TiDBはデフォルトのサンプリング率を使用して更新を再試行します。この変数の値が負またはゼロの場合、TiDBは手動および自動の更新タスクのメモリ使用量を制限しません。

> **Note:**
>
> `auto_analyze`は、TiDBクラスターでのみ`run-auto-analyze`がTiDB起動構成ファイルで有効になっている場合にトリガーされます。

### tidb\_mem\_quota\_apply\_cache <span class="version-mark">v5.0で新規</span> {#tidb-mem-quota-apply-cache-span-class-version-mark-new-in-v5-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `33554432` (32 MiB)
- 範囲: `[0, 9223372036854775807]`
- 単位: バイト
- この変数は、`Apply`オペレーターのローカルキャッシュのメモリ使用量の閾値を設定するために使用されます。
- `Apply`オペレーターのローカルキャッシュは、`Apply`オペレーターの計算を高速化するために使用されます。この変数を`0`に設定して`Apply`キャッシュ機能を無効にすることができます。

### tidb\_mem\_quota\_binding\_cache <span class="version-mark">v6.0.0で新規</span> {#tidb-mem-quota-binding-cache-span-class-version-mark-new-in-v6-0-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `67108864`
- 範囲: `[0, 2147483647]`
- 単位: バイト
- この変数は、バインディングのキャッシュに使用されるメモリの閾値を設定するために使用されます。
- システムが過剰なバインディングを作成またはキャプチャし、メモリスペースの過剰使用を引き起こす場合、TiDBはログで警告を返します。この場合、キャッシュは利用可能なすべてのバインディングを保持することができず、どのバインディングを保存するかを決定できません。このため、一部のクエリはバインディングを見逃す可能性があります。この問題を解決するために、この変数の値を増やすことで、バインディングのキャッシュに使用されるメモリを増やすことができます。このパラメータを変更した後は、`admin reload bindings`を実行してバインディングを再読み込みし、変更を検証する必要があります。

### tidb\_mem\_quota\_query {#tidb-mem-quota-query}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1073741824` (1 GiB)
- 範囲: `[-1, 9223372036854775807]`
- 単位: バイト

<CustomContent platform="tidb">

- TiDB v6.1.0より前のバージョンでは、これはセッションスコープ変数であり、初期値として`tidb.toml`から`mem-quota-query`の値を使用します。v6.1.0以降、`tidb_mem_quota_query`は`SESSION | GLOBAL`スコープの変数です。
- TiDB v6.5.0より前のバージョンでは、この変数は**クエリ**のメモリクォータの閾値を設定するために使用されます。実行中のクエリのメモリクォータが閾値を超えると、TiDBは[`tidb_mem_oom_action`](#tidb_mem_oom_action-new-in-v610)で定義された操作を実行します。
- TiDB v6.5.0以降のバージョンでは、この変数は**セッション**のメモリクォータの閾値を設定するために使用されます。実行中のセッションのメモリクォータが閾値を超えると、TiDBは[`tidb_mem_oom_action`](#tidb_mem_oom_action-new-in-v610)で定義された操作を実行します。TiDB v6.5.0以降、セッションのメモリ使用量にはセッション内のトランザクションによって消費されるメモリも含まれることに注意してください。TiDB v6.5.0以降のトランザクションメモリ使用量の制御動作については、[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)を参照してください。
- 変数値を`0`または`-1`に設定すると、メモリ閾値は正の無限大になります。128より小さい値を設定すると、値は`128`にデフォルトで設定されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- TiDB v6.1.0より前のバージョンでは、これはセッションスコープ変数です。v6.1.0以降、`tidb_mem_quota_query`は`SESSION | GLOBAL`スコープの変数です。
- TiDB v6.5.0より前のバージョンでは、この変数は**クエリ**のメモリクォータの閾値を設定するために使用されます。実行中のクエリのメモリクォータが閾値を超えると、TiDBは[`tidb_mem_oom_action`](#tidb_mem_oom_action-new-in-v610)で定義された操作を実行します。
- TiDB v6.5.0以降のバージョンでは、この変数は**セッション**のメモリクォータの閾値を設定するために使用されます。実行中のセッションのメモリクォータが閾値を超えると、TiDBは[`tidb_mem_oom_action`](#tidb_mem_oom_action-new-in-v610)で定義された操作を実行します。TiDB v6.5.0以降、セッションのメモリ使用量にはセッション内のトランザクションによって消費されるメモリも含まれることに注意してください。
- 変数値を`0`または`-1`に設定すると、メモリ閾値は正の無限大になります。128より小さい値を設定すると、値は`128`にデフォルトで設定されます。

</CustomContent>

### tidb\_memory\_debug\_mode\_alarm\_ratio {#tidb-memory-debug-mode-alarm-ratio}

- Scope: SESSION
- タイプ: 浮動小数点数
- デフォルト値: `0`
- この変数は、TiDBメモリデバッグモードで許可されるメモリ統計エラー値を表します。
- この変数は、TiDBの内部テストに使用されます。この変数を設定することは**推奨されません**。

### tidb\_memory\_debug\_mode\_min\_heap\_inuse {#tidb-memory-debug-mode-min-heap-inuse}

- Scope: SESSION
- タイプ: 整数
- デフォルト値: `0`
- この変数は、TiDBの内部テストに使用されます。この変数を設定することは**推奨されません**。この変数を有効にすると、TiDBのパフォーマンスに影響を与えます。
- このパラメータを構成した後、TiDBはメモリトラッキングの精度を分析するためにメモリデバッグモードに入ります。TiDBは、後続のSQLステートメントの実行中に頻繁にGCをトリガーし、実際のメモリ使用量とメモリ統計を比較します。現在のメモリ使用量が`tidb_memory_debug_mode_min_heap_inuse`を超え、メモリ統計エラーが`tidb_memory_debug_mode_alarm_ratio`を超える場合、TiDBは関連するメモリ情報をログとファイルに出力します。

### tidb\_memory\_usage\_alarm\_ratio {#tidb-memory-usage-alarm-ratio}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- デフォルト値: `0.7`
- 範囲: `[0.0, 1.0]`

<CustomContent platform="tidb">

- この変数は、tidb-serverメモリアラームをトリガーするメモリ使用率の比率を設定します。デフォルトでは、TiDBのメモリ使用量が合計メモリの70%を超え、[アラーム条件](/configure-memory-usage.md#trigger-the-alarm-of-excessive-memory-usage)のいずれかが満たされた場合、TiDBはアラームログを出力します。
- この変数を`0`または`1`に構成すると、メモリ閾値アラーム機能が無効になります。
- この変数を`0`より大きく`1`より小さい値に構成すると、メモリ閾値アラーム機能が有効になります。

  - システム変数[`tidb_server_memory_limit`](#tidb_server_memory_limit-new-in-v640)の値が`0`の場合、メモリアラーム閾値は`tidb_memory-usage-alarm-ratio * システムメモリサイズ`です。
  - システム変数`tidb_server_memory_limit`の値が0より大きい場合、メモリアラーム閾値は`tidb_memory-usage-alarm-ratio * tidb_server_memory_limit`です。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、[tidb-serverメモリアラーム](https://docs.pingcap.com/tidb/stable/configure-memory-usage#trigger-the-alarm-of-excessive-memory-usage)をトリガーするメモリ使用率の比率を設定します。
- この変数を`0`または`1`に構成すると、メモリ閾値アラーム機能が無効になります。
- この変数を`0`より大きく`1`より小さい値に構成すると、メモリ閾値アラーム機能が有効になります。

</CustomContent>

### tidb\_memory\_usage\_alarm\_keep\_record\_num <span class="version-mark">v6.4.0で新規</span> {#tidb-memory-usage-alarm-keep-record-num-span-class-version-mark-new-in-v6-4-0-span}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

- Scope: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `5`
- 範囲: `[1, 10000]`
- tidb-serverメモリ使用量がメモリアラーム閾値を超え、アラームをトリガーした場合、TiDBはデフォルトで最近の5回のアラームで生成されたステータスファイルのみを保持します。この数値をこの変数で調整することができます。

### tidb\_merge\_join\_concurrency {#tidb-merge-join-concurrency}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- 範囲: `[1, 256]`
- デフォルト値: `1`
- この変数は、クエリの実行時に`MergeJoin`オペレーターの並行性を設定します。
- この変数を設定することは**推奨されません**。この変数の値を変更すると、データの正確性に問題が発生する可能性があります。

### tidb\_merge\_partition\_stats\_concurrency {#tidb-merge-partition-stats-concurrency}

> **Warning:**
>
> この変数で制御される機能は、現在のTiDBバージョンでは完全に機能していません。デフォルト値を変更しないでください。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `1`
- この変数は、TiDBがパーティションテーブルの統計をマージする際の並行性を指定します。"

### tidb\_metric\_query\_range\_duration <span class="version-mark">v4.0で新規</span> {#tidb-metric-query-range-duration-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: SESSION
- タイプ: 整数
- デフォルト値: `60`
- 範囲: `[10, 216000]`
- 単位: 秒
- この変数は、`METRICS_SCHEMA`をクエリする際に生成されるPrometheusステートメントの範囲期間を設定するために使用されます。

### tidb\_metric\_query\_step <span class="version-mark">v4.0で新規</span> {#tidb-metric-query-step-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: SESSION
- タイプ: 整数
- デフォルト値: `60`
- 範囲: `[10, 216000]`
- 単位: 秒
- この変数は、`METRICS_SCHEMA`をクエリする際に生成されるPrometheusステートメントのステップを設定するために使用されます。

### tidb\_min\_paging\_size <span class="version-mark">v6.2.0で新規</span> {#tidb-min-paging-size-span-class-version-mark-new-in-v6-2-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `128`
- 範囲: `[1, 9223372036854775807]`
- 単位: 行
- この変数は、コプロセッサーページングリクエストプロセス中の最小行数を設定するために使用されます。値を小さく設定すると、TiDBとTiKV間のRPCリクエスト数が増加し、値を大きく設定すると、Limitを使用してクエリを実行する際にパフォーマンスが低下する可能性があります。この変数のデフォルト値は、OLTPシナリオでのパフォーマンスをOLAPシナリオよりも向上させます。アプリケーションがストレージエンジンとしてTiKVのみを使用する場合は、OLAPワークロードクエリを実行する際にこの変数の値を増やすことを検討してください。これにより、より良いパフォーマンスが得られる可能性があります。

![Paging size impact on TPCH](/media/paging-size-impact-on-tpch.png)

この図に示すように、[`tidb_enable_paging`](#tidb_enable_paging-new-in-v540)が有効になっている場合、`tidb_min_paging_size`および[`tidb_max_paging_size`](#tidb_max_paging_size-new-in-v630)の設定がTPCHのパフォーマンスに影響を与えます。垂直軸は実行時間であり、小さいほど良いです。

### tidb\_mpp\_store\_fail\_ttl {#tidb-mpp-store-fail-ttl}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 持続時間
- デフォルト値: `60s`
- 新しく開始されたTiFlashノードはサービスを提供しません。クエリが失敗しないように、TiDBは新しく開始されたTiFlashノードにクエリを送信することを制限します。この変数は、新しく開始されたTiFlashノードにリクエストが送信されない時間範囲を示します。

### tidb\_multi\_statement\_mode <span class="version-mark">v4.0.11で新規</span> {#tidb-multi-statement-mode-span-class-version-mark-new-in-v4-0-11-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙型
- デフォルト値: `OFF`
- 可能な値: `OFF`, `ON`, `WARN`
- この変数は、複数のクエリを同じ`COM_QUERY`呼び出しで実行するかどうかを制御します。
- SQLインジェクション攻撃の影響を軽減するために、TiDBはデフォルトで同じ`COM_QUERY`呼び出しで複数のクエリを実行することを防止します。この変数は、TiDBの以前のバージョンからのアップグレードパスの一部として使用することを意図しています。次の動作が適用されます:

| クライアント設定                  | `tidb_multi_statement_mode`の値 | 複数のステートメントが許可されますか？ |
| ------------------------- | ----------------------------- | ------------------- |
| Multiple Statements = ON  | OFF                           | はい                  |
| Multiple Statements = ON  | ON                            | はい                  |
| Multiple Statements = ON  | WARN                          | はい                  |
| Multiple Statements = OFF | OFF                           | いいえ                 |
| Multiple Statements = OFF | ON                            | はい                  |
| Multiple Statements = OFF | WARN                          | はい（+警告が返されます）       |

> **Note:**
>
> `OFF`のデフォルト値のみが安全と見なすことができます。`tidb_multi_statement_mode=ON`を設定する必要がある場合は、アプリケーションがTiDBの以前のバージョン向けに特別に設計されている場合です。アプリケーションが複数のステートメントサポートを必要とする場合は、`tidb_multi_statement_mode`オプションの代わりにクライアントライブラリが提供する設定を使用することをお勧めします。例:
>
> - [go-sql-driver](https://github.com/go-sql-driver/mysql#multistatements) (`multiStatements`)
> - [Connector/J](https://dev.mysql.com/doc/connector-j/en/connector-j-reference-configuration-properties.html) (`allowMultiQueries`)
> - PHP [mysqli](https://www.php.net/manual/en/mysqli.quickstart.multiple-statement.php) (`mysqli_multi_query`)

### tidb\_nontransactional\_ignore\_error <span class="version-mark">v6.1.0で新規</span> {#tidb-nontransactional-ignore-error-span-class-version-mark-new-in-v6-1-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、非トランザクションDMLステートメントでエラーが発生した場合にエラーを即座に返すかどうかを指定します。
- 値が`OFF`に設定されている場合、非トランザクションDMLステートメントは最初のエラーで直ちに停止し、エラーを返します。その後のバッチはすべてキャンセルされます。
- 値が`ON`に設定されている場合、バッチでエラーが発生しても、その後のバッチは引き続き実行されます。実行プロセス中に発生したすべてのエラーが結果で一緒に返されます。

### tidb\_opt\_agg\_push\_down {#tidb-opt-agg-push-down}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、オプティマイザが集約関数をJoin、Projection、およびUnionAllの前にプッシュダウンする最適化操作を実行するかどうかを設定するために使用されます。
- クエリで集約操作が遅い場合は、変数の値をONに設定できます。

### tidb\_opt\_broadcast\_cartesian\_join {#tidb-opt-broadcast-cartesian-join}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[0, 2]`
- ブロードキャストカルテシアンジョインを許可するかどうかを示します。
- `0`はブロードキャストカルテシアンジョインを許可しないことを意味します。`1`は、[`tidb_broadcast_join_threshold_count`](#tidb_broadcast_join_threshold_count-new-in-v50)に基づいて許可されることを意味します。`2`は、テーブルサイズがしきい値を超えていても常に許可されることを意味します。
- この変数はTiDB内部で使用され、その値を変更することは**推奨されません**。

### tidb\_opt\_concurrency\_factor {#tidb-opt-concurrency-factor}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- 範囲: `[0, 18446744073709551615]`
- デフォルト値: `3.0`
- TiDBでGolangゴルーチンを開始するCPUコストを示します。この変数は[Cosモデル](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。

### tidb\_opt\_copcpu\_factor {#tidb-opt-copcpu-factor}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- 範囲: `[0, 18446744073709551615]`
- デフォルト値: `3.0`
- TiKV Coprocessorが1行を処理するためのCPUコストを示します。この変数は[Cosモデル](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。

### tidb\_opt\_correlation\_exp\_factor {#tidb-opt-correlation-exp-factor}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: `[0, 2147483647]`
- 列の順序相関に基づいて行数を推定するメソッドが利用できない場合、ヒューリスティック推定メソッドが使用されます。この変数はヒューリスティックメソッドの動作を制御するために使用されます。
  - 値が0の場合、ヒューリスティックメソッドは使用されません。
  - 値が0より大きい場合:
    - 大きな値はヒューリスティックメソッドでインデックススキャンが使用される可能性が高いことを示します。
    - 小さな値はヒューリスティックメソッドでテーブルスキャンが使用される可能性が高いことを示します。

### tidb\_opt\_correlation\_threshold {#tidb-opt-correlation-threshold}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- デフォルト値: `0.9`
- 範囲: `[0, 1]`
- この変数は、列の順序相関を使用して行数を推定するかどうかを決定する閾値値を設定するために使用されます。現在の列と`handle`列の間の順序相関が閾値値を超える場合、このメソッドが有効になります。

### tidb\_opt\_cpu\_factor {#tidb-opt-cpu-factor}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- 範囲: `[0, 2147483647]`
- デフォルト値: `3.0`
- TiDBが1行を処理するためのCPUコストを示します。この変数は[Cosモデル](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。

### `tidb_opt_derive_topn` <span class="version-mark">v7.0.0で新規</span> {#tidb-opt-derive-topn-span-class-version-mark-new-in-v7-0-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- [ウィンドウ関数からのTopNまたはLimitの導出](/derive-topn-from-window.md)の最適化ルールを有効にするかどうかを制御します。

### tidb\_opt\_desc\_factor {#tidb-opt-desc-factor}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Float
- Range: `[0, 18446744073709551615]`
- Default value: `3.0`
- Indicates the cost for TiKV to scan one row from the disk in descending order. This variable is internally used in the [Cost Model](/cost-model.md), and it is **NOT** recommended to modify its value.

### tidb\_opt\_disk\_factor {#tidb-opt-disk-factor}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Float
- Range: `[0, 18446744073709551615]`
- Default value: `1.5`
- Indicates the I/O cost for TiDB to read or write one byte of data from or to the temporary disk. This variable is internally used in the [Cost Model](/cost-model.md), and it is **NOT** recommended to modify its value.

### tidb\_opt\_distinct\_agg\_push\_down {#tidb-opt-distinct-agg-push-down}

- Scope: SESSION
- Type: Boolean
- Default value: `OFF`
- This variable is used to set whether the optimizer executes the optimization operation of pushing down the aggregate function with `distinct` (such as `select count(distinct a) from t`) to Coprocessor.
- When the aggregate function with the `distinct` operation is slow in the query, you can set the variable value to `1`.

In the following example, before `tidb_opt_distinct_agg_push_down` is enabled, TiDB needs to read all data from TiKV and execute `distinct` on the TiDB side. After `tidb_opt_distinct_agg_push_down` is enabled, `distinct a` is pushed down to Coprocessor, and a `group by` column `test.t.a` is added to `HashAgg_5`.

```sql
mysql> desc select count(distinct a) from test.t;
+-------------------------+----------+-----------+---------------+------------------------------------------+
| id                      | estRows  | task      | access object | operator info                            |
+-------------------------+----------+-----------+---------------+------------------------------------------+
| StreamAgg_6             | 1.00     | root      |               | funcs:count(distinct test.t.a)->Column#4 |
| └─TableReader_10        | 10000.00 | root      |               | data:TableFullScan_9                     |
|   └─TableFullScan_9     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo           |
+-------------------------+----------+-----------+---------------+------------------------------------------+
3 rows in set (0.01 sec)

mysql> set session tidb_opt_distinct_agg_push_down = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> desc select count(distinct a) from test.t;
+---------------------------+----------+-----------+---------------+------------------------------------------+
| id                        | estRows  | task      | access object | operator info                            |
+---------------------------+----------+-----------+---------------+------------------------------------------+
| HashAgg_8                 | 1.00     | root      |               | funcs:count(distinct test.t.a)->Column#3 |
| └─TableReader_9           | 1.00     | root      |               | data:HashAgg_5                           |
|   └─HashAgg_5             | 1.00     | cop[tikv] |               | group by:test.t.a,                       |
|     └─TableFullScan_7     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo           |
+---------------------------+----------+-----------+---------------+------------------------------------------+
4 rows in set (0.00 sec)
```

### tidb\_opt\_enable\_correlation\_adjustment {#tidb-opt-enable-correlation-adjustment}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は、オプティマイザが列順序相関に基づいて行数を推定するかどうかを制御するために使用されます。

### tidb\_opt\_enable\_hash\_join <span class="version-mark">v6.5.6およびv7.1.2で新規</span> {#tidb-opt-enable-hash-join-span-class-version-mark-new-in-v6-5-6-and-v7-1-2-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は、オプティマイザがテーブルのハッシュ結合を選択するかどうかを制御するために使用されます。デフォルト値は`ON`です。`OFF`に設定されている場合、他の結合アルゴリズムが利用できない限り、オプティマイザは実行計画を生成する際にハッシュ結合を選択しません。
- システム変数`tidb_opt_enable_hash_join`と`HASH_JOIN`ヒントの両方が構成されている場合、`HASH_JOIN`ヒントが優先されます。`tidb_opt_enable_hash_join`が`OFF`に設定されていても、クエリで`HASH_JOIN`ヒントを指定すると、TiDBオプティマイザは引き続きハッシュ結合プランを強制します。

### tidb\_opt\_enable\_late\_materialization <span class="version-mark">v7.0.0で新規</span> {#tidb-opt-enable-late-materialization-span-class-version-mark-new-in-v7-0-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は、[TiFlashの遅延マテリアリゼーション](/tiflash/tiflash-late-materialization.md)機能を有効にするかどうかを制御するために使用されます。ただし、TiFlashの遅延マテリアリゼーションは[高速スキャンモード](/tiflash/use-fastscan.md)では有効になりません。
- この変数を`OFF`に設定してTiFlashの遅延マテリアリゼーション機能を無効にすると、`SELECT`ステートメントを処理する際に、TiFlashはフィルタ条件（`WHERE`句）を処理する前に必要な列のデータをすべてスキャンします。TiFlashの遅延マテリアリゼーション機能を有効にするためにこの変数を`ON`に設定すると、TiFlashは最初にTableScan演算子にプッシュダウンされたフィルタ条件に関連する列データをスキャンし、条件を満たす行をフィルタリングし、その後、これらの行の他の列のデータをスキャンしてさらなる計算を行うことで、IOスキャンとデータ処理の計算を減らします。

### tidb\_opt\_fix\_control <span class="version-mark">v6.5.7およびv7.1.0で新規</span> {#tidb-opt-fix-control-span-class-version-mark-new-in-v6-5-7-and-v7-1-0-span}

<CustomContent platform="tidb">

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: String
- Default value: `""`
- この変数は、オプティマイザの内部動作を制御するために使用されます。
- オプティマイザの動作は、ユーザーシナリオやSQLステートメントによって異なる場合があります。この変数はオプティマイザの動作の変更によるパフォーマンスの低下を防ぐために、オプティマイザに対してより細かい制御を提供します。
- 詳細については、[Optimizer Fix Controls](/optimizer-fix-controls.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: String
- Default value: `""`
- この変数は、オプティマイザの内部動作を制御するために使用されます。
- オプティマイザの動作は、ユーザーシナリオやSQLステートメントによって異なる場合があります。この変数はオプティマイザの動作の変更によるパフォーマンスの低下を防ぐために、オプティマイザに対してより細かい制御を提供します。
- 詳細については、[Optimizer Fix Controls](https://docs.pingcap.com/tidb/v7.1/optimizer-fix-controls)を参照してください。

</CustomContent>

### tidb\_opt\_force\_inline\_cte <span class="version-mark">v6.3.0で新規</span> {#tidb-opt-force-inline-cte-span-class-version-mark-new-in-v6-3-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は、セッション全体の共通テーブル式（CTE）をインライン化するかどうかを制御するために使用されます。デフォルト値は`OFF`であり、デフォルトではCTEのインライン化は強制されません。ただし、`MERGE()`ヒントを指定することでCTEをインライン化することができます。変数が`ON`に設定されている場合、このセッションのすべてのCTE（再帰CTEを除く）は強制的にインライン化されます。

### tidb\_opt\_advanced\_join\_hint <span class="version-mark">v7.0.0で新規</span> {#tidb-opt-advanced-join-hint-span-class-version-mark-new-in-v7-0-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は、[`HASH_JOIN()`ヒント](/optimizer-hints.md#hash_joint1_name--tl_name-)や[`MERGE_JOIN()`ヒント](/optimizer-hints.md#merge_joint1_name--tl_name-)などのJoin MethodヒントがJoin Reorder最適化プロセスに影響を与えるかどうかを制御するために使用されます。デフォルト値は`ON`であり、影響を与えません。`OFF`に設定されている場合、Join Methodヒントと`LEADING()`ヒントが同時に使用されるシナリオで競合が発生する可能性があります。

> **Note:**
>
> v7.0.0より前のバージョンの動作は、この変数を`OFF`に設定した場合と一致します。将来の互換性を確保するために、以前のバージョンからv7.0.0以降のクラスタにアップグレードする際に、この変数は`OFF`に設定されます。より柔軟なヒントの動作を得るためには、パフォーマンスの低下がない場合にこの変数を`ON`に切り替えることを強くお勧めします。

### tidb\_opt\_insubq\_to\_join\_and\_agg {#tidb-opt-insubq-to-join-and-agg}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は、サブクエリを結合および集計に変換する最適化ルールを有効にするかどうかを設定するために使用されます。
- たとえば、この最適化ルールを有効にした後、サブクエリは次のように変換されます：

  ```sql
  select * from t where t.a in (select aa from t1);
  ```

  サブクエリは次のように結合されます：

  ```sql
  select t.* from t, (select aa from t1 group by aa) tmp_t where t.a = tmp_t.aa;
  ```

  `t1`が`aa`列で`unique`および`not null`に制限されている場合、次のステートメントを使用できます（集計なし）。

  ```sql
  select t.* from t, t1 where t.a=t1.aa;
  ```

### tidb\_opt\_join\_reorder\_threshold {#tidb-opt-join-reorder-threshold}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `0`
- Range: `[0, 2147483647]`
- この変数は、TiDB Join Reorderアルゴリズムの選択を制御するために使用されます。Join Reorderに参加するノードの数がこの閾値よりも大きい場合、TiDBは貪欲アルゴリズムを選択し、この閾値よりも小さい場合、TiDBは動的プログラミングアルゴリズムを選択します。
- 現在、OLTPクエリではデフォルト値を維持することが推奨されています。OLAPクエリでは、変数の値を10〜15に設定することを推奨しています。これにより、OLAPシナリオでより良い接続順序を得ることができます。

### tidb\_opt\_limit\_push\_down\_threshold {#tidb-opt-limit-push-down-threshold}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `100`
- Range: `[0, 2147483647]`
- この変数は、LimitまたはTopN演算子をTiKVにプッシュダウンするかどうかを決定する閾値を設定するために使用されます。
- LimitまたはTopN演算子の値がこの閾値以下の場合、これらの演算子は強制的にTiKVにプッシュダウンされます。この変数は、誤った推定によるためにLimitまたはTopN演算子をTiKVにプッシュダウンできない問題を解決します。

### tidb\_opt\_memory\_factor {#tidb-opt-memory-factor}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Float
- Range: `[0, 2147483647]`
- Default value: `0.001`
- TiDBが1行を格納するためのメモリコストを示します。この変数は[Cost Model](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。

### tidb\_opt\_mpp\_outer\_join\_fixed\_build\_side <span class="version-mark">v5.1.0で新規</span> {#tidb-opt-mpp-outer-join-fixed-build-side-span-class-version-mark-new-in-v5-1-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- 変数値が`ON`の場合、左結合演算子は常に内部テーブルをビルドサイドとして使用し、右結合演算子は常に外部テーブルをビルドサイドとして使用します。`OFF`に設定すると、外部結合演算子はテーブルのどちらかをビルドサイドとして使用できます。

### tidb\_opt\_network\_factor {#tidb-opt-network-factor}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Float
- Range: `[0, 2147483647]`
- Default value: `1.0`
- 1バイトのデータをネットワークを介して転送するためのネットコストを示します。この変数は[Cost Model](/cost-model.md)で内部的に使用され、その値を変更することは**推奨されません**。"

### tidb\_opt\_ordering\_index\_selectivity\_threshold <span class="version-mark">v7.0.0で新規</span> {#tidb-opt-ordering-index-selectivity-threshold-span-class-version-mark-new-in-v7-0-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点
- デフォルト値: `0`
- 範囲: `[0, 1]`
- この変数は、SQLステートメントで`ORDER BY`および`LIMIT`句にフィルタ条件がある場合に、最適化プログラムがインデックスを選択する方法を制御するために使用されます。
- このようなクエリの場合、最適化プログラムは、`ORDER BY`および`LIMIT`句を満たす対応するインデックスを選択することを検討します（このインデックスがフィルタ条件を満たさない場合でも）。ただし、データ分布の複雑さにより、このシナリオで最適でないインデックスを選択する可能性があります。
- この変数は閾値を表します。フィルタ条件を満たすインデックスが存在し、その選択性の推定値がこの閾値よりも低い場合、最適化プログラムは`ORDER BY`および`LIMIT`を満たすために使用されるインデックスを選択せず、代わりにフィルタ条件を満たすインデックスを優先します。
- たとえば、変数が`0`に設定されている場合、最適化プログラムはデフォルトの動作を維持します。`1`に設定されている場合、最適化プログラムは常にフィルタ条件を満たすインデックスを選択し、`ORDER BY`および`LIMIT`句を両方満たすインデックスを選択しないようにします。
- 次の例では、テーブル`t`には合計1,000,000行あります。列`b`のインデックスを使用すると、その推定行数は約8,748なので、その選択性の推定値は約0.0087です。デフォルトでは、最適化プログラムは列`a`のインデックスを選択します。ただし、この変数を0.01に設定した後、列`b`のインデックスの選択性（0.0087）が0.01よりも低いため、最適化プログラムは列`b`のインデックスを選択します。

```sql
> EXPLAIN SELECT * FROM t WHERE b <= 9000 ORDER BY a LIMIT 1;
+-----------------------------------+---------+-----------+----------------------+--------------------+
| id                                | estRows | task      | access object        | operator info      |
+-----------------------------------+---------+-----------+----------------------+--------------------+
| Limit_12                          | 1.00    | root      |                      | offset:0, count:1  |
| └─Projection_25                   | 1.00    | root      |                      | test.t.a, test.t.b |
|   └─IndexLookUp_24                | 1.00    | root      |                      |                    |
|     ├─IndexFullScan_21(Build)     | 114.30  | cop[tikv] | table:t, index:ia(a) | keep order:true    |
|     └─Selection_23(Probe)         | 1.00    | cop[tikv] |                      | le(test.t.b, 9000) |
|       └─TableRowIDScan_22         | 114.30  | cop[tikv] | table:t              | keep order:false   |
+-----------------------------------+---------+-----------+----------------------+--------------------+

> SET SESSION tidb_opt_ordering_index_selectivity_threshold = 0.01;

> EXPLAIN SELECT * FROM t WHERE b <= 9000 ORDER BY a LIMIT 1;
+----------------------------------+---------+-----------+----------------------+-------------------------------------+
| id                               | estRows | task      | access object        | operator info                       |
+----------------------------------+---------+-----------+----------------------+-------------------------------------+
| TopN_9                           | 1.00    | root      |                      | test.t.a, offset:0, count:1         |
| └─IndexLookUp_20                 | 1.00    | root      |                      |                                     |
|   ├─IndexRangeScan_17(Build)     | 8748.62 | cop[tikv] | table:t, index:ib(b) | range:[-inf,9000], keep order:false |
|   └─TopN_19(Probe)               | 1.00    | cop[tikv] |                      | test.t.a, offset:0, count:1         |
|     └─TableRowIDScan_18          | 8748.62 | cop[tikv] | table:t              | keep order:false                    |
+----------------------------------+---------+-----------+----------------------+-------------------------------------+
```

### tidb\_opt\_prefer\_range\_scan <span class="version-mark">v5.0で新規</span> {#tidb-opt-prefer-range-scan-span-class-version-mark-new-in-v5-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数の値を`ON`に設定した後、オプティマイザーは常にフルテーブルスキャンよりも範囲スキャンを選択します。
- 次の例では、`tidb_opt_prefer_range_scan`を有効にする前は、TiDBオプティマイザーがフルテーブルスキャンを実行します。`tidb_opt_prefer_range_scan`を有効にした後、オプティマイザーはインデックス範囲スキャンを選択します。

```sql
explain select * from t where age=5;
+-------------------------+------------+-----------+---------------+-------------------+
| id                      | estRows    | task      | access object | operator info     |
+-------------------------+------------+-----------+---------------+-------------------+
| TableReader_7           | 1048576.00 | root      |               | data:Selection_6  |
| └─Selection_6           | 1048576.00 | cop[tikv] |               | eq(test.t.age, 5) |
|   └─TableFullScan_5     | 1048576.00 | cop[tikv] | table:t       | keep order:false  |
+-------------------------+------------+-----------+---------------+-------------------+
3 rows in set (0.00 sec)

set session tidb_opt_prefer_range_scan = 1;

explain select * from t where age=5;
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
| id                            | estRows    | task      | access object               | operator info                 |
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
| IndexLookUp_7                 | 1048576.00 | root      |                             |                               |
| ├─IndexRangeScan_5(Build)     | 1048576.00 | cop[tikv] | table:t, index:idx_age(age) | range:[5,5], keep order:false |
| └─TableRowIDScan_6(Probe)     | 1048576.00 | cop[tikv] | table:t                     | keep order:false              |
+-------------------------------+------------+-----------+-----------------------------+-------------------------------+
3 rows in set (0.00 sec)
```

### tidb\_opt\_prefix\_index\_single\_scan <span class="version-mark">v6.4.0で新規</span> {#tidb-opt-prefix-index-single-scan-span-class-version-mark-new-in-v6-4-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ON`
- この変数は、TiDBオプティマイザがいくつかのフィルタ条件をプレフィックスインデックスにプッシュダウンして不要なテーブル検索を回避し、クエリのパフォーマンスを向上させるかどうかを制御します。
- この変数の値が`ON`に設定されている場合、いくつかのフィルタ条件がプレフィックスインデックスにプッシュダウンされます。`col`列がテーブルのインデックスプレフィックス列であるとします。クエリ内の`col is null`または`col is not null`条件は、テーブル検索のフィルタ条件ではなく、インデックスのフィルタ条件として処理されるため、不要なテーブル検索が回避されます。

<details>
<summary><code>tidb_opt_prefix_index_single_scan</code>の使用例</summary>

プレフィックスインデックスを持つテーブルを作成します:

```sql
CREATE TABLE t (a INT, b VARCHAR(10), c INT, INDEX idx_a_b(a, b(5)));
```

`tidb_opt_prefix_index_single_scan`を無効にします。

```sql
SET tidb_opt_prefix_index_single_scan = 'OFF';
```

次のクエリについて、実行計画はプレフィックスインデックス `idx_a_b` を使用しますが、テーブルのルックアップが必要です（`IndexLookUp` オペレーターが表示されます）。

```sql
EXPLAIN FORMAT='brief' SELECT COUNT(1) FROM t WHERE a = 1 AND b IS NOT NULL;
+-------------------------------+---------+-----------+------------------------------+-------------------------------------------------------+
| id                            | estRows | task      | access object                | operator info                                         |
+-------------------------------+---------+-----------+------------------------------+-------------------------------------------------------+
| HashAgg                       | 1.00    | root      |                              | funcs:count(Column#8)->Column#5                       |
| └─IndexLookUp                 | 1.00    | root      |                              |                                                       |
|   ├─IndexRangeScan(Build)     | 99.90   | cop[tikv] | table:t, index:idx_a_b(a, b) | range:[1 -inf,1 +inf], keep order:false, stats:pseudo |
|   └─HashAgg(Probe)            | 1.00    | cop[tikv] |                              | funcs:count(1)->Column#8                              |
|     └─Selection               | 99.90   | cop[tikv] |                              | not(isnull(test.t.b))                                 |
|       └─TableRowIDScan        | 99.90   | cop[tikv] | table:t                      | keep order:false, stats:pseudo                        |
+-------------------------------+---------+-----------+------------------------------+-------------------------------------------------------+
6 rows in set (0.00 sec)
```

`tidb_opt_prefix_index_single_scan`を有効にします：

```sql
SET tidb_opt_prefix_index_single_scan = 'ON';
```

この変数を有効にした後、次のクエリについて、実行計画はプレフィックスインデックス `idx_a_b` を使用しますが、テーブルの検索は必要ありません。

```sql
EXPLAIN FORMAT='brief' SELECT COUNT(1) FROM t WHERE a = 1 AND b IS NOT NULL;
+--------------------------+---------+-----------+------------------------------+-------------------------------------------------------+
| id                       | estRows | task      | access object                | operator info                                         |
+--------------------------+---------+-----------+------------------------------+-------------------------------------------------------+
| StreamAgg                | 1.00    | root      |                              | funcs:count(Column#7)->Column#5                       |
| └─IndexReader            | 1.00    | root      |                              | index:StreamAgg                                       |
|   └─StreamAgg            | 1.00    | cop[tikv] |                              | funcs:count(1)->Column#7                              |
|     └─IndexRangeScan     | 99.90   | cop[tikv] | table:t, index:idx_a_b(a, b) | range:[1 -inf,1 +inf], keep order:false, stats:pseudo |
+--------------------------+---------+-----------+------------------------------+-------------------------------------------------------+
4 rows in set (0.00 sec)
```

</details>

### tidb\_opt\_projection\_push\_down <span class="version-mark">v6.1.0で新規</span> {#tidb-opt-projection-push-down-span-class-version-mark-new-in-v6-1-0-span}

- スコープ: SESSION
- タイプ: ブール値
- デフォルト値: `OFF`
- オプティマイザーが`Projection`をTiKVまたはTiFlashコプロセッサーにプッシュダウンするかどうかを指定します。

### tidb\_opt\_range\_max\_size <span class="version-mark">v6.4.0で新規</span> {#tidb-opt-range-max-size-span-class-version-mark-new-in-v6-4-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `67108864` (64 MiB)
- スコープ: `[0, 9223372036854775807]`
- 単位: バイト
- この変数は、オプティマイザーがスキャン範囲を構築するためのメモリ使用量の上限を設定するために使用されます。変数値が`0`の場合、スキャン範囲の構築にメモリ制限はありません。正確なスキャン範囲の構築によって上限を超えるメモリを消費する場合、オプティマイザーはより緩和されたスキャン範囲（`[[NULL,+inf]]`など）を使用します。実行計画が正確なスキャン範囲を使用しない場合、この変数の値を増やしてオプティマイザーに正確なスキャン範囲を構築させることができます。

この変数の使用例は次のとおりです：

<details>
<summary><code>tidb_opt_range_max_size</code>の使用例</summary>

この変数のデフォルト値を表示します。結果から、オプティマイザーがスキャン範囲を構築するために最大64 MiBのメモリを使用することがわかります。

```sql
SELECT @@tidb_opt_range_max_size;
```

```sql
+----------------------------+
| @@tidb_opt_range_max_size |
+----------------------------+
| 67108864                   |
+----------------------------+
1 row in set (0.01 sec)
```

```sql
EXPLAIN SELECT * FROM t use index (idx) WHERE a IN (10,20,30) AND b IN (40,50,60);
```

64 MiBメモリの上限では、オプティマイザは次の正確なスキャン範囲を構築します `[10 40,10 40], [10 50,10 50], [10 60,10 60], [20 40,20 40], [20 50,20 50], [20 60,20 60], [30 40,30 40], [30 50,30 50], [30 60,30 60]`、以下の実行計画結果に示すように。

```sql
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                            | estRows | task      | access object            | operator info                                                                                                                                                               |
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| IndexLookUp_7                 | 0.90    | root      |                          |                                                                                                                                                                             |
| ├─IndexRangeScan_5(Build)     | 0.90    | cop[tikv] | table:t, index:idx(a, b) | range:[10 40,10 40], [10 50,10 50], [10 60,10 60], [20 40,20 40], [20 50,20 50], [20 60,20 60], [30 40,30 40], [30 50,30 50], [30 60,30 60], keep order:false, stats:pseudo |
| └─TableRowIDScan_6(Probe)     | 0.90    | cop[tikv] | table:t                  | keep order:false, stats:pseudo                                                                                                                                              |
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
3 rows in set (0.00 sec)
```

最適化プログラムがスキャン範囲を構築するためのメモリ使用量の上限を1500バイトに設定します。

```sql
SET @@tidb_opt_range_max_size = 1500;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

```sql
EXPLAIN SELECT * FROM t USE INDEX (idx) WHERE a IN (10,20,30) AND b IN (40,50,60);
```

1500バイトのメモリ制限では、オプティマイザーはより緩和されたスキャン範囲 `[10,10], [20,20], [30,30]` を構築し、正確なスキャン範囲を構築するために必要なメモリ使用量が `tidb_opt_range_max_size` の制限を超えることをユーザーに警告します。

```sql
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------+
| id                            | estRows | task      | access object            | operator info                                                   |
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------+
| IndexLookUp_8                 | 0.09    | root      |                          |                                                                 |
| ├─Selection_7(Build)          | 0.09    | cop[tikv] |                          | in(test.t.b, 40, 50, 60)                                        |
| │ └─IndexRangeScan_5          | 30.00   | cop[tikv] | table:t, index:idx(a, b) | range:[10,10], [20,20], [30,30], keep order:false, stats:pseudo |
| └─TableRowIDScan_6(Probe)     | 0.09    | cop[tikv] | table:t                  | keep order:false, stats:pseudo                                  |
+-------------------------------+---------+-----------+--------------------------+-----------------------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

```sql
SHOW WARNINGS;
```

```sql
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                     |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1105 | Memory capacity of 1500 bytes for 'tidb_opt_range_max_size' exceeded when building ranges. Less accurate ranges such as full range are chosen |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

次に、メモリ使用量の上限を100バイトに設定します。

```sql
set @@tidb_opt_range_max_size = 100;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

```sql
EXPLAIN SELECT * FROM t USE INDEX (idx) WHERE a IN (10,20,30) AND b IN (40,50,60);
```

100バイトのメモリ制限では、オプティマイザーは`IndexFullScan`を選択し、正確なスキャン範囲を構築するために必要なメモリが`tidb_opt_range_max_size`の制限を超えることをユーザーに警告しています。

```sql
+-------------------------------+----------+-----------+--------------------------+----------------------------------------------------+
| id                            | estRows  | task      | access object            | operator info                                      |
+-------------------------------+----------+-----------+--------------------------+----------------------------------------------------+
| IndexLookUp_8                 | 8000.00  | root      |                          |                                                    |
| ├─Selection_7(Build)          | 8000.00  | cop[tikv] |                          | in(test.t.a, 10, 20, 30), in(test.t.b, 40, 50, 60) |
| │ └─IndexFullScan_5           | 10000.00 | cop[tikv] | table:t, index:idx(a, b) | keep order:false, stats:pseudo                     |
| └─TableRowIDScan_6(Probe)     | 8000.00  | cop[tikv] | table:t                  | keep order:false, stats:pseudo                     |
+-------------------------------+----------+-----------+--------------------------+----------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

```sql
SHOW WARNINGS;
```

```sql
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                     |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1105 | Memory capacity of 100 bytes for 'tidb_opt_range_max_size' exceeded when building ranges. Less accurate ranges such as full range are chosen |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

</details>

### tidb\_opt\_scan\_factor {#tidb-opt-scan-factor}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Float
- Range: `[0, 2147483647]`
- Default value: `1.5`
- Indicates the cost for TiKV to scan one row of data from the disk in ascending order. This variable is internally used in the [Cost Model](/cost-model.md), and it is **NOT** recommended to modify its value.

### tidb\_opt\_seek\_factor {#tidb-opt-seek-factor}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Float
- Range: `[0, 2147483647]`
- Default value: `20`
- Indicates the start-up cost for TiDB to request data from TiKV. This variable is internally used in the [Cost Model](/cost-model.md), and it is **NOT** recommended to modify its value.

### tidb\_opt\_skew\_distinct\_agg <span class="version-mark">New in v6.2.0</span> {#tidb-opt-skew-distinct-agg-span-class-version-mark-new-in-v6-2-0-span}

> **Note:**
>
> The query performance optimization by enabling this variable is effective **only for TiFlash**.

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- This variable sets whether the optimizer rewrites the aggregate functions with `DISTINCT` to the two-level aggregate functions, such as rewriting `SELECT b, COUNT(DISTINCT a) FROM t GROUP BY b` to `SELECT b, COUNT(a) FROM (SELECT b, a FROM t GROUP BY b, a) t GROUP BY b`. When the aggregation column has serious skew and the `DISTINCT` column has many different values, this rewriting can avoid the data skew in the query execution and improve the query performance.

### tidb\_opt\_three\_stage\_distinct\_agg <span class="version-mark">New in v6.3.0</span> {#tidb-opt-three-stage-distinct-agg-span-class-version-mark-new-in-v6-3-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- This variable specifies whether to rewrite a `COUNT(DISTINCT)` aggregation into a three-stage aggregation in MPP mode.
- This variable currently applies to an aggregation that only contains one `COUNT(DISTINCT)`.

### tidb\_opt\_tiflash\_concurrency\_factor {#tidb-opt-tiflash-concurrency-factor}

- Scope: SESSION | GLOBAL
- Persists to cluster: YES
- Type: Float
- Range: `[0, 2147483647]`
- Default value: `24.0`
- Indicates the concurrency number of TiFlash computation. This variable is internally used in the Cost Model, and it is NOT recommended to modify its value.

### tidb\_opt\_write\_row\_id {#tidb-opt-write-row-id}

> **Note:**
>
> This TiDB variable is not applicable to TiDB Cloud.

- Scope: SESSION
- Type: Boolean
- Default value: `OFF`
- This variable is used to control whether to allow `INSERT`, `REPLACE`, and `UPDATE` statements to operate on the `_tidb_rowid` column. This variable can be used only when you import data using TiDB tools.

### tidb\_optimizer\_selectivity\_level {#tidb-optimizer-selectivity-level}

- Scope: SESSION
- Type: Integer
- Default value: `0`
- Range: `[0, 2147483647]`
- This variable controls the iteration of the optimizer's estimation logic. After changing the value of this variable, the estimation logic of the optimizer will change greatly. Currently, `0` is the only valid value. It is not recommended to set it to other values.

### tidb\_partition\_prune\_mode <span class="version-mark">New in v5.1</span> {#tidb-partition-prune-mode-span-class-version-mark-new-in-v5-1-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Enumeration
- Default value: `dynamic`
- Possible values: `static`, `dynamic`, `static-only`, `dynamic-only`
- Specifies whether to use `dynamic` or `static` mode for partitioned tables. Note that dynamic partitioning is effective only after full table-level statistics, or GlobalStats, are collected. Before GlobalStats are collected, TiDB will use the `static` mode instead. For detailed information about GlobalStats, see [Collect statistics of partitioned tables in dynamic pruning mode](/statistics.md#collect-statistics-of-partitioned-tables-in-dynamic-pruning-mode). For details about the dynamic pruning mode, see [Dynamic Pruning Mode for Partitioned Tables](/partitioned-table.md#dynamic-pruning-mode).

### tidb\_persist\_analyze\_options <span class="version-mark">New in v5.4.0</span> {#tidb-persist-analyze-options-span-class-version-mark-new-in-v5-4-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- This variable controls whether to enable the [ANALYZE configuration persistence](/statistics.md#persist-analyze-configurations) feature.

### tidb\_pessimistic\_txn\_fair\_locking <span class="version-mark">New in v7.0.0</span> {#tidb-pessimistic-txn-fair-locking-span-class-version-mark-new-in-v7-0-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- Determines whether to use enhanced pessimistic locking wake-up model for pessimistic transactions. This model strictly controls the wake-up order of pessimistic transactions in the pessimistic locking single-point conflict scenarios to avoid unnecessary wake-ups. It greatly reduces the uncertainty brought by the randomness of the existing wake-up mechanism. If you encounter frequent single-point pessimistic locking conflicts in your business scenario (such as frequent updates to the same row of data), and thus cause frequent statement retries, high tail latency, or even occasional `pessimistic lock retry limit reached` errors, you can try to enable this variable to solve the problem.
- This variable is disabled by default for TiDB clusters that are upgraded from versions earlier than v7.0.0 to v7.0.0 or later versions.

> **Note:**
>
> - Depending on the specific business scenario, enabling this option might cause a certain degree of throughput reduction (average latency increase) for transactions with frequent lock conflicts.
> - This option only takes effect on statements that need to lock a single key. If a statement needs to lock multiple rows at the same time, this option will not take effect on such statements.
> - This feature is introduced in v6.6.0 by the [`tidb_pessimistic_txn_aggressive_locking`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_pessimistic_txn_aggressive_locking-new-in-v660) variable, which is disabled by default.

### tidb\_placement\_mode <span class="version-mark">New in v6.0.0</span> {#tidb-placement-mode-span-class-version-mark-new-in-v6-0-0-span}

> **Note:**
>
> This variable is read-only for [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless).

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Enumeration
- Default value: `STRICT`
- Possible values: `STRICT`, `IGNORE`
- This variable controls whether DDL statements ignore the [placement rules specified in SQL](/placement-rules-in-sql.md). When the variable value is `IGNORE`, all placement rule options are ignored.
- It is intended to be used by logical dump/restore tools to ensure that tables can always be created even if invalid placement rules are assigned. This is similar to how mysqldump writes `SET FOREIGN_KEY_CHECKS=0;` to the start of every dump file.

### `tidb_plan_cache_invalidation_on_fresh_stats` <span class="version-mark">New in v7.1.0</span> {#tidb-plan-cache-invalidation-on-fresh-stats-span-class-version-mark-new-in-v7-1-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- This variable controls whether to invalidate the plan cache automatically when statistics on related tables are updated.
- After enabling this variable, plan cache can make use of statistics more sufficiently to generate execution plans. For example:
  - If execution plans are generated before statistics are available, plan cache re-generates execution plans once the statistics are available.
  - If the data distribution of a table changes, causing the previously optimal execution plan to become non-optimal, plan cache re-generates execution plans after the statistics are re-collected.
- This variable is disabled by default for TiDB clusters that are upgraded from a version earlier than v7.1.0 to v7.1.0 or later.

### `tidb_plan_cache_max_plan_size` <span class="version-mark">New in v7.1.0</span> {#tidb-plan-cache-max-plan-size-span-class-version-mark-new-in-v7-1-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Default value: `2097152` (which is 2 MB)
- Range: `[0, 9223372036854775807]`, in bytes. The memory format with the units "KB|MB|GB|TB" is also supported. `0` means no limit.
- This variable controls the maximum size of a plan that can be cached in prepared or non-prepared plan cache. If the size of a plan exceeds this value, the plan will not be cached. For more details, see [Memory management of prepared plan cache](/sql-prepared-plan-cache.md#memory-management-of-prepared-plan-cache) and [Non-prepared plan cache](/sql-plan-management.md#usage).

### tidb\_pprof\_sql\_cpu <span class="version-mark">v4.0で新規</span> {#tidb-pprof-sql-cpu-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 1]`
- この変数は、対応するSQLステートメントをプロファイル出力で識別し、パフォーマンスの問題を特定およびトラブルシューティングするかどうかを制御するために使用されます。

### tidb\_prefer\_broadcast\_join\_by\_exchange\_data\_size <span class="version-mark">v7.1.0で新規</span> {#tidb-prefer-broadcast-join-by-exchange-data-size-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `OFF`
- この変数は、TiDBが[MPP Hash Joinアルゴリズム](/tiflash/use-tiflash-mpp-mode.md#algorithm-support-for-the-mpp-mode)を選択する際にネットワーク転送のオーバーヘッドが最小限になるアルゴリズムを使用するかどうかを制御します。この変数が有効になっている場合、TiDBはネットワークで交換されるデータのサイズを`Broadcast Hash Join`および`Shuffled Hash Join`それぞれで推定し、その後、サイズが小さい方を選択します。
- この変数が有効になった後は、[`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50)および[`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50)は効果を持ちません。

### tidb\_prepared\_plan\_cache\_memory\_guard\_ratio <span class="version-mark">v6.1.0で新規</span> {#tidb-prepared-plan-cache-memory-guard-ratio-span-class-version-mark-new-in-v6-1-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- デフォルト値: `0.1`
- 範囲: `[0, 1]`
- プリペアドプランキャッシュがメモリ保護メカニズムをトリガーする閾値。詳細については、[プリペアドプランキャッシュのメモリ管理](/sql-prepared-plan-cache.md)を参照してください。
- この設定は以前は`tidb.toml`オプション（`prepared-plan-cache.memory-guard-ratio`）でしたが、TiDB v6.1.0からシステム変数に変更されました。

### tidb\_prepared\_plan\_cache\_size <span class="version-mark">v6.1.0で新規</span> {#tidb-prepared-plan-cache-size-span-class-version-mark-new-in-v6-1-0-span}

> **Warning:**
>
> v7.1.0から、この変数は非推奨となりました。代わりに[`tidb_session_plan_cache_size`](#tidb_session_plan_cache_size-new-in-v710)を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[1, 100000]`
- セッションでキャッシュできるプランの最大数。詳細については、[プリペアドプランキャッシュのメモリ管理](/sql-prepared-plan-cache.md)を参照してください。
- この設定は以前は`tidb.toml`オプション（`prepared-plan-cache.capacity`）でしたが、TiDB v6.1.0からシステム変数に変更されました。

### tidb\_projection\_concurrency {#tidb-projection-concurrency}

> **Warning:**
>
> v5.0から、この変数は非推奨となりました。代わりに[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[-1, 256]`
- 単位: スレッド
- この変数は`Projection`オペレーターの並列処理を設定するために使用されます。
- `-1`の値は、`tidb_executor_concurrency`の値が代わりに使用されることを意味します。

### tidb\_query\_log\_max\_len {#tidb-query-log-max-len}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `4096`（4 KiB）
- 範囲: `[0, 1073741824]`
- 単位: バイト
- SQLステートメントの出力の最大長。ステートメントの出力長が`tidb_query_log_max_len`の値よりも大きい場合、ステートメントは切り捨てられて出力されます。
- この設定は以前は`tidb.toml`オプション（`log.query-log-max-len`）としても利用可能でしたが、TiDB v6.1.0からはシステム変数のみとなりました。

### tidb\_rc\_read\_check\_ts <span class="version-mark">v6.0.0で新規</span> {#tidb-rc-read-check-ts-span-class-version-mark-new-in-v6-0-0-span}

> **Warning:**
>
> - この機能は[`replica-read`](#tidb_replica_read-new-in-v40)と互換性がありません。`tidb_rc_read_check_ts`と`replica-read`を同時に有効にしないでください。
> - クライアントがカーソルを使用する場合、前のバッチの返されたデータがすでにクライアントによって使用されており、ステートメントが最終的に失敗する可能性があるため、`tidb_rc_read_check_ts`を有効にすることは推奨されません。
> - v7.0.0から、この変数は、プリペアドステートメントプロトコルを使用するカーソルフェッチ読み取りモードにはもはや有効ではありません。

- スコープ: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数はタイムスタンプの取得を最適化するために使用され、読み取りコミットの分離レベルで読み書きの競合がまれなシナリオに適しています。この変数を有効にすると、グローバルタイムスタンプの取得の遅延とコストを回避し、トランザクションレベルの読み取りレイテンシを最適化できます。
- 読み書きの競合が深刻な場合、この機能を有効にすると、グローバルタイムスタンプの取得のコストとレイテンシが増加し、パフォーマンスの低下が発生する可能性があります。詳細については、[Read Committed isolation level](/transaction-isolation-levels.md#read-committed-isolation-level)を参照してください。

### tidb\_rc\_write\_check\_ts <span class="version-mark">v6.3.0で新規</span> {#tidb-rc-write-check-ts-span-class-version-mark-new-in-v6-3-0-span}

> **Warning:**
>
> この機能は現在、[`replica-read`](#tidb_replica_read-new-in-v40)と互換性がありません。この変数を有効にした後、クライアントが送信するすべてのリクエストは`replica-read`を使用できません。したがって、`tidb_rc_write_check_ts`と`replica-read`を同時に有効にしないでください。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数はタイムスタンプの取得を最適化するために使用され、悲観的トランザクションの`READ-COMMITTED`分離レベルでポイントライト競合が少ないシナリオに適しています。この変数を有効にすると、ポイントライトステートメントの実行中にグローバルタイムスタンプを取得する遅延とオーバーヘッドを回避できます。現在、この変数は3種類のポイントライトステートメントに適用されます：`UPDATE`、`DELETE`、および`SELECT ...... FOR UPDATE`。ポイントライトステートメントとは、プライマリキーまたはユニークキーをフィルタ条件として使用し、最終的な実行オペレーターに`POINT-GET`を含む書き込みステートメントを指します。
- ポイントライト競合が深刻な場合、この変数を有効にすると、追加のオーバーヘッドとレイテンシが増加し、パフォーマンスの低下が発生する可能性があります。詳細については、[Read Committed isolation level](/transaction-isolation-levels.md#read-committed-isolation-level)を参照してください。

### tidb\_read\_consistency <span class="version-mark">v5.4.0で新規</span> {#tidb-read-consistency-span-class-version-mark-new-in-v5-4-0-span}

- スコープ: SESSION
- タイプ: 文字列
- デフォルト値: `strict`
- この変数は、自動コミット読み取りステートメントの読み取り一貫性を制御するために使用されます。
- 変数値が`weak`に設定されている場合、読み取りステートメントで遭遇したロックは直接スキップされ、読み取り実行がより速くなる可能性があります。これは弱い一貫性読み取りモードです。ただし、トランザクションのセマンティクス（原子性など）および分散一貫性（直線性など）は保証されません。
- 自動コミット読み取りが迅速に返す必要があるユーザーシナリオでは、弱い一貫性読み取り結果が許容される場合、弱い一貫性読み取りモードを使用できます。

### tidb\_read\_staleness <span class="version-mark">v5.4.0で新規</span> {#tidb-read-staleness-span-class-version-mark-new-in-v5-4-0-span}

- スコープ: SESSION
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[-2147483648, 0]`
- この変数は、TiDBが現在のセッションで読み取ることができる歴史データの時間範囲を設定するために使用されます。この変数で許可される範囲からできるだけ新しいタイムスタンプをTiDBが選択し、その後のすべての読み取り操作はこのタイムスタンプに対して行われます。たとえば、この変数の値が`-5`に設定されている場合、TiKVが対応する歴史バージョンのデータを持っている条件下で、TiDBは5秒以内の時間範囲でできるだけ新しいタイムスタンプを選択します。

### tidb\_record\_plan\_in\_slow\_log {#tidb-record-plan-in-slow-log}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- スコープ: GLOBAL
- クラスターに永続化: いいえ、接続している現在のTiDBインスタンスにのみ適用されます。
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は遅いクエリの実行計画を遅いログに含めるかどうかを制御するために使用されます。

### tidb\_redact\_log {#tidb-redact-log}

> **Note:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は、TiDBログとスローログに記録されるSQLステートメント内のユーザー情報を非表示にするかどうかを制御します。
- 変数を`1`に設定すると、ユーザー情報が非表示になります。たとえば、実行されたSQLステートメントが`insert into t values (1,2)`の場合、ログには`insert into t values (?,?)`と記録されます。

### tidb\_regard\_null\_as\_point <span class="version-mark">v5.4.0で新規</span> {#tidb-regard-null-as-point-span-class-version-mark-new-in-v5-4-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、オプティマイザが、nullと等価性を持つクエリ条件をインデックスアクセスの接頭条件として使用できるかどうかを制御します。
- この変数はデフォルトで有効になっています。有効にすると、オプティマイザはアクセスするインデックスデータのボリュームを減らし、クエリの実行を高速化できます。たとえば、複数列インデックス`index(a, b)`を含むクエリに`a<=>null and b=1`という条件が含まれる場合、オプティマイザはクエリ条件に`a<=>null`と`b=1`の両方をインデックスアクセスに使用できます。変数が無効になっている場合、`a<=>null and b=1`がnull等価条件を含むため、オプティマイザは`b=1`をインデックスアクセスに使用しません。

### tidb\_remove\_orderby\_in\_subquery <span class="version-mark">v6.1.0で新規</span> {#tidb-remove-orderby-in-subquery-span-class-version-mark-new-in-v6-1-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- サブクエリ内の`ORDER BY`句を削除するかどうかを指定します。

### tidb\_replica\_read <span class="version-mark">v4.0で新規</span> {#tidb-replica-read-span-class-version-mark-new-in-v4-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙型
- デフォルト値: `leader`
- 可能な値: `leader`, `follower`, `leader-and-follower`, `prefer-leader`, `closest-replicas`, `closest-adaptive`, `learner`。`learner`の値はv6.6.0で導入されました。
- この変数は、TiDBがデータを読み取る場所を制御するために使用されます。
- 使用方法と実装の詳細については、[Follower read](/follower-read.md)を参照してください。

### tidb\_restricted\_read\_only <span class="version-mark">v5.2.0で新規</span> {#tidb-restricted-read-only-span-class-version-mark-new-in-v5-2-0-span}

> **Note:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- `tidb_restricted_read_only`と[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)は同様の動作をします。ほとんどの場合、[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)のみを使用する必要があります。
- `SUPER`または`SYSTEM_VARIABLES_ADMIN`権限を持つユーザーは、この変数を変更できます。ただし、[Security Enhanced Mode](#tidb_enable_enhanced_security)が有効になっている場合、追加の`RESTRICTED_VARIABLES_ADMIN`権限が必要です。
- `tidb_restricted_read_only`は、次の場合に[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)に影響します。
  - `tidb_restricted_read_only`を`ON`に設定すると、[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)が`ON`に更新されます。
  - `tidb_restricted_read_only`を`OFF`に設定すると、[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)は変更されません。
  - `tidb_restricted_read_only`が`ON`の場合、[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)を`OFF`に設定できません。
- TiDBのDBaaSプロバイダーの場合、TiDBクラスターが他のデータベースの下流データベースである場合、TiDBクラスターを読み取り専用にするには、[Security Enhanced Mode](#tidb_enable_enhanced_security)を有効にして`tidb_super_read_only`を使用してクラスターを書き込み可能にすることを防ぐために、`tidb_restricted_read_only`を使用する必要があります。これには、[Security Enhanced Mode](#tidb_enable_enhanced_security)を有効にし、`SYSTEM_VARIABLES_ADMIN`および`RESTRICTED_VARIABLES_ADMIN`権限を持つ管理ユーザーを使用して`tidb_restricted_read_only`を制御し、データベースユーザーには`SUPER`権限を持つrootユーザーを使用して[`tidb_super_read_only`](#tidb_super_read_only-new-in-v531)を制御させる必要があります。
- この変数は、クラスター全体の読み取り専用ステータスを制御します。変数が`ON`の場合、クラスター全体のすべてのTiDBサーバーが読み取り専用モードになります。この場合、TiDBは`SELECT`、`USE`、`SHOW`などのデータを変更しないステートメントのみを実行します。`INSERT`や`UPDATE`などの他のステートメントについては、TiDBは読み取り専用モードでこれらのステートメントを実行を拒否します。
- この変数を使用して読み取り専用モードを有効にしても、クラスター全体が最終的に読み取り専用ステータスになることを保証します。TiDBクラスターでこの変数の値を変更したが、変更がまだ他のTiDBサーバーに伝播していない場合、未更新のTiDBサーバーはまだ読み取り専用モードでは**ありません**。
- TiDBはSQLステートメントが実行される前に読み取り専用フラグをチェックします。v6.2.0以降、フラグはSQLステートメントがコミットされる前にもチェックされます。これにより、長時間実行される[auto commit](/transaction-overview.md#autocommit)ステートメントが、サーバーが読み取り専用モードに配置された後にデータを変更するケースを防ぐのに役立ちます。
- この変数が有効になっている場合、TiDBは未コミットのトランザクションを次のように処理します。
  - 未コミットの読み取り専用トランザクションの場合、通常通りトランザクションをコミットできます。
  - 読み取り専用でない未コミットトランザクションの場合、これらのトランザクションで書き込み操作を行うSQLステートメントは拒否されます。
  - 変更されたデータを持つ未コミットの読み取り専用トランザクションの場合、これらのトランザクションのコミットは拒否されます。
- 読み取り専用モードが有効になった後、すべてのユーザー（`SUPER`権限を持つユーザーを含む）は、ユーザーが明示的に`RESTRICTED_REPLICA_WRITER_ADMIN`権限を付与されていない限り、データを書き込む可能性のあるSQLステートメントを実行できません。

### tidb\_retry\_limit {#tidb-retry-limit}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `10`
- 範囲: `[-1, 9223372036854775807]`
- この変数は、楽観的トランザクションの最大リトライ回数を設定するために使用されます。トランザクションがリトライ可能なエラー（トランザクションの競合、非常に遅いトランザクションのコミット、またはテーブルスキーマの変更など）に遭遇した場合、この変数に従ってトランザクションが再実行されます。なお、`tidb_retry_limit`を`0`に設定すると、自動リトライが無効になります。この変数は楽観的トランザクションにのみ適用され、悲観的トランザクションには適用されません。

### tidb\_row\_format\_version {#tidb-row-format-version}

> **Note:**
>
> このTiDB変数は、TiDB Cloudには適用されません。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `2`
- 範囲: `[1, 2]`
- テーブルに新しく保存されるデータのフォーマットバージョンを制御します。TiDB v4.0では、[新しいストレージ行フォーマット](https://github.com/pingcap/tidb/blob/master/docs/design/2018-07-19-row-format.md)のバージョン`2`がデフォルトで使用されます。
- v4.0.0以前のTiDBバージョンからv4.0.0以降のバージョンにアップグレードする場合、フォーマットバージョンは変更されず、TiDBは引き続き古いバージョン`1`のフォーマットでデータを書き込みます。つまり、**新しく作成されたクラスターのみがデフォルトで新しいデータフォーマットを使用します**。
- この変数を変更しても、保存された古いデータには影響しませんが、この変数を変更した後に書き込まれる新しいデータにのみ対応するバージョンフォーマットが適用されます。

### tidb\_scatter\_region {#tidb-scatter-region}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- デフォルトでは、TiDBで新しいテーブルが作成されると、リージョンが分割されます。この変数を有効にした後、`CREATE TABLE`ステートメントの実行中に新しく分割されたリージョンはすぐに散在します。これは、テーブルがバッチで作成された直後にデータをバッチで書き込む必要があるシナリオに適用されます。なぜなら、新しく分割されたリージョンはPDによってスケジュールされるのを待つ必要がなく、事前にTiKVで散在できるからです。バッチでテーブルが作成された後、リージョンが正常に散在した後に`CREATE TABLE`ステートメントが成功を返します。これにより、ステートメントの実行時間は、この変数を無効にした場合よりも複数倍長くなります。
- `SHARD_ROW_ID_BITS`と`PRE_SPLIT_REGIONS`がテーブル作成時に設定されている場合、指定された数のリージョンがテーブル作成後に均等に分割されます。"

### tidb\_server\_memory\_limit <span class="version-mark">v6.4.0で新規</span> {#tidb-server-memory-limit-span-class-version-mark-new-in-v6-4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `80%`
- 範囲:
  - パーセンテージ形式で値を設定できます。これはメモリ使用量の総メモリに対するパーセンテージを意味します。値の範囲は`[1%, 99%]`です。
  - メモリサイズでも値を設定できます。値の範囲はバイト単位で`0`および`[536870912, 9223372036854775807]`です。単位"KB|MB|GB|TB"を使用したメモリフォーマットがサポートされています。`0`はメモリ制限がないことを意味します。
  - この変数が512 MB未満のメモリサイズに設定されている場合、ただし`0`でない場合、TiDBは実際のサイズとして512 MBを使用します。
- この変数は、TiDBインスタンスのメモリ制限を指定します。TiDBのメモリ使用量が制限に達すると、TiDBは現在実行中のメモリ使用量が最も高いSQLステートメントをキャンセルします。SQLステートメントが正常にキャンセルされた後、TiDBはできるだけ早くメモリストレスを緩和するためにGolang GCを呼び出そうとします。
- [`tidb_server_memory_limit_sess_min_size`](/system-variables.md#tidb_server_memory_limit_sess_min_size-new-in-v640)の制限を超えるメモリ使用量のSQLステートメントのみが、最初にキャンセルされるSQLステートメントとして選択されます。
- 現在、TiDBは1回につき1つのSQLステートメントのみをキャンセルします。TiDBがSQLステートメントを完全にキャンセルし、リソースを回復した後、メモリ使用量がこの変数で設定された制限よりも依然として大きい場合、TiDBは次のキャンセル操作を開始します。

### tidb\_server\_memory\_limit\_gc\_trigger <span class="version-mark">v6.4.0で新規</span> {#tidb-server-memory-limit-gc-trigger-span-class-version-mark-new-in-v6-4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `70%`
- 範囲: `[50%, 99%]`
- TiDBがGCをトリガしようとするしきい値。TiDBのメモリ使用量が`tidb_server_memory_limit`の値と`tidb_server_memory_limit_gc_trigger`の値の積に達すると、TiDBは積極的にGolang GC操作をトリガしようとします。1分間に1回だけGC操作がトリガされます。

### tidb\_server\_memory\_limit\_sess\_min\_size <span class="version-mark">v6.4.0で新規</span> {#tidb-server-memory-limit-sess-min-size-span-class-version-mark-new-in-v6-4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `134217728` (128 MB)
- 範囲: `[128, 9223372036854775807]`、バイト単位。単位"KB|MB|GB|TB"を使用したメモリフォーマットもサポートされています。
- メモリ制限を有効にした後、TiDBは現在のインスタンスで最も高いメモリ使用量のSQLステートメントを終了します。この変数は、終了するSQLステートメントの最小メモリ使用量を指定します。制限を超えるTiDBインスタンスのメモリ使用量が低いメモリ使用量のセッションが多すぎる場合、この変数の値を適切に下げて、より多くのセッションをキャンセルできるようにすることができます。

### `tidb_session_plan_cache_size` <span class="version-mark">v7.1.0で新規</span> {#tidb-session-plan-cache-size-span-class-version-mark-new-in-v7-1-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `100`
- 範囲: `[1, 100000]`
- この変数は、キャッシュできるプランの最大数を制御します。[プリペアドプランキャッシュ](/sql-prepared-plan-cache.md)と[プリペアドプランキャッシュ](/sql-non-prepared-plan-cache.md)は同じキャッシュを共有します。
- 以前のバージョンからv7.1.0またはそれ以降のバージョンにアップグレードする場合、この変数は[`tidb_prepared_plan_cache_size`](#tidb_prepared_plan_cache_size-new-in-v610)と同じ値のままです。

### tidb\_shard\_allocate\_step <span class="version-mark">v5.0で新規</span> {#tidb-shard-allocate-step-span-class-version-mark-new-in-v5-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `9223372036854775807`
- 範囲: `[1, 9223372036854775807]`
- この変数は、[`AUTO_RANDOM`](/auto-random.md)または[`SHARD_ROW_ID_BITS`](/shard-row-id-bits.md)属性に割り当てられる連続したIDの最大数を制御します。一般的に、`AUTO_RANDOM` IDまたは`SHARD_ROW_ID_BITS`で注釈付けされた行IDは、1つのトランザクション内で増分および連続です。この変数を使用して、大規模なトランザクションシナリオでのホットスポット問題を解決できます。

### tidb\_simplified\_metrics {#tidb-simplified-metrics}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数が有効になっていると、TiDBはGrafanaパネルで使用されていないメトリクスを収集または記録しません。

### tidb\_skip\_ascii\_check <span class="version-mark">v5.0で新規</span> {#tidb-skip-ascii-check-span-class-version-mark-new-in-v5-0-span}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数はASCII検証をスキップするかどうかを設定するために使用されます。
- ASCII文字の検証はパフォーマンスに影響を与えます。入力文字が有効なASCII文字であることが確実な場合、変数値を`ON`に設定できます。

### tidb\_skip\_isolation\_level\_check {#tidb-skip-isolation-level-check}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- このスイッチが有効になると、`tx_isolation`にTiDBでサポートされていない分離レベルが割り当てられた場合、エラーが報告されません。これにより、異なる分離レベルを設定（ただし依存しない）アプリケーションとの互換性が向上します。"

```sql
tidb> set tx_isolation='serializable';
ERROR 8048 (HY000): The isolation level 'serializable' is not supported. Set tidb_skip_isolation_level_check=1 to skip this error
tidb> set tidb_skip_isolation_level_check=1;
Query OK, 0 rows affected (0.00 sec)

tidb> set tx_isolation='serializable';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

### tidb\_skip\_utf8\_check {#tidb-skip-utf8-check}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- This variable is used to set whether to skip UTF-8 validation.
- Validating UTF-8 characters affects the performance. When you are sure that the input characters are valid UTF-8 characters, you can set the variable value to `ON`.

> **Note:**
>
> If the character check is skipped, TiDB might fail to detect illegal UTF-8 characters written by the application, cause decoding errors when `ANALYZE` is executed, and introduce other unknown encoding issues. If your application cannot guarantee the validity of the written string, it is not recommended to skip the character check.

### tidb\_slow\_log\_threshold {#tidb-slow-log-threshold}

> **Note:**
>
> This TiDB variable is not applicable to TiDB Cloud.

- Scope: GLOBAL
- Persists to cluster: No, only applicable to the current TiDB instance that you are connecting to.
- Type: Integer
- Default value: `300`
- Range: `[-1, 9223372036854775807]`
- Unit: Milliseconds
- This variable is used to output the threshold value of the time consumed by the slow log. When the time consumed by a query is larger than this value, this query is considered as a slow log and its log is output to the slow query log.

### tidb\_slow\_query\_file {#tidb-slow-query-file}

> **Note:**
>
> This TiDB variable is not applicable to TiDB Cloud.

- Scope: SESSION
- Default value: ""
- When `INFORMATION_SCHEMA.SLOW_QUERY` is queried, only the slow query log name set by `slow-query-file` in the configuration file is parsed. The default slow query log name is "tidb-slow\.log". To parse other logs, set the `tidb_slow_query_file` session variable to a specific file path, and then query `INFORMATION_SCHEMA.SLOW_QUERY` to parse the slow query log based on the set file path.

<CustomContent platform="tidb">

For details, see [Identify Slow Queries](/identify-slow-queries.md).

</CustomContent>

### tidb\_snapshot {#tidb-snapshot}

- Scope: SESSION
- Default value: ""
- This variable is used to set the time point at which the data is read by the session. For example, when you set the variable to "2017-11-11 20:20:20" or a TSO number like "400036290571534337", the current session reads the data of this moment.

### tidb\_source\_id <span class="version-mark">New in v6.5.0</span> {#tidb-source-id-span-class-version-mark-new-in-v6-5-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `1`
- Range: `[1, 15]`

<CustomContent platform="tidb">

- This variable is used to configure the different cluster IDs in a [bi-directional replication](/ticdc/ticdc-bidirectional-replication.md) cluster.

</CustomContent>

<CustomContent platform="tidb-cloud">

- This variable is used to configure the different cluster IDs in a [bi-directional replication](https://docs.pingcap.com/tidb/stable/ticdc-bidirectional-replication) cluster.

</CustomContent>

### tidb\_stats\_cache\_mem\_quota <span class="version-mark">New in v6.1.0</span> {#tidb-stats-cache-mem-quota-span-class-version-mark-new-in-v6-1-0-span}

> **Warning:**
>
> This variable is an experimental feature. It is not recommended to use it in production environments.

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `0`
- Range: `[0, 1099511627776]`
- This variable sets the memory quota for the TiDB statistics cache.

### tidb\_stats\_load\_pseudo\_timeout <span class="version-mark">New in v5.4.0</span> {#tidb-stats-load-pseudo-timeout-span-class-version-mark-new-in-v5-4-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- This variable controls how TiDB behaves when the waiting time of SQL optimization reaches the timeout to synchronously load complete column statistics. The default value `ON` means that the SQL optimization gets back to using pseudo statistics after the timeout. If this variable to `OFF`, SQL execution fails after the timeout.

### tidb\_stats\_load\_sync\_wait <span class="version-mark">New in v5.4.0</span> {#tidb-stats-load-sync-wait-span-class-version-mark-new-in-v5-4-0-span}

> **Note:**
>
> This variable is read-only for [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless).

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `100`
- Range: `[0, 2147483647]`
- Unit: Milliseconds
- This variable controls whether to enable the synchronously loading statistics feature. The value `0` means that the feature is disabled. To enable the feature, you can set this variable to a timeout (in milliseconds) that SQL optimization can wait for at most to synchronously load complete column statistics. For details, see [Load statistics](/statistics.md#load-statistics).

### tidb\_stmt\_summary\_enable\_persistent <span class="version-mark">New in v6.6.0</span> {#tidb-stmt-summary-enable-persistent-span-class-version-mark-new-in-v6-6-0-span}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> This TiDB variable is not applicable to TiDB Cloud.

</CustomContent>

> **Warning:**
>
> Statements summary persistence is an experimental feature. It is not recommended that you use it in the production environment. This feature might be changed or removed without prior notice. If you find a bug, you can report an [issue](https://github.com/pingcap/tidb/issues) on GitHub.

- Scope: GLOBAL
- Type: Boolean
- Default value: `OFF`
- This variable is read-only. It controls whether to enable [statements summary persistence](/statement-summary-tables.md#persist-statements-summary).

<CustomContent platform="tidb">

- The value of this variable is the same as that of the configuration item [`tidb_stmt_summary_enable_persistent`](/tidb-configuration-file.md#tidb_stmt_summary_enable_persistent-new-in-v660).

</CustomContent>

### tidb\_stmt\_summary\_filename <span class="version-mark">New in v6.6.0</span> {#tidb-stmt-summary-filename-span-class-version-mark-new-in-v6-6-0-span}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> This TiDB variable is not applicable to TiDB Cloud.

</CustomContent>

> **Warning:**
>
> Statements summary persistence is an experimental feature. It is not recommended that you use it in the production environment. This feature might be changed or removed without prior notice. If you find a bug, you can report an [issue](https://github.com/pingcap/tidb/issues) on GitHub.

- Scope: GLOBAL
- Type: String
- Default value: `"tidb-statements.log"`
- This variable is read-only. It specifies the file to which persistent data is written when [statements summary persistence](/statement-summary-tables.md#persist-statements-summary) is enabled.

<CustomContent platform="tidb">

- The value of this variable is the same as that of the configuration item [`tidb_stmt_summary_filename`](/tidb-configuration-file.md#tidb_stmt_summary_filename-new-in-v660).

</CustomContent>

### tidb\_stmt\_summary\_file\_max\_backups <span class="version-mark">New in v6.6.0</span> {#tidb-stmt-summary-file-max-backups-span-class-version-mark-new-in-v6-6-0-span}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> This TiDB variable is not applicable to TiDB Cloud.

</CustomContent>

> **Warning:**
>
> Statements summary persistence is an experimental feature. It is not recommended that you use it in the production environment. This feature might be changed or removed without prior notice. If you find a bug, you can report an [issue](https://github.com/pingcap/tidb/issues) on GitHub.

- Scope: GLOBAL
- Type: Integer
- Default value: `0`
- This variable is read-only. It specifies the maximum number of data files that can be persisted when [statements summary persistence](/statement-summary-tables.md#persist-statements-summary) is enabled.

<CustomContent platform="tidb">

- The value of this variable is the same as that of the configuration item [`tidb_stmt_summary_file_max_backups`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_backups-new-in-v660).

</CustomContent>

### tidb\_stmt\_summary\_file\_max\_days <span class="version-mark">v6.6.0で新規</span> {#tidb-stmt-summary-file-max-days-span-class-version-mark-new-in-v6-6-0-span}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

> **Warning:**
>
> ステートメントのサマリー永続性は実験的な機能です。本番環境で使用しないことをお勧めします。この機能は事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- スコープ: GLOBAL
- タイプ: 整数
- デフォルト値: `3`
- 単位: 日
- この変数は読み取り専用です。[ステートメントのサマリー永続性](/statement-summary-tables.md#persist-statements-summary)が有効になっている場合、永続データファイルを保持する最大日数を指定します。

<CustomContent platform="tidb">

- この変数の値は、構成項目[`tidb_stmt_summary_file_max_days`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_days-new-in-v660)と同じです。

</CustomContent>

### tidb\_stmt\_summary\_file\_max\_size <span class="version-mark">v6.6.0で新規</span> {#tidb-stmt-summary-file-max-size-span-class-version-mark-new-in-v6-6-0-span}

<CustomContent platform="tidb-cloud">

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

</CustomContent>

> **Warning:**
>
> ステートメントのサマリー永続性は実験的な機能です。本番環境で使用しないことをお勧めします。この機能は事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告できます。

- スコープ: GLOBAL
- タイプ: 整数
- デフォルト値: `64`
- 単位: MiB
- この変数は読み取り専用です。[ステートメントのサマリー永続性](/statement-summary-tables.md#persist-statements-summary)が有効になっている場合、永続データファイルの最大サイズを指定します。

<CustomContent platform="tidb">

- この変数の値は、構成項目[`tidb_stmt_summary_file_max_size`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_size-new-in-v660)と同じです。

</CustomContent>

### tidb\_stmt\_summary\_history\_size <span class="version-mark">v4.0で新規</span> {#tidb-stmt-summary-history-size-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `24`
- 範囲: `[0, 255]`
- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)の履歴容量を設定するために使用されます。

### tidb\_stmt\_summary\_internal\_query <span class="version-mark">v4.0で新規</span> {#tidb-stmt-summary-internal-query-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `OFF`
- この変数は[TiDBのSQL情報をステートメントのサマリーテーブル](/statement-summary-tables.md)に含めるかどうかを制御するために使用されます。

### tidb\_stmt\_summary\_max\_sql\_length <span class="version-mark">v4.0で新規</span> {#tidb-stmt-summary-max-sql-length-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `4096`
- 範囲: `[0, 2147483647]`

<CustomContent platform="tidb">

- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)、[`SLOW_QUERY`](/information-schema/information-schema-slow-query.md)テーブル、および[TiDBダッシュボード](/dashboard/dashboard-intro.md)のSQL文字列の長さを制御するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)および[`SLOW_QUERY`](/information-schema/information-schema-slow-query.md)テーブルのSQL文字列の長さを制御するために使用されます。

</CustomContent>

### tidb\_stmt\_summary\_max\_stmt\_count <span class="version-mark">v4.0で新規</span> {#tidb-stmt-summary-max-stmt-count-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `3000`
- 範囲: `[1, 32767]`
- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)にメモリ内に保存されるステートメントの最大数を設定するために使用されます。

### tidb\_stmt\_summary\_refresh\_interval <span class="version-mark">v4.0で新規</span> {#tidb-stmt-summary-refresh-interval-span-class-version-mark-new-in-v4-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1800`
- 範囲: `[1, 2147483647]`
- 単位: 秒
- この変数は[ステートメントのサマリーテーブル](/statement-summary-tables.md)のリフレッシュ時間を設定するために使用されます。

### tidb\_store\_batch\_size {#tidb-store-batch-size}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `4`
- 範囲: `[0, 25000]`
- この変数は`IndexLookUp`オペレーターのCoprocessorタスクのバッチサイズを制御するために使用されます。`0`はバッチを無効にします。タスクの数が比較的多く、遅いクエリが発生する場合、この変数を増やしてクエリを最適化することができます。

### tidb\_store\_limit <span class="version-mark">v3.0.4およびv4.0で新規</span> {#tidb-store-limit-span-class-version-mark-new-in-v3-0-4-and-v4-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[0, 9223372036854775807]`
- この変数はTiDBが同時にTiKVに送信できるリクエストの最大数を制限するために使用されます。0は制限がないことを意味します。

### tidb\_streamagg\_concurrency {#tidb-streamagg-concurrency}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- この変数はクエリの実行時に`StreamAgg`オペレーターの並列処理を設定します。
- この変数の設定は**推奨されません**。変数の値を変更するとデータの正確性に問題が発生する可能性があります。

### tidb\_super\_read\_only <span class="version-mark">v5.3.1で新規</span> {#tidb-super-read-only-span-class-version-mark-new-in-v5-3-1-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- `tidb_super_read_only`はMySQL変数`super_read_only`の代替として実装されることを目指しています。ただし、TiDBは分散データベースであるため、`tidb_super_read_only`は実行後すぐにデータベースを読み取り専用にはしませんが、最終的には読み取り専用にします。
- `SUPER`または`SYSTEM_VARIABLES_ADMIN`権限を持つユーザーはこの変数を変更できます。
- この変数はクラスタ全体の読み取り専用状態を制御します。変数が`ON`の場合、クラスタ全体のすべてのTiDBサーバーが読み取り専用モードになります。この場合、TiDBは`SELECT`、`USE`、`SHOW`などのデータを変更しないステートメントのみを実行します。`INSERT`や`UPDATE`などの他のステートメントについては、TiDBは読み取り専用モードでこれらのステートメントを実行を拒否します。
- この変数を使用して読み取り専用モードを有効にしても、クラスタ全体が最終的に読み取り専用状態になることしか保証しません。TiDBクラスタでこの変数の値を変更したが、変更が他のTiDBサーバーにまだ伝播していない場合、未更新のTiDBサーバーはまだ読み取り専用モードではありません。
- TiDBはSQLステートメントが実行される前に読み取り専用フラグをチェックします。v6.2.0以降、ステートメントがコミットされる前にもフラグがチェックされます。これにより、長時間実行される[自動コミット](/transaction-overview.md#autocommit)ステートメントが、サーバーが読み取り専用モードに配置された後にデータを変更する可能性が防止されます。
- この変数が有効になると、TiDBは未コミットのトランザクションを次のように処理します。
  - 未コミットの読み取り専用トランザクションの場合、通常にトランザクションをコミットできます。
  - 読み取り専用でない未コミットトランザクションの場合、これらのトランザクションで書き込み操作を実行するSQLステートメントは拒否されます。
  - 変更されたデータを持つ未コミットの読み取り専用トランザクションの場合、これらのトランザクションのコミットは拒否されます。
- 読み取り専用モードが有効になった後、すべてのユーザー（`SUPER`権限を持つユーザーも含む）は、ユーザーが明示的に`RESTRICTED_REPLICA_WRITER_ADMIN`権限を付与されていない限り、データを書き込む可能性のあるSQLステートメントを実行できません。
- [`tidb_restricted_read_only`](#tidb_restricted_read_only-new-in-v520)システム変数が`ON`に設定されている場合、`tidb_super_read_only`は一部の場合で[`tidb_restricted_read_only`](#tidb_restricted_read_only-new-in-v520)に影響を受けます。詳細な影響については、[`tidb_restricted_read_only`](#tidb_restricted_read_only-new-in-v520)の説明を参照してください。

### tidb\_sysdate\_is\_now <span class="version-mark">v6.0.0で新規</span> {#tidb-sysdate-is-now-span-class-version-mark-new-in-v6-0-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `OFF`
- この変数は`SYSDATE`関数を`NOW`関数で置き換えるかどうかを制御するために使用されます。この構成項目は、MySQLオプション[`sysdate-is-now`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_sysdate-is-now)と同じ効果があります。

### tidb\_sysproc\_scan\_concurrency <span class="version-mark">v6.5.0で新規</span> {#tidb-sysproc-scan-concurrency-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `1`
- Range: `[1, 256]`
- この変数は、TiDBが内部SQLステートメント（統計の自動更新など）を実行する際に実行されるスキャン操作の並行性を設定するために使用されます。

### tidb\_table\_cache\_lease <span class="version-mark">v6.0.0で新規</span> {#tidb-table-cache-lease-span-class-version-mark-new-in-v6-0-0-span}

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `3`
- Range: `[1, 10]`
- Unit: Seconds
- この変数は、デフォルト値が`3`の[cached tables](/cached-tables.md)のリース時間を制御するために使用されます。この変数の値は、キャッシュされたテーブルへの変更に影響します。キャッシュされたテーブルに変更が加えられた場合、最長待機時間は`tidb_table_cache_lease`秒になる可能性があります。テーブルが読み取り専用であるか、高い書き込み遅延を受け入れることができる場合、この変数の値を増やして、キャッシュされたテーブルの有効時間を増やし、リースの頻度を減らすことができます。

### tidb\_tmp\_table\_max\_size <span class="version-mark">v5.3.0で新規</span> {#tidb-tmp-table-max-size-span-class-version-mark-new-in-v5-3-0-span}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `67108864`
- Range: `[1048576, 137438953472]`
- Unit: Bytes
- この変数は、単一の[temporary table](/temporary-tables.md)の最大サイズを設定するために使用されます。この変数値よりも大きいサイズの一時テーブルはエラーを引き起こします。

### tidb\_top\_sql\_max\_meta\_count <span class="version-mark">v6.0.0で新規</span> {#tidb-top-sql-max-meta-count-span-class-version-mark-new-in-v6-0-0-span}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `5000`
- Range: `[1, 10000]`

<CustomContent platform="tidb">

- この変数は、[Top SQL](/dashboard/top-sql.md)によって1分あたりに収集されるSQLステートメントの最大数を制御するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、[Top SQL](https://docs.pingcap.com/tidb/stable/top-sql)によって1分あたりに収集されるSQLステートメントの最大数を制御するために使用されます。

</CustomContent>

### tidb\_top\_sql\_max\_time\_series\_count <span class="version-mark">v6.0.0で新規</span> {#tidb-top-sql-max-time-series-count-span-class-version-mark-new-in-v6-0-0-span}

> **Note:**
>
> このTiDB変数はTiDB Cloudには適用されません。

> **Note:**
>
> 現在、TiDB DashboardのTop SQLページは、負荷に最も貢献する上位5種類のSQLクエリのみを表示しますが、これは`tidb_top_sql_max_time_series_count`の構成とは関係ありません。

- Scope: GLOBAL
- Persists to cluster: Yes
- Type: Integer
- Default value: `100`
- Range: `[1, 5000]`

<CustomContent platform="tidb">

- この変数は、負荷に最も貢献するSQLステートメント（つまり、上位N）を[Top SQL](/dashboard/top-sql.md)によって1分あたりに記録できる数を制御するために使用されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、負荷に最も貢献するSQLステートメント（つまり、上位N）を[Top SQL](https://docs.pingcap.com/tidb/stable/top-sql)によって1分あたりに記録できる数を制御するために使用されます。

</CustomContent>

### tidb\_track\_aggregate\_memory\_usage {#tidb-track-aggregate-memory-usage}

- Scope: SESSION | GLOBAL
- Persists to cluster: Yes
- Type: Boolean
- Default value: `ON`
- この変数は、TiDBが集約関数のメモリ使用量を追跡するかどうかを制御します。

> **Warning:**
>
> この変数を無効にすると、TiDBはメモリ使用量を正確に追跡できず、対応するSQLステートメントのメモリ使用量を制御できなくなる可能性があります。

### tidb\_tso\_client\_batch\_max\_wait\_time <span class="version-mark">v5.3.0で新規</span> {#tidb-tso-client-batch-max-wait-time-span-class-version-mark-new-in-v5-3-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 浮動小数点数
- デフォルト値: `0`
- 範囲: `[0, 10]`
- 単位: ミリ秒
- この変数は、TiDBがPDからTSOを要求する際のバッチ操作の最大待ち時間を設定するために使用されます。デフォルト値は`0`で、追加の待ち時間はありません。
- PDクライアントは、TiDBによって使用されるPDから受信したできるだけ多くのTSO要求を収集します。その後、PDクライアントは収集された要求をバッチで1つのRPC要求にマージし、要求をPDに送信します。これにより、PDへの負荷が軽減されます。
- この変数を`0`より大きな値に設定した後、TiDBは各バッチマージの終了前にこの値の最大期間待機します。これにより、より多くのTSO要求が収集され、バッチ操作の効果が向上します。
- この変数の値を増やすシナリオ:
  - TSO要求の高い圧力により、PDリーダーのCPUがボトルネックに達し、TSO RPC要求のレイテンシが高くなる。
  - クラスターにはTiDBインスタンスが多くありませんが、各TiDBインスタンスが高い並行性である。
- この変数はできるだけ小さな値に設定することをお勧めします。

> **Note:**
>
> PDリーダーのCPU使用率（ネットワークの問題などの他の理由により）以外の理由でTSO RPCレイテンシが増加した場合、`tidb_tso_client_batch_max_wait_time`の値を増やすと、TiDBの実行レイテンシが増加し、クラスターのQPSパフォーマンスに影響を与える可能性があります。

### tidb\_ttl\_delete\_rate\_limit <span class="version-mark">v6.5.0で新規</span> {#tidb-ttl-delete-rate-limit-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `0`
- 範囲: `[0, 9223372036854775807]`
- この変数は、各TiDBノードでTTLジョブの`DELETE`ステートメントのレートを制限するために使用されます。この値は、TTLジョブの単一ノードで1秒あたり許可される`DELETE`ステートメントの最大数を表します。この変数が`0`に設定されている場合、制限は適用されません。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_ttl\_delete\_batch\_size <span class="version-mark">v6.5.0で新規</span> {#tidb-ttl-delete-batch-size-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `100`
- 範囲: `[1, 10240]`
- この変数は、TTLジョブの単一の`DELETE`トランザクションで削除できる行の最大数を設定するために使用されます。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_ttl\_delete\_worker\_count <span class="version-mark">v6.5.0で新規</span> {#tidb-ttl-delete-worker-count-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `4`
- 範囲: `[1, 256]`
- この変数は、各TiDBノードでのTTLジョブの最大並行性を設定するために使用されます。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_ttl\_job\_enable <span class="version-mark">v6.5.0で新規</span> {#tidb-ttl-job-enable-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ON`
- タイプ: ブール値
- この変数は、TTLジョブが有効かどうかを制御するために使用されます。`OFF`に設定されている場合、TTL属性を持つすべてのテーブルは自動的に期限切れのデータのクリーンアップを停止します。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_ttl\_scan\_batch\_size <span class="version-mark">v6.5.0で新規</span> {#tidb-ttl-scan-batch-size-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `500`
- 範囲: `[1, 10240]`
- この変数は、TTLジョブで期限切れのデータをスキャンするために使用される各`SELECT`ステートメントの`LIMIT`値を設定するために使用されます。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_ttl\_scan\_worker\_count <span class="version-mark">v6.5.0で新規</span> {#tidb-ttl-scan-worker-count-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `4`
- 範囲: `[1, 256]`
- この変数は、各TiDBノードでのTTLスキャンジョブの最大並行性を設定するために使用されます。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_ttl\_job\_schedule\_window\_start\_time <span class="version-mark">v6.5.0で新規</span> {#tidb-ttl-job-schedule-window-start-time-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- タイプ: 時間
- クラスターに永続化: はい
- デフォルト値: `00:00 +0000`
- この変数は、バックグラウンドでのTTLジョブのスケジューリングウィンドウの開始時間を制御するために使用されます。この変数の値を変更する際には、小さなウィンドウは期限切れのデータのクリーンアップに失敗する可能性があるため注意してください。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_ttl\_job\_schedule\_window\_end\_time <span class="version-mark">v6.5.0で新規</span> {#tidb-ttl-job-schedule-window-end-time-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- タイプ: 時間
- クラスターに永続化: はい
- デフォルト値: `23:59 +0000`
- この変数は、バックグラウンドでのTTLジョブのスケジューリングウィンドウの終了時間を制御するために使用されます。この変数の値を変更する際には、小さなウィンドウは期限切れのデータのクリーンアップに失敗する可能性があるため注意してください。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_ttl\_running\_tasks <span class="version-mark">v7.0.0で新規</span> {#tidb-ttl-running-tasks-span-class-version-mark-new-in-v7-0-0-span}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `-1`および`[1, 256]`
- クラスタ全体で実行されるTTLタスクの最大数を指定します。`-1`はTTLタスクの数がTiKVノードの数と同等であることを意味します。詳細については、[Time to Live](/time-to-live.md)を参照してください。

### tidb\_txn\_assertion\_level <span class="version-mark">v6.0.0で新規</span> {#tidb-txn-assertion-level-span-class-version-mark-new-in-v6-0-0-span}

- スコープ: SESSION | GLOBAL

- クラスターに永続化: はい

- タイプ: 列挙型

- デフォルト値: `FAST`

- 可能な値: `OFF`, `FAST`, `STRICT`

- この変数は、アサーションレベルを制御するために使用されます。アサーションは、データとインデックスの整合性チェックであり、トランザクションのコミットプロセスで書き込まれるキーが存在するかどうかをチェックします。詳細については、[Troubleshoot Inconsistency Between Data and Indexes](/troubleshoot-data-inconsistency-errors.md)を参照してください。

  - `OFF`: このチェックを無効にします。
  - `FAST`: ほとんどパフォーマンスに影響を与えずにほとんどのチェック項目を有効にします。
  - `STRICT`: システムのワークロードが高い場合、悲観的トランザクションのパフォーマンスにわずかな影響を与えながら、すべてのチェック項目を有効にします。

- v6.0.0以降の新しいクラスターの場合、デフォルト値は`FAST`です。v6.0.0より前のバージョンからアップグレードされた既存のクラスターの場合、デフォルト値は`OFF`です。

### tidb\_txn\_commit\_batch\_size <span class="version-mark">v6.2.0で新規</span> {#tidb-txn-commit-batch-size-span-class-version-mark-new-in-v6-2-0-span}

- Scope: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `16384`
- 範囲: `[1, 1073741824]`
- 単位: バイト

<CustomContent platform="tidb">

- この変数は、TiDBがTiKVに送信するトランザクションコミットリクエストのバッチサイズを制御するために使用されます。アプリケーションのワークロードのほとんどのトランザクションが多数の書き込み操作を持つ場合、この変数を大きな値に調整すると、バッチ処理のパフォーマンスが向上する可能性があります。ただし、この変数が大きすぎてTiKVの[`raft-entry-max-size`](/tikv-configuration-file.md#raft-entry-max-size)の制限を超えると、コミットが失敗する可能性があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- この変数は、TiDBがTiKVに送信するトランザクションコミットリクエストのバッチサイズを制御するために使用されます。アプリケーションのワークロードのほとんどのトランザクションが多数の書き込み操作を持つ場合、この変数を大きな値に調整すると、バッチ処理のパフォーマンスが向上する可能性があります。ただし、この変数が大きすぎてTiKVの単一ログの最大サイズ（デフォルトでは8 MB）を超えると、コミットが失敗する可能性があります。

</CustomContent>

### tidb\_txn\_mode {#tidb-txn-mode}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙型
- デフォルト値: `pessimistic`
- 可能な値: `pessimistic`, `optimistic`
- この変数はトランザクションモードを設定するために使用されます。TiDB 3.0では悲観的トランザクションがサポートされています。TiDB 3.0.8以降、[悲観的トランザクションモード](/pessimistic-transaction.md)がデフォルトで有効になっています。
- TiDBをv3.0.7またはそれ以前のバージョンからv3.0.8またはそれ以降のバージョンにアップグレードする場合、デフォルトのトランザクションモードは変更されません。**デフォルトで悲観的トランザクションモードを使用するのは、新しく作成されたクラスターのみです**。
- この変数が"optimistic"または""に設定されている場合、TiDBは[楽観的トランザクションモード](/optimistic-transaction.md)を使用します。

### tidb\_use\_plan\_baselines <span class="version-mark">v4.0で新規</span> {#tidb-use-plan-baselines-span-class-version-mark-new-in-v4-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、実行計画バインディング機能を有効にするかどうかを制御するために使用されます。デフォルトでは有効になっており、`OFF`値を割り当てることで無効にすることができます。実行計画バインディングの使用については、[実行計画バインディング](/sql-plan-management.md#create-a-binding)を参照してください。

### tidb\_wait\_split\_region\_finish {#tidb-wait-split-region-finish}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: SESSION
- タイプ: ブール値
- デフォルト値: `ON`
- 通常、リージョンの分散にはPDのスケジューリングとTiKVの負荷によって長い時間がかかります。この変数は、`SPLIT REGION`ステートメントが実行されているときに、すべてのリージョンが完全に分散された後にクライアントに結果を返すかどうかを設定するために使用されます。
  - `ON`は、`SPLIT REGIONS`ステートメントがすべてのリージョンが分散されるまで待機することを要求します。
  - `OFF`は、`SPLIT REGIONS`ステートメントがすべてのリージョンの分散が完了する前に返ることを許可します。
- リージョンの分散中に、分散されているリージョンの書き込みおよび読み取りのパフォーマンスに影響を与える可能性があります。バッチ書き込みやデータのインポートシナリオでは、リージョンの分散が完了した後にデータをインポートすることを推奨します。

### tidb\_wait\_split\_region\_timeout {#tidb-wait-split-region-timeout}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- Scope: SESSION
- タイプ: 整数
- デフォルト値: `300`
- 範囲: `[1, 2147483647]`
- 単位: 秒
- この変数は、`SPLIT REGION`ステートメントの実行タイムアウトを設定するために使用されます。指定された時間内にステートメントが完全に実行されない場合、タイムアウトエラーが返されます。

### tidb\_window\_concurrency <span class="version-mark">v4.0で新規</span> {#tidb-window-concurrency-span-class-version-mark-new-in-v4-0-span}

> **Warning:**
>
> v5.0以降、この変数は非推奨です。代わりに、[`tidb_executor_concurrency`](#tidb_executor_concurrency-new-in-v50)を使用してください。

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `-1`
- 範囲: `[1, 256]`
- 単位: スレッド
- この変数は、ウィンドウ演算子の並列度を設定するために使用されます。
- `-1`の値は、`tidb_executor_concurrency`の値が代わりに使用されることを意味します。

### tiflash\_fastscan <span class="version-mark">v6.3.0で新規</span> {#tiflash-fastscan-span-class-version-mark-new-in-v6-3-0-span}

- Scope: SESSION | GLOBAL
- デフォルト値: `OFF`
- タイプ: ブール値
- [FastScan](/tiflash/use-fastscan.md)が有効になっている場合（`ON`に設定されている場合）、TiFlashはより効率的なクエリパフォーマンスを提供しますが、クエリ結果やデータの整合性を保証しません。

### tiflash\_fine\_grained\_shuffle\_batch\_size <span class="version-mark">v6.2.0で新規</span> {#tiflash-fine-grained-shuffle-batch-size-span-class-version-mark-new-in-v6-2-0-span}

- Scope: SESSION | GLOBAL
- デフォルト値: `8192`
- 範囲: `[1, 18446744073709551615]`
- Fine Grained Shuffleが有効になっている場合、TiFlashにプッシュダウンされたウィンドウ関数を並列で実行できます。この変数は、送信者が送信するデータのバッチサイズを制御します。
- パフォーマンスへの影響: ビジネス要件に応じて適切なサイズを設定してください。不適切な設定はパフォーマンスに影響を与えます。たとえば、値を`1`のように小さく設定すると、1つのブロックごとに1回のネットワーク転送が発生します。値を大きく設定すると、例えば、テーブルの行の合計数の場合、受信側がデータを待つためにほとんどの時間を費やし、パイプライン化された計算が機能しなくなります。適切な値を設定するには、TiFlash受信側が受信する行数の分布を観察できます。ほとんどのスレッドが、たとえば数百行のような少数の行しか受信しない場合、この値を増やしてネットワークオーバーヘッドを減らすことができます。

### tiflash\_fine\_grained\_shuffle\_stream\_count <span class="version-mark">v6.2.0で新規</span> {#tiflash-fine-grained-shuffle-stream-count-span-class-version-mark-new-in-v6-2-0-span}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `0`
- 範囲: `[-1, 1024]`
- ウィンドウ関数がTiFlashにプッシュダウンされて実行される場合、この変数を使用してウィンドウ関数の並列レベルを制御できます。可能な値は次のとおりです。

  - \-1: Fine Grained Shuffle機能が無効になっています。TiFlashにプッシュダウンされたウィンドウ関数は単一スレッドで実行されます。
  - 0: Fine Grained Shuffle機能が有効になっています。[`tidb_max_tiflash_threads`](/system-variables.md#tidb_max_tiflash_threads-new-in-v610)が有効な値（0より大きい）に設定されている場合、`tiflash_fine_grained_shuffle_stream_count`は[`tidb_max_tiflash_threads`](/system-variables.md#tidb_max_tiflash_threads-new-in-v610)の値に設定されます。それ以外の場合、8に設定されます。TiFlash上のウィンドウ関数の実際の並列レベルは、min(`tiflash_fine_grained_shuffle_stream_count`, TiFlashノードの物理スレッド数)です。
  - 0より大きい整数: Fine Grained Shuffle機能が有効になっています。TiFlashにプッシュダウンされたウィンドウ関数は複数のスレッドで実行されます。並列レベルは、min(`tiflash_fine_grained_shuffle_stream_count`, TiFlashノードの物理スレッド数)です。
- 理論的には、この値が増加するとウィンドウ関数のパフォーマンスが線形に向上します。ただし、この値が実際の物理スレッド数を超えると、パフォーマンスの低下につながります。

### time\_zone {#time-zone}

- Scope: SESSION | GLOBAL
- クラスターに永続化: はい
- デフォルト値: `SYSTEM`
- この変数は現在のタイムゾーンを返します。値は、オフセット（たとえば'-8:00'）または名前付きゾーン（'America/Los\_Angeles'）として指定できます。
- 値`SYSTEM`は、タイムゾーンがシステムホストと同じであるべきことを意味し、[`system_time_zone`](#system_time_zone)変数を介して利用できます。

### timestamp {#timestamp}

- Scope: SESSION
- タイプ: 浮動小数点数
- デフォルト値: `0`
- 範囲: `[0, 2147483647]`
- この変数の空でない値は、`CURRENT_TIMESTAMP()`、`NOW()`、およびその他の関数のタイムスタンプとして使用されるUNIXエポックを示します。この変数はデータの復元やレプリケーションで使用される可能性があります。

### トランザクション分離 {#transaction-isolation}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙
- デフォルト値: `REPEATABLE-READ`
- 可能な値: `READ-UNCOMMITTED`, `READ-COMMITTED`, `REPEATABLE-READ`, `SERIALIZABLE`
- この変数はトランザクション分離を設定します。TiDBはMySQLとの互換性のために`REPEATABLE-READ`を宣伝していますが、実際の分離レベルはスナップショット分離です。詳細については、[トランザクション分離レベル](/transaction-isolation-levels.md)を参照してください。

### tx\_isolation {#tx-isolation}

この変数は`transaction_isolation`のエイリアスです。

### tx\_isolation\_one\_shot {#tx-isolation-one-shot}

> **Note:**
>
> この変数はTiDB内部で使用されます。使用することは想定されていません。

TiDBパーサーは、`SET TRANSACTION ISOLATION LEVEL [READ COMMITTED| REPEATABLE READ | ...]`ステートメントを`SET @@SESSION.TX_ISOLATION_ONE_SHOT = [READ COMMITTED| REPEATABLE READ | ...]`に変換します。

### tx\_read\_ts {#tx-read-ts}

- スコープ: SESSION
- デフォルト値: ""
- ステイルリードのシナリオでは、このセッション変数はステーブルリードタイムスタンプ値の記録に使用されます。
- この変数はTiDBの内部操作に使用されます。この変数を設定することは**推奨されません**。

### txn\_scope {#txn-scope}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION
- デフォルト値: `global`
- 値のオプション: `global` および `local`
- この変数は、現在のセッショントランザクションがグローバルトランザクションかローカルトランザクションかを設定するために使用されます。
- この変数はTiDBの内部操作に使用されます。この変数を設定することは**推奨されません**。

### validate\_password.check\_user\_name <span class="version-mark">v6.5.0で新規</span> {#validate-password-check-user-name-span-class-version-mark-new-in-v6-5-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `ON`
- タイプ: ブール値
- この変数はパスワードの複雑さチェックのチェック項目です。パスワードがユーザー名と一致するかどうかをチェックします。この変数は、[`validate_password.enable`](#validate_passwordenable-new-in-v650)が有効になっている場合にのみ有効です。
- この変数が有効で`ON`に設定されている場合、パスワードを設定すると、TiDBはパスワードをユーザー名（ホスト名を除く）と比較します。パスワードがユーザー名と一致する場合、パスワードは拒否されます。
- この変数は[`validate_password.policy`](#validate_passwordpolicy-new-in-v650)には独立しており、パスワードの複雑さチェックレベルには影響されません。

### validate\_password.dictionary <span class="version-mark">v6.5.0で新規</span> {#validate-password-dictionary-span-class-version-mark-new-in-v6-5-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `""`
- タイプ: 文字列
- この変数はパスワードの複雑さチェックのチェック項目です。パスワードが辞書と一致するかどうかをチェックします。この変数は、[`validate_password.enable`](#validate_passwordenable-new-in-v650)が有効になっていて、[`validate_password.policy`](#validate_passwordpolicy-new-in-v650)が`2`（STRONG）に設定されている場合にのみ有効です。
- この変数は1024文字を超えない文字列です。パスワードに存在してはならない単語のリストを含んでいます。各単語はセミコロン（`;`）で区切られています。
- この変数はデフォルトで空の文字列に設定されており、辞書チェックは実行されません。辞書チェックを実行するには、一致させる単語を文字列に含める必要があります。この変数が構成されている場合、パスワードを設定すると、TiDBはパスワードの各部分文字列（長さが4から100文字）を辞書の単語と比較します。パスワードの任意の部分文字列が辞書の単語と一致する場合、パスワードは拒否されます。比較は大文字と小文字を区別しません。

### validate\_password.enable <span class="version-mark">v6.5.0で新規</span> {#validate-password-enable-span-class-version-mark-new-in-v6-5-0-span}

> **Note:**
>
> この変数は常に[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して有効です。

- スコープ: GLOBAL
- クラスターに永続化: はい
- デフォルト値: `OFF`
- タイプ: ブール値
- この変数はパスワードの複雑さチェックを実行するかどうかを制御します。この変数が`ON`に設定されている場合、TiDBはパスワードを設定する際にパスワードの複雑さチェックを実行します。

### validate\_password.length <span class="version-mark">v6.5.0で新規</span> {#validate-password-length-span-class-version-mark-new-in-v6-5-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `8`
- 範囲: TiDB Self-Hostedおよび[TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated)の場合は`[0, 2147483647]`、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)の場合は`[8, 2147483647]`
- この変数はパスワードの複雑さチェックのチェック項目です。パスワードの長さが十分かどうかをチェックします。デフォルトでは、最小パスワード長は`8`です。この変数は、[`validate_password.enable`](#validate_passwordenable-new-in-v650)が有効になっている場合にのみ有効です。
- この変数の値は、`validate_password.number_count + validate_password.special_char_count + (2 * validate_password.mixed_case_count)`の式より小さくしてはいけません。
- `validate_password.number_count`、`validate_password.special_char_count`、または`validate_password.mixed_case_count`の値を変更して、式の値が`validate_password.length`より大きくなるようにした場合、`validate_password.length`の値は自動的に式の値に合わせて変更されます。

### validate\_password.mixed\_case\_count <span class="version-mark">v6.5.0で新規</span> {#validate-password-mixed-case-count-span-class-version-mark-new-in-v6-5-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: TiDB Self-Hostedおよび[TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated)の場合は`[0, 2147483647]`、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)の場合は`[1, 2147483647]`
- この変数はパスワードの複雑さチェックのチェック項目です。パスワードに十分な大文字と小文字が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](#validate_passwordenable-new-in-v650)が有効になっていて、[`validate_password.policy`](#validate_passwordpolicy-new-in-v650)が`1`（MEDIUM）以上に設定されている場合にのみ有効です。
- パスワードに含まれる大文字の数や小文字の数が、`validate_password.mixed_case_count`の値より少なくなってはいけません。たとえば、変数が`1`に設定されている場合、パスワードには少なくとも1つの大文字と1つの小文字が含まれている必要があります。

### validate\_password.number\_count <span class="version-mark">v6.5.0で新規</span> {#validate-password-number-count-span-class-version-mark-new-in-v6-5-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: TiDB Self-Hostedおよび[TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated)の場合は`[0, 2147483647]`、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)の場合は`[1, 2147483647]`
- この変数はパスワードの複雑さチェックのチェック項目です。パスワードに十分な数値が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](#password_reuse_interval-new-in-v650)が有効になっていて、[`validate_password.policy`](#validate_passwordpolicy-new-in-v650)が`1`（MEDIUM）以上に設定されている場合にのみ有効です。

### validate\_password.policy <span class="version-mark">v6.5.0で新規</span> {#validate-password-policy-span-class-version-mark-new-in-v6-5-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 列挙
- デフォルト値: `1`
- 値のオプション: TiDB Self-Hostedおよび[TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated)の場合は`0`、`1`、`2`、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)の場合は`1`、`2`
- この変数はパスワードの複雑さチェックのポリシーを制御します。この変数は、[`validate_password.enable`](#password_reuse_interval-new-in-v650)が有効になっている場合にのみ有効です。この変数の値は、`validate-password`変数以外のパスワードの複雑さチェックにどのように影響するかを決定します。ただし、`validate_password.check_user_name`を除きます。
- この変数の値は`0`、`1`、`2`（LOW、MEDIUM、STRONGに対応）であり、異なるポリシーレベルには異なるチェックがあります:
  - 0またはLOW: パスワードの長さ。
  - 1またはMEDIUM: パスワードの長さ、大文字と小文字の文字、数値、特殊文字。
  - 2またはSTRONG: パスワードの長さ、大文字と小文字の文字、数値、特殊文字、辞書の一致。

### validate\_password.special\_char\_count <span class="version-mark">v6.5.0で新規</span> {#validate-password-special-char-count-span-class-version-mark-new-in-v6-5-0-span}

- スコープ: GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `1`
- 範囲: TiDB Self-Hosted および [TiDB Dedicated](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-dedicated) では `[0, 2147483647]`、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) では `[1, 2147483647]`
- この変数はパスワードの複雑さをチェックする項目です。パスワードに十分な特殊文字が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](#password_reuse_interval-new-in-v650) が有効になっており、[`validate_password.policy`](#validate_passwordpolicy-new-in-v650) が `1` (MEDIUM) またはそれ以上に設定されている場合にのみ有効です。

### version {#version}

- スコープ: NONE
- デフォルト値: `5.7.25-TiDB-`(tidb version)
- この変数は、MySQLのバージョン、その後にTiDBのバージョンを返します。例: '5.7.25-TiDB-v7.1.0'。

### version\_comment {#version-comment}

- スコープ: NONE
- デフォルト値: (string)
- この変数は、TiDBのバージョンに関する追加の詳細を返します。例: 'TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible'。

### version\_compile\_machine {#version-compile-machine}

- スコープ: NONE
- デフォルト値: (string)
- この変数は、TiDBが実行されているCPUアーキテクチャの名前を返します。

### version\_compile\_os {#version-compile-os}

- スコープ: NONE
- デフォルト値: (string)
- この変数は、TiDBが実行されているOSの名前を返します。

### wait\_timeout {#wait-timeout}

> **Note:**
>
> この変数は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)に対して読み取り専用です。

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: 整数
- デフォルト値: `28800`
- 範囲: `[0, 31536000]`
- 単位: 秒
- この変数は、ユーザーセッションのアイドルタイムアウトを制御します。ゼロ値は無制限を意味します。

### warning\_count {#warning-count}

- スコープ: SESSION
- デフォルト値: `0`
- この読み取り専用変数は、以前に実行されたステートメントで発生した警告の数を示します。

### windowing\_use\_high\_precision {#windowing-use-high-precision}

- スコープ: SESSION | GLOBAL
- クラスターに永続化: はい
- タイプ: ブール値
- デフォルト値: `ON`
- この変数は、ウィンドウ関数を計算する際に高精度モードを使用するかどうかを制御します。
