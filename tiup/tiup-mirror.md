---
title: Create a Private Mirror
summary: Learn how to create a private mirror.
---

# プライベートミラーの作成 {#create-a-private-mirror}

プライベートクラウドを作成する場合、通常、公式のTiUPミラーにアクセスできない隔離されたネットワーク環境を使用する必要があります。そのため、主に`mirror`コマンドによってプライベートミラーを作成できます。また、オフライン展開にも`mirror`コマンドを使用できます。プライベートミラーを使用すると、自分で構築しパッケージ化したコンポーネントを使用できます。

## TiUP `mirror`の概要 {#tiup-mirror-overview}

`mirror`コマンドのヘルプ情報を取得するには、次のコマンドを実行します。

```bash
tiup mirror --help
```

```bash
`mirror`コマンドは、TiUPのコンポーネントリポジトリを管理するために使用され、プライベートリポジトリを作成したり、既存のリポジトリに新しいコンポーネントを追加したりできます。リポジトリはオンラインまたはオフラインのどちらでも使用できます。また、コンポーネントやリポジトリ自体のキー、ユーザー、バージョンを管理するための便利なユーティリティも提供します。

使用法:
  tiup mirror <command> [flags]

利用可能なコマンド:
  init        空のリポジトリを初期化します
  sign        マニフェストファイルに署名を追加します
  genkey      新しいキーペアを生成します
  clone       ローカルミラーをリモートミラーからクローンし、選択したすべてのコンポーネントをダウンロードします
  merge       2つ以上のオフラインミラーをマージします
  publish     コンポーネントを公開します
  show        ミラーアドレスを表示します
  set         ミラーアドレスを設定します
  modify      公開されたコンポーネントを変更します
  renew       公開されたコンポーネントのマニフェストを更新します。
  grant       新しい所有者に権限を付与します
  rotate      root.jsonを回転します

グローバルフラグ:
      --help                 このコマンドのヘルプ

コマンドについての詳細情報については、"tiup mirror [command] --help"を使用してください。
```

## ミラーのクローン {#clone-a-mirror}

ローカルミラーを構築するには、`tiup mirror clone`コマンドを実行できます。

```bash
tiup mirror clone <target-dir> [global-version] [flags]
```

- `target-dir`: クローンされたデータが保存されるディレクトリを指定するために使用されます。
- `global-version`: すべてのコンポーネントに対して迅速にグローバルバージョンを設定するために使用されます。

`tiup mirror clone`コマンドには多くのオプションフラグ（将来的にはさらに多くのフラグが提供されるかもしれません）があります。これらのフラグは、意図された使用方法に応じて次のカテゴリに分類できます。

- クローン時にプレフィックス一致を使用するかどうかを決定する

  `--prefix`フラグが指定されている場合、クローン時にバージョン番号がプレフィックスに一致するようになります。たとえば、"v5.0.0"と指定した場合、"v5.0.0-rc"と"v5.0.0"が一致します。

- フルクローンを使用するかどうかを決定する

  `--full`フラグを指定すると、公式ミラーを完全にクローンできます。

  > **Note:**
  >
  > `--full`、`global-version`フラグ、およびコンポーネントのバージョンが指定されていない場合、メタ情報のみがクローンされます。

- 特定のプラットフォームからパッケージをクローンするかどうかを決定する

  特定のプラットフォームのパッケージのみをクローンする場合は、`-os`および`-arch`を使用してプラットフォームを指定します。たとえば:

  - `tiup mirror clone <target-dir> [global-version] --os=linux`コマンドを実行して、Linux用にクローンします。
  - `tiup mirror clone <target-dir> [global-version] --arch=amd64`コマンドを実行して、amd64用にクローンします。
  - `tiup mirror clone <target-dir> [global-version] --os=linux --arch=amd64`コマンドを実行して、Linux/amd64用にクローンします。

- パッケージの特定のバージョンをクローンするかどうかを決定する

  コンポーネントのすべてのバージョンではなく、特定のバージョンのみをクローンする場合は、`--<component>=<version>`を使用してこのバージョンを指定します。たとえば:

  - `tiup mirror clone <target-dir> --tidb v7.1.3`コマンドを実行して、TiDBコンポーネントのv7.1.3バージョンをクローンします。
  - `tiup mirror clone <target-dir> --tidb v7.1.3 --tikv all`コマンドを実行して、TiDBコンポーネントのv7.1.3バージョンとすべてのTiKVコンポーネントのバージョンをクローンします。
  - `tiup mirror clone <target-dir> v7.1.3`コマンドを実行して、クラスター内のすべてのコンポーネントのv7.1.3バージョンをクローンします。

