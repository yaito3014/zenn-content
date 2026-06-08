---
title: "他のキャスト"
---

基本編の型変換の章で、`static_cast` を使って型変換を明示的に書いた。`static_cast` は、自分の型に定義した `explicit` な変換演算子を呼ぶのにも使う。また、C++ には `static_cast` のほかにも、用途の違うキャストがある。本章では、`static_cast` で `explicit` な変換演算子を呼ぶこと、そして `dynamic_cast`・`const_cast`・`reinterpret_cast` を扱う。

## static_cast と explicit な変換演算子

演算子オーバーロードの章で、自分の型からほかの型への変換を変換演算子で定義した。変換演算子に `explicit` を付けると、その変換は明示的に書いたときだけ行われる。その「明示的に書く」やり方が `static_cast` である。

摂氏の温度を表す `Celsius` から `double` への変換演算子に、`explicit` を付ける。

```cpp
struct Celsius
{
    double value;

    explicit operator double() const
    {
        return value;
    }
};
```

`explicit` が付いているので、`double` が要る場所に `Celsius` をそのまま書く暗黙の変換はできない。`static_cast` で変換先の型を明示すると、変換演算子が呼ばれる。

```cpp
Celsius c = {.value = 25.0};
double d = static_cast<double>(c);
std::println("{}", d);   // 25
```

:::message
explicit な変換を暗黙に書く
```cpp
Celsius c = {.value = 25.0};
double d = c;
```
`Celsius` の変換演算子は `explicit` なので、暗黙の変換は行われない。`static_cast` で明示せずに書くと、ビルドに失敗する。
:::

## dynamic_cast

**dynamic_cast** は、基底クラスの参照やポインタを、その動的な型である派生クラスのポインタへ変換する。動的な型が指定した派生クラスであれば、そのポインタが得られる。仮想関数を持つ基底クラスでは動的な型に応じた振る舞いを呼び分けられたが、その動的な型を派生クラスとして取り出したいときに、これを使う。

```cpp
class Shape
{
public:
    virtual ~Shape() = default;
};

class Square : public Shape
{
public:
    int side = 4;
};

class Circle : public Shape
{
public:
    int radius = 3;
};
```

`Shape const&` から `Square const*` への `dynamic_cast` は、動的な型が `Square` ならそのポインタを、そうでなければ `nullptr` を返す。

```cpp
void check(Shape const& s)
{
    Square const* sq = dynamic_cast<Square const*>(&s);
    if (sq != nullptr)
    {
        std::println("square, side {}", sq->side);
    }
    else
    {
        std::println("not a square");
    }
}
```

```cpp
check(Square{});   // square, side 4
check(Circle{});   // not a square
```

`check(Square{})` では動的な型が `Square` なので、`dynamic_cast` は `Square const*` を返し、`sq != nullptr` が成り立って `side` を読める。`check(Circle{})` では動的な型が `Square` でないので `nullptr` が返り、`else` に進む。`dynamic_cast` は、こうして動的な型を実行時に確かめる。基底から派生を取り出すこの使い方には、仮想関数を持つ型が要る。

参照への `dynamic_cast` もあり、失敗したときの扱いがポインタとは違う。これは例外の章で扱う。

## const_cast

`const` を付けたり外したりするのが **const_cast** である。`const` の参照やポインタしか手元にないが、`const` でない参照やポインタが要る場面で使う。

```cpp
int x = 5;
int const& r = x;
const_cast<int&>(r) = 10;
std::println("{}", x);   // 10
```

`r` は `int const&` で、そのままでは書き換えられない。`const_cast<int&>(r)` で `const` を外すと、`r` が指す `x` を書き換えられる。`x` はもともと `const` ではないので、これは問題ない。

ただし、本当に `const` なオブジェクトを `const_cast` で書き換えると、深刻な問題が起こりうる。`const_cast` で外せるのは型の上の `const` であって、オブジェクトそのものを書き換えてよくなるわけではない。

## reinterpret_cast

ポインタを別の型のポインタに変換したり、ポインタと整数とを変換したりするのが **reinterpret_cast** である。低水準の場面で使う。

```cpp
#include <cstdint>

int n = 7;
int* p = &n;
std::uintptr_t a = reinterpret_cast<std::uintptr_t>(p);
int* q = reinterpret_cast<int*>(a);
std::println("{}", *q);   // 7
```

`reinterpret_cast<std::uintptr_t>(p)` は、ポインタ `p` を整数として取り出す。その整数 `a` を `reinterpret_cast<int*>(a)` でポインタに戻すと、もとの `p` と同じものになり、`*q` は `7` になる。`std::uintptr_t` は、ポインタを格納できる整数型である。多くの処理系が `<cstdint>` で提供するが、規格上は必須ではない。

整数として取り出した値そのものは、処理系によって異なる(規格は値を定めていない)。reinterpret_cast による変換の結果は、こうして処理系に依存することが多く、扱いを誤ると深刻な問題が起こりうる。低水準の必要があるときにだけ使う。
