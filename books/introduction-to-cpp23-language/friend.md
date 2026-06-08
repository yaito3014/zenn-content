---
title: "friend"
---

前章では、`struct` の public なメンバに、非メンバの演算子から触れた。だが、基本編のアクセス制御の章で見たように、private なメンバには、そのクラスのメンバ関数からしか触れられない。private なデータを持つ型に、非メンバの関数や別のクラスから触れたいことがある。**friend** 宣言は、指定した非メンバ関数や別のクラスに、private なメンバへのアクセスを許す。本章では、friend 宣言、別のクラスを friend にすること、そしてクラス本体の中で定義する hidden friend を扱う。

## friend で private へのアクセスを許す

private なデータを持つ `Point` を考える。

```cpp
class Point
{
    int x;
    int y;

public:
    Point(int a, int b) : x(a), y(b) {}
};
```

この `Point` に、非メンバの `operator==` を定義したい。だが、`x`・`y` は private なので、メンバ関数でない `operator==` からは触れられない。

:::message
非メンバの関数から private なメンバに触れる
```cpp
bool operator==(Point const& a, Point const& b)
{
    return a.x == b.x && a.y == b.y;
}
```
`x`・`y` は private で、`Point` のメンバ関数からしか触れられない。メンバ関数でない `operator==` は `a.x` に触れられず、ビルドに失敗する。
:::

そこで、`operator==` を `Point` の **friend** にする。クラス本体に `friend` を付けて関数を宣言すると、その関数に private へのアクセスを許せる。

```cpp
class Point
{
    int x;
    int y;

public:
    Point(int a, int b) : x(a), y(b) {}

    friend bool operator==(Point const& a, Point const& b);
};

bool operator==(Point const& a, Point const& b)
{
    return a.x == b.x && a.y == b.y;
}
```

`friend bool operator==(...);` は、この `operator==` を `Point` の friend として宣言する。friend にした関数は、メンバ関数ではなく非メンバの関数のままだが、`Point` の private な `x`・`y` に触れられる。friend は、クラスのメンバではなく、private へのアクセスを許された外部の関数である。

```cpp
Point p(1, 2);
Point q(1, 2);
Point s(3, 4);
std::println("{} {}", p == q, p != s);   // true true
```

## 別のクラスを friend にする

friend は、関数だけでなくクラスにも与えられる。`friend class` でクラスを friend にすると、そのクラスのメンバ関数すべてに private へのアクセスを許す。

```cpp
class Account
{
    int balance;

public:
    Account(int b) : balance(b) {}

    friend class Auditor;
};

class Auditor
{
public:
    int inspect(Account const& a)
    {
        return a.balance;
    }
};
```

`friend class Auditor;` により、`Auditor` のメンバ関数は `Account` の private な `balance` に触れられる。`Auditor::inspect` は、`Account` のメンバ関数でないのに、`a.balance` を読める。

```cpp
Account acc(100);
Auditor au;
std::println("{}", au.inspect(acc));   // 100
```

## hidden friend

ここまでは、`friend` 宣言をクラス本体に書き、関数の定義を外に置いた。friend 関数は、クラス本体の中で定義することもできる。

```cpp
class Point
{
    int x;
    int y;

public:
    Point(int a, int b) : x(a), y(b) {}

    friend bool operator==(Point const& a, Point const& b)
    {
        return a.x == b.x && a.y == b.y;
    }
};
```

このように、クラス本体の中で定義した friend 関数を、英語で **hidden friend** と呼ぶ(定訳はない)。hidden friend は非メンバの関数で、private なメンバに触れられる。

hidden friend には、見つかり方に特徴がある。最初の例では、`operator==` の定義をクラスの外、名前空間の側にも置いた。この定義は名前空間の側の宣言にもなり、通常の名前探索で見つかる。一方 hidden friend は、クラスの外に宣言を持たない。そのため通常の名前探索では見つからず、名前空間の章で見た ADL でだけ見つかる。`p == q` のように `Point` を被演算子にして書くと、被演算子の型 `Point` に結びついたこの `operator==` が、ADL によって見つかる。

```cpp
Point p(1, 2);
Point q(1, 2);
std::println("{}", p == q);   // true
```

通常の名前探索の対象にならないため、hidden friend は、まわりの名前空間に `operator==` という名前を加えない。private なメンバに触れる非メンバの演算子を、名前空間に名前を持ち込まずに定義できる。`==` や `<=>` のような演算子を、private なメンバを持つ型に与えるとき、この hidden friend の形で書ける。
