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

ここまでで、値を持つコンテナ、それを巡るアルゴリズム、所有せずに参照する `std::span`、複数の値をまとめる `std::pair`・`std::tuple` を見てきた。次章では、動的なオブジェクトの所有を引き受ける `std::unique_ptr`・`std::shared_ptr` を見ていく。基本編の new と delete の章で、続きの編で扱うとしたものである。
