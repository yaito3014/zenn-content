---
title: "ビルド時計算"
---

プログラムの計算の多くは実行時に行われるが、一部はビルドの時点で済ませられる。ビルド時に値が決まる式を **定数式** と呼ぶ。本章では、定数式と `constexpr`、`constexpr` 関数、`static_assert`、`consteval` と `constinit`、そして定数式で枝を選ぶ `if constexpr` を扱う。

## 定数式と constexpr

変数に `constexpr` を付けると、定数式で初期化することを要求し、その値はビルド時に決まる。

```cpp
constexpr int size = 5;
```

基本編の const は、「書き換えない」という制約だった。`const` な変数は、初期化の値が実行時に決まることもある。`constexpr` は、値がビルド時に決まることを要求する点で違う。`constexpr` な変数は書き換えられず、その値は定数式として使える。

## constexpr 関数

関数に `constexpr` を付けると、定数式の中で呼べるようになる。つまり、ビルド時に評価できる。

```cpp
constexpr int square(int x) { return x * x; }

constexpr int n = square(5);   // ビルド時に 25 になる
```

`n` は `constexpr` なので、その初期化子 `square(5)` はビルド時に計算され、`n` は `25` になる。`constexpr` 関数は、実行時の値を渡せば、実行時にも使える。

```cpp
int r = 7;
std::println("{}", square(r));   // 49(実行時)
```

`r` は実行時の値なので、この `square(r)` は実行時に評価される。`constexpr` 関数は、ビルド時と実行時のどちらでも使える。

## static_assert

`static_assert` は、定数式の条件をビルド時に検査する。条件が偽なら、ビルドに失敗する。基本編の宣言の章で、名前を導入しない宣言の例として触れたのが、この `static_assert` である。

```cpp
static_assert(square(5) == 25);
```

`square(5)` はビルド時に `25` になり、`25 == 25` が成り立つので、このビルドは通る。条件が成り立たなければ、ビルドの時点で誤りとして示される。

:::message
成り立たない条件を検査する
```cpp
static_assert(square(5) == 24);
```
`square(5)` は `25` で、`25 == 24` は偽である。`static_assert` の条件が偽なので、ビルドに失敗する。
:::

## consteval と constinit

`constexpr` 関数は、ビルド時にも実行時にも使えた。これに対し、`consteval` を付けた関数は、必ずビルド時に評価される。実行時の値では呼べない。

```cpp
consteval int cube(int x) { return x * x * x; }

constexpr int c = cube(3);   // ビルド時に 27 になる
```

`cube` に実行時の値を渡すと、ビルド時に評価できないため、ビルドに失敗する。

:::message
実行時の値を consteval 関数に渡す
```cpp
int r = 3;
int bad = cube(r);
```
`r` は実行時の値なので、`cube(r)` はビルド時に評価できない。`consteval` 関数は実行時の値では呼べず、ビルドに失敗する。
:::

ビルド時の計算だけに使うと決めた関数を、`consteval` で表す。

`constinit` は、静的記憶域期間を持つ変数を、ビルド時に初期化することを要求する。関数の外で宣言した変数は、寿命と記憶域期間の章で見た関数内の `static` 変数と同じく、静的記憶域期間を持つ。

```cpp
constinit int counter = square(2);   // ビルド時に 4 で初期化
```

静的記憶域期間の変数は、初期化のしかたによっては実行時に初期化される。`constinit` を付けると、ビルド時の初期化を要求し、そうでなければビルドに失敗する。`constexpr` と違い、`constinit` は値の書き換えを妨げない。初期化のタイミングだけを定める。

## if constexpr

`if` の条件を `if constexpr` とすると、条件を定数式としてビルド時に評価し、選ばれなかった枝を捨てる。テンプレートの中では、捨てた枝はその型でビルドできなくてもよい。ある操作がその型でできるときだけ使い分けたい、という場面で役立つ。

`.value` を持つ型ならそれを取り出し、持たない型はそのまま返す関数を書く。

```cpp
template <typename T>
auto unwrap(T x)
{
    if constexpr (requires { x.value; })
    {
        return x.value;
    }
    else
    {
        return x;
    }
}
```

`requires { x.value; }` は、コンセプトの章で見た requires 式である(引数リストを省いて、外側の `x` をそのまま試す形)。`x.value` が書けるかどうかを表すビルド時の条件で、書ければ成り立つ。`if constexpr` は、この条件が成り立つ枝だけを残し、もう一方を捨てる。

```cpp
struct Wrap { int value; };

Wrap w = {.value = 42};
std::println("{}", unwrap(w));   // 42
std::println("{}", unwrap(7));   // 7
```

`unwrap(w)` では `requires { x.value; }` が成り立ち、`return x.value;` の枝が残って `42` を返す。`unwrap(7)` では `int` に `.value` がなく条件が成り立たないので、`return x.value;` の枝は捨てられ、`return x;` が残って `7` を返す。ふつうの `if` なら、`int` でも `x.value` を書いた枝がビルドされて失敗する。`if constexpr` は捨てた枝をビルドしないので、これが通る。
