---
title: "値があるかないか"
---

ここまで、値を並べて持つコンテナと、それを操作するアルゴリズムを見てきた。ここからは、一つの値を表す型を見る。最初は、値があるかないかを表す `std::optional` である。

## 値があるか、ないか

`std::optional<T>` は、`T` の値を一つ持つか、何も持たないかの、どちらかを表す。使うには `<optional>` を取り込む。

```cpp
std::optional<int> o = 5;
std::optional<int> none = std::nullopt;
```

`o` は値 `5` を持ち、`none` は何も持たない。何も持たないことは `std::nullopt` で表す。値を持つかは、そのまま `if` で調べられ、値は `*` で取り出す。

```cpp
if (o) std::println("{}", *o);   // 5
```

## 値がないかもしれない関数

値を返せないことがある関数で、`std::optional` が活きる。範囲から最初の偶数を返す関数を考える。偶数がなければ、返す値がない。

```cpp
std::optional<int> first_even(std::vector<int> const& xs)
{
    for (int x : xs)
        if (x % 2 == 0) return x;
    return std::nullopt;
}
```

```cpp
std::vector<int> v = {1, 3, 4, 7};
auto r = first_even(v);
if (r) std::println("{}", *r);   // 4
```

`first_even` は、偶数があればそれを、なければ `std::nullopt` を返す。呼ぶ側は、`if (r)` で値があるかを確かめてから取り出す。値がないことを `-1` のような特別な値で表すと、`-1` という答えと「ない」印とを見分けられない。`std::optional` は、「ない」を値とは別に表す。

## 空の optional を取り出す

`*` は、値があるかを確かめない。何も持たない `std::optional` を `*` で取り出すと、持っていない値を読む。

:::message alert
空の optional を取り出す
```cpp
std::optional<int> o = std::nullopt;
std::println("{}", *o);
```
`o` は何も持たない。`*o` は、持っていない値を取り出している。ビルドは通るが、結果は定まらず、実行時に深刻な問題が起こりうる。
:::

確かめてから取り出すか、`value` を使う。`value` は、何も持たなければ `std::bad_optional_access` を投げる。これは、文字列の章で見た `at` と同じ考え方である。値がないときの既定値を返す `value_or` もある。

```cpp
std::optional<int> o = std::nullopt;
o.value();         // std::bad_optional_access を投げる
o.value_or(-1);    // -1
```

## 値があるときだけつなぐ

`std::optional` の値を使うには、`if` で値があるかを確かめてきた。値に処理をしてまた `std::optional` を得る、という手順をいくつもつなぐと、そのたびに `if` を書くことになる。`std::optional` は、これを `if` なしでつなぐ操作を持つ。

`transform` は、値があれば関数を適用し、結果をまた `std::optional` に包む。値がなければ、何もせず空のままにする。

```cpp
std::optional<int> o = 5;
o.transform([](int x) { return x * 2; });        // 10 を持つ

std::optional<int> empty = std::nullopt;
empty.transform([](int x) { return x * 2; });    // 空のまま
```

`o` は値を持つので、関数を適用して `10` を持つ `std::optional` になる。`empty` は空なので、関数は適用されず、空のままになる。

`transform` は `std::optional` を返すので、つなげられる。

```cpp
std::optional<int> o = 5;
o.transform([](int x) { return x * 2; })
 .transform([](int x) { return x + 1; });        // 11 を持つ

std::optional<int> empty = std::nullopt;
empty.transform([](int x) { return x * 2; })
     .transform([](int x) { return x + 1; });    // 空のまま
```

空の `std::optional` に `transform` しても、関数は適用されず、空のまま次へ伝わる。だから、つないだ途中で空になれば、それ以降は適用されない。値があるときの処理だけを並べられる。

`transform` のほかに、つなぐ関数自身が `std::optional` を返すときに使う `and_then` と、空のときに代わりの `std::optional` を与える `or_else` がある。これらをまとめて monadic operations と呼ぶ。

`std::optional` は、値があるかないかを表した。次章では、値か、あるいはエラーかを表す `std::expected` を見る。
