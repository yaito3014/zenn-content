---
title: "例外"
---

エラーが起きた場所では、その後の処理を決められないことがある。エラーを呼び出し側に伝えて、別の場所で対処する仕組みの一つが **例外** である。本章では、例外を投げて捕まえること、例外が伝わるときのスタックの巻き戻しと RAII の関わり、そして `noexcept` を扱う。エラーは戻り値として表すこともでき、その方法はこの先の編で扱う。例外は、それとは別の系統の仕組みである。

## 例外を投げて捕まえる

割り算で、`0` での除算をエラーとして扱う。エラーを表す型を用意し、`throw` で投げる。

```cpp
struct DivideByZero {};

int divide(int a, int b)
{
    if (b == 0) throw DivideByZero{};
    return a / b;
}
```

`b` が `0` のとき、`throw DivideByZero{}` で `DivideByZero` の値を投げる。`try` ブロックの中で投げられた例外を、続く `catch` で捕まえる。

```cpp
try
{
    std::println("{}", divide(10, 2));   // 5
    std::println("{}", divide(10, 0));   // ここで投げる
    std::println("ここには来ない");
}
catch (DivideByZero const&)
{
    std::println("0 で割った");
}
```

`divide(10, 2)` は `5` を出力する。次の `divide(10, 0)` が例外を投げると、`try` ブロックの残りは実行されず、`catch` に移る。だから `ここには来ない` は出力されず、`catch` の `0 で割った` が出力される。出力は `5` と `0 で割った` である。

## スタックの巻き戻しと RAII

例外は、投げた場所から `catch` まで伝わる。その途中のブロックを抜けていく。寿命と記憶域期間の章で見たとおり、ブロックを抜けるときには、その中の自動記憶域期間のオブジェクトが破棄される。例外でブロックを抜けるときも同じで、破棄のときにデストラクタが呼ばれる。この、例外が `catch` まで伝わる過程でブロックを抜けていくことを **スタックの巻き戻し** と呼ぶ。

デストラクタが呼ばれる様子を見る。

```cpp
struct Guard
{
    ~Guard() { std::println("guard を破棄"); }
};

void f()
{
    Guard g;
    throw DivideByZero{};
}
```

`f` の中で `Guard g` を作ったあとに例外を投げる。例外が `f` を抜けるとき、`g` が破棄され、デストラクタが呼ばれる。

```cpp
try
{
    f();
}
catch (DivideByZero const&)
{
    std::println("捕まえた");
}
```

出力は `guard を破棄` と `捕まえた` である。例外が `f` を抜けて `catch` に届くまでの間に、`g` のデストラクタが呼ばれている。

RAII で資源の後始末をデストラクタに結びつけておけば、例外がそのブロックを通り抜けるときにも、後始末は行われる。スタックの巻き戻しでデストラクタが呼ばれるからである。

## noexcept

関数に `noexcept` を付けると、その関数は例外を投げないと宣言する。

```cpp
int add(int a, int b) noexcept
{
    return a + b;
}
```

`noexcept` を付けた関数が、それでも例外を投げると、プログラムはそこで終了する。投げないと分かっている関数に、`noexcept` を付けておく。

:::details 参照への dynamic_cast
他のキャストの章で、`dynamic_cast` がポインタの変換に失敗すると `nullptr` を返すことを見た。参照への `dynamic_cast` は、`nullptr` を返せない。失敗すると、代わりに例外を投げる。投げられるのは、`std::bad_cast` という標準ライブラリの例外型に当たる例外で、これも `try`/`catch` で捕まえられる。

```cpp
try
{
    Square const& sq = dynamic_cast<Square const&>(shape);
}
catch (...)
{
    std::println("変換に失敗");
}
```

`catch (...)` は、どんな型の例外も捕まえる。`shape` の動的な型が `Square` でなければ、`dynamic_cast` が例外を投げ、この `catch` に移る。標準ライブラリの例外型そのものは、この先の編で扱う。
:::
