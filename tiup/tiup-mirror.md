---
title: Create a Private Mirror
summary: Learn how to create a private mirror.
---

# プライベートミラーの作成 {#create-a-private-mirror}

プライベートクラウドを作成する際には、通常、公式のTiUPミラーにアクセスできない隔離されたネットワーク環境を使用する必要があります。そのため、`mirror`コマンドを使用してプライベートミラーを作成することができます。また、オフライン展開にも`mirror`コマンドを使用することができます。プライベートミラーを使用することで、自分でビルドしてパッケージ化したコンポーネントを使用することもできます。

## TiUP `mirror`の概要 {#tiup-mirror-overview}

以下のコマンドを実行して、`mirror`コマンドのヘルプ情報を取得します：

```bash
tiup mirror --help
```

```bash
The `mirror` command is used to manage a component repository for TiUP, you can use
it to create a private repository, or to add new component to an existing repository.
The repository can be used either online or offline.
It also provides some useful utilities to help manage keys, users, and versions
of components or the repository itself.

Usage:
  tiup mirror <command> [flags]

Available Commands:
  init        Initialize an empty repository
  sign        Add signatures to a manifest file
  genkey      Generate a new key pair
  clone       Clone a local mirror from a remote mirror and download all selected components
  merge       Merge two or more offline mirrors
  publish     Publish a component
  show        Show the mirror address
  set         Set mirror address
  modify      Modify published component
  renew       Renew the manifest of a published component.
  grant       grant a new owner
  rotate      Rotate root.json

Global Flags:
      --help                 Help for this command

Use "tiup mirror [command] --help" for more information about a command.
```

## ミラーのクローン {#clone-a-mirror}

ローカルミラーを構築するには、`tiup mirror clone`コマンドを実行できます。

```bash
tiup mirror clone <target-dir> [global-version] [flags]
```

- `target-dir`：クローンされたデータが保存されるディレクトリを指定するために使用されます。
- `global-version`：すべてのコンポーネントに対してグローバルバージョンを迅速に設定するために使用されます。

`tiup mirror clone`コマンドには多くのオプションフラグが用意されています（将来的にはさらに多くのフラグが用意されるかもしれません）。これらのフラグは、以下のカテゴリに分類することができます。

- クローン時にバージョンをプレフィックスでマッチングするかどうかを決定する

  `--prefix`フラグが指定された場合、バージョン番号はプレフィックスによってマッチングされます。例えば、`--prefix`を"v5.0.0"と指定した場合、"v5.0.0-rc"や"v5.0.0"がマッチングされます。

- フルクローンを使用するかどうかを決定する

  `--full`フラグを指定すると、公式ミラーを完全にクローンすることができます。

  > **Note:**
  >
  > `--full`、`global-version`フラグ、およびコンポーネントのバージョンが指定されていない場合、メタ情報のみがクローンされます。

- 特定のプラットフォームからパッケージをクローンするかどうかを決定する

  特定のプラットフォームのパッケージのみをクローンしたい場合は、`-os`と`-arch`を使用してプラットフォームを指定します。例えば：

  - `tiup mirror clone <target-dir> [global-version] --os=linux`コマンドを実行して、linux用にクローンします。
  - `tiup mirror clone <target-dir> [global-version] --arch=amd64`コマンドを実行して、amd64用にクローンします。
  - `tiup mirror clone <target-dir> [global-version] --os=linux --arch=amd64`コマンドを実行して、linux/amd64用にクローンします。

- 特定のバージョンのパッケージをクローンするかどうかを決定する

  コンポーネントのすべてのバージョンではなく、特定のバージョンのみをクローンしたい場合は、`--<component>=<version>`を使用してこのバージョンを指定します。例えば：

  - `tiup mirror clone <target-dir> --tidb v7.1.3`コマンドを実行して、TiDBコンポーネントのv7.1.3バージョンをクローンします。
  - `tiup mirror clone <target-dir> --tidb v7.1.3 --tikv all`コマンドを実行して、TiDBコンポーネントのv7.1.3バージョンとすべてのTiKVコンポーネントのバージョンをクローンします。
  - `tiup mirror clone <target-dir> v7.1.3`コマンドを実行して、クラスター内のすべてのコンポーネントのv7.1.3バージョンをクローンします。

