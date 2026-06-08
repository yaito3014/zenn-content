---
title: "値カテゴリ"
---

基本編の参照とポインタの章で、`int& r = a;` のように、名前のついたオブジェクトに別名を付ける参照を見た。式には、オブジェクトを指して続けて使えるものと、値を取り出して使うものとがある。この区別を **値カテゴリ** と呼ぶ。本章では、lvalue と rvalue という値カテゴリと、それぞれに束縛できる参照を扱う。

## lvalue と rvalue

```cpp
int a = 5;
```

`a` という式は、オブジェクトを指す。アドレスを取れ、式が終わったあとも残り、続けて使える。このような式を **lvalue** と呼ぶ。

一方、`5` や `a + 1` という式は、その場かぎりの値で、続けては使わない。値を取り出して使うだけである。このような式を **rvalue** と呼ぶ。

```cpp
a;        // lvalue: 続けて使えるオブジェクト
5;        // rvalue: 取り出して使う値
a + 1;    // rvalue: 計算の結果の値
```

同じ `int` でも、`a` のように続けて使えるオブジェクトと、`a + 1` のように取り出して使う値とがある。値カテゴリは、この違いを表す。

## lvalue 参照と rvalue 参照

基本編で見た `int&` の参照は、lvalue にしか束縛できない。これを **lvalue 参照** と呼ぶ。

```cpp
int a = 5;
int& r = a;   // lvalue の a に束縛できる
```

rvalue には束縛できない。

:::message
lvalue 参照を rvalue に束縛する
```cpp
int& r = 5;
```
`5` は rvalue である。`int&` は lvalue にしか束縛できないので、ビルドに失敗する。
:::

rvalue に束縛するには、`int&&` と書く。これを **rvalue 参照** と呼ぶ。

```cpp
int&& r = 5;   // rvalue の 5 に束縛できる
```

ただし、`r` 自体は名前のついた式なので、`r` と書けばそれは lvalue である。`int&&` という型と、`r` という式の値カテゴリは別のものである。

なお、`int const&` は例外で、lvalue にも rvalue にも束縛できる。rvalue に束縛すると、その一時的な値は参照が使える間は保たれる。基本編で、コピーの重い型を `const` 参照で受け取ってコピーを避けたのは、この性質による。

## 値カテゴリで呼び分ける

lvalue 参照と rvalue 参照は、オーバーロードの引数として使い分けられる。オーバーロード解決の章では値で受け取る関数を並べたが、参照で受け取る関数も同じように並べられる。

```cpp
void f(int& x)  { std::println("lvalue: {}", x); }
void f(int&& x) { std::println("rvalue: {}", x); }
```

`f(int&)` は lvalue を、`f(int&&)` は rvalue を受け取る。

```cpp
int a = 5;
f(a);       // lvalue: 5
f(5);       // rvalue: 5
f(a + 1);   // rvalue: 6
```

`f(a)` では `a` が lvalue なので `f(int&)` が、`f(5)` や `f(a + 1)` では実引数が rvalue なので `f(int&&)` が呼ばれる。実引数の値カテゴリによって束縛できる参照が決まり、それがオーバーロード解決でどの候補を選ぶかを決める。

rvalue は、その場かぎりの値で、続けて使うことを前提としない。この性質を使って rvalue の中身を移すムーブは、コピーとムーブの章で扱う。続けて使うオブジェクトと、その場かぎりの値を、値カテゴリはこのように区別する。

## decltype

式の型を取り出すのが `decltype` である。名前をそのまま書くと、その名前の宣言された型になる。

```cpp
int x = 5;
decltype(x) a = 10;   // a は int
```

`decltype(x)` は、`x` の宣言された型 `int` である。

名前ではない式に使うと、`decltype` はその式の値カテゴリを型に反映する。lvalue の式なら、lvalue 参照(`T&`)の型になる。`x` を括弧で囲んだ `(x)` は lvalue の式なので、`decltype((x))` は `int&` になる。

```cpp
decltype((x)) r = x;   // r は int&
r = 20;
std::println("{}", x);   // 20
```

`r` は `x` への参照になり、`r = 20` は `x` を書き換える。

参照を付けずに書く `auto` は、参照や `const` を落として値の型にする。値カテゴリを保ったまま型を決めたいときは、`decltype(auto)` を使う。`decltype(auto)` は、`decltype` と同じ規則で型を決める。

```cpp
int x = 5;
auto p = (x);             // p は int(参照は落ちる)
decltype(auto) q = (x);   // q は int&((x) の値カテゴリを保つ)
q = 20;
std::println("{} {}", p, x);   // 5 20
```

`auto` の `p` は `x` をコピーした `int` で、`x` とは別である。`decltype(auto)` の `q` は `(x)` の値カテゴリを保って `int&` になり、`q = 20` で `x` を書き換える。関数の戻り値の型でも同じで、`auto` は値を返し、`decltype(auto)` は式の値カテゴリを保った型を返す。

:::details 値カテゴリの分類
式は、lvalue・xvalue・prvalue の三つに分かれる。本章で rvalue と呼んだものは、このうち xvalue と prvalue をまとめた呼び名である。また、lvalue と xvalue をまとめて glvalue と呼ぶ。本章は、lvalue と rvalue の区別にとどめる。
:::
