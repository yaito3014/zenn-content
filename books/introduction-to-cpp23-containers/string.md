---
title: "文字列"
---

基本編の整数の型の章で、`char` が一つの文字を表すと見た。文字を並べた文字列を一つの値として扱う型は、標準ライブラリが提供すると断り、基本編では扱わなかった。それが `std::string` である。本章では、`std::string` で文字列を持つこと、つなげたり調べたりする操作、そして文字列を所有せずに参照する `std::string_view` を扱う。

## 文字列を持つ

`std::string` は、文字の並びを一つの値として持つ型である。使うには `<string>` を取り込む。

```cpp
std::string s = "hello";
std::println("{}", s);   // hello
```

`std::string s = "hello";` は、リテラル `"hello"` の文字を `s` の持つ領域に複製して格納する。`std::string` は、こうして自分が持つ文字を格納する領域を引き受け、文字を足せば必要に応じて伸びる。基本編の配列の章で見た、大きさの決まった `char` の並びと違い、長さをあらかじめ決めておく必要はない。

## つなげる・調べる

`std::string` どうしは `+` でつなげられる。これは、言語編の演算子オーバーロードの章で見た、型に与えられた `+` である。

```cpp
std::string greeting = "Hello";
std::string name = "world";
std::string message = greeting + ", " + name;
std::println("{}", message);         // Hello, world
std::println("{}", message.size());  // 12
```

`size()` は、文字列の長さ、すなわち持っている `char` の数を返す。`message` は12個の `char` を持つので `12` になる。

個々の文字は、添字で取り出せる。`message[0]` は先頭の文字 `H` である。添字は `0` から始まり、最後の文字は `size() - 1` にある。

`[]` は、添字が範囲の内にあるかを確かめない。`size()` を超える添字を渡すと、文字列の外を読むことになる。

:::message alert
範囲の外を添字で読む
```cpp
std::string s = "hi";
std::println("{}", s[5]);
```
`s` は2文字なので、添字 `5` は範囲の外にある。`[]` は範囲を確かめないので、ビルドは通るが、文字列の外を読む。結果は定まらず、実行時に深刻な問題が起こりうる。
:::

範囲を確かめてほしいときは、`at` を使う。`at` は、添字が範囲の外なら `std::out_of_range` を投げる。これは、言語編の例外の章で見た例外で、`try`/`catch` で捕まえられる。

```cpp
std::string s = "hi";
s.at(5);   // std::out_of_range を投げる
```

## 非所有のビュー

文字列を読むだけの関数に `std::string` を値で渡すと、その文字列が丸ごと複製される。`std::string const&` で参照を渡せば複製は避けられるが、文字列リテラルを渡すと、いったん `std::string` が作られて、やはり複製が起きる。読むだけなら、由来によらず複製せずに中身を参照したい。そのための型が `std::string_view` である。使うには `<string_view>` を取り込む。

`std::string_view` は、文字の並びを所有せず、参照するだけの型である。`std::string` からも、文字列リテラルからも、複製せずに作れる。

```cpp
std::size_t count(std::string_view sv)
{
    return sv.size();
}
```

```cpp
std::string s = "hello";
count(s);            // 5    std::string から
count("hi there");   // 8    文字列リテラルから
```

`count` は文字数を返すだけで、文字列を書き換えない。`std::string_view` は、参照先の文字を読めるが、書き換えはできない。

:::message
ビューを通して書き換える
```cpp
void f(std::string_view sv)
{
    sv[0] = 'x';
}
```
`std::string_view` は参照するだけの型で、参照先の文字は書き換えられない。ビルドに失敗する。
:::

`std::string_view` は参照先を所有しないので、参照先の寿命を延ばさない。言語編のオブジェクトの寿命の章で見たとおり、参照先が破棄されれば、そこを指すものは、もう存在しないものを指す。

:::message alert
破棄される文字列を指すビュー
```cpp
std::string_view make()
{
    std::string s = "hello";
    return s;
}
```
`make` は、局所変数 `s` の文字を指すビューを返す。`s` はこの関数を抜けると破棄され、返ってきたビューは、もう存在しない文字を指す。ビルドは通るが、そのビューを使うと、結果は定まらず、実行時に深刻な問題が起こりうる。
:::

文字を後々まで持っておきたいときは、それを所有する `std::string` を持つ。`std::string_view` は、別の場所にあって、ビューより長く生きる文字を、複製せずに読むために使う。