---
title: "環境構築"
---

本書のコードを実際に動かすための環境を整える。はじめにで触れたビルドツールにあたるのが、本書では **cabin** である。cabin はプロジェクトの作成からビルド、実行までをまとめて扱う。

ここで挙げる導入手順やバージョンは、時間とともに変わりうる。最新の正確な手順は、それぞれの公式の案内に従う。

## 必要なもの

本書の環境には、次のものが要る。

- **cabin** 本体。Rust のツールチェーンを入れたうえで、`cargo install cabinpkg` で導入する。導入されるコマンドの名前は `cabin` である。対応する OS は Linux と macOS である。
- cabin がビルドに使う外部のビルドツール。**GCC** や **Clang**、そして **Ninja** である。

本書のコードは `std::println`(ヘッダ `<print>`)を使う。これを使うには、`<print>` に対応したビルドツールが要る(たとえば GCC では 14 以降)。対応していないと、本書のコードはビルドに失敗する。

これらの入れ方は OS や環境によって異なる。cabin の導入は [cabin のドキュメント](https://cabinpkg.com/docs/installation/) に、GCC や Clang、Ninja の導入は各環境の案内に従う。

## cabin を入れる

Rust のツールチェーンがある状態で、次を実行する。

```sh
cargo install cabinpkg
```

導入できたかは、バージョンを表示して確かめられる。

```sh
cabin --version
```

## プロジェクトを作る

`cabin new` で、新しいプロジェクトを作る。ここでは `hello` という名前にする。

```sh
cabin new hello
```

これは `hello` というディレクトリを作り、次のものを置く。

```
hello/
  cabin.toml
  src/
    main.cc
  .gitignore
```

`cabin.toml` は、プロジェクトの設定を書くファイルである。生成された時点では次のようになっている。

```toml
[package]
name = "hello"
version = "0.1.0"

[target.hello]
type = "executable"
sources = ["src/main.cc"]
```

`[package]` はプロジェクトの名前とバージョンを示す。`[target.hello]` はビルドの対象で、`type` が実行可能なプログラム(`executable`)であること、`sources` がその元になるソースファイルであることを示す。ソースは `src/main.cc` に置く。

## C++23 を使えるようにする

cabin が既定で使う C++ の版は C++17 で、このままでは本書が使う `std::println` が使えない。`cabin.toml` に次の `[profile]` を加えて、C++23 を使うように指定する。

```toml
[package]
name = "hello"
version = "0.1.0"

[profile]
cxxflags = ["-std=c++23"]

[target.hello]
type = "executable"
sources = ["src/main.cc"]
```

`cxxflags` に書いた値は、ビルドのときに、cabin が使うビルドツールへそのまま渡される。`-std=c++23` は、使う C++ の版を C++23 にするものである。

## プログラムを動かす

生成された `src/main.cc` を、本書で使う `std::println` の形に書き換える。

```cpp
#include <print>

int main()
{
    std::println("Hello, world!");
}
```

プロジェクトのディレクトリで `cabin run` を実行すると、ビルドしてから実行までが行われる。

```sh
cabin run
```

`Hello, world!` と出力されれば、本書のコードを動かす環境は整っている。ビルドだけを行うなら `cabin build` を使う。ビルドの結果は `build/` ディレクトリに置かれる。

次章からは、こうして動くプログラムが何でできているかを見ていく。
