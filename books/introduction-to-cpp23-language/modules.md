---
title: "モジュール"
---

基本編から、宣言や定義を別のファイルに置き、`#include` で取り込んできた。`#include` は、ヘッダの中身をその場に貼り付ける指示である。ODR と inline の章で見たように、定義をヘッダに置くと、取り込んだ翻訳単位ごとに同じ定義が現れ、重複する。C++20 の **モジュール** は、別のやり方を与える。モジュールの中身は `import` で取り込む。`#include` のようにテキストを貼り付け、翻訳単位ごとに読み直すのではない。本章では、モジュールを書いて取り込むこと、`export` による内と外の区切り、グローバルモジュールフラグメント、そして基本編で据えた「宣言の並び」という見方との関わりを扱う。

## モジュールを書いて取り込む

値を二倍にする `twice` を、モジュールとして書く。

```cpp:greeter.cppm
export module greeter;

export int twice(int n)
{
    return n * 2;
}
```

`export module greeter;` で、`greeter` という名前のモジュールを宣言する。これを書いたファイルを **モジュールインターフェース単位** と呼ぶ。宣言に `export` を付けると、このモジュールを `import` した側から見える。

別のファイルから、`greeter` を取り込んで使う。

```cpp:main.cc
#include <print>
import greeter;

int main()
{
    std::println("{}", twice(21));   // 42
}
```

`import greeter;` で、`greeter` モジュールを取り込む。`export` された `twice` が使える。`#include` と違い、`greeter` の中身はテキストとして貼り付けられない。コンパイル済みの `greeter` が取り込まれる。

## export で内と外を分ける

モジュールの中の宣言のうち、`export` を付けたものだけが、`import` した側から見える。付けないものは、モジュールの中だけで使える。

```cpp:math.cppm
export module math;

int helper(int n) { return n + 1; }              // モジュールの中だけ
export int next(int n) { return helper(n); }     // import 側から見える
```

```cpp:main.cc
#include <print>
import math;

int main()
{
    std::println("{}", next(10));   // 11
}
```

`next` は `export` したので、`import` した側から呼べる。`next` は中で `helper` を使うが、`helper` は `export` していないので、`math` の外からは見えない。

:::message
export していない名前を外から使う
```cpp:main.cc
import math;

int main()
{
    helper(10);
}
```
`helper` は `export` されていないので、`math` を `import` した側からは見えない。ビルドに失敗する。
:::

## グローバルモジュールフラグメント

モジュールの中で `#include` を使いたいことがある。たとえば、`<print>` のようなヘッダを取り込むときである。`#include` は、モジュールの宣言より前の **グローバルモジュールフラグメント** に置く。`module;` だけの行で始める。

```cpp:log.cppm
module;
#include <print>
export module log;

export void note(int n)
{
    std::println("note: {}", n);
}
```

`module;` から `export module log;` までがグローバルモジュールフラグメントで、ここに `#include` を書く。`note` は `std::println` を使えるが、`<print>` で取り込んだ名前は、`log` を `import` した側には漏れない。モジュールは、取り込んだヘッダの中身を外へ流さない。

```cpp:main.cc
import log;

int main()
{
    note(7);   // note: 7
}
```

:::details モジュールパーティション
大きなモジュールは、パーティションに分けて書ける。`export module m:part;` でパーティションを書き、本体の側で `export import :part;` とまとめる。

```cpp:m-part.cppm
export module m:part;
export int forty() { return 40; }
```

```cpp:m.cppm
export module m;
export import :part;
export int two() { return 2; }
```
`import m;` した側からは、`forty` も `two` も見える。パーティションは、一つのモジュールを複数のファイルに分けるための仕組みである。
:::

## 宣言の並びとモジュール

基本編では、プログラムを宣言の並びとみる見方を主軸に据え、モジュールを使う翻訳単位はその見方に収まらないと断った。モジュールインターフェース単位は、`export module greeter;` のようなモジュールの宣言や、グローバルモジュールフラグメントを含む。ふつうの宣言の並びに、モジュール固有の構成要素が加わる。基本編が宣言の並びという見方を保つためにモジュールを外していたのは、このためである。

モジュールは C++20 で入ったが、コンパイラやビルドツールの対応は、まだ進んでいる途中である。標準ライブラリをまとめて取り込む `import std;` や、一部の機能は、環境によっては使えないことがある。
