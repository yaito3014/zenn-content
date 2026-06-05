---
title: "演算子オーバーロード"
---

基本編から、`+` で足し算を、`==` で等値の比較をと、演算子を組み込みの型に使ってきた。これらの演算子には、自分で定義した型での意味も与えられる。本章では、自分の型に `+` や比較の演算子を定義すること、複数の比較のもとになる三方比較 `<=>`、そして自分の型からほかの型への変換を定義する変換演算子を扱う。

## 演算子に意味を与える

組み込みの型では、`+` で足し算ができる。次のような、自分で定義した型では、`+` は既定では使えない。

```cpp
struct Point
{
    int x;
    int y;
};
```

`Point` の二つのオブジェクトを `+` でつなぐと、ビルドに失敗する。

:::message
意味を定義していない演算子を使う
```cpp
Point p = {.x = 1, .y = 2};
Point q = {.x = 3, .y = 4};
Point r = p + q;
```
`Point` には `+` の意味が定義されていない。`p + q` はビルドに失敗する。
:::

`+` や `==` のような演算子に、自分の型での意味を与えることを **演算子オーバーロード** と呼ぶ。演算子は、`operator+` のような名前の関数として定義する。`Point` どうしの `+` を、メンバ関数として定義する。

```cpp
struct Point
{
    int x;
    int y;

    Point operator+(Point const& other) const
    {
        return {.x = x + other.x, .y = y + other.y};
    }
};
```

`operator+` は、`+` の右側の値を引数に取り、`x` と `y` をそれぞれ足した新しい `Point` を返す。これで `p + q` が書ける。

```cpp
Point p = {.x = 1, .y = 2};
Point q = {.x = 3, .y = 4};
Point r = p + q;   // r は {4, 6}
```

`p + q` は、`p.operator+(q)` の呼び出しになる。`+` の左側の `p` がメンバ関数を呼ぶオブジェクトになり、右側の `q` が引数 `other` に渡る。

## 非メンバの演算子と ADL

演算子は、メンバ関数ではなく、二つの引数を取る非メンバの関数としても定義できる。左右の二つの被演算子が、二つの引数になる。

```cpp
namespace geo
{
    struct Point
    {
        int x;
        int y;
    };

    Point operator+(Point const& a, Point const& b)
    {
        return {.x = a.x + b.x, .y = a.y + b.y};
    }
}
```

ここでは `Point` と `operator+` を名前空間 `geo` の中で定義した。この `+` は、`geo` の外から修飾せずに使える。

```cpp
geo::Point p = {.x = 1, .y = 2};
geo::Point q = {.x = 3, .y = 4};
geo::Point r = p + q;   // r は {4, 6}
```

`p + q` は `operator+(p, q)` の呼び出しになる。`+` の形で書くとき、関数の名前に `geo::` のような修飾は付けられない。それでも、実引数 `p`・`q` の型 `geo::Point` の属する `geo` が探索されるため、`geo::operator+` が見つかる。名前空間の章で見た引数依存の名前探索(ADL)である。演算子を非メンバで定義したものを `+` の形で使うには、この探索が働く。

## 比較する

等値の比較は、`operator==` で定義する。

```cpp
struct Point
{
    int x;
    int y;

    bool operator==(Point const& other) const
    {
        return x == other.x && y == other.y;
    }
};
```

`x` と `y` がともに等しいとき、二つの `Point` は等しいとする。これで `==` が書ける。`!=` は、`==` の否定として使えるようになる。

```cpp
Point p = {.x = 1, .y = 2};
Point q = {.x = 1, .y = 2};
Point s = {.x = 3, .y = 4};
std::println("{}", p == q);   // true
std::println("{}", p != s);   // true
```

`p != s` は別に定義していないが、`operator==` を定義すると、`!=` はその否定として扱われる。

## 三方比較

大小の比較には、`<`・`>`・`<=`・`>=` の四つがある。これらを一つずつ定義する代わりに、**三方比較** の `operator<=>` を一つ定義すると、四つの比較が書けるようになる。`<=>` は、左右の大小を一度に表す演算子である。

`= default` と `= delete` の章では、`= default` を特殊メンバ関数に使った。`operator<=>` は特殊メンバ関数ではないが、これにも `= default` を付けて、定義をメンバからの自動生成にまかせられる。

```cpp
#include <compare>

struct Point
{
    int x;
    int y;

    auto operator<=>(Point const&) const = default;
};
```

`= default` の `operator<=>` は、データメンバを宣言した順に比べる。`Point` なら、まず `x` を比べ、等しければ `y` を比べる。これで `<`・`>`・`<=`・`>=` が書ける。

```cpp
Point p = {.x = 1, .y = 2};
Point q = {.x = 1, .y = 3};
std::println("{}", p < q);    // true(x は等しく、y は 2 < 3)
std::println("{}", p >= q);   // false
```

`operator<=>` を `= default` で定義したとき、`==` を別に定義していなければ、`==` も同時に自動で定義される。そのため、大小と等値の両方が要るときは、`<=>` の `= default` だけで、`==`・`!=` を含む六つの比較がそろう。`<=>` の比較には `<compare>` の取り込みが要る。

## 変換演算子

基本編の変換の章では、`int` から `Counter` への変換を、`Counter(int)` という変換コンストラクタで定義した。逆向きに、自分の型からほかの型への変換は、**変換演算子** で定義する。

摂氏の温度を表す `Celsius` から `double` への変換を定義する。`operator` に続けて変換先の型を書く。

```cpp
struct Celsius
{
    double value;

    operator double() const
    {
        return value;
    }
};
```

これで `Celsius` を `double` が要る場所に書くと、`value` が取り出される。

```cpp
Celsius c = {.value = 25.0};
double d = c;   // 変換演算子で double になる
std::println("{}", d);   // 25
```

変換コンストラクタに `explicit` を付けて暗黙の変換を防げたのと同じく、変換演算子にも `explicit` を付けられる。`explicit` を付けると、この変換は明示的に書いたときだけ行われる。明示的な変換の書き方は、キャストの章で扱う。
