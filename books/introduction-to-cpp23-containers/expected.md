---
title: "値かエラーか"
---

`std::optional` は、値があるかないかを表した。「ない」ときに、なぜないのか、どんなエラーだったのかを持たせたいこともある。それを表すのが `std::expected` である。

## 値か、エラーか

`std::expected<T, E>` は、`T` の値か、`E` のエラーかの、どちらかを表す。使うには `<expected>` を取り込む。正の数だけ受け取る関数を、`std::expected` で書く。

```cpp
std::expected<int, std::string> parse_positive(int n)
{
    if (n > 0) return n;
    return std::unexpected("not positive");
}
```

値を返すときはそのまま `return n;`、エラーを返すときは `std::unexpected` に包んで返す。

```cpp
auto r = parse_positive(5);
if (r) std::println("{}", *r);          // 5
auto e = parse_positive(-1);
if (!e) std::println("{}", e.error());  // not positive
```

`if (r)` で値を持つかを調べ、値は `*r`、エラーは `error` で取り出す。

## エラーを値として表す

言語編の例外の章で、エラーを投げて別の場所で捕まえる仕組みを見た。同じ章で、エラーを戻り値として表す方法もあると断り、この編で扱うとした。それが `std::expected` である。エラーが戻り値に表れるので、呼ぶ側は、戻り値を見てエラーかどうかを確かめる。例外とは別の系統の仕組みである。

## エラーを持つ expected を取り出す

`std::optional` の `*` と同じく、`std::expected` の `*` も、値を持つかを確かめない。エラーを持つ `std::expected` を `*` で取り出すと、持っていない値を読む。

:::message alert
エラーを持つ expected を取り出す
```cpp
std::expected<int, std::string> e = std::unexpected("oops");
std::println("{}", *e);
```
`e` はエラーを持ち、値を持たない。`*e` は、持っていない値を取り出している。ビルドは通るが、結果は定まらず、実行時に深刻な問題が起こりうる。
:::

`value` は、エラーを持つときに `std::bad_expected_access` を投げる。`std::optional` の `value` と同じ考え方である。

## 値があるときだけつなぐ

`std::expected` も、`std::optional` と同じ monadic operations を持つ。`transform` や `and_then` は、値を持つときだけ続きをつなぎ、エラーを持つときは、そのエラーがそのまま伝わる。

```cpp
std::expected<int, std::string> e = 5;
e.transform([](int x) { return x * 10; });          // 50 を持つ

std::expected<int, std::string> err = std::unexpected("bad");
err.transform([](int x) { return x * 10; });        // "bad" のまま
```

`e` は値を持つので、関数を適用して `50` を持つ。`err` はエラーを持つので、関数は適用されず、`"bad"` のエラーがそのまま残る。`std::optional` で空が伝わったように、`std::expected` ではエラーが伝わる。

`std::optional` と `std::expected` は、値か、その代わりのもの(なし、あるいはエラー)かを表した。次章では、いくつかの型のうち一つを持つ `std::variant` を見る。
