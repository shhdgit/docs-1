---
title: Connect to TiDB with MyBatis
summary: Learn how to connect to TiDB using MyBatis. This tutorial gives Java sample code snippets that work with TiDB using MyBatis.
---

# TiDBとMyBatisを接続する {#connect-to-tidb-with-mybatis}

TiDBはMySQL互換のデータベースであり、[MyBatis](https://mybatis.org/mybatis-3/index.html)は人気のあるオープンソースのJava ORMです。

このチュートリアルでは、TiDBとMyBatisを使用して次のタスクを実行する方法を学ぶことができます。

- 環境を設定します。
- MyBatisを使用してTiDBクラスタに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **Note:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。

## 前提条件 {#prerequisites}

このチュートリアルを完了するには、次のものが必要です。

- **Java Development Kit (JDK) 17** 以上。ビジネスおよび個人の要件に基づいて、[OpenJDK](https://openjdk.org/)または[Oracle JDK](https://www.oracle.com/hk/java/technologies/downloads/)を選択できます。
- [Maven](https://maven.apache.org/install.html) **3.8** 以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する {#run-the-sample-app-to-connect-to-tidb}

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする {#step-1-clone-the-sample-app-repository}

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-java-mybatis-quickstart.git
cd tidb-java-mybatis-quickstart
```

### ステップ2：接続情報の構成 {#step-2-configure-connection-information}

選択したTiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅にある **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境に一致していることを確認します。

   - **Endpoint Type** が `Public` に設定されていること

   - **Branch** が `main` に設定されていること

   - **Connect With** が `General` に設定されていること

   - **Operating System** が環境に一致していること

   > **ヒント：**
   >
   > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. **Generate Password** をクリックしてランダムなパスワードを作成します。

   > **ヒント：**
   >
   > 以前にパスワードを作成した場合は、元のパスワードを使用するか、**Reset Password** をクリックして新しいパスワードを生成できます。

5. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します：

   ```shell
   cp env.sh.example env.sh
   ```

6. 対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```shell
   export TIDB_HOST='{host}'  # 例：gateway01.ap-northeast-1.prod.aws.tidbcloud.com
   export TIDB_PORT='4000'
   export TIDB_USER='{user}'  # 例：xxxxxx.root
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='true'
   ```

   接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

   TiDB Serverlessでは安全な接続が必要です。そのため、`USE_SSL` の値を `true` に設定する必要があります。

7. `env.sh` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅にある **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します：

   ```shell
   cp env.sh.example env.sh
   ```

5. 対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```shell
   export TIDB_HOST='{host}'  # 例：tidb.xxxx.clusters.tidb-cloud.com
   export TIDB_PORT='4000'
   export TIDB_USER='{user}'  # 例：root
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='false'
   ```

   接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

6. `env.sh` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します：

   ```shell
   cp env.sh.example env.sh
   ```

2. 対応する接続文字列を `env.sh` ファイルにコピーして貼り付けます。例の結果は次のとおりです：

   ```shell
   export TIDB_HOST='{host}'
   export TIDB_PORT='4000'
   export TIDB_USER='root'
   export TIDB_PASSWORD='{password}'
   export TIDB_DB_NAME='test'
   export USE_SSL='false'
   ```

   接続パラメータでプレースホルダ `{}` を置き換え、`USE_SSL` を `false` に設定してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` で、パスワードは空です。

3. `env.sh` ファイルを保存します。

</div>
</SimpleTab>

### ステップ3：コードを実行して結果を確認する {#step-3-run-the-code-and-check-the-result}

1. 次のコマンドを実行してサンプルコードを実行します：

   ```shell
   make
   ```

2. 出力が一致するかどうかを確認するために [Expected-Output.txt](https://github.com/tidb-samples/tidb-java-mybatis-quickstart/blob/main/Expected-Output.txt) を確認します。

## サンプルコードスニペット {#sample-code-snippets}

独自のアプリケーション開発を完了するためのサンプルコードスニペットを参照できます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-java-mybatis-quickstart](https://github.com/tidb-samples/tidb-java-mybatis-quickstart) リポジトリをチェックしてください。

### TiDBに接続 {#connect-to-tidb}

MyBatis構成ファイル `mybatis-config.xml` を編集します：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="lazyLoadingEnabled" value="false"/>
        <setting name="aggressiveLazyLoading" value="true"/>
        <setting name="logImpl" value="LOG4J"/>
    </settings>

    <environments default="development">
        <environment id="development">
            <!-- JDBC transaction manager -->
            <transactionManager type="JDBC"/>
            <!-- Database pool -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="${tidb_jdbc_url}"/>
                <property name="username" value="${tidb_user}"/>
                <property name="password" value="${tidb_password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="${mapper_location}.xml"/>
    </mappers>
</configuration>
```

`${tidb_jdbc_url}`, `${tidb_user}`, そして `${tidb_password}` を TiDB クラスタの実際の値に置き換えてください。また、`${mapper_location}` をマッパー XML 構成ファイルのパスに置き換えてください。複数のマッパー XML 構成ファイルの場合、それぞれに `<mapper/>` タグを追加する必要があります。その後、次の関数を定義します。

```java
public SqlSessionFactory getSessionFactory() {
    InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
}
```

### データの挿入 {#insert-data}

マッパーXMLにノードを追加し、XML構成ファイルの`mapper.namespace`属性で構成されたインターフェースクラスに同じ名前の関数を追加します。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
    <insert id="insert" parameterType="com.pingcap.model.Player">
    insert into player (id, coins, goods)
    values (#{id,jdbcType=VARCHAR}, #{coins,jdbcType=INTEGER}, #{goods,jdbcType=INTEGER})
    </insert>
</mapper>
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ {#query-data}

マッパーXMLにノードを追加し、XML構成ファイルの`mapper.namespace`属性で構成されたインターフェースクラスに同じ名前の関数を追加します。特に、MyBatisクエリ関数の戻り値として`resultMap`を使用する場合は、`<resultMap/>`ノードが正しく構成されていることを確認してください。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
    <resultMap id="BaseResultMap" type="com.pingcap.model.Player">
        <constructor>
            <idArg column="id" javaType="java.lang.String" jdbcType="VARCHAR" />
            <arg column="coins" javaType="java.lang.Integer" jdbcType="INTEGER" />
            <arg column="goods" javaType="java.lang.Integer" jdbcType="INTEGER" />
        </constructor>
    </resultMap>

    <select id="selectByPrimaryKey" parameterType="java.lang.String" resultMap="BaseResultMap">
    select id, coins, goods
    from player
    where id = #{id,jdbcType=VARCHAR}
    </select>
</mapper>
```

詳細については、[Query data](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新 {#update-data}

マッパーXMLにノードを追加し、XML構成ファイルの`mapper.namespace`属性で構成されたインターフェースクラスに同じ名前の関数を追加します。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
    <update id="updateByPrimaryKey" parameterType="com.pingcap.model.Player">
    update player
    set coins = #{coins,jdbcType=INTEGER},
      goods = #{goods,jdbcType=INTEGER}
    where id = #{id,jdbcType=VARCHAR}
    </update>
</mapper>
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除 {#delete-data}

マッパーXMLにノードを追加し、XML構成ファイルの`mapper.namespace`属性で構成されたインターフェースクラスに同じ名前の関数を追加します。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
    <delete id="deleteByPrimaryKey" parameterType="java.lang.String">
    delete from player
    where id = #{id,jdbcType=VARCHAR}
    </delete>
</mapper>
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ {#next-steps}

- [MyBatisのドキュメント](http://www.mybatis.org/mybatis-3/)からMyBatisのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を使用して、TiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を受講し、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。
- Java開発者向けのコースを受講し、[JavaからTiDBと連携する方法](https://eng.edu.pingcap.com/catalog/info/id:212)を学びます。

## ヘルプが必要ですか？ {#need-help}

[Discord](https://discord.gg/vYU9h56kAX)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