クローン後、署名キーが自動的に設定されます。

### プライベートリポジトリの管理 {#manage-the-private-repository}

`tiup mirror clone`でクローンしたリポジトリを、SCP、NFSを介してファイルを共有するか、HTTPまたはHTTPSプロトコルを介してリポジトリを利用可能にすることで、ホスト間で共有できます。リポジトリの場所を指定するには、`tiup mirror set <location>`を使用します。

```bash
tiup mirror set /shared_data/tiup
```

```bash
tiup mirror set https://tiup-mirror.example.com/
```

> **Note:**
>
> `tiup mirror clone`を実行するマシンで`tiup mirror set...`を実行した場合、次回`tiup mirror clone...`を実行すると、マシンはリモートミラーではなくローカルミラーからクローンします。そのため、プライベートミラーを更新する前に、`tiup mirror set --reset`を実行してミラーをリセットする必要があります。

ミラーを使用する別の方法は、`TIUP_MIRRORS`環境変数を使用することです。プライベートリポジトリを使用して`tiup list`を実行する例を以下に示します。

```bash
export TIUP_MIRRORS=/shared_data/tiup
tiup list
```

`TIUP_MIRRORS`の設定は、永続的にミラーの構成を変更できます。詳細については、[tiup issue #651](https://github.com/pingcap/tiup/issues/651)を参照してください。

### プライベートリポジトリの更新 {#update-the-private-repository}

同じ`target-dir`で`tiup mirror clone`コマンドを再度実行すると、マシンは新しいマニフェストを作成し、利用可能なコンポーネントの最新バージョンをダウンロードします。

> **Note:**
>
> マニフェストを再作成する前に、以前にダウンロードしたすべてのコンポーネントとバージョンが含まれていることを確認してください。

## カスタムリポジトリ {#custom-repository}

自分で構築したTiDBコンポーネント（TiDB、TiKV、またはPDなど）と一緒に使用するために、カスタムリポジトリを作成できます。また、独自のtiupコンポーネントを作成することも可能です。

独自のコンポーネントを作成するには、`tiup package`コマンドを実行し、[Component packaging](https://github.com/pingcap/tiup/blob/master/doc/user/package.md)で指示された手順を実行します。

### カスタムリポジトリの作成 {#create-a-custom-repository}

`/data/mirror`に空のリポジトリを作成するには、次のコマンドを実行します。

```bash
tiup mirror init /data/mirror
```

リポジトリを作成する際に、キーが`/data/mirror/keys`に書き込まれます。

新しいプライベートキーを`~/.tiup/keys/private.json`に作成するには、次のコマンドを実行します。

```bash
tiup mirror genkey
```

`jdoe`に`~/.tiup/keys/private.json`のプライベートキーの所有権を`/data/mirror`に付与するには、次のコマンドを実行します。

```bash
tiup mirror set /data/mirror
tiup mirror grant jdoe
```

### カスタムコンポーネントの操作 {#work-with-custom-components}

1. helloというカスタムコンポーネントを作成します。

   ```bash
   $ cat > hello.c << END
   > #include <stdio.h>
   int main() {
     printf("hello\n");
     return (0);
   }
   END
   $ gcc hello.c -o hello
   $ tiup package hello --entry hello --name hello --release v0.0.1
   ```

   `package/hello-v0.0.1-linux-amd64.tar.gz`が作成されます。

2. リポジトリとプライベートキーを作成し、リポジトリに所有権を付与します。

   ```bash
   $ tiup mirror init /tmp/m
   $ tiup mirror genkey
   $ tiup mirror set /tmp/m
   $ tiup mirror grant $USER
   ```

   ```bash
   tiup mirror publish hello v0.0.1 package/hello-v0.0.1-linux-amd64.tar.gz hello
   ```

3. コンポーネントを実行します。まだインストールされていない場合は、まずダウンロードされます。

   ```bash
   $ tiup hello
   ```

   ```
   コンポーネント`hello`のバージョンがインストールされていません。リポジトリからダウンロードしています。
   コンポーネント`hello`の開始: /home/dvaneeden/.tiup/components/hello/v0.0.1/hello
   hello
   ```

   `tiup mirror merge`を使用すると、カスタムコンポーネントが含まれるリポジトリを別のリポジトリにマージできます。これには、`/data/my_custom_components`のすべてのコンポーネントが現在の`$USER`によって署名されていることが前提です。

   ```bash
   $ tiup mirror set /data/my_mirror
   $ tiup mirror grant $USER
   $ tiup mirror merge /data/my_custom_components
   ```
