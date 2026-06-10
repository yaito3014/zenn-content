---
title: "複数の値をまとめる"
---

前章の `std::span` は、一つの範囲を参照した。今度は、複数の値を一つにまとめて持つ `std::pair` と `std::tuple` を見る。

`std::pair<A, B>` は、二つの値を一つにまとめて持つ。型は違っていてよい。使うには `<utility>` を取り込む。三つ以上の値をまとめるには `std::tuple` を使い、`<tuple>` を取り込む。

```cpp
std::pair<std::string, int> p = {"alice", 30};
std::tuple<int, double, std::string> t = {1, 2.5, "hi"};
```

`p` は文字列と整数を、`t` は整数・実数・文字列をまとめて持つ。

## 構造化束縛で分ける

まとめた値は、言語編の構造化束縛で取り出す。言語編の構造化束縛の章では、構造体と配列を `auto [a, b]` で分解した。`std::pair` や `std::tuple` も、同じ構造化束縛で分解できる。

```cpp
auto [name, age] = p;
std::println("{} {}", name, age);   // alice 30

auto [a, b, c] = t;
std::println("{} {} {}", a, b, c);   // 1 2.5 hi
```

`auto [name, age] = p` は、`p` の二つの値を `name` と `age` に分ける。`auto [a, b, c] = t` は、`t` の三つの値を分ける。名前の数は、まとめた値の数と合わせる。

:::message
まとめた値の数と名前の数が合わない
```cpp
std::tuple<int, double, std::string> t = {1, 2.5, "hi"};
auto [a, b] = t;
```
`t` は三つの値をまとめているのに、名前は二つしかない。数が合わず、ビルドに失敗する。
:::

## 構造化束縛のしくみ

`std::pair` や `std::tuple` を構造化束縛で分解できるのは、要素の数と各要素を問い合わせる手立てがあるからである。要素の数は `std::tuple_size`、各要素の型は `std::tuple_element`、各要素そのものは `std::get` で得られる。これら(`std::tuple_size`・`std::tuple_element`・`std::get`)をまとめて、タプルプロトコルと呼ぶことがある。

`T` を `std::tuple<int, double, std::string>` とすると、`std::tuple_size_v<T>` は要素の数の `3`、`std::tuple_element_t<0, T>` は0番目の要素の型 `int` である。各要素は `std::get` で取り出す。

```cpp
std::tuple<int, double, std::string> t = {1, 2.5, "hi"};
std::println("{}", std::get<0>(t));   // 1
std::println("{}", std::get<2>(t));   // hi
```

`auto [a, b, c] = t` は、`std::tuple_size` で名前の数(3つ)が要素の数と合うかを確かめ、`get<0>(t)`・`get<1>(t)`・`get<2>(t)` で各要素を取り出して、`a`・`b`・`c` に結びつける。ここで呼ばれる `get` は、`std::get` を名指ししたものではない。`get<i>(t)` という式の `get` を、`t` の型に関連する名前空間(`std::tuple` なら `std`)も含めて探索して見つけたものである。言語編の構造化束縛の章で見た構造体や配列は、メンバや要素を直接たどって分解した。`std::pair` や `std::tuple` は、このタプルプロトコルを通して分解される。自分で定義した型でも、`std::tuple_size` と `std::tuple_element` を特殊化し、その型と同じ名前空間に `get` を用意すれば、探索でその `get` が見つかり、構造化束縛で分解できる。

:::details メンバの get で分解する型もある
構造化束縛が `get` を見つける手立ては二通りある。型がメンバとして `get<i>()` の形で呼べる関数テンプレート(最初のテンプレート引数が型ではなく値であるもの)を持つなら、`e.get<i>()` を使う。持たないなら、`get<i>(e)` を、`e` の型に関連する名前空間も含めて探索する。実引数の型に応じたこの探索を、英語では argument-dependent lookup と呼ぶ。いずれの場合も、通常の非修飾名の探索は行わない。

`std::tuple`・`std::pair`・`std::array` はメンバの `get` を持たないので、後者の経路で、`std` にある `get` が見つかる。自作の型は、どちらの手立てでも分解できるようにできる。メンバ `get` を用意すれば前者、型と同じ名前空間に非メンバ `get` を置けば後者で見つかる。
:::

## 複数の値を返す

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

ただし、この `minmax` は、要素が一つ以上ある範囲を前提とする。空の範囲には最小値も最大値もなく、`xs[0]` は範囲の外を読む。結果がないかもしれないことを値として表す道具は、この編の後半の `std::optional` で扱う。

ここまでで、値を持つコンテナ、それを巡るアルゴリズム、所有せずに参照する `std::span`、複数の値をまとめる `std::pair`・`std::tuple` を見てきた。次章では、動的なオブジェクトの所有を引き受ける `std::unique_ptr`・`std::shared_ptr` を見ていく。基本編の new と delete の章で、続きの編で扱うとしたものである。
