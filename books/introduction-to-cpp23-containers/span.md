---
title: "範囲を所有せずに参照する"
---

前章までで、コンテナとアルゴリズム、そして範囲から範囲を導くビューを見てきた。いずれも範囲を巡るものだった。本章では、要素を所有せずに範囲を参照する `std::span` を扱う。`std::span` は、文字列の章で見た `std::string_view` の考え方を、文字に限らない要素へ広げたものである。

整数の並びを読むだけの関数を書きたいとする。`std::vector<int> const&` で受け取ると、`std::array` や生の配列を渡せない。要素が並んでいる範囲なら、何に持たれているかを問わずに受け取りたい。そのための型が `std::span` である。使うには `<span>` を取り込む。

`std::span<int const>` は、連続して並んだ `int` の範囲を、所有せずに参照する。`std::vector` でも `std::array` でも、要素が連続して並んでいれば、複製せずに渡せる。

```cpp
int sum(std::span<int const> xs)
{
    int total = 0;
    for (int x : xs) total += x;
    return total;
}
```

```cpp
std::vector<int> v = {1, 2, 3};
std::array<int, 4> a = {4, 5, 6, 7};
sum(v);   // 6
sum(a);   // 22
```

## 読むだけと書き換え

`std::string_view` は常に読むだけだったが、`std::span` は要素の型で読み書きが分かれる。要素の型を `int const` にすると読むだけ、`int` にすると書き換えもできる。

```cpp
void double_all(std::span<int> xs)
{
    for (int& x : xs) x *= 2;
}
```

```cpp
std::vector<int> v = {1, 2, 3};
double_all(v);
for (int x : v) std::println("{}", x);   // 2 4 6
```

## 長さを型に固定する

ここまでは要素の型に注目してきたが、`std::span` にはもう一つ、長さという軸がある。これまでの `std::span` は、長さを実行時に持ち、どんな長さの範囲でも参照できた。`std::span` のこの長さを **extent** と呼ぶ。長さは、型に固定することもできる。`std::span<int, 3>` は、ちょうど3つの要素を参照する `std::span` で、長さが型の一部になっている。`std::span<int>` のように長さを実行時に持つものを動的な extent、`std::span<int, 3>` のように型に固定したものを静的な extent という。

```cpp
std::array<int, 3> a = {1, 2, 3};
std::span<int, 3> s = a;
std::println("{}", s.size());   // 3
```

`std::span<int, 3>` の長さ `3` は型に書かれていて、`size()` もその `3` を返す。長さが型に含まれるので、長さの合わない範囲を渡すと、ビルドの時点で分かる。

:::message
長さの合わない範囲を静的な extent の span にする
```cpp
std::array<int, 4> a = {1, 2, 3, 4};
std::span<int, 3> s = a;
```
`std::span<int, 3>` はちょうど3つを参照するが、`a` は4つある。長さが合わず、ビルドに失敗する。
:::

## 破棄される範囲を指す

`std::span` は参照先を所有しない。`std::string_view` と同じく、言語編のオブジェクトの寿命の章で見たとおり、参照先が破棄されれば、`std::span` はもう存在しない要素を指す。

:::message alert
破棄される範囲を指す span
```cpp
std::span<int const> make()
{
    std::vector<int> v = {1, 2, 3};
    return v;
}
```
`make` は、局所変数 `v` の要素を指す `std::span` を返す。`v` はこの関数を抜けると破棄され、返ってきた `std::span` は、もう存在しない要素を指す。ビルドは通るが、その `std::span` を使うと、結果は定まらず、実行時に深刻な問題が起こりうる。
:::

`std::span` は、範囲を所有せずに参照した。次章では、複数の値を一つにまとめる `std::pair`・`std::tuple` を見ていく。
