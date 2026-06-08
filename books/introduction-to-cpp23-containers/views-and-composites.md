---
title: "ビューと複合"
---

前章までで、コンテナは要素を持ち、アルゴリズムはそれを巡って操作した。本章では、要素を所有せずに範囲を参照する `std::span` と、複数の値を一つにまとめる `std::pair`・`std::tuple` を扱う。`std::span` は、文字列の章で見た `std::string_view` の考え方を、文字に限らない要素へ広げたものである。`std::pair`・`std::tuple` は、言語編の構造化束縛で分解できる組を作る。

## 範囲を所有せずに参照する

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

## 複数の値をまとめる

`std::pair<A, B>` は、二つの値を一つにまとめて持つ。型は違っていてよい。使うには `<utility>` を取り込む。中身は、言語編の構造化束縛の章で見た `[ ]` で分けられる。

```cpp
std::pair<std::string, int> p = {"alice", 30};
auto [name, age] = p;
std::println("{} {}", name, age);   // alice 30
```

三つ以上の値をまとめるには `std::tuple` を使う。使うには `<tuple>` を取り込む。

```cpp
std::tuple<int, double, std::string> t = {1, 2.5, "hi"};
auto [a, b, c] = t;
std::println("{} {} {}", a, b, c);   // 1 2.5 hi
```

構造化束縛の名前の数は、まとめた値の数と合わせる。

:::message
まとめた値の数と名前の数が合わない
```cpp
std::tuple<int, double, std::string> t = {1, 2.5, "hi"};
auto [a, b] = t;
```
`t` は三つの値をまとめているのに、名前は二つしかない。数が合わず、ビルドに失敗する。
:::

`std::pair`・`std::tuple` は、関数から複数の値を返すのに使える。範囲の最小値と最大値を一度に返す `minmax` を書く。

```cpp
std::pair<int, int> minmax(std::span<int const> xs)
{
    int lo = xs[0];
    int hi = xs[0];
    for (int x : xs)
    {
        if (x < lo) lo = x;
        if (x > hi) hi = x;
    }
    return {lo, hi};
}
```

```cpp
std::vector<int> v = {3, 1, 4, 1, 5};
auto [lo, hi] = minmax(v);
std::println("{} {}", lo, hi);   // 1 5
```

`minmax` は `std::span<int const>` で範囲を受け取り、最小値と最大値を `std::pair` で返す。呼ぶ側は、構造化束縛で `lo` と `hi` に分けて受け取る。範囲を所有せずに受け取ること、複数の値をまとめて返すこと、まとめた値を分けて受け取ることが、ここで一つにつながる。

ここまでで、値を持つコンテナ、それを巡るアルゴリズム、所有せずに参照するビュー、複数の値をまとめる複合を見てきた。次章では、動的なオブジェクトの所有を引き受ける `std::unique_ptr`・`std::shared_ptr` を見ていく。基本編の new と delete の章で、続きの編で扱うとしたものである。
