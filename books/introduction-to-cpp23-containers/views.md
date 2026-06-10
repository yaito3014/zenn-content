---
title: "ビューとレンジアダプタ"
---

`std::ranges` のアルゴリズムは、範囲を巡って、並べ替えたり探したりした。範囲から別の範囲を導きたいこともある。偶数だけを残す、各要素を二倍にする、といった具合である。それを担うのが **ビュー** と、ビューを作る **レンジアダプタ** である。使うには `<ranges>` を取り込む。

## 絞る・変換する

`std::views::filter` は、条件を満たす要素だけを残したビューを作る。`std::views::transform` は、各要素に関数を適用したビューを作る。`|` でつなぐ。

```cpp
std::vector<int> v = {1, 2, 3, 4, 5, 6};
auto r = v | std::views::filter([](int x) { return x % 2 == 0; })
           | std::views::transform([](int x) { return x * 2; });
for (int x : r) std::println("{}", x);   // 4 8 12
```

`v | std::views::filter(...)` は、偶数だけを残したビューである。それに `| std::views::transform(...)` をつなぐと、残った要素を二倍にしたビューになる。`|` は、左の範囲に右のレンジアダプタを適用する。範囲for で巡ると、`2`・`4`・`6` を二倍した `4`・`8`・`12` が出る。

## 遅延して計算する

ビューは、名前のついた範囲を渡すと、`std::span` と同じく、その範囲を所有せず参照する。そのうえ、作った時点では計算しない。範囲for で巡るときに、要素ごとに計算する。途中の結果をためる別のコンテナは作らず、ビューは計算の手順を持つだけである。

```cpp
std::vector<int> v = {1, 2, 3};
auto doubled = v | std::views::transform([](int x) { return x * 10; });
v[0] = 9;
for (int x : doubled) std::println("{}", x);   // 90 20 30
```

`doubled` を作った後で `v[0]` を `9` に変えても、巡ったときの結果に反映される。`doubled` は作った時点では計算しておらず、`v` を参照して、巡るそのときに読んで計算するからである。この、必要になったときに計算するやり方を **遅延評価** と呼ぶ。

## 範囲を作る・切り取る

`std::views::iota` は、連続する整数の範囲を作る。`std::views::take` は先頭から、`std::views::drop` は先頭を飛ばした残りを、切り取ったビューにする。

```cpp
for (int x : std::views::iota(1, 4)) std::println("{}", x);   // 1 2 3

std::vector<int> v = {1, 2, 3, 4, 5};
for (int x : v | std::views::take(3)) std::println("{}", x);  // 1 2 3
for (int x : v | std::views::drop(2)) std::println("{}", x);  // 3 4 5
```

`std::views::iota(1, 4)` は `1` から `4` の手前までの範囲、`v | std::views::take(3)` は `v` の先頭3つ、`v | std::views::drop(2)` は先頭2つを飛ばした残りのビューである。

## コンテナに戻す

ビューは遅延するので、巡った結果をコンテナのようにまとめて持っているわけではない。結果を持つコンテナがほしいときは、`std::ranges::to`(C++23)でビューをコンテナにする。

```cpp
auto w = v | std::views::filter([](int x) { return x > 3; })
           | std::ranges::to<std::vector>();
```

`w` は `std::vector<int>` で、`v` のうち `3` より大きい要素を持つ。ビューを巡って、その結果を新しいコンテナに集める。

## 破棄される範囲を見るビュー

名前のついた範囲を渡したビューは、`std::span` や `std::string_view` と同じく、その範囲を所有しない。元の範囲が破棄されれば、ビューはもう存在しない範囲を見る。

:::message alert
破棄される範囲を見るビュー
```cpp
auto make()
{
    std::vector<int> v = {1, 2, 3};
    return v | std::views::transform([](int x) { return x * 2; });
}
```
`make` は、局所変数 `v` を見るビューを返す。`v` はこの関数を抜けると破棄され、返ってきたビューは、もう存在しない範囲を見る。ビルドは通るが、そのビューを巡ると、結果は定まらず、実行時に深刻な問題が起こりうる。
:::

:::details ムーブされた範囲を渡すと所有する
名前のついた範囲(`v` など)を渡したビューは、その範囲を参照するだけで、所有しない。一方、`std::move(v)` のようにムーブして渡したり、一時的な範囲を渡したりすると、ビューはその範囲を自分の中に持ち、所有する。所有する場合は、元の範囲がビューより先に破棄されることはないので、上の `make` のような問題は起きない。

```cpp
auto make()
{
    std::vector<int> v = {1, 2, 3};
    return std::move(v) | std::views::transform([](int x) { return x * 2; });
}
```
この `make` は、`v` をムーブしたビューを返す。ビューが `v` を所有して返るので、巡っても問題ない。
:::