クローン後、署名キーが自動的に設定されます。

### プライベートリポジトリの管理 {#manage-the-private-repository}

`tiup mirror clone`でクローンされたリポジトリは、SCP、NFSを介してファイルを共有するか、HTTPまたはHTTPSプロトコルを介してリポジトリを利用できるようにすることで、ホスト間で共有することができます。`tiup mirror set <location>`を使用して、リポジトリの場所を指定します。

```bash
tiup mirror set /shared_data/tiup
```

```bash
tiup mirror set https://tiup-mirror.example.com/
```

> **注意:**
>
> `tiup mirror clone`を実行するマシンで`tiup mirror set...`を実行すると、次回`tiup mirror clone...`を実行するときに、マシンはリモートではなくローカルミラーからクローンします。そのため、プライベートミラーを更新する前に、`tiup mirror set --reset`を実行してミラーをリセットする必要があります。

ミラーを使用する別の方法は、`TIUP_MIRRORS`環境変数を使用することです。ここでは、プライベートリポジトリを使用して`tiup list`を実行する例を示します。

```bash
export TIUP_MIRRORS=/shared_data/tiup
tiup list
```

`TIUP_MIRRORS`設定は、例えば`tiup mirror set`のようにミラーの構成を永久的に変更することができます。詳細については、[tiup issue #651](https://github.com/pingcap/tiup/issues/651)を参照してください。

### プライベートリポジトリの更新 {#update-the-private-repository}

同じ`target-dir`で`tiup mirror clone`コマンドを再実行すると、マシンは新しいマニフェストを作成し、利用可能なコンポーネントの最新バージョンをダウンロードします。

> **Note:**
>
> マニフェストを再作成する前に、すべてのコンポーネントとバージョン（以前にダウンロードしたものも含む）が含まれていることを確認してください。

## カスタムリポジトリ {#custom-repository}

自分でビルドしたTiDB、TiKV、またはPDなどのTiDBコンポーネントと一緒に使用するためのカスタムリポジトリを作成することができます。また、独自のtiupコンポーネントを作成することもできます。

独自のコンポーネントを作成するには、`tiup package`コマンドを実行し、[コンポーネントのパッケージング](https://github.com/pingcap/tiup/blob/master/doc/user/package.md)の指示に従ってください。

### カスタムリポジトリの作成 {#create-a-custom-repository}

`/data/mirror`に空のリポジトリを作成するには：

```bash
tiup mirror init /data/mirror
```

リポジトリを作成する際、キーは`/data/mirror/keys`に書き込まれます。

`~/.tiup/keys/private.json`に新しいプライベートキーを作成するには：

```bash
tiup mirror genkey
```

`jdoe`に`~/.tiup/keys/private.json`のプライベートキーを使用して`/data/mirror`の所有権を付与する：

```bash
tiup mirror set /data/mirror
tiup mirror grant jdoe
```

### カスタムコンポーネントを使用する {#work-with-custom-components}

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

2. リポジトリとプライベートキーを作成し、リポジトリの所有権を付与します。

   ```bash
   $ tiup mirror init /tmp/m
   $ tiup mirror genkey
   $ tiup mirror set /tmp/m
   $ tiup mirror grant $USER
   ```

   ```bash
   tiup mirror publish hello v0.0.1 package/hello-v0.0.1-linux-amd64.tar.gz hello
   ```

3. コンポーネントを実行します。まだインストールされていない場合は、最初にダウンロードされます。

   ```bash
   $ tiup hello
   ```

       コンポーネント `hello` のバージョンはインストールされていません。リポジトリからダウンロードしています。
       コンポーネント `hello` を起動中: /home/dvaneeden/.tiup/components/hello/v0.0.1/hello
       hello

   `tiup mirror merge`を使用すると、カスタムコンポーネントを含むリポジトリを別のリポジトリにマージできます。これには、`/data/my_custom_components`内のすべてのコンポーネントが現在の`$USER`によって署名されていることが前提です。

   ```bash
   $ tiup mirror set /data/my_mirror
   $ tiup mirror grant $USER
   $ tiup mirror merge /data/my_custom_components
   ```